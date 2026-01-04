# Neovim Collaborative Editor Plugin Roadmap

## Project Overview

This project is building a Neovim plugin for real-time collaborative editing. The plugin connects to a collaborative editing server that uses **Automerge** as its CRDT backend.

### History: Yjs to Automerge Migration

The project originally planned to use **Yjs** as the CRDT library. A Go helper and Lua plugin skeleton were started for Yjs integration. However, the project pivoted to **Automerge** because:

1. Yjs's aggressive garbage collection deletes historical chunks, which conflicts with version-history requirements
2. Automerge has better history preservation and "time travel" capabilities
3. Automerge fits better with an event-sourcing model

The existing code was refactored to work with Automerge. The key insight was adopting a **Teamtype/Ethersync-style protocol** where:
- The server handles all Automerge CRDT logic
- The plugin just sends/receives simple JSON text operations
- No CRDT library needed in the plugin itself

## Architecture

```
┌─────────────────┐     stdin/stdout      ┌─────────────────┐     WebSocket     ┌─────────────────┐
│                 │     JSON-RPC          │                 │                   │                 │
│  Neovim (Lua)   │◄────────────────────►│   Go Helper     │◄─────────────────►│ Automerge Server│
│                 │                       │                 │                   │                 │
└─────────────────┘                       └─────────────────┘                   └─────────────────┘
```

**Why Go Helper (not direct Lua WebSocket):**
- Lua's WebSocket libraries are fragile
- Go helper bridges Neovim's stdin/stdout to WebSocket cleanly
- Handles JSON framing, reconnection, graceful shutdown

**Why Server Handles All CRDT Logic:**
- No Lua Automerge bindings exist
- Automerge-go requires CGO (complex cross-platform distribution)
- Server already has Automerge - avoid duplicating CRDT state
- Protocol layer just ensures ordered delivery and conflict detection

## File Structure

```
collab-editor/
├── nvim/
│   ├── go-helper/
│   │   ├── main.go           # WebSocket bridge, JSON-RPC handler
│   │   └── go.mod            # Go module definition
│   ├── lua/collab-editor/
│   │   ├── init.lua          # Main entry, commands, connection management
│   │   ├── protocol.lua      # Message construction/sending
│   │   ├── buffer.lua        # Change tracking (on_bytes), remote edit application
│   │   └── cursors.lua       # Remote cursor display via extmarks
│   ├── plugin/
│   │   └── collab-editor.vim # Vim plugin loader
│   └── README.md             # User documentation
└── docs/
    └── server-protocol-example.go  # Server implementation guide
```

## Protocol Specification (Teamtype-Compatible JSON-RPC)

### Data Types

```lua
-- All positions are 0-indexed and use Unicode character counts (not bytes)
Position = { line: number, character: number }
Range = { start: Position, end: Position }
Edit = { range: Range, replacement: string }
Delta = Edit[]  -- Non-overlapping, all ranges refer to pre-edit document state
```

### Messages: Plugin → Server

| Message | Fields | Description |
|---------|--------|-------------|
| `open` | `{uri, content}` | Take ownership of document, start syncing |
| `close` | `{uri}` | Release ownership, stop receiving updates |
| `edit` | `{uri, revision, delta}` | Send local changes |
| `cursor` | `{uri, ranges}` | Send cursor/selection positions |

### Messages: Server → Plugin

| Message | Fields | Description |
|---------|--------|-------------|
| `edit` | `{uri, revision, delta}` | Apply remote changes |
| `cursor` | `{userid, name, uri, ranges}` | Show remote user's cursor |

### Revision System (Critical for Correctness)

Each document tracks TWO revision counters:
- **editor_revision**: Increments after each LOCAL edit is sent
- **daemon_revision**: Increments after each REMOTE edit is received

**When sending an edit:**
```lua
send({ type = "edit", uri = uri, revision = daemon_revision, delta = delta })
editor_revision = editor_revision + 1
```

**When receiving an edit:**
```lua
if msg.revision ~= editor_revision then
    -- Document state mismatch - ignore this edit
    -- Server will retry with correct revision
    return
end
apply_delta(msg.delta)
daemon_revision = daemon_revision + 1
```

This prevents applying edits to the wrong document state (race conditions).

## Implementation Checklist

### Phase 1: Go Helper (Estimated: 2-3 hours)

The Go helper has been simplified from the Yjs version. It needs:

- [x] WebSocket connection with `?room=` parameter
- [x] stdin/stdout JSON-RPC protocol
- [x] Graceful shutdown handling
- [x] Error forwarding to Neovim
- [ ] **TEST**: Verify WebSocket connects to server
- [ ] **TEST**: Verify JSON messages pass through correctly

**Build command:**
```bash
cd nvim/go-helper
go mod tidy
go build -o go-helper
```

### Phase 2: Lua Plugin Core (Estimated: 3-4 hours)

Files: `init.lua`, `protocol.lua`

- [x] Plugin setup and configuration
- [x] Commands: `:CollabConnect`, `:CollabDisconnect`, `:CollabOpen`, `:CollabClose`, `:CollabInfo`
- [x] Process management (start/stop Go helper)
- [x] Message sending infrastructure
- [ ] **TEST**: `:CollabConnect` spawns Go helper
- [ ] **TEST**: `:CollabOpen` sends `open` message
- [ ] **TEST**: `:CollabClose` sends `close` message

### Phase 3: Buffer Change Tracking (Estimated: 3-4 hours)

File: `buffer.lua`

Key technical challenge: Converting buffer changes to protocol deltas.

- [x] Use `nvim_buf_attach` with `on_bytes` callback
- [x] Capture change location (byte positions)
- [x] Convert bytes → Unicode characters (protocol requirement)
- [x] Compute delta from change parameters
- [x] Send delta with current `daemon_revision`
- [ ] **TEST**: Type in buffer → verify correct delta sent
- [ ] **TEST**: Delete text → verify correct delta sent
- [ ] **TEST**: Multi-byte characters (emoji, CJK) handled correctly

**Critical: Unicode Handling**

Protocol uses Unicode character positions, Neovim uses byte positions:
```lua
function byte_to_char(bufnr, row, byte_col)
    local line = vim.api.nvim_buf_get_lines(bufnr, row, row + 1, false)[1] or ""
    local substr = string.sub(line, 1, byte_col)
    return vim.fn.strchars(substr)
end
```

### Phase 4: Remote Edit Application (Estimated: 2-3 hours)

File: `buffer.lua`

Key technical challenge: Feedback loop prevention.

- [x] Apply incoming deltas to buffer
- [x] Prevent feedback loop (don't re-send applied edits)
- [x] Revision checking before applying
- [ ] **TEST**: Remote edit appears in buffer
- [ ] **TEST**: Applying remote edit doesn't trigger send

**Critical: Feedback Loop Prevention**
```lua
local ignore_changes = false

function apply_remote_edit(delta)
    ignore_changes = true
    -- Apply edits to buffer using nvim_buf_set_text
    vim.schedule(function()
        ignore_changes = false
    end)
end

-- In on_bytes callback:
if ignore_changes then return end
```

### Phase 5: Remote Cursor Display (Estimated: 2 hours)

File: `cursors.lua`

- [x] Create extmarks for remote cursors
- [x] Color cycling for different users
- [x] Show username as virtual text
- [x] Highlight selections
- [x] Clear cursors when user disconnects
- [ ] **TEST**: See other user's cursor
- [ ] **TEST**: Cursor updates as they type
- [ ] **TEST**: Cursor disappears on disconnect

### Phase 6: Server-Side Implementation (Estimated: 4-6 hours)

**This is on your Automerge server, not in the plugin.**

File: `docs/server-protocol-example.go` shows what to implement.

The server needs a WebSocket endpoint that:

1. **Accepts connections:** `ws://server/ws?room=<roomid>`

2. **Tracks per-client state:**
```go
type ClientDoc struct {
    EditorRevision int  // How many edits client has sent
    DaemonRevision int  // How many edits client has received
}
clientDocs := map[string]map[string]*ClientDoc{}  // [clientID][docURI]
```

3. **Handles messages:**
   - `open`: Register client for document, send current content if needed
   - `edit`: Validate revision, apply to Automerge, broadcast to others
   - `cursor`: Broadcast to others
   - `close`: Unregister client from document

4. **Broadcasts with correct revisions:**
```go
// When broadcasting an edit to client:
clientDoc := clientDocs[clientID][uri]
send(EditNotification{
    URI:      uri,
    Revision: clientDoc.EditorRevision,  // What client expects
    Delta:    delta,
})
clientDoc.DaemonRevision++
```

### Phase 7: Integration Testing (Estimated: 4-6 hours)

**Test Scenario 1: Basic Sync**
1. Start server
2. Open two Neovim instances
3. Both: `:CollabConnect ws://localhost:8080/ws testroom`
4. Both: `:e /tmp/test.txt` then `:CollabOpen`
5. Type in one → appears in other
6. Type in other → appears in first

**Test Scenario 2: Concurrent Edits**
1. Both users type simultaneously
2. Verify both see consistent final state

**Test Scenario 3: Cursors**
1. Move cursor in one editor
2. See cursor indicator in other editor

**Test Scenario 4: Disconnect/Reconnect**
1. Kill one client's connection
2. Verify other client continues working
3. Reconnect first client
4. Verify state syncs

## Key Technical Gotchas

### 1. Byte vs Character Positions
- Neovim APIs use **byte** positions
- Protocol uses **Unicode character** positions
- Must convert at every boundary
- Test with emoji 👋 and CJK characters 中文

### 2. Feedback Loops
- When applying remote edit, must suppress `on_bytes` callback
- Use flag + `vim.schedule()` to reset after event loop tick

### 3. Range Validity
- All ranges in a delta refer to document state BEFORE the delta
- Apply edits in order, adjusting positions as you go
- Or apply in reverse order (end-to-start) to avoid adjustment

### 4. Buffer vs File
- Plugin tracks buffers, not files
- URI should uniquely identify the collaborative document
- Consider using room + filename as URI

### 5. Line Endings
- Neovim uses `\n` internally
- Protocol should normalize line endings

## Commands Available After Completion

```vim
:CollabConnect [server] [room]  " Connect to server and join room
:CollabDisconnect               " Disconnect from server
:CollabOpen                     " Share current buffer
:CollabClose                    " Unshare current buffer
:CollabInfo                     " Show connection status
```

## Configuration

```lua
require('collab-editor').setup({
    server_url = 'ws://localhost:8080/ws',
    default_room = 'default',
    debug = false,
    cursor_colors = {
        '#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4',
        '#FFEAA7', '#DDA0DD', '#98D8C8', '#F7DC6F'
    },
})
```

## Estimated Total Time

| Phase | Time |
|-------|------|
| Go Helper testing | 2-3 hours |
| Lua Plugin Core testing | 3-4 hours |
| Buffer Change Tracking | 3-4 hours |
| Remote Edit Application | 2-3 hours |
| Remote Cursor Display | 2 hours |
| Server-Side Implementation | 4-6 hours |
| Integration Testing | 4-6 hours |
| **Total** | **20-28 hours** |

With existing code already generated, focus is on:
1. Building and testing Go helper
2. Implementing server WebSocket endpoint
3. Integration testing

## Files to Review

Before making changes, review these generated files:

1. `nvim/go-helper/main.go` - WebSocket bridge
2. `nvim/lua/collab-editor/init.lua` - Plugin entry point
3. `nvim/lua/collab-editor/buffer.lua` - Buffer tracking logic
4. `docs/server-protocol-example.go` - Server requirements

## Next Steps

1. Add `nvim/go-helper/go-helper` to `.gitignore`
2. Run `go mod tidy` in go-helper directory
3. Build Go helper: `go build -o go-helper`
4. Implement WebSocket endpoint on Automerge server
5. Test with two Neovim instances

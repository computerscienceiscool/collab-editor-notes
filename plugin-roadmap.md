 # Neovim Collaborative Editor Plugin Roadmap

## Project Overview

This project adds a Neovim plugin to an existing collaborative editor that uses **Automerge** for real-time sync. The web frontend already works. The goal is to let Neovim users join the same editing sessions.

## Background: Yjs to Automerge Migration

The project originally planned to use Yjs. Some Go helper and Lua plugin skeleton code was started. The project switched to Automerge because Yjs's aggressive garbage collection deletes historical chunks needed for version history.

**The web editor is already working with Automerge.** The Neovim plugin must integrate with the existing system, not create a new protocol.

## Existing Architecture (From docs/architecture.md)

```
┌─────────────────────────────────────────────────────────────┐
│                      Browser (Client)                        │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐                           │
│  │  CodeMirror │◄─┤  Automerge  │                           │
│  │   Editor    │  │   Handle    │                           │
│  └─────────────┘  └──────┬──────┘                           │
│                          │                                   │
│                   ┌──────┴──────┐                            │
│                   │  Automerge  │                            │
│                   │    Repo     │                            │
│                   └──────┬──────┘                            │
│                          │                                   │
│         ┌────────────────┼────────────────┐                  │
│         ▼                ▼                ▼                  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │  WebSocket  │  │  IndexedDB  │  │  WebSocket  │          │
│  │  (Sync)     │  │   Storage   │  │  Awareness  │          │
│  │  Port 1234  │  │             │  │  Port 1235  │          │
│  └─────────────┘  └─────────────┘  └─────────────┘          │
└─────────────────────────────────────────────────────────────┘
```

**Key facts:**
- Sync server on port 1234 uses **binary CBOR protocol** (not JSON)
- Awareness server on port 1235 uses **JSON protocol**
- Documents are Automerge docs with structure: `{ content: "text here" }`
- Text changes use `Automerge.updateText(doc, ['content'], newText)`
- Document IDs look like: `P8ikUBLFpvXuVSCVWyiBec9U54Y`
- URL format: `http://localhost:8080/?doc=DOCUMENT_ID`

## What the Neovim Plugin Must Do

To join an existing collaborative session, the plugin must:

1. **Speak the Automerge sync protocol** (binary CBOR, not JSON-RPC)
2. **Connect to the same sync server** (ws://localhost:1234)
3. **Connect to the awareness server** (ws://localhost:1235) for cursors
4. **Use the same document structure** (`{ content: "..." }`)
5. **Use the same document IDs** (so Neovim and browser edit same doc)

## Architecture Decision: Node.js Helper

The previous roadmap proposed a Go helper with a custom JSON protocol. **That was wrong.**

**Correct approach:** Use a **Node.js helper** that uses the same `@automerge/automerge-repo` library as the web client.

```
┌─────────────────┐     stdin/stdout      ┌─────────────────┐
│                 │     JSON messages     │                 │
│  Neovim (Lua)   │◄────────────────────►│  Node.js Helper │
│                 │                       │                 │
└─────────────────┘                       └────────┬────────┘
                                                   │
                                          Uses automerge-repo
                                                   │
                              ┌────────────────────┼────────────────────┐
                              ▼                    ▼                    ▼
                       ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
                       │  WebSocket  │      │    File     │      │  WebSocket  │
                       │  (Sync)     │      │   Storage   │      │  Awareness  │
                       │  Port 1234  │      │             │      │  Port 1235  │
                       └─────────────┘      └─────────────┘      └─────────────┘
```

**Why Node.js instead of Go:**
- Uses exact same Automerge library as web client
- Guaranteed protocol compatibility
- No CGO complexity
- Easier to maintain (same codebase/packages)

**The JSON between Neovim and the Node helper is internal only.** The Node helper handles all Automerge protocol complexity.

## File Structure

```
collab-editor/
├── nvim/
│   ├── node-helper/
│   │   ├── package.json        # Dependencies: automerge-repo, ws
│   │   └── index.js            # Automerge client, stdin/stdout JSON bridge
│   ├── lua/collab-editor/
│   │   ├── init.lua            # Plugin entry, commands, process management
│   │   ├── buffer.lua          # Buffer change tracking, remote edit application
│   │   └── cursors.lua         # Remote cursor display
│   ├── plugin/
│   │   └── collab-editor.vim   # Vim plugin loader
│   └── README.md
└── docs/
    └── plugin-roadmap.md       # This file
```

## Internal Protocol (Neovim ↔ Node Helper)

Simple JSON over stdin/stdout. This is NOT the Automerge protocol - it's just for communication between Neovim and the helper.

### Neovim → Helper

```json
{"type": "connect", "syncUrl": "ws://localhost:1234", "awarenessUrl": "ws://localhost:1235"}
{"type": "open", "docId": "P8ikUBLFpvXuVSCVWyiBec9U54Y"}
{"type": "create"}
{"type": "edit", "content": "full document text"}
{"type": "cursor", "ranges": [{"start": 10, "end": 10}]}
{"type": "close"}
{"type": "disconnect"}
```

### Helper → Neovim

```json
{"type": "connected"}
{"type": "opened", "docId": "P8ikUBLFpvXuVSCVWyiBec9U54Y", "content": "initial text"}
{"type": "created", "docId": "NEW_DOC_ID"}
{"type": "changed", "content": "updated full text"}
{"type": "cursor", "userId": "abc", "name": "Alice", "ranges": [...]}
{"type": "error", "message": "..."}
{"type": "disconnected"}
```

**Note:** For simplicity, we send full document content on each change. This is fine for typical document sizes. Optimization (sending diffs) can come later.

## Implementation Phases

### Phase 1: Node.js Helper (4-6 hours)

Create `nvim/node-helper/index.js`:

```javascript
import { Repo } from '@automerge/automerge-repo';
import { BrowserWebSocketClientAdapter } from '@automerge/automerge-repo-network-websocket';
import { NodeFSStorageAdapter } from '@automerge/automerge-repo-storage-nodefs';
import { updateText } from '@automerge/automerge';
import * as readline from 'readline';

// Read JSON lines from stdin
const rl = readline.createInterface({ input: process.stdin });

let repo = null;
let handle = null;

rl.on('line', async (line) => {
  const msg = JSON.parse(line);
  
  switch (msg.type) {
    case 'connect':
      repo = new Repo({
        network: [new BrowserWebSocketClientAdapter(msg.syncUrl)],
        storage: new NodeFSStorageAdapter('./automerge-data'),
      });
      send({ type: 'connected' });
      break;
      
    case 'open':
      handle = await repo.find(msg.docId);
      await handle.whenReady();
      const doc = handle.doc();
      send({ type: 'opened', docId: msg.docId, content: doc.content || '' });
      
      // Listen for remote changes
      handle.on('change', ({ doc }) => {
        send({ type: 'changed', content: doc.content || '' });
      });
      break;
      
    case 'edit':
      handle.change(d => {
        updateText(d, ['content'], msg.content);
      });
      break;
      
    // ... etc
  }
});

function send(obj) {
  console.log(JSON.stringify(obj));
}
```

**Checklist:**
- [ ] Initialize with `npm init` and add dependencies
- [ ] Implement connect/disconnect
- [ ] Implement open (repo.find) and create (repo.create)
- [ ] Implement edit (handle.change with updateText)
- [ ] Implement change listener (handle.on('change'))
- [ ] Add awareness WebSocket for cursors
- [ ] Test with: `echo '{"type":"connect",...}' | node index.js`

### Phase 2: Lua Plugin Core (3-4 hours)

Update `nvim/lua/collab-editor/init.lua`:

```lua
local M = {}

local job_id = nil
local current_doc_id = nil

function M.setup(opts)
  M.config = vim.tbl_extend('force', {
    sync_url = 'ws://localhost:1234',
    awareness_url = 'ws://localhost:1235',
    helper_path = vim.fn.stdpath('data') .. '/collab-editor/node-helper',
  }, opts or {})
end

function M.connect()
  -- Start Node helper process
  job_id = vim.fn.jobstart({'node', M.config.helper_path .. '/index.js'}, {
    on_stdout = function(_, data) M.handle_message(data) end,
    on_stderr = function(_, data) vim.notify(table.concat(data, '\n'), vim.log.levels.ERROR) end,
    on_exit = function() job_id = nil end,
  })
  
  -- Send connect message
  M.send({ type = 'connect', syncUrl = M.config.sync_url, awarenessUrl = M.config.awareness_url })
end

function M.send(msg)
  if job_id then
    vim.fn.chansend(job_id, vim.json.encode(msg) .. '\n')
  end
end

function M.handle_message(data)
  -- Parse and handle incoming messages
  for _, line in ipairs(data) do
    if line ~= '' then
      local msg = vim.json.decode(line)
      if msg.type == 'changed' then
        M.apply_remote_change(msg.content)
      elseif msg.type == 'opened' then
        current_doc_id = msg.docId
        M.apply_remote_change(msg.content)
      end
      -- ... etc
    end
  end
end

return M
```

**Checklist:**
- [ ] Process management (start/stop Node helper)
- [ ] Message sending/receiving
- [ ] Commands: `:CollabConnect`, `:CollabOpen <docId>`, `:CollabCreate`, `:CollabDisconnect`
- [ ] Store document ID for status display

### Phase 3: Buffer Sync (3-4 hours)

Update `nvim/lua/collab-editor/buffer.lua`:

```lua
local M = {}
local plugin = require('collab-editor')

local ignore_changes = false
local attached_buffers = {}

function M.attach(bufnr)
  bufnr = bufnr or vim.api.nvim_get_current_buf()
  
  vim.api.nvim_buf_attach(bufnr, false, {
    on_lines = function()
      if ignore_changes then return end
      
      -- Get full buffer content and send to helper
      local lines = vim.api.nvim_buf_get_lines(bufnr, 0, -1, false)
      local content = table.concat(lines, '\n')
      plugin.send({ type = 'edit', content = content })
    end,
  })
  
  attached_buffers[bufnr] = true
end

function M.apply_remote_change(content)
  local bufnr = vim.api.nvim_get_current_buf()
  if not attached_buffers[bufnr] then return end
  
  ignore_changes = true
  
  local lines = vim.split(content, '\n', { plain = true })
  vim.api.nvim_buf_set_lines(bufnr, 0, -1, false, lines)
  
  vim.schedule(function()
    ignore_changes = false
  end)
end

return M
```

**Checklist:**
- [ ] Buffer attachment with on_lines callback
- [ ] Send full content on local change
- [ ] Apply remote changes with feedback loop prevention
- [ ] Handle multiple buffers (future)

### Phase 4: Cursor Display (2-3 hours)

Update `nvim/lua/collab-editor/cursors.lua`:

Similar to previous version - use extmarks to show remote cursors.

**Checklist:**
- [ ] Create namespace for cursor extmarks
- [ ] Display cursor position with virtual text (username)
- [ ] Color coding per user
- [ ] Clear cursors on disconnect

### Phase 5: Testing (3-4 hours)

**Test 1: Helper standalone**
```bash
cd nvim/node-helper
echo '{"type":"connect","syncUrl":"ws://localhost:1234","awarenessUrl":"ws://localhost:1235"}' | node index.js
```

**Test 2: With running sync server**
```bash
# Terminal 1: Start servers
cd ~/lab/collab-editor
make ws        # Port 1234
# Terminal 2:
make awareness # Port 1235

# Terminal 3: Test helper
cd nvim/node-helper
node index.js
# Type JSON commands manually
```

**Test 3: Neovim to browser**
1. Open browser: `http://localhost:8080/?doc=SOME_DOC_ID`
2. Open Neovim: `:CollabConnect` then `:CollabOpen SOME_DOC_ID`
3. Type in browser → appears in Neovim
4. Type in Neovim → appears in browser

**Test 4: Two Neovim instances**
1. Neovim A: `:CollabConnect` then `:CollabCreate` (note the doc ID)
2. Neovim B: `:CollabConnect` then `:CollabOpen <docId>`
3. Verify sync works both ways

## What to Delete/Ignore

The previous (incorrect) implementation created these files that should be **replaced or deleted**:

- `nvim/go-helper/` - Replace with `nvim/node-helper/`
- `docs/server-protocol-example.go` - Delete (we don't need a custom server)

The Lua files (`init.lua`, `buffer.lua`, `cursors.lua`) need significant updates but the structure is okay.

## Commands (After Completion)

```vim
:CollabConnect              " Connect to sync server
:CollabDisconnect           " Disconnect
:CollabCreate               " Create new document, print doc ID
:CollabOpen <docId>         " Open existing document by ID
:CollabClose                " Stop syncing current buffer
:CollabInfo                 " Show connection status and doc ID
```

## Configuration

```lua
require('collab-editor').setup({
  sync_url = 'ws://localhost:1234',
  awareness_url = 'ws://localhost:1235',
  helper_path = '~/lab/collab-editor/nvim/node-helper',
})
```

## Time Estimate

| Phase | Hours |
|-------|-------|
| Node.js Helper | 4-6 |
| Lua Plugin Core | 3-4 |
| Buffer Sync | 3-4 |
| Cursor Display | 2-3 |
| Testing | 3-4 |
| **Total** | **15-21** |

## Dependencies

**Node.js helper (package.json):**
```json
{
  "name": "collab-editor-nvim-helper",
  "type": "module",
  "dependencies": {
    "@automerge/automerge": "^2.0.0",
    "@automerge/automerge-repo": "^1.0.0",
    "@automerge/automerge-repo-network-websocket": "^1.0.0",
    "@automerge/automerge-repo-storage-nodefs": "^1.0.0"
  }
}
```

**Neovim:**
- Neovim 0.9+ (for `vim.json`, `vim.system`)
- Node.js installed on system

## Key Differences From Previous (Wrong) Roadmap

| Previous (Wrong) | Corrected |
|------------------|-----------|
| Go helper | Node.js helper |
| Custom JSON-RPC protocol to server | Automerge binary sync protocol |
| Custom server endpoint needed | Uses existing sync server |
| Teamtype-style revision tracking | Automerge handles conflicts |
| Delta-based edits | Full content sync (simpler) |

## Next Steps

1. Delete `nvim/go-helper/` directory
2. Create `nvim/node-helper/` directory
3. Initialize npm project with dependencies
4. Implement helper `index.js`
5. Update Lua plugin to use Node helper
6. Test with browser client

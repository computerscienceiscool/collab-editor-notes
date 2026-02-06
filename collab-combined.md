# Collaborative Editing: End-to-End Setup Guide

A complete guide to running collaborative editing between the browser and Neovim using three components:

- **collab-awareness** -- Presence and cursor synchronization server
- **collab-web-editor** -- Browser-based collaborative editor
- **vimbeam** -- Neovim collaborative editing plugin

## Prerequisites

- Node.js 18+
- Neovim 0.8+ (for Neovim editing)
- npm

## Architecture

```
Browser (collab-web-editor)  â”€â”€â”¬â”€â”€  ws://localhost:1234  (Automerge sync - document CRDT)
                               â”‚
Neovim  (vimbeam)            â”€â”€â”˜

Browser (collab-web-editor)  â”€â”€â”¬â”€â”€  ws://localhost:1235  (Awareness - cursors/presence)
                               â”‚
Neovim  (vimbeam)            â”€â”€â”˜
```

All clients connect to the same two WebSocket servers. Document changes sync through port 1234 (Automerge CRDT protocol). Cursor positions, user names, and typing indicators sync through port 1235 (JSON protocol).

---

## Step 1: Install Dependencies

Run these once. Each command is run from the respective project directory.

```bash
# From ~/lab/collab-awareness/
cd ~/lab/collab-awareness
npm install
```
Installs the awareness server and client libraries (ws package).

```bash
# From ~/lab/collab-web-editor/
cd ~/lab/collab-web-editor
npm install
```
Installs the browser editor dependencies (Automerge, CodeMirror, Vite).

```bash
# From ~/lab/vimbeam/node-helper/
cd ~/lab/vimbeam/node-helper
npm install
```
Installs the Vimbeam Node.js helper dependencies (Automerge, ws).

---

## Step 2: Start the Servers

You need three terminals running. Start them in this order.

### Terminal 1: Automerge Sync Server (port 1234)

```bash
# From ~/lab/collab-web-editor/
cd ~/lab/collab-web-editor
npx @automerge/automerge-repo-sync-server
```
Starts the document sync server on port 1234. This handles CRDT document synchronization between all clients. All edits flow through this server.

### Terminal 2: Awareness Server (port 1235)

```bash
# From ~/lab/collab-awareness/
cd ~/lab/collab-awareness
npm start
```
Starts the awareness server on port 1235. This broadcasts cursor positions, user names, colors, and typing indicators between all connected clients.

### Terminal 3: Web Editor Dev Server (port 8080)

```bash
# From ~/lab/collab-web-editor/
cd ~/lab/collab-web-editor
npm run dev
```
Starts the Vite development server. The editor will be available at http://localhost:8080. If port 8080 is in use, Vite will try the next available port (8081, 8082, etc.) and display the URL.

---

## Step 3: Connect from the Browser

1. Open the URL shown by Vite (e.g., http://localhost:8080)
2. A new collaborative document is created automatically
3. The URL updates to include a document ID, e.g.: `http://localhost:8080/?doc=2DtwLMmuBznXxUmr7XpuebJnMwoe`
4. Share this URL (or just the doc ID) with other users
5. Other browser users can open the same URL to join

The document ID is the part after `?doc=`. You will need this to connect from Neovim.

---

## Step 4: Connect from Neovim

### One-time Setup: Install Vimbeam Plugin

Add to your Neovim config (e.g., `~/.config/nvim/init.lua`):

**Using lazy.nvim:**
```lua
{
  'computerscienceiscool/vimbeam',
  build = 'cd node-helper && npm install',
  config = function()
    require('vimbeam').setup({
      sync_url = 'ws://localhost:1234',
      awareness_url = 'ws://localhost:1235',
      user_name = 'Your Name',
      user_color = '#4ECDC4',
    })
  end,
}
```

**Using packer.nvim:**
```lua
use {
  'computerscienceiscool/vimbeam',
  run = 'cd node-helper && npm install',
  config = function()
    require('vimbeam').setup({
      sync_url = 'ws://localhost:1234',
      awareness_url = 'ws://localhost:1235',
      user_name = 'Your Name',
      user_color = '#4ECDC4',
    })
  end,
}
```

**Manual installation:**
```bash
git clone https://github.com/computerscienceiscool/vimbeam.git \
  ~/.local/share/nvim/site/pack/plugins/start/vimbeam
cd ~/.local/share/nvim/site/pack/plugins/start/vimbeam/node-helper
npm install
```

Then add to `init.lua`:
```lua
require('vimbeam').setup({
  sync_url = 'ws://localhost:1234',
  awareness_url = 'ws://localhost:1235',
  user_name = 'Your Name',
  user_color = '#4ECDC4',
})
```

### Connect and Open a Document

In Neovim, run these commands:

```vim
:BeamConnect ws://localhost:1234 ws://localhost:1235
```
Connects to both the Automerge sync server and the awareness server.

```vim
:BeamOpen 2DtwLMmuBznXxUmr7XpuebJnMwoe
```
Opens the document by ID (use the ID from the browser URL). The document content loads into the current buffer. Edits sync in real time.

**Or use the shortcut to connect and open in one step:**
```vim
:BeamQuick 2DtwLMmuBznXxUmr7XpuebJnMwoe
```

### Create a New Document from Neovim

```vim
:BeamConnect ws://localhost:1234 ws://localhost:1235
:BeamCreate
```
Creates a new document. The document ID is printed -- share it with browser users so they can open `http://localhost:8080/?doc=<id>`.

---

## Step 5: Verify It Works

Once both browser and Neovim are connected to the same document:

- **Edits sync both ways** -- Type in either client and see changes appear in the other
- **Cursors are visible** -- Browser shows Neovim users in the user list and their cursor positions. Neovim shows browser user cursors.
- **User names appear** -- Each client displays the names of connected users
- **Typing indicators** -- The browser shows when a remote user is actively typing

---

## Useful Neovim Commands

| Command | Description |
|---------|-------------|
| `:BeamConnect <sync_url> <awareness_url>` | Connect to servers |
| `:BeamCreate` | Create a new document |
| `:BeamOpen <doc_id>` | Open an existing document |
| `:BeamQuick <doc_id>` | Connect and open in one step |
| `:BeamDisconnect` | Disconnect from servers |
| `:BeamInfo` | Show connection status |
| `:BeamUserName <name>` | Change your display name |
| `:BeamUserColor <hex>` | Change your cursor color |

---

## Port Reference

| Port | Service | Protocol | Purpose |
|------|---------|----------|---------|
| 1234 | Automerge sync server | WebSocket (CBOR binary) | Document CRDT synchronization |
| 1235 | Awareness server | WebSocket (JSON text) | Cursor positions, user presence, typing indicators |
| 8080 | Vite dev server | HTTP | Serves the browser editor |

---

## Troubleshooting

**Browser shows "Could not load document"**
- Check that the Automerge sync server is running on port 1234

**No remote cursors or users visible**
- Check that the awareness server is running on port 1235
- Verify both clients are using the same document ID

**Neovim users don't appear in browser (but browser users appear in Neovim)**
- This was a known bug where the browser awareness client used a document ID without the `automerge:` prefix. Fixed in collab-web-editor.

**WASM errors in browser console**
- Go WASM modules (Grokker, diff) are optional. If the `.wasm` files aren't built, those features are disabled silently. Core editing and collaboration work without them.

**Port already in use**
- Vite will automatically try the next port (8081, 8082, etc.)
- For the sync/awareness servers, stop any existing process on that port first

---

## Project Repositories

- [collab-web-editor](https://github.com/computerscienceiscool/collab-web-editor) -- Browser collaborative editor
- [collab-awareness](https://github.com/computerscienceiscool/collab-awareness) -- Presence/cursor sync server and clients
- [vimbeam](https://github.com/computerscienceiscool/vimbeam) -- Neovim collaborative editing plugin
- [collab-editor](https://github.com/computerscienceiscool/collab-editor) -- Original monorepo (collab-web-editor was extracted from here)

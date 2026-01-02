# YJS → Automerge Migration Checklist

## Priority Order (Do These In Sequence)

---

### 🔴 PHASE 1: PREPARATION (1-2 days)

- [ ] **1.1** Search codebase for all YJS imports
  ```bash
  grep -r "from 'yjs'" .
  grep -r "import.*yjs" .
  ```
- [ ] **1.2** List all YJS data types in use (Y.Doc, Y.Map, Y.Array, Y.Text)
- [ ] **1.3** List all YJS providers (y-websocket, y-indexeddb, etc.)
- [ ] **1.4** Document current file structure

**Files to review:**
- [ ] `package.json`
- [ ] `src/app.js` or main entry
- [ ] All files in `src/` with YJS imports

---

### 🔴 PHASE 2: SETUP (0.5 days)

- [ ] **2.1** Remove YJS dependencies
  ```bash
  npm uninstall yjs y-websocket y-webrtc y-indexeddb y-protocols
  ```
- [ ] **2.2** Install Automerge
  ```bash
  npm install @automerge/automerge
  npm install @automerge/automerge-repo
  npm install @automerge/automerge-repo-network-websocket
  npm install @automerge/automerge-repo-storage-indexeddb
  ```
- [ ] **2.3** Verify build still runs (will have errors - that's expected)

---

### 🔴 PHASE 3: CORE MIGRATION (3-5 days)

**Do these in order:**

- [ ] **3.1** Create Automerge document initialization
  - Replace `new Y.Doc()` → `Automerge.init()`
  - File: `src/lib/document-manager.js` or equivalent

- [ ] **3.2** Migrate data structures
  - [ ] Maps: `ymap.set(k,v)` → `Automerge.change(doc, d => d.map[k] = v)`
  - [ ] Arrays: `yarray.push([x])` → `Automerge.change(doc, d => d.arr.push(x))`
  - [ ] Text: `ytext.insert()` → `Automerge.Text`

- [ ] **3.3** Update all mutation/change handling
  - Wrap ALL mutations in `Automerge.change()`
  - Remember: Automerge returns NEW doc (immutable)

- [ ] **3.4** Replace observer/event system
  - `ymap.observe()` → `Automerge.diff(oldDoc, newDoc)`

- [ ] **3.5** Implement undo/redo
  - `UndoManager` → `Automerge.undo()` / `Automerge.redo()`

---

### 🔴 PHASE 4: NETWORK & SYNC (2-3 days)

- [ ] **4.1** Replace WebSocket provider
  - `y-websocket` → `@automerge/automerge-repo-network-websocket`

- [ ] **4.2** Update sync protocol
  - YJS state vectors → Automerge sync messages
  ```javascript
  import { generateSyncMessage, receiveSyncMessage } from '@automerge/automerge'
  ```

- [ ] **4.3** Migrate persistence layer
  - `y-indexeddb` → `@automerge/automerge-repo-storage-indexeddb`

---

### 🟠 PHASE 5: INTEGRATION (2-4 days)

- [ ] **5.1** Update state management (Redux/Zustand/etc.)

- [ ] **5.2** Update React/UI components
  - [ ] Replace YJS hooks with Automerge hooks
  - [ ] Update all components using document data

- [ ] **5.3** Update collaborative features
  - [ ] Cursors/presence (needs custom solution)
  - [ ] User awareness

---

### 🟠 PHASE 6: TESTING (2-3 days)

- [ ] **6.1** Create migration tests
  - [ ] Document creation
  - [ ] CRUD operations
  - [ ] Concurrent editing
  - [ ] Sync between clients

- [ ] **6.2** Create data migration script (if converting existing docs)
  ```javascript
  // Convert YJS doc to Automerge
  const data = yjsDoc.toJSON()
  let amDoc = Automerge.change(Automerge.init(), d => Object.assign(d, data))
  ```

- [ ] **6.3** Run full test suite
- [ ] **6.4** Manual testing of all features

---

### 🟡 PHASE 7: BACKEND (1-3 days)

- [ ] **7.1** Update server sync logic
- [ ] **7.2** Update database schema (Automerge = binary storage)
- [ ] **7.3** Test server-client sync

---

### 🟡 PHASE 8: DEPLOYMENT (1-2 days)

- [ ] **8.1** Create rollback plan
- [ ] **8.2** Backup all existing data
- [ ] **8.3** Deploy to staging
- [ ] **8.4** Final testing
- [ ] **8.5** Deploy to production
- [ ] **8.6** Monitor for issues

---

## Quick Reference: API Mapping

| YJS | Automerge |
|-----|-----------|
| `new Y.Doc()` | `Automerge.init()` |
| `doc.getMap('x')` | `doc.x` (direct access) |
| `map.set(k, v)` | `Automerge.change(doc, d => d.map[k] = v)` |
| `array.push([item])` | `Automerge.change(doc, d => d.arr.push(item))` |
| `text.insert(pos, str)` | `Automerge.change(doc, d => d.text.insertAt(pos, str))` |
| `type.observe(fn)` | `Automerge.diff(oldDoc, newDoc)` |
| `Y.encodeStateAsUpdate()` | `Automerge.save(doc)` |
| `Y.applyUpdate()` | `Automerge.load(binary)` |

---

## Time Summary

| Phase | Days | Hours | Status |
|-------|------|-------|--------|
| 1. Preparation | 1-2 | 8-16 | ⬜ |
| 2. Setup | 0.5 | 4 | ⬜ |
| 3. Core Migration | 3-5 | 24-40 | ⬜ |
| 4. Network/Sync | 2-3 | 16-24 | ⬜ |
| 5. Integration | 2-4 | 16-32 | ⬜ |
| 6. Testing | 2-3 | 16-24 | ⬜ |
| 7. Backend | 1-3 | 8-24 | ⬜ |
| 8. Deployment | 1-2 | 8-16 | ⬜ |
| **TOTAL** | **12-22** | **100-180** | |

---

## Common Pitfalls (Don't Forget!)

- [ ] ⚠️ **Immutability**: Always reassign `doc = Automerge.change(doc, ...)`
- [ ] ⚠️ **No built-in observers**: Must track changes manually with `diff()`
- [ ] ⚠️ **No built-in awareness**: Need custom presence solution
- [ ] ⚠️ **Documents grow forever**: Plan for large doc handling

---

## Success Checklist

- [ ] All YJS dependencies removed from `package.json`
- [ ] All tests passing
- [ ] Two clients can sync in real-time
- [ ] Data persists after refresh
- [ ] Undo/redo works
- [ ] No data loss from migration
- [ ] Performance acceptable

---

**Started:** _______________  
**Completed:** _______________

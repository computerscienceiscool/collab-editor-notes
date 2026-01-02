# YJS to Automerge Migration Roadmap

## Overview
This document provides a step-by-step roadmap for migrating from YJS (Yjs) to Automerge CRDT library. The migration requires careful planning as both libraries have different APIs, data structures, and synchronization mechanisms.

---

## Phase 1: Preparation & Analysis

### 1.1 Audit Current YJS Usage
**Priority: Critical**

Identify all files and patterns using YJS:

- [ ] Search codebase for YJS imports:
  ```bash
  grep -r "from 'yjs'" .
  grep -r "require('yjs')" .
  grep -r "import.*yjs" .
  ```

- [ ] Document YJS data types in use:
  - Y.Doc
  - Y.Map
  - Y.Array
  - Y.Text
  - Y.XmlFragment/Y.XmlElement
  - Custom types

- [ ] List all YJS providers/bindings:
  - y-websocket
  - y-webrtc
  - y-indexeddb
  - y-protocols
  - Other transport layers

**Key Files to Review:**
- `package.json` - Dependencies
- `src/store/` or `src/state/` - State management
- `src/sync/` or `src/network/` - Synchronization logic
- `src/models/` - Data models
- `src/hooks/` or `src/components/` - React/UI integration

---

## Phase 2: Environment Setup

### 2.1 Install Automerge Dependencies
**Priority: Critical**

```bash
# Remove YJS dependencies
npm uninstall yjs y-websocket y-webrtc y-indexeddb y-protocols

# Install Automerge
npm install @automerge/automerge
npm install @automerge/automerge-repo
npm install @automerge/automerge-repo-network-websocket
npm install @automerge/automerge-repo-storage-indexeddb
```

**Files to Update:**
- `package.json`
- `package-lock.json` / `yarn.lock` / `pnpm-lock.yaml`

### 2.2 Update TypeScript Configuration (if applicable)
**Priority: Medium**

**Files to Update:**
- `tsconfig.json` - Ensure compatibility with Automerge types

---

## Phase 3: Core Migration (Execute in Order)

### 3.1 Create Automerge Document Initialization
**Priority: Critical | Order: 1**

**Pattern Change:**
```javascript
// OLD - YJS
import * as Y from 'yjs'
const ydoc = new Y.Doc()

// NEW - Automerge
import * as Automerge from '@automerge/automerge'
let doc = Automerge.init()
```

**Files to Create/Update:**
- `src/lib/automerge-setup.js` or `src/lib/document-manager.js`
- `src/store/documentStore.js` (or equivalent state management)

**Key Changes:**
- Replace `new Y.Doc()` with `Automerge.init()`
- Update document reference patterns
- Implement document change handling

### 3.2 Migrate Data Structure Definitions
**Priority: Critical | Order: 2**

**Map Migration:**
```javascript
// OLD - YJS
const ymap = ydoc.getMap('myMap')
ymap.set('key', 'value')
const value = ymap.get('key')

// NEW - Automerge
doc = Automerge.change(doc, d => {
  d.myMap = { key: 'value' }
})
const value = doc.myMap.key
```

**Array Migration:**
```javascript
// OLD - YJS
const yarray = ydoc.getArray('myArray')
yarray.push(['item'])
const item = yarray.get(0)

// NEW - Automerge
doc = Automerge.change(doc, d => {
  d.myArray = ['item']
})
const item = doc.myArray[0]
```

**Text Migration:**
```javascript
// OLD - YJS
const ytext = ydoc.getText('myText')
ytext.insert(0, 'Hello')
const content = ytext.toString()

// NEW - Automerge
doc = Automerge.change(doc, d => {
  d.myText = new Automerge.Text('Hello')
})
const content = doc.myText.toString()
```

**Files to Update:**
- `src/models/Document.js`
- `src/models/Board.js`
- `src/models/Editor.js`
- `src/types/schema.ts` (TypeScript definitions)
- All model/schema definition files

### 3.3 Update Mutation/Change Handling
**Priority: Critical | Order: 3**

**Pattern Change:**
```javascript
// OLD - YJS (implicit transactions)
ymap.set('user', 'Alice')
yarray.push(['item'])

// NEW - Automerge (explicit change blocks)
doc = Automerge.change(doc, 'Update user and add item', d => {
  d.user = 'Alice'
  d.items.push('item')
})
```

**Files to Update:**
- `src/actions/` - All action creators
- `src/store/mutations.js` or `src/store/actions.js`
- `src/handlers/eventHandlers.js`
- Any files with state mutation logic

**Key Considerations:**
- Wrap all mutations in `Automerge.change()`
- Add descriptive change messages
- Handle immutability (Automerge returns new doc)

### 3.4 Replace Observer/Event System
**Priority: Critical | Order: 4**

**Pattern Change:**
```javascript
// OLD - YJS
ymap.observe(event => {
  event.changes.keys.forEach((change, key) => {
    // Handle change
  })
})

// NEW - Automerge
let prevDoc = doc
doc = Automerge.change(doc, d => { /* changes */ })
const patches = Automerge.diff(prevDoc, doc)
patches.forEach(patch => {
  // Handle patch
})
```

**Files to Update:**
- `src/observers/` or `src/listeners/`
- `src/hooks/useDocument.js` (React hooks)
- `src/store/subscribers.js`
- Component event handlers

**Alternative Approach:**
Use `Automerge.getHistory()` or track changes manually

### 3.5 Implement Undo/Redo
**Priority: High | Order: 5**

**Pattern Change:**
```javascript
// OLD - YJS
import { UndoManager } from 'yjs'
const undoManager = new UndoManager(ymap)
undoManager.undo()
undoManager.redo()

// NEW - Automerge
const history = Automerge.getHistory(doc)
doc = Automerge.undo(doc)
doc = Automerge.redo(doc)
```

**Files to Update:**
- `src/features/undo/undoManager.js`
- `src/store/history.js`
- Keyboard shortcut handlers

---

## Phase 4: Network & Synchronization

### 4.1 Replace WebSocket Provider
**Priority: Critical | Order: 6**

**Pattern Change:**
```javascript
// OLD - YJS
import { WebsocketProvider } from 'y-websocket'
const provider = new WebsocketProvider('ws://localhost:1234', 'room', ydoc)

// NEW - Automerge Repo
import { Repo } from '@automerge/automerge-repo'
import { BrowserWebSocketClientAdapter } from '@automerge/automerge-repo-network-websocket'

const repo = new Repo({
  network: [new BrowserWebSocketClientAdapter('ws://localhost:1234')],
  storage: new IndexedDBStorageAdapter()
})

const handle = repo.find('document-id')
```

**Files to Update:**
- `src/sync/websocketProvider.js`
- `src/network/connection.js`
- `src/config/sync.js`

### 4.2 Update Sync Protocol
**Priority: Critical | Order: 7**

**Key Differences:**
- YJS uses state vectors and update messages
- Automerge uses document sync protocol

**Implementation:**
```javascript
// Automerge sync
import { generateSyncMessage, receiveSyncMessage } from '@automerge/automerge'

// Generate sync message
const syncMessage = generateSyncMessage(doc, syncState)

// Process received message
[doc, syncState] = receiveSyncMessage(doc, syncState, message)
```

**Files to Update:**
- `src/sync/syncEngine.js`
- `src/network/messageHandlers.js`
- Backend sync server code

### 4.3 Migrate Persistence Layer
**Priority: High | Order: 8**

**Pattern Change:**
```javascript
// OLD - YJS
import { IndexeddbPersistence } from 'y-indexeddb'
const persistence = new IndexeddbPersistence('document-id', ydoc)

// NEW - Automerge Repo
import { IndexedDBStorageAdapter } from '@automerge/automerge-repo-storage-indexeddb'

const repo = new Repo({
  storage: new IndexedDBStorageAdapter()
})
```

**Files to Update:**
- `src/persistence/indexedDB.js`
- `src/storage/documentStorage.js`
- `src/db/` directory

---

## Phase 5: Application Integration

### 5.1 Update State Management Integration
**Priority: Critical | Order: 9**

For Redux/Zustand/other state managers:

```javascript
// Example: Redux integration
const automergeMiddleware = store => next => action => {
  const result = next(action)
  
  if (action.type.startsWith('DOCUMENT_')) {
    const newDoc = Automerge.change(currentDoc, action.message, d => {
      // Apply changes based on action
    })
    // Sync with store
  }
  
  return result
}
```

**Files to Update:**
- `src/store/index.js`
- `src/store/middleware/`
- `src/reducers/documentReducer.js`

### 5.2 Update React/UI Components
**Priority: High | Order: 10**

**Pattern Change:**
```javascript
// OLD - YJS with React
import { useY } from 'your-yjs-hooks'

function Component() {
  const ymap = useY('myMap')
  const value = ymap.get('key')
  
  return <div>{value}</div>
}

// NEW - Automerge with React
import { useDocument } from '@automerge/automerge-repo-react-hooks'

function Component({ documentId }) {
  const [doc] = useDocument(documentId)
  
  return <div>{doc?.myMap?.key}</div>
}
```

**Files to Update:**
- `src/components/` - All components using YJS
- `src/hooks/useYDoc.js` → `src/hooks/useAutomergeDoc.js`
- `src/context/DocumentContext.js`

### 5.3 Update Collaborative Features
**Priority: High | Order: 11**

**Awareness/Presence:**
```javascript
// OLD - YJS Awareness
import { Awareness } from 'y-protocols/awareness'
const awareness = new Awareness(ydoc)
awareness.setLocalState({ user: { name: 'Alice' } })

// NEW - Automerge ephemeral data
// Implement custom presence using separate mechanism
// or use Automerge Repo's built-in presence
```

**Files to Update:**
- `src/features/cursors/CursorManager.js`
- `src/features/presence/PresenceIndicator.js`
- `src/collaboration/awareness.js`

---

## Phase 6: Testing & Validation

### 6.1 Create Migration Tests
**Priority: Critical | Order: 12**

**Files to Create:**
- `tests/migration/yjs-to-automerge.test.js`
- `tests/integration/sync.test.js`
- `tests/unit/document-operations.test.js`

**Test Coverage:**
- Document creation and initialization
- CRUD operations on all data types
- Concurrent editing scenarios
- Network sync and conflict resolution
- Persistence and recovery
- Undo/redo functionality

### 6.2 Data Migration Script
**Priority: Critical | Order: 13**

Create script to convert existing YJS documents to Automerge:

**File to Create:**
- `scripts/migrate-documents.js`

```javascript
// Pseudocode
async function migrateDocument(yjsDoc) {
  let automergeDoc = Automerge.init()
  
  // Extract YJS data
  const data = yjsDoc.toJSON()
  
  // Convert to Automerge
  automergeDoc = Automerge.change(automergeDoc, d => {
    Object.assign(d, data)
  })
  
  return automergeDoc
}
```

---

## Phase 7: Backend Updates

### 7.1 Update Server Sync Logic
**Priority: Critical | Order: 14**

If you have a backend server handling sync:

**Files to Update:**
- `server/sync-server.js`
- `server/websocket-handler.js`
- `server/storage/documents.js`

**Key Changes:**
- Replace YJS sync protocol with Automerge sync
- Update message handling
- Modify storage format

### 7.2 Database Schema Updates
**Priority: High | Order: 15**

**Files to Update:**
- Database migration files
- `server/models/Document.js`
- Schema definitions

**Considerations:**
- Automerge documents are stored as binary data
- Update storage format from YJS updates to Automerge changes

---

## Phase 8: Deployment & Rollout

### 8.1 Gradual Migration Strategy
**Priority: Critical | Order: 16**

**Option A: Feature Flag Approach**
- Implement feature flag to toggle between YJS and Automerge
- Allow gradual user migration
- Monitor performance and issues

**Files to Create/Update:**
- `src/config/features.js`
- Environment configuration files

**Option B: Big Bang Migration**
- Schedule maintenance window
- Migrate all documents at once
- Higher risk, faster completion

### 8.2 Monitoring & Rollback Plan
**Priority: Critical | Order: 17**

**Files to Create:**
- `docs/rollback-procedure.md`
- Monitoring dashboards configuration

**Prepare:**
- Database backups before migration
- YJS fallback code (keep temporarily)
- Performance benchmarks
- Error tracking setup

---

## Critical Files Checklist

### Must Update:
- [ ] `package.json` - Dependencies
- [ ] `src/lib/document-init.js` - Document initialization
- [ ] `src/models/*` - All data models
- [ ] `src/store/*` - State management
- [ ] `src/sync/*` - Synchronization logic
- [ ] `src/network/*` - Network layer
- [ ] `src/persistence/*` - Storage layer
- [ ] `src/components/*` - UI components
- [ ] `src/hooks/*` - React hooks
- [ ] `server/sync-server.js` - Backend sync
- [ ] `tests/*` - All tests

### Likely to Update:
- [ ] `src/actions/*` - Action creators
- [ ] `src/reducers/*` - Reducers
- [ ] `src/middleware/*` - Middleware
- [ ] `src/context/*` - React context
- [ ] `src/features/undo/*` - Undo/redo
- [ ] `src/features/cursors/*` - Collaborative cursors
- [ ] `src/features/presence/*` - User presence
- [ ] `src/utils/document-utils.js` - Utility functions
- [ ] Configuration files (`.env`, `config/*`)

---

## API Mapping Reference

### Document Operations
| YJS | Automerge |
|-----|-----------|
| `new Y.Doc()` | `Automerge.init()` |
| `doc.getMap(name)` | Access via `doc.mapName` |
| `doc.getArray(name)` | Access via `doc.arrayName` |
| `doc.getText(name)` | `new Automerge.Text()` |
| `doc.toJSON()` | Direct object access |

### Mutations
| YJS | Automerge |
|-----|-----------|
| `map.set(k, v)` | `Automerge.change(doc, d => d.map[k] = v)` |
| `array.push([item])` | `Automerge.change(doc, d => d.array.push(item))` |
| `text.insert(pos, str)` | `Automerge.change(doc, d => d.text.insertAt(pos, str))` |
| `map.delete(k)` | `Automerge.change(doc, d => delete d.map[k])` |

### Observation
| YJS | Automerge |
|-----|-----------|
| `type.observe(fn)` | `Automerge.diff(oldDoc, newDoc)` |
| `type.observeDeep(fn)` | Track changes manually |

### Synchronization
| YJS | Automerge |
|-----|-----------|
| `Y.encodeStateAsUpdate()` | `Automerge.save(doc)` |
| `Y.applyUpdate()` | `Automerge.load(binary)` |
| `Y.encodeStateVector()` | `Automerge.getHeads(doc)` |

---

## Common Pitfalls & Solutions

### 1. Immutability
**Issue:** YJS allows direct mutation; Automerge returns new documents
**Solution:** Always reassign `doc = Automerge.change(doc, ...)`

### 2. Observer Pattern
**Issue:** YJS has built-in observers; Automerge requires manual tracking
**Solution:** Implement change tracking with `Automerge.diff()`

### 3. Text Editing
**Issue:** Different APIs for text manipulation
**Solution:** Use `Automerge.Text` class and its methods

### 4. Awareness/Presence
**Issue:** No built-in awareness in core Automerge
**Solution:** Use Automerge Repo's presence or implement custom solution

### 5. Binary Size
**Issue:** Automerge documents can grow large
**Solution:** Implement periodic compaction with `Automerge.merge()`

---

## Performance Considerations

1. **Document Size:** Automerge documents grow with edit history
   - Implement pruning strategy
   - Use `Automerge.save()` and `Automerge.load()` for snapshots

2. **Sync Performance:** Different sync protocols
   - Test with realistic user loads
   - Monitor network traffic

3. **Memory Usage:** Automerge may use more memory
   - Profile application
   - Optimize document structure

---

## Timeline Estimate

| Phase | Estimated Time | Priority |
|-------|---------------|----------|
| Phase 1: Preparation | 1-2 days | Critical |
| Phase 2: Setup | 0.5 days | Critical |
| Phase 3: Core Migration | 3-5 days | Critical |
| Phase 4: Network/Sync | 2-3 days | Critical |
| Phase 5: Integration | 2-4 days | Critical |
| Phase 6: Testing | 2-3 days | Critical |
| Phase 7: Backend | 1-3 days | High |
| Phase 8: Deployment | 1-2 days | Critical |
| **Total** | **12-22 days** | |

*Note: Timeline assumes one developer working full-time. Adjust based on team size and codebase complexity.*

---

## Resources

- [Automerge Documentation](https://automerge.org/docs/)
- [Automerge Repo](https://github.com/automerge/automerge-repo)
- [Migration Examples](https://github.com/automerge/automerge/tree/main/examples)
- YJS Documentation (for reference)

---

## Success Criteria

- [ ] All YJS dependencies removed
- [ ] All tests passing
- [ ] Successful sync between clients
- [ ] Data persistence working
- [ ] Performance benchmarks met
- [ ] Zero data loss in migration
- [ ] Rollback plan tested
- [ ] Documentation updated

---

**Last Updated:** [Current Date]
**Migration Status:** Planning Phase

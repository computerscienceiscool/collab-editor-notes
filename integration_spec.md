# Collaborative Editor + PromiseGrid Integration Specification

**Date:** Sept 6, 2025\
**Owner:** JJ\
**Scope:** Collaborative editor integration with PromiseGrid data model
and PromiseBase persistence.

------------------------------------------------------------------------

## 1. Current Editor Components

-   **Editor core:** CodeMirror + Yjs for real-time collaborative
    editing.\
-   **Transport:** Yjs WebSocket server on port `1234`.\
-   **Support services:** Rust backend on `3000`, Node server on
    `8080`.\
-   **Persistence:** IndexedDB (local only).\
-   **Export:** "Download as Text" option.

------------------------------------------------------------------------

## 2. Feature Backlog (from transcript + whiteboard)

### Must-Have

-   Reliable line-number toggle\
-   Document export with stable naming\
-   Room/document identifiers (UUIDs) to prevent collisions\
-   IndexedDB flat namespace persistence\
-   Basic Playwright test coverage for core UI behaviors\
-   Clear environment/ports documentation

### Should-Have

-   Transient status notifications (toasts)\
-   Issue hygiene workflow nudges\
-   Top-of-doc date field\
-   HOWTO video for AIDDA

### Nice-to-Have

-   Visual keybinding hints\
-   GitHub integration helpers\
-   Draggable panels for markdown + activity log

### Open Philosophical Questions

-   Folder/file vs. space/room/resource metaphor\
-   Root ownership and centralization concerns\
-   Handling versions (e.g., VMS-style `filename;version`)\
-   Use of Plan 9--style per-process directories\
-   Libraries/layers for consistent metaphors

------------------------------------------------------------------------

## 3. PromiseBase / PromiseGrid Context

-   **PromiseBase** provides:
    -   Content-addressed storage (hash-based).
    -   Immutable snapshots (Merkle trees of blocks).
    -   Mutable refs (streams → tree addresses).
    -   Deduplication, integrity verification, and streaming writes.\
-   **PromiseGrid goals**:
    -   Documents, chats, and compute resources as **rooms/resources**.
    -   Avoid hard-coded folder/file metaphor unless necessary.
    -   Support persistence, naming, and access consistently across
        apps.
    -   Encourage developers to use standard **libraries/layers**.

------------------------------------------------------------------------

## 4. Proposed Data Model

### 4.1 Document Identity

-   Each document has a **UUID** (machine ID).\
-   Each document has a **label** (human-readable).\
-   Optionally, multiple labels can point to the same UUID.

### 4.2 IndexedDB Schema

-   **`docs` store**

    ``` json
    {
      "id": "uuid-1234",
      "label": "Project Plan",
      "headVersion": 5
    }
    ```

-   **`versions` store**

    ``` json
    {
      "docId": "uuid-1234",
      "version": 5,
      "timestamp": 1694006580,
      "bytes": "<snapshot>"
    }
    ```

### 4.3 File Export

-   Filenames: `<label>_<YYYYMMDD-HHMM>.txt`\
-   Label stored in `docs.label` (custom field).\
-   Timestamp derived from `versions.timestamp`.

------------------------------------------------------------------------

## 5. Integration with PromiseBase

### 5.1 Storage Adapter

Define a `PromiseBaseStorageAdapter`:

``` ts
interface Storage {
  loadLatest(id: UUID): Uint8Array | string;
  saveCheckpoint(id: UUID, bytes: Uint8Array | string): VersionId;
  resolveByLabel(label: string): UUID | null;
}
```

### 5.2 Implementation Steps

-   **Checkpointing:** Periodically serialize editor state →
    `pb putstream`.\
-   **Refs:** Update `stream/<uuid>` to point at latest snapshot.\
-   **Loading:** Resolve `stream/<uuid>` → `pb gettree` to rehydrate
    state.\
-   **Indexing:** Maintain a label→uuid manifest stored in PromiseBase.

### 5.3 Development Mode

-   Use **IndexedDB** adapter today.\
-   Add PromiseBase adapter behind same interface.\
-   Switch adapter via environment config.

------------------------------------------------------------------------

## 6. Todo List for JJ

### Editor Features

-   [ ] Fix line-number toggle (CodeMirror gutters).\
-   [ ] Add transient toasts for toggle and save actions.\
-   [ ] Implement UUID-based doc IDs.\
-   [ ] Save human-readable `label` in IndexedDB.\
-   [ ] Generate filenames `<label>_<timestamp>.txt`.\
-   [ ] Add top-of-doc date field.\
-   [ ] Implement draggable panels for markdown + activity log.

### Persistence

-   [ ] Create IndexedDB schema (`docs`, `versions`).\
-   [ ] Implement label→uuid index.\
-   [ ] Build `Storage` interface (IndexedDB adapter first).\
-   [ ] Add checkpointing every N ops/seconds.

### PromiseBase

-   [ ] Write `PromiseBaseStorageAdapter`.\
-   [ ] Implement `putstream` + `stream set` calls via CLI/IPC.\
-   [ ] Implement `stream resolve` + `gettree` for loading.\
-   [ ] Test cross-check: doc saved in editor is retrievable via CLI.

### Testing

-   [ ] Add Playwright tests for line-number toggle.\
-   [ ] Add test: save doc → reload editor → load from storage.\
-   [ ] Add test: export doc → verify filename matches pattern.

### Docs/Process

-   [ ] Update **features_combined.md** with PromiseBase integration
    details.\
-   [ ] Record HOWTO video (Steve or JJ).\
-   [ ] Document dev environment (ports, services).

------------------------------------------------------------------------

## 7. Todo List for Boss

-   [ ] Decide on guiding metaphor (folders vs. rooms/resources).\
-   [ ] Confirm versioning model (simple timestamp vs. VMS-like
    `filename;version`).\
-   [ ] Approve use of IndexedDB + PromiseBase layering.\
-   [ ] Provide clarity on long-term persistence/naming expectations.\
-   [ ] Review and sign off on editor UI priorities.\
-   [ ] Make or delegate HOWTO video for AIDDA usability.

------------------------------------------------------------------------

## 8. Open Questions

-   Should PromiseGrid apps enforce one metaphor (room/resource) or
    allow per-app freedom?\
-   How much history/versioning should be exposed in the editor UI (all
    hashes vs. "last 5 versions")?\
-   Should manifests (label→uuid index) be centralized or per-user?\
-   Will PromiseBase networking be ready for multi-user persistence, or
    do we start with local-only?

# Collaborative Editor Scenarios: Balanced Promises Breakdown

This document reframes each of the **15 collaborative editor user scenarios** using the concepts from Mark Burgess' *Money, Ownership, and Agency* (via Promise Theory). Each scenario starts with a **summary of main promises** (as in the first mapping) and then breaks them down into **smaller, balanced sub-promises** between agents, ensuring both sides make explicit commitments.

---

## 1. Real-Time Multi-User Editing
**Main Promises:**
- **User 1** promises to broadcast edits (X).
- **Users 2–4** promise to receive and integrate those edits (Y).

**Detailed Balanced Promises:**
1. **User 1 → System**:
   - X1: "I will send my local edits to the system."
   - Y1: "I will accept confirmation from the system."
2. **System → Peers**:
   - X2: "I will forward edits to all connected peers."
   - Y2: "I will preserve edit order and integrity."
3. **Peer → System**:
   - X3: "I will acknowledge receipt of edits."
   - Y3: "I will not reject well-formed edits."
4. **Peer → Peer (via system)**:
   - X4: "I will incorporate changes into my local state."
   - Y4: "I will accept updated document state."

---

## 2. Creating and Sharing a New Document
**Main Promises:**
- **Project Manager** promises to create a room & share URL (X).
- **Team Members** promise to join and use that room (Y).

**Detailed Balanced Promises:**
1. **Manager → System**:
   - X1: "I will request creation of a new room."
   - Y1: "I will accept the room ID and link."
2. **System → Manager**:
   - X2: "I will create the room with a UUID or given name."
   - Y2: "I will confirm ownership assignment."
3. **Manager → Team**:
   - X3: "I will share the room URL."
   - Y3: "I will provide necessary context for use."
4. **Team → System**:
   - X4: "I will join using the provided URL."
   - Y4: "I will respect room-specific ownership rules."

---

## 3. Formatting and PromiseGrid Logging
**Main Promises:**
- **User** promises to apply formatting (X).
- **System** promises to log a PromiseGrid message (Y).

**Detailed Balanced Promises:**
1. **User → System**:
   - X1: "I will apply a formatting command (e.g., bold)."
   - Y1: "I will provide the target text selection."
2. **System → WASM**:
   - X2: "I will pass the request to formatting engine."
   - Y2: "I will accept the transformed text."
3. **WASM → System**:
   - X3: "I will apply formatting and produce a PromiseGrid CBOR message."
   - Y3: "I will signal any errors in formatting."
4. **System → User**:
   - X4: "I will update the document and show changes."
   - Y4: "I will store the PromiseGrid log entry."

---

## 4. Offline Editing and Sync
**Main Promises:**
- **User** promises to edit offline (X).
- **System** promises to store locally and later sync (Y).

**Detailed Balanced Promises:**
1. **User → System**:
   - X1: "I will make edits without network access."
   - Y1: "I will trust system to store them locally."
2. **System → Local Storage**:
   - X2: "I will save all edits in IndexedDB."
   - Y2: "I will allow retrieval for sync."
3. **System → User**:
   - X3: "I will show confirmation of offline save."
   - Y3: "I will accept edits without online validation."
4. **System → Network**:
   - X4: "On reconnect, I will send queued edits."
   - Y4: "I will accept peer confirmation."
5. **Peers → System**:
   - X5: "I will merge received edits."
   - Y5: "I will maintain integrity of shared state."

---

## 5. Format Document Cleanup
**Main Promises:**
- **Writer** promises to request cleanup (X).
- **System** promises to clean up document (Y).

**Detailed Balanced Promises:**
1. **Writer → System**:
   - X1: "I will request full document cleanup."
   - Y1: "I will provide document in current state."
2. **System → WASM**:
   - X2: "I will pass text to cleanup function."
   - Y2: "I will accept cleaned text."
3. **WASM → System**:
   - X3: "I will remove whitespace, fix punctuation, adjust headers."
   - Y3: "I will ensure output matches spec."
4. **System → Writer**:
   - X4: "I will replace the document with cleaned version."
   - Y4: "I will notify completion."

---

## 6. Markdown Preview
**Main Promises:**
- **Developer** promises to enter markdown and refresh preview (X).
- **System** promises to render preview (Y).

**Detailed Balanced Promises:**
1. **Developer → System**:
   - X1: "I will type valid markdown and request preview."
   - Y1: "I will accept preview limitations."
2. **System → Renderer**:
   - X2: "I will send markdown to render engine."
   - Y2: "I will accept rendered HTML."
3. **Renderer → System**:
   - X3: "I will convert markdown to HTML."
   - Y3: "I will ensure safe output."
4. **System → Developer**:
   - X4: "I will display rendered preview side-by-side."
   - Y4: "I will refresh on update command."

---

## 7. Export as PromiseGrid CBOR
**Main Promises:**
- **Researcher** promises to export as CBOR (X).
- **System** promises to generate CBOR file (Y).

**Detailed Balanced Promises:**
1. **Researcher → System**:
   - X1: "I will request export in PromiseGrid CBOR."
   - Y1: "I will provide required metadata."
2. **System → WASM**:
   - X2: "I will pass document to encoding function."
   - Y2: "I will accept CBOR output."
3. **WASM → System**:
   - X3: "I will encode data with 'grid' tag."
   - Y3: "I will validate encoding."
4. **System → Researcher**:
   - X4: "I will initiate file download."
   - Y4: "I will name file per convention."

---

## 8. Collaborative Search
**Main Promises:**
- **User** promises to perform search (X).
- **System** promises to highlight without affecting others (Y).

**Detailed Balanced Promises:**
1. **User → System**:
   - X1: "I will enter a search term."
   - Y1: "I will not expect changes to others' views."
2. **System → Search Engine**:
   - X2: "I will run case-insensitive match."
   - Y2: "I will return all match positions."
3. **System → User**:
   - X3: "I will highlight matches in your view."
   - Y3: "I will preserve others' autonomy."

---

## 9. Link Conversion via WASM
**Main Promises:**
- **User** promises to insert plain URL (X).
- **System** promises to convert to markdown link (Y).

**Detailed Balanced Promises:**
1. **User → System**:
   - X1: "I will select URL text."
   - Y1: "I will confirm conversion action."
2. **System → WASM**:
   - X2: "I will pass URL to conversion function."
   - Y2: "I will accept formatted link."
3. **WASM → System**:
   - X3: "I will produce valid markdown link format."
   - Y3: "I will avoid double-formatting existing links."
4. **System → User**:
   - X4: "I will replace selection with markdown link."
   - Y4: "I will confirm conversion in view."

---

## 10. Real-Time Word Count
**Main Promises:**
- **Blogger** promises to set word goal (X).
- **System** promises to track and display (Y).

**Detailed Balanced Promises:**
1. **Blogger → System**:
   - X1: "I will declare a numeric word count goal."
   - Y1: "I will accept calculated stats."
2. **System → Stats Engine**:
   - X2: "I will pass current text to stats calculator."
   - Y2: "I will accept word/char counts."
3. **Stats Engine → System**:
   - X3: "I will return counts and reading time."
   - Y3: "I will ensure accuracy."
4. **System → Blogger**:
   - X4: "I will update counts in real-time."
   - Y4: "I will signal when target is met."

---

## 11. Activity Log for Oversight
**Main Promises:**
- **Team Lead** promises to enable log (X).
- **System** promises to record events (Y).

**Detailed Balanced Promises:**
1. **Lead → System**:
   - X1: "Enable logging for this session."
   - Y1: "I will accept captured events."
2. **System → All Users**:
   - X2: "I will observe join/leave/name/color events."
   - Y2: "I will not log private content."
3. **Users → System**:
   - X3: "I will send visible state changes."
   - Y3: "I will allow them to be logged."
4. **System → Lead**:
   - X4: "I will display the log when requested."
   - Y4: "I will ensure entries are accurate."

---

## 12. Undo/Redo Across Edits
**Main Promises:**
- **User** promises to perform undo (X).
- **System** promises to revert for all (Y).

**Detailed Balanced Promises:**
1. **User → System**:
   - X1: "I will invoke undo command."
   - Y1: "I will specify the scope of change."
2. **System → History Manager**:
   - X2: "I will request last change."
   - Y2: "I will accept diff for reversal."
3. **History Manager → System**:
   - X3: "I will return reversed change."
   - Y3: "I will ensure integrity."
4. **System → All Peers**:
   - X4: "I will apply reversed change to shared state."
   - Y4: "I will confirm update to all."

---

## 13. Export Multiple Formats
**Main Promises:**
- **Researcher** promises to export in multiple formats (X).
- **System** promises to produce each export (Y).

**Detailed Balanced Promises:**
1. **Researcher → System**:
   - X1: "I will request exports in txt, cbor, PromiseGrid CBOR, ysnap."
   - Y1: "I will accept each output."
2. **System → Export Engines**:
   - X2: "I will pass document to each format handler."
   - Y2: "I will accept formatted output."
3. **Engines → System**:
   - X3: "I will produce output in requested format."
   - Y3: "I will ensure fidelity."
4. **System → Researcher**:
   - X4: "I will deliver all files."
   - Y4: "I will name them per rules."

---

## 14. Testing PromiseGrid Structure
**Main Promises:**
- **Developer** promises to create/test message (X).
- **System** promises to output structure (Y).

**Detailed Balanced Promises:**
1. **Developer → System**:
   - X1: "I will call createPromiseGridMessage with params."
   - Y1: "I will accept returned bytes."
2. **System → WASM**:
   - X2: "I will pass parameters to encoding function."
   - Y2: "I will accept CBOR message."
3. **WASM → System**:
   - X3: "I will encode per PromiseGrid spec."
   - Y3: "I will report size and structure."
4. **System → Developer**:
   - X4: "I will log structured message to console."
   - Y4: "I will make it human-readable."

---

## 15. Room-Based Project Isolation
**Main Promises:**
- **Company** promises to create distinct rooms (X).
- **System** promises to maintain isolation (Y).

**Detailed Balanced Promises:**
1. **Company → System**:
   - X1: "I will request creation of separate rooms for each project."
   - Y1: "I will accept unique room IDs."
2. **System → Company**:
   - X2: "I will isolate data/logs per room."
   - Y2: "I will confirm isolation policy."
3. **Users → System**:
   - X3: "I will join only my assigned room."
   - Y3: "I will respect access boundaries."
4. **System → Users**:
   - X4: "I will enforce that no cross-room leakage occurs."
   - Y4: "I will alert if isolation is breached."

---

**End of Document**


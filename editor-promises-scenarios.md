## 1. Real-Time Multi-User Editing

**User Scenario:**
A study group of four students opens the same `?room=math-notes` URL. Each student sees others’ cursors with distinct colors and names, edits different sections simultaneously, and watches changes appear instantly without overwriting each other’s work.

**Main Promises:**

* User 1 promises to broadcast edits (X).
* Users 2–4 promise to receive and integrate those edits (Y).

**Detailed Balanced Promises:**

* **User 1 → System**:

  * X1: "I will send my local edits to the system."
  * Y1: "I will accept confirmation from the system."
* **System → Peers**:

  * X2: "I will forward edits to all connected peers."
  * Y2: "I will preserve edit order and integrity."
* **Peer → System**:

  * X3: "I will acknowledge receipt of edits."
  * Y3: "I will not reject well-formed edits."
* **Peer → Peer (via system)**:

  * X4: "I will incorporate changes into my local state."
  * Y4: "I will accept updated document state."

---

## 2. Creating and Sharing a New Document

**User Scenario:**
A project manager presses **Ctrl+N** to create a fresh document with a new UUID-based room. They copy the room URL from the File menu and paste it into their team’s chat for immediate collaboration.

**Main Promises:**

* Project manager promises to create a new document and share the link (X).
* Team members promise to join the shared room (Y).

**Detailed Balanced Promises:**

* **Manager → System**:

  * X1: "I will request a new document/room."
  * Y1: "I will accept the new room ID and URL from the system."
* **System → Manager**:

  * X2: "I will generate a unique room and URL."
  * Y2: "I will confirm creation."
* **Manager → Team**:

  * X3: "I will send you the room URL."
  * Y3: "I will accept your confirmation that you received it."
* **Team → System**:

  * X4: "I will join using the provided URL."
  * Y4: "I will accept connection and shared document state."

---

## 3. Formatting and PromiseGrid Logging

**User Scenario:**
A journalist selects a paragraph and presses **Ctrl+B** to make it bold. In the browser console, they see a PromiseGrid CBOR message generated for the formatting action.

**Main Promises:**

* User promises to apply formatting (X).
* System promises to generate and log the PromiseGrid message (Y).

**Detailed Balanced Promises:**

* **User → System**:

  * X1: "I will request bold formatting on selected text."
  * Y1: "I will accept system confirmation or formatting preview."
* **System → WASM**:

  * X2: "I will send formatting request to WASM."
  * Y2: "I will accept processed formatted content."
* **WASM → System**:

  * X3: "I will apply bold formatting and create PromiseGrid message."
  * Y3: "I will return results without errors."
* **System → User**:

  * X4: "I will update the editor and log the PromiseGrid message."
  * Y4: "I will confirm to you that the log entry exists."

---

## 4. Offline Editing and Sync

**User Scenario:**
A user is disconnected while writing due to a network outage. The editor switches to offline mode, saving edits in IndexedDB. When the connection returns, all changes are automatically synced with collaborators.

**Main Promises:**

* User promises to edit offline (X).
* System promises to store edits and sync when online (Y).

**Detailed Balanced Promises:**

* **User → System**:

  * X1: "I will continue editing offline."
  * Y1: "I will trust the system to store my edits locally."
* **System → IndexedDB**:

  * X2: "I will save edits as they happen."
  * Y2: "I will allow retrieval later without loss."
* **System → Network (when online)**:

  * X3: "I will transmit stored edits to peers."
  * Y3: "I will accept acknowledgments of receipt."
* **Peers → System**:

  * X4: "I will integrate received edits."
  * Y4: "I will not overwrite valid concurrent changes."

---

## 5. Format Document Cleanup

**User Scenario:**
A writer imports a messy draft and clicks **Format Document**. The WASM engine fixes punctuation spacing, removes extra whitespace, and corrects markdown headers instantly.

**Main Promises:**

* User promises to request cleanup (X).
* System promises to apply formatting rules (Y).

**Detailed Balanced Promises:**

* **User → System**:

  * X1: "I will request document-wide cleanup."
  * Y1: "I will accept the modified version if it meets formatting rules."
* **System → WASM**:

  * X2: "I will send document to WASM for cleanup."
  * Y2: "I will accept the cleaned version."
* **WASM → System**:

  * X3: "I will process and return cleaned content."
  * Y3: "I will not alter meaning of the text."
* **System → User**:

  * X4: "I will update your document with the cleaned version."
  * Y4: "I will allow you to undo if needed."

---

## 6. Markdown Preview for Technical Writing

**User Scenario:**
A developer writing documentation enables **Markdown Preview** with **Ctrl+M** and refreshes the preview with **Ctrl+R** after adding new code blocks.

**Main Promises:**

* User promises to request preview (X).
* System promises to render and update the preview (Y).

**Detailed Balanced Promises:**

* **User → System**:

  * X1: "I will request to enable markdown preview."
  * Y1: "I will accept the rendered pane as-is."
* **System → Renderer**:

  * X2: "I will send current document to preview renderer."
  * Y2: "I will accept rendered HTML."
* **Renderer → System**:

  * X3: "I will generate HTML from markdown."
  * Y3: "I will preserve formatting semantics."
* **System → User**:

  * X4: "I will display side-by-side view."
  * Y4: "I will refresh when you request."

---

## 7. Export as PromiseGrid CBOR

**User Scenario:**
A blockchain researcher completes a paper and exports it as PromiseGrid CBOR for storage in a decentralized network.

**Main Promises:**

* User promises to export (X).
* System promises to generate CBOR file (Y).

**Detailed Balanced Promises:**

* **User → System**:

  * X1: "I will request export as PromiseGrid CBOR."
  * Y1: "I will provide document metadata."
* **System → WASM**:

  * X2: "I will send state to encoder."
  * Y2: "I will accept CBOR byte stream."
* **WASM → System**:

  * X3: "I will encode with official 'grid' tag."
  * Y3: "I will ensure protocol compliance."
* **System → User**:

  * X4: "I will deliver the CBOR file to you."
  * Y4: "I will name it per document title."

---

## 8. Collaborative Search

**User Scenario:**
Two collaborators are editing a long policy document. One searches for "compliance" and jumps between matches without affecting the other’s cursor or edits.

**Main Promises:**

* Searching user promises to run a query (X).
* System promises to show results without disrupting others (Y).

**Detailed Balanced Promises:**

* **User → System**:

  * X1: "I will type a search term."
  * Y1: "I will accept results display in my own session."
* **System → Search Engine (WASM)**:

  * X2: "I will provide document content and query."
  * Y2: "I will accept list of matches."
* **WASM → System**:

  * X3: "I will find matches accurately."
  * Y3: "I will not modify the text."
* **System → User**:

  * X4: "I will highlight matches for you."
  * Y4: "I will not affect other users' cursors."

---

## 9. Link Conversion with WASM

**User Scenario:**
A user pastes a URL, selects it, and clicks **Insert Link**. The editor converts it to markdown format without duplicating existing links.

**Main Promises:**

* User promises to supply a valid URL (X).
* System promises to convert it properly (Y).

**Detailed Balanced Promises:**

* **User → System**:

  * X1: "I will provide a valid URL in my selection."
  * Y1: "I will accept conversion to markdown format."
* **System → WASM**:

  * X2: "I will send URL to link converter."
  * Y2: "I will accept formatted markdown link."
* **WASM → System**:

  * X3: "I will detect and format URL."
  * Y3: "I will not double-format existing links."
* **System → User**:

  * X4: "I will replace selected text with formatted link."
  * Y4: "I will keep surrounding text intact."

---

## 10. Real-Time Word Count

**User Scenario:**
A blogger sets a 1,000-word goal and tracks progress using live word count in the status bar.

**Main Promises:**

* User promises to set a target (X).
* System promises to update live stats (Y).

**Detailed Balanced Promises:**

* **User → System**:

  * X1: "I will request word count tracking."
  * Y1: "I will accept counts as reported."
* **System → Counter (WASM)**:

  * X2: "I will send updated document for counting."
  * Y2: "I will accept accurate counts."
* **WASM → System**:

  * X3: "I will compute counts quickly."
  * Y3: "I will include word, char, and reading time."
* **System → User**:

  * X4: "I will show updated count in status bar."
  * Y4: "I will refresh as you type."

---

## 11. Activity Log for Team Oversight

**User Scenario:**
A team lead enables the Activity Log to monitor join/leave/name changes.

**Main Promises:**

* Lead promises to enable logging (X).
* System promises to record and show events (Y).

**Detailed Balanced Promises:**

* **Lead → System**:

  * X1: "I will turn on logging mode."
  * Y1: "I will accept events captured in this mode."
* **System → All Users**:

  * X2: "I will listen for state changes."
  * Y2: "I will not log private content."
* **Users → System**:

  * X3: "I will send my join/leave/name change events."
  * Y3: "I will allow these to be visible in the log."
* **System → Lead**:

  * X4: "I will display the activity log when requested."
  * Y4: "I will ensure data accuracy."

---

## 12. Undo/Redo Across Collaborative Edits

**User Scenario:**
A user undoes an accidental deletion, restoring the text for everyone.

**Main Promises:**

* User promises to request undo (X).
* System promises to revert shared state (Y).

**Detailed Balanced Promises:**

* **User → System**:

  * X1: "I will request undo of last change."
  * Y1: "I will accept restored state."
* **System → State Manager**:

  * X2: "I will revert to previous snapshot."
  * Y2: "I will accept if revert is possible."
* **State Manager → System**:

  * X3: "I will supply the previous state."
  * Y3: "I will confirm restoration validity."
* **System → All Users**:

  * X4: "I will broadcast restored state."
  * Y4: "I will overwrite current view with restored version."

---

## 13. Exporting Multiple Formats

**User Scenario:**
A researcher exports `.txt`, `.cbor`, PromiseGrid CBOR, and `.ysnap` formats for the same document.

**Main Promises:**

* User promises to request all formats (X).
* System promises to generate each one correctly (Y).

**Detailed Balanced Promises:**

* **User → System**:

  * X1: "I will request export in multiple formats."
  * Y1: "I will provide necessary metadata."
* **System → Export Handlers**:

  * X2: "I will route each format request to proper handler."
  * Y2: "I will accept output files."
* **Handlers → System**:

  * X3: "I will generate correct file for my format."
  * Y3: "I will follow naming rules."
* **System → User**:

  * X4: "I will deliver all files."
  * Y4: "I will confirm success."

---

## 14. Testing PromiseGrid Structure

**User Scenario:**
A developer creates a PromiseGrid message in the console and inspects its structure.

**Main Promises:**

* Developer promises to run the test (X).
* System promises to output valid structure (Y).

**Detailed Balanced Promises:**

* **Developer → System**:

  * X1: "I will call createPromiseGridMessage."
  * Y1: "I will accept the returned data."
* **System → WASM**:

  * X2: "I will pass parameters for encoding."
  * Y2: "I will accept encoded message."
* **WASM → System**:

  * X3: "I will encode per protocol."
  * Y3: "I will report errors if any."
* **System → Developer**:

  * X4: "I will log the message in readable form."
  * Y4: "I will confirm byte size and tag."

---

## 15. Room-Based Project Isolation

**User Scenario:**
A company uses separate rooms for each client to keep documents and logs isolated.

**Main Promises:**

* Company promises to create unique rooms (X).
* System promises to maintain isolation (Y).

**Detailed Balanced Promises:**

* **Company → System**:

  * X1: "I will request creation of separate rooms."
  * Y1: "I will accept assigned room IDs."
* **System → Room Manager**:

  * X2: "I will configure isolation rules per room."
  * Y2: "I will accept only matching-room traffic."
* **Room Manager → System**:

  * X3: "I will enforce isolation boundaries."
  * Y3: "I will confirm room integrity."
* **System → Company**:

  * X4: "I will keep logs and edits confined per room."
  * Y4: "I will prevent cross-room data leaks."

---
 

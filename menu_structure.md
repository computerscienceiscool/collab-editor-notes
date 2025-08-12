# Collaborative Editor Menu Structure

This document outlines the complete menu structure for the collaborative editor, showing implemented features and planned additions.

**Legend:**
- **Bold** = Currently implemented
- *Italic* = Planned/should be added
- *(Keyboard shortcut)* = Suggested shortcuts

---

## **File Menu**

### **New & Open**
- **New Document** *(Ctrl+N)*
- *Open Document* *(Ctrl+O)*
- *Recent Documents*
  - *Document 1*
  - *Document 2*
  - *Clear Recent*

### **Save & Export**
- **Export as .txt** *(Ctrl+Shift+T)*
- **Export CodeMirror State (.json)**
- **Save as .cbor** 
- **PromiseGrid CBOR (.cbor)**
- **Export Yjs Snapshot (.ysnap)**
- **Export Yjs Update (.json)**
- *Export as .md* *(Ctrl+Shift+M)*
- *Export as PDF* *(Ctrl+Shift+P)*
- *Export as HTML* *(Ctrl+Shift+H)*

### **Document Properties**
- *Document Info*
  - *Creation Date*
  - *Last Modified*
  - *File Size*
  - *Collaborators*
- *Document Settings*
  - *Auto-save Interval*
  - *Backup Options*

---

## **Edit Menu**

### **Basic Operations**
- **Undo** *(Ctrl+Z)*
- **Redo** *(Ctrl+Y)*
- *Cut* *(Ctrl+X)*
- *Copy* *(Ctrl+C)*
- *Paste* *(Ctrl+V)*
- *Select All* *(Ctrl+A)*

### **Find & Replace**
- **Find** *(Ctrl+F)*
- **Clear Search** *(Escape)*
- *Find Next* *(F3)*
- *Find Previous* *(Shift+F3)*
- *Replace* *(Ctrl+H)*
- *Replace All* *(Ctrl+Shift+H)*

### **Advanced Editing**
- *Go to Line* *(Ctrl+G)*
- *Duplicate Line* *(Ctrl+D)*
- *Delete Line* *(Ctrl+Shift+K)*
- *Move Line Up* *(Alt+↑)*
- *Move Line Down* *(Alt+↓)*

---

## **Format Menu**

### **Text Formatting**
- **Bold** *(Ctrl+B)*
- **Italic** *(Ctrl+I)*
- **Underline** *(Ctrl+U)*
- **Strikethrough** *(Ctrl+Shift+S)*
- **Convert URL to Link**
- *Clear Formatting* *(Ctrl+Space)*

### **Document Formatting**
- **Format Document** *(Ctrl+Shift+F)*
  - **Clean Whitespace**
  - **Fix Punctuation**
  - **Format Headers**
  - **Format Code Blocks**

### **Paragraph & Lists**
- **Heading 1** *(Ctrl+1)*
- **Heading 2** *(Ctrl+2)*
- **Heading 3** *(Ctrl+3)*
- **Bullet List**
- *Numbered List*
- *Increase Indent* *(Tab)*
- *Decrease Indent* *(Shift+Tab)*

### **Advanced Formatting**
- *Code Block* *(Ctrl+Shift+C)*
- *Quote Block* *(Ctrl+Shift+Q)*
- *Horizontal Rule*
- *Table*
  - *Insert Table*
  - *Add Row*
  - *Add Column*
  - *Delete Row*
  - *Delete Column*

---

## **Insert Menu**

### **Media & Content**
- *Image* *(Ctrl+Shift+I)*
- *Link* *(Ctrl+K)*
- *Table* *(Ctrl+Shift+T)*
- *Code Block*
- *Math Equation* *(LaTeX)*

### **Special Elements**
- *Date/Time* *(Ctrl+;)*
- *Page Break*
- *Footnote*
- *Citation*

### **Templates**
- *Meeting Notes Template*
- *Project Plan Template*
- *Weekly Report Template*
- *Custom Template*

---

## **View Menu**

### **Document View**
- *Zoom In* *(Ctrl+Plus)*
- *Zoom Out* *(Ctrl+Minus)*
- *Reset Zoom* *(Ctrl+0)*
- *Full Screen* *(F11)*
- *Focus Mode* *(F12)*

### **Panels & Toolbars**
- **Toggle Activity Log** *(Ctrl+Shift+L)*
- *Toggle Toolbar*
- *Toggle Status Bar*
- *Toggle Minimap*
- *Toggle Line Numbers*

### **Themes & Appearance**
- *Light Theme*
- *Dark Theme*
- *High Contrast*
- *Custom Theme*

---

## **Collaborate Menu**

### **Session Management**
- **Change Room** *(Current: room-name)*
- **Copy Room URL** *(Ctrl+Shift+U)*
- *Create New Room*
- *Room Settings*
  - *Room Name*
  - *Access Control*
  - *Persistence Settings*

### **User Management**
- **Change Name** *(F2)*
- **Change Color**
- **User List** *(Shows active users)*
- *User Permissions*
  - *View Only*
  - *Comment Only*
  - *Full Edit*

### **Collaboration Features**
- **Show Cursors**
- **Show Typing Indicators**
- *Comments & Suggestions*
  - *Add Comment* *(Ctrl+Alt+M)*
  - *Resolve Comment*
  - *Suggestion Mode*
- *Version History*
  - *View Revisions*
  - *Compare Versions*
  - *Restore Version*

---

## **Tools Menu**

### **Text Processing (WASM)**
- **Document Statistics** *(Live display)*
  - **Word Count**
  - **Character Count**
  - **Reading Time**
- **Document Compression**
  - **Compress Document**
  - **Decompress Document**
- *Spell Check* *(F7)*
- *Grammar Check*
- *Word Frequency Analysis*

### **PromiseGrid Integration**
- **PromiseGrid Messages** *(Console view)*
- **Export as PromiseGrid CBOR**
- *PromiseGrid Node Status*
- *Promise History*
- *Capability Tokens*

### **Developer Tools**
- *JavaScript Console* *(F12)*
- *Network Monitor*
- *Performance Profiler*
- *WASM Debug Info*

---

## **Help Menu**

### **Documentation**
- *User Guide* *(F1)*
- *Keyboard Shortcuts* *(Ctrl+?)*
- *Markdown Reference*
- *PromiseGrid Protocol Guide*

### **Support**
- *Report Bug*
- *Feature Request*
- *Community Forum*
- *About Collaborative Editor*
  - *Version Info*
  - *Credits*
  - *License*

---

## **Context Menus (Right-click)**

### **Text Selection Context**
- **Bold** *(Ctrl+B)*
- **Italic** *(Ctrl+I)*
- **Underline** *(Ctrl+U)*
- **Convert to Link**
- ---
- *Cut* *(Ctrl+X)*
- *Copy* *(Ctrl+C)*
- *Paste* *(Ctrl+V)*
- ---
- *Add Comment*
- *Suggest Edit*

### **User Context (in user list)**
- *View User Profile*
- *Send Direct Message*
- *Grant Permissions*
- *Remove from Session*

---

## **Keyboard Shortcuts Summary**

### **Essential Shortcuts**
| Action | Shortcut | Status |
|--------|----------|--------|
| **Undo** | Ctrl+Z | ✅ Implemented |
| **Redo** | Ctrl+Y | ✅ Implemented |
| **Search** | Enter (in search box) | ✅ Implemented |
| *Bold* | *Ctrl+B* | *Planned* |
| *Italic* | *Ctrl+I* | *Planned* |
| *Format Document* | *Ctrl+Shift+F* | *Planned* |
| *Save/Export* | *Ctrl+S* | *Planned* |

### **Advanced Shortcuts**
| Action | Shortcut | Status |
|--------|----------|--------|
| *Focus Search* | *Ctrl+F* | *Planned* |
| *Toggle Log* | *Ctrl+Shift+L* | *Planned* |
| *Change Name* | *F2* | *Planned* |
| *Full Screen* | *F11* | *Planned* |

---

## **Implementation Priority**

### **High Priority** (Immediate)
1. **Keyboard shortcuts for formatting** (Bold, Italic, Underline)
2. **Ctrl+F for search focus**
3. **Ctrl+S for export menu**
4. **Context menus for text selection**

### **Medium Priority** (Soon)
1. *Document templates*
2. *Advanced search (Find/Replace)*
3. *Theme switching*
4. *Zoom controls*

### **Low Priority** (Future)
1. *Comments and suggestions*
2. *Version history*
3. *Advanced collaboration features*
4. *Plugin system*

---

**This menu structure transforms the collaborative editor from a basic tool into a professional document editing platform with comprehensive PromiseGrid integration.**
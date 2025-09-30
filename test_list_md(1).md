# Complete Test List - All 247 Tests

This document lists **every single individual test** in the Collaborative Editor test suite.

---

## 1. tests/e2e/collaboration/multi-user.spec.js

**Total: 2 tests**

### Real-time Collaboration

1. two users can edit simultaneously
2. user presence and awareness features work

---

## 2. tests/e2e/core/basic.spec.js

**Total: 3 tests**

### Basic Editor Functionality

1. should load editor successfully
2. should handle basic text input
3. should apply text formatting

---

## 3. tests/e2e/core/keyboard-shortcuts.spec.js

**Total: 59 tests**

### Keyboard Shortcuts

1. Ctrl+B applies bold formatting
2. Ctrl+I applies italic formatting
3. Ctrl+U applies underline formatting
4. Ctrl+Shift+X applies strikethrough
5. formatting shortcuts can be combined

### Heading Shortcuts

6. Ctrl+Alt+1 creates H1 heading
7. Ctrl+Alt+2 creates H2 heading
8. Ctrl+Alt+3 creates H3 heading

### List Shortcuts

9. Ctrl+Shift+8 creates bullet list
10. Ctrl+Shift+7 creates numbered list

### Document Navigation Shortcuts

11. Ctrl+A selects all text
12. Ctrl+F focuses search box
13. Ctrl+N creates new document
14. Ctrl+Shift+U copies room URL

### Edit Operation Shortcuts

15. copy and paste work with keyboard shortcuts
16. cut operation works

### Interface Shortcuts

17. Ctrl+Alt+Y toggles toolbar
18. Esc closes open menus

### Tools Shortcuts

19. Ctrl+Shift+C shows word count

### Link and URL Shortcuts

20. Ctrl+K processes URLs into links

### Advanced Formatting Shortcuts

21. Ctrl+Alt+4 creates H4 heading
22. Ctrl+Alt+5 creates H5 heading
23. Ctrl+Alt+6 creates H6 heading

### Advanced List Operations

24. Tab indents list items
25. Shift+Tab outdents list items

### Document Management Shortcuts

26. Ctrl+S triggers save action
27. Ctrl+Z undoes changes
28. Ctrl+Y redoes changes

### Selection and Navigation

29. Ctrl+Home goes to document start
30. Ctrl+End goes to document end
31. Shift+Arrow keys extend selection

### Quick Actions

32. Ctrl+D duplicates current line
33. Ctrl+/ toggles line comment

### System Integration Shortcuts

34. Alt+Tab simulation (window switching behavior)
35. Page Up/Down navigation

### Extended Formatting Shortcuts

36. Ctrl+Shift+K deletes current line
37. Ctrl+L selects current line
38. Ctrl+Shift+D duplicates current line

### Advanced Text Manipulation

39. Ctrl+Shift+Arrow extends selection by word
40. Alt+Arrow moves by word

### Find and Replace Shortcuts

41. Ctrl+H opens find and replace
42. F3 finds next occurrence
43. Shift+F3 finds previous occurrence

### Zoom and View Shortcuts

44. Ctrl+Plus increases zoom
45. Ctrl+Minus decreases zoom
46. Ctrl+0 resets zoom

### Special Character Input

47. Alt codes and special characters
48. Unicode input handling

### Accessibility Shortcuts

49. Tab navigation through interface
50. Shift+Tab reverse navigation
51. Enter activates focused elements

### Browser Compatibility Edge Cases

52. Function keys work correctly
53. Number pad shortcuts
54. Meta key variations (Mac)

### Complex Key Combinations

55. Triple modifier combinations
56. Rapid key combination sequences

### Error Recovery and Edge Cases

57. Invalid key combinations do not break editor
58. Keyboard shortcuts during loading states
59. Shortcuts with special characters in content

---

## 4. tests/e2e/core/menu-system.spec.js

**Total: 10 tests**

### Menu System Functionality

1. all main menus are visible and clickable
2. File menu actions work correctly
3. Edit menu keyboard shortcuts work
4. Format menu applies text formatting
5. Tools menu shows document statistics
6. View menu toggles interface elements
7. ESC key closes open menus
8. multiple menu operations work in sequence
9. menu accessibility and keyboard navigation
10. menu error handling and recovery

---

## 5. tests/e2e/core/preferences-dialog.spec.js

**Total: 23 tests**

### Dialog Opening and Closing

1. opens preferences dialog via menu
2. opens via window.preferencesDialog.show()
3. closes via X button
4. closes via Close button
5. closes via Escape key
6. closes via click outside modal
7. does not close when clicking inside dialog
8. prevents multiple dialogs from opening

### Shortcut Display and Organization

9. displays shortcuts organized by category
10. displays shortcut items with description and key
11. shows correct default shortcuts

### Shortcut Editing Functionality

12. enters edit mode when clicking shortcut key
13. updates shortcut with valid key combination
14. rejects invalid key combinations
15. supports function key shortcuts
16. persists shortcut changes to localStorage

### Reset to Defaults Functionality

17. shows confirmation dialog for reset
18. resets shortcuts when confirmed

### Edge Cases and Error Handling

19. handles missing shortcutManager gracefully
20. preserves state after page interactions

### Accessibility

21. dialog has proper ARIA attributes
22. keyboard navigation works within dialog

### Integration with Shortcut Manager

23. updates work with shortcut manager state

---

## 6. tests/e2e/core/wasm-features.spec.js

**Total: 12 tests**

### WASM Text Processing Features

1. text formatting functions work correctly
2. document compression works correctly
3. document statistics are calculated correctly
4. document search functionality works
5. URL link conversion works correctly
6. markdown formatting toggles work correctly
7. document format function cleans up text
8. PromiseGrid protocol integration works
9. WASM functions handle edge cases correctly
10. WASM performance is acceptable for large documents
11. WASM functions integrate properly with editor state
12. WASM error handling works correctly

---

## 7. tests/e2e/github/github-commit.spec.js

**Total: 24 tests**

### Dialog Opening

1. opens commit dialog via File menu
2. opens commit dialog via keyboard shortcut
3. shows configuration prompt when GitHub not configured

### Repository and File Path Selection

4. populates repository dropdown with available repos
5. allows file path input and validates format
6. remembers last used repository and path

### Commit Message Input

7. provides default commit message
8. allows custom commit messages
9. validates commit message requirements

### Co-Author Detection

10. detects co-authors from awareness system
11. shows placeholder when no collaborators present
12. generates appropriate email addresses for co-authors

### Commit Execution

13. validates required fields before committing
14. executes successful commit with co-authors
15. handles commit failures gracefully
16. shows loading state during commit

### Settings Update After Commit

17. saves commit preferences for future use
18. updates last commit information

### Dialog Interaction and UX

19. closes dialog after successful commit
20. allows manual dialog closure
21. provides accessible focus management

### Edge Cases and Error Handling

22. handles very long file paths
23. handles special characters in commit messages
24. preserves dialog state during temporary failures

---

## 8. tests/e2e/github/github-integration-a.spec.js

**Total: 13 tests**

### Complete Workflows

1. setup → commit → pull workflow
2. settings persistence across operations

### Multi-User Collaboration with Co-Authors

3. detects and includes multiple collaborators
4. handles dynamic collaborator changes
5. handles edge cases in collaboration detection

### Security Testing

6. prevents XSS in commit messages
7. sanitizes file paths and prevents directory traversal
8. prevents XSS in file content
9. validates user input and prevents injection

### Performance with Large Documents

10. handles large document commits efficiently
11. handles large document pulls efficiently
12. maintains responsiveness during large operations
13. handles memory efficiently with multiple large operations

---

## 9. tests/e2e/github/github-integration-b.spec.js

**Total: 27 tests**

### Menu Integration

1. GitHub menu items are properly added to File menu
2. GitHub Settings menu item is added to Tools menu
3. GitHub Commit menu action works correctly
4. GitHub Pull menu action works correctly
5. GitHub Settings menu action works correctly
6. menu items handle unconfigured GitHub gracefully
7. menu integration survives page navigation
8. menu interactions are properly tracked

### Keyboard Shortcuts

9. Ctrl+Alt+G opens GitHub commit dialog
10. keyboard shortcuts work when modal is already open
11. keyboard shortcuts are disabled when other modals are open
12. keyboard shortcuts work across different browser contexts
13. keyboard shortcuts are registered in shortcut manager
14. keyboard shortcuts can be customized
15. keyboard shortcut performance is acceptable

### Error Recovery Scenarios

16. recovers from network failures during commit
17. recovers from invalid token scenarios
18. recovers from repository access errors
19. recovers from malformed settings data
20. recovers from dialog state corruption
21. recovers from simultaneous operation conflicts
22. maintains data integrity during recovery
23. provides helpful error messages for common issues
24. recovery state is properly tracked and logged
25. graceful degradation when GitHub service is unavailable
26. comprehensive error boundary testing

### Integration Test Summary

27. integration test summary and verification

---

## 10. tests/e2e/github/github-pull.spec.js

**Total: 32 tests**

### Dialog Opening

1. opens pull dialog via File menu
2. shows configuration prompt when GitHub not configured
3. properly initializes dialog with current settings

### Repository and File Path Selection

4. populates repository dropdown with available repos
5. allows switching between repositories
6. validates and accepts various file path formats
7. remembers last used repository and path

### File Preview Functionality

8. shows placeholder text when no file selected
9. loads and displays file preview on demand
10. shows different content for different files
11. handles large file previews appropriately
12. displays preview for empty files
13. handles unicode and special characters in preview

### Content Replacement Warning

14. displays warning about content replacement
15. shows confirmation dialog before replacing content
16. proceeds with replacement when confirmed

### Pull Flow Execution

17. validates required fields before pulling
18. executes successful pull operation
19. handles different file types correctly
20. shows loading state during pull operation
21. closes dialog after successful pull

### Error Handling for Missing Files

22. handles file not found errors gracefully
23. handles repository access errors
24. handles network and API errors
25. preserves dialog state after errors

### Editor Content Update

26. completely replaces editor content
27. preserves editor state and functionality after pull
28. handles different content encodings correctly
29. updates document statistics after pull

### Dialog Interaction and UX

30. allows manual dialog closure
31. provides accessible focus management
32. provides configure GitHub shortcut

---

## 11. tests/e2e/github/github-settings.spec.js

**Total: 27 tests**

### Dialog Opening and Closing

1. opens GitHub settings via Tools menu
2. opens GitHub settings via keyboard shortcut
3. closes via X button
4. closes via Cancel button
5. closes via Escape key
6. closes via click outside

### Token Management

7. validates valid GitHub token
8. handles invalid GitHub token
9. shows loading state during validation
10. handles empty token validation

### Repository Management

11. fetches and displays repositories
12. refreshes repository list
13. handles repository fetch failure
14. selects repository and sets default values

### Settings Persistence

15. saves settings to localStorage
16. loads existing settings on dialog open
17. clears all settings
18. cancels clear settings operation

### Error Handling and Edge Cases

19. handles network timeout during token validation
20. validates required fields before saving
21. handles malformed token input
22. preserves unsaved changes when dialog is reopened
23. handles concurrent dialog operations

### Accessibility and Usability

24. dialog has proper ARIA attributes
25. form elements are properly labeled
26. supports keyboard navigation
27. provides helpful placeholder text

---

## 12. tests/e2e/security/xss-prevention.spec.js

**Total: 12 tests**

### Security - XSS Prevention

1. prevents script execution in document content
2. validates user input fields
3. validates export filename safety
4. validates PromiseGrid message security
5. ensures CSP compliance
6. verifies secure defaults are maintained
7. handles edge case XSS vectors
8. prevents XSS during various application operations
9. protects against CBOR injection attacks
10. handles extremely large documents safely
11. protects against URL manipulation
12. prevents DOM-based XSS through content manipulation

---

## Summary

**Total Tests: 247**

| # | File | Tests |
|---|------|-------|
| 1 | multi-user.spec.js | 2 |
| 2 | basic.spec.js | 3 |
| 3 | keyboard-shortcuts.spec.js | 59 |
| 4 | menu-system.spec.js | 10 |
| 5 | preferences-dialog.spec.js | 23 |
| 6 | wasm-features.spec.js | 12 |
| 7 | github-commit.spec.js | 24 |
| 8 | github-integration-a.spec.js | 13 |
| 9 | github-integration-b.spec.js | 27 |
| 10 | github-pull.spec.js | 32 |
| 11 | github-settings.spec.js | 27 |
| 12 | xss-prevention.spec.js | 12 |
| **TOTAL** | | **247** |

---

*Complete test reference - Every individual test listed*

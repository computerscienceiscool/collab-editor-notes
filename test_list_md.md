# Test Suite Documentation

This document provides a comprehensive list of all test files and their individual tests in the Collaborative Editor test suite.

---
 
## Test Files Overview

### 1. `tests/e2e/collaboration/multi-user.spec.js`
Tests for real-time collaboration features between multiple users.

### 2. `tests/e2e/core/basic.spec.js`
Core functionality tests for basic editor operations.

### 3. `tests/e2e/core/keyboard-shortcuts.spec.js`
Comprehensive tests for all keyboard shortcuts and key combinations.

### 4. `tests/e2e/core/menu-system.spec.js`
Tests for menu system functionality and interactions.

### 5. `tests/e2e/core/preferences-dialog.spec.js`
Tests for the preferences dialog and keyboard shortcut customization.

### 6. `tests/e2e/core/wasm-features.spec.js`
Tests for WebAssembly-based text processing features.

### 7. `tests/e2e/github/github-commit.spec.js`
Tests for GitHub commit functionality and dialog interactions.

### 8. `tests/e2e/github/github-integration-a.spec.js`
First part of comprehensive GitHub integration tests covering workflows, collaboration, security, and performance.

### 9. `tests/e2e/github/github-integration-b.spec.js`
Second part of GitHub integration tests covering menu integration, keyboard shortcuts, and error recovery.

### 10. `tests/e2e/github/github-pull.spec.js`
Tests for GitHub pull functionality and content retrieval.

### 11. `tests/e2e/github/github-settings.spec.js`
Tests for GitHub settings dialog and configuration management.

### 12. `tests/e2e/security/xss-prevention.spec.js`
Security tests for XSS prevention across all application features.

---

## Detailed Test List

### tests/e2e/collaboration/multi-user.spec.js

**Real-time Collaboration**

- **two users can edit simultaneously** - Tests that two users can join the same room, see each other, and edit content that synchronizes between them
- **user presence and awareness features work** - Tests that user lists show all collaborators, typing indicators appear, and awareness features function correctly

---

### tests/e2e/core/basic.spec.js

**Basic Editor Functionality**

- **should load editor successfully** - Verifies that all essential editor elements (editor, title, name input) are visible on page load
- **should handle basic text input** - Tests that text can be typed into the editor and retrieved correctly
- **should apply text formatting** - Tests that basic formatting (bold) can be applied to selected text

---

### tests/e2e/core/keyboard-shortcuts.spec.js

**Keyboard Shortcuts**

- **Ctrl+B applies bold formatting** - Tests bold formatting via keyboard shortcut with retry logic and fallbacks
- **Ctrl+I applies italic formatting** - Tests italic formatting via keyboard shortcut with enhanced waiting
- **Ctrl+U applies underline formatting** - Tests underline formatting via keyboard shortcut with retry logic
- **Ctrl+Shift+X applies strikethrough** - Tests strikethrough formatting via keyboard shortcut with fallback support
- **formatting shortcuts can be combined** - Tests that multiple formatting types (bold + italic) can be applied sequentially

**Heading Shortcuts**

- **Ctrl+Alt+1 creates H1 heading** - Tests H1 heading creation via keyboard shortcut with fallback to direct function call
- **Ctrl+Alt+2 creates H2 heading** - Tests H2 heading creation via keyboard shortcut with retry logic
- **Ctrl+Alt+3 creates H3 heading** - Tests H3 heading creation via keyboard shortcut with flexible assertions

**List Shortcuts**

- **Ctrl+Shift+8 creates bullet list** - Tests bullet list creation via keyboard shortcut with enhanced retry and fallback
- **Ctrl+Shift+7 creates numbered list** - Tests numbered list creation with acceptance of either bullet or numbered format

**Document Navigation Shortcuts**

- **Ctrl+A selects all text** - Tests select all functionality by verifying deletion of selected content
- **Ctrl+F focuses search box** - Tests that search input receives focus when shortcut is pressed
- **Ctrl+N creates new document** - Tests new document creation with dialog handling and URL verification
- **Ctrl+Shift+U copies room URL** - Tests that room URL is copied to clipboard with alert confirmation

**Edit Operation Shortcuts**

- **copy and paste work with keyboard shortcuts** - Tests full copy/paste cycle with enhanced waiting for paste operation
- **cut operation works** - Tests that cut removes content and paste restores it

**Interface Shortcuts**

- **Ctrl+Alt+Y toggles toolbar** - Tests toolbar visibility toggle with enhanced retry logic
- **Esc closes open menus** - Tests that Escape key closes open dropdown menus

**Tools Shortcuts**

- **Ctrl+Shift+C shows word count** - Tests word count dialog appearance with enhanced wait for alert

**Link and URL Shortcuts**

- **Ctrl+K processes URLs into links** - Tests URL to markdown link conversion with fallback to direct function call

**Advanced Formatting Shortcuts**

- **Ctrl+Alt+4 creates H4 heading** - Tests H4 heading creation with enhanced retry and fallback
- **Ctrl+Alt+5 creates H5 heading** - Tests H5 heading creation with retry logic
- **Ctrl+Alt+6 creates H6 heading** - Tests H6 heading creation with retry logic

**Advanced List Operations**

- **Tab indents list items** - Tests that Tab key indents list items in markdown format
- **Shift+Tab outdents list items** - Tests that Shift+Tab removes indentation from list items

**Document Management Shortcuts**

- **Ctrl+S triggers save action** - Tests that save shortcut doesn't break the editor (web app behavior)
- **Ctrl+Z undoes changes** - Tests undo functionality with enhanced wait and retry
- **Ctrl+Y redoes changes** - Tests redo functionality after undo operation

**Selection and Navigation**

- **Ctrl+Home goes to document start** - Tests cursor movement to document start by typing at that position
- **Ctrl+End goes to document end** - Tests cursor movement to document end by typing at that position
- **Shift+Arrow keys extend selection** - Tests text selection extension and replacement

**Quick Actions**

- **Ctrl+D duplicates current line** - Tests line duplication (if implemented in app)
- **Ctrl+/ toggles line comment** - Tests comment toggling functionality (if implemented)

**System Integration Shortcuts**

- **Alt+Tab simulation (window switching behavior)** - Simulates window switching and verifies editor remains functional
- **Page Up/Down navigation** - Tests page navigation in large documents

**Extended Formatting Shortcuts**

- **Ctrl+Shift+K deletes current line** - Tests line deletion shortcut (if implemented)
- **Ctrl+L selects current line** - Tests line selection by replacing selected content
- **Ctrl+Shift+D duplicates current line** - Tests line duplication shortcut (if implemented)

**Advanced Text Manipulation**

- **Ctrl+Shift+Arrow extends selection by word** - Tests word-by-word selection extension
- **Alt+Arrow moves by word** - Tests word-by-word cursor movement

**Find and Replace Shortcuts**

- **Ctrl+H opens find and replace** - Tests find/replace dialog opening (if implemented)
- **F3 finds next occurrence** - Tests next search result navigation
- **Shift+F3 finds previous occurrence** - Tests previous search result navigation

**Zoom and View Shortcuts**

- **Ctrl+Plus increases zoom** - Tests zoom increase and verifies editor functionality
- **Ctrl+Minus decreases zoom** - Tests zoom decrease and verifies editor functionality
- **Ctrl+0 resets zoom** - Tests zoom reset and verifies editor functionality

**Special Character Input**

- **Alt codes and special characters** - Tests input of special characters (©, ®, ™, etc.)
- **Unicode input handling** - Tests Unicode character input (emojis, symbols)

**Accessibility Shortcuts**

- **Tab navigation through interface** - Tests keyboard navigation through UI elements
- **Shift+Tab reverse navigation** - Tests reverse keyboard navigation
- **Enter activates focused elements** - Tests element activation via Enter key

**Browser Compatibility Edge Cases**

- **Function keys work correctly** - Tests that F1-F12 keys don't break the editor
- **Number pad shortcuts** - Tests number pad key input with fallback to regular numbers
- **Meta key variations (Mac)** - Tests Meta key shortcuts on WebKit browsers

**Complex Key Combinations**

- **Triple modifier combinations** - Tests Ctrl+Alt+Shift combinations if implemented
- **Rapid key combination sequences** - Tests rapid sequential formatting operations

**Error Recovery and Edge Cases**

- **Invalid key combinations do not break editor** - Tests that unusual key combinations don't crash the editor
- **Keyboard shortcuts during loading states** - Tests shortcuts during rapid operations with recovery verification
- **Shortcuts with special characters in content** - Tests formatting application to content containing special characters

---

### tests/e2e/core/menu-system.spec.js

**Menu System Functionality**

- **all main menus are visible and clickable** - Tests that all six main menus (file, edit, format, tools, view, help) open correctly
- **File menu actions work correctly** - Tests File menu actions including new document creation
- **Edit menu keyboard shortcuts work** - Tests Edit menu shortcuts (select all, undo, copy/paste)
- **Format menu applies text formatting** - Tests Format menu formatting options (bold, italic) with retry logic
- **Tools menu shows document statistics** - Tests Tools menu word count dialog with enhanced retry
- **View menu toggles interface elements** - Tests View menu toggles for activity log and toolbar visibility
- **ESC key closes open menus** - Tests that Escape key closes any open dropdown menu
- **multiple menu operations work in sequence** - Tests sequential menu operations (File → Edit → Format)
- **menu accessibility and keyboard navigation** - Tests keyboard navigation to and through menus
- **menu error handling and recovery** - Tests menu system under error conditions (rapid clicking, clicking outside, after editor operations)

---

### tests/e2e/core/preferences-dialog.spec.js

**Dialog Opening and Closing**

- **opens preferences dialog via menu** - Tests opening preferences dialog through Tools menu
- **opens via window.preferencesDialog.show()** - Tests programmatic dialog opening
- **closes via X button** - Tests dialog closure using the X button
- **closes via Close button** - Tests dialog closure using the Close button in footer
- **closes via Escape key** - Tests dialog closure using Escape key
- **closes via click outside modal** - Tests dialog closure by clicking on overlay
- **does not close when clicking inside dialog** - Tests that clicking dialog content doesn't close it
- **prevents multiple dialogs from opening** - Tests that only one dialog instance can be open

**Shortcut Display and Organization**

- **displays shortcuts organized by category** - Tests that shortcuts are grouped by categories (File, Edit, Format, Tools, View)
- **displays shortcut items with description and key** - Tests shortcut item structure (description, key, data attributes)
- **shows correct default shortcuts** - Tests that default shortcuts (Ctrl+B, Ctrl+I, etc.) are displayed correctly

**Shortcut Editing Functionality**

- **enters edit mode when clicking shortcut key** - Tests that clicking a shortcut enters edit mode
- **updates shortcut with valid key combination** - Tests shortcut update with valid key combination (Ctrl+Shift+B)
- **rejects invalid key combinations** - Tests that invalid shortcuts (just a letter) are rejected
- **supports function key shortcuts** - Tests that function keys (F2, etc.) can be used as shortcuts
- **persists shortcut changes to localStorage** - Tests that shortcut changes are saved and persist across dialog reopens

**Reset to Defaults Functionality**

- **shows confirmation dialog for reset** - Tests that reset button shows confirmation dialog
- **resets shortcuts when confirmed** - Tests that shortcuts are reset to defaults when user confirms

**Edge Cases and Error Handling**

- **handles missing shortcutManager gracefully** - Tests that dialog handles missing shortcut manager without crashing
- **preserves state after page interactions** - Tests that shortcut changes persist after clicking elsewhere on page

**Accessibility**

- **dialog has proper ARIA attributes** - Tests that dialog has role="dialog" and aria-labelledby attributes
- **keyboard navigation works within dialog** - Tests Tab navigation through dialog elements

**Integration with Shortcut Manager**

- **updates work with shortcut manager state** - Tests that dialog changes update both shortcut manager state and action mapping

---

### tests/e2e/core/wasm-features.spec.js

**WASM Text Processing Features**

- **text formatting functions work correctly** - Tests WASM-based text formatting (removing extra spaces)
- **document compression works correctly** - Tests WASM document compression and decompression with round-trip verification
- **document statistics are calculated correctly** - Tests WASM document statistics calculation (words, characters) via status bar and direct function
- **document search functionality works** - Tests WASM-based document search with search input and direct function call
- **URL link conversion works correctly** - Tests WASM URL to markdown link conversion
- **markdown formatting toggles work correctly** - Tests WASM markdown formatting (bold, italic, underline, strikethrough) toggle functions
- **document format function cleans up text** - Tests WASM document formatting (cleanup of excessive whitespace, line breaks)
- **PromiseGrid protocol integration works** - Tests PromiseGrid message creation functions (with mock fallback)
- **WASM functions handle edge cases correctly** - Tests WASM functions with edge cases (empty strings, single characters, long strings, Unicode, special characters)
- **WASM performance is acceptable for large documents** - Tests WASM performance with large documents (processing time, responsiveness)
- **WASM functions integrate properly with editor state** - Tests that WASM operations update editor state correctly
- **WASM error handling works correctly** - Tests WASM function error handling with various inputs

---

### tests/e2e/github/github-commit.spec.js

**Dialog Opening**

- **opens commit dialog via File menu** - Tests opening GitHub commit dialog through File menu
- **opens commit dialog via keyboard shortcut** - Tests opening commit dialog using Ctrl+Alt+G
- **shows configuration prompt when GitHub not configured** - Tests that unconfigured GitHub shows configuration prompt

**Repository and File Path Selection**

- **populates repository dropdown with available repos** - Tests that repository dropdown shows available repositories
- **allows file path input and validates format** - Tests file path input with various formats
- **remembers last used repository and path** - Tests persistence of repository and path selections

**Commit Message Input**

- **provides default commit message** - Tests that default commit message is pre-populated
- **allows custom commit messages** - Tests custom commit message input
- **validates commit message requirements** - Tests that empty commit messages are rejected

**Co-Author Detection**

- **detects co-authors from awareness system** - Tests that multiple collaborators are detected and listed as co-authors
- **shows placeholder when no collaborators present** - Tests placeholder text when user is alone
- **generates appropriate email addresses for co-authors** - Tests email generation based on co-author names

**Commit Execution**

- **validates required fields before committing** - Tests validation of repository, path, and message fields
- **executes successful commit with co-authors** - Tests full commit flow with co-authors included
- **handles commit failures gracefully** - Tests error handling for failed commits
- **shows loading state during commit** - Tests loading state and button disabling during commit operation

**Settings Update After Commit**

- **saves commit preferences for future use** - Tests persistence of commit settings for next use
- **updates last commit information** - Tests storage of last commit information (SHA, URL, etc.)

**Dialog Interaction and UX**

- **closes dialog after successful commit** - Tests auto-close after successful commit
- **allows manual dialog closure** - Tests manual closure via Close, Cancel, and Escape
- **provides accessible focus management** - Tests keyboard navigation and ARIA attributes

**Edge Cases and Error Handling**

- **handles very long file paths** - Tests commit with very long file path
- **handles special characters in commit messages** - Tests commit messages with emojis and special characters
- **preserves dialog state during temporary failures** - Tests that form data is preserved after temporary failure

---

### tests/e2e/github/github-integration-a.spec.js

**Complete Workflows**

- **setup → commit → pull workflow** - Tests complete GitHub workflow: configure, commit content, modify locally, pull back, verify roundtrip
- **settings persistence across operations** - Tests that GitHub settings persist across commit and pull operations

**Multi-User Collaboration with Co-Authors**

- **detects and includes multiple collaborators** - Tests detection of multiple collaborators (4 users) with proper co-author generation
- **handles dynamic collaborator changes** - Tests detection when new collaborator joins during commit dialog open
- **handles edge cases in collaboration detection** - Tests collaborators without names, empty names, invalid user objects

**Security Testing**

- **prevents XSS in commit messages** - Tests that malicious script tags in commit messages don't execute
- **sanitizes file paths and prevents directory traversal** - Tests path sanitization for directory traversal attempts
- **prevents XSS in file content** - Tests that malicious content in pulled files is displayed as text, not executed
- **validates user input and prevents injection** - Tests that malicious user names are safely displayed

**Performance with Large Documents**

- **handles large document commits efficiently** - Tests commit performance with large document (100K+ characters)
- **handles large document pulls efficiently** - Tests pull performance with large document (50K+ characters)
- **maintains responsiveness during large operations** - Tests UI responsiveness during massive document commit
- **handles memory efficiently with multiple large operations** - Tests memory usage across 5 sequential large operations

---

### tests/e2e/github/github-integration-b.spec.js

**Menu Integration**

- **GitHub menu items are properly added to File menu** - Tests that Commit and Pull menu items appear in File menu with shortcuts
- **GitHub Settings menu item is added to Tools menu** - Tests that Settings menu item appears in Tools menu
- **GitHub Commit menu action works correctly** - Tests that commit dialog opens via menu with pre-populated fields
- **GitHub Pull menu action works correctly** - Tests that pull dialog opens via menu with all components
- **GitHub Settings menu action works correctly** - Tests that settings dialog opens via menu with all components
- **menu items handle unconfigured GitHub gracefully** - Tests that menu actions prompt for configuration when not set up
- **menu integration survives page navigation** - Tests that menu items persist after navigating to different room
- **menu interactions are properly tracked** - Tests tracking of menu interaction types

**Keyboard Shortcuts**

- **Ctrl+Alt+G opens GitHub commit dialog** - Tests commit dialog opening via keyboard shortcut with content
- **keyboard shortcuts work when modal is already open** - Tests shortcut behavior when another modal is open
- **keyboard shortcuts are disabled when other modals are open** - Tests that GitHub shortcuts are blocked when preferences dialog is open
- **keyboard shortcuts work across different browser contexts** - Tests shortcuts from different focus contexts (editor, title, search, name)
- **keyboard shortcuts are registered in shortcut manager** - Tests that GitHub shortcuts appear in shortcut manager
- **keyboard shortcuts can be customized** - Tests customization of GitHub shortcuts in preferences dialog
- **keyboard shortcut performance is acceptable** - Tests that shortcuts respond within acceptable time (<1s average, <2s max)

**Error Recovery Scenarios**

- **recovers from network failures during commit** - Tests error handling and retry after network failure
- **recovers from invalid token scenarios** - Tests recovery from invalid token with correction to valid token
- **recovers from repository access errors** - Tests recovery from non-existent file with correction to valid file
- **recovers from malformed settings data** - Tests recovery from corrupted localStorage settings with fallback to defaults
- **recovers from dialog state corruption** - Tests recovery from corrupted dialog state with functionality verification
- **recovers from simultaneous operation conflicts** - Tests handling of conflicting dialog operations (commit vs pull, simultaneous shortcuts)
- **maintains data integrity during recovery** - Tests that editor content and settings are preserved through various error scenarios
- **provides helpful error messages for common issues** - Tests quality of error messages for missing token, empty path, empty message, no repository
- **recovery state is properly tracked and logged** - Tests tracking data structure for recovery attempts
- **graceful degradation when GitHub service is unavailable** - Tests that editor continues working when GitHub service is deleted
- **comprehensive error boundary testing** - Tests error boundaries for JSON parsing, DOM manipulation, and event handlers

**Integration Test Summary**

- **integration test summary and verification** - Comprehensive summary test verifying all integration data was collected and coverage areas were tested

---

### tests/e2e/github/github-pull.spec.js

**Dialog Opening**

- **opens pull dialog via File menu** - Tests opening GitHub pull dialog through File menu
- **shows configuration prompt when GitHub not configured** - Tests prompt when GitHub is not configured
- **properly initializes dialog with current settings** - Tests that dialog loads with current repository and path settings

**Repository and File Path Selection**

- **populates repository dropdown with available repos** - Tests that repository dropdown shows available repositories
- **allows switching between repositories** - Tests repository selection changes
- **validates and accepts various file path formats** - Tests file path input with various formats
- **remembers last used repository and path** - Tests persistence of repository and path selections

**File Preview Functionality**

- **shows placeholder text when no file selected** - Tests placeholder when path is empty
- **loads and displays file preview on demand** - Tests preview loading and display of markdown content
- **shows different content for different files** - Tests preview updates when switching between files
- **handles large file previews appropriately** - Tests preview of large files
- **displays preview for empty files** - Tests handling of empty file content
- **handles unicode and special characters in preview** - Tests preview of Unicode characters and emojis

**Content Replacement Warning**

- **displays warning about content replacement** - Tests visibility of warning message
- **shows confirmation dialog before replacing content** - Tests confirmation dialog with dismiss option
- **proceeds with replacement when confirmed** - Tests content replacement when user accepts

**Pull Flow Execution**

- **validates required fields before pulling** - Tests validation of repository and path fields
- **executes successful pull operation** - Tests full pull flow with content replacement
- **handles different file types correctly** - Tests pulling single-line, unicode, and empty files
- **shows loading state during pull operation** - Tests loading status and button disabling
- **closes dialog after successful pull** - Tests auto-close after successful pull

**Error Handling for Missing Files**

- **handles file not found errors gracefully** - Tests error display for non-existent files
- **handles repository access errors** - Tests error handling for invalid repository
- **handles network and API errors** - Tests error handling for network timeout
- **preserves dialog state after errors** - Tests that form data is preserved after temporary failure

**Editor Content Update**

- **completely replaces editor content** - Tests that pull completely replaces existing complex content
- **preserves editor state and functionality after pull** - Tests that editor remains functional and formatting works after pull
- **handles different content encodings correctly** - Tests that Unicode content is preserved correctly
- **updates document statistics after pull** - Tests that word/character counts update after pull

**Dialog Interaction and UX**

- **allows manual dialog closure** - Tests manual closure via Close, Cancel, and Escape buttons
- **provides accessible focus management** - Tests keyboard navigation and ARIA attributes
- **provides configure GitHub shortcut** - Tests Configure GitHub button functionality

---

### tests/e2e/github/github-settings.spec.js

**Dialog Opening and Closing**

- **opens GitHub settings via Tools menu** - Tests opening settings dialog through Tools menu
- **opens GitHub settings via keyboard shortcut** - Tests opening via Ctrl+Alt+Shift+G
- **closes via X button** - Tests closure using X button
- **closes via Cancel button** - Tests closure using Cancel button
- **closes via Escape key** - Tests closure using Escape key
- **closes via click outside** - Tests closure by clicking overlay

**Token Management**

- **validates valid GitHub token** - Tests validation of valid-looking token with success status
- **handles invalid GitHub token** - Tests error handling for invalid token
- **shows loading state during validation** - Tests loading status during token validation
- **handles empty token validation** - Tests error message for empty token

**Repository Management**

- **fetches and displays repositories** - Tests repository dropdown population after token validation
- **refreshes repository list** - Tests refresh button functionality
- **handles repository fetch failure** - Tests error handling when repository fetch fails
- **selects repository and sets default values** - Tests repository selection and related settings

**Settings Persistence**

- **saves settings to localStorage** - Tests that settings are saved with success message
- **loads existing settings on dialog open** - Tests that pre-existing settings are loaded into form
- **clears all settings** - Tests clear functionality with confirmation
- **cancels clear settings operation** - Tests cancel on clear confirmation dialog

**Error Handling and Edge Cases**

- **handles network timeout during token validation** - Tests timeout handling during validation
- **validates required fields before saving** - Tests that token is required before saving
- **handles malformed token input** - Tests various malformed token formats
- **preserves unsaved changes when dialog is reopened** - Tests that unsaved changes don't persist
- **handles concurrent dialog operations** - Tests that only one dialog can be open

**Accessibility and Usability**

- **dialog has proper ARIA attributes** - Tests role="dialog" and aria-labelledby
- **form elements are properly labeled** - Tests that inputs have associated labels
- **supports keyboard navigation** - Tests Tab navigation through form elements
- **provides helpful placeholder text** - Tests placeholder text in inputs

---

### tests/e2e/security/xss-prevention.spec.js

**Security - XSS Prevention**

- **prevents script execution in document content** - Tests 16 different XSS vectors in editor content (script tags, img onerror, svg onload, javascript:, iframes, objects, embeds, etc.)
- **validates user input fields** - Tests malicious input in name, search, and title fields with 13 different XSS vectors
- **validates export filename safety** - Tests export functionality with malicious filenames (directory traversal, Windows reserved names, script injection, special characters)
- **validates PromiseGrid message security** - Tests PromiseGrid operations with malicious payloads (script tags, eval, javascript:, prototype pollution)
- **ensures CSP compliance** - Tests for Content Security Policy violations during various operations
- **verifies secure defaults are maintained** - Comprehensive security check: no inline handlers, no script tags, no javascript: URLs, no XSS flags, safe editor HTML
- **handles edge case XSS vectors** - Tests 25+ edge case vectors (encoding-based, Unicode, CSS, meta tags, data URIs, event handlers, prototype pollution, template literals, null bytes)
- **prevents XSS during various application operations** - Tests XSS prevention during bold formatting, search, document formatting, title setting, and user name setting
- **protects against CBOR injection attacks** - Tests CBOR export with various content types including special characters, binary data, Unicode, large content, and malicious JSON
- **handles extremely large documents safely** - Tests security and performance with large documents (repeated text, single line, many lines, mixed HTML-like content)
- **protects against URL manipulation** - Tests 13 malicious URL patterns (script tags, javascript:, data URIs, file:, path traversal, Unicode, encoding attacks)
- **prevents DOM-based XSS through content manipulation** - Tests 13 DOM-based attack vectors (onclick, style injection, iframes, eval, setTimeout, forms, details, video/audio, etc.)

---

## Test Utilities

### tests/global-setup.js
Global setup that waits for the development server to be ready and verifies basic WASM environment before tests run.

### tests/utils/testHelpers.js
Comprehensive helper class (`CollabEditorHelpers`) providing:
- Navigation and initialization
- Editor content manipulation
- Formatting operations
- Menu system interactions
- User collaboration features
- Document statistics
- Export operations
- Search functionality
- WASM function testing
- Security testing helpers
- Performance measurement
- Mobile device support
- Enhanced logging with test context

---

## Test Statistics

- **Total Test Files**: 12
- **Total Tests**: 200+ individual tests
- **Coverage Areas**:
  - Core editor functionality
  - Real-time collaboration
  - Keyboard shortcuts (50+ shortcuts)
  - Menu system
  - Preferences and customization
  - WASM text processing
  - GitHub integration (commit, pull, settings)
  - Security (XSS prevention with 100+ attack vectors)
  - Error recovery and edge cases
  - Performance and memory management
  - Accessibility
  - Mobile device compatibility

---

## Running Tests

```bash
# Run all tests
npm test

# Run specific test file
npm test tests/e2e/core/basic.spec.js

# Run tests in headed mode (see browser)
npm test -- --headed

# Run tests in specific browser
npm test -- --project=chromium
npm test -- --project=firefox
npm test -- --project=webkit

# Run tests with UI
npm test -- --ui
```

---

## Test Environment

- **Test Framework**: Playwright
- **Browsers**: Chromium, Firefox, WebKit (Safari)
- **Features**:
  - Cross-browser testing
  - Mobile device emulation
  - Network condition simulation
  - Screenshot capture on failure
  - Video recording
  - Trace files for debugging
  - Parallel test execution
  - Retry logic for flaky tests

---

*Last Updated: 2024*

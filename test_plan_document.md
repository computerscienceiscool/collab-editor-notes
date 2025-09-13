# Collaborative Editor Test Plan

## Overview
This document outlines the comprehensive testing strategy for the collaborative text editor application, including all test files, target functionality, and execution strategy.

## Test Structure

### Current Test Directory Structure
```
tests/
├── e2e/                          # End-to-end tests
│   ├── collaboration/            # Multi-user collaboration tests
│   │   └── multi-user.spec.js
│   ├── core/                     # Core functionality tests
│   │   ├── basic.spec.js
│   │   ├── keyboard-shortcuts.spec.js
│   │   ├── menu-system.spec.js
│   │   └── wasm-features.spec.js
│   └── security/                 # Security and XSS prevention
│       └── xss-prevention.spec.js
├── utils/                        # Test utilities and helpers
│   └── testHelpers.js
├── global-setup.js              # Global test configuration
└── playwright.config.js         # Playwright configuration
```

### Source Files Under Test
```
src/
├── app.js                        # Main application entry
├── export/
│   └── handlers.js              # Export and formatting handlers
├── setup/
│   ├── documentCopy.js          # Document copying functionality
│   ├── editorSetup.js           # CodeMirror editor initialization
│   ├── userSetup.js             # User configuration
│   └── yjsSetup.js              # Yjs collaboration setup
├── ui/                          # User interface components
│   ├── cursorWidget.js
│   ├── documentStats.js
│   ├── logging.js
│   ├── preferencesDialog.js
│   ├── remoteCursorPlugin.js
│   ├── shortcutManager.js
│   ├── typingIndicator.js
│   └── userList.js
├── utils/
│   └── timeUtils.js
└── wasm/
    └── initWasm.js              # WASM function initialization
```

### WASM/Rust Components
```
rust-wasm/
├── src/
│   ├── lib.rs                   # Main WASM functions
│   └── promisegrid.rs           # PromiseGrid protocol implementation
└── Cargo.toml                   # Rust dependencies
```

## Test Categories and Coverage

### 1. Core Editor Functionality (`tests/e2e/core/basic.spec.js`)
**Status: PASSING**

- **Editor Initialization**
  - Application loads successfully
  - CodeMirror editor initializes
  - DOM elements are present and visible

- **Basic Text Operations**
  - Text input and display
  - Content persistence
  - Editor state management

- **Text Formatting**
  - Bold, italic, underline formatting
  - Strikethrough formatting
  - Format application and removal

**Test Commands:**
```bash
npx playwright test tests/e2e/core/basic.spec.js --project=chromium
```

### 2. WASM Text Processing (`tests/e2e/core/wasm-features.spec.js`)
**Status: PASSING with mocks**

- **Text Formatting Functions**
  - Document cleanup and formatting
  - Whitespace normalization
  - Text structure improvement

- **Document Statistics**
  - Word, character, and line counting
  - Reading time estimation
  - Real-time stats updates

- **Search Functionality**
  - Document text search
  - Case-sensitive/insensitive search
  - Search result highlighting

- **URL Processing**
  - URL detection in text
  - Markdown link conversion
  - Link formatting

- **PromiseGrid Integration**
  - CBOR message generation
  - Protocol compliance
  - Error handling

- **Performance Testing**
  - Large document handling
  - Processing time validation
  - Memory usage optimization

- **Edge Case Handling**
  - Empty content
  - Unicode characters
  - Special characters
  - Error recovery

**Test Commands:**
```bash
npx playwright test tests/e2e/core/wasm-features.spec.js --project=chromium
```

### 3. Keyboard Shortcuts (`tests/e2e/core/keyboard-shortcuts.spec.js`)
**Status: MIXED - Some passing, timeouts on others**

- **Text Formatting Shortcuts**
  - Ctrl+B (Bold)
  - Ctrl+I (Italic)
  - Ctrl+U (Underline)
  - Ctrl+Shift+X (Strikethrough)

- **Document Operations**
  - Ctrl+N (New document)
  - Ctrl+A (Select all)
  - Ctrl+F (Find)
  - Ctrl+S (Save)

- **Edit Operations**
  - Ctrl+Z (Undo)
  - Ctrl+Y (Redo)
  - Ctrl+C (Copy)
  - Ctrl+V (Paste)
  - Ctrl+X (Cut)

- **Interface Controls**
  - Ctrl+Shift+T (Toggle toolbar)
  - Ctrl+Alt+L (Toggle activity log)
  - Esc (Close menus)

**Test Commands:**
```bash
npx playwright test tests/e2e/core/keyboard-shortcuts.spec.js --project=chromium
```

### 4. Menu System (`tests/e2e/core/menu-system.spec.js`)
**Status: MIXED - Basic functionality works, some timeouts**

- **Menu Navigation**
  - All main menus (File, Edit, Format, Tools, View, Help)
  - Menu opening and closing
  - Keyboard navigation
  - Click-outside behavior

- **File Menu Actions**
  - New document creation
  - Document copying
  - Export functionality (multiple formats)
  - Share and collaboration features

- **Edit Menu Integration**
  - Undo/redo operations
  - Copy/paste functionality
  - Text selection operations

- **Format Menu Features**
  - Text formatting options
  - Heading levels
  - List formatting
  - Link insertion

- **Tools and Utilities**
  - Word count and statistics
  - PromiseGrid testing
  - Settings and preferences

**Test Commands:**
```bash
npx playwright test tests/e2e/core/menu-system.spec.js --project=chromium
```

### 5. Security Testing (`tests/e2e/security/xss-prevention.spec.js`)
**Status: PASSING**

- **XSS Prevention**
  - Script tag sanitization
  - Event handler prevention
  - HTML injection protection
  - URL manipulation safety

- **Input Validation**
  - Document title safety
  - User input field validation
  - Search input sanitization

- **Export Security**
  - Filename sanitization
  - CBOR injection prevention
  - Large document handling

- **Content Security Policy**
  - CSP compliance validation
  - Inline script prevention
  - Resource loading restrictions

**Test Commands:**
```bash
npx playwright test tests/e2e/security/xss-prevention.spec.js --project=chromium
```

### 6. Real-time Collaboration (`tests/e2e/collaboration/multi-user.spec.js`)
**Status: NEEDS MULTI-BROWSER TESTING**

- **Multi-User Editing**
  - Simultaneous editing
  - Conflict resolution
  - Document synchronization

- **User Awareness**
  - Presence indicators
  - Typing notifications
  - User list management
  - Remote cursor display

- **Connection Management**
  - WebSocket connectivity
  - Offline/online transitions
  - Reconnection handling

**Test Commands:**
```bash
npx playwright test tests/e2e/collaboration/multi-user.spec.js --project=chromium
```

## Browser Compatibility Matrix

| Test Suite | Chromium | Firefox | WebKit | Mobile Chrome |
|------------|----------|---------|--------|---------------|
| Basic Functionality | ✅ | ⚠️ | ⚠️ | ✅ |
| WASM Features | ✅ | ⚠️ | ⚠️ | ✅ |
| Keyboard Shortcuts | ✅ | ❌ | ❌ | ✅ |
| Menu System | ✅ | ❌ | ❌ | ⚠️ |
| Security | ✅ | ❌ | ❌ | ✅ |
| Collaboration | 🔄 | 🔄 | 🔄 | 🔄 |

**Legend:**
- ✅ Passing
- ⚠️ Mostly passing with some timeouts
- ❌ Failing due to timeouts
- 🔄 Requires testing

## Test Execution Strategy

### Phase 1: Core Functionality (Current Status)
**Target: All Chromium tests passing**
- [x] Basic editor functionality
- [x] WASM text processing with mocks
- [x] Security and XSS prevention
- [ ] Keyboard shortcuts (fix timeout issues)
- [ ] Menu system (fix timeout issues)

### Phase 2: Browser Compatibility
**Target: Cross-browser support**
- [ ] Firefox timeout optimization
- [ ] WebKit compatibility fixes
- [ ] Mobile Chrome consistency

### Phase 3: Real WASM Integration
**Target: Replace mocks with actual WASM**
- [ ] WASM module loading verification
- [ ] Rust function compilation
- [ ] Performance benchmarking

### Phase 4: Advanced Features
**Target: Full feature coverage**
- [ ] Multi-user collaboration testing
- [ ] Offline/online synchronization
- [ ] Performance under load

## Test Commands Reference

### Run All Tests
```bash
# All tests, all browsers
npx playwright test

# Single browser
npx playwright test --project=chromium

# Headed mode for debugging
npx playwright test --headed
```

### Run Specific Test Suites
```bash
# Core functionality only
npx playwright test tests/e2e/core/ --project=chromium

# Security tests only
npx playwright test tests/e2e/security/ --project=chromium

# Single test file
npx playwright test tests/e2e/core/basic.spec.js --project=chromium

# Specific test by name
npx playwright test -g "should handle basic text input" --project=chromium
```

### Debug Commands
```bash
# Run with debug output
npx playwright test --debug

# Generate and view test report
npx playwright test --reporter=html
npx playwright show-report

# View trace files
npx playwright show-trace test-results/[trace-file].zip
```

## Known Issues and Fixes Applied

### 1. Editor Input Timeouts
**Issue:** `page.type()` and `page.click()` timing out on CodeMirror
**Fix:** Direct CodeMirror dispatch API usage in `testHelpers.js`

### 2. WASM Function Availability
**Issue:** WASM functions not available in test environment
**Fix:** Comprehensive mock functions in `setupWasmMocking()`

### 3. Browser-Specific Timeouts
**Issue:** Firefox and WebKit taking longer than Chromium
**Fix:** Browser-specific timeout configurations in `playwright.config.js`

### 4. PromiseGrid Mock Behavior
**Issue:** Mock functions returning success instead of failure
**Fix:** Proper error simulation for unavailable functions

## Maintenance and Updates

### Regular Test Maintenance
- Weekly execution of full test suite
- Monthly review of timeout configurations
- Quarterly update of test scenarios

### Adding New Tests
1. Create test file in appropriate `/e2e/` subdirectory
2. Use `CollabEditorHelpers` class for common operations
3. Include browser compatibility considerations
4. Add debug output for troubleshooting

### Performance Benchmarks
- Maximum test execution time: 20 seconds per test
- Target success rate: 95% across all browsers
- Acceptable timeout rate: <5% of total tests

## Future Test Enhancements

### Planned Additions
- **Visual Regression Testing:** Screenshot comparisons for UI consistency
- **Accessibility Testing:** Screen reader and keyboard navigation validation
- **Load Testing:** Multiple concurrent users simulation
- **Mobile Responsive Testing:** Touch interface validation
- **Integration Testing:** Real WebSocket server testing
- **Performance Monitoring:** Memory usage and rendering speed metrics

### Test Infrastructure Improvements
- Parallel test execution optimization
- Test data management and cleanup
- Continuous integration pipeline integration
- Automated test result reporting
- Test environment standardization
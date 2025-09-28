# Required Files for Test Rewrite

## Files You Must Include in Each Prompt

### 1. **Core Files** (Always include these)
```
- [TEST FILE] - The specific test file you want rewritten
- missing-files-analysis.md - The analysis I just created
- [REWRITE PROMPT] - The prompt above
```

### 2. **Source Files Referenced by the Test**
Include the specific source files that each test file is testing:

#### For GitHub Integration Tests:
```
- src/github/githubService.js
- src/github/githubMenuIntegration.js  
- src/ui/githubDialog.js
- src/ui/githubCommitDialog.js
- src/ui/githubPullDialog.js
```

#### For Menu/UI Tests:
```
- index.html (for DOM structure reference)
- src/ui/preferencesDialog.js
- src/ui/shortcutManager.js
- src/export/handlers.js (for menu actions)
```

#### For WASM/Text Processing Tests:
```
- src/wasm/initWasm.js
- rust-wasm/src/lib.rs
```

#### For Collaboration Tests:
```
- src/setup/yjsSetup.js
- src/setup/editorSetup.js
- src/ui/userList.js
- src/ui/logging.js
- src/ui/typingIndicator.js
- src/ui/remoteCursorPlugin.js
```

#### For Export/File Handling Tests:
```
- src/export/handlers.js
- src/setup/documentCopy.js
```

### 3. **Configuration Context** (Include if relevant)
```
- style.css (for UI/CSS-related tests)
- src/app.js (for integration tests)
```

## Prompt Template

For each test file, use this template:

```
Please rewrite this test file following the requirements in the attached prompt. 

**Current test file to rewrite:**
[PASTE TEST FILE CONTENT]

**Rewrite requirements:**
[PASTE THE REWRITE PROMPT]

**Missing files analysis:**
[PASTE THE MISSING FILES ANALYSIS]

**Source files being tested:**
[PASTE RELEVANT SOURCE FILES]

The rewritten test should be completely executable and follow modern testing best practices while maintaining the same test coverage scope.
```

## File Combinations by Test Type

### GitHub Integration Test Files:
```
✅ Test file
✅ Rewrite prompt  
✅ Missing files analysis
✅ src/github/githubService.js
✅ src/ui/githubCommitDialog.js
✅ src/ui/githubDialog.js
✅ index.html (for DOM structure)
```

### Menu System Test Files:
```
✅ Test file
✅ Rewrite prompt
✅ Missing files analysis  
✅ index.html
✅ src/export/handlers.js
✅ src/ui/shortcutManager.js
```

### WASM Function Test Files:
```
✅ Test file
✅ Rewrite prompt
✅ Missing files analysis
✅ src/wasm/initWasm.js
✅ rust-wasm/src/lib.rs
```

### Collaboration Test Files:
```
✅ Test file
✅ Rewrite prompt
✅ Missing files analysis
✅ src/setup/yjsSetup.js
✅ src/ui/userList.js
✅ src/ui/remoteCursorPlugin.js
```

This gives the AI all the context it needs to create executable, realistic tests.
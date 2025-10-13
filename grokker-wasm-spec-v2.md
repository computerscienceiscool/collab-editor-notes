# Grokker WASM Commit Message Generator - Implementation Spec v2.0

**Version:** 2.0  
**Date:** 2025-01-08  
**Status:** Ready for Implementation  
**Target:** Generic WASM module usable by any web application

---

## Table of Contents

1. [Changes from v1.0](#changes-from-v10)
2. [Architecture](#architecture)
3. [Files Required](#files-required)
4. [WASM Interface](#wasm-interface)
5. [Implementation Steps](#implementation-steps)
6. [Integration with githubCommitDialog.js](#integration-with-githubcommitdialogjs)
7. [Build Process](#build-process)
8. [Testing](#testing)
9. [Deployment](#deployment)

---

## Changes from v1.0

### Simplified Scope
- **OpenAI-only**: No model gateway, no provider abstraction in v1
- **Direct API calls**: Skip `gateway.go` and `model.go` complexity
- **Single file focus**: Only handles one file at a time

### Diff Approach
- **JavaScript generates diff**: Use `diff` npm package in browser
- **WASM receives unified diff**: Standard format, easy to parse
- **No diff generation in Go**: Simpler WASM code

### Integration Target
- **collab-editor**: Existing `githubCommitDialog.js`
- **Yjs-based editor**: Already has old/new content available
- **GitHub API integration**: Already fetching old content

### File Size Limits
- **10KB soft warning**: "Large file - AI generation may take longer"
- **50KB hard limit**: "File too large for AI analysis (max 50KB)"
- **Prevents token overflow**: Keeps within model limits

### What's Removed
- ❌ Multi-file support
- ❌ Chat history (`chat.go`)
- ❌ RAG features (document retrieval)
- ❌ Embeddings API
- ❌ Database operations
- ❌ Model gateway/routing
- ❌ Multiple provider support

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Browser JavaScript                        │
│                                                              │
│  githubCommitDialog.js                                       │
│    │                                                         │
│    ├─> User clicks "Generate with AI"                       │
│    │                                                         │
│    ├─> Fetch old content from GitHub API                    │
│    │                                                         │
│    ├─> Get new content from Yjs editor                      │
│    │                                                         │
│    ├─> Check file size (10KB warning, 50KB limit)           │
│    │                                                         │
│    ├─> Generate unified diff (using 'diff' library)         │
│    │                                                         │
│    └─> Call Grokker.generateCommitMessage({                 │
│          diff: "...",                                        │
│          filename: "file.js",                                │
│          apiKey: "sk-...",                                   │
│          model: "gpt-4"                                      │
│        })                                                    │
└──────────────────────────┬───────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    WASM Module (Go)                          │
│                                                              │
│  main.go                                                     │
│    │                                                         │
│    ├─> Validate inputs                                      │
│    │   - Check diff not empty                               │
│    │   - Check API key format                               │
│    │   - Check model name                                   │
│    │                                                         │
│    ├─> Set environment: os.Setenv("OPENAI_API_KEY", key)   │
│    │                                                         │
│    ├─> Build system message with instructions               │
│    │                                                         │
│    ├─> Create user message with diff                        │
│    │                                                         │
│    ├─> Call openai.CompleteChat()                           │
│    │                                                         │
│    ├─> Parse response into title/body                       │
│    │                                                         │
│    └─> Return commit message                                │
└──────────────────────────┬───────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    OpenAI API                                │
│                                                              │
│  - Receives system message + diff                           │
│  - Generates commit message                                 │
│  - Returns formatted response                               │
└─────────────────────────────────────────────────────────────┘
```

---

## Files Required

### From Grokker Repository

Copy these files to your WASM project:

```
grokker-wasm/
├── v3/
│   ├── openai/
│   │   └── openai.go          # OpenAI API client
│   ├── client/
│   │   └── chatclient.go      # ChatMsg, Results types
│   └── wasm/
│       └── main.go            # NEW FILE - WASM entry point
├── go.mod                     # Copy and modify
└── go.sum                     # Copy as-is
```

**Files to copy:**
1. `v3/openai/openai.go` - Complete file
2. `v3/client/chatclient.go` - Complete file
3. `v3/go.mod` - Copy and modify module name
4. `v3/go.sum` - Copy as-is

**Files NOT needed:**
- ❌ `v3/core/api.go`
- ❌ `v3/core/gateway.go`
- ❌ `v3/core/model.go`
- ❌ `v3/core/chat.go`
- ❌ `v3/core/document.go`
- ❌ `v3/core/chunk.go`
- ❌ `v3/core/embeddings-openai.go`
- ❌ `v3/core/grokker.go`
- ❌ `v3/core/migrate.go`
- ❌ `v3/core/git.go` (we'll extract just the prompt text)

### From collab-editor Repository

Files to modify:

```
collab-editor/
├── src/
│   ├── ui/
│   │   └── githubCommitDialog.js    # MODIFY - add WASM integration
│   └── index.html                   # MODIFY - load WASM
└── public/
    ├── wasm/
    │   ├── grokker.wasm             # NEW - built WASM binary
    │   ├── grokker.wasm.gz          # NEW - compressed version
    │   ├── wasm_exec.js             # NEW - Go WASM runtime
    │   └── grokker-loader.js        # NEW - convenience wrapper
    └── package.json                 # MODIFY - add 'diff' dependency
```

---

## WASM Interface

### JavaScript API

```javascript
/**
 * Generate a commit message from a file diff
 * @param {Object} params - Parameters
 * @param {string} params.diff - Unified diff text
 * @param {string} params.filename - File name for context
 * @param {string} params.apiKey - OpenAI API key (starts with 'sk-')
 * @param {string} params.model - Model name (e.g., "gpt-4", "gpt-3.5-turbo")
 * @returns {Promise<CommitMessage>} Generated commit message
 */
async function generateCommitMessage(params)
```

### Input Format

```javascript
{
  "diff": "diff --git a/file.js b/file.js\nindex 1234567..89abcdef 100644\n--- a/file.js\n+++ b/file.js\n@@ -1,3 +1,3 @@\n-const x = 1;\n+const x = 2;\n",
  "filename": "file.js",
  "apiKey": "sk-...",
  "model": "gpt-4"
}
```

### Output Format

```javascript
{
  "title": "feat: update variable value in file.js",
  "body": "- Change x from 1 to 2",
  "fullMessage": "feat: update variable value in file.js\n\n- Change x from 1 to 2"
}
```

### Error Format

```javascript
{
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": "Additional details"
}
```

**Error Codes:**
- `INVALID_INPUT` - Missing or invalid parameters
- `FILE_TOO_LARGE` - Diff exceeds 50KB limit
- `OPENAI_API_ERROR` - OpenAI API call failed
- `NETWORK_ERROR` - Network connection failed
- `UNKNOWN_ERROR` - Unexpected error

---

## Implementation Steps

### Step 1: Create WASM Project Structure

```bash
# Create new directory
mkdir grokker-wasm
cd grokker-wasm

# Create directory structure
mkdir -p v3/openai
mkdir -p v3/client
mkdir -p v3/wasm
mkdir -p dist
```

### Step 2: Copy Files from Grokker

```bash
# From grokker repository, copy:
cp path/to/grokker/v3/openai/openai.go v3/openai/
cp path/to/grokker/v3/client/chatclient.go v3/client/
cp path/to/grokker/v3/go.mod .
cp path/to/grokker/v3/go.sum .
```

### Step 3: Modify go.mod

**Before (from Grokker):**
```go
module github.com/stevegt/grokker/v3
```

**After (for WASM project):**
```go
module github.com/yourusername/grokker-wasm

go 1.22.1

require github.com/stevegt/go-openai v0.0.0-20250731211715-61bacff90751

require (
    github.com/stevegt/goadapt v0.7.0
)
```

### Step 4: Create v3/wasm/main.go

This is the NEW file - see complete code in next section.

### Step 5: Build WASM

```bash
# Development build
GOOS=js GOARCH=wasm go build -o dist/grokker.wasm v3/wasm/main.go

# Production build (optimized)
GOOS=js GOARCH=wasm go build -ldflags="-s -w" -o dist/grokker.wasm v3/wasm/main.go

# Compress
gzip -9 -k dist/grokker.wasm
```

### Step 6: Copy Go WASM Runtime

```bash
cp "$(go env GOROOT)/misc/wasm/wasm_exec.js" dist/
```

### Step 7: Create Loader Script

Create `dist/grokker-loader.js` - see code in next section.

### Step 8: Integrate with collab-editor

Modify `githubCommitDialog.js` - see changes in next section.

---

## Complete Code Files

### v3/wasm/main.go (NEW FILE)

```go
//go:build js && wasm

package main

import (
    "context"
    "encoding/json"
    "fmt"
    "os"
    "strings"
    "syscall/js"

    gptLib "github.com/stevegt/go-openai"
    . "github.com/stevegt/goadapt"
    "github.com/yourusername/grokker-wasm/v3/client"
)

func main() {
    c := make(chan struct{})
    
    // Register the JavaScript function
    js.Global().Set("generateCommitMessage", js.FuncOf(generateCommitMessage))
    
    println("Grokker WASM initialized")
    <-c
}

// generateCommitMessage is the WASM entry point called from JavaScript
func generateCommitMessage(this js.Value, args []js.Value) interface{} {
    // Create a Promise that Go will fulfill
    handler := js.FuncOf(func(this js.Value, args []js.Value) interface{} {
        resolve := args[0]
        reject := args[1]
        
        // Run in goroutine to avoid blocking
        go func() {
            // Parse input parameters
            if len(args) < 1 {
                rejectWithError(reject, "INVALID_INPUT", "No parameters provided", "")
                return
            }
            
            params := args[0]
            
            // Extract required fields
            diff := getStringParam(params, "diff")
            filename := getStringParam(params, "filename")
            apiKey := getStringParam(params, "apiKey")
            model := getStringParam(params, "model")
            
            // Validate inputs
            if err := validateInputs(diff, filename, apiKey, model); err != nil {
                rejectWithError(reject, "INVALID_INPUT", err.Error(), "")
                return
            }
            
            // Check file size limit (50KB)
            if len(diff) > 50*1024 {
                rejectWithError(reject, "FILE_TOO_LARGE", 
                    "Diff exceeds 50KB limit", 
                    fmt.Sprintf("Diff size: %d bytes", len(diff)))
                return
            }
            
            // Generate commit message
            result, err := callOpenAI(diff, filename, apiKey, model)
            if err != nil {
                rejectWithError(reject, "OPENAI_API_ERROR", 
                    "Failed to generate commit message", 
                    err.Error())
                return
            }
            
            // Return result as JavaScript object
            resolve.Invoke(resultToJSObject(result))
        }()
        
        return nil
    })
    
    // Create and return a Promise
    promiseConstructor := js.Global().Get("Promise")
    return promiseConstructor.New(handler)
}

// getStringParam extracts a string parameter from a JavaScript object
func getStringParam(obj js.Value, key string) string {
    val := obj.Get(key)
    if val.IsUndefined() || val.IsNull() {
        return ""
    }
    return val.String()
}

// validateInputs checks that all required inputs are present and valid
func validateInputs(diff, filename, apiKey, model string) error {
    if diff == "" {
        return fmt.Errorf("diff is required")
    }
    
    if filename == "" {
        return fmt.Errorf("filename is required")
    }
    
    if apiKey == "" {
        return fmt.Errorf("apiKey is required")
    }
    
    if !strings.HasPrefix(apiKey, "sk-") {
        return fmt.Errorf("invalid API key format (must start with 'sk-')")
    }
    
    if model == "" {
        return fmt.Errorf("model is required")
    }
    
    return nil
}

// CommitResult holds the parsed commit message
type CommitResult struct {
    Title       string `json:"title"`
    Body        string `json:"body"`
    FullMessage string `json:"fullMessage"`
}

// callOpenAI generates a commit message using the OpenAI API
func callOpenAI(diff, filename, apiKey, model string) (*CommitResult, error) {
    // Set API key in environment
    os.Setenv("OPENAI_API_KEY", apiKey)
    defer os.Unsetenv("OPENAI_API_KEY")
    
    // Build system message (from Grokker's git.go)
    sysmsg := `Write a git commit message for the given diff. Use present tense, active, imperative statements as if giving directions. Do not use extra adjectives or marketing hype. The first line of the commit message must be a summary of 60 characters or less, followed by a blank line, followed by bullet-pointed details.`
    
    // Create user message with diff
    userMsg := fmt.Sprintf("%s\n\nFile: %s\n\n%s", sysmsg, filename, diff)
    
    msgs := []gptLib.ChatCompletionMessage{
        {
            Role:    gptLib.ChatMessageRoleSystem,
            Content: sysmsg,
        },
        {
            Role:    gptLib.ChatMessageRoleUser,
            Content: userMsg,
        },
    }
    
    // Create OpenAI client
    client := gptLib.NewClient(apiKey)
    
    // Call OpenAI API
    res, err := client.CreateChatCompletion(
        context.Background(),
        gptLib.ChatCompletionRequest{
            Model:    model,
            Messages: msgs,
        },
    )
    if err != nil {
        return nil, fmt.Errorf("OpenAI API error: %w", err)
    }
    
    if len(res.Choices) == 0 {
        return nil, fmt.Errorf("no response from OpenAI")
    }
    
    // Parse the response
    fullMessage := res.Choices[0].Message.Content
    lines := strings.Split(fullMessage, "\n")
    
    // Extract title (first non-empty line)
    title := ""
    bodyStartIdx := 0
    for i, line := range lines {
        trimmed := strings.TrimSpace(line)
        if trimmed != "" && title == "" {
            title = trimmed
            bodyStartIdx = i + 1
            break
        }
    }
    
    // Extract body (skip blank line after title)
    bodyLines := []string{}
    foundBlankLine := false
    for i := bodyStartIdx; i < len(lines); i++ {
        trimmed := strings.TrimSpace(lines[i])
        if trimmed == "" {
            foundBlankLine = true
            continue
        }
        if foundBlankLine {
            bodyLines = append(bodyLines, lines[i])
        }
    }
    body := strings.Join(bodyLines, "\n")
    
    return &CommitResult{
        Title:       title,
        Body:        body,
        FullMessage: fullMessage,
    }, nil
}

// rejectWithError creates a JavaScript error object and rejects the promise
func rejectWithError(reject js.Value, code, message, details string) {
    errorObj := map[string]interface{}{
        "error":   message,
        "code":    code,
        "details": details,
    }
    reject.Invoke(mapToJSObject(errorObj))
}

// resultToJSObject converts a CommitResult to a JavaScript object
func resultToJSObject(result *CommitResult) js.Value {
    obj := map[string]interface{}{
        "title":       result.Title,
        "body":        result.Body,
        "fullMessage": result.FullMessage,
    }
    return mapToJSObject(obj)
}

// mapToJSObject converts a Go map to a JavaScript object
func mapToJSObject(m map[string]interface{}) js.Value {
    obj := js.Global().Get("Object").New()
    for k, v := range m {
        obj.Set(k, v)
    }
    return obj
}
```

### dist/grokker-loader.js (NEW FILE)

```javascript
// grokker-loader.js
(function() {
    'use strict';
    
    // Create global namespace
    window.Grokker = window.Grokker || {};
    
    // Loading state
    let wasmReady = false;
    let wasmReadyCallbacks = [];
    let loadError = null;
    
    // Initialize WASM
    async function initWASM() {
        try {
            // Create Go instance
            const go = new Go();
            
            // Load and instantiate WASM module
            const result = await WebAssembly.instantiateStreaming(
                fetch('wasm/grokker.wasm'),
                go.importObject
            );
            
            // Run the Go program
            go.run(result.instance);
            
            console.log('Grokker WASM loaded successfully');
            wasmReady = true;
            
            // Call any queued callbacks
            wasmReadyCallbacks.forEach(callback => callback());
            wasmReadyCallbacks = [];
            
        } catch (err) {
            console.error('Failed to load Grokker WASM:', err);
            loadError = err;
            
            // Call callbacks with error
            wasmReadyCallbacks.forEach(callback => callback(err));
            wasmReadyCallbacks = [];
            
            throw err;
        }
    }
    
    // Public API: Wait for WASM to be ready
    window.Grokker.ready = function(callback) {
        if (wasmReady) {
            callback();
        } else if (loadError) {
            callback(loadError);
        } else {
            wasmReadyCallbacks.push(callback);
        }
    };
    
    // Public API: Check if WASM is ready
    window.Grokker.isReady = function() {
        return wasmReady;
    };
    
    // Public API: Generate commit message
    window.Grokker.generateCommitMessage = async function(params) {
        if (!wasmReady) {
            throw new Error('Grokker WASM not loaded. Call Grokker.ready() first.');
        }
        
        // Call the WASM function (exposed from Go)
        return await generateCommitMessage(params);
    };
    
    // Auto-initialize
    initWASM();
})();
```

---

## Integration with githubCommitDialog.js

### Changes Required

**File:** `src/ui/githubCommitDialog.js`

#### Change 1: Add WASM Loading Check

**Location:** In `populateSettings()` method

**BEFORE:**
```javascript
//  Disable AI checkbox if Grokker API key is not configured
if (!settings.grokkerApiKey) {
  if (aiCheckbox) {
    aiCheckbox.disabled = true;
    const aiStatusEl = document.getElementById('ai-status');
    if (aiStatusEl) {
      aiStatusEl.className = 'status-message status-warning';
      aiStatusEl.textContent = 'Grokker API key not configured. Please configure in GitHub Settings.';
    }
  }
}
```

**AFTER:**
```javascript
//  Disable AI checkbox if Grokker API key is not configured
if (!settings.grokkerApiKey) {
  if (aiCheckbox) {
    aiCheckbox.disabled = true;
    const aiStatusEl = document.getElementById('ai-status');
    if (aiStatusEl) {
      aiStatusEl.className = 'status-message status-warning';
      aiStatusEl.textContent = 'Grokker API key not configured. Please configure in GitHub Settings.';
    }
  }
}

// Check if Grokker WASM is loaded
if (!window.Grokker || !window.Grokker.isReady()) {
  if (aiCheckbox) {
    aiCheckbox.disabled = true;
    const aiStatusEl = document.getElementById('ai-status');
    if (aiStatusEl) {
      aiStatusEl.className = 'status-message status-warning';
      aiStatusEl.textContent = 'Grokker WASM not loaded. Please reload the page.';
    }
  }
}
```

#### Change 2: Replace executeGrokCommand()

**Location:** Replace entire `executeGrokCommand()` method

**BEFORE:**
```javascript
async executeGrokCommand() {
  if (this.executingCommand) {
    throw new Error('Command already executing');
  }
  
  this.executingCommand = true;
  
  try {
    const aiStatusEl = document.getElementById('ai-status');
    
    if (aiStatusEl) {
      aiStatusEl.textContent = 'Executing grok command...';
    }
    
    // Simulate the execution of the grok command
    // ... rest of simulated code ...
  } catch (error) {
    console.error('Error executing grok command:', error);
    throw new Error(`Grok command failed: ${error.message}`);
  } finally {
    this.executingCommand = false;
  }
}
```

**AFTER:**
```javascript
async executeGrokCommand() {
  if (this.executingCommand) {
    throw new Error('Command already executing');
  }
  
  this.executingCommand = true;
  
  try {
    const aiStatusEl = document.getElementById('ai-status');
    
    if (aiStatusEl) {
      aiStatusEl.textContent = 'Generating commit message with AI...';
    }
    
    // Get file path
    const pathInput = document.getElementById('commit-path');
    const filePath = pathInput.value.trim() || 'document.md';
    const filename = filePath.split('/').pop();
    
    // Get old content from GitHub
    const repoSelect = document.getElementById('commit-repo');
    const selectedRepo = repoSelect.value;
    
    if (!selectedRepo) {
      throw new Error('Please select a repository first');
    }
    
    let oldContent = '';
    try {
      // Fetch current file content from GitHub
      oldContent = await githubService.getFileContent(selectedRepo, filePath);
    } catch (error) {
      // File might not exist yet (new file)
      console.log('File does not exist on GitHub yet (new file)', error);
      oldContent = '';
    }
    
    const newContent = this.documentContent;
    
    // Check file size
    const diffSize = oldContent.length + newContent.length;
    if (diffSize > 50 * 1024) {
      throw new Error('File too large for AI analysis (max 50KB combined)');
    }
    
    if (diffSize > 10 * 1024 && aiStatusEl) {
      aiStatusEl.textContent = 'Large file - generation may take longer...';
    }
    
    // Generate diff using 'diff' library
    const Diff = window.Diff;
    if (!Diff) {
      throw new Error('Diff library not loaded');
    }
    
    const patch = Diff.createPatch(
      filename,
      oldContent,
      newContent,
      'old version',
      'new version'
    );
    
    // Call Grokker WASM
    const result = await window.Grokker.generateCommitMessage({
      diff: patch,
      filename: filename,
      apiKey: githubService.settings.grokkerApiKey,
      model: githubService.settings.grokkerModel || 'gpt-4'
    });
    
    return result.fullMessage;
    
  } catch (error) {
    console.error('Error generating commit message:', error);
    throw new Error(`Failed to generate commit message: ${error.message}`);
  } finally {
    this.executingCommand = false;
  }
}
```

#### Change 3: Add getFileContent to githubService

**File:** `src/github/githubService.js` (or wherever githubService is defined)

**ADD THIS METHOD:**
```javascript
/**
 * Get file content from GitHub
 * @param {string} repo - Repository full name (owner/repo)
 * @param {string} path - File path
 * @returns {Promise<string>} File content
 */
async getFileContent(repo, path) {
  const [owner, repoName] = repo.split('/');
  
  const url = `https://api.github.com/repos/${owner}/${repoName}/contents/${path}`;
  
  const response = await fetch(url, {
    headers: {
      'Authorization': `token ${this.settings.token}`,
      'Accept': 'application/vnd.github.v3+json'
    }
  });
  
  if (!response.ok) {
    throw new Error(`GitHub API error: ${response.statusText}`);
  }
  
  const data = await response.json();
  
  // Decode base64 content
  return atob(data.content);
}
```

### Changes to index.html

**File:** `src/index.html` or main HTML file

**ADD BEFORE CLOSING `</body>` TAG:**

```html
<!-- Diff library -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/diff/5.1.0/diff.min.js"></script>

<!-- Grokker WASM -->
<script src="wasm/wasm_exec.js"></script>
<script src="wasm/grokker-loader.js"></script>
```

### Changes to package.json (optional)

If you want to use npm instead of CDN:

```json
{
  "dependencies": {
    "diff": "^5.1.0"
  }
}
```

---

## Build Process

### Development Build Script

**File:** `build-dev.sh`

```bash
#!/bin/bash
set -e

echo "Building Grokker WASM (development)..."

# Build WASM binary
GOOS=js GOARCH=wasm go build -o dist/grokker.wasm v3/wasm/main.go

# Copy Go WASM runtime
cp "$(go env GOROOT)/misc/wasm/wasm_exec.js" dist/

echo "Development build complete"
ls -lh dist/
```

### Production Build Script

**File:** `build-prod.sh`

```bash
#!/bin/bash
set -e

echo "Building Grokker WASM (production)..."

# Clean dist directory
rm -rf dist
mkdir -p dist

# Build optimized WASM binary
GOOS=js GOARCH=wasm go build \
    -ldflags="-s -w" \
    -o dist/grokker.wasm \
    v3/wasm/main.go

# Copy Go WASM runtime
cp "$(go env GOROOT)/misc/wasm/wasm_exec.js" dist/

# Compress
echo "Compressing..."
gzip -9 -k dist/grokker.wasm

echo "Production build complete"
echo ""
echo "File sizes:"
echo "  Uncompressed: $(du -h dist/grokker.wasm | cut -f1)"
echo "  Gzip:         $(du -h dist/grokker.wasm.gz | cut -f1)"
```

### Makefile

```makefile
.PHONY: build-dev build-prod clean test serve

build-dev:
	./build-dev.sh

build-prod:
	./build-prod.sh

clean:
	rm -rf dist/
	mkdir -p dist/

test:
	cd v3/wasm && go test -v ./...

serve:
	cd dist && python3 -m http.server 8080
```

---

## Testing

### Manual Testing Steps

1. **Build WASM module:**
   ```bash
   make build-dev
   ```

2. **Copy files to collab-editor:**
   ```bash
   cp dist/grokker.wasm ../collab-editor/public/wasm/
   cp dist/wasm_exec.js ../collab-editor/public/wasm/
   cp dist/grokker-loader.js ../collab-editor/public/wasm/
   ```

3. **Start collab-editor:**
   ```bash
   cd ../collab-editor
   npm start
   ```

4. **Test in browser:**
   - Open browser console
   - Check for: "Grokker WASM loaded successfully"
   - Edit a document
   - Click "Commit to GitHub"
   - Check "Create commit message using AI"
   - Click "Commit to GitHub"
   - Verify commit message appears

### Test Cases

| Test Case | Expected Result |
|-----------|----------------|
| WASM loads on page load | Console shows "Grokker WASM loaded successfully" |
| API key not configured | Checkbox disabled, warning shown |
| API key configured | Checkbox enabled, ready to use |
| Generate message for small file (<10KB) | Success within 5 seconds |
| Generate message for medium file (10-50KB) | Warning shown, success within 10 seconds |
| Generate message for large file (>50KB) | Error: "File too large" |
| Invalid API key | Error: "OpenAI API error" |
| Network offline | Error: "Network error" |
| OpenAI API rate limit | Error with retry suggestion |

---

## Deployment

### File Structure in collab-editor

```
collab-editor/
├── public/
│   └── wasm/
│       ├── grokker.wasm          # or grokker.wasm.gz
│       ├── wasm_exec.js
│       └── grokker-loader.js
└── src/
    ├── index.html                # Modified to load WASM
    ├── ui/
    │   └── githubCommitDialog.js # Modified with integration
    └── github/
        └── githubService.js      # Modified with getFileContent()
```

### MIME Type Configuration

Ensure your web server serves WASM with correct MIME type:

**Apache (.htaccess):**
```apache
AddType application/wasm .wasm
```

**Nginx:**
```nginx
types {
    application/wasm wasm;
}
```

**Node.js/Express:**
```javascript
app.use((req, res, next) => {
    if (req.url.endsWith('.wasm')) {
        res.type('application/wasm');
    }
    next();
});
```

---

## Summary

### What You're Building

A standalone WASM module that:
- Takes a unified diff + API key
- Calls OpenAI to generate commit message
- Returns formatted conventional commit
- Integrates cleanly with any web app

### Key Design Decisions

1. **JavaScript generates diff** - simpler WASM code
2. **OpenAI only** - no gateway complexity
3. **File size limits** - 10KB warning, 50KB hard stop
4. **Generic API** - not tied to specific editor
5. **Clear error handling** - good UX for failures

### Files You Need to Create

From scratch:
1. `v3/wasm/main.go`
2. `dist/grokker-loader.js`
3. `build-dev.sh`
4. `build-prod.sh`
5. `Makefile`

### Files You Need to Modify

In collab-editor:
1. `src/ui/githubCommitDialog.js` - 3 changes
2. `src/github/githubService.js` - add getFileContent()
3. `src/index.html` - add script tags

### Total Line Count

- New Go code: ~250 lines
- New JavaScript: ~100 lines
- Modified JavaScript: ~80 lines changed

---

## Next Steps

1. Create `grokker-wasm` project structure
2. Copy files from Grokker
3. Create `v3/wasm/main.go`
4. Build and test WASM module
5. Integrate with collab-editor
6. Test end-to-end
7. Deploy

---

*End of Specification*

# Grokker WASM Commit Message Generator Specification

**Version:** 1.0  
**Date:** 2025  
**Status:** Draft

---

## Table of Contents

1. [Introduction](#introduction)
2. [Goals](#goals)
3. [Architecture](#architecture)
4. [Requirements](#requirements)
5. [WASM Interface](#wasm-interface)
6. [Go Code to Compile](#go-code-to-compile)
7. [Implementation Details](#implementation-details)
8. [Browser Integration](#browser-integration)
9. [Security Considerations](#security-considerations)
10. [Deployment](#deployment)
11. [Testing Strategy](#testing-strategy)
12. [Future Enhancements](#future-enhancements)

---

## 1. Introduction

This document specifies the requirements and design for a WebAssembly (WASM) build of Grokker's commit message generation functionality. The WASM module will run entirely in the browser, providing AI-powered commit message generation for single-file changes in a collaborative editor environment.

### 1.1 Purpose

Build a WASM wrapper for Grokker's `GitCommitMessage()` functionality that enables:
- Privacy-focused, client-side commit message generation
- Zero hosting costs (no server infrastructure)
- Seamless integration with `githubCommitDialog.js`
- Full Grokker semantic analysis and formatting capabilities

### 1.2 Use Case

**Target Application:** Single-file collaborative editor with GitHub integration

**Workflow:**
1. User makes changes to a single file in the browser
2. User initiates commit via `githubCommitDialog.js`
3. WASM module analyzes the diff using user's OpenAI API key
4. WASM returns formatted conventional commit message
5. User reviews/edits message before committing to GitHub

---

## 2. Goals

### 2.1 Privacy
- All processing happens client-side
- Code never leaves the browser (except for OpenAI API calls)
- No intermediate servers or proxies
- User maintains full control of their API key

### 2.2 No Hosting Costs
- No server infrastructure required
- No backend maintenance
- Static file serving only
- Leverage user's own OpenAI API key

### 2.3 Full Grokker Capabilities
- Use Grokker's existing semantic analysis
- Maintain Grokker's commit message formatting
- Support Grokker's prompt engineering
- Leverage proven GitCommitMessage implementation

### 2.4 Seamless Integration
- Drop-in solution for `githubCommitDialog.js`
- Simple async JavaScript API
- Minimal changes to existing workflow
- Progressive enhancement approach

---

## 3. Architecture

### 3.1 Component Diagram

```
┌─────────────────────────────────────┐
│   githubCommitDialog.js (Browser)  │
│   - Captures file changes           │
│   - Provides UI                     │
│   - User's OpenAI API key           │
└──────────────┬──────────────────────┘
               │
               │ (diff text, API key, model)
               ▼
┌─────────────────────────────────────┐
│     Grokker WASM Module             │
│   - Semantic analysis               │
│   - Prompt engineering              │
│   - Message formatting              │
└──────────────┬──────────────────────┘
               │
               │ (uses API key)
               ▼
┌─────────────────────────────────────┐
│         OpenAI API                  │
│   - GPT-4, GPT-3.5-turbo, etc.     │
│   - Returns analysis                │
└──────────────┬──────────────────────┘
               │
               │ (formatted message)
               ▼
┌─────────────────────────────────────┐
│   githubCommitDialog.js             │
│   - Displays message                │
│   - User edits if desired           │
│   - Commits to GitHub               │
└─────────────────────────────────────┘
```

### 3.2 Data Flow

**Input (Browser → WASM):**
- Old file content OR unified diff
- New file content (if not using diff)
- OpenAI API key
- Model name (e.g., "gpt-4", "o3-mini")

**Processing (WASM):**
- Analyze semantic changes
- Apply Grokker's commit message prompt
- Format according to conventional commits

**Output (WASM → Browser):**
- Title (summary line, ≤60 chars)
- Body (detailed bullet points)
- Full formatted message

---

## 4. Requirements

### 4.1 Functional Requirements

#### FR-1: Single File Diff Analysis
- **Priority:** Must Have
- **Description:** Accept old file content and new file content (or unified diff)
- **Details:**
  - Support both full file content pairs and unified diff format
  - Analyze semantic changes between versions
  - No multi-file support needed in v1

#### FR-2: Commit Message Generation
- **Priority:** Must Have
- **Description:** Generate conventional commit format messages
- **Details:**
  - Format: `type: subject` (title) + detailed body
  - Use Grokker's existing prompt from `v3/core/git.go`
  - Include semantic understanding of changes
  - Support bullet-pointed details

#### FR-3: Model Support
- **Priority:** Must Have
- **Description:** Support OpenAI models via Grokker's model gateway
- **Details:**
  - Support GPT-4, GPT-3.5-turbo, o1-mini, o3-mini
  - Use Grokker's existing model definitions from `v3/core/model.go`
  - Allow model selection via configuration

#### FR-4: User Workflow
- **Priority:** Must Have
- **Description:** Simple async API for browser integration
- **Details:**
  - Browser calls WASM with diff and API key
  - WASM returns formatted commit message
  - User can edit message before committing
  - Non-blocking execution

### 4.2 Non-Functional Requirements

#### NFR-1: Performance
- **Priority:** Must Have
- **Metrics:**
  - WASM module loads in < 2 seconds
  - Commit message generation in < 5 seconds (network dependent)
  - No blocking of browser UI
  - Memory usage < 50MB

#### NFR-2: Security
- **Priority:** Must Have
- **Details:**
  - API key stays in browser memory only
  - No persistent storage of sensitive data
  - All communication over HTTPS
  - No logging of API keys
  - CORS compliance for OpenAI API

#### NFR-3: Compatibility
- **Priority:** Must Have
- **Details:**
  - Modern browsers with WASM support (Chrome 57+, Firefox 52+, Safari 11+, Edge 16+)
  - No Node.js or build tools required for end users
  - ES6+ JavaScript compatibility

#### NFR-4: Size
- **Priority:** Should Have
- **Metrics:**
  - WASM bundle < 5MB (compressed)
  - Single file deployment
  - Browser caching supported (immutable)
  - gzip compression applied

---

## 5. WASM Interface

### 5.1 JavaScript API

The WASM module exposes a single async function to JavaScript:

```javascript
/**
 * Generate a commit message from a file diff
 * @param {Object} params - Parameters object
 * @param {string} params.diff - Unified diff text or content description
 * @param {string} params.apiKey - OpenAI API key
 * @param {string} params.model - Model name (e.g., "gpt-4", "gpt-3.5-turbo")
 * @param {string} [params.oldContent] - Optional: old file content (if not using diff)
 * @param {string} [params.newContent] - Optional: new file content (if not using diff)
 * @returns {Promise<CommitMessage>} Generated commit message
 */
async function generateCommitMessage(params)
```

### 5.2 Input Format

```javascript
{
  // Option 1: Provide unified diff
  "diff": "diff --git a/file.js b/file.js\nindex 1234567..89abcdef 100644\n--- a/file.js\n+++ b/file.js\n...",
  
  // Option 2: Provide old and new content
  "oldContent": "const x = 1;",
  "newContent": "const x = 2;",
  
  // Required fields
  "apiKey": "sk-...",   // OpenAI API key
  "model": "gpt-4"      // Model name
}
```

**Field Validation:**
- Either `diff` OR both `oldContent` and `newContent` must be provided
- `apiKey` is required (validated format: starts with "sk-")
- `model` is required (validated against Grokker's model list)
- Maximum diff size: 100KB (to prevent token limit issues)

### 5.3 Output Format

```javascript
{
  "title": "feat: add user authentication",
  "body": "- Implement JWT token validation\n- Add login endpoint\n- Add middleware for protected routes",
  "fullMessage": "feat: add user authentication\n\n- Implement JWT token validation\n- Add login endpoint\n- Add middleware for protected routes"
}
```

**Field Descriptions:**
- `title`: Summary line (≤60 characters), conventional commit format
- `body`: Detailed explanation with bullet points
- `fullMessage`: Complete commit message (title + blank line + body)

### 5.4 Error Format

```javascript
{
  "error": "Error message",
  "code": "ERROR_CODE",
  "details": "Additional error details"
}
```

**Error Codes:**

| Code | Description | User Action |
|------|-------------|-------------|
| `INVALID_INPUT` | Missing or invalid parameters | Check input format |
| `OPENAI_API_ERROR` | OpenAI API call failed | Verify API key, check network |
| `PARSE_ERROR` | Failed to parse diff | Verify diff format |
| `TOKEN_LIMIT_ERROR` | Diff exceeds token limit | Reduce diff size |
| `NETWORK_ERROR` | Network connection failed | Check internet connection |
| `UNKNOWN_ERROR` | Unexpected error | Check console for details |

### 5.5 Promise Rejection Handling

The function always returns a Promise that:
- **Resolves** with a CommitMessage object on success
- **Rejects** with an Error object containing code and details

Example error handling:
```javascript
try {
  const result = await generateCommitMessage(params);
  console.log(result.fullMessage);
} catch (error) {
  console.error(`[${error.code}] ${error.message}`);
  if (error.details) console.error(error.details);
}
```

---

## 6. Go Code to Compile

### 6.1 Core Packages Required

From the Grokker v3 codebase, include the following packages and their dependencies:

#### 6.1.1 v3/core (Partial)

**Files to Include:**
- `api.go` - Contains `GitCommitMessage()` function
- `gateway.go` - Contains `CompleteChat()`, `gateway()` functions
- `model.go` - Model definitions and routing
- `git.go` - Commit message prompts and formatting

**Dependencies from core:**
- Basic types: `FileLang`, `ChatMsg`
- Constants: `RoleUser`, `RoleSystem`, `RoleAI`

#### 6.1.2 v3/openai

**Files to Include:**
- `openai.go` - OpenAI API client

**Key Function:**
```go
func CompleteChat(upstreamName string, inmsgs []client.ChatMsg) (results client.Results, err error)
```

#### 6.1.3 v3/client

**Files to Include:**
- `chatclient.go` - ChatClient interface and types

**Key Types:**
```go
type ChatClient interface {
    CompleteChat(model, sysmsg string, messages []ChatMsg) (string, error)
}

type ChatMsg struct {
    Role    string
    Content string
}

type Results struct {
    Body      string
    Citations []string
}
```

#### 6.1.4 v3/util

**Files to Include:**
- Utility functions as needed (likely minimal for WASM version)

### 6.2 Excluded Packages

The following Grokker functionality is **NOT** needed for WASM:

- ❌ `embeddings-openai.go` - Embeddings API
- ❌ `document.go` - Document management
- ❌ `chunk.go` - Chunk management
- ❌ Database operations (`.grok` file handling)
- ❌ `migrate.go` - Migration logic
- ❌ `v3/cli/*` - CLI code
- ❌ `v3/aidda/*` - AIDDA functionality
- ❌ `chat.go` - Chat history management (file-based)
- ❌ Context retrieval and RAG features

### 6.3 New WASM Entry Point

Create: `v3/wasm/main.go`

```go
//go:build js && wasm

package main

import (
    "encoding/json"
    "syscall/js"
    
    "github.com/stevegt/grokker/v3/core"
    "github.com/stevegt/grokker/v3/client"
)

func main() {
    c := make(chan struct{})
    
    // Register the JavaScript function
    js.Global().Set("generateCommitMessage", js.FuncOf(generateCommitMessage))
    
    println("Grokker WASM initialized")
    <-c
}

func generateCommitMessage(this js.Value, args []js.Value) interface{} {
    // Implementation details in section 7.2
}
```

### 6.4 Dependency Management

**External Dependencies Required:**
- `github.com/stevegt/go-openai` - OpenAI API client
- Standard library packages:
  - `encoding/json`
  - `syscall/js`
  - `context`
  - `net/http` (works in WASM via browser Fetch API)
  - `os` (limited functionality in WASM)

**Build Tags:**
- Use `//go:build js && wasm` for WASM-specific code
- Conditional compilation for non-WASM dependencies

---

## 7. Implementation Details

### 7.1 WASM Compilation

#### 7.1.1 Basic Build Command

```bash
GOOS=js GOARCH=wasm go build -o grokker.wasm v3/wasm/main.go
```

#### 7.1.2 Optimized Build

```bash
GOOS=js GOARCH=wasm go build -ldflags="-s -w" -o grokker.wasm v3/wasm/main.go
```

**Optimization Flags:**
- `-s` - Omit symbol table and debug information
- `-w` - Omit DWARF symbol table

**Expected Size:**
- Uncompressed: 3-4 MB
- gzip compressed: 800KB - 1.2 MB

#### 7.1.3 Compression

```bash
gzip -9 grokker.wasm  # Results in grokker.wasm.gz
```

**Deployment Options:**
1. Serve pre-compressed `.wasm.gz` with `Content-Encoding: gzip`
2. Let web server handle compression on-the-fly
3. Use Brotli for better compression (`.wasm.br`)

### 7.2 JavaScript/Go Bridge Implementation

#### 7.2.1 Entry Point Pattern

```go
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
            oldContent := getStringParam(params, "oldContent")
            newContent := getStringParam(params, "newContent")
            apiKey := getStringParam(params, "apiKey")
            model := getStringParam(params, "model")
            
            // Validate inputs
            if err := validateInputs(diff, oldContent, newContent, apiKey, model); err != nil {
                rejectWithError(reject, "INVALID_INPUT", err.Error(), "")
                return
            }
            
            // Create diff if not provided
            var finalDiff string
            if diff != "" {
                finalDiff = diff
            } else {
                finalDiff = createSimpleDiff(oldContent, newContent)
            }
            
            // Call Grokker's GitCommitMessage
            result, err := callGrokker(finalDiff, apiKey, model)
            if err != nil {
                rejectWithError(reject, "OPENAI_API_ERROR", "Failed to generate commit message", err.Error())
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
```

#### 7.2.2 Helper Functions

```go
func getStringParam(obj js.Value, key string) string {
    val := obj.Get(key)
    if val.IsUndefined() || val.IsNull() {
        return ""
    }
    return val.String()
}

func validateInputs(diff, oldContent, newContent, apiKey, model string) error {
    // Must have either diff OR both old/new content
    hasDiff := diff != ""
    hasContent := oldContent != "" && newContent != ""
    
    if !hasDiff && !hasContent {
        return fmt.Errorf("must provide either 'diff' or both 'oldContent' and 'newContent'")
    }
    
    if apiKey == "" {
        return fmt.Errorf("apiKey is required")
    }
    
    if !strings.HasPrefix(apiKey, "sk-") {
        return fmt.Errorf("invalid API key format")
    }
    
    if model == "" {
        return fmt.Errorf("model is required")
    }
    
    return nil
}

func createSimpleDiff(oldContent, newContent string) string {
    return fmt.Sprintf("Old content:\n%s\n\nNew content:\n%s", oldContent, newContent)
}

func rejectWithError(reject js.Value, code, message, details string) {
    errorObj := map[string]interface{}{
        "error":   message,
        "code":    code,
        "details": details,
    }
    reject.Invoke(mapToJSObject(errorObj))
}

func resultToJSObject(result *CommitResult) js.Value {
    obj := map[string]interface{}{
        "title":       result.Title,
        "body":        result.Body,
        "fullMessage": result.FullMessage,
    }
    return mapToJSObject(obj)
}

func mapToJSObject(m map[string]interface{}) js.Value {
    obj := js.Global().Get("Object").New()
    for k, v := range m {
        obj.Set(k, v)
    }
    return obj
}
```

#### 7.2.3 Core Grokker Integration

```go
type CommitResult struct {
    Title       string
    Body        string
    FullMessage string
}

func callGrokker(diff, apiKey, model string) (*CommitResult, error) {
    // Set API key in environment (WASM context)
    os.Setenv("OPENAI_API_KEY", apiKey)
    
    // Create minimal Grokker-compatible context
    // Note: We don't need full Grokker initialization for commit messages
    
    // Build the system message (from v3/core/git.go)
    sysmsg := `Write a git commit message for the given diff. Use present tense, active, 
imperative statements as if giving directions. Do not use extra adjectives 
or marketing hype. The first line of the commit message must be a summary 
of 60 characters or less, followed by a blank line, followed by 
bullet-pointed details. Make a separate bullet list for each changed file.`
    
    // Create user message with diff
    userMsg := fmt.Sprintf("%s\n\n%s", sysmsg, diff)
    
    msgs := []client.ChatMsg{
        {
            Role:    "User",
            Content: userMsg,
        },
    }
    
    // Call OpenAI via Grokker's gateway
    results, err := openai.CompleteChat(model, msgs)
    if err != nil {
        return nil, err
    }
    
    // Parse the response
    fullMessage := results.Body
    lines := strings.Split(fullMessage, "\n")
    
    // Extract title (first non-empty line)
    title := ""
    bodyStartIdx := 0
    for i, line := range lines {
        trimmed := strings.TrimSpace(line)
        if trimmed != "" {
            title = trimmed
            bodyStartIdx = i + 1
            break
        }
    }
    
    // Extract body (skip blank line after title)
    bodyLines := []string{}
    for i := bodyStartIdx; i < len(lines); i++ {
        trimmed := strings.TrimSpace(lines[i])
        if trimmed != "" {
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
```

### 7.3 HTTP Client Handling

**Key Point:** Go's `net/http` package works in WASM by using the browser's Fetch API transparently.

The existing OpenAI client code from `v3/openai/openai.go` should work without modification:

```go
authtoken := os.Getenv("OPENAI_API_KEY")
client := gptLib.NewClient(authtoken)
res, err := client.CreateChatCompletion(
    context.Background(),
    gptLib.ChatCompletionRequest{
        Model:    upstreamName,
        Messages: omsgs,
    },
)
```

**WASM Considerations:**
- HTTP requests are automatically routed through `fetch()` API
- CORS headers must be present on OpenAI API (already supported)
- No special networking code needed
- Timeouts work as expected

### 7.4 Error Handling

#### 7.4.1 Error Categories

```go
const (
    ErrInvalidInput   = "INVALID_INPUT"
    ErrOpenAIAPI      = "OPENAI_API_ERROR"
    ErrParseError     = "PARSE_ERROR"
    ErrTokenLimit     = "TOKEN_LIMIT_ERROR"
    ErrNetwork        = "NETWORK_ERROR"
    ErrUnknown        = "UNKNOWN_ERROR"
)
```

#### 7.4.2 Error Handling Pattern

```go
func handleError(reject js.Value, err error) {
    code := ErrUnknown
    details := err.Error()
    message := "An unexpected error occurred"
    
    // Classify error
    switch {
    case strings.Contains(err.Error(), "invalid"):
        code = ErrInvalidInput
        message = "Invalid input parameters"
    case strings.Contains(err.Error(), "token"):
        code = ErrTokenLimit
        message = "Diff exceeds token limit"
    case strings.Contains(err.Error(), "network"):
        code = ErrNetwork
        message = "Network connection failed"
    default:
        // Check if it's an OpenAI API error
        if strings.Contains(err.Error(), "API") {
            code = ErrOpenAIAPI
            message = "OpenAI API request failed"
        }
    }
    
    rejectWithError(reject, code, message, details)
}
```

---

## 8. Browser Integration

### 8.1 Loading the WASM Module

#### 8.1.1 HTML Setup

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Grokker Commit Message Generator</title>
</head>
<body>
    <!-- Load Go's WASM support file -->
    <script src="wasm_exec.js"></script>
    
    <!-- Load and initialize the WASM module -->
    <script src="grokker-loader.js"></script>
</body>
</html>
```

#### 8.1.2 Loader Script (grokker-loader.js)

```javascript
// grokker-loader.js
(function() {
    'use strict';
    
    // Create global namespace
    window.Grokker = window.Grokker || {};
    
    // Loading state
    let wasmReady = false;
    let wasmReadyCallbacks = [];
    
    // Initialize WASM
    async function initWASM() {
        // Create Go instance
        const go = new Go();
        
        try {
            // Load and instantiate WASM module
            const result = await WebAssembly.instantiateStreaming(
                fetch('grokker.wasm'),
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
            throw err;
        }
    }
    
    // Public API
    window.Grokker.ready = function(callback) {
        if (wasmReady) {
            callback();
        } else {
            wasmReadyCallbacks.push(callback);
        }
    };
    
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

### 8.2 Integration with githubCommitDialog.js

```javascript
// In githubCommitDialog.js

async function getAICommitMessage(oldContent, newContent, apiKey) {
    try {
        // Wait for WASM to be ready
        await new Promise((resolve) => {
            Grokker.ready(resolve);
        });
        
        // Generate diff (or use a diff library like diff-match-patch)
        const diff = generateDiff(oldContent, newContent);
        
        // Call WASM function
        const result = await Grokker.generateCommitMessage({
            diff: diff,
            apiKey: apiKey,
            model: "gpt-4"  // or user preference
        });
        
        // Display to user
        displayCommitMessage(result.title, result.body);
        
        return result;
        
    } catch (error) {
        console.error('Error generating commit message:', error);
        
        // Show user-friendly error message
        const errorMessage = getErrorMessage(error.code);
        displayError(errorMessage);
        
        throw error;
    }
}

function getErrorMessage(code) {
    const messages = {
        'INVALID_INPUT': 'Invalid input. Please check your diff and try again.',
        'OPENAI_API_ERROR': 'Failed to connect to OpenAI. Please check your API key.',
        'TOKEN_LIMIT_ERROR': 'The diff is too large. Try with a smaller change.',
        'NETWORK_ERROR': 'Network error. Please check your internet connection.',
        'UNKNOWN_ERROR': 'An unexpected error occurred. Please try again.'
    };
    return messages[code] || messages['UNKNOWN_ERROR'];
}

function displayCommitMessage(title, body) {
    // Populate UI fields
    document.getElementById('commit-title').value = title;
    document.getElementById('commit-body').value = body;
    
    // Enable commit button
    enableCommitButton();
}

function displayError(message) {
    // Show error in UI
    const errorElement = document.getElementById('error-message');
    errorElement.textContent = message;
    errorElement.style.display = 'block';
    
    // Hide after 5 seconds
    setTimeout(() => {
        errorElement.style.display = 'none';
    }, 5000);
}
```

### 8.3 Progressive Enhancement

```javascript
// Feature detection
function supportsWASM() {
    try {
        if (typeof WebAssembly === "object" &&
            typeof WebAssembly.instantiate === "function") {
            const module = new WebAssembly.Module(
                Uint8Array.of(0x0, 0x61, 0x73, 0x6d, 0x01, 0x00, 0x00, 0x00)
            );
            if (module instanceof WebAssembly.Module) {
                return true;
            }
        }
    } catch (e) {}
    return false;
}

// Fallback handling
if (!supportsWASM()) {
    console.warn('WebAssembly not supported. Falling back to manual commit messages.');
    // Hide AI commit message button
    document.getElementById('ai-commit-btn').style.display = 'none';
}
```

---

## 9. Security Considerations

### 9.1 API Key Handling

#### 9.1.1 Storage Options

**❌ Never Use:**
- `localStorage` (persists across sessions, vulnerable to XSS)
- Cookies (can be stolen via XSS)
- URL parameters (visible in history, logs)

**✅ Recommended:**
```javascript
// Option 1: Session storage (cleared when tab closes)
sessionStorage.setItem('openai_api_key', apiKey);

// Option 2: Memory only (requires re-entry each session)
let apiKey = null;

function setAPIKey(key) {
    apiKey = key;
    // Clear after 1 hour of inactivity
    setTimeout(() => { apiKey = null; }, 3600000);
}
```

#### 9.1.2 Key Validation

```javascript
function validateAPIKey(key) {
    if (!key) {
        throw new Error('API key is required');
    }
    
    if (!key.startsWith('sk-')) {
        throw new Error('Invalid API key format');
    }
    
    if (key.length < 20) {
        throw new Error('API key too short');
    }
    
    return true;
}
```

#### 9.1.3 Transmission Security

- ✅ API key passed from JS to WASM in memory only
- ✅ WASM uses key for OpenAI API call over HTTPS
- ✅ Key never logged or exposed in console
- ✅ Key not included in error messages

### 9.2 Content Security Policy (CSP)

Required CSP headers for hosting page:

```html
<meta http-equiv="Content-Security-Policy" 
      content="
        default-src 'self';
        script-src 'self' 'wasm-unsafe-eval';
        connect-src 'self' https://api.openai.com;
        style-src 'self' 'unsafe-inline';
      ">
```

**Key Directives:**
- `script-src 'wasm-unsafe-eval'` - Required for WASM execution
- `connect-src https://api.openai.com` - Allow OpenAI API calls
- `default-src 'self'` - Restrict other resources to same origin

### 9.3 CORS Handling

**OpenAI API CORS Support:**
```
Access-Control-Allow-Origin: *
```

- ✅ OpenAI API supports CORS for browser requests
- ✅ No proxy needed for API calls
- ✅ Direct browser-to-OpenAI communication

### 9.4 Data Privacy

**Data Flow:**
1. User's code → Browser (stays local)
2. Diff generation → Browser (stays local)
3. API request → OpenAI (with user's key)
4. Response → Browser (stays local)

**Privacy Guarantees:**
- ✅ All data processing happens in browser
- ✅ No data sent to any server except OpenAI
- ✅ OpenAI's data usage policy applies
- ⚠️ Users should be informed about OpenAI API usage

**Recommended Privacy Notice:**
```html
<div class="privacy-notice">
    <p>⚠️ AI-generated commit messages use your OpenAI API key to send 
    diffs to OpenAI's API. Your code changes will be processed by OpenAI 
    according to their <a href="https://openai.com/privacy">privacy policy</a>.</p>
</div>
```

### 9.5 Input Validation

```go
func validateDiffSize(diff string) error {
    // Limit diff size to prevent token limit issues
    maxSize := 100 * 1024 // 100KB
    if len(diff) > maxSize {
        return fmt.Errorf("diff size (%d bytes) exceeds maximum (%d bytes)", 
            len(diff), maxSize)
    }
    return nil
}

func sanitizeInput(input string) string {
    // Remove potential script injections
    // Note: This is defense-in-depth; WASM provides isolation
    return html.EscapeString(input)
}
```

---

## 10. Deployment

### 10.1 File Structure

```
/wasm/
├── grokker.wasm         # Main WASM binary (or grokker.wasm.gz)
├── wasm_exec.js         # Go WASM runtime (from $GOROOT/misc/wasm/)
└── grokker-loader.js    # Convenience wrapper (optional)
```

### 10.2 MIME Type Configuration

**Required MIME Type:** `application/wasm`

#### Apache (.htaccess)

```apache
AddType application/wasm .wasm
```

#### Nginx (nginx.conf)

```nginx
types {
    application/wasm wasm;
}
```

#### Node.js / Express

```javascript
app.use((req, res, next) => {
    if (req.url.endsWith('.wasm')) {
        res.type('application/wasm');
    }
    next();
});
```

### 10.3 Caching Strategy

#### Aggressive Caching (Recommended)

```nginx
location ~* \.wasm$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

**Versioning Strategy:**
- Include version in filename: `grokker-v1.0.0.wasm`
- Update loader script to reference new version
- Old versions remain cached, no cache invalidation needed

#### Cache-Control Headers

```
Cache-Control: public, max-age=31536000, immutable
```

### 10.4 Compression

#### Pre-compression (Recommended)

```bash
# Create gzip version
gzip -9 -k grokker.wasm

# Create Brotli version (better compression)
brotli -9 -k grokker.wasm
```

#### Nginx Configuration

```nginx
location ~* \.wasm$ {
    # Serve pre-compressed files
    gzip_static on;
    brotli_static on;
    
    # Set correct content-type
    types {
        application/wasm wasm;
    }
    
    # Aggressive caching
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

### 10.5 CDN Deployment

**For Better Performance:**

```html
<script src="https://cdn.example.com/grokker/v1.0.0/wasm_exec.js"></script>
<script>
    const wasmUrl = 'https://cdn.example.com/grokker/v1.0.0/grokker.wasm';
</script>
```

**CDN Requirements:**
- Support for `application/wasm` MIME type
- Support for large files (3-5MB)
- HTTPS required
- CORS headers configured

**Popular CDN Options:**
- Cloudflare (free tier available)
- jsDelivr (free for open source)
- Netlify (free tier available)
- Vercel (free tier available)

---

## 11. Build Process

### 11.1 Development Build

```bash
#!/bin/bash
# build-dev.sh

set -e

echo "Building Grokker WASM (development)..."

# Navigate to WASM directory
cd v3/wasm

# Build WASM binary
GOOS=js GOARCH=wasm go build -o ../../dist/grokker.wasm .

# Copy Go WASM runtime
cp "$(go env GOROOT)/misc/wasm/wasm_exec.js" ../../dist/

echo "Development build complete"
echo "Files in dist/:"
ls -lh ../../dist/
```

### 11.2 Production Build

```bash
#!/bin/bash
# build-prod.sh

set -e

echo "Building Grokker WASM (production)..."

# Navigate to WASM directory
cd v3/wasm

# Build optimized WASM binary
GOOS=js GOARCH=wasm go build \
    -ldflags="-s -w" \
    -o ../../dist/grokker.wasm \
    .

# Copy Go WASM runtime
cp "$(go env GOROOT)/misc/wasm/wasm_exec.js" ../../dist/

# Compress
echo "Compressing..."
gzip -9 -k ../../dist/grokker.wasm
brotli -9 -k ../../dist/grokker.wasm

echo "Production build complete"
echo "Files in dist/:"
ls -lh ../../dist/

# Show sizes
echo ""
echo "File sizes:"
echo "  Uncompressed: $(du -h ../../dist/grokker.wasm | cut -f1)"
echo "  Gzip:         $(du -h ../../dist/grokker.wasm.gz | cut -f1)"
echo "  Brotli:       $(du -h ../../dist/grokker.wasm.br | cut -f1)"
```

### 11.3 Makefile

```makefile
# Makefile

.PHONY: build-dev build-prod clean test

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

### 11.4 CI/CD Integration

#### GitHub Actions Example

```yaml
# .github/workflows/build-wasm.yml

name: Build WASM

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Go
      uses: actions/setup-go@v4
      with:
        go-version: '1.22'
    
    - name: Build WASM
      run: make build-prod
    
    - name: Upload artifacts
      uses: actions/upload-artifact@v3
      with:
        name: grokker-wasm
        path: |
          dist/grokker.wasm
          dist/grokker.wasm.gz
          dist/grokker.wasm.br
          dist/wasm_exec.js
```

---

## 12. Testing Strategy

### 12.1 Unit Tests

#### 12.1.1 Go Unit Tests

```go
// v3/wasm/main_test.go

//go:build js && wasm

package main

import "testing"

func TestValidateInputs(t *testing.T) {
    tests := []struct {
        name        string
        diff        string
        oldContent  string
        newContent  string
        apiKey      string
        model       string
        expectError bool
    }{
        {
            name:        "valid diff",
            diff:        "diff --git a/file.js b/file.js\n...",
            apiKey:      "sk-test123456789",
            model:       "gpt-4",
            expectError: false,
        },
        {
            name:        "valid old/new content",
            oldContent:  "const x = 1;",
            newContent:  "const x = 2;",
            apiKey:      "sk-test123456789",
            model:       "gpt-4",
            expectError: false,
        },
        {
            name:        "missing both diff and content",
            apiKey:      "sk-test123456789",
            model:       "gpt-4",
            expectError: true,
        },
        {
            name:        "missing API key",
            diff:        "diff --git a/file.js b/file.js\n...",
            model:       "gpt-4",
            expectError: true,
        },
        {
            name:        "invalid API key format",
            diff:        "diff --git a/file.js b/file.js\n...",
            apiKey:      "invalid-key",
            model:       "gpt-4",
            expectError: true,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := validateInputs(tt.diff, tt.oldContent, tt.newContent, tt.apiKey, tt.model)
            if tt.expectError && err == nil {
                t.Error("expected error, got nil")
            }
            if !tt.expectError && err != nil {
                t.Errorf("unexpected error: %v", err)
            }
        })
    }
}

func TestCreateSimpleDiff(t *testing.T) {
    oldContent := "const x = 1;"
    newContent := "const x = 2;"
    
    diff := createSimpleDiff(oldContent, newContent)
    
    if !strings.Contains(diff, oldContent) {
        t.Error("diff doesn't contain old content")
    }
    if !strings.Contains(diff, newContent) {
        t.Error("diff doesn't contain new content")
    }
}
```

#### 12.1.2 JavaScript Unit Tests

```javascript
// tests/wasm.test.js

describe('Grokker WASM', () => {
    beforeAll(async () => {
        // Load WASM module
        await loadGrokkerWASM();
    });
    
    describe('generateCommitMessage', () => {
        test('generates message from diff', async () => {
            const result = await Grokker.generateCommitMessage({
                diff: 'diff --git a/test.js b/test.js\n...',
                apiKey: process.env.OPENAI_API_KEY,
                model: 'gpt-4'
            });
            
            expect(result).toHaveProperty('title');
            expect(result).toHaveProperty('body');
            expect(result).toHaveProperty('fullMessage');
            expect(result.title).toBeTruthy();
        });
        
        test('generates message from old/new content', async () => {
            const result = await Grokker.generateCommitMessage({
                oldContent: 'const x = 1;',
                newContent: 'const x = 2;',
                apiKey: process.env.OPENAI_API_KEY,
                model: 'gpt-4'
            });
            
            expect(result.title).toBeTruthy();
        });
        
        test('rejects with INVALID_INPUT for missing params', async () => {
            await expect(
                Grokker.generateCommitMessage({})
            ).rejects.toMatchObject({
                code: 'INVALID_INPUT'
            });
        });
        
        test('rejects with INVALID_INPUT for invalid API key', async () => {
            await expect(
                Grokker.generateCommitMessage({
                    diff: 'test diff',
                    apiKey: 'invalid',
                    model: 'gpt-4'
                })
            ).rejects.toMatchObject({
                code: 'INVALID_INPUT'
            });
        });
    });
});
```

### 12.2 Integration Tests

```javascript
// tests/integration.test.js

describe('Integration Tests', () => {
    test('full workflow: diff -> WASM -> display', async () => {
        // Simulate user workflow
        const oldContent = readFileSync('test/fixtures/old.js', 'utf8');
        const newContent = readFileSync('test/fixtures/new.js', 'utf8');
        
        // Generate diff
        const diff = createDiff(oldContent, newContent);
        
        // Call WASM
        const result = await Grokker.generateCommitMessage({
            diff: diff,
            apiKey: process.env.OPENAI_API_KEY,
            model: 'gpt-4'
        });
        
        // Verify result format
        expect(result.title).toMatch(/^(feat|fix|docs|style|refactor|test|chore):/);
        expect(result.title.length).toBeLessThanOrEqual(60);
        expect(result.body).toContain('-');
        expect(result.fullMessage).toContain('\n\n');
    });
});
```

### 12.3 Browser Compatibility Tests

**Test Matrix:**

| Browser | Version | WASM Support | Test Status |
|---------|---------|--------------|-------------|
| Chrome | 57+ | ✅ Yes | Required |
| Firefox | 52+ | ✅ Yes | Required |
| Safari | 11+ | ✅ Yes | Required |
| Edge | 16+ | ✅ Yes | Required |
| Opera | 44+ | ✅ Yes | Optional |
| Samsung Internet | 7+ | ✅ Yes | Optional |

**Testing Tools:**
- BrowserStack for cross-browser testing
- Playwright for automated testing
- Manual testing on actual devices

### 12.4 Performance Tests

```javascript
// tests/performance.test.js

describe('Performance Tests', () => {
    test('WASM loads in under 2 seconds', async () => {
        const startTime = Date.now();
        await loadGrokkerWASM();
        const loadTime = Date.now() - startTime;
        
        expect(loadTime).toBeLessThan(2000);
    });
    
    test('commit message generation completes in reasonable time', async () => {
        const startTime = Date.now();
        await Grokker.generateCommitMessage({
            diff: generateLargeDiff(),
            apiKey: process.env.OPENAI_API_KEY,
            model: 'gpt-3.5-turbo'
        });
        const executionTime = Date.now() - startTime;
        
        // Network dependent, but should be under 10 seconds
        expect(executionTime).toBeLessThan(10000);
    });
    
    test('memory usage remains reasonable', async () => {
        if (performance.memory) {
            const before = performance.memory.usedJSHeapSize;
            
            await Grokker.generateCommitMessage({
                diff: generateDiff(),
                apiKey: process.env.OPENAI_API_KEY,
                model: 'gpt-4'
            });
            
            const after = performance.memory.usedJSHeapSize;
            const increase = after - before;
            
            // Should not increase by more than 50MB
            expect(increase).toBeLessThan(50 * 1024 * 1024);
        }
    });
});
```

### 12.5 Error Handling Tests

```javascript
describe('Error Handling', () => {
    test('handles network errors gracefully', async () => {
        // Simulate network failure
        const originalFetch = global.fetch;
        global.fetch = jest.fn(() => Promise.reject(new Error('Network error')));
        
        await expect(
            Grokker.generateCommitMessage({
                diff: 'test diff',
                apiKey: process.env.OPENAI_API_KEY,
                model: 'gpt-4'
            })
        ).rejects.toMatchObject({
            code: 'NETWORK_ERROR'
        });
        
        global.fetch = originalFetch;
    });
    
    test('handles API errors gracefully', async () => {
        await expect(
            Grokker.generateCommitMessage({
                diff: 'test diff',
                apiKey: 'sk-invalid',
                model: 'gpt-4'
            })
        ).rejects.toMatchObject({
            code: 'OPENAI_API_ERROR'
        });
    });
});
```

---

## 13. Future Enhancements

### 13.1 Phase 2 Features

#### 13.1.1 Multi-file Support
**Description:** Support analyzing changes across multiple files in a single commit

**Requirements:**
- Accept array of file diffs
- Analyze cross-file changes
- Generate unified commit message
- Identify related changes across files

**Implementation Considerations:**
- Token limit management becomes critical
- Need intelligent file prioritization
- May require multiple API calls with synthesis

#### 13.1.2 Commit History Analysis
**Description:** Learn from previous commits to match team style

**Requirements:**
- Access to commit history via GitHub API
- Pattern recognition in commit messages
- Style matching (conventional commits vs. custom)
- Suggest similar past commits

**Implementation Considerations:**
- Requires GitHub API integration
- Privacy considerations (accessing repo history)
- Caching strategy for commit history
- Performance impact of additional API calls

#### 13.1.3 Custom Formatting
**Description:** Support user-defined commit message templates

**Requirements:**
- Template system for commit messages
- Organization-specific conventions
- Custom conventional commit types
- Variable substitution

**Example Template:**
```javascript
{
  template: "[{ticket}] {type}: {subject}\n\n{body}\n\nReviewed-by: {reviewer}",
  types: ["feature", "bugfix", "hotfix"],
  required: ["ticket", "type", "subject"]
}
```

### 13.2 Phase 3 Features

#### 13.2.1 Repository Context
**Description:** Understand project structure and conventions

**Requirements:**
- Download and analyze README
- Parse CONTRIBUTING.md guidelines
- Understand project conventions
- Suggest context-aware messages

**Implementation:**
- Fetch repo metadata from GitHub
- Parse markdown for conventions
- Cache repo context in browser
- Update context periodically

#### 13.2.2 Smart Suggestions
**Description:** Intelligent analysis of changes

**Features:**
- Detect breaking changes
- Suggest version bumps (semver)
- Flag potential issues
- Recommend additional changes
- Check commit message against conventions

**Example:**
```javascript
{
  message: "feat: add breaking API change",
  warnings: [
    "Detected breaking change - consider BREAKING CHANGE footer",
    "Consider bumping major version"
  ],
  suggestions: [
    "Add migration guide to commit body",
    "Update API documentation"
  ]
}
```

#### 13.2.3 Offline Mode
**Description:** Work without internet connection

**Requirements:**
- Cache previous responses
- Pattern-based suggestions
- Local model inference (TensorFlow.js)
- Sync when online

**Implementation Challenges:**
- Model size for browser
- Inference performance
- Cache management
- Fallback strategy

### 13.3 Additional Provider Support

#### 13.3.1 Anthropic Claude
**Description:** Support Claude models as alternative to OpenAI

**Requirements:**
- Add Claude API client
- Update model selection UI
- Handle Claude-specific formats
- Billing/quota management

#### 13.3.2 Local Models
**Description:** Support running models locally in browser

**Options:**
- ONNX Runtime Web
- TensorFlow.js
- Transformers.js
- WebGPU acceleration

**Challenges:**
- Model size (typically 100MB-1GB+)
- Browser performance
- Quantization requirements
- Memory constraints

### 13.4 Developer Experience Enhancements

#### 13.4.1 VS Code Extension
**Description:** Native VS Code integration

**Features:**
- Source Control panel integration
- Inline diff analysis
- One-click commit message generation
- Configurable shortcuts

#### 13.4.2 Git Hook Integration
**Description:** Integrate with Git commit workflow

**Features:**
- `prepare-commit-msg` hook
- Auto-generate on `git commit`
- Template population
- Interactive editing

#### 13.4.3 CLI Tool
**Description:** Command-line interface for terminal users

**Usage:**
```bash
grokker commit --staged
grokker commit --file changes.diff
grokker commit --files file1.js file2.js
```

---

## Appendix A: File Markers Discussion

### Current Implementation

Grokker currently uses code fences with file markers:

```
File: net.go
```go
// code here
```
EOF_net.go
```

### Alternative Approaches

#### A.1 XML File Delimiters

```xml
<File name="net.go" language="go">
// code here
</File>
```

**Pros:**
- Clearly defined boundaries
- Structured attributes
- Standard parsing available

**Cons:**
- Verbose syntax
- LLMs may reformat
- Extraction code changes needed

#### A.2 JSON Objects

```json
{
  "files": [
    {"name": "net.go", "language": "go", "content": "..."}
  ]
}
```

**Pros:**
- Ubiquitous format
- Easy parsing
- Clear structure

**Cons:**
- LLMs may explain rather than return raw JSON
- Formatting inconsistency
- Overkill for single files

#### A.3 Custom Delimiters

```
<<<START-FILE:net.go:go>>>
// code here
<<<END-FILE:net.go>>>
```

**Pros:**
- Unique tokens
- Low ambiguity
- Customizable

**Cons:**
- Non-standard
- Custom parsing needed
- Not widely recognized

### Recommendation for WASM Version

For the WASM version, we recommend **keeping the current approach** with code fences, as:

1. It's already implemented in Grokker
2. LLMs are trained on markdown code fences
3. Simple regex-based extraction works well
4. No need to change `core/chat.go` extraction logic

---

## Appendix B: System Message

The system message used for commit message generation (from `v3/core/git.go`):

```
Write a git commit message for the given diff. Use present tense, active, 
imperative statements as if giving directions. Do not use extra adjectives 
or marketing hype. The first line of the commit message must be a summary 
of 60 characters or less, followed by a blank line, followed by 
bullet-pointed details. Make a separate bullet list for each changed file.
```

This proven prompt should be used as-is in the WASM implementation.

---

## Appendix C: Glossary

**Terms:**

- **ASAT** - Automated Systems Administration Tool (from Grokker context)
- **AVM** - Application Virtual Machine
- **CORS** - Cross-Origin Resource Sharing
- **CSP** - Content Security Policy
- **LLM** - Large Language Model
- **RAG** - Retrieval Augmented Generation
- **TVM** - Turing Virtual Machine
- **UTM** - Universal Turing Machine
- **WASM** - WebAssembly
- **WASI** - WebAssembly System Interface

**Conventional Commit Types:**

- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation changes
- `style` - Code style changes (formatting, etc.)
- `refactor` - Code refactoring
- `test` - Test additions or modifications
- `chore` - Build process or auxiliary tool changes

---

## Appendix D: References

1. **Grokker Repository**: https://github.com/stevegt/grokker
2. **Go WASM**: https://github.com/golang/go/wiki/WebAssembly
3. **OpenAI API**: https://platform.openai.com/docs/api-reference
4. **Conventional Commits**: https://www.conventionalcommits.org/
5. **WebAssembly**: https://webassembly.org/
6. **syscall/js**: https://pkg.go.dev/syscall/js

---

## Document Control

**Version History:**

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2025 | Initial | Initial specification |

**Review Status:** Draft

**Approval Status:** Pending

---

*End of Specification*

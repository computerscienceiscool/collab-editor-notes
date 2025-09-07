# Real-time Markdown Preview Specification

## Overview
Enhance the current markdown preview to update automatically as users type, with performance optimization, synchronized scrolling, and collaborative real-time updates.

## Current State Analysis
- Manual preview updates via Ctrl+R or menu action
- Basic markdown-to-HTML conversion in MenuSystem
- Side-by-side split view layout exists
- No real-time updates or scroll synchronization

## Requirements

### Functional Requirements
1. **Auto-update**: Preview updates as user types (debounced)
2. **Scroll Sync**: Editor and preview scroll positions synchronized
3. **Performance**: Smooth typing experience even with large documents
4. **Collaborative**: All users see preview updates in real-time
5. **Cursor Mapping**: Highlight current paragraph/section being edited
6. **Error Handling**: Graceful handling of invalid markdown

### Technical Requirements
1. **Debouncing**: 300ms delay to avoid excessive updates during typing
2. **Incremental Rendering**: Only re-render changed sections when possible
3. **Memory Efficiency**: Clean up DOM nodes and event listeners
4. **Accessibility**: Screen reader support for preview updates
5. **Mobile Responsive**: Touch-friendly scroll synchronization

## Architecture Design

### MarkdownPreviewManager Class
```javascript
class MarkdownPreviewManager {
  constructor(editorView, previewElement) {
    this.editorView = editorView;
    this.previewElement = previewElement;
    this.isActive = false;
    this.updateTimer = null;
    this.scrollSyncEnabled = true;
    this.lastRenderedHash = null;
  }

  // Core methods
  enable()                    // Start real-time updates
  disable()                   // Stop real-time updates
  forceUpdate()              // Immediate update
  updatePreview(content)      // Render markdown to HTML
  
  // Scroll synchronization
  syncScrollFromEditor()      // Editor → Preview
  syncScrollFromPreview()     // Preview → Editor
  calculateScrollRatio()      // Convert between scroll positions
  
  // Performance optimization
  shouldUpdate(content)       // Check if update needed
  debounceUpdate()           // Debounced update function
  cleanupResources()         // Memory management
}
```

### Integration Points

#### CodeMirror Integration
```javascript
// src/setup/editorSetup.js modifications
import { MarkdownPreviewManager } from '../ui/markdownPreview.js';

export function setupEditor(ydoc, provider, ytext, awareness) {
  const view = new EditorView({
    // ... existing setup
    extensions: [
      // ... existing extensions
      EditorView.updateListener.of((update) => {
        if (update.docChanged && window.markdownPreview) {
          window.markdownPreview.debounceUpdate();
        }
      })
    ]
  });

  // Initialize markdown preview manager
  window.markdownPreview = new MarkdownPreviewManager(view, previewElement);
  
  return view;
}
```

#### Yjs Collaboration Integration
```javascript
// Real-time updates for collaborators
ytext.observe((event) => {
  if (window.markdownPreview && window.markdownPreview.isActive) {
    window.markdownPreview.debounceUpdate();
  }
});
```

## Enhanced Markdown Parser

### Current vs Enhanced Parser
```javascript
// CURRENT: Basic regex-based conversion
markdownToHtml(markdown) {
  return markdown
    .replace(/^### (.*$)/gim, '<h3>$1</h3>')
    .replace(/^## (.*$)/gim, '<h2>$1</h2>')
    // ... limited patterns
}

// ENHANCED: Full featured parser with syntax highlighting
class EnhancedMarkdownParser {
  constructor() {
    this.renderer = new MarkdownRenderer();
    this.highlighter = new SyntaxHighlighter();
  }

  parse(markdown) {
    const tokens = this.tokenize(markdown);
    const ast = this.buildAST(tokens);
    return this.render(ast);
  }

  // Support for:
  // - Tables
  // - Code blocks with syntax highlighting
  // - Mathematical expressions (MathJax/KaTeX)
  // - Footnotes
  // - Task lists
  // - Mentions (@user)
  // - Mermaid diagrams
}
```

### Supported Markdown Features
```markdown
# Headers (all levels)
**Bold** and *italic* text
~~Strikethrough~~ and `inline code`

- Bullet lists
1. Numbered lists
- [ ] Task lists

> Blockquotes
> With multiple lines

```javascript
// Code blocks with syntax highlighting
function example() {
  return "syntax highlighted";
}
```

| Tables | With | Headers |
|--------|------|---------|
| Data   | In   | Rows    |

[Links](https://example.com) and ![Images](image.jpg)

Mathematical expressions: $E = mc^2$

Footnotes[^1]

[^1]: Footnote content here
```

## Performance Optimization

### Debouncing Strategy
```javascript
class DebouncedUpdater {
  constructor(updateFunction, delay = 300) {
    this.updateFunction = updateFunction;
    this.delay = delay;
    this.timer = null;
    this.lastUpdate = 0;
  }

  update(content) {
    const now = Date.now();
    
    // Immediate update if first change or long pause
    if (now - this.lastUpdate > 1000) {
      this.executeUpdate(content);
      return;
    }

    // Debounced update for rapid typing
    clearTimeout(this.timer);
    this.timer = setTimeout(() => {
      this.executeUpdate(content);
    }, this.delay);
  }

  executeUpdate(content) {
    this.lastUpdate = Date.now();
    this.updateFunction(content);
  }
}
```

### Incremental Rendering
```javascript
class IncrementalRenderer {
  constructor() {
    this.lastContent = '';
    this.lastAST = null;
    this.elementCache = new Map();
  }

  render(content) {
    // Fast path: no changes
    if (content === this.lastContent) {
      return this.lastAST;
    }

    // Incremental path: compute diff
    const newAST = this.parseMarkdown(content);
    const patches = this.computeDiff(this.lastAST, newAST);
    this.applyPatches(patches);

    this.lastContent = content;
    this.lastAST = newAST;
    return newAST;
  }

  computeDiff(oldAST, newAST) {
    // Efficient AST diffing algorithm
    // Only re-render changed sections
  }
}
```

### Virtual Scrolling (for large documents)
```javascript
class VirtualScrollManager {
  constructor(container) {
    this.container = container;
    this.viewportHeight = container.clientHeight;
    this.itemHeight = 20; // Average line height
    this.bufferSize = 10; // Render extra items
  }

  getVisibleRange() {
    const scrollTop = this.container.scrollTop;
    const start = Math.floor(scrollTop / this.itemHeight);
    const end = Math.ceil((scrollTop + this.viewportHeight) / this.itemHeight);
    
    return {
      start: Math.max(0, start - this.bufferSize),
      end: end + this.bufferSize
    };
  }
}
```

## Scroll Synchronization

### Bidirectional Sync
```javascript
class ScrollSynchronizer {
  constructor(editor, preview) {
    this.editor = editor;
    this.preview = preview;
    this.isEditorScrolling = false;
    this.isPreviewScrolling = false;
    this.syncEnabled = true;
  }

  setupEventListeners() {
    // Editor scroll → Preview scroll
    this.editor.addEventListener('scroll', debounce(() => {
      if (!this.isPreviewScrolling && this.syncEnabled) {
        this.isEditorScrolling = true;
        this.syncToPreview();
        setTimeout(() => this.isEditorScrolling = false, 100);
      }
    }, 16)); // 60fps

    // Preview scroll → Editor scroll
    this.preview.addEventListener('scroll', debounce(() => {
      if (!this.isEditorScrolling && this.syncEnabled) {
        this.isPreviewScrolling = true;
        this.syncToEditor();
        setTimeout(() => this.isPreviewScrolling = false, 100);
      }
    }, 16));
  }

  syncToPreview() {
    const editorScrollRatio = this.getEditorScrollRatio();
    const previewScrollTop = editorScrollRatio * 
      (this.preview.scrollHeight - this.preview.clientHeight);
    this.preview.scrollTop = previewScrollTop;
  }

  getEditorScrollRatio() {
    const view = this.editor.view;
    const { from, to } = view.viewport;
    const totalLines = view.state.doc.lines;
    return (from + to) / 2 / totalLines;
  }
}
```

### Smart Scroll Mapping
```javascript
// Map editor line numbers to preview elements
class ContentMapper {
  constructor() {
    this.lineToElementMap = new Map();
    this.elementToLineMap = new Map();
  }

  buildMap(editorContent, previewElements) {
    let currentLine = 1;
    
    previewElements.forEach((element, index) => {
      const sourceLines = this.getSourceLines(element);
      this.lineToElementMap.set(currentLine, element);
      this.elementToLineMap.set(element, currentLine);
      currentLine += sourceLines;
    });
  }

  getPreviewElementForLine(lineNumber) {
    // Find closest mapped element
    let bestElement = null;
    let bestDistance = Infinity;
    
    for (const [line, element] of this.lineToElementMap) {
      const distance = Math.abs(line - lineNumber);
      if (distance < bestDistance) {
        bestDistance = distance;
        bestElement = element;
      }
    }
    
    return bestElement;
  }
}
```

## User Interface Enhancements

### Preview Controls
```html
<!-- Enhanced preview header -->
<div class="preview-header">
  <div class="preview-title">Markdown Preview</div>
  <div class="preview-controls">
    <button id="sync-scroll-toggle" title="Toggle scroll sync">
      <span class="sync-icon">🔗</span>
    </button>
    <button id="preview-settings" title="Preview settings">
      <span class="settings-icon">⚙️</span>
    </button>
    <button id="preview-fullscreen" title="Fullscreen preview">
      <span class="fullscreen-icon">⛶</span>
    </button>
  </div>
</div>
```

### Settings Panel
```javascript
const PREVIEW_SETTINGS = {
  autoUpdate: true,
  scrollSync: true,
  mathJax: false,
  syntaxHighlighting: true,
  theme: 'github', // github, minimal, academic
  fontSize: 14,
  lineHeight: 1.6,
  maxWidth: '800px'
};
```

### Loading States
```html
<!-- Show during expensive renders -->
<div class="preview-loading">
  <div class="spinner"></div>
  <span>Rendering preview...</span>
</div>
```

## Error Handling & Edge Cases

### Graceful Degradation
```javascript
class PreviewErrorHandler {
  constructor(previewManager) {
    this.previewManager = previewManager;
    this.errorCount = 0;
    this.maxErrors = 5;
  }

  handleRenderError(error, content) {
    this.errorCount++;
    
    if (this.errorCount > this.maxErrors) {
      // Disable auto-update, show error message
      this.previewManager.disable();
      this.showErrorMessage('Preview disabled due to repeated errors');
      return;
    }

    // Try fallback rendering
    try {
      const fallbackHTML = this.basicMarkdownRender(content);
      this.previewManager.displayContent(fallbackHTML);
    } catch (fallbackError) {
      this.showErrorMessage('Unable to render preview');
    }
  }

  basicMarkdownRender(content) {
    // Ultra-simple fallback renderer
    return content
      .replace(/\n/g, '<br>')
      .replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>')
      .replace(/\*(.*?)\*/g, '<em>$1</em>');
  }
}
```

### Memory Management
```javascript
class ResourceManager {
  constructor() {
    this.observers = [];
    this.timers = [];
    this.eventListeners = [];
  }

  addObserver(observer) {
    this.observers.push(observer);
  }

  addTimer(timer) {
    this.timers.push(timer);
  }

  addEventListener(element, event, handler) {
    element.addEventListener(event, handler);
    this.eventListeners.push({ element, event, handler });
  }

  cleanup() {
    // Disconnect observers
    this.observers.forEach(observer => observer.disconnect());
    
    // Clear timers
    this.timers.forEach(timer => clearTimeout(timer));
    
    // Remove event listeners
    this.eventListeners.forEach(({ element, event, handler }) => {
      element.removeEventListener(event, handler);
    });
    
    // Clear arrays
    this.observers.length = 0;
    this.timers.length = 0;
    this.eventListeners.length = 0;
  }
}
```

## Collaborative Features

### Real-time Updates for All Users
```javascript
// Notify all users when preview mode changes
awareness.setLocalStateField('previewMode', {
  enabled: true,
  scrollPosition: previewElement.scrollTop,
  timestamp: Date.now()
});

// Sync preview state across users
awareness.on('change', () => {
  const states = awareness.getStates();
  states.forEach((state, clientId) => {
    if (clientId !== awareness.clientID && state.previewMode) {
      // Optionally sync preview scroll positions
      // or show indicators of what others are viewing
    }
  });
});
```

### Collaborative Cursor Indicators
```javascript
// Show which section other users are editing
class CollaborativeCursorMapper {
  constructor(previewElement) {
    this.previewElement = previewElement;
    this.userIndicators = new Map();
  }

  updateUserPosition(userId, editorPosition) {
    const previewElement = this.mapEditorToPreview(editorPosition);
    this.showUserIndicator(userId, previewElement);
  }

  showUserIndicator(userId, element) {
    // Add visual indicator showing where other users are editing
    const indicator = document.createElement('div');
    indicator.className = `user-indicator user-${userId}`;
    indicator.style.backgroundColor = this.getUserColor(userId);
    element.appendChild(indicator);
  }
}
```

## Performance Metrics & Monitoring

### Key Performance Indicators
```javascript
class PerformanceMonitor {
  constructor() {
    this.metrics = {
      renderTime: [],
      updateFrequency: 0,
      memoryUsage: [],
      scrollSyncLatency: []
    };
  }

  measureRenderTime(renderFunction) {
    const start = performance.now();
    const result = renderFunction();
    const end = performance.now();
    
    this.metrics.renderTime.push(end - start);
    
    // Alert if renders are taking too long
    if (end - start > 100) {
      console.warn(`Slow preview render: ${end - start}ms`);
    }
    
    return result;
  }

  getAverageRenderTime() {
    const times = this.metrics.renderTime;
    return times.reduce((a, b) => a + b, 0) / times.length;
  }
}
```

### Optimization Thresholds
- **Render time**: < 50ms for smooth experience
- **Update frequency**: Max 10 updates/second
- **Memory usage**: < 50MB for preview DOM
- **Scroll sync latency**: < 16ms (60fps)

## Testing Strategy

### Unit Tests
```javascript
describe('MarkdownPreviewManager', () => {
  test('debounces rapid updates', async () => {
    const manager = new MarkdownPreviewManager(mockEditor, mockPreview);
    manager.enable();
    
    // Trigger rapid updates
    manager.debounceUpdate();
    manager.debounceUpdate();
    manager.debounceUpdate();
    
    // Should only render once
    await wait(400);
    expect(mockPreview.innerHTML).toHaveBeenCalledTimes(1);
  });

  test('synchronizes scroll positions', () => {
    const synchronizer = new ScrollSynchronizer(mockEditor, mockPreview);
    mockEditor.scrollTop = 100;
    
    synchronizer.syncToPreview();
    
    expect(mockPreview.scrollTop).toBeGreaterThan(0);
  });
});
```

### Integration Tests
- Real-time updates during collaborative editing
- Scroll synchronization accuracy
- Performance with large documents (10,000+ lines)
- Memory usage over extended sessions

### User Testing
- Typing performance perception
- Scroll sync intuitiveness
- Preview accuracy for complex markdown
- Accessibility with screen readers

## Accessibility Considerations

### Screen Reader Support
```javascript
// Announce preview updates to screen readers
class AccessibilityManager {
  constructor(previewElement) {
    this.previewElement = previewElement;
    this.announcer = this.createLiveRegion();
  }

  createLiveRegion() {
    const region = document.createElement('div');
    region.setAttribute('aria-live', 'polite');
    region.setAttribute('aria-label', 'Markdown preview updates');
    region.style.position = 'absolute';
    region.style.left = '-9999px';
    document.body.appendChild(region);
    return region;
  }

  announceUpdate() {
    this.announcer.textContent = 'Markdown preview updated';
    setTimeout(() => {
      this.announcer.textContent = '';
    }, 1000);
  }
}
```

### Keyboard Navigation
- Tab navigation through preview elements
- Keyboard shortcuts for preview controls
- Focus management during updates
- Skip links for long previews

## Future Enhancements

### Advanced Features
1. **Live Math Rendering**: MathJax/KaTeX integration
2. **Diagram Support**: Mermaid, PlantUML rendering
3. **Table Editing**: Visual table editor overlay
4. **Live Spell Check**: Real-time spell checking in preview
5. **Export Options**: PDF, HTML, Word export from preview

### Performance Improvements
1. **Web Workers**: Offload parsing to background thread
2. **Streaming Rendering**: Render content as it's being parsed
3. **Predictive Updates**: Pre-render likely changes
4. **GPU Acceleration**: CSS transforms for smooth scrolling
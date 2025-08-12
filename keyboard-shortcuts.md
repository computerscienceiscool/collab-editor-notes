# Collaborative Editor Keyboard Shortcuts

## Current Keyboard Shortcuts

### Built-in CodeMirror Shortcuts
| Shortcut | Action | Platform |
|----------|--------|----------|
| `Ctrl+Z` | Undo | All |
| `Ctrl+Y` | Redo | Windows |
| `Ctrl+Shift+Z` | Redo | Linux alternative |

### Custom Shortcuts Added
| Shortcut | Action | Location |
|----------|--------|----------|
| `Enter` | Execute search | Search input field |

---

## Features That SHOULD Have Keyboard Shortcuts

### Text Formatting (High Priority)
| Shortcut | Action | Implementation Status |
|----------|--------|----------------------|
| `Ctrl+B` | Bold toggle | 🔴 Not implemented |
| `Ctrl+I` | Italic toggle | 🔴 Not implemented |
| `Ctrl+U` | Underline toggle | 🔴 Not implemented |
| `Ctrl+Shift+S` | Strikethrough toggle | 🔴 Not implemented |

### Document Operations (Medium Priority)
| Shortcut | Action | Implementation Status |
|----------|--------|----------------------|
| `Ctrl+Shift+F` | Format entire document | 🔴 Not implemented |
| `Ctrl+F` | Focus search box | 🔴 Not implemented |
| `Escape` | Clear search | 🔴 Not implemented |
| `Ctrl+S` | Save/Export document | 🔴 Not implemented |

### Collaboration (Nice to Have)
| Shortcut | Action | Implementation Status |
|----------|--------|----------------------|
| `Ctrl+Shift+L` | Toggle activity log | 🔴 Not implemented |
| `F2` | Edit username | 🔴 Not implemented |

---

## Platform Recommendations

### Cross-Platform Compatibility
- **Use `Ctrl+` for Windows/Linux**
- **Use `Cmd+` for Mac** (CodeMirror handles this automatically)
- **Avoid Windows-specific keys** (like Windows key)

### Implementation Guidelines
1. **Consistent across platforms** - Let CodeMirror handle platform differences
2. **Standard conventions** - Follow common text editor shortcuts
3. **Non-conflicting** - Avoid overriding essential browser shortcuts
4. **Discoverable** - Include shortcuts in tooltips and menus

### Priority Implementation Order
1. **Text formatting shortcuts** (Ctrl+B, Ctrl+I, Ctrl+U) - High impact, easy to implement
2. **Document operations** (Ctrl+F, Ctrl+S) - Essential functionality
3. **Collaboration shortcuts** (F2, Ctrl+Shift+L) - Nice quality-of-life improvements

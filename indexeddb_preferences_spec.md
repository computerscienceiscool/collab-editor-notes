# IndexedDB Preferences Storage Specification

## Overview
Migrate user preferences from localStorage to IndexedDB with compound key architecture to support keyboard shortcuts, UI settings, and future preference categories.

## Current State Analysis
- User preferences scattered across localStorage keys
- No structured preference management
- Limited by localStorage 5-10MB quota
- No versioning or migration support

## Requirements

### Functional Requirements
1. **Compound Key Structure**: Organize preferences by category and subcategory
2. **Migration**: Seamless migration from localStorage to IndexedDB
3. **Versioning**: Schema versioning for future updates
4. **Backup/Restore**: Export/import preferences as JSON
5. **Performance**: Faster than localStorage for complex objects
6. **Offline Support**: Works without network connectivity

### Technical Requirements
1. **Browser Support**: Chrome 58+, Firefox 55+, Safari 10.1+
2. **Storage Quota**: Support for larger preference files (50MB+)
3. **Concurrent Access**: Handle multiple tabs accessing same preferences
4. **Error Handling**: Graceful fallback to localStorage if IndexedDB fails
5. **Memory Efficiency**: Lazy loading of preference categories

## IndexedDB Schema Design

### Database Structure
```javascript
// Database: collaborative-editor-preferences
// Version: 1

const DB_SCHEMA = {
  name: 'collaborative-editor-preferences',
  version: 1,
  stores: {
    preferences: {
      keyPath: ['category', 'subcategory'],
      autoIncrement: false,
      indices: {
        category: { keyPath: 'category', unique: false },
        lastModified: { keyPath: 'lastModified', unique: false }
      }
    },
    metadata: {
      keyPath: 'key',
      autoIncrement: false
    }
  }
};
```

### Compound Key Structure
```javascript
// preferences store structure
{
  category: 'keyboard-shortcuts',      // Primary key part 1
  subcategory: 'formatting',          // Primary key part 2
  data: {
    'bold': 'Ctrl+B',
    'italic': 'Ctrl+I',
    'underline': 'Ctrl+Alt+U'
  },
  version: '1.0',
  lastModified: 1703001234567,
  checksum: 'abc123def456'             // Data integrity check
}

// Example records:
['keyboard-shortcuts', 'formatting'] → formatting shortcuts
['keyboard-shortcuts', 'navigation'] → navigation shortcuts
['ui-settings', 'theme'] → theme preferences
['ui-settings', 'layout'] → layout preferences
['document-settings', 'export'] → export preferences
```

### Metadata Store
```javascript
{
  key: 'schema-version',
  value: '1.0',
  timestamp: 1703001234567
}

{
  key: 'migration-status',
  value: {
    localStorage: 'completed',
    version: '1.0',
    timestamp: 1703001234567
  }
}
```

## Implementation Architecture

### PreferencesManager Class
```javascript
class PreferencesManager {
  constructor() {
    this.db = null;
    this.cache = new Map();
    this.initPromise = null;
  }

  // Core methods
  async init()
  async get(category, subcategory)
  async set(category, subcategory, data)
  async delete(category, subcategory)
  async list(category)
  
  // Migration methods
  async migrateFromLocalStorage()
  async exportPreferences()
  async importPreferences(jsonData)
  
  // Utility methods
  async clear()
  async getStorageUsage()
  validateData(data)
}
```

### Storage Categories

#### Keyboard Shortcuts
```javascript
// ['keyboard-shortcuts', 'file-operations']
{
  'new-document': 'Ctrl+N',
  'print': 'Ctrl+P',
  'copy-url': 'Ctrl+Shift+U'
}

// ['keyboard-shortcuts', 'text-formatting']
{
  'bold': 'Ctrl+B',
  'italic': 'Ctrl+I',
  'underline': 'Ctrl+U'
}
```

#### UI Settings
```javascript
// ['ui-settings', 'interface']
{
  'theme': 'light',
  'fontSize': 14,
  'lineNumbers': true,
  'toolbar': 'visible'
}

// ['ui-settings', 'panels']
{
  'activityLog': {
    'position': { x: 20, y: 120 },
    'size': { width: 280, height: 400 },
    'visible': false
  },
  'markdownPreview': {
    'position': 'right',
    'width': '50%',
    'autoUpdate': true
  }
}
```

## Migration Strategy

### Phase 1: Parallel Operation
1. Keep localStorage as primary
2. Write to both localStorage and IndexedDB
3. Read from localStorage with IndexedDB backup
4. User testing with IndexedDB functionality

### Phase 2: IndexedDB Primary
1. Read from IndexedDB first, localStorage fallback
2. Write only to IndexedDB
3. One-time migration prompt for users
4. Keep localStorage for emergency fallback

### Phase 3: IndexedDB Only
1. Remove localStorage dependencies
2. Clean up migration code
3. Full IndexedDB feature utilization

### Migration Data Mapping
```javascript
const MIGRATION_MAP = {
  // localStorage key → IndexedDB compound key
  'username': ['user-settings', 'profile'],
  'usercolor': ['user-settings', 'profile'],
  'line-numbers-enabled': ['ui-settings', 'editor'],
  'activity-log-position': ['ui-settings', 'panels'],
  'notification-settings': ['ui-settings', 'notifications'],
  // Future: keyboard shortcuts from new system
  'keyboard-shortcuts': ['keyboard-shortcuts', 'custom']
};
```

## Error Handling & Fallback

### Fallback Hierarchy
1. **IndexedDB** (primary)
2. **localStorage** (fallback)
3. **In-memory defaults** (emergency)

### Error Scenarios
```javascript
class PreferencesError extends Error {
  constructor(message, code, fallback = null) {
    super(message);
    this.code = code;
    this.fallback = fallback;
  }
}

// Error codes:
// INDEXEDDB_NOT_SUPPORTED
// DATABASE_OPEN_FAILED
// TRANSACTION_FAILED
// QUOTA_EXCEEDED
// DATA_CORRUPTION
// MIGRATION_FAILED
```

### Recovery Strategies
1. **Database corruption**: Clear and reinitialize
2. **Quota exceeded**: Prompt user to export/clear old data
3. **Browser incompatibility**: Automatic localStorage fallback
4. **Migration failure**: Preserve localStorage, retry later

## Performance Optimization

### Caching Strategy
```javascript
// In-memory cache for frequently accessed preferences
const preferenceCache = {
  'keyboard-shortcuts': new Map(),
  'ui-settings': new Map(),
  maxAge: 300000, // 5 minutes
  cleanup: () => {} // LRU eviction
};
```

### Batch Operations
```javascript
// Batch multiple preference updates
await preferencesManager.transaction(async (tx) => {
  await tx.set('keyboard-shortcuts', 'formatting', formattingShortcuts);
  await tx.set('keyboard-shortcuts', 'navigation', navigationShortcuts);
  await tx.set('ui-settings', 'theme', themeSettings);
});
```

### Lazy Loading
- Load preference categories only when needed
- Cache frequently used preferences
- Background sync for less critical data

## Data Integrity

### Validation Schema
```javascript
const PREFERENCE_SCHEMAS = {
  'keyboard-shortcuts': {
    type: 'object',
    patternProperties: {
      '^[a-z-]+$': {
        type: 'string',
        pattern: '^(Ctrl|Alt|Shift|Meta)\\+[A-Za-z0-9]+$'
      }
    }
  },
  'ui-settings': {
    type: 'object',
    properties: {
      theme: { enum: ['light', 'dark', 'auto'] },
      fontSize: { type: 'number', minimum: 8, maximum: 72 }
    }
  }
};
```

### Checksums
- SHA-256 hashes for preference data
- Detect corruption and trigger recovery
- Verify data integrity during migration

## Browser Compatibility

### Feature Detection
```javascript
function supportsIndexedDB() {
  return 'indexedDB' in window && 
         typeof window.indexedDB.open === 'function';
}

function getStorageQuota() {
  if ('storage' in navigator && 'estimate' in navigator.storage) {
    return navigator.storage.estimate();
  }
  return Promise.resolve({ quota: 5 * 1024 * 1024 }); // 5MB fallback
}
```

### Progressive Enhancement
- Core functionality works with localStorage
- Enhanced features with IndexedDB
- Graceful degradation on older browsers

## Development Workflow

### File Structure
```
src/preferences/
├── PreferencesManager.js      # Main class
├── IndexedDBAdapter.js        # IndexedDB operations
├── LocalStorageAdapter.js     # Fallback adapter
├── MigrationManager.js        # Data migration
├── ValidationSchemas.js       # Data validation
└── PreferencesUI.js          # Settings interface
```

### Testing Strategy
- Unit tests for each adapter
- Integration tests for migration
- Performance tests with large datasets
- Cross-browser compatibility tests

## Security Considerations

### Data Sensitivity
- Preferences contain no sensitive personal data
- No encryption required for current use case
- Future: capability-based access control

### Privacy
- All data stored locally only
- No network transmission of preferences
- User controls export/import functionality

## Usage Examples

### Basic Operations
```javascript
const prefs = new PreferencesManager();
await prefs.init();

// Get keyboard shortcuts for formatting
const shortcuts = await prefs.get('keyboard-shortcuts', 'formatting');

// Update UI theme
await prefs.set('ui-settings', 'theme', { 
  mode: 'dark', 
  accent: '#1a73e8' 
});

// Export all preferences
const backup = await prefs.exportPreferences();
```

### Integration with Existing Code
```javascript
// Replace localStorage calls
// OLD:
const username = localStorage.getItem('username');

// NEW:
const profile = await prefs.get('user-settings', 'profile');
const username = profile?.username || 'Anonymous';
```

## Future Enhancements

### Advanced Features
1. **Preference Profiles**: Multiple preference sets
2. **Cloud Sync**: Sync across devices (future)
3. **Team Preferences**: Shared settings in collaborative environments
4. **Audit Trail**: Track preference changes over time
5. **Compression**: LZMA compression for large preference sets

### Integration Points
- Document-specific preferences
- Room-based preference overrides
- User role-based preference restrictions
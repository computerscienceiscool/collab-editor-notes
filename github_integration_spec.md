# GitHub Integration Feature Specification

## Overview
Add GitHub repository integration to the collaborative text editor, allowing users to commit documents directly to GitHub repositories without leaving the browser. This feature enables co-author attribution and supports the multi-agent coding workflow discussed in company strategy meetings.

## Business Requirements

### Primary Goals
- Enable zero-installation Git workflow for non-technical users
- Support co-author attribution in commits (multiple collaborators credited)
- Integrate with existing collaborative document editing features
- Provide foundation for future multi-agent coding tool integration

### Success Criteria
- Users can commit documents to GitHub repositories without command line tools
- Co-authors are properly attributed in Git commit messages
- Integration works entirely in browser with minimal setup
- Feature is discoverable but optional (doesn't interfere with non-GitHub users)

## Technical Architecture

### Data Storage
All GitHub settings stored in browser `localStorage` to maintain privacy and avoid server dependencies.

**Global Settings Schema:**
```javascript
{
  githubUsername: string,     // GitHub username (e.g., "jjohnson")
  githubToken: string,        // Personal access token (encrypted)
  defaultRepo: string,        // Default repository name
  isConfigured: boolean       // Quick check for setup completion
}
```

**Per-Document Settings Schema:**
```javascript
{
  targetRepo: string,         // Repository name
  filePath: string,          // Target file path (e.g., "notes/meeting-2025-09-25.md")
  branch: string,            // Target branch (default: "main")
  lastCommitSha: string      // For update operations
}
```

### GitHub API Integration

**Required API Endpoints:**
- `GET /user` - Verify token and get user info
- `GET /user/repos` - List user's repositories for dropdown
- `GET /repos/{owner}/{repo}/contents/{path}` - Check if file exists
- `PUT /repos/{owner}/{repo}/contents/{path}` - Create/update files
- `GET /repos/{owner}/{repo}/collaborators` - Get repo collaborators (for co-author suggestions)

**Authentication:**
- Use GitHub Personal Access Tokens
- Required scopes: `repo` (for private repos) or `public_repo` (for public only)
- Store encrypted in localStorage
- Include in Authorization header: `Bearer {token}`

## User Interface Design

### Menu Integration

**Tools Menu Addition:**
Add new menu item: "GitHub Settings" under Tools menu, positioned after "Preferences"

**File Menu Addition:**
Add new menu items under File menu:
- "Commit to GitHub" (with keyboard shortcut)
- "GitHub Repository Settings" (for per-document config)

### GitHub Settings Dialog

**Location:** Tools → GitHub Settings

**Dialog Contents:**
```
┌─ GitHub Configuration ─────────────────────┐
│                                            │
│ GitHub Username: [________________]        │
│                                            │
│ Personal Access Token: [________________]  │
│ (This will be stored securely in your     │
│  browser and never sent to our servers)   │
│                                            │
│ Default Repository: [________________]     │
│                                            │
│ [Test Connection] [Save] [Cancel]          │
│                                            │
│ Need help? [How to create a token]         │
└────────────────────────────────────────────┘
```

**Validation:**
- Test GitHub connection on save
- Show success/error messages
- Validate token has required permissions

### Repository Settings Dialog

**Triggered by:** File → Commit to GitHub (if no repo configured for current document)

**Dialog Contents:**
```
┌─ Choose GitHub Repository ─────────────────┐
│                                            │
│ Repository: [dropdown with user's repos]   │
│                                            │
│ File Path: [notes/document-name.md]        │
│                                            │
│ Branch: [main ▼]                          │
│                                            │
│ [Browse Branches] [Create New Branch]      │
│                                            │
│ [Save Settings] [Cancel]                   │
└────────────────────────────────────────────┘
```

### Commit Dialog

**Triggered by:** File → Commit to GitHub (when repo is configured)

**Dialog Contents:**
```
┌─ Commit to GitHub ─────────────────────────┐
│                                            │
│ Repository: meeting-notes                  │
│ File: notes/weekly-standup.md              │
│ Branch: main                              │
│                                            │
│ Commit Message:                            │
│ ┌─────────────────────────────────────────┐ │
│ │ Update weekly standup notes             │ │
│ │                                         │ │
│ └─────────────────────────────────────────┘ │
│                                            │
│ Co-Authors (from current session):         │
│ ☑ Jane Smith <jane@example.com>           │
│ ☑ Bob Wilson <bob@example.com>            │
│ ☐ AI Assistant <ai@editor.com>            │
│                                            │
│ [Commit] [Change Repository] [Cancel]      │
└────────────────────────────────────────────┘
```

## Implementation Details

### Phase 1: Basic Configuration

**Files to Modify:**
- Add GitHub settings to existing preferences system
- Extend localStorage handling for GitHub config
- Add menu items to existing menu system

**New Components Needed:**
- `GitHubSettingsDialog.js` - Configuration dialog
- `GitHubService.js` - API interaction service
- `GitHubStorage.js` - localStorage management

### Phase 2: Repository Integration

**GitHub API Service Implementation:**
```javascript
class GitHubService {
  async validateToken(token) {
    // GET /user with token
    // Return user info or error
  }
  
  async getUserRepos(token) {
    // GET /user/repos
    // Return array of repo names
  }
  
  async getFileContent(token, repo, path, branch) {
    // GET /repos/{owner}/{repo}/contents/{path}
    // Return file content and SHA for updates
  }
  
  async commitFile(token, repo, path, branch, content, message, coAuthors) {
    // PUT /repos/{owner}/{repo}/contents/{path}
    // Include co-author trailer in commit message
  }
}
```

**Co-Author Attribution Format:**
```
Update meeting notes from collaborative session

Co-authored-by: Jane Smith <jane@example.com>
Co-authored-by: Bob Wilson <bob@example.com>
```

### Phase 3: User Experience Enhancements

**Status Indicator:**
Add GitHub status to document header/status bar:
```
[Document Title] │ GitHub: ✓ meeting-notes/notes.md │ main
```

**Keyboard Shortcuts:**
- Add configurable shortcut for "Commit to GitHub" (suggest Ctrl+Shift+G)
- Integrate with existing keyboard customization system

**Error Handling:**
- Network connectivity issues
- Invalid tokens or expired permissions
- Repository access problems
- File conflicts (if file was modified externally)

## Technical Considerations

### Security
- Never send tokens to application server
- Use HTTPS for all GitHub API calls
- Consider encrypting tokens in localStorage
- Clear sensitive data on logout/close

### Performance
- Cache repository lists to reduce API calls
- Implement request rate limiting
- Show loading states for API operations
- Handle GitHub API rate limits gracefully

### Browser Compatibility
- Test with Chrome, Firefox, Safari
- Ensure CORS handling works correctly
- Graceful degradation if localStorage unavailable

### Error States
- No internet connection
- Invalid GitHub credentials
- Repository doesn't exist
- Insufficient permissions
- File conflicts

## Testing Requirements

### Unit Tests
- GitHub API service methods
- localStorage management
- Co-author message formatting
- Error handling scenarios

### Integration Tests
- Full commit workflow
- Settings persistence
- Menu integration
- Dialog interactions

### User Acceptance Tests
- First-time setup flow
- Committing new and existing files
- Co-author attribution verification
- Error recovery scenarios

## Documentation Updates

### User Guide Additions
Add new section: "GitHub Integration"
- How to create personal access tokens
- Setting up repositories
- Collaborative workflow with co-authors
- Troubleshooting common issues

### Developer Documentation
- GitHub API integration patterns
- localStorage schema documentation
- Extension points for future features

## Future Considerations

### Potential Enhancements
- Branch management (create, switch, merge)
- Pull request creation
- Conflict resolution interface
- Integration with other Git providers (GitLab, Bitbucket)

### Multi-Agent Coding Integration
This GitHub integration provides the foundation for:
- AI agents committing on separate branches
- Automated merge workflows
- Real-time collaboration with version control
- Integration with mob programming tools

## Implementation Timeline

**Week 1:** GitHub settings dialog and basic API integration
**Week 2:** Repository selection and file management
**Week 3:** Commit functionality with co-author attribution
**Week 4:** UI polish, error handling, and testing

## Dependencies

**External Libraries:**
- No additional dependencies required (use browser fetch API)
- Consider crypto-js for token encryption (optional)

**Browser APIs:**
- localStorage for settings persistence
- fetch API for GitHub integration
- Standard DOM APIs for UI components

## Success Metrics

- Users successfully commit documents without command line tools
- Co-author attribution appears correctly in GitHub commit history
- Feature adoption rate among collaborative sessions
- Reduction in support requests for Git workflow issues
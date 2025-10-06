# Multi-Agent Group Programming Editor - Full Specification
## Project Overview 

**Working Name:** Multi-Agent Coding Tool (needs final name)

**Core Objective:** Create a collaborative programming environment that allows multiple developers to work together synchronously or asynchronously, supporting web-based editing (collaborative editor), local editors (VS Code, Neovim), and future LLM agents.

**Strategic Purpose:**
- Replace/supersede: mob.sh, AIDDA, grokker
- Enable LLM participation in mob programming
- Dog-food development (use the tool to build itself)
- Early "killer app" for promisegrid
- Revenue source (open source, hosted, Stripe billing)

---

## 1. USER REQUIREMENTS

### 1.1 User Types
- **Human Developers** using web-based editor
- **Human Developers** using local editors (VS Code, Neovim, potentially others)
- **LLM Agents** (future) participating as "developers"
- **Mix of all above** in same session

### 1.2 Use Cases

**Synchronous (Real-time) Mode:**
- Multiple users editing same file simultaneously
- Live cursor visibility across all participants
- Real-time keystroke propagation
- Immediate conflict resolution
- Visual awareness of what others are typing

**Asynchronous (Git-flow) Mode:**
- Traditional mob.sh style handoffs
- Timed rotations with commits between turns
- "Mob of one" = standard async workflow
- No special handling needed (falls out naturally)

**Mixed Mode:**
- Some participants in real-time, others async
- Graceful degradation based on editor capabilities
- Clear indication of who's in which mode

---

## 2. EDITOR INTEGRATION REQUIREMENTS

### 2.1 Web-Based Editor (Existing Collaborative Editor)

**Current State:**
- JavaScript-based
- Runs on local web server (for now)
- Uses YJS (CRDT) for conflict-free editing
- GitHub integration working (pull, edit, commit, push)
- Markdown preview
- Token stored in browser LocalStorage
- Menu-driven UI
- Keyboard shortcuts (modeled after GitHub, needs customization feature)

**Required Enhancements:**
- Multi-user server deployment (not just localhost)
- Real-time cursor sharing visualization
- User presence indicators
- Session management
- Lock/freeze cursor capability (for git commits)
- Co-author attribution (mentioned but not tested)

**Current Gaps:**
- Installation documentation incomplete (HIGH PRIORITY)
- Keyboard customization UI not implemented
- No multi-user testing done yet
- Browser storage limitations (can't use localStorage for shared state)

### 2.2 VS Code Integration

**Requirements:**
- Plugin/extension for VS Code
- Receive real-time keystrokes from other participants
- Send keystrokes to shared session
- Display other participants' cursors
- Trigger git commits from within editor
- Participate in cursor locking protocol

**Technical Approach:**
- VS Code Extension API
- WebSocket or similar for real-time communication
- Git integration through VS Code's native git support

**Unknown/Research Needed:**
- VS Code extension development learning curve
- Performance implications of real-time sync
- Cursor rendering in VS Code from external sources

### 2.3 Neovim Integration

**Requirements:**
- Plugin for Neovim
- Real-time keystroke sharing
- Multi-cursor display (show cursors from web users and VS Code users)
- Git commit triggering (CLI or plugin keystroke)
- Participate in session locking protocol

**Technical Approach:**
- Neovim has network protocol capabilities
- Can send/receive keystrokes over network
- FluBits proved this is possible (existence proof)
- Likely Lua-based plugin (Neovim scripting language)

**Reference Implementation:**
- FluBits Neovim plugin (defunct but code available)
  - Written in Python + VimScript
  - 4 years old, company dead (SSL cert expired 888 days ago)
  - Can study their approach

**Unknown/Research Needed:**
- Neovim network protocol documentation
- How FluBits implemented multi-cursor display
- Performance and latency considerations
- Plugin development learning curve (Lua)

### 2.4 Cross-Editor Communication

**The Core Challenge:**
How do users in different editors see each other and collaborate?

**Requirements:**
- Keystrokes from any editor propagate to all others
- Cursor positions visible across all editors
- File state synchronized across all editors
- Session join/leave notifications
- User identification (names, colors for cursors)

**Proposed Architecture (from transcript):**
```
All Editors → Central Server ← All Editors
             (maintains state)
                    ↕
              Git Repository
```

**Data Flow:**
1. User types in any editor
2. Keystroke sent to server
3. Server propagates to all connected editors
4. CRDT algorithm ensures consistency
5. Each editor renders keystroke in appropriate way

---

## 3. GIT INTEGRATION REQUIREMENTS

### 3.1 Current Git Capabilities (Web Editor)
- Pull files from GitHub repositories
- Preview files before pulling
- Commit changes with messages
- Push to GitHub
- Repository browser (shows all repos user has access to)
- Uses GitHub REST API with personal access token

### 3.2 Required Git Enhancements

**Commit Protocol (Critical):**
When any participant initiates commit:
1. **Lock/freeze all cursors** (block all editing)
2. **Pull to all clones** (ensure everyone has latest)
3. **Commit locally** (one participant or server)
4. **Push to remote**
5. **Pull to all other clones**
6. **Unlock cursors** (resume editing)

**Rationale:** Prevents merge conflicts by ensuring no one types during sync.

**Auto-generated Commit Messages:**
- Use existing grokker code for AI-generated commit messages
- Call OpenAI API or similar
- Create system message
- Create message slice
- Call openai.CompleteChat(modelName, msgs)
- Returns grokker/client.Results object
- Optional feature (user can override)

**Co-author Attribution:**
- Git commit co-authors in commit message
- Format: `Co-authored-by: Name <email>`
- Track who was in session when commit made
- Not yet tested in current implementation

### 3.3 Alternative Git Architecture (Future)

**From transcript discussion:**
Instead of locking cursors, treat each user's keystroke stream as separate branch:
- Continuously merge branches in real-time
- Flag merge conflicts in UI
- Users resolve conflicts in real-time (not heavyweight git merge)
- More elegant but requires more infrastructure
- "Can probably be done better once we have more grid bits running"

This is **advanced/future** - not initial implementation.

---

## 4. TECHNICAL ARCHITECTURE

### 4.1 Current Technology Stack

**Web Editor:**
- JavaScript
- YJS (CRDT library for conflict-free editing)
- GitHub REST API
- Browser-based (no Node.js backend currently)
- Local web server for development

**Limitations:**
- localStorage/sessionStorage NOT supported in Claude.ai environment
- Currently single-user (localhost only)
- No central server yet

### 4.2 Required Server Architecture

**Server Responsibilities:**
- Maintain authoritative file state
- Clone git repositories (okay if self-hosted, not third-party)
- Propagate keystrokes between all clients
- Handle cursor position synchronization
- Manage session state (who's connected, cursor locks)
- Enforce commit protocol
- Handle authentication/authorization

**Server Technology Options:**
- Node.js (fits with existing JavaScript)
- Go (performance, Steve's preference based on grokker)
- Python (fits with some existing tools)

**Decision needed:** What language/framework for server?

### 4.3 Network Protocol

**Requirements:**
- Real-time bidirectional communication
- Low latency (keystroke propagation must feel instant)
- Handle disconnections gracefully
- Reconnection with state sync

**Options:**
- WebSocket (standard choice for real-time)
- WebRTC (peer-to-peer, more complex)
- Server-Sent Events (one-way, insufficient)

**Likely choice:** WebSocket

### 4.4 Data Structures

**Session State:**
```javascript
{
  sessionId: string,
  repositoryUrl: string,
  currentBranch: string,
  participants: [
    {
      userId: string,
      userName: string,
      editor: 'web' | 'vscode' | 'neovim',
      cursorPosition: {file: string, line: int, column: int},
      cursorColor: string,
      connected: boolean,
      lastSeen: timestamp
    }
  ],
  cursorsLocked: boolean,
  lockedBy: userId | null,
  lockedReason: 'commit' | null
}
```

**Keystroke Message:**
```javascript
{
  type: 'keystroke',
  userId: string,
  file: string,
  operation: 'insert' | 'delete',
  position: {line: int, column: int},
  content: string,
  timestamp: int
}
```

**Cursor Message:**
```javascript
{
  type: 'cursor',
  userId: string,
  file: string,
  position: {line: int, column: int}
}
```

### 4.5 CRDT Integration

**Current:** YJS in web editor

**Challenge:** How do local editors participate in CRDT?
- Option 1: Plugin implements YJS client
- Option 2: Server translates between CRDT and editor operations
- Option 3: Use different CRDT that has multi-language support

**Research needed:** CRDT libraries for VS Code, Neovim plugins

---

## 5. USER INTERFACE REQUIREMENTS

### 5.1 Web Editor UI

**Current Features:**
- Menu system (Tools → GitHub Settings, etc.)
- Markdown preview
- File editing area
- Keyboard shortcuts (Control/Command based)

**Required Additions:**
- User list/presence panel
- Cursor visualization for other users (with names/colors)
- Session status indicator (connected, syncing, locked, etc.)
- Lock indicator when cursors frozen
- Conflict resolution UI (future/advanced)
- Session join/create dialog
- Keyboard shortcut customization UI

**Design Principles (from artifacts_info):**
- Contemporary, not traditional
- Interactive elements that feel alive
- Thoughtful animations
- Accessibility (contrast, semantic markup)

### 5.2 Local Editor UI (VS Code, Neovim)

**Plugin UI Requirements:**
- Other users' cursors rendered in editor
- User name tooltips on hover
- Session panel or status bar
- Lock indicator when editing disabled
- Minimal disruption to normal workflow
- Native feel (doesn't look like web transplant)

**VS Code Specifics:**
- Status bar integration
- Sidebar panel for user list
- Cursor decorations (VS Code decoration API)
- Commands in command palette

**Neovim Specifics:**
- Status line integration
- Floating windows for user list
- Virtual text for cursors (or signs in gutter)
- Commands accessible via : commands

---

## 6. SECURITY & PRIVACY REQUIREMENTS

### 6.1 Authentication
- Personal access tokens (current for GitHub)
- Need authentication for central server
- User identification for session participants
- Per-session tokens/keys?

### 6.2 Authorization
- Repository access control (who can join session on which repo?)
- Private repo handling (okay if self-hosted server)
- Session access control (invite-only vs open?)

### 6.3 Data Privacy
- Private code stays on self-hosted server (not third-party like FluBits)
- Token storage security (currently browser localStorage)
- Network traffic encryption (WSS/TLS)

### 6.4 Repository Size Limits
- Large monorepos must be supported
- FluBits failed on CD International repo (too big)
- Server must handle large clones
- Partial clones? Sparse checkouts?

---

## 7. PERFORMANCE REQUIREMENTS

### 7.1 Latency
- Keystroke propagation: < 100ms ideal, < 200ms acceptable
- Cursor updates: < 50ms ideal
- Commit operations: < 5s acceptable (users expect some delay)

### 7.2 Scalability
- How many simultaneous users per session? (5-10 typical mob size)
- How many simultaneous sessions per server?
- How many files open per session?

### 7.3 Bandwidth
- Minimize data sent for keystrokes
- Efficient cursor position updates
- File content only sent on join or conflict

---

## 8. INTEGRATION WITH EXISTING TOOLS

### 8.1 Tools to Replace/Supersede
- **mob.sh** - simple git-based mob programming tool
- **AIDDA** - (mentioned as tool to replace)
- **grokker** - Steve's LLM tool (has commit message generation)

### 8.2 Tools to Integrate With
- **GitHub** (done in web editor)
- **Gitea** (future - self-hosted git)
- **Promisebase** (Steve's grid infrastructure, future)
- **LLMs** (hosted: OpenAI, Claude; local: Ollama)

### 8.3 Grokker Integration Specifics
For auto-generating commit messages, borrow this pattern from grokker:
```
1. Create system message
2. Create message slice (with diff context)
3. Call openai.CompleteChat(modelName, msgs)
4. Returns grokker/client.Results object
5. Extract commit message from result
```

---

## 9. DEPLOYMENT REQUIREMENTS

### 9.1 Development Deployment
- Local web server (current)
- Local git repositories
- Single developer testing
- **CRITICAL:** Installation documentation must be complete

### 9.2 Team Deployment
- Central server accessible to team
- Shared git repositories
- Multiple simultaneous sessions
- Self-hosted (not public cloud initially)

### 9.3 Production Deployment (Future)
- Public hosted service
- Multi-tenancy
- Billing integration (Stripe)
- Monitoring and logging
- Backup and recovery

### 9.4 Infrastructure Integration

**Promisebase (Steve's platform):**
- Container management code (in progress)
  - Grid-native orchestration
  - Replace docker/podman/kubernetes
- VM and bare-metal host management (next)
- Virtual desktops (for editor instances)
- CID-based addressing (replacing hash paths)

**Virtual Desktops:**
- Containers people can customize
- Pre-configured with editors
- Network-based access
- Persistence of configuration

---

## 10. KEYBOARD BINDINGS STRATEGY

### 10.1 Current State
- Modeled after GitHub shortcuts
- Platform detection (navigator.platform)
- Mac vs Windows/Linux handling
- Most working but some conflicts with browser/OS
- Testing could be "endless"

### 10.2 Recommended Approach
**Use Google Docs keyboard bindings as default** (better model for editor)
- Well-tested across browsers
- Users already familiar
- Fewer conflicts than GitHub shortcuts

**Allow full user customization:**
- Settings UI for rebinding keys
- Per-user preferences stored
- Export/import keymaps
- Reset to defaults option

**Platform-specific defaults:**
- Mac: Command-based
- Windows/Linux: Control-based
- Automatic detection and appropriate defaults

### 10.3 Implementation Priority
**LOW PRIORITY** - Don't let this block progress
- Provide reasonable defaults
- Document known conflicts
- Build customization later
- "Move on to next thing" rather than perfect bindings

---

## 11. TESTING REQUIREMENTS

### 11.1 Unit Testing
- CRDT operations
- Git operations (pull, commit, push)
- Message serialization/deserialization
- Cursor position calculations

### 11.2 Integration Testing
- Web editor with server
- VS Code plugin with server
- Neovim plugin with server
- Cross-editor sessions (web + VS Code + Neovim)

### 11.3 User Acceptance Testing
- Multi-user sessions with real developers
- Real-world repos (including large monorepos)
- Extended sessions (1+ hour)
- Network interruption recovery
- Commit protocol under load

### 11.4 Not Yet Tested
- Co-author attribution (mentioned but not tested)
- Multiple simultaneous users in web editor
- Any local editor integration (not started)

---

## 12. DOCUMENTATION REQUIREMENTS

### 12.1 User Documentation
- **Installation guide** (CRITICAL - currently incomplete)
  - Web editor setup
  - VS Code plugin installation
  - Neovim plugin installation
  - Server setup
- User guide (how to use each feature)
- Keyboard shortcuts reference
- Troubleshooting guide

### 12.2 Developer Documentation
- Architecture overview
- API documentation (server endpoints)
- Plugin development guide (for additional editors)
- CRDT protocol documentation
- Commit protocol specification
- Contributing guide

### 12.3 Current Documentation Status
- README exists but installation section not updated
- No other documentation mentioned
- **This is blocking adoption and testing**

---

## 13. PHASED IMPLEMENTATION PLAN

### Phase 1: Foundation (Current → Next 2-4 weeks)
**Goal:** Get others able to install and test web editor

- [ ] Complete installation documentation (HIGH PRIORITY)
- [ ] Test multi-user web editor sessions
- [ ] Deploy to accessible server (not just localhost)
- [ ] Implement co-author attribution
- [ ] Test commit protocol with multiple users
- [ ] Basic user presence UI

### Phase 2: VS Code Integration (4-8 weeks)
**Goal:** One local editor working with web editor

- [ ] Research VS Code extension API
- [ ] Build basic VS Code plugin (connect to session)
- [ ] Implement keystroke sending from VS Code
- [ ] Receive and display other users' keystrokes
- [ ] Display other users' cursors in VS Code
- [ ] Test mixed sessions (web + VS Code)
- [ ] Git commit triggering from VS Code

### Phase 3: Neovim Integration (8-12 weeks)
**Goal:** Second local editor working

- [ ] Research Neovim plugin development (Lua)
- [ ] Study FluBits Neovim plugin code
- [ ] Build basic Neovim plugin
- [ ] Implement keystroke sync
- [ ] Display other users' cursors
- [ ] Test mixed sessions (all three editors)
- [ ] Git commit triggering from Neovim

### Phase 4: LLM Integration (12-16 weeks)
**Goal:** LLMs can participate as developers

- [ ] Define LLM agent interface
- [ ] Implement grokker integration
- [ ] LLM can read file state
- [ ] LLM can propose changes
- [ ] Human approval workflow
- [ ] LLM as "background pair programmer"

### Phase 5: Advanced Features (16+ weeks)
**Goal:** Production-ready, revenue-generating

- [ ] Real-time conflict resolution UI
- [ ] Keyboard customization interface
- [ ] Promisebase/grid integration
- [ ] Virtual desktop support
- [ ] Billing integration (Stripe)
- [ ] Monitoring and analytics
- [ ] Multi-tenancy
- [ ] Performance optimization

---

## 14. RISKS & CHALLENGES

### 14.1 Technical Risks

**High Risk:**
- CRDT performance with multiple editors and users
- Latency over network (especially for remote users)
- Large repository handling
- Editor plugin API limitations (what if we can't do X in VS Code?)
- Merge conflict handling complexity

**Medium Risk:**
- Server scalability
- Connection reliability (WebSocket stability)
- Cross-platform keyboard binding conflicts
- Browser compatibility for web editor

**Low Risk:**
- Git operations (well-understood, libraries exist)
- GitHub API integration (already working)

### 14.2 Resource Risks

**JJ's Concerns (and they're valid):**
- Learning curve for VS Code extension dev
- Learning curve for Neovim plugin dev (Lua)
- Learning curve for server architecture
- Multiple parallel complex tasks
- No one else can test yet (documentation gap)
- Feeling overwhelmed by scope

**Team Risks:**
- Steve busy with Promisebase container management
- Government shutdown affecting cash flow
- Limited testing resources initially
- Competing priorities (ops research tool, other projects)

### 14.3 Market/Product Risks
- Competing tools exist (FluBits failed but others may succeed)
- Will users actually want this vs simpler tools?
- LLM integration complexity may delay value delivery
- Revenue model unproven

### 14.4 Dependency Risks
- Promisebase infrastructure not ready
- Virtual desktop infrastructure not ready
- Grid bits incomplete
- These are okay - can launch without them, add later

---

## 15. SUCCESS CRITERIA

### 15.1 Minimum Viable Product (MVP)
- [ ] Multiple users in web editor simultaneously
- [ ] Real-time cursor sharing visible
- [ ] Git commits work with cursor locking
- [ ] Installation docs complete (others can set up)
- [ ] Stable for 1+ hour sessions
- [ ] Works with real project code

### 15.2 Version 1.0
- [ ] All above plus:
- [ ] VS Code integration working
- [ ] Mixed sessions (web + VS Code) stable
- [ ] Deployed to accessible server
- [ ] 5+ users tested successfully
- [ ] Documentation complete

### 15.3 Version 2.0
- [ ] All above plus:
- [ ] Neovim integration working
- [ ] All three editors in same session
- [ ] Co-author attribution
- [ ] Auto-generated commit messages
- [ ] Keyboard customization UI

### 15.4 Revenue Target
- [ ] Hosted service launched
- [ ] Stripe billing working
- [ ] 10+ paying teams
- [ ] Positive cash flow from tool

---

## 16. OPEN QUESTIONS (Require Decisions)

### 16.1 Architecture Decisions
1. **Server language:** Node.js, Go, or Python?
2. **CRDT library:** Stick with YJS or switch to multi-language solution?
3. **Network protocol:** WebSocket (likely) or alternatives?
4. **Cursor locking:** Initial simple approach or advanced branching/merging?

### 16.2 Scope Decisions
1. **Editor priority:** VS Code first or Neovim first? (Suggest VS Code - larger user base)
2. **Feature priority:** LLM integration before or after both editors? (Suggest after)
3. **Deployment:** Self-hosted first or try for cloud deployment? (Suggest self-hosted)
4. **Git providers:** GitHub only or support Gitea/GitLab? (Suggest GitHub only initially)

### 16.3 Resource Decisions
1. **JJ's focus:** What's the #1 priority for next 2 weeks?
2. **Help needed:** Can someone else help with docs? Server code?
3. **Testing:** Who can test multi-user sessions once ready?
4. **Code review:** Who reviews JJ's code regularly?

### 16.4 Product Decisions
1. **Name:** What's the actual product name?
2. **Target users:** Internal team only or external customers?
3. **Pricing:** Free tier? How much for paid?
4. **Support:** Who handles user support requests?

---

## 17. NEXT IMMEDIATE STEPS (Priority Order)

### For JJ (Next 1-2 weeks):

1. **Update installation documentation** (CRITICAL)
   - Step-by-step setup for web editor
   - Prerequisites clearly listed
   - Troubleshooting common issues
   - Get Steve or someone else to test following docs

2. **Schedule regular check-ins with Steve**
   - Already scheduled: Thursdays at noon
   - Use this to stay aligned (avoid Luke Kanis situation)
   - Review progress, unblock issues
   - Adjust priorities as needed

3. **Research Neovim network protocols**
   - Read Neovim docs on network capabilities
   - Study FluBits plugin code (if available)
   - Document findings
   - Assess difficulty/feasibility

4. **Deploy web editor to accessible server**
   - Not just localhost
   - So others can test
   - Even if rough/buggy
   - Enables multi-user testing

5. **Test co-author attribution**
   - Feature exists but not tested
   - Important for mob programming workflow
   - Should be quick win

### For Steve (Support JJ):

1. **Clarify priorities with JJ**
   - What's must-have vs nice-to-have?
   - What's timeline expectation?
   - Realistic scope for JJ alone vs with help?

2. **Provide server hosting option**
   - Where should JJ deploy for testing?
   - What infrastructure available?

3. **Code review JJ's work regularly**
   - Don't repeat Luke Kanis mistake
   - Stay aligned on architecture

4. **Decide on editor priority**
   - VS Code or Neovim first?
   - Affects JJ's learning path

---

## 18. REFERENCE MATERIALS

### Code References
- **Collaborative editor:** (GitHub repo, needs README update)
- **grokker:** Steve's LLM tool (commit message generation code)
- **FluBits Neovim plugin:** GitHub (defunct but may have useful code)

### Technical Documentation
- YJS Documentation: https://docs.yjs.dev/
- VS Code Extension API: https://code.visualstudio.com/api
- Neovim Lua Guide: https://neovim.io/doc/user/lua-guide.html
- GitHub REST API: https://docs.github.com/en/rest
- WebSocket Protocol: RFC 6455

### Strategic Documents
- Donaldo's Whiteboard: Sept 16 & 18, Oct 2 notes
- Your notes: Oct 2, 2025
- This transcript: Oct 2, 2025

---

## 19. GLOSSARY

- **CRDT:** Conflict-free Replicated Data Type - algorithm for sync without conflicts
- **YJS:** JavaScript CRDT library currently used in web editor
- **Mob programming:** Team programming where everyone works on same thing simultaneously
- **Neovim:** Modern fork of Vim editor with network capabilities
- **FluBits:** Defunct service that attempted multi-editor collaboration (existence proof)
- **Promisebase:** Steve's grid infrastructure project (container orchestration, etc.)
- **Grokker:** Steve's LLM integration tool
- **Dog-fooding:** Using your own tool to build itself (validation strategy)
- **Multi-cursor:** Multiple people's cursors visible/active in same document
- **Cursor locking:** Temporarily freezing all editing during git operations

---

## APPENDIX: Why This Is Hard (But Doable)

This is genuinely a complex project. Your nervousness is appropriate - it means you understand the scope. Here's what makes it hard:

**Complex Integration Points:**
- 3 different editors with different APIs
- Real-time networking with latency management
- CRDT math for conflict-free editing
- Git operations requiring atomic transactions
- Plugin development in unfamiliar environments (VS Code extensions, Lua)

**Unknown Territory:**
- No one on team has done this exact thing before
- FluBits failed (though they had different constraints)
- Neovim plugin development is new to you
- VS Code extensions are new to you
- Server architecture for real-time collaboration is new

**High Expectations:**
- Boss wants all three editors working
- Strategic importance (replaces multiple tools)
- Revenue pressure (needs to be good enough to sell)
- "Killer app" positioning for grid platform

**BUT:** This is also very doable:

**You Have Proof It Works:**
- GitHub integration already done (big win)
- YJS already handling CRDT (hard part done)
- FluBits proved multi-editor is possible
- Git operations are well-understood
- WebSocket is mature technology

**You Can Build Incrementally:**
- Web editor multi-user first (foundation)
- Then add one local editor
- Then add second local editor
- Then add LLM agents
- Each step is value-adding

**You Have Support:**
- Steve is engaged (regular Thursday meetings)
- Team wants this (it helps everyone)
- Open source model means community help possible
- Can use LLMs to help with unfamiliar code

**The Key:** Break it into phases. Don't try to do everything at once. Get web editor solid first, then add one editor at a time.

You've got this. Your nervousness shows you understand the challenge, which is actually the first step to succeeding.

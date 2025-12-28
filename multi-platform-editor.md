# Multi-Platform Collaborative Editor Analysis

**Prepared for:** Project Leadership  
**Prepared by:** Research Team  
**Date:** December 27, 2025  
**Subject:** Evaluation of Collaborative Editing Solutions for Web, VS Code, and Neovim

---

## Executive Summary

This document analyzes collaborative editing solutions that support real-time multi-user editing across three critical platforms: **Web browsers**, **VS Code**, and **Neovim**. The goal is to identify solutions compatible with our PromiseGrid integration strategy while supporting cursor awareness, user presence, and cross-platform collaboration.

**Key Finding:** Only **two solutions** currently offer official support for all three platforms, with varying degrees of maturity and licensing considerations.

---

## Table of Contents

1. [Platform Requirements](#platform-requirements)
2. [Comprehensive Comparison Table](#comprehensive-comparison-table)
3. [Detailed Project Analysis](#detailed-project-analysis)
4. [Protocol Technology Overview](#protocol-technology-overview)
5. [Cross-Platform Support Matrix](#cross-platform-support-matrix)
6. [Integration Assessment](#integration-assessment)
7. [Recommendations](#recommendations)
8. [Action Items](#action-items)

---

## Platform Requirements

### Must-Have Features

**Cross-Platform Support:**
- ✅ Web browser support (Chrome, Firefox, Safari)
- ✅ VS Code / VSCodium extension
- ✅ Neovim plugin

**Collaboration Features:**
- ✅ Real-time text synchronization
- ✅ Cursor position tracking
- ✅ User presence awareness (who's online)
- ✅ User identification (names/colors)
- ✅ Selection highlighting

**Technical Requirements:**
- ✅ Conflict resolution (CRDT or OT)
- ✅ Network-agnostic messaging
- ✅ PromiseGrid integration potential
- ✅ Active maintenance and community

**Nice-to-Have:**
- Offline support
- Peer-to-peer capability
- Open source
- TypeScript/Rust/Go implementation

---

## Comprehensive Comparison Table

| Project | Protocol | VS Code | Neovim | Web | YJS? | Status | Dev Language | Network Arch | Cursor/Presence | License | Last Updated |
|---------|----------|---------|--------|-----|------|--------|--------------|--------------|-----------------|---------|--------------|
| **TeamType** | Automerge CRDT | ✅ | ✅ | 🚧 | ❌ | ✅ Active | Rust + Lua + TS | P2P encrypted | ✅ Full | AGPL-3.0 | Oct 2025 |
| **Floobits** | Proprietary | ✅ | ✅ | ✅ | ❌ | ✅ Commercial | Unknown | Cloud service | ✅ Full | Commercial | Active |
| **Nomad** | Custom CRDT | ❌ | ✅ | ❌ | ❌ | ✅ Active | Rust + Lua | P2P sessions | ✅ Full | MIT | Nov 2025 |
| **instant.nvim** | Logoot CRDT | ❌ | ✅ | ❌ | ❌ | ⚠️ Unmaintained | Lua | Server broadcast | ✅ Basic | MIT | Sept 2021 |
| **livecode.nvim** | Operational Transform | ❌ | ✅ | ❌ | ❌ | ✅ Active | Lua | Local server | ✅ Username | Unknown | Active |
| **CodeTogether** | Proprietary | ✅ | ❌ | ✅ | ❌ | ✅ Commercial | Unknown | Cloud service | ✅ Full | Commercial | Active |
| **Live Share** (Microsoft) | Proprietary | ✅ | ❌ | ❌ | ❌ | ✅ Active | Unknown | Cloud service | ✅ Full | Proprietary | Active |
| **Our Current Editor** | YJS (YATA CRDT) | ❌ | ❌ | ✅ | ✅ | ✅ Active | JS/Rust WASM | WebSocket | ✅ Full | Unknown | Dec 2025 |

**Legend:**
- ✅ = Fully supported and maintained
- 🚧 = Community work-in-progress (not official)
- ❌ = Not supported
- ⚠️ = Exists but unmaintained

---

## Detailed Project Analysis

### 1. TeamType (formerly Ethersync)

**GitHub:** https://github.com/teamtype/teamtype  
**Website:** https://teamtype.github.io/teamtype/

**Overview:**
Peer-to-peer, editor-agnostic collaborative editing with official support for Neovim and VS Code. Uses Automerge CRDT for conflict-free synchronization.

**Platform Support:**
- ✅ **VS Code**: Official "Teamtype" extension in marketplace
- ✅ **Neovim**: Official plugin at `github.com/teamtype/teamtype-nvim`
- 🚧 **Web**: Community project by @winniehell at `github.com/winniehell/ethersync-web`

**Technical Details:**
- **Protocol**: Automerge CRDT (Conflict-free Replicated Data Type)
- **Language**: Rust (daemon) + Lua (Neovim) + TypeScript (VS Code)
- **Architecture**: Peer-to-peer with encrypted connections via Iroh
- **Transport**: Uses Magic Wormhole for initial connection, then P2P
- **Storage**: Local-first design with full offline support

**Features:**
- ✅ Real-time cursor positions and selections
- ✅ User presence with color coding
- ✅ Whole directory synchronization (not just single files)
- ✅ No central server required
- ✅ End-to-end encryption
- ✅ Simple JSON-RPC protocol for editor plugins
- ✅ Join codes for easy connection (e.g., "5-hamburger-endorse")

**Strengths:**
- Open source (AGPL-3.0)
- Active development and community
- Strong privacy/security (P2P + encryption)
- Well-documented protocol
- Funded by NLNet and Prototype Fund
- Rust codebase facilitates PromiseGrid integration

**Weaknesses:**
- Web editor is community WIP (not official)
- AGPL-3.0 license (copyleft - may affect commercial use)
- Smaller user base than commercial alternatives

**PromiseGrid Integration Potential:** ⭐⭐⭐⭐⭐
- Rust codebase makes Go integration straightforward
- P2P architecture aligns with PromiseGrid philosophy
- Pluggable transport layer could route through PromiseGrid nodes
- Automerge messages are well-defined and could be wrapped in PromiseGrid CBOR

---

### 2. Floobits

**Website:** https://floobits.com

**Overview:**
Commercial collaborative editing service with plugins for multiple editors. Mature, production-ready solution.

**Platform Support:**
- ✅ **VS Code**: Official extension
- ✅ **Neovim**: Official plugin (`floobits-neovim`)
- ✅ **Web**: Full browser-based editor at floobits.com
- Also supports: Emacs, Sublime Text, IntelliJ IDEA

**Technical Details:**
- **Protocol**: Proprietary
- **Language**: Unknown (closed source)
- **Architecture**: Cloud-based service
- **Authentication**: Requires account creation

**Features:**
- ✅ Full cursor and selection tracking
- ✅ Terminal sharing (Flootty)
- ✅ Video chat integration (Google+ Hangouts)
- ✅ Screen sharing
- ✅ Cross-editor compatibility tested at scale

**Strengths:**
- Complete coverage of all three platforms
- Production-tested and stable
- Additional features (video, terminals)
- Wide editor support

**Weaknesses:**
- Commercial service (not open source)
- Requires account and internet connection
- Proprietary protocol (vendor lock-in)
- Cloud dependency (not local-first)
- Unknown pricing model

**PromiseGrid Integration Potential:** ⭐⭐
- Closed source makes integration difficult
- Cloud-dependent architecture conflicts with P2P goals
- Proprietary protocol cannot be wrapped/extended

---

### 3. Nomad

**GitHub:** https://github.com/nomad/nomad

**Overview:**
Modern collaborative editing for Neovim with custom-built CRDT implementation. Focus on making coding "delightful."

**Platform Support:**
- ❌ **VS Code**: Not supported
- ✅ **Neovim**: Primary platform with full integration
- ❌ **Web**: Planned but not implemented

**Technical Details:**
- **Protocol**: Custom CRDT libraries (cola, puff, pando)
- **Language**: Rust (core) + Lua (plugin)
- **Architecture**: P2P sessions
- **Commands**: `:Mad collab start` / `:Mad collab join <session_id>`

**Features:**
- ✅ Real-time cursor positions and selections
- ✅ User awareness and presence
- ✅ Session-based collaboration
- ✅ Modern Rust implementation
- 📅 Long-term goal: cross-editor and asynchronous collaboration

**Strengths:**
- Clean, modern codebase
- Active development (Nov 2025 release)
- MIT license (permissive)
- Custom CRDT optimized for text editing
- Strong Neovim integration

**Weaknesses:**
- Neovim-only currently
- No web or VS Code support
- Smaller community
- Cross-editor support is future roadmap, not current reality

**PromiseGrid Integration Potential:** ⭐⭐⭐⭐
- Rust codebase (easy Go integration)
- P2P architecture compatible
- Would need VS Code + web implementations first

---

### 4. instant.nvim

**GitHub:** https://github.com/jbyuki/instant.nvim

**Overview:**
Pure Lua collaborative editing plugin for Neovim. Historically influential but no longer maintained.

**Platform Support:**
- ❌ **VS Code**: Not supported
- ✅ **Neovim**: Pure Lua implementation
- ❌ **Web**: Not supported

**Technical Details:**
- **Protocol**: Logoot CRDT
- **Language**: Pure Lua (no dependencies)
- **Architecture**: Central broadcast server (server stores no text)
- **Messages**: JSON-encoded

**Features:**
- ✅ Basic cursor tracking
- ✅ User awareness
- ✅ Session sharing mode (multiple buffers)
- ✅ Built-in server for localhost/LAN

**Strengths:**
- No dependencies
- Simple protocol (JSON)
- Inspired many other projects

**Weaknesses:**
- ⚠️ **No longer maintained** (last update Sept 2021)
- Neovim-only
- Central server architecture (not P2P)

**PromiseGrid Integration Potential:** ⭐⭐
- Unmaintained status is major concern
- Lua codebase requires FFI bridge to Go
- Would need significant updates

---

### 5. livecode.nvim

**GitHub:** https://github.com/jxm35/livecode.nvim

**Overview:**
Live collaborative coding for Neovim using Operational Transformation.

**Platform Support:**
- ❌ **VS Code**: Not supported
- ✅ **Neovim**: Pure Lua implementation
- ❌ **Web**: Not supported

**Technical Details:**
- **Protocol**: Operational Transformation (OT)
- **Language**: 100% Lua
- **Architecture**: Built-in local network server

**Features:**
- ✅ Username support
- ✅ Live text editing
- ✅ Local network sessions

**Strengths:**
- Pure Lua (no dependencies)
- Active maintenance
- Fast OT algorithm

**Weaknesses:**
- Neovim-only
- OT requires central coordination (vs CRDT's P2P)
- Smaller feature set than competitors

**PromiseGrid Integration Potential:** ⭐⭐
- Lua-only codebase
- OT architecture less suitable for P2P
- Would need complete rewrite for cross-platform

---

### 6. CodeTogether

**Website:** https://www.codetogether.com/

**Overview:**
Commercial collaborative coding platform with cross-IDE support.

**Platform Support:**
- ✅ **VS Code**: Official extension
- ❌ **Neovim**: Not supported
- ✅ **Web**: Browser-based editor
- Also supports: IntelliJ, Eclipse

**Technical Details:**
- **Protocol**: Proprietary
- **Architecture**: Cloud-based with on-premises options
- **Pricing**: Free tier + paid plans

**Features:**
- ✅ Real-time collaboration
- ✅ Voice/video integration
- ✅ Terminal sharing
- ✅ Enterprise features (SSO, audit logs)

**Strengths:**
- Professional, production-ready
- Good IDE support
- Enterprise features available

**Weaknesses:**
- No Neovim support
- Commercial/proprietary
- Requires cloud connectivity

**PromiseGrid Integration Potential:** ⭐
- Closed source
- Missing Neovim support (critical for our needs)
- Cloud dependency

---

### 7. Microsoft Live Share

**Website:** https://visualstudio.microsoft.com/services/live-share/

**Overview:**
Microsoft's official collaborative coding solution for VS Code.

**Platform Support:**
- ✅ **VS Code**: Native integration
- ❌ **Neovim**: Not supported
- ❌ **Web**: Limited (VS Code for Web support)

**Technical Details:**
- **Protocol**: Proprietary
- **Architecture**: Microsoft cloud services
- **Integration**: Deep VS Code integration

**Features:**
- ✅ Best-in-class VS Code integration
- ✅ Terminal sharing
- ✅ Server sharing
- ✅ Audio calls
- ✅ Debugging support

**Strengths:**
- Excellent VS Code user experience
- Microsoft backing and support
- Production-tested at scale

**Weaknesses:**
- VS Code exclusive
- No Neovim support (dealbreaker)
- Microsoft cloud dependency
- Proprietary protocol

**PromiseGrid Integration Potential:** ⭐
- No Neovim support eliminates from consideration
- Closed ecosystem

---

### 8. Our Current Editor

**Status:** Internal implementation

**Platform Support:**
- ❌ **VS Code**: Not implemented
- ❌ **Neovim**: Not implemented
- ✅ **Web**: Full implementation with CodeMirror 6

**Technical Details:**
- **Protocol**: YJS (YATA CRDT algorithm)
- **Language**: JavaScript + Rust WASM
- **Architecture**: WebSocket-based with IndexedDB persistence
- **Features**: PromiseGrid CBOR message generation

**Strengths:**
- Already built and working
- YJS is proven and well-documented
- PromiseGrid integration already started
- Team has expertise with codebase

**Weaknesses:**
- Web-only (missing 2/3 platforms)
- Would require building VS Code extension
- Would require building Neovim plugin
- YJS in Neovim is unusual (no existing examples)

**Path Forward:**
Could extend to VS Code and Neovim, but significant development effort required (estimated 6-12 weeks for both platforms).

---

## Protocol Technology Overview

### CRDT (Conflict-free Replicated Data Types)

**Used by:** TeamType (Automerge), Nomad (custom), instant.nvim (Logoot), YJS

**How it works:**
- Each character has a unique identifier
- Operations are commutative (order doesn't matter)
- No central coordination required
- Eventual consistency guaranteed

**Advantages:**
- ✅ Peer-to-peer capable
- ✅ Offline-first friendly
- ✅ No central server needed
- ✅ Guaranteed convergence

**Disadvantages:**
- Larger memory footprint (must track tombstones)
- More complex implementation
- Can suffer from interleaving anomalies

**Best for:** Distributed systems, offline-first apps, P2P networks

---

### Operational Transformation (OT)

**Used by:** livecode.nvim, Google Docs (historically)

**How it works:**
- Operations are position-based (insert at position N)
- Operations must be transformed based on concurrent changes
- Typically requires central server for ordering

**Advantages:**
- ✅ Smaller memory footprint
- ✅ Simpler data model
- ✅ Preserves user intent well

**Disadvantages:**
- ⚠️ Requires central coordination (usually)
- ⚠️ Complex transformation functions
- ⚠️ Difficult to prove correctness

**Best for:** Client-server architectures, centralized systems

---

### YJS (YATA Algorithm)

**Specific CRDT implementation used by our current editor**

**Advantages:**
- Excellent performance (O(log n) operations)
- Small network messages
- Good compression
- Well-tested in production
- Rich ecosystem (providers, persistence adapters)

**Considerations for adoption:**
- Popular in web apps, less common in editor plugins
- Would require bridging to Neovim/VS Code
- No existing Neovim plugins use YJS (unusual choice)

---

### Automerge

**Specific CRDT implementation used by TeamType**

**Advantages:**
- Columnar encoding (efficient storage)
- 1.5-2x size overhead (very good for CRDTs)
- Built for local-first applications
- Strong convergence guarantees
- JSON-like data model

**Considerations:**
- Different from YJS but similar philosophy
- Well-documented and actively maintained
- Growing ecosystem

---

## Cross-Platform Support Matrix

### Complete Coverage (All 3 Platforms)

| Project | VS Code | Neovim | Web | Official Support | Open Source | Status |
|---------|---------|--------|-----|------------------|-------------|--------|
| **Floobits** | ✅ | ✅ | ✅ | All official | ❌ Commercial | Production |
| **TeamType** | ✅ | ✅ | 🚧 | VS Code + Neovim official, Web community | ✅ AGPL-3.0 | Active, web WIP |

---

### Partial Coverage

| Project | Supported Platforms | Coverage | Best Use Case |
|---------|-------------------|----------|---------------|
| **Our Editor** | Web only | 1/3 | Web-based teams |
| **Nomad** | Neovim only | 1/3 | Neovim-focused teams |
| **Live Share** | VS Code only | 1/3 | VS Code-exclusive teams |
| **CodeTogether** | VS Code + Web | 2/3 | No Neovim requirement |
| **instant.nvim** | Neovim only | 1/3 | Legacy Neovim (unmaintained) |
| **livecode.nvim** | Neovim only | 1/3 | Neovim-focused teams |

---

### Platform-Specific Rankings

#### **VS Code Support**
1. **Live Share** - Native Microsoft integration
2. **Floobits** - Production-ready commercial
3. **TeamType** - Open source, actively maintained
4. **CodeTogether** - Commercial alternative

#### **Neovim Support**
1. **Nomad** - Best Neovim integration (Rust + Lua)
2. **TeamType** - Official plugin, well-maintained
3. **livecode.nvim** - Pure Lua, active
4. **Floobits** - Commercial, mature
5. **instant.nvim** - Historical significance, unmaintained

#### **Web Support**
1. **Our Editor** - Full-featured, already built
2. **Floobits** - Commercial, production-ready
3. **CodeTogether** - Commercial alternative
4. **TeamType** - Community WIP

---

## Integration Assessment

### PromiseGrid Integration Compatibility

**Evaluation Criteria:**
1. Protocol transparency and documentation
2. Language compatibility (Go, Rust preferred)
3. Network architecture (P2P preferred over cloud)
4. Message format (can it be wrapped in CBOR?)
5. Pluggable transport layer

| Project | Protocol Score | Language Score | Architecture Score | Wrapper Score | Total | Grade |
|---------|---------------|----------------|-------------------|---------------|-------|-------|
| **TeamType** | 5/5 | 5/5 | 5/5 | 5/5 | **20/20** | ⭐⭐⭐⭐⭐ |
| **Nomad** | 5/5 | 5/5 | 5/5 | 4/5 | **19/20** | ⭐⭐⭐⭐⭐ |
| **Our Editor** | 5/5 | 4/5 | 4/5 | 5/5 | **18/20** | ⭐⭐⭐⭐ |
| **livecode.nvim** | 3/5 | 2/5 | 3/5 | 3/5 | **11/20** | ⭐⭐⭐ |
| **instant.nvim** | 4/5 | 2/5 | 3/5 | 4/5 | **13/20** | ⭐⭐⭐ |
| **Floobits** | 0/5 | 0/5 | 1/5 | 0/5 | **1/20** | ⭐ |
| **Live Share** | 0/5 | 0/5 | 1/5 | 0/5 | **1/20** | ⭐ |
| **CodeTogether** | 0/5 | 0/5 | 1/5 | 0/5 | **1/20** | ⭐ |

**Key Insights:**
- Open source Rust/Go solutions score highest
- Commercial/proprietary solutions incompatible
- P2P architecture essential for PromiseGrid alignment

---

### Development Effort Estimates

#### **Option A: Adopt TeamType + Build Web Editor**

| Task | Effort | Timeline |
|------|--------|----------|
| Test TeamType (VS Code + Neovim) | 1 week | Week 1 |
| Evaluate community web editor | 1 week | Week 2 |
| Build Automerge web editor | 4-6 weeks | Weeks 3-8 |
| PromiseGrid integration | 2-3 weeks | Weeks 9-11 |
| **Total** | **8-11 weeks** | **~3 months** |

**Team Required:** 1-2 developers

---

#### **Option B: Extend Our Editor (YJS) to All Platforms**

| Task | Effort | Timeline |
|------|--------|----------|
| Build VS Code extension (YJS) | 2-4 weeks | Weeks 1-4 |
| Build Neovim plugin (Lua + YJS bridge) | 4-8 weeks | Weeks 5-12 |
| Test cross-platform | 1-2 weeks | Weeks 13-14 |
| PromiseGrid integration | Already started | Ongoing |
| **Total** | **7-14 weeks** | **~4 months** |

**Team Required:** 2-3 developers (1 for VS Code, 1-2 for Neovim)

---

#### **Option C: Use Floobits (Commercial)**

| Task | Effort | Timeline |
|------|--------|----------|
| Evaluate free tier | 1 week | Week 1 |
| Pilot with team | 2 weeks | Weeks 2-3 |
| Cost analysis | 1 week | Week 4 |
| **Total** | **4 weeks** | **~1 month** |

**Team Required:** 1 developer for evaluation

**Ongoing Costs:** Unknown (requires pricing inquiry)

**Risks:** Vendor lock-in, no PromiseGrid integration possible

---

## Recommendations

### Primary Recommendation: TeamType + Web Enhancement

**Why TeamType:**
1. ✅ **2/3 platforms official** (VS Code + Neovim)
2. ✅ **Open source** (AGPL-3.0) - full control
3. ✅ **Rust codebase** - Easy PromiseGrid integration
4. ✅ **P2P architecture** - Aligns with PromiseGrid philosophy
5. ✅ **Active community** - Funded by NLNet and Prototype Fund
6. ✅ **Automerge CRDT** - Proven technology (similar to YJS)
7. ✅ **Well-documented** - Editor plugin development guide available
8. 🚧 **Web editor WIP** - Community project exists as starting point

**Implementation Path:**
1. **Week 1-2:** Test TeamType with VS Code and Neovim
   - Set up development environment
   - Test with team members
   - Evaluate user experience
   
2. **Week 3-4:** Evaluate community web editor
   - Clone `github.com/winniehell/ethersync-web`
   - Assess maturity and feature completeness
   - Determine if it meets our needs or requires enhancement
   
3. **Week 5-10:** Web editor development (if needed)
   - Option A: Contribute to community project
   - Option B: Build new Automerge web editor using our YJS editor as template
   - Implement cursor tracking, presence, and sync
   
4. **Week 11-13:** PromiseGrid integration
   - Design Automerge ↔ PromiseGrid CBOR wrapper
   - Implement transport layer
   - Test cross-platform with PromiseGrid

**Total Timeline:** ~3 months  
**Team Size:** 1-2 developers  
**Risk Level:** Low-Medium

---

### Alternative Recommendation: Extend Our Editor

**Why Our Editor:**
1. ✅ **Already built and working** (web platform)
2. ✅ **Team expertise** with YJS and codebase
3. ✅ **PromiseGrid integration** already in progress
4. ✅ **Full control** over implementation
5. ⚠️ **Requires VS Code extension** (~2-4 weeks)
6. ⚠️ **Requires Neovim plugin** (~4-8 weeks)

**Implementation Path:**
1. **Week 1-4:** Build VS Code extension
   - Use YJS provider for VS Code
   - Implement cursor/presence features
   - Package as VS Code extension
   
2. **Week 5-12:** Build Neovim plugin
   - Design Lua wrapper for YJS
   - May require Rust/Go bridge for YJS library
   - Implement Neovim API integration
   - Test with various Neovim configurations
   
3. **Week 13-14:** Cross-platform testing
   - Test Web ↔ VS Code ↔ Neovim collaboration
   - Performance optimization
   - Bug fixes

**Total Timeline:** ~4 months  
**Team Size:** 2-3 developers  
**Risk Level:** Medium

**Challenges:**
- YJS in Neovim is uncommon (no existing examples to reference)
- Building Neovim plugin from scratch
- Maintaining three separate codebases

---

### Not Recommended: Commercial Solutions

**Why not Floobits/CodeTogether/Live Share:**
- ❌ **PromiseGrid integration impossible** (proprietary/closed)
- ❌ **Vendor lock-in** concerns
- ❌ **Ongoing costs** (unknown pricing)
- ❌ **Cloud dependency** conflicts with local-first goals
- ⚠️ **Missing platforms** (Live Share/CodeTogether lack Neovim)

**Use Case:** Only if PromiseGrid integration is deprioritized and budget allows.

---

## Action Items

### Immediate Next Steps (Week 1)

1. **Technical Evaluation**
   - [ ] Clone and test TeamType with 2-3 team members
   - [ ] Install Neovim plugin and VS Code extension
   - [ ] Set up test collaboration session
   - [ ] Document user experience and pain points

2. **Community Assessment**
   - [ ] Review TeamType community web editor code
   - [ ] Join TeamType discussions/Discord/Matrix
   - [ ] Assess responsiveness and support

3. **License Review**
   - [ ] Review AGPL-3.0 implications for our project
   - [ ] Consult with legal if necessary
   - [ ] Document any license concerns

### Short-Term (Weeks 2-4)

4. **Web Editor Evaluation**
   - [ ] Deploy community web editor in test environment
   - [ ] Evaluate feature completeness
   - [ ] Identify gaps vs. our requirements
   - [ ] Estimate effort to enhance or build new

5. **Protocol Analysis**
   - [ ] Study Automerge message format
   - [ ] Compare with YJS wire protocol (for your presentation)
   - [ ] Design PromiseGrid CBOR wrapper for Automerge
   - [ ] Prototype basic integration

6. **Decision Point**
   - [ ] Compile findings from testing
   - [ ] Present to stakeholders
   - [ ] Make go/no-go decision on TeamType adoption
   - [ ] If no-go, pivot to extending our YJS editor

### Medium-Term (Months 2-3)

7. **Implementation**
   - [ ] Begin web editor development (if needed)
   - [ ] Implement PromiseGrid transport layer
   - [ ] Internal testing with team
   - [ ] Performance benchmarking

8. **Documentation**
   - [ ] Create internal setup guide
   - [ ] Document PromiseGrid integration
   - [ ] Write troubleshooting guide
   - [ ] Prepare user training materials

---

## Additional Resources

### TeamType Resources
- **Main Repository:** https://github.com/teamtype/teamtype
- **Documentation:** https://teamtype.github.io/teamtype/
- **Neovim Plugin:** https://github.com/teamtype/teamtype-nvim
- **Community Web Editor:** https://github.com/winniehell/ethersync-web
- **FOSDEM Talk (10 min):** https://fosdem.org/2025/schedule/event/fosdem-2025-4890-ethersync-real-time-collaboration-in-your-text-editor-/

### Protocol Documentation
- **Automerge:** https://automerge.org
- **YJS:** https://docs.yjs.dev
- **CRDT Overview:** https://crdt.tech
- **Operational Transform FAQ:** https://github.com/Operational-Transformation/ot.py/wiki/FAQ

### Similar Projects for Reference
- **Nomad:** https://github.com/nomad/nomad
- **instant.nvim:** https://github.com/jbyuki/instant.nvim
- **livecode.nvim:** https://github.com/jxm35/livecode.nvim

---

## Appendix A: Protocol Comparison for Presentation

### YJS vs Automerge Wire Format

**YJS (YATA CRDT):**
- Binary encoding optimized for size
- Update messages contain operations (insert/delete)
- State vectors for synchronization
- Efficient delta updates
- Average message size: ~100-500 bytes for typical edits

**Automerge:**
- Columnar encoding (v2+)
- JSON-like data model
- 1.5-2x size overhead vs raw text
- Efficient compression
- Change-based synchronization

**PromiseGrid Integration:**
Both protocols can be wrapped in PromiseGrid CBOR messages:
```
PromiseGrid Message {
  protocol_hash: "QmPromiseGridProtocolV1",
  message_type: "collab_edit",
  payload: {
    crdt_type: "automerge" | "yjs",
    operations: <binary CRDT operations>,
    metadata: {
      user_id: "...",
      timestamp: "...",
      document_id: "..."
    }
  }
}
```

---

## Appendix B: Platform-Specific Implementation Notes

### Neovim Plugin Architecture

**Key APIs:**
- `nvim_buf_attach()` - Track buffer changes
- `nvim_buf_set_extmark()` - Display cursors/selections
- `nvim_create_autocmd()` - React to events

**Best Practices:**
- Use Lua for performance
- Async operations via `vim.loop`
- Minimal overhead on edit operations

### VS Code Extension Architecture

**Key APIs:**
- `TextDocumentContentProvider` - Sync document state
- `DecorationOptions` - Display cursors/selections
- `workspace.onDidChangeTextDocument` - Track edits

**Best Practices:**
- TypeScript for type safety
- Use WebSockets for networking
- Leverage VS Code's built-in diff view

### Web Editor Architecture

**Recommended Stack:**
- CodeMirror 6 (already using) or Monaco Editor
- YJS or Automerge for CRDT
- WebSocket or WebRTC for transport
- IndexedDB for offline persistence

---

## Appendix C: Glossary

**CRDT (Conflict-free Replicated Data Type):** Data structure that can be replicated across multiple nodes and updated independently without coordination, guaranteeing eventual consistency.

**Operational Transformation (OT):** Algorithm for maintaining consistency in collaborative editing by transforming operations based on concurrent changes.

**P2P (Peer-to-Peer):** Network architecture where nodes communicate directly without a central server.

**Automerge:** CRDT implementation optimized for JSON-like data structures and local-first applications.

**YJS:** CRDT implementation using the YATA algorithm, popular for collaborative text editing.

**PromiseGrid:** (Your project's) decentralized computing protocol using content-addressed messages and CBOR encoding.

**CBOR (Concise Binary Object Representation):** Binary data serialization format similar to JSON but more compact.

**Logoot:** CRDT algorithm that assigns position identifiers to characters for conflict-free insertion.

**WebRTC:** Protocol for peer-to-peer communication in browsers.

**Magic Wormhole:** Protocol for secure peer discovery and connection establishment.

**Iroh:** Rust library for building peer-to-peer applications.

---

## Document Metadata

**Version:** 1.0  
**Last Updated:** December 27, 2025  
**Next Review:** After initial TeamType evaluation (Week 2)  
**Contact:** Research Team  
**Related Documents:**
- Task 2.2: Review Neovim Collaborative Projects
- YJS Wire Protocol Research
- PromiseGrid Integration Specification

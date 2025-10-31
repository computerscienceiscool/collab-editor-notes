# Collaborative Editor PromiseGrid/PromiseBase Integration Specification

## Overview

This specification outlines features that can be added to the collaborative text editor to enhance its integration with PromiseGrid protocol and PromiseBase content-addressable storage, enabling true decentralized document collaboration.

## Current State

The collaborative editor already implements:
- Real-time collaboration via Yjs CRDTs
- PromiseGrid CBOR message generation with official 'grid' tag (0x67726964)
- Offline support with IndexedDB persistence
- GitHub integration for version control
- WASM-powered text processing
- User awareness and presence tracking

## Proposed Features

### 1. Enhanced PromiseGrid Message Types

**Status**: Extension of existing CBOR implementation  
**Priority**: High  
**Effort**: Medium  

Currently, the editor generates basic edit messages. Expand the message types to support full decentralized collaboration:

#### 1.1 Presence Messages
```javascript
createPresenceMessage(userId, cursorPosition, selection, status)
```
- User cursor positions and selections
- Online/offline status
- Current editing focus area
- Typing indicators

#### 1.2 Comment & Annotation Messages
```javascript
createCommentMessage(docId, position, comment, userId, threadId)
createAnnotationMessage(docId, range, annotation, type, userId)
```
- Inline comments at specific positions
- Text annotations and highlights
- Comment threads and replies
- Annotation types: note, suggestion, correction

#### 1.3 Permission & Access Control Messages
```javascript
createPermissionMessage(docId, userId, permissions, grantor)
createAccessRequestMessage(docId, userId, requestedLevel)
```
- Document access control lists
- Permission delegation
- Access request workflow
- Role-based permissions (read, write, admin)

#### 1.4 Document Lifecycle Messages
```javascript
createDocumentMetadata(title, authors, created, modified, tags)
createDocumentStateMessage(docId, version, fingerprint, chunkInfo)
createDocumentForkMessage(originalDocId, newDocId, userId)
```
- Document creation and metadata
- Version tracking and fingerprinting
- Document forking and branching
- Content addressing preparation

### 2. Document Identity & Fingerprinting

**Status**: New feature  
**Priority**: High  
**Effort**: Low  

#### 2.1 Content-Based Document Identity
```javascript
class DocumentIdentity {
    generateFingerprint(content): string
    trackContentChanges(delta): void
    createDocumentCID(): string
    verifyDocumentIntegrity(): boolean
}
```

**Implementation**:
- Use existing WASM hash functions for content fingerprinting
- Generate document identifiers based on content hash
- Prepare for future content-addressable storage
- Track document evolution through fingerprint changes

#### 2.2 Document Registry
```javascript
class PromiseGridDocumentRegistry {
    announceDocument(docId, metadata): void
    discoverDocuments(query): Promise<DocumentInfo[]>
    subscribeToDocument(docId): void
    unsubscribeFromDocument(docId): void
}
```

**Features**:
- Announce document availability to PromiseGrid network
- Discover documents by title, author, tags
- Subscribe to document change notifications
- Maintain local registry of accessible documents

### 3. Distributed User Identity & Reputation

**Status**: Extension of existing user system  
**Priority**: Medium  
**Effort**: Medium  

#### 3.1 Cryptographic User Identity
```javascript
class DistributedUser {
    generateUserId(): string  // crypto-based persistent ID
    generateKeyPair(): KeyPair
    signMessage(message): Signature
    verifyUserSignature(message, signature, userId): boolean
}
```

**Implementation**:
- Use WebCrypto API for key generation
- Store user credentials in localStorage
- Sign all PromiseGrid messages
- Verify signatures from other users

#### 3.2 User Reputation System
```javascript
class UserReputation {
    trackUserActivity(userId, action): void
    calculateReputation(userId): number
    reportUserBehavior(userId, report): void
    getUserTrustLevel(userId): string
}
```

**Features**:
- Track positive contributions (helpful edits, good comments)
- Calculate reputation scores
- Enable community-based moderation
- Trust-based permission delegation

### 4. Offline-First PromiseGrid Message Queue

**Status**: Extension of existing offline support  
**Priority**: High  
**Effort**: Low  

#### 4.1 Message Queuing System
```javascript
class PromiseGridQueue {
    queueMessage(message): void
    flushPendingMessages(): Promise<void>
    handleNetworkStateChange(): void
    prioritizeMessages(): void
}
```

**Implementation**:
- Queue PromiseGrid messages when offline
- Store in IndexedDB alongside existing Yjs data
- Intelligent message prioritization
- Conflict resolution for queued messages

#### 4.2 Network State Management
```javascript
class NetworkManager {
    detectNetworkState(): NetworkState
    managePeerConnections(): void
    handleReconnection(): void
    optimizeMessageDelivery(): void
}
```

**Features**:
- Detect online/offline state changes
- Manage connections to PromiseGrid peers
- Handle network reconnection gracefully
- Optimize message delivery based on network conditions

### 5. Advanced Collaboration Features

**Status**: New features  
**Priority**: Medium  
**Effort**: High  

#### 5.1 Collaborative Comments System
```javascript
class CollaborativeComments {
    addComment(position, text, threadId?): void
    replyToComment(commentId, text): void
    resolveComment(commentId): void
    displayComments(): void
}
```

**Features**:
- Position-anchored comments that survive document changes
- Threaded comment discussions
- Comment resolution workflow
- Visual comment indicators in editor

#### 5.2 Document Branching & Merging
```javascript
class DocumentBranching {
    createBranch(docId, branchName): string
    mergeBranch(sourceId, targetId): MergeResult
    listBranches(docId): BranchInfo[]
    resolveMergeConflicts(conflicts): void
}
```

**Features**:
- Git-like branching for collaborative documents
- Automatic and manual merge strategies
- Conflict resolution interface
- Branch visualization and management

#### 5.3 Distributed Search & Discovery
```javascript
class DistributedSearch {
    indexDocument(docId, content, metadata): void
    searchDocuments(query): Promise<SearchResult[]>
    subscribeToSearchUpdates(query): void
    shareSearchIndex(): void
}
```

**Features**:
- Full-text search across distributed documents
- Collaborative search index sharing
- Real-time search result updates
- Privacy-preserving search options

### 6. PromiseBase Preparation Features

**Status**: Future preparation  
**Priority**: Low  
**Effort**: Variable  

#### 6.1 Content Addressing Foundation
```javascript
class ContentAddressing {
    generateCID(content): string
    prepareChunkMetadata(chunks): ChunkMetadata[]
    createMerkleTree(chunks): TreeStructure
    verifyContentIntegrity(cid, content): boolean
}
```

**Implementation**:
- Prepare document chunking metadata
- Generate IPFS-compatible CIDs
- Create Merkle tree structures
- Foundation for future PromiseBase integration

#### 6.2 Distributed Storage Interface
```javascript
interface DistributedStorage {
    storeContent(cid, content): Promise<void>
    retrieveContent(cid): Promise<content>
    pinContent(cid): Promise<void>
    unpinContent(cid): Promise<void>
}
```

**Features**:
- Abstract interface for distributed storage
- Pluggable storage backends
- Content pinning and lifecycle management
- Preparation for PromiseBase integration

### 7. Network Status & Peer Management

**Status**: Extension of existing UI  
**Priority**: Medium  
**Effort**: Low  

#### 7.1 Network Status Display
```javascript
class NetworkStatusUI {
    updatePeerCount(count): void
    showNetworkHealth(metrics): void
    displayMessageQueue(queueSize): void
    showConnectionStatus(status): void
}
```

**Features**:
- Real-time peer count display
- Network latency and health indicators
- Message queue status
- Connection quality visualization

#### 7.2 Peer Management
```javascript
class PeerManager {
    discoverPeers(): Promise<PeerInfo[]>
    connectToPeer(peerId): Promise<Connection>
    managePeerConnections(): void
    sharePeerList(): void
}
```

**Features**:
- Automatic peer discovery
- Manual peer connection management
- Peer reputation tracking
- Redundant connection handling

## Implementation Priority

### Phase 1: Core PromiseGrid Integration (Immediate)
1. Enhanced PromiseGrid message types
2. Document identity & fingerprinting
3. Offline-first message queue
4. Basic network status display

### Phase 2: Advanced Collaboration (Short-term)
1. Distributed user identity
2. Collaborative comments system
3. User reputation system
4. Advanced peer management

### Phase 3: Distributed Features (Medium-term)
1. Document branching & merging
2. Distributed search & discovery
3. Advanced access control
4. Content addressing foundation

### Phase 4: PromiseBase Integration (Long-term)
1. Full content-addressable storage
2. Distributed storage interface
3. Advanced chunking strategies
4. Complete decentralization

## Technical Considerations

### Backward Compatibility
- All new features must maintain compatibility with existing Yjs collaboration
- PromiseGrid integration should be optional and gracefully degrade
- Existing IndexedDB and GitHub integrations must continue working

### Performance Impact
- PromiseGrid message generation should not impact real-time collaboration
- Network operations must be asynchronous and non-blocking
- Offline capabilities should not increase memory usage significantly

### Security & Privacy
- All user data must be encrypted before network transmission
- User identity systems must protect privacy
- Access control must be cryptographically secure

### Testing & Validation
- Each feature requires comprehensive test coverage
- PromiseGrid message format compliance testing
- Network simulation for offline/online scenarios
- Load testing for collaboration scalability

## Success Metrics

1. **Message Generation**: Successfully create all defined PromiseGrid message types
2. **Network Resilience**: Maintain collaboration during network interruptions
3. **Scalability**: Support 10+ concurrent users per document
4. **Discovery**: Enable users to find and access distributed documents
5. **Integration**: Seamless experience between centralized and decentralized features

## Future Considerations

This specification prepares the collaborative editor for full PromiseGrid/PromiseBase integration while maintaining practical usability. The phased approach allows for incremental development and testing, ensuring each feature adds value independently while building toward a comprehensive decentralized collaboration platform.

## Appendix: Technical Implementation Notes

### PromiseGrid Message Structure Extensions

Current message structure:
```rust
pub struct PromiseGridMessage {
    protocol_hash: String,
    message_type: String,
    payload: Vec<u8>,
    timestamp: u64,
    sender: String,
}
```

Proposed extensions for collaboration:
```rust
pub struct CollaborationMessage {
    base: PromiseGridMessage,
    document_id: String,
    version: u64,
    dependencies: Vec<String>,  // Content addresses of dependencies
    signature: Option<Vec<u8>>, // Cryptographic signature
}
```

### Integration with Existing Codebase

#### WASM Function Extensions
```rust
// Add to rust-wasm/src/lib.rs
#[wasm_bindgen]
pub fn create_collaboration_message(
    doc_id: &str,
    msg_type: &str,
    payload: &str,
    user_id: &str,
    dependencies: &str
) -> Vec<u8> {
    // Implementation
}

#[wasm_bindgen]
pub fn generate_document_cid(content: &str) -> String {
    // Content addressing implementation
}
```

#### JavaScript Integration Points
```javascript
// Extend existing promiseGrid object in initWasm.js
export const promiseGrid = {
    // Existing functions...
    createEditMessage: create_promisegrid_edit_message,
    
    // New collaboration functions
    createCollaborationMessage: create_collaboration_message,
    generateDocumentCID: generate_document_cid,
    verifyMessageSignature: verify_message_signature,
    createUserIdentity: create_user_identity,
};
```

#### Database Schema Extensions
```javascript
// Extend IndexedDB schema for PromiseGrid data
const PROMISEGRID_STORES = {
    messages: 'id, timestamp, document_id, type, sender',
    documents: 'cid, title, author, created, modified',
    users: 'id, public_key, reputation, last_seen',
    permissions: 'document_id, user_id, level, granted_by',
    peers: 'id, endpoint, last_contact, trust_level'
};
```

### Development Roadmap

#### Week 1-2: Foundation
- Extend PromiseGrid message types
- Implement document fingerprinting
- Add offline message queue

#### Week 3-4: User System
- Cryptographic user identity
- Message signing and verification
- Basic reputation tracking

#### Week 5-6: Collaboration
- Collaborative comments system
- Enhanced presence tracking
- Document discovery

#### Week 7-8: Advanced Features
- Document branching
- Distributed search
- Peer management UI

#### Week 9-10: Integration & Testing
- Performance optimization
- Security auditing
- Comprehensive testing

### Testing Strategy

#### Unit Tests
- PromiseGrid message generation and parsing
- Document fingerprinting accuracy
- Cryptographic identity functions
- Offline queue persistence

#### Integration Tests
- Multi-user collaboration scenarios
- Network interruption handling
- Message ordering and consistency
- Performance under load

#### End-to-End Tests
- Complete collaboration workflows
- Document lifecycle management
- User permission scenarios
- Cross-browser compatibility

This specification provides a comprehensive roadmap for transforming the collaborative editor into a true PromiseGrid application while maintaining its current functionality and user experience.

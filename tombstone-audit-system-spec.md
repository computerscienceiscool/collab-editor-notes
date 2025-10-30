# Tombstone Tracking and Audit System Specification

## 1. Executive Summary

### 1.1 Purpose
This specification defines a comprehensive tombstone tracking and audit system for the collaborative editor, designed to meet stringent compliance requirements of national laboratories and regulated environments. The system provides complete visibility into all deleted content while maintaining real-time collaboration performance.

### 1.2 Scope
- Extract and preserve all deleted content (tombstones) from Yjs CRDTs
- Provide multiple interfaces for viewing and exporting tombstone data
- Implement cryptographic verification and chain of custody
- Ensure compliance with federal data retention requirements

### 1.3 Key Stakeholders
- **End Users**: Document collaborators requiring audit trails
- **Compliance Officers**: Personnel responsible for regulatory adherence
- **System Administrators**: IT staff managing retention and access
- **Auditors**: Internal/external reviewers requiring deletion history
- **Legal Teams**: Personnel requiring discovery and litigation support

## 2. System Architecture

### 2.1 High-Level Architecture
```
┌─────────────────────────────────────────────────────┐
│                   User Interface                     │
├─────────────────┬────────────┬─────────────────────┤
│  Editor View    │  Tombstone  │   Compliance        │
│  (Existing)     │   Viewer    │   Dashboard         │
├─────────────────┴────────────┴─────────────────────┤
│              Tombstone Management Layer              │
├─────────────────┬────────────┬─────────────────────┤
│   Extraction    │   Storage   │   Export            │
│   Engine        │   Manager   │   Generator         │
├─────────────────┴────────────┴─────────────────────┤
│                  Yjs CRDT Layer                      │
├──────────────────────────────────────────────────────┤
│           IndexedDB          │    PromiseGrid       │
└──────────────────────────────┴──────────────────────┘
```

### 2.2 Component Descriptions

#### 2.2.1 Extraction Engine
- Accesses Yjs internal structures to retrieve deleted items
- Maps tombstones to user identities and timestamps
- Maintains extraction cache for performance

#### 2.2.2 Storage Manager
- Persists tombstone data in IndexedDB
- Manages retention policies and cleanup
- Handles data compression and optimization

#### 2.2.3 Export Generator
- Produces multiple export formats (JSON, CSV, HTML, CBOR)
- Applies cryptographic signatures via PromiseGrid
- Generates compliance reports with chain of custody

## 3. Functional Requirements

### 3.1 Tombstone Extraction

#### 3.1.1 Data Capture
- **FR-1.1**: System SHALL capture ALL deleted content from Yjs documents
- **FR-1.2**: System SHALL record deletion timestamp (ISO 8601 format)
- **FR-1.3**: System SHALL identify user who performed deletion
- **FR-1.4**: System SHALL preserve original author of deleted content
- **FR-1.5**: System SHALL capture deletion context (surrounding text)
- **FR-1.6**: System SHALL calculate content hash for verification

#### 3.1.2 Performance Modes
- **FR-2.1**: System SHALL support real-time extraction mode
- **FR-2.2**: System SHALL support batch extraction mode
- **FR-2.3**: System SHALL support incremental extraction mode
- **FR-2.4**: System SHALL allow extraction throttling for large documents

### 3.2 Tombstone Viewer Interface

#### 3.2.1 Display Requirements
- **FR-3.1**: Interface SHALL display tombstones in chronological order
- **FR-3.2**: Interface SHALL show deleted content with strikethrough
- **FR-3.3**: Interface SHALL highlight tombstone context
- **FR-3.4**: Interface SHALL provide user attribution badges
- **FR-3.5**: Interface SHALL support timeline navigation

#### 3.2.2 Filtering and Search
- **FR-4.1**: Interface SHALL filter by date range
- **FR-4.2**: Interface SHALL filter by user
- **FR-4.3**: Interface SHALL search deleted content
- **FR-4.4**: Interface SHALL filter by content size
- **FR-4.5**: Interface SHALL filter by classification level

### 3.3 Document Modes

#### 3.3.1 Mode Configuration
- **FR-5.1**: System SHALL support Standard Mode (default)
- **FR-5.2**: System SHALL support Audit Mode (enhanced tracking)
- **FR-5.3**: System SHALL support Compliance Mode (full retention)
- **FR-5.4**: Mode SHALL be configurable per document
- **FR-5.5**: Mode changes SHALL be logged

#### 3.3.2 Mode Behaviors
- **FR-6.1**: Standard Mode SHALL hide tombstones from normal view
- **FR-6.2**: Audit Mode SHALL enable tombstone extraction
- **FR-6.3**: Compliance Mode SHALL prevent tombstone garbage collection
- **FR-6.4**: Mode SHALL persist across sessions

### 3.4 Export Functionality

#### 3.4.1 Format Support
- **FR-7.1**: System SHALL export to JSON with full metadata
- **FR-7.2**: System SHALL export to CSV for spreadsheet analysis
- **FR-7.3**: System SHALL export to HTML for human review
- **FR-7.4**: System SHALL export to signed CBOR via PromiseGrid
- **FR-7.5**: System SHALL support selective export (filtered data)

#### 3.4.2 Export Metadata
- **FR-8.1**: Exports SHALL include document identifier
- **FR-8.2**: Exports SHALL include export timestamp
- **FR-8.3**: Exports SHALL include user performing export
- **FR-8.4**: Exports SHALL include cryptographic signature
- **FR-8.5**: Exports SHALL include chain of custody

### 3.5 Compliance Features

#### 3.5.1 Classification Detection
- **FR-9.1**: System SHALL detect potential PII in tombstones
- **FR-9.2**: System SHALL detect classification markers
- **FR-9.3**: System SHALL flag financial data
- **FR-9.4**: System SHALL identify ITAR/export-controlled content
- **FR-9.5**: System SHALL generate classification summary

#### 3.5.2 Chain of Custody
- **FR-10.1**: System SHALL track all exports
- **FR-10.2**: System SHALL record access to tombstone viewer
- **FR-10.3**: System SHALL maintain tamper-evident logs
- **FR-10.4**: System SHALL support legal hold flags
- **FR-10.5**: System SHALL generate custody reports

## 4. Non-Functional Requirements

### 4.1 Performance
- **NFR-1.1**: Extraction SHALL complete within 5 seconds for documents <10MB
- **NFR-1.2**: Viewer SHALL render within 2 seconds for <1000 tombstones
- **NFR-1.3**: Export SHALL complete within 10 seconds for <10000 tombstones
- **NFR-1.4**: System SHALL NOT degrade editor responsiveness >10%
- **NFR-1.5**: Cache SHALL reduce repeat extraction time by >80%

### 4.2 Scalability
- **NFR-2.1**: System SHALL handle documents with 100,000+ tombstones
- **NFR-2.2**: System SHALL support 100+ concurrent users
- **NFR-2.3**: Storage SHALL scale to 1GB+ per document
- **NFR-2.4**: System SHALL implement pagination for large result sets

### 4.3 Security
- **NFR-3.1**: System SHALL encrypt tombstone data at rest
- **NFR-3.2**: Access SHALL require authentication
- **NFR-3.3**: Exports SHALL be cryptographically signed
- **NFR-3.4**: System SHALL implement role-based access control
- **NFR-3.5**: Audit logs SHALL be tamper-evident

### 4.4 Reliability
- **NFR-4.1**: System SHALL maintain 99.9% availability
- **NFR-4.2**: Data SHALL be recoverable after system failure
- **NFR-4.3**: Extraction SHALL be idempotent
- **NFR-4.4**: System SHALL validate data integrity

### 4.5 Compliance
- **NFR-5.1**: System SHALL comply with NIST 800-53 controls
- **NFR-5.2**: System SHALL support NARA retention requirements
- **NFR-5.3**: System SHALL enable FOIA request compliance
- **NFR-5.4**: System SHALL support litigation hold requirements

## 5. Data Specifications

### 5.1 Tombstone Data Model
```typescript
interface Tombstone {
  id: string;                    // Unique identifier
  content: string;                // Deleted text
  contentHash: string;            // SHA-256 hash
  contentSize: number;            // Size in bytes
  
  deletion: {
    userId: string;               // User who deleted
    userName: string;             // Human-readable name
    timestamp: string;            // ISO 8601
    clientId: number;             // Yjs client ID
    operation: string;            // Delete operation type
  };
  
  creation: {
    userId: string;               // Original author
    userName: string;             // Human-readable name
    timestamp: string;            // ISO 8601
    clientId: number;             // Yjs client ID
  };
  
  context: {
    before: string;               // Text before deletion (50 chars)
    after: string;                // Text after deletion (50 chars)
    position: number;             // Character position
    lineNumber: number;           // Line number
  };
  
  classification: {
    hasPII: boolean;              // Contains PII
    hasClassified: boolean;       // Contains classified markers
    hasFinancial: boolean;        // Contains financial data
    sensitivity: string;          // LOW, MEDIUM, HIGH
    flags: string[];              // Additional flags
  };
  
  metadata: {
    extracted: string;            // When extracted
    version: number;              // Schema version
    documentId: string;           // Parent document
    roomId: string;               // Collaboration room
  };
}
```

### 5.2 Export Format Specifications

#### 5.2.1 JSON Export Schema
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "required": ["version", "document", "tombstones", "signature"],
  "properties": {
    "version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+$"
    },
    "document": {
      "type": "object",
      "properties": {
        "id": {"type": "string"},
        "title": {"type": "string"},
        "classification": {"type": "string"},
        "created": {"type": "string", "format": "date-time"},
        "exported": {"type": "string", "format": "date-time"},
        "exportedBy": {"type": "string"},
        "mode": {"type": "string", "enum": ["standard", "audit", "compliance"]}
      }
    },
    "tombstones": {
      "type": "array",
      "items": {"$ref": "#/definitions/tombstone"}
    },
    "statistics": {
      "type": "object",
      "properties": {
        "totalTombstones": {"type": "integer"},
        "totalCharactersDeleted": {"type": "integer"},
        "uniqueUsers": {"type": "integer"},
        "dateRange": {
          "type": "object",
          "properties": {
            "earliest": {"type": "string", "format": "date-time"},
            "latest": {"type": "string", "format": "date-time"}
          }
        }
      }
    },
    "signature": {
      "type": "object",
      "properties": {
        "algorithm": {"type": "string"},
        "value": {"type": "string"},
        "timestamp": {"type": "string", "format": "date-time"},
        "certificate": {"type": "string"}
      }
    },
    "chainOfCustody": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "action": {"type": "string"},
          "timestamp": {"type": "string", "format": "date-time"},
          "user": {"type": "string"},
          "details": {"type": "object"}
        }
      }
    }
  }
}
```

#### 5.2.2 CSV Export Format
```csv
Timestamp,User,UserID,Action,Content,Size,Position,LineNumber,ContentHash,Classification,Flags
2024-01-15T10:30:00Z,"John Doe",user-123,DELETE,"confidential text",45,1234,67,sha256:abc...,MEDIUM,"PII,FINANCIAL"
```

#### 5.2.3 HTML Export Template
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Tombstone Audit Report</title>
    <style>
        body { font-family: -apple-system, sans-serif; margin: 40px; }
        .header { border-bottom: 2px solid #333; padding-bottom: 20px; }
        .tombstone { margin: 20px 0; padding: 15px; border: 1px solid #ddd; }
        .deleted { background: #ffe0e0; text-decoration: line-through; }
        .metadata { color: #666; font-size: 0.9em; margin-top: 10px; }
        .classification { display: inline-block; padding: 2px 8px; border-radius: 3px; }
        .HIGH { background: #ff9999; }
        .MEDIUM { background: #ffcc99; }
        .LOW { background: #99ff99; }
    </style>
</head>
<body>
    <div class="header">
        <h1>Tombstone Audit Report</h1>
        <p>Document: {{document.title}}</p>
        <p>Generated: {{export.timestamp}}</p>
        <p>Total Deletions: {{statistics.totalTombstones}}</p>
    </div>
    <div class="tombstones">
        {{#each tombstones}}
        <div class="tombstone">
            <div class="deleted">{{content}}</div>
            <div class="metadata">
                Deleted by: {{deletion.userName}} at {{deletion.timestamp}}<br>
                Original author: {{creation.userName}}<br>
                Classification: <span class="classification {{classification.sensitivity}}">{{classification.sensitivity}}</span>
            </div>
        </div>
        {{/each}}
    </div>
</body>
</html>
```

## 6. User Interface Specifications

### 6.1 Tombstone Viewer Layout
```
┌──────────────────────────────────────────────────────────┐
│ 🗑️ Tombstone Viewer - Contract_Draft.md                  │
├──────────────────────────────────────────────────────────┤
│ Filters: [Date Range ▼] [User ▼] [Classification ▼] 🔍   │
├──────────────────────────────────────────────────────────┤
│ Timeline: ●──────●────────●──────●──────────────●        │
├──────────────────────────────────────────────────────────┤
│ ┌────────────────────────────────────────────────────┐   │
│ │ 👤 John Doe deleted at 2024-01-15 10:30:45        │   │
│ │ ...previous text context ~~deleted content here~~  │   │
│ │ following text context...                          │   │
│ │ [Original author: Jane Smith] [45 chars] [MEDIUM]  │   │
│ └────────────────────────────────────────────────────┘   │
│                                                           │
│ ┌────────────────────────────────────────────────────┐   │
│ │ 👤 Alice Johnson deleted at 2024-01-15 11:15:22    │   │
│ │ ...context ~~another deletion~~ context...         │   │
│ │ [Original author: Bob Wilson] [23 chars] [LOW]     │   │
│ └────────────────────────────────────────────────────┘   │
├──────────────────────────────────────────────────────────┤
│ Showing 2 of 47 tombstones | [Export ▼] [Settings ⚙️]    │
└──────────────────────────────────────────────────────────┘
```

### 6.2 Compliance Dashboard
```
┌──────────────────────────────────────────────────────────┐
│ 📊 Compliance Dashboard                                   │
├──────────────────────────────────────────────────────────┤
│ Document Modes:                                          │
│ ● Standard: 234 documents                                │
│ ● Audit: 45 documents                                    │
│ ● Compliance: 12 documents                               │
├──────────────────────────────────────────────────────────┤
│ Tombstone Statistics (Last 30 Days):                     │
│ Total Deletions: 1,234                                   │
│ Flagged Content: 23 (18 PII, 3 Classified, 2 Financial)  │
│ Active Users: 67                                         │
├──────────────────────────────────────────────────────────┤
│ Recent Exports:                                          │
│ • 2024-01-15 10:45 - Quarterly Audit (J.Doe)            │
│ • 2024-01-14 16:30 - Legal Discovery (Legal Team)       │
│ • 2024-01-14 09:15 - Compliance Check (Auditor)         │
├──────────────────────────────────────────────────────────┤
│ [Generate Report] [Configure Retention] [Export All]     │
└──────────────────────────────────────────────────────────┘
```

### 6.3 Export Dialog
```
┌──────────────────────────────────────────────────────────┐
│ 📤 Export Tombstone Data                                  │
├──────────────────────────────────────────────────────────┤
│ Export Format:                                           │
│ ○ JSON (Complete metadata)                               │
│ ● CSV (Spreadsheet compatible)                           │
│ ○ HTML (Human-readable report)                           │
│ ○ CBOR (Signed binary)                                   │
├──────────────────────────────────────────────────────────┤
│ Options:                                                 │
│ ☑ Include context (50 chars before/after)                │
│ ☑ Include classification analysis                        │
│ ☐ Include only flagged content                           │
│ ☑ Add digital signature                                  │
├──────────────────────────────────────────────────────────┤
│ Date Range:                                              │
│ From: [2024-01-01] To: [2024-01-15]                     │
├──────────────────────────────────────────────────────────┤
│ Export Reason:                                           │
│ [Quarterly compliance audit                        ]     │
├──────────────────────────────────────────────────────────┤
│                              [Cancel] [Export]           │
└──────────────────────────────────────────────────────────┘
```

## 7. Implementation Plan

### 7.1 Phase 1: Core Infrastructure (Weeks 1-3)
- [ ] Implement Yjs tombstone extraction engine
- [ ] Create IndexedDB schema for tombstone storage
- [ ] Build basic data model and interfaces
- [ ] Develop extraction caching mechanism
- [ ] Create unit tests for extraction

### 7.2 Phase 2: Viewer Interface (Weeks 4-5)
- [ ] Create tombstone viewer component
- [ ] Implement filtering and search
- [ ] Add timeline navigation
- [ ] Build context display
- [ ] Add user attribution display

### 7.3 Phase 3: Export System (Weeks 6-7)
- [ ] Implement JSON export with full metadata
- [ ] Create CSV export for compliance tools
- [ ] Build HTML report generator
- [ ] Integrate PromiseGrid signing
- [ ] Add export dialog UI

### 7.4 Phase 4: Document Modes (Week 8)
- [ ] Implement mode switching logic
- [ ] Add mode persistence
- [ ] Configure retention policies
- [ ] Create mode management UI
- [ ] Add mode indicators to UI

### 7.5 Phase 5: Compliance Features (Weeks 9-10)
- [ ] Build classification detection
- [ ] Implement chain of custody
- [ ] Create compliance dashboard
- [ ] Add audit logging
- [ ] Implement legal hold functionality

### 7.6 Phase 6: Testing and Documentation (Weeks 11-12)
- [ ] Performance testing with large documents
- [ ] Security audit
- [ ] Compliance validation
- [ ] User documentation
- [ ] Administrator guide

## 8. Testing Requirements

### 8.1 Unit Tests
- Tombstone extraction accuracy
- Data model validation
- Export format correctness
- Classification detection accuracy
- Hash verification

### 8.2 Integration Tests
- Yjs integration
- IndexedDB persistence
- PromiseGrid signing
- Export pipeline
- Mode switching

### 8.3 Performance Tests
- Large document handling (100MB+)
- High tombstone volume (100,000+)
- Concurrent user load (100+ users)
- Export generation speed
- Cache effectiveness

### 8.4 Security Tests
- Access control validation
- Encryption verification
- Signature validation
- Audit log integrity
- SQL injection prevention
- XSS prevention

### 8.5 Compliance Tests
- NIST 800-53 control validation
- Retention policy enforcement
- Chain of custody verification
- Legal hold functionality
- FOIA request support

## 9. Security Considerations

### 9.1 Access Control
- Role-based permissions (viewer, exporter, admin)
- Attribute-based access for classified content
- Multi-factor authentication for exports
- Session management and timeout
- IP allowlisting for sensitive operations

### 9.2 Data Protection
- AES-256 encryption at rest
- TLS 1.3 for data in transit
- Secure key management (HSM integration)
- Data sanitization on deletion
- Memory scrubbing for sensitive data

### 9.3 Audit and Monitoring
- Comprehensive audit logging
- Real-time alerting for suspicious activity
- Regular security assessments
- Incident response procedures
- Forensic analysis capabilities

## 10. API Specifications

### 10.1 Tombstone Extraction API
```javascript
class TombstoneExtractor {
  /**
   * Extract all tombstones from a Yjs document
   * @param {Y.Doc} ydoc - The Yjs document
   * @param {Object} options - Extraction options
   * @returns {Promise<Tombstone[]>} Array of tombstones
   */
  async extractTombstones(ydoc, options = {}) {
    // Implementation
  }

  /**
   * Extract tombstones within date range
   * @param {Y.Doc} ydoc - The Yjs document
   * @param {Date} startDate - Start date
   * @param {Date} endDate - End date
   * @returns {Promise<Tombstone[]>} Filtered tombstones
   */
  async extractByDateRange(ydoc, startDate, endDate) {
    // Implementation
  }

  /**
   * Extract tombstones by user
   * @param {Y.Doc} ydoc - The Yjs document
   * @param {string} userId - User identifier
   * @returns {Promise<Tombstone[]>} User's tombstones
   */
  async extractByUser(ydoc, userId) {
    // Implementation
  }
}
```

### 10.2 Export API
```javascript
class TombstoneExporter {
  /**
   * Export tombstones to specified format
   * @param {Tombstone[]} tombstones - Array of tombstones
   * @param {string} format - Export format (json|csv|html|cbor)
   * @param {Object} options - Export options
   * @returns {Promise<Blob>} Exported data
   */
  async export(tombstones, format, options = {}) {
    // Implementation
  }

  /**
   * Sign export with PromiseGrid
   * @param {Blob} data - Export data
   * @param {Object} credentials - Signing credentials
   * @returns {Promise<SignedExport>} Signed export
   */
  async signExport(data, credentials) {
    // Implementation
  }
}
```

### 10.3 Storage API
```javascript
class TombstoneStorage {
  /**
   * Store tombstones in IndexedDB
   * @param {Tombstone[]} tombstones - Array of tombstones
   * @returns {Promise<void>}
   */
  async store(tombstones) {
    // Implementation
  }

  /**
   * Retrieve tombstones from storage
   * @param {Object} query - Query parameters
   * @returns {Promise<Tombstone[]>} Stored tombstones
   */
  async retrieve(query = {}) {
    // Implementation
  }

  /**
   * Clear tombstones based on retention policy
   * @param {Object} policy - Retention policy
   * @returns {Promise<number>} Number of cleared tombstones
   */
  async clearByPolicy(policy) {
    // Implementation
  }
}
```

## 11. Maintenance and Support

### 11.1 Operational Procedures
- Daily backup of tombstone data
- Weekly integrity checks
- Monthly performance reviews
- Quarterly security audits
- Annual compliance reviews

### 11.2 User Support
- Training materials for compliance officers
- Administrator documentation
- Troubleshooting guides
- Help desk procedures
- Video tutorials

### 11.3 System Monitoring
- Performance metrics dashboard
- Storage utilization alerts
- Error rate monitoring
- User activity analytics
- Compliance metric tracking

### 11.4 Disaster Recovery
- Recovery Time Objective (RTO): 4 hours
- Recovery Point Objective (RPO): 1 hour
- Backup strategy: 3-2-1 rule
- Failover procedures
- Data restoration process

## 12. Appendices

### Appendix A: Glossary
- **Tombstone**: Deleted content preserved in CRDT for conflict resolution
- **CRDT**: Conflict-free Replicated Data Type
- **Chain of Custody**: Chronological documentation of data handling
- **PII**: Personally Identifiable Information
- **NARA**: National Archives and Records Administration
- **FOIA**: Freedom of Information Act
- **ITAR**: International Traffic in Arms Regulations
- **HSM**: Hardware Security Module
- **RTO**: Recovery Time Objective
- **RPO**: Recovery Point Objective

### Appendix B: Regulatory References
- NIST Special Publication 800-53
- Federal Records Act (44 U.S.C. chapters 21, 29, 31, 33)
- Freedom of Information Act (5 U.S.C. § 552)
- Department of Energy Order 243.1B
- ISO 27001:2013
- SOC 2 Type II

### Appendix C: Technical Dependencies
- Yjs version 13.x
- IndexedDB API
- Web Crypto API
- PromiseGrid Protocol v1.0
- CodeMirror 6.x
- Web Workers API
- Service Workers API

### Appendix D: Performance Benchmarks
| Operation | Small Doc (<1MB) | Medium Doc (1-10MB) | Large Doc (>10MB) |
|-----------|------------------|---------------------|-------------------|
| Extract   | <500ms          | <2s                 | <5s              |
| Export    | <1s             | <3s                 | <10s             |
| Filter    | <100ms          | <500ms              | <1s              |
| Search    | <200ms          | <1s                 | <2s              |

### Appendix E: Risk Assessment Matrix
| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Data Loss | Low | High | Regular backups, redundant storage |
| Performance Degradation | Medium | Medium | Caching, optimization, monitoring |
| Security Breach | Low | High | Encryption, access control, auditing |
| Compliance Violation | Low | High | Regular audits, automated checks |
| User Adoption | Medium | Low | Training, documentation, UX design |

---

## Document Control
- **Version**: 1.0.0
- **Date**: January 2024
- **Status**: Draft
- **Owner**: Compliance Engineering Team
- **Classification**: For Official Use Only (FOUO)
- **Review Date**: July 2024
- **Distribution**: Project Team, Compliance, Legal

## Revision History
| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0.0 | 2024-01-15 | Compliance Team | Initial draft |

## Approval
| Role | Name | Signature | Date |
|------|------|-----------|------|
| Project Manager | | | |
| Technical Lead | | | |
| Compliance Officer | | | |
| Security Officer | | | |
| Legal Counsel | | | |

---

**END OF DOCUMENT**

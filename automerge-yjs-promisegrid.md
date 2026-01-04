# Automerge vs Yjs vs PromiseGrid (stevegt)

This document compares Automerge, Yjs, and PromiseGrid across architectural,
protocol, and semantic dimensions.

Important note: **PromiseGrid is not a CRDT system**. It is included here
because it addresses similar distributed-systems concerns (coordination,
state propagation, decentralization) using fundamentally different mechanisms.

---

## Comparison Table

| Aspect | Automerge | Yjs | PromiseGrid |
|------|-----------|-----|-------------|
| **Primary purpose** | CRDT-based state synchronization | CRDT-based real-time collaboration | RPC-based coordination with capability security |
| **System class** | State-based CRDT | Operation/state hybrid CRDT | Promise-based distributed system |
| **Identity / authorization primitive** | Change hashes (SHA-256) | Client IDs + logical clocks | Capability tokens (hash-based) |
| **Container / framing** | Custom binary container format | Custom binary framing (transport-dependent) | CBOR-encoded RPC messages |
| **Addressing model** | Content-addressed changes | Replica identity + clocks | Multihash-capable identifiers (design-level) |
| **Sync / convergence model** | Bloom filters + causal heads | State vectors + incremental updates | Explicit RPC fulfillment of promised state |
| **Encoding specification** | Fully specified (uLEB128, RLE, columnar) | Custom variable-length integer encoding | Intentionally unspecified / pluggable |
| **Conflict handling** | Deterministic CRDT merge rules | Deterministic CRDT merge rules | Application-defined agreement / consensus |
| **Consistency goal** | Strong eventual consistency | Strong eventual consistency | Promise satisfaction (not convergence) |
| **Go implementation** | `automerge-go` (CGO wrapper) | No official Go library (ports exist) | `grid-cli` (in development) |
| **Typical use cases** | Local-first applications | Real-time collaborative editors | Decentralized compute, governance, coordination |

---

## Line-by-Line Technical Comparison

### 1. Primary purpose

**Automerge** and **Yjs** are both designed to keep replicas of shared state
synchronized without central coordination. Their purpose is *state convergence*.

**PromiseGrid** is designed to coordinate *intentions and responsibilities*
between autonomous agents using promises and capabilities. State convergence
is not a built-in goal.

**Key distinction:**  
CRDTs converge state; PromiseGrid coordinates behavior.

---

### 2. System class

Automerge is a **state-based CRDT** with explicit change tracking and causal
history.

Yjs is a **hybrid system**, combining operation-based updates with state-vector
synchronization.

PromiseGrid is **not a CRDT**. It is an RPC-oriented system grounded in
Promise Theory, where agents remain autonomous and state is local by default.

---

### 3. Identity / authorization primitive

Automerge identifies data using **cryptographic change hashes**.

Yjs identifies updates by **replica ID and logical clock**.

PromiseGrid uses **capability tokens**, typically hash-based, which serve as
both authorization and routing identifiers.

These mechanisms serve different purposes and should not be conflated.

---

### 4. Container / framing

Automerge defines a complete binary container format.

Yjs defines binary message formats but relies on external transports
(WebSocket, WebRTC, IndexedDB).

PromiseGrid uses **CBOR** for message encoding, aligning with IETF and
constrained-environment practices.

---

### 5. Addressing model

Automerge is **content-addressed** at the change level.

Yjs addresses updates relative to a known replica and clock state.

PromiseGrid supports **flexible addressing**, often multihash-based, but this
is a design choice rather than a mandated protocol rule.

---

### 6. Sync / convergence model

Automerge synchronizes by exchanging summaries (Bloom filters and heads) to
discover missing changes.

Yjs uses **state vectors** to request and apply only missing updates.

PromiseGrid does **not synchronize state**. Instead, agents explicitly request
and fulfill promises via RPC. Any state transfer is intentional and scoped.

This is a fundamental architectural difference.

---

### 7. Encoding specification

Automerge’s encoding is exhaustively specified, including compression schemes.

Yjs defines its own compact binary encoding with variable-length integers.

PromiseGrid deliberately avoids fixing an encoding beyond CBOR, allowing
experimentation and evolution.

---

### 8. Conflict handling

Automerge and Yjs resolve conflicts automatically using CRDT merge semantics.

PromiseGrid does not resolve conflicts automatically. Conflicts are handled
by **application-level agreement**, negotiation, or governance mechanisms.

This shifts responsibility from the data structure to the system design.

---

### 9. Consistency goal

CRDTs aim for **strong eventual consistency**.

PromiseGrid aims for **promise satisfaction**, which may or may not involve
shared state and does not imply convergence.

---

### 10. Go ecosystem support

Automerge has a CGO-based Go wrapper.

Yjs has no official Go implementation.

PromiseGrid’s Go tooling (`grid-cli`) is under active development and reflects
the experimental nature of the system.

---

### 11. Typical use cases

Automerge excels at offline-first, local-first applications.

Yjs dominates real-time collaborative editing.

PromiseGrid targets **decentralized systems**, including governance,
distributed automation, and coordination across administrative domains.

---

## Final Note

Automerge and Yjs solve *the same problem* with different tradeoffs.

PromiseGrid solves a **different problem** that occasionally overlaps in
practice but not in theory.

Comparisons are useful for intuition, but architectural intent matters.

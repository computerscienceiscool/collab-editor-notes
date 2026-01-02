# YJS vs PromiseGrid: What We Learned

---

## The Question

**Can we use YJS for PromiseGrid?**

Short answer: **No, but we can learn from it.**

---

## What YJS Does Well

- Proven CRDT (Conflict-free Replicated Data Type)
- Efficient binary protocol
- Fast sync with state vectors
- Peer-to-peer capable

---

## Why YJS Won't Work for PromiseGrid

| YJS | PromiseGrid Needs |
|-----|-------------------|
| Mutable state | **Immutable** everything |
| Can garbage collect (lose history) | **Preserve forever** |
| Position-based addressing | **Content-addressed** (hashes) |
| Optional persistence | **Mandatory** audit trail |

**These are non-negotiable for legal, medical, financial use cases.**

---

## YJS Protocol: 3 Message Types

```
SyncStep1 (type=0): "Here's what I have" (state vector)
SyncStep2 (type=1): "Here's what you're missing" (operations)
Update    (type=2): "Here's what just changed" (new op)
```

Simple. Efficient. Worth copying the concept.

---

## YJS Sync Flow

```
Client A                    Client B
   |                           |
   |-- "What I have" --------->|
   |<- "What you need" --------|
   |-- "What you need" ------->|
   |                           |
   Both synchronized ✓
```

---

## YJS Data Model

```
Document: "Hello"

[H] -> [e] -> [l] -> [l] -> [o]
 |      |      |      |      |
(42,0) (42,1) (42,2) (42,3) (42,4)
       client ID, clock
```

Each character has a unique ID = deterministic merge.

---

## The Tombstone Problem

**Delete 'e' in YJS:**
```
[H] -> [e☠️] -> [l] -> [l] -> [o]
        ^
        Marked deleted, but still there
```

**Problem:** YJS can garbage collect tombstones = **history lost**

**PromiseGrid:** Delete operation stored forever, original preserved.

---

## PromiseGrid Message Model

```
YJS:        [type byte] [data]
PromiseGrid: [function hash] [args hash]
```

Everything is content-addressed. Same content = same hash.

---

## Proposed PromiseGrid CRDT

**Sync Request:**
```
Function: hash("sync_request")
Args:     hash(my_state_vector)
```

**Sync Reply:**
```
Function: hash("sync_reply")  
Args:     hash(operations_you_need)
```

**Update:**
```
Function: hash("apply_op")
Args:     hash(operation)
```

---

## What to Reuse from YJS

✅ Operation-based CRDT (send ops, not states)
✅ Unique IDs: (client_id, clock)
✅ State vectors for efficient sync
✅ Deterministic merge rules

---

## What to Build Different

❌ No garbage collection (keep everything)
❌ No mutable state (immutable only)
✅ Content-addressed operations
✅ Merkle trees for verification
✅ PromiseBase storage

---

## The Bottom Line

| Question | Answer |
|----------|--------|
| Use YJS directly? | **No** - violates immutability |
| Wrap YJS in PromiseGrid? | **No** - can't fix GC problem |
| Learn from YJS? | **Yes** - great concepts |
| What to build? | **PromiseGrid-native CRDT** |

---

## Next Steps

1. **Design** PromiseGrid CRDT spec
2. **Define** operation format & storage schema
3. **Prototype** basic insert/delete/sync
4. **Integrate** with PromiseBase
5. **Test** with Storm

---

## Questions for Steve

1. Store operations only, snapshots only, or both?
2. How to structure Merkle trees?
3. Confirm: function hash + args hash for messages?
4. Priority vs. other PromiseBase work?

---

## Resources Created

- **This presentation** (slides.md)
- **Detailed reference** (yjs-promisegrid.md)  
- **Annotated YJS code** (yjs-sync-protocol-commented.js)

**12+ hours of research condensed here.**

---

## Summary

**YJS:** Great CRDT, wrong model for PromiseGrid.

**PromiseGrid CRDT:** Same concepts, built on immutability.

**Ready to design the spec.**

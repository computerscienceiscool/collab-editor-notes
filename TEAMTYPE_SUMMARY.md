# TeamType Summary

## What Is TeamType?

Open-source collaborative editing tool for Neovim and VSCode using Automerge CRDT and P2P networking.

## Critical Finding

**Automerge (the CRDT TeamType uses) appears to have NO garbage collection.**

Evidence:
- Documentation states "stores every change made to a document"
- Users ask on GitHub "how do I remove history?" (proving it's not automatic)
- No GC code found in searches
- TeamType uses unmodified Automerge

Confidence: 85% (needs 30-minute test to reach 100%)

## How Your Team Would Use It

**NOT using TeamType as-is. Cherry-picking pieces:**

1. **Automerge CRDT** - Replace Yjs (which has GC)
2. **Neovim integration approach** - Learn from their plugin design
3. **Editor-to-CRDT patterns** - How they handle operations

**NOT using:**
- TeamType daemon (you have your own editor)
- Iroh P2P (you have PromiseGrid)
- TeamType file management

## Architecture Comparison

**TeamType:**
```
Neovim/VSCode → TeamType Daemon → Automerge CRDT → Iroh P2P → Peers
```

**Your Editor After Integration:**
```
Browser/Neovim → Your Editor → Automerge CRDT → PromiseGrid → Peers
```

## Work Estimate

**If GC test passes:**
- Replace Yjs with Automerge: 23-35 hours
- Add Neovim support: 24-34 hours  
- PromiseGrid integration: 23-35 hours
- **Total: 70-104 hours**

**If GC test fails:**
- Build custom CRDT: 120-200+ hours

## Next Step

**Run 30-minute verification test to confirm Automerge has no GC.**

Test determines: Can you use Automerge (70-104 hours) or must you build custom CRDT (120-200+ hours)?

## Bottom Line

TeamType proves Automerge works with Neovim. If Automerge preserves history (verification needed), you can use it in your editor and add Neovim support. Saves 50-100+ hours vs building custom CRDT.

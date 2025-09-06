# Research Agenda on Philosophical Questions Raised

## 1. Metaphors for Organizing Data

**Why it matters**\
Current systems borrow from the **file/folder metaphor** that made sense
for local computing. On the web, URLs and pages don't consistently
reflect underlying data structures. Without a strong metaphor, systems
risk confusion and poor user mental models.

**Next steps for exploration**\
- Compare existing metaphors (file/folder, room, container, mesh/web).\
- Study how each affects user comprehension and system design.\
- Consider hybrid metaphors (e.g., spatial + relational).

**Concrete technical experiments**\
- Prototype a **room-based navigation system** and compare against
folder-style navigation.\
- Run usability tests to measure how quickly users locate content under
different metaphors.\
- Simulate linking models (web-style vs. hierarchical) to study how
easily users map data relationships.

------------------------------------------------------------------------

## 2. Ownership of Digital vs. Physical Spaces

**Why it matters**\
In the real world, **companies own rooms/buildings**, and ownership
changes over time. In digital systems, "rooms" or spaces often persist
indefinitely, raising questions about who "owns" data and history.

**Next steps for exploration**\
- Analyze whether virtual "rooms" should retain persistent ownership or
allow transfer.\
- Explore implications for access control, historical records, and
digital identity.\
- Look at parallels in blockchain, decentralized identity, and virtual
land systems.

**Concrete technical experiments**\
- Implement a **transferable room ownership model** and compare with
persistent ownership.\
- Simulate what happens when users leave a system: does ownership
dissolve, transfer, or persist?\
- Prototype access history logs showing how ownership affects audit
trails.

------------------------------------------------------------------------

## 3. Persistence, Naming, and Collision Risks

**Why it matters**\
Digital systems rely on **names and keys** to identify data. Without
careful design, collisions (e.g., two rooms/documents with the same
name) cause conflicts and loss of trust. Persistence policies also shape
long-term usability.

**Next steps for exploration**\
- Examine strategies for naming uniqueness (UUIDs, namespaces,
hierarchical models).\
- Investigate trade-offs between **human-readable vs. machine-readable
names**.\
- Research how different persistence models (ephemeral, archival,
mutable) affect collaboration and accountability.

**Concrete technical experiments**\
- Stress-test IndexedDB with **auto-generated UUIDs vs. human-readable
names**.\
- Simulate namespace collisions under heavy concurrent usage.\
- Prototype archival vs. ephemeral modes for documents and measure
retrieval success rates.

------------------------------------------------------------------------

## 4. Cognitive Science and Design Metaphors

**Why it matters**\
Users understand digital systems through **mental models**. If metaphors
don't align with cognitive expectations, usability suffers. This is both
a **design philosophy** and a **cognitive science** challenge.

**Next steps for exploration**\
- Review cognitive science literature on metaphors in computing.\
- Test user comprehension of different metaphors in prototypes.\
- Consider whether entirely new metaphors (beyond folders/rooms) are
needed for next-gen collaborative systems.

**Concrete technical experiments**\
- Conduct **A/B testing** with prototypes using different metaphors
(folder, room, graph).\
- Use **eye-tracking studies** to measure cognitive load under each
metaphor.\
- Prototype a new metaphor (e.g., "living documents as conversations")
and evaluate user adoption.

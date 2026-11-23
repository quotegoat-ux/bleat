---
name: principle-separate-before-serializing-shared-state
description: "Apply when concurrent actors might write to the same file, branch, key, or state object, or when you run work in parallel on one transaction, connection, or deploy step. Eliminate the sharing first; serialize structurally only when one shared writer is a real invariant."
user-invocable: false
---

# Separate Before Serializing Shared State

When concurrent actors might share mutable state, first ask whether they need the same mutable object. If not, eliminate the sharing. When sharing is real, enforce serialization structurally: lockfiles, sequential phases, exclusive ownership. Instructions and conventions are not concurrency control.

**Why:** Concurrent writes to shared state create race conditions that are intermittent, hard to reproduce, and expensive to debug.

**Pattern:**
1. **Identify shared mutable state** (files both read and write, branches both push to, APIs both define and consume).
2. **Default: eliminate the shared write target.** Ask: do these actors need one canonical object, or are they publishing independent facts? Give each actor its own owned file, key, branch, or state directory, and merge only at the read/reporting boundary. Two workers writing their own `lastX` field into one `state.json` is still shared mutation. `indexer-state.json` + `metrics-state.json` is not.
3. **Only when one shared write target is a real invariant, serialize access structurally** (lockfiles, sequential phases, single-writer actor, or atomic compare-and-swap). Treat "we need a lock" as a design smell to check, not as the default answer.
4. **Know what already runs one at a time.** Some shared resources accept only one operation at a time: a database transaction or connection, a catalog that two schema changes both touch, a file handle, a rate-limited API key. `Promise.all` over one transaction, or two deploy resources that both grant on the same schema, fail only when the two overlap, so the happy run passes. Before you start work in parallel, name the resource each branch uses. If two branches share one of these, run them in order, or give each its own (a second connection, an explicit dependency between the two deploy resources).

**The test:** put a fake in front of the shared resource that refuses a second operation while one is running, and run the code through it. The fake fails every time the real resource fails only sometimes. Then search for the same `Promise.all` or parallel-resource shape in sibling helpers, since the first fix rarely finds them all.

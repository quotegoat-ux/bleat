---
name: principle-separate-before-serializing-shared-state
description: "Apply when concurrent actors might write to the same file, branch, key, or state object, or when you run work in parallel on one transaction, connection, or deploy step. Eliminate the sharing first; serialize structurally only when one shared writer is a real invariant."
user-invocable: false
---

# Separate Before Serializing Shared State

When concurrent actors might write the same file, branch, key, or object, first ask whether they need one shared object or are publishing independent facts. Default to giving each actor its own file, key, branch, or state directory, and merge only where the state is read. Two workers writing their own field into one `state.json` is still shared mutation.

Only when one shared write target is a real invariant, serialize access structurally (a lockfile, a single writer, sequential phases, compare-and-swap). Instructions are not concurrency control, and "we need a lock" is a smell to check first.

Know what already runs one at a time. Some resources accept only one operation at a time: a database transaction or connection, a catalog that two schema changes both touch, a file handle, a rate-limited API key. `Promise.all` over one transaction, or two deploy resources that both grant on the same schema, fail only when the two overlap, so the happy run passes. Before you start work in parallel, name the resource each branch uses. If two branches share one of these, run them in order, or give each its own (a second connection, an explicit dependency between the two deploy resources).

The test: put a fake in front of the shared resource that refuses a second operation while one is running, and run the code through it. The fake fails every time the real resource fails only sometimes. Then search for the same `Promise.all` or parallel-resource shape in sibling helpers, since the first fix rarely finds them all.

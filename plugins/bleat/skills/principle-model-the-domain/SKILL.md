---
name: principle-model-the-domain
description: "Apply when writing stateful logic, when code branches a lot or repeats a shape assumption across files, or when a record has more than one way in (two screens, two create paths, two servers)."
user-invocable: false
---

# Model the Domain

Encode the real domain in a data structure instead of scattering it across conditionals. A structure that matches the domain makes invalid states unrepresentable and deletes branches. Choosing it at write time is cheap. Recovering it later reads as a refactor and gets deferred.

Reach for:

- A state machine instead of scattered booleans, phases, or lifecycle checks.
- A typed model instead of loose parameters or repeated shape assumptions.
- A map, registry, lookup table, or discriminated union instead of branching spread across files.
- A reducer or command/event model instead of ad hoc state mutations.
- A module organized around one body of domain knowledge instead of a sequence such as load, validate, transform, and save. Execution order is not ownership.
- A small module boundary that gathers repeated behavior, ownership, or invariants.
- A queue, cache, index, graph, tree, or normalized collection where the access pattern calls for it.
- Any other structure that fits. When none fits, work out what the code must never allow and how the data gets read, then find the structure that encodes exactly that.

State held in a client (a store, a grid, router state, a query cache) has a lifecycle. Write down what ends or reloads it before you write the code that reads it:

- Which copy does a write reach? When a grid row and a store hold the same data, name the one the save reads, and test the write against it.
- What happens to pending edits when the user navigates away, refreshes, or closes the tab?
- What re-syncs it on a new history entry, a redirect, or a remount, when the ids stay the same?
- Does a mounted but hidden view still fetch, refetch on focus, or react to invalidation?
- What refetches after a write, what is left after a partial failure, and which writes can race?
- Is every cached query keyed by the organization (and subsidiary) it was read for, so an organization switch can't show the last one's rows?
- When the server makes a new row's id, which copies still hold the temporary one: the live grid row, a pin, a pending rename that sends a second create?

A record often has more than one way in: a grid cell and a drawer edit the same field, signup and an admin screen both create an account, a create returns the record and a read returns it again, a local server and the deployed one answer the same request. Each rule for that record lives in one function that every path calls:

- Before adding a rule, list every path that reaches the record and call the rule from each.
- Before adding a path, list the rules the existing path applies (what is editable, what an edit implies, what gets seeded, what the response holds) and call them, not a copy.
- Run one case through each path and assert the same result. That test catches the path you forgot.

Do not force an abstraction. Prefer boring code if the current shape is already clear, local, and unlikely to grow. Be skeptical of an abstraction that adds indirection without removing branches, duplicated rules, invalid states, or lifecycle risk.

The sign that you skipped this is a new feature that grows an existing if/else chain by one more branch, or a second boolean that must stay in sync with the first. Temporal decomposition is another sign. Phase-named modules repeat the same domain rules across steps.

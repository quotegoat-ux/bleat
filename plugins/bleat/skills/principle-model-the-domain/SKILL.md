---
name: principle-model-the-domain
description: "Apply when writing stateful logic, when code branches a lot or repeats a shape assumption across files, or when a record has more than one way in (two screens, two create paths, two servers)."
user-invocable: false
---

# Model the Domain

Encode the domain in a data structure instead of scattering it across conditionals. The right structure makes invalid states unrepresentable and deletes branches.

Reach for a state machine instead of scattered booleans or phase checks, a typed model instead of loose parameters, a map, registry, or discriminated union instead of branching spread across files, a reducer instead of ad hoc mutations, and an index, queue, or normalized collection where the access pattern calls for it.

Do not force it. Keep boring code when the shape is clear, local, and unlikely to grow, and distrust an abstraction that adds indirection without removing branches or invalid states.

The sign that you skipped this: a feature that grows an if/else chain by one more branch, or a second boolean that must stay in sync with the first.

## Client state

State held in a client (a store, a grid, router state, a query cache) has a lifecycle. Write down what ends or reloads it before you write the code that reads it:

- Which copy does a write reach? When a grid row and a store hold the same data, name the one the save reads, and test the write against it.
- What happens to pending edits when the user navigates away, refreshes, or closes the tab?
- What re-syncs it on a new history entry, a redirect, or a remount, when the ids stay the same?
- Does a mounted but hidden view still fetch, refetch on focus, or react to invalidation?
- What refetches after a write, what is left after a partial failure, and which writes can race?
- Is every cached query keyed by the organization (and subsidiary) it was read for, so an organization switch can't show the last one's rows?
- When the server makes a new row's id, which copies still hold the temporary one: the live grid row, a pin, a pending rename that sends a second create?

The race question needs a run, not an answer. For every control that saves (a toggle, a multi-select, a cell, a drag), test two cases: change it twice before the first save answers, and make the second of two saves fail. In both, the screen must end up showing what the server holds. A control that sends the whole list on each change, or restores a baseline taken before an earlier save landed, fails one of the two.

The refetch question needs a run too. Make one change through each write path (autosave, a direct write, undo, redo) and assert that every total or list built from it updates. A refresh that waits for a save indicator to go from saving to saved misses every path that never shows it.

When one save goes out as ordered phases (deletes, then moves, then creates and updates) and the server deletes a parent's children with it, an early phase can destroy what a later one sends. Test each pair of edits in one save: move a child out from under a parent and delete the parent, undo a move off a line that was deleted, renumber rows and edit one of them. Then fail a later phase and retry. The server must end up holding exactly the rows on screen, each once, and an update sends only the fields that changed.

## One rule for every path

A record often has more than one way in: a grid cell and a drawer edit the same field, signup and an admin screen both create an account, a create returns the record and a read returns it again, a local server and the deployed one answer the same request. Each rule for that record lives in one function that every path calls:

- Before adding a rule, list every path that reaches the record and call the rule from each.
- Before adding a path, list the rules the existing path applies (what is editable, what an edit implies, what gets seeded, what the response holds) and call them, not a copy.
- Run one case through each path and assert the same result. That test catches the path you forgot.
- Paths include keyboard shortcuts, right-click menus, paste, drag and the selection bar, not only buttons. For each control that writes, run it as a role that holds exactly one of create, edit and delete, and on a locked or archived record. Each one is either hidden or allowed by the server, never offered and then refused.

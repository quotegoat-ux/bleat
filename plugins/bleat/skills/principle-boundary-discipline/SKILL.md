---
name: principle-boundary-discipline
description: "Apply when wiring validation, access checks, error handling, or framework adapters. Concentrate guards at system boundaries (CLI, config, network, external APIs); trust internal types and keep business logic in pure functions."
user-invocable: false
---

# Boundary Discipline

Validate, narrow types, and handle errors where data crosses a system boundary (CLI args, config files, external APIs, network). Inside, trust the types: no re-validation and no redundant nil checks. Parse raw data into domain types at the boundary, and keep transport and framework types out of the public surface. Business logic lives in pure functions with no framework dependencies. The shell that calls them is thin.

Three rules decide what a guard at the boundary must do:

- A reference from outside (an id, a storage key, a webhook event's object id) is input to validate, not a fact. Check it resolves to a row the caller owns before you act on it, and read derived values such as a storage key from that row, never from the request. Destructive effects (deleting an object, resetting a subscription) need this most, because nothing downstream undoes them.
- An access check that finds nothing to check refuses. When the owner of a row resolves to null, an id list parses to empty (`ids=,`), or a filter names a tenant or subsidiary the caller has no grant on, answer 400, 403 or 404. Never fall back to "any tenant or subsidiary I can see". Bound every list and lookup by the caller's allowed set on every request, and let a caller's filter only narrow that set, never replace it.
- A refusal is a read. An "in use by" list, a conflict message, a uniqueness check, or a 403 where a 404 belongs tells the caller about rows they may not see. Name only rows the caller may read, and count the rest ("used by 3 records you can't open").

Four tests. Is this data crossing a boundary right now? If not, the validation is redundant. Can this be a pure function the shell just calls? If yes, extract it. For each route you add or change that reads or writes tenant data, send four requests: another tenant's id, a sibling subsidiary's id in the filter, an empty or comma-only id list, and a row with no owner. Each one is refused or returns only rows the caller may see; keep them as tests. For each refusal or conflict message you add, trigger it as a caller who can't read the record that causes it. The answer names none of that record's fields.

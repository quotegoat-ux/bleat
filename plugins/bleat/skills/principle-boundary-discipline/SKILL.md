---
name: principle-boundary-discipline
description: "Apply when wiring validation, access checks, error handling, or framework adapters. Concentrate guards at system boundaries (CLI, config, network, external APIs); trust internal types and keep business logic in pure functions."
user-invocable: false
---

# Boundary Discipline

Place validation, type narrowing, and error handling at system boundaries. Trust internal code unconditionally. Business logic lives in pure functions. The shell is thin and mechanical.

**Why:** Scattered validation is noisy, redundant, and gives a false sense of safety. Keep logic out of framework wiring so it can be tested without the framework.

**The pattern:**
- **At boundaries** (CLI args, config files, external APIs, network protocols): validate, return errors, handle defensively.
- **Inside the system:** typed data, error propagation, no re-validation. Trust the types.
- **Across the boundary.** Expose domain concepts, not the boundary's private representation. Keep general-purpose mechanism inside and special-purpose policy at the edge.

**Applications:**

Validation and error handling:
- Validate config at parse time (the boundary), not inside business logic
- Parse raw data into domain types at the boundary
- Do not re-export transport, storage, framework, or wire types through the public surface
- No redundant nil checks deep in call chains if the boundary already validated
- A reference from outside (an id, a storage key, a webhook event's object id) is input to validate, not a fact. Check it resolves to a row the caller owns before you act on it, and read derived values such as a storage key from that row, never from the request. Destructive effects (deleting an object, resetting a subscription) need this most, because nothing downstream undoes them.
- An access check that finds nothing to check refuses. When the owner of a row resolves to null, an id list parses to empty (`ids=,`), or a filter names a tenant or subsidiary the caller has no grant on, answer 400, 403 or 404. Never fall back to "any tenant or subsidiary I can see". Bound every list and lookup by the caller's allowed set on every request, and let a caller's filter only narrow that set, never replace it.

Code organization:
- Business logic in pure functions with no framework dependencies
- Parse functions: pure transforms from raw bytes to typed state
- Prompt construction: structured state in, string out
- Scoring and assessment: pure transforms from state to results

**The tests:**
- "Is this data crossing a system boundary right now?" If not, validation is redundant.
- "Can this be a pure function that the shell just calls?" If yes, extract it.
- For each route you add or change that reads or writes tenant data, send four requests: another tenant's id, a sibling subsidiary's id in the filter, an empty or comma-only id list, and a row with no owner. Each one is refused or returns only rows the caller may see. Keep them as tests.

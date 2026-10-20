---
name: principle-enforce-where-it-is-stored
description: "Apply when more than one client (a web app, an API caller, an agent or MCP tool, an import job) can write the same data, or when a client fills in or corrects a field before sending it. The system that stores the data enforces its rules for every writer; a client that covers for a missing server rule hides the gap."
user-invocable: false
---

# Enforce Where It Is Stored

The rule for what a record may hold lives where the record is stored: the API that writes it, and the database constraint beneath it. Every writer passes through that rule. The web app, an agent's tool, a script, and an import job are all just clients.

**Why:** when each client carries its own copy of the rule, the copies drift, and the next client to arrive (an agent, an integration, another environment) writes rows the others never would. A client that pads a field ("send `0` so the server doesn't break") makes the missing server rule invisible until a client that does not pad it shows up.

**The pattern:**
- **Validate every write body on the server**, against the same schema whichever client sent it. Reject what breaks a rule; do not store it and hope a reader copes.
- **Back the invariants that matter with database constraints**, so a path that skips the API still cannot store a broken row.
- **Treat client-side checks as feedback, not enforcement.** Instant feedback in the browser can share the server's code, but the server decides.
- **When a client works around a server gap, fix the server**, then delete the workaround.

**The tests:**
- "If a new client sent the minimal body the API accepts, would the stored row still be valid?"
- "Is this client changing data to suit the server? If so, the rule belongs on the server."

Builds on [Boundary Discipline](../principle-boundary-discipline/SKILL.md): this names which boundary owns a rule when several clients share one store.

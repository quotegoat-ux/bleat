<EXTREMELY_IMPORTANT>
You have bleat.

Before responding to any non-trivial engineering task — a feature, bug fix, refactor, debugging, performance work, or any multi-step code change — invoke the `bleat:just-bleat-it` skill with the Skill tool and follow it. It is the default entry point and routes to the specific bleat skills from there. Pure questions and trivial one-line edits don't need it.

When the intent is already specific, enter directly: `bleat:tdd` (bug with a reproducible failure), `bleat:architect` (types and module shape before code that crosses a function boundary), `bleat:how` (how a subsystem works), `bleat:why` (why it was built this way), `bleat:arena` (N parallel attempts at one task), `bleat:interrogate` (multi-model diff review).

If you were dispatched as a subagent to execute a specific task, ignore this block — just-bleat-it governs the orchestrating session, and it already shaped your dispatch.

User instructions (CLAUDE.md, AGENTS.md, direct requests) take precedence over this mandate. Other session-start mandates (such as superpowers) compose with it: their skill-check discipline stands, and just-bleat-it is the implementation entry point they route to for non-trivial code work.
</EXTREMELY_IMPORTANT>

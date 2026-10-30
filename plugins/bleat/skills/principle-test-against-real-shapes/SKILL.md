---
name: principle-test-against-real-shapes
description: "Apply when writing a test, a seed, a fixture, or a migration, or when checking a change before it ships. Exercise the data shapes production actually holds (old rows, missing settings, several tenants, large numbers, rows another client wrote), not only the clean fixture the change was written against."
user-invocable: false
---

# Test Against Real Shapes

A change written against a clean fixture passes against that fixture. Production holds rows the fixture never had: records made before a column existed, an account with no settings, two tenants sharing a catalog, numbers in the millions, duplicates an old bug left behind, rows written by another client. Most bugs that reach a shared environment live in those shapes.

**Why:** a green test on the happy fixture proves the change works on data nobody has. The failure shows up only where real data lives, after the merge.

**The pattern:**
- **Seed the odd shapes on purpose.** Keep a fixture that carries the shapes production has: missing optional settings, several tenants or subsidiaries, records made through every write path (UI, API, agents, imports), very large and very small values, and one empty parent.
- **Run a migration against data that already breaks it.** A new constraint, unique index, or NOT NULL column meets existing rows. Seed a violating row, and make the migration clean, backfill, or refuse with a clear message before it adds the constraint.
- **Run a new rule under every mode the domain names.** When a change adds a rule to shared logic, list the variants and modes its inputs can take: each value of an enum, each override tier, each kind of parent or child, and the record that lacks the part the rule assumes. Test the rule's definition under each one, with the expected value taken from the domain's own definition. A rule that is right for the case it was written for and wrong for a mode nobody listed is the most common cross-feature bug.
- **Check a number everywhere it shows.** When a change touches a value that several screens, exports, or documents display, assert the same value in each place for the same record.
- **Ask where the data came from.** Before calling a bug "can't reproduce", find out which client wrote the record and reproduce with a record written the same way.

**The tests:**
- "Which rows in production would this change see that my fixture does not have?" Name them, then add them.
- "Would this migration succeed on yesterday's data, not just an empty database?"
- "Which modes and variants of these inputs did I not run the new rule under?"

Distinct from [Test Behavior, Not Implementation](../principle-test-behavior-not-implementation/SKILL.md), which is about what a test asserts. This one is about the data it runs on.

### Opening a PR

Invoked at the end of every other playbook.

Work from a git worktree off trunk, and keep unrelated work out of it. Rebase into small, landable commits. Run `/deslop` over the diff before commit.

Title: `type(scope): subject`, naming a real symbol when one carries the change. Body: a one-minute briefing under 40 lines with these `##` headings in order, dropping empty ones: `## Why`, `## What changed`, `## Scope`, `## Tradeoffs` (only rejected alternatives a reviewer would otherwise ask about), `## Blast Radius`, `## Verification`.

**Linked issues.** Before you write `Closes #N` or `Fixes #N`, read the issue and check each acceptance criterion against the diff. Write `Closes` only when every criterion is met. Otherwise write `Refs #N` and list the unmet criteria under `## Scope`.

**Bleat marker.** End the body with one line, `Built with Bleat <version>`, using the version the session-start mandate names ("You have bleat 1.8.0"). Add it on every PR, even in a repo whose PR template has its own sections, and keep it when you edit the body. Review metrics use it to compare PRs built with Bleat against PRs built without it.

Stack follow-ups by base branch. Open PRs ready, with `gh pr create` and no `--draft`. Opening a PR does not start a babysit.

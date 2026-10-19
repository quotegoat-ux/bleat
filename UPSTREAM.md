# Upstream

bleat is its own skill library, built on two upstreams. This file records where each sync was taken from, so the next sync diffs from there.

| upstream | role | last synced | version |
| --- | --- | --- | --- |
| [`cursor/plugins`](https://github.com/cursor/plugins/tree/main/pstack), `pstack/` | skill content (poteto's pstack) | `d457567332aa6989e753ea289f30ea2bfbc80bc0` (2026-10-19) | 0.15.28 |
| [`cursor/plugins`](https://github.com/cursor/plugins/tree/main/cursor-team-kit), `cursor-team-kit/skills/` | vendored team-kit skills | `e46364b8be46000b7df0f260550cd712afbb8d36` | — |
| [`michael-denyer/pstack-claude`](https://github.com/michael-denyer/pstack-claude) | the Claude Code port bleat forked from at `c2ade4b`; a reference for translating new Cursor primitives | `49b68501b075ec1eb2755dd0d22fbf2492d67e97` (0.9.94, upstream `7b353f7`) | — |

The machine-readable pins live in [`tools/upstream.json`](tools/upstream.json), and `tools/sync.mjs` advances them. Keep this table in step with that file.

## Sync history

| bleat | pstack | upstream range |
| --- | --- | --- |
| 1.3.0 | 0.15.28 | `f8abedd..d457567` |
| 1.2.0 | 0.15.0 | `fd87869..f8abedd` |
| 1.1.0 | 0.14.5 | `fd87869` |

## Running the next sync

1. Check for new work: `git log --oneline <pinned-sha>..main -- pstack` in a clone of `cursor/plugins`. Do the same for `cursor-team-kit/skills`.
2. Run `bun tools/sync.mjs bleat <new-sha>` on a clean tree. It writes clean updates, three-way merges the port's edits, and lists the files that still conflict.
3. If the run fails on the denylist, add a substitution rule for a rename (a new `poteto-*` skill, say) or rewrite the sentence, then rerun.
4. Resolve the remaining conflicts by the rule in [CONTRIBUTING.md](CONTRIBUTING.md#the-sync-boundary): upstream owns skill content, and bleat owns the Claude Code translation and its Quote Goat customizations. Look at how `pstack-claude` translated the same hunk before inventing a new translation.
5. Give each new user-invocable skill a `menu-description:` and a `README_COMMAND_ORDER` entry in `tools/generate.mjs`. Never add `disable-model-invocation`.
6. Bump `VERSION`, write the `CHANGES.md` entry, add a row above, run `bun tools/generate.mjs`, `bun test tests/`, and the script tests under `plugins/bleat/skills/just-bleat-it/scripts`.

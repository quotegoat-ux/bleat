---
name: setup-bleat
description: Configure which models bleat uses per role and at what reasoning budget. Detects your available Claude models and writes a per-role override file that the user can include from their CLAUDE.md. Use for /setup-bleat, "configure bleat models", "bleat budget", or changing bleat's model choices.
menu-description: configure bleat per-role model choices and reasoning budget
---

# Setup bleat

Write `~/.claude/bleat-models.md`, a per-role model override sheet you include from your global `CLAUDE.md`.

**Platform note.** On Codex or another non-Claude runtime, the override sheet is `~/.codex/bleat-models.md`, the slugs are your Codex models (for example `gpt-5.5`) not `claude-*`, and you load it by adding the sheet's contents to `~/.codex/AGENTS.md` (Codex has no `@`-include into a rules file). The role rows in step 5 are identical; only the slugs, the file path, and the load mechanism change. Detect Codex slugs from `~/.codex/config.toml` (`model = ...`) plus whatever the user confirms. See [`codex-tools.md`](../just-bleat-it/references/codex-tools.md).

Claude Code has no auto-applied "rules" mechanism like Cursor's `.mdc`. Inclusion is explicit: the user adds a line to `~/.claude/CLAUDE.md` (or their project `CLAUDE.md`) such as:

```text
@~/.claude/bleat-models.md
```

so the file is loaded as context for every session.

## Steps

### 1. Detect available models

Enumerate the model slugs you can pass to an `Agent` subagent in this session. That is the dependable source. The currently available Claude models and the default panel are listed in [Models](#models) below; the quad is chosen for cross-family, cross-tier diversity, and the single-role default stays out of the panels because it already covers the single-model roles. Ask the user to confirm or paste any additional slugs they want available. Never write a real slug you have not confirmed is available. The aliases `inherit-parent` and `auto` are always valid even though they are not detected slugs; both mean the role runs on the parent session's model, which the `Agent` call expresses by omitting `model`.

### 2. Load current state

The default role-to-model mapping is the rule shape shown in step 5 below. If `~/.claude/bleat-models.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults. A line whose role is not in step 5, such as `how critics`, is from a retired role. Drop it.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Prefer `AskUserQuestion` over free text. Offer these four options with these exact labels, and name the current budget when the sheet records one. With no sheet, say that `large` matches the skill defaults.

- `unlimited — max reasoning`
- `large — xhigh reasoning`
- `medium — high reasoning`
- `small — medium reasoning`

**(b) Apply it.** Build the working table from the skill defaults, and on a re-run keep any role you changed by model, list, or alias (`inherit-parent`, `auto`). The budget does not change any role value. It sets the reasoning effort every bleat subagent starts from: `unlimited`, `large`, `medium`, and `small` start at `max`, `xhigh`, `high`, and `medium` on the ladder `max` > `xhigh` > `high` > `medium` > `low`. The parent passes that level as the `Agent` call's `effort`, raising or lowering it per job as just-bleat-it's **Match the model to the job** describes. A model that does not take a level runs at its nearest one. `large` keeps the defaults.

**(c) Show the roles and confirm.** Show every role with its model, marking any real slug not in the detected set as needing a choice. Also list each line step 2 dropped. Ask whether to accept as-is or change specific roles, offering the detected models plus `inherit-parent` and `auto` as the options. Prefer `AskUserQuestion` over free text. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. `arena cross-judge pool` is also a list, but Arena selects one value from it whose model family differs from the parent's when possible. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every real slug written must be in the detected set. `inherit-parent` and `auto` always pass. The budget must be one of the four labels in step 3. If a chosen real slug is not available, stop and ask again.

### 5. Write the override sheet

Write `~/.claude/bleat-models.md` with the shape below. Overwrite the whole file so re-runs stay idempotent.

```markdown
# bleat model configuration

Per-role model overrides for bleat skills. Each bleat SKILL.md names its defaults in a Models section; the values here override those defaults. Delete a line to fall back to the skill default. A value of `inherit-parent` or `auto` runs that role on the parent session's model (the `Agent` call omits `model`); an alias entry in a panel list still counts toward that panel's fan-out.

feature, refactoring: claude-opus-5-5
bug-fix: claude-opus-5-5
perf-issue: claude-opus-5-5
hillclimb: claude-opus-5-5
judgment and prose: claude-opus-5-5
strongest judgment: claude-fable-5-1
how explorer: claude-opus-5-5
how explainer: claude-opus-5-5
why investigators: claude-opus-5-5
why synthesizer: claude-opus-5-5
reflect tooling: claude-opus-5-5
reflect judgment, divergent, synthesizer: claude-opus-5-5
arena runners: claude-opus-5-5, claude-fable-5-1, claude-sonnet-5-5
arena cross-judge pool: claude-opus-5-5, claude-fable-5-1, claude-sonnet-5-5
swarm workers: claude-opus-5-5
architect runners: claude-opus-5-5, claude-fable-5-1, claude-sonnet-5-5
interrogate reviewers: claude-opus-5-5, claude-fable-5-1, claude-sonnet-5-5
```

After the role lines, add one last line with the chosen budget label and its starting effort, such as `# budget: large (xhigh)`.

### 6. Wire it in

If `~/.claude/CLAUDE.md` does not already include `~/.claude/bleat-models.md`, append the `@~/.claude/bleat-models.md` line so it loads on every session. If the user prefers project scope, add the include to the project's `CLAUDE.md` instead.

### 7. Confirm

Tell the user where the override was written, the budget it records, and how it loads (via the `@` include in CLAUDE.md). Re-running this skill updates the override sheet.

### 8. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a project `verify` or `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke [`/create-verification-skill`](../create-verification-skill/SKILL.md). On no, move on without pushing.

### 9. Offer a formal verification skill (optional)

Check whether the project already has a formal verification skill (a project skill that runs a model checker, a proof, or another verifier against the code). If not, offer once: "want a formal verification skill, so agents can prove a key property of your code holds in every case in scope, not only the cases tests reach? I can generate one with /create-formal-verification-skill." On yes, invoke [`/create-formal-verification-skill`](../create-formal-verification-skill/SKILL.md). On no, move on without pushing.

## Models

Stamped from `plugins/bleat/models.json` (edit there, rerun `tools/generate.mjs`).

- Available Claude models: Opus 5.5 (`claude-opus-5-5`), Opus 5 (`claude-opus-5`), Fable 5.1 (`claude-fable-5-1`), Fable 5 (`claude-fable-5`), Sonnet 5.5 (`claude-sonnet-5-5`), Sonnet 5 (`claude-sonnet-5`)
- Default panel: `claude-opus-5-5`, `claude-fable-5-1`, `claude-sonnet-5-5`
- Single-role default: `claude-opus-5-5`

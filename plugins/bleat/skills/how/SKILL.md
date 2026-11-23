---
name: how
description: "Use for \"how does X work\", code walkthroughs before changing something, and placement / ownership / layering questions (\"where should this live\", \"which package owns this\", \"is this the right layer\"). Explains subsystem architecture, runtime flow, onboarding mental models. Use why for motivation."
menu-description: walk through how a subsystem works
---

# How

Explore the codebase to answer "how does X work?" questions.

**Platform note.** On Codex or another non-Claude runtime, the Claude tool names and `claude-*` slugs named below are Claude defaults. Resolve them via [`codex-tools.md`](../just-bleat-it/references/codex-tools.md).

If the scope is ambiguous, state your interpretation and explore. The user can redirect.

## 1. Explain

Spawn one `Agent` subagent with `subagent_type: "general-purpose"`, told not to edit files, that explores and explains in one pass. Set `model` to the `how explainer` line in `~/.claude/bleat-models.md`, or to its default in [Models](#models) if the sheet or the line is missing. Leave `model` unset when the value is `auto` or `inherit-parent`. If the `Agent` tool rejects a slug, use the default and say so. Pick its reasoning level for the job, per the Subagents section of the **just-bleat-it** skill. Give it the user's question and this brief:

> You are writing an architectural explanation for a senior engineer. Explore the codebase and write an explanation that a senior engineer unfamiliar with this area could read and walk away with a solid mental model. They should understand the architecture well enough to start working in it confidently. Do not edit any files.

## 2. Present

Present the subagent's output. Light edits for clarity or context from the conversation are fine. Do not substantially rewrite it.

## Models

Role defaults, stamped from `plugins/bleat/models.json` (edit there, rerun `tools/generate.mjs`). A matching role line in `~/.claude/bleat-models.md` overrides each at runtime; see `/setup-bleat`.

- how explainer: `claude-opus-5-5`

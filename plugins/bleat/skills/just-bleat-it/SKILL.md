---
name: just-bleat-it
description: poteto's agent style for concise, detailed responses, deliberate subagents, unslopped prose, simple code, and verified work. Use for poteto, /just-bleat-it, or requests to work in this style.
menu-description: default entry point for any non-trivial task
---

# Just-bleat-it mode

Bundled skills live at `../<name>/SKILL.md` from this file, so put this file's absolute path in every `bleat-agent` prompt.

## Platform Adaptation

These skills use Claude Code tool names (the `Skill` tool, the `Agent` tool, `AskUserQuestion`) and Claude model slugs (`claude-*`). On Claude Code they work as written. On Codex and other runtimes, the skills are the same files; only the tool, model, and built-in-skill names resolve differently. When a skill names a Claude tool, a `claude-*` model, or a Claude built-in skill (`run`, `verify`, `plugin-dev:skill-development`), read [`references/codex-tools.md`](references/codex-tools.md) for the Codex equivalent.

## Non-negotiables

The Principles section below grounds every trigger. In your reply, name the principles that shaped decisions and the choice each changed. A sentence per principle carries both; the justification stays in the work, not the reply. Cite only principles whose leaf SKILL.md you read this session.

The remaining triggers live only here:

- First message → restate the user's goal and the smallest change that meets it, then proceed. Open each subagent brief, PR `## Why`, and final report with the goal.
- Starting any subagent with the `Agent` tool → the first line of its kickoff is `Read <path>/worker.md before anything else.`, with `<path>` the absolute path of this skill's folder. The operator's rules live in skills that only load from a slash command, so they reach another agent no other way. Keep the rest of the kickoff to the goal and the task, plus this file's absolute path for a `bleat-agent`.
- A fork you could settle by running something → prototype it per `playbooks/prototype.md` instead of asking.
- Code → name the data shape first. Across a function boundary → **architect**. Fan-out → **swarm**. Bakeoff → **arena**. Contested design → **interrogate**. For Bug fix only, check the [focused-fix exception](playbooks/bug-fix.md#focused-fix-exception) before the `how`, `why`, or `architect` fan-out.
- Prose, your reply included → **unslop**. Docs, PR descriptions, commit messages → **technical-writing**. Agent-facing prose also follows **plugin-dev:skill-development**.
- Before commit → **deslop** (`/deslop`). Before review → **no-comments**.
- Shipping UI / IDE / CLI → the Claude Code built-ins, `run` for CLIs and TUIs, `verify` for UIs.
- A number you measured → **benchmark-checklist**.
- PR status → the **Babysit** playbook, not the bundled **babysit** skill. Landing a green stack → the **Shipping** playbook.
- Bot review comments → judge each on its merits.
- Long or unattended work → **show-me-your-work**.

## Principles

Read the leaf skill in full for any principle you apply. Each entry names when it applies. Each leaf is the **principle-<title>** skill, with the title in kebab case, hyphens kept and other punctuation dropped, unless the entry gives its name.

**Core**

- **Laziness Protocol.** Refactoring, sizing a diff, or tempted to add abstractions, layers, or signal threading. Bias to deletion and the smallest change that solves the problem.
- **Foundational Thinking.** Before writing logic: core types and data structures, scaffold-vs-feature sequencing, what concurrent actors share.
- **Redesign from First Principles.** Integrating a new requirement into an existing design. Redesign as if it had been foundational from day one.
- **Attack the Premise.** Two or more fixes that share one premise have failed the same gate. Write the premise down and test it directly before you write another fix that assumes it.
- **Subtract Before You Add.** Sequencing an addition, refactor, or rewrite. Remove dead weight first, then build on the simpler base.
- **Minimize Reader Load.** Reviewing or shaping code that's hard to trace. Count layers and hidden state, collapse one-caller wrappers, shrink mutable scope.
- **Outcome-Oriented Execution.** Planned rewrites and migrations with explicit phase boundaries. Converge on the target architecture, don't preserve throwaway compatibility states.
- **Experience First.** Product, UX, or feature-scope tradeoffs. Choose user delight over implementation convenience.
- **Exhaust the Design Space.** A novel interaction or architectural decision with no precedent. Build 2-3 competing prototypes and compare before committing.
- **Build the Lever.** Any non-trivial work. Build the tool that does or proves it (codemod, script, generator), not by hand. The tool is the artifact a reviewer reruns.

**Architecture**

- **Model the Domain.** Writing stateful logic, code that branches a lot or repeats a shape assumption across files, or a record with more than one way in. Encode the domain in a structure (state machine, typed model, table or registry, reducer, boundary, the right collection) instead of scattered conditionals.
- **Boundary Discipline.** Wiring validation, access checks, error handling, or framework adapters. Guards at system boundaries, access checks that refuse when they find nothing to check, trust internal types, keep business logic pure.
- **Enforce Where It Is Stored.** More than one client writes the same data, or a client fills in a field to suit the server. The store enforces the rule for every writer.
- **Type System Discipline.** Designing types or a signature in any typed language. Make illegal states unrepresentable, brand primitives, parse external data at boundaries.
- **Make Operations Idempotent.** Designing commands, lifecycle steps, or loops that run amid crashes and retries. Converge to the same end state.
- **Migrate Callers Then Delete Legacy APIs.** Introducing a new internal API while old callers exist. Migrate and delete in one wave.
- **Separate Before Serializing Shared State.** Concurrent actors might write the same file, branch, key, or object, or parallel work shares one transaction or deploy step. Eliminate the sharing first.

**Verification**

- **Prove It Works.** After a task, before declaring done. Verify against the real artifact, not a proxy or "it compiles".
- **Fix Root Causes.** Debugging. Trace each symptom back to the first place the system went wrong and fix it there. A guard is a fix only when the state it handles is legitimate.
- **Sequence Work into Verifiable Units** (**principle-sequence-verifiable-units**). Multi-step work (sweeps, migrations, runs of similar edits) and how you stack commits and PRs. Break work into small units that each end in a check, verify each before the next, and order delivery so the sequence proves itself.
- **Test Behavior, Not Implementation.** Writing, changing, or keeping a test. Assert what the code's users can observe, so the test fails when that behavior breaks, keeps passing when only the internals change, and gives the same answer however it runs.
- **Test Against Real Shapes.** Writing a test, seed, fixture, or migration, or checking a change before it ships. Run it on the shapes production holds (old rows, missing settings, several tenants, rows other clients wrote), not only the clean fixture.
- **Explain the Number.** Apply before you measure, and before you trust, report, or act on a number you measured (a speedup, a regression, a throughput, a latency, or an eval result). Find what limits it, and rule out that it measured something other than the work you think.

**Delegation**

- **Guard the Context Window.** Context fills up: large outputs, long files, repeated reads, fan-out planning. Route bulk to subagents, keep summaries in the main thread.
- **Never Block on the Human.** Tempted to ask "should I do X?" on reversible work. Proceed, present the result, let the human course-correct.

**Meta**

- **Encode Lessons in Structure.** You catch yourself writing the same instruction a second time. Encode it as a lint, metadata flag, runtime check, or script instead of more text.

## Autonomy

**Just do it.** Use any MCP tool. Reversible work and external actions (team chat, comments on an issue you are working, kicking off evals) proceed without asking.

**Always pause** for irreversible writes: force-push to shared branches, deploys, data deletion, customer messages, and filing, closing, or relabeling issues in a shared tracker. A team's bug counts and boards read those, so ask first, even for a bug you found yourself.

**Session overrides:** "Don't stop" / "going to bed" / "run until done" / "be fully autonomous" → keep going.

**No is an acceptable answer.** Asked whether to do something, invited to add scope, or shown an approach, reply with your real judgment. Decline, push back, or say "this doesn't earn its place" when true. A recommendation is a judgment, not a validation. Agreement is not the default, candor over sycophancy.

## Subagents

**Use `subagent_type: "bleat-agent"` for any subagent you spawn directly inside a playbook step.** The chat that receives `/just-bleat-it` runs the playbook itself. Routed workflow skills (`interrogate`, `swarm`) set their own `subagent_type`. Respect it.

**Defaults for every `Agent` call.** `run_in_background: true` (a steer or resume of a running agent too), full tool access (do not pick a subagent_type that strips MCP), file pointers not inlined context, and an explicit model and effort per role. `/setup-bleat` writes `~/.claude/bleat-models.md`, whose per-role lines override the defaults below, the role defaults in [Models](#models), and the role models in the routed skills (`how`, `why`, `arena`, `swarm`, `architect`, `interrogate`, `reflect`). A role with no line keeps its default. A line of `inherit-parent` or `auto` runs the role on the parent session's model, so omit `model` and spawn `general-purpose`, because `bleat-agent` defaults to Opus. Each code playbook reads its own line (`feature, refactoring`, `bug-fix`, `perf-issue`, or `hillclimb`), prose and judgment read `judgment and prose`, and trivial mechanical edits read `swarm workers`.

- Code, prose, and judgment: the role's model (Opus 5.5 by default) at effort `xhigh`.
- Trivial mechanical edits: the `swarm workers` model at effort `high`.
- Diverse review and design panels: one runner per model in the panel skill's Models section, each at effort `high`.

**Fable is an exception.** The `strongest judgment` role (Fable by default) runs outside a panel only when the user named it for this task or two Opus 5.5 xhigh runs of the same brief failed its check. Say which. Difficulty alone never qualifies.

**Match the model to the job.** Start from the role's line or default, keep its model, and set the effort the job needs, passed as the `Agent` call's `effort`. The `# budget` line that `/setup-bleat` writes sets each role's starting level. Panel seats keep their listed level. Raise any other role one step, up to xhigh, for the hardest or subtlest work, lower it one step for judgment, and two or more steps for routine work. No role runs above xhigh. Claude Code sets fast mode for the whole session, so a spawn changes only its effort. Verification always runs on the `swarm workers` model at effort high. A role on `auto` or `inherit-parent` stays on the parent session's model. If the `Agent` tool rejects a model or a level, never pass it again. Fall back to the role's line or default, then the calling skill's fallback, then the closest valid model of the same family (Opus, Fable, or Sonnet), then omit `model`. Say so.

You own every subagent's work. Review the diff and write your own summary. A second opinion is the same prompt on another model family.

**Fresh subagents by default.** Give a fix round, a follow-up, a retry, or the next queue item to a fresh subagent with the original brief, every later directive, and the prior agent's report and branch. Resume an existing subagent only when the work needs state that lives in it and is costly to move: its checkout, its uncommitted changes, or a process it still runs.

## Writing the reply

Write short declarative sentences with no long dash and no colon as a connector. Lead with what changes for the user. Give each claim its evidence or a label (measured, inferred, or guess). Never hand the human a check you could run. Keep every item the playbook's reply names, each as a sentence or two of prose, with no section headers. Never fabricate a link, citation, or transcript reference.

## Playbooks

Match the task to a playbook and open its steps at `playbooks/<title>.md`, with the title in kebab case. Copy its steps into a todolist first. Large, cross-cutting, or unattended work, or a task no playbook fits, routes to the **figure-it-out** skill.

Investigation, Triage, Bug fix, Perf issue, Hillclimb, Runtime forensics, Trace forensics, Feature, Refactoring, Prototype, Visual parity, Authoring a skill, Eval, Babysit, Shipping, Autonomous run, Autopilot-full, Autopilot-stack, Session pickup, Pause safely, Multi-phase plan, Worktree cleanup, Opening a PR.

## Models

Role defaults, stamped from `plugins/bleat/models.json` (edit there, rerun `tools/generate.mjs`). A matching role line in `~/.claude/bleat-models.md` overrides each at runtime; see `/setup-bleat`.

- feature, refactoring: `claude-opus-5-5`
- bug-fix: `claude-opus-5-5`
- perf-issue: `claude-opus-5-5`
- hillclimb: `claude-opus-5-5`
- judgment and prose: `claude-opus-5-5`
- strongest judgment: `claude-fable-5-2`

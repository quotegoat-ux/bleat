# Set up bleat

In this page you install the plugin, pick which models bleat uses, and run your first task. Setup is one command plus a short conversation.

## Install the plugin

In Claude Code, run:

```text
/plugin marketplace add quotegoat-ux/bleat
/plugin install bleat@bleat
```

Claude Code confirms the plugin is installed. It also prefixes plugin commands, so each command in this guide also answers to a `bleat:` prefix, such as `/bleat:setup-bleat`.

## Pick your models

Run:

```text
/setup-bleat
```

[`/setup-bleat`](../../skills/setup-bleat/SKILL.md) detects the models you have access to, asks for a reasoning budget, shows you each role (code delegates, judgment, the review panels), and asks what you want. Answer the questions. It writes `~/.claude/bleat-models.md`, a small file every bleat skill reads.

The defaults run at `xhigh` reasoning, the same as the `large` budget. `unlimited` raises every subagent to `max`. `medium` and `small` lower the reasoning and spend fewer tokens.

You only override what you care about. A role with no line in the sheet keeps the skill's default. To restore a default, delete that role's line. A rerun of `/setup-bleat` keeps any role whose model differs from the default. When a default changes, a sheet written before the change still pins the old default, so delete those role lines, or delete the file, then run `/setup-bleat` again.

You might want a role to run on whatever model your session uses. Set it to `inherit-parent` or `auto` and bleat omits the subagent `model` field, so the subagent inherits the parent session's model. Both values mean the same thing, and neither is a model slug. For a panel role the value is a list, and one subagent runs per entry, so the list length sets the panel size. Setup also configures `swarm workers`, the default model for every `/swarm` worker unless a race names a model for each arm.

## Accept the verification offers, or don't

At the end of setup, `/setup-bleat` looks for a way to prove app behavior in your project, either a `verify-*` skill or an existing harness. If it finds neither, it offers once to generate one with [`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md).

Say yes and it writes `.claude/skills/verify-<app>/`, a project-local skill that teaches agents to drive your app the way a user does. It proves the skill works once before handing it over. Say no and setup moves on. You can run `/create-verification-skill` yourself any time. [Verify and ship](./06-verify-and-ship.md#create-a-project-verification-skill) covers it in depth.

If you're new to bleat, say yes. An agent that can check its own work keeps going until the check passes. An agent that can't hands every result back to you to check by hand. Of everything in this guide, the verification skill pays off the most.

Then setup offers once to generate a formal verification skill with [`/create-formal-verification-skill`](../../skills/create-formal-verification-skill/SKILL.md), unless the project already has one. Tests check the cases someone thought to write. A formal verification skill checks one key property of your code, such as "a job never runs twice at once", in every case within a stated scope. Say yes when your code has concurrency, retries, a state machine, or a parser, where a bug is costly and tests rarely reach the case that breaks. Otherwise say no, and run it later if that changes. [Verify and ship](./06-verify-and-ship.md#prove-a-property-with-a-formal-verification-skill) covers it.

After setup, start a new session. The override sheet loads through the `@~/.claude/bleat-models.md` line that setup adds to your `CLAUDE.md`, so it applies to new sessions.

## Keep the cost in check

bleat spends extra tokens on subagents and review panels. That's the price of the rigor. To spend fewer:

- Rerun `/setup-bleat` and pick a smaller reasoning budget or cheaper models. A strong model in the main chat with cheaper, faster models in the code roles is a good split.
- Set a role to `auto` or `inherit-parent` so it runs on the chat's own model.
- Shorten a panel list. Each entry runs one subagent.
- Save `/just-bleat-it` for work that needs rigor. A small, obvious edit doesn't.

## Run your first task

Pick something real but small, and describe it the way you'd describe it to a colleague:

```text
/just-bleat-it add a --json flag to this command. text output stays byte-identical. verify both.
```

Watch the todo list. Its first items are the matched playbook's steps copied in, the Feature playbook for this prompt. If `/just-bleat-it` skips a step, the step stays in the list with `skip: <reason>`, so you can see what it chose not to do.

From here you can type normal follow-ups. `/just-bleat-it` is sticky. It stays on for the conversation until you opt out by saying so.

Next: [Route work through `/just-bleat-it`](./02-just-bleat-it.md).

# bleat

Quote Goat's agent-workflow toolkit for Claude Code. bleat turns a capable agent into a disciplined one: every nontrivial task starts with a todolist and a matched playbook, the work is grounded in twenty-one engineering principles, delegation is deliberate, prose carries no AI tells, and nothing counts as done until it is proven against the real artifact.

> if you want to go fast, go deep first. bleat helps you write less, but higher quality code. rigorous agent workflows you can parallelize with confidence.

## Install

In Claude Code:

```shell
/plugin marketplace add quotegoat-ux/bleat
/plugin install bleat@bleat
```

The plugin routes on its own once installed. A `SessionStart` hook (on startup, `/clear`, and after `/compact`) injects a short mandate that routes any non-trivial engineering task into `just-bleat-it` before the first response, and a `UserPromptSubmit` hook keeps the mode on for the rest of a session once you invoke it. Explicit user instructions take precedence, and dispatched subagents ignore the mandate. To opt out, delete `hooks/hooks.json` from the installed copy (`~/.claude/plugins/cache/bleat/bleat/<version>/hooks/hooks.json`); a plugin update restores it.

### Cloud sessions

Claude Code on the web starts every session from a fresh clone of the repository, so a plugin you installed with `/plugin install` on your own machine isn't there, and `/plugin` itself doesn't run in a cloud session. To use bleat in a repository's cloud sessions, declare the marketplace and the plugin in that repository's `.claude/settings.json` and commit the file. Claude Code installs the plugin at session start:

```json
{
  "extraKnownMarketplaces": {
    "bleat": {
      "source": { "source": "github", "repo": "quotegoat-ux/bleat" }
    }
  },
  "enabledPlugins": {
    "bleat@bleat": true
  }
}
```

The cloud environment needs network access that reaches `github.com`; the default **Trusted** access level allows it. The same file also registers the marketplace for anyone who opens the repository locally once they trust the folder.

To get bleat in every cloud session without editing each repository, add this repository as a marketplace on your claude.ai account. Open **Customize** in the claude.ai sidebar, go to the **Plugins** tab, click **+** under **Personal plugins**, choose **Add marketplace**, and enter `quotegoat-ux/bleat`. Turn bleat on, and every cloud and Cowork session loads it as `bleat@synced` with no marketplace or install step. On a Team or Enterprise plan, an Owner can add the same marketplace under **Organization settings > Plugins** so every member sees it. A repository that also declares `bleat@bleat` in its own settings takes precedence over the synced copy.

The skill-authoring routes work best with the `plugin-dev` plugin installed (`/plugin install plugin-dev@claude-plugins-official`); everything else runs without it.

## Start

- `/just-bleat-it` turns the mode on for the current session.
- `/bleat-off` turns it off.
- `/setup-bleat` maps each role (code, judgment, the review panels) to the models you actually have.

For a guided tour, read [the guide](plugins/bleat/docs/guide/README.md).

## Slash commands

Every skill is also a slash command. The mode routes to these on its own; call one directly when you want just that behavior.

| command | use it when |
| --- | --- |
| `/just-bleat-it` | default entry point for any non-trivial task |
| `/how` | walk through how a subsystem works |
| `/why` | investigate why something was built this way (parallel multi-MCP evidence) |
| `/architect` | settle types and module shape before writing code that crosses a function boundary |
| `/arena` | run N parallel attempts at the same task and pick the best parts |
| `/interrogate` | have three different models try to break a diff |
| `/automate-me` | draft your own personal -mode skill from recent transcripts |
| `/reflect` | capture a long task's lessons as a skill edit |
| `/tdd` | fix a bug by writing the failing test first, then the fix |
| `/typescript-best-practices` | ground type-system discipline in TypeScript syntax |
| `/teach` | explain a subsystem plainly by composing how + why |
| `/swarm` | fan out N parallel workers across slices or races, then return one aggregated report |
| `/technical-writing` | write docs, RFCs, readmes, PR descriptions, and commit messages to one layered standard |
| `/bro` | restate the last message in plain human language, no jargon |
| `/figure-it-out` | design a rigorous, auditable playbook for a task no bundled playbook fits |
| `/show-me-your-work` | log decisions to a reviewable tsv decision trail |
| `/blast-radius` | find what a change could break beyond the diff and prove safety by running code |
| `/recall` | catch up on recent working context from chat history, live state, and the shared record |
| `/setup-bleat` | configure bleat per-role model choices |
| `/unslop` | clean up writing by removing AI tells |
| `/no-comments` | strip comments before review, fix the accepted findings, encode claimed constraints |
| `/create-verification-skill` | generate a project-local verification skill and feature map |
| `/maintain-verification-skill` | re-sync a drifted verification skill and its feature map |
| `/deslop` | deslop a diff before commit |
| `/babysit` | monitor an open PR, fix CI/comments, keep it merge-ready |
| `/thermo-nuclear-code-quality-review` | extremely strict maintainability audit |
| `/make-pr-easy-to-review` | clean noisy history and improve PR description before review |
| `/fix-ci` | find failing PR checks, inspect logs, apply focused fixes |
| `/fix-merge-conflicts` | non-interactively resolve merge conflicts, validate, finalize |
| `/get-pr-comments` | fetch and summarize review comments from the active PR |
| `/what-did-i-get-done` | summarize authored commits over a user-chosen period |

## Subagents

`bleat-agent` is the routing subagent for delegated work inside the mode; spawn it with `subagent_type: "bleat-agent"`. `comment-sicko` is the read-only comment reviewer the `no-comments` skill spawns; invoke it through `/no-comments`, not directly.

## Running on Codex

The same skills ship as a Codex plugin (`plugins/bleat/.codex-plugin`). Link them into your cross-runtime skills directory:

```shell
git clone https://github.com/quotegoat-ux/bleat
cd bleat
for s in plugins/bleat/skills/*/; do ln -s "$PWD/$s" ~/.agents/skills/"$(basename "$s")"; done
```

Codex namespaces them under `bleat`, so they list as `bleat:just-bleat-it`, `bleat:tdd`, and so on. For slash shortcuts (`/just-bleat-it`, `/tdd`), link the prompt stubs:

```shell
mkdir -p ~/.codex/prompts
for c in plugins/bleat/.codex-plugin/prompts/*.md; do ln -s "$PWD/$c" ~/.codex/prompts/"$(basename "$c")"; done
```

The multi-model and parallel-subagent skills (`interrogate`, `arena`, `how`, `why`, `reflect`, `architect`) need subagents turned on in `~/.codex/config.toml`:

```toml
[features]
multi_agent = true
```

When a skill names a Claude Code tool, model slug, or built-in skill, [`codex-tools.md`](plugins/bleat/skills/just-bleat-it/references/codex-tools.md) gives the Codex equivalent. On Codex, `/setup-bleat` writes `~/.codex/bleat-models.md` with your Codex model slugs. Codex has no plugin-hook runtime, so the session auto-routing above is Claude Code only; invoke `bleat:just-bleat-it` by name, or add a standing instruction to `~/.codex/AGENTS.md`.

## Contributing

Layout, CI, the local checks, and the release rules live in [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT. Third-party notices live in [NOTICE.md](NOTICE.md).

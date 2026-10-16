# Porting notes

This repo is a Claude Code port of [cursor/plugins `pstack`](https://github.com/cursor/plugins/tree/main/pstack) at upstream version `0.15.25`. The table below is the translation every file followed. Apply it again when syncing a newer upstream.

## Dropped

- `.cursor-plugin/plugin.json` and the root `plugin.json`. Replaced by `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`.
- `automations/benny/`. It targets Cursor Automations (webhook-triggered cloud agents), which Claude Code has no equivalent for.
- `skills/make-bot-ui/`. It is built on Cursor's Grok Bot product (routines, secret cards, Cursor webhooks), which has no Claude Code equivalent.
- Fast-mode guidance. Claude Code has no per-subagent fast variant (`/fast` is a session toggle).

## Models

Cursor slugs encode family, effort, and speed (`grok-4.7-xhigh-fast`). Claude Code's Agent tool takes `model` (`opus`, `sonnet`, `haiku`, `fable`, or omitted to inherit the parent) and `effort` (`low` < `medium` < `high` < `xhigh` < `max`) as separate parameters.

| Upstream default | Port default |
| --- | --- |
| `grok-4.7-xhigh-fast` (code delegates, swarm workers, how explorer, why investigators, reflect tooling) | `sonnet` |
| `claude-opus-5-5-xhigh` (judgment and prose, hardest tasks, explainers, synthesizers) | `opus` |
| Diverse panels `claude-opus-5-5-xhigh, grok-4.7-xhigh-fast` | `opus, sonnet` |
| Model families (`claude-*` vs `grok-*`/`composer-*`) | Model tiers (`opus`/`fable` vs `sonnet`/`haiku`) |
| Effort token inside the slug | Agent `effort` parameter. The budget sets the default effort (`large` = `xhigh`) |

Per-role config moved from `~/.cursor/rules/pstack-models.mdc` (an `alwaysApply` rule) to `~/.claude/rules/pstack-models.md`, which Claude Code loads into every session. A role value is `<model>` or `<model>@<effort>`, for example `opus@max`. With no `@effort`, the role uses the budget's effort. `inherit-parent` and `auto` both mean omit `model`.

## Tools and parameters

| Cursor | Claude Code |
| --- | --- |
| `Task` tool, Task spawn | `Agent` tool, Agent spawn |
| `subagent_type: generalPurpose` | `subagent_type: "general-purpose"` |
| `subagent_type: "poteto-agent"` / `"comment-sicko"` | `subagent_type: "pstack:poteto-agent"` / `"pstack:comment-sicko"` (plugin agents are namespaced) |
| `readonly: true` | `subagent_type: "pstack:read-only"`, a bundled agent with edit tools disallowed |
| `readonly: false` | omitted (default) |
| `run_in_background: true` | omitted. Subagents run in the background and notify on completion |
| `environment: "cloud"` | `isolation: "worktree"` |
| `environment: "local"` | omitted |
| Resume, message, or queue a follow-up on a running agent | `SendMessage` to the agent's name or ID |
| `AskQuestion` | `AskUserQuestion` |
| `readFile` | `Read` |
| Cursor Plan Mode | Claude Code plan mode |

## Paths

| Cursor | Claude Code |
| --- | --- |
| `~/.cursor/skills/`, `.cursor/skills/` | `~/.claude/skills/`, `.claude/skills/` |
| `~/.cursor/rules/*.mdc`, `.cursor/rules/*.mdc` | `~/.claude/rules/*.md`, `.claude/rules/*.md`, or `CLAUDE.md` |
| `~/.cursor/plugins/.../pstack/` | `${CLAUDE_PLUGIN_ROOT}` inside skill and agent bodies. Installed copies live at `~/.claude/plugins/cache/<marketplace>/pstack/<version>/` |
| `~/.cursor/projects/<slug>/agent-transcripts/` | `~/.claude/projects/<slug>/*.jsonl`, where `<slug>` is the absolute working directory with `/` and `.` replaced by `-` |
| `AGENTS.md` | `CLAUDE.md` (keep `AGENTS.md` where a repo already uses it) |

## Cross-plugin references

| Cursor | Claude Code |
| --- | --- |
| `cursor-team-kit` `/deslop` | the built-in `/simplify` skill, plus pstack's own `unslop` for prose |
| `cursor-team-kit` `control-cli` / `control-ui` | a project `verify-*` skill made with `/create-verification-skill` |
| Cursor's built-in `create-skill` | Anthropic's `skill-creator` skill when installed, else the `technical-writing` and `unslop` skills |
| Cursor's built-in `babysit` skill | removed. There is no conflicting built-in |
| Bugbot | unchanged. Cursor Bugbot is a GitHub reviewer and works on any PR |

## Skill names

Claude Code derives the slash command from the directory, and plugin skills are namespaced (`/pstack:how`). The bare `/how` also works unless another skill takes the name. Skill `name` fields were made kebab case (`Poteto Mode` became `poteto-mode`).

## Loading sibling skills

Upstream marks every skill except `setup-pstack` with `disable-model-invocation: true`, so nothing fires unless the user calls it. The port keeps that. In Claude Code this also makes the Skill tool refuse those skills when the model calls them, which would break routing such as poteto-mode's "→ the **how** skill". Every skill or agent that routes to a sibling therefore carries one line telling the model to Read `${CLAUDE_PLUGIN_ROOT}/skills/<name>/SKILL.md` instead of calling the Skill tool. Add that line to any upstream skill that starts routing to a sibling.

## Other harnesses

`install.py` installs the skills into `~/.agents/skills`, which Codex, OpenCode, Amp, and Gemini CLI all read, and converts `agents/*.md` per harness. It writes Codex TOML (`~/.codex/agents`), OpenCode Markdown with `mode: subagent` (`~/.config/opencode/agents`), and Gemini CLI Markdown (`~/.gemini/agents`). When converting, it replaces `${CLAUDE_PLUGIN_ROOT}` with the checkout path and renames `pstack:<agent>` to `pstack-<agent>`. `skills/poteto-mode/references/harnesses.md` maps Claude Code tools and parameters for the model at run time. Keep both in step when an upstream sync adds an agent or a new Agent parameter.

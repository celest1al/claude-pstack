# Running pstack outside Claude Code

pstack's skills name Claude Code's tools and parameters. In another harness, map each one with the table below before acting on it. Keep each skill's intent. When a parameter has no equivalent, drop it and say so in the reply once.

## Agents

`install.sh` converts pstack's agents for each harness and names them `pstack-<name>`.

| pstack names | Codex CLI | OpenCode | Gemini CLI | Amp |
| --- | --- | --- | --- | --- |
| `subagent_type: "pstack:poteto-agent"` | custom agent `pstack-poteto-agent` | subagent `pstack-poteto-agent` | subagent `pstack-poteto-agent` | the built-in Task subagent, with the poteto-agent prompt pasted in |
| `subagent_type: "pstack:read-only"` | custom agent `pstack-read-only` | subagent `pstack-read-only` (edits denied) | subagent `pstack-read-only` | Task subagent told not to edit |
| `subagent_type: "pstack:comment-sicko"` | custom agent `pstack-comment-sicko` | subagent `pstack-comment-sicko` | subagent `pstack-comment-sicko` | Task subagent with the comment-sicko prompt |
| `subagent_type: "general-purpose"` | the default agent | the `general` subagent | the generalist subagent | the Task subagent |

Amp has no file-based agents. Its Task subagent gets the agent's prompt from `<pstack>/agents/<name>.md`, minus the frontmatter.

## Agent tool parameters

| Claude Code | Codex CLI | OpenCode | Gemini CLI | Amp |
| --- | --- | --- | --- | --- |
| Agent tool | `spawn_agent` | Task tool | the subagent's own tool | Task tool |
| `model: "opus"` / `"sonnet"` | `model` on spawn: the strongest and the fast coding model you have | not per spawn. The subagent inherits the parent's model | not per spawn | not per spawn |
| `effort` | `reasoning_effort` on spawn | drop | drop | drop |
| `isolation: "worktree"` | run the worker in its own `git worktree add` checkout and tell it the path | same | same | same |
| `SendMessage` to a running agent | follow up on the spawned thread | start a fresh subagent with consolidated scope | start a fresh subagent | start a fresh subagent |
| `AskUserQuestion` | ask in the reply | the question tool, else ask in the reply | ask in the reply | ask in the reply |

A diverse panel (`opus, sonnet`) needs two different models to stay diverse. Where the harness cannot pick a model per spawn, run the panel on one model and say the panel was not diverse.

## Paths and config

| Claude Code | Elsewhere |
| --- | --- |
| `${CLAUDE_PLUGIN_ROOT}/skills/<name>/SKILL.md` | `../<name>/SKILL.md` from the current skill's directory |
| `~/.claude/rules/pstack-models.md` | the same file if it exists, else the skill defaults. Codex reads `~/.codex/AGENTS.md`, OpenCode `~/.config/opencode/AGENTS.md`, Gemini CLI `~/.gemini/GEMINI.md`, and Amp `~/.config/amp/AGENTS.md` |
| `~/.claude/projects/<slug>/*.jsonl` transcripts | the harness's own session store. Say when no transcript is reachable |
| `CLAUDE.md` | `AGENTS.md` (Gemini CLI: `GEMINI.md`) |
| `/simplify` | a manual simplification pass over the diff |

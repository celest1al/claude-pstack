---
name: setup-pstack
description: Configure which models pstack uses per role and at what reasoning budget. Writes a user-level Claude Code rule that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

pstack skills are user-invocable only. To use a sibling pstack skill this file names (the **<name>** skill, `/<name>`, or `/pstack:<name>`), read `../<name>/SKILL.md` relative to this skill's directory and follow it. Under Claude Code that file is `${CLAUDE_PLUGIN_ROOT}/skills/<name>/SKILL.md`, and the Skill tool refuses pstack skills, so never call it for one.

Outside Claude Code, map every agent, tool, and parameter below with `../poteto-mode/references/harnesses.md`.

Write `~/.claude/rules/pstack-models.md`, a user-level rule that Claude Code loads into every session and that sets pstack's model and effort per role.

## Steps

### 1. Detect available models

The Agent tool's `model` parameter takes `opus`, `sonnet`, `haiku`, or `fable`. Read the Agent tool's schema in this session and treat the values it lists as the detected set. Not every account can run every model. If the user says a model is unavailable to them, drop it from the set. The aliases `inherit-parent` and `auto` are always valid even though they are not models.

### 2. Load current state

The default role-to-model mapping is the rule shape shown in step 5 below. If `~/.claude/rules/pstack-models.md` already exists, read it and treat its `# budget` line and its role values as the current choices. Otherwise start from those defaults. A line whose role is not in step 5, such as `how critics`, is from a retired role. Drop it.

### 3. Budget, map, and confirm

**(a) Ask for a budget.** Use AskUserQuestion. Offer these four options with these exact labels, and name the current budget when the rule records one. With no rule, say that `large` matches the skill defaults.

- `unlimited (max effort)`
- `large (xhigh effort)`
- `medium (high effort)`
- `small (medium effort)`

**(b) Apply it.** The budget sets the `# budget` line, and every role without its own `@effort` runs at that effort. On a re-run keep any role you changed by model, list, alias (`inherit-parent`, `auto`), or explicit `@effort`. The effort ladder is `max` > `xhigh` > `high` > `medium` > `low`. Some models do not offer every level. If a role's model rejects the budget's effort at spawn time, the skills fall back to the role default, so warn the user when a chosen model is known to lack that level.

**(c) Show the roles and confirm.** Show every role with its model and effective effort, marking any model not in the detected set as needing a choice. Also list each line step 2 dropped. Ask with AskUserQuestion whether to accept as-is or change specific roles, offering the detected models plus `inherit-parent` and `auto` (both mean this role runs on the parent chat model) as the options. For panel roles (arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, alias entries included, so the list length sets the count. A panel stays diverse only when its entries name different models. `arena cross-judge pool` is also a list, but Arena selects one value from it whose model differs from the parent's when possible. `swarm workers` is the default model for every worker unless a race or comparison assigns another model per arm.

### 4. Validate

Every model written must be in the detected set, and every `@effort` must be on the ladder. `inherit-parent` and `auto` always pass. If a chosen model is not available, stop and ask again.

### 5. Write the rule

Create `~/.claude/rules/` if needed and write `~/.claude/rules/pstack-models.md` with no `paths` frontmatter, so it loads in every project. Include a `# budget` line with the chosen label and its effort, and one line per role, using the same labels poteto-mode uses. A role value is `<model>` or `<model>@<effort>`. Overwrite the whole file so re-runs stay idempotent. Shape:

```
# pstack model configuration

Overrides pstack skill defaults. One line per role. Delete a line to fall back to the skill default.
A value is `<model>` or `<model>@<effort>`. With no `@effort`, the role runs at the budget's effort.
`inherit-parent` or `auto` as a value: the role runs on the parent chat model (omit the Agent `model`). Alias entries in a panel list still count toward its fan-out.

# budget: large (xhigh)
feature, refactoring: sonnet
bug-fix: sonnet
perf-issue: sonnet
hillclimb: sonnet
judgment and prose: opus
hardest tasks: opus
how explorer: sonnet
how explainer: opus
why investigators: sonnet
why synthesizer: opus
reflect tooling: sonnet
reflect judgment, divergent, synthesizer: opus
arena runners: opus, sonnet
arena cross-judge pool: opus, sonnet
swarm workers: sonnet
architect runners: opus, sonnet
interrogate reviewers: opus, sonnet
```

### 6. Confirm

Tell the user the rule was written and that it applies to new sessions. Re-running this skill updates it.

### 7. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /create-verification-skill." On yes, invoke `/create-verification-skill` (resolves wherever pstack is installed: project, user, or plugin). On no, move on without pushing.

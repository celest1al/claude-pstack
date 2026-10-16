# Set up pstack

In this page you install the plugin, pick which models pstack uses, and run your first task. Setup is one command plus a short conversation.

## Install the plugin

In a Claude Code session, add the marketplace and install the plugin:

```text
/plugin marketplace add celest1al/claude-pstack
/plugin install pstack@pstack-marketplace
```

Run `/plugin` to confirm pstack is installed and enabled. To try a local clone instead, start Claude Code from the clone with `claude --plugin-dir .`.

Plugin skills are namespaced, so `/pstack:poteto-mode` always works. The bare `/poteto-mode` works too unless another skill takes the name. This guide uses the bare form.

## Pick your models

Run:

```text
/setup-pstack
```

[`/setup-pstack`](../../skills/setup-pstack/SKILL.md) asks for a reasoning budget, shows you each role (code delegates, judgment, the review panels) with its model, and asks what you want. Answer the questions. It writes `~/.claude/rules/pstack-models.md`, a small rule Claude Code loads into every session and every pstack skill reads.

Out of the box, code roles run on `sonnet` and judgment, prose, and the hardest changes run on `opus`. Panels pair `opus` with `sonnet`. The defaults run at `xhigh` effort, the same as the `large` budget. `unlimited` lifts every role to `max`. `medium` and `small` lower the effort to `high` and `medium` and spend fewer tokens. A single role can pin its own effort with `<model>@<effort>`, such as `opus@max`.

You only override what you care about. A role with no line in the rule keeps the skill's default. To restore a default, delete that role's line. A rerun of `/setup-pstack` keeps any role whose model differs from the default. When a default changes, a rule written before the change still pins the old default, so delete those role lines, or delete the file, then run `/setup-pstack` again.

Want a role to follow whatever model your chat uses? Set it to `inherit-parent` or `auto` and pstack omits the subagent `model` field, so the subagent inherits your parent chat model. Both values mean the same thing, and neither is a model name. For a panel role the value is a list, and one subagent runs per entry, so the list length sets the panel size. Setup also configures `swarm workers`, the default model for every `/swarm` worker unless a race names a model for each arm.

## Accept the verification offer, or don't

At the end of setup, `/setup-pstack` looks for a way to prove app behavior in your project, either a `verify-*` skill or an existing harness. If it finds neither, it offers once to generate one with [`/create-verification-skill`](../../skills/create-verification-skill/SKILL.md).

Say yes and it writes `.claude/skills/verify-<app>/`, a project-local skill that teaches agents to drive your app the way a user does. It proves the skill works once before handing it over. Say no and setup moves on. You can run `/create-verification-skill` yourself any time. [Verify and ship](./06-verify-and-ship.md#create-a-project-verification-skill) covers it in depth.

If you're new to pstack, say yes. An agent that can check its own work keeps going until the check passes. An agent that can't hands every result back to you to check by hand. Of everything in this guide, the verification skill pays off the most.

After setup, start a new session. Claude Code loads the rule at launch.

## Keep the cost in check

pstack spends extra tokens on subagents and review panels. That's the price of the rigor. To spend fewer:

- Rerun `/setup-pstack` and pick a smaller reasoning budget or cheaper models. A strong model in the main chat with cheaper, faster models in the code roles is a good split.
- Set a role to `auto` or `inherit-parent` so it runs on the chat's own model.
- Shorten a panel list. Each entry runs one subagent.
- Save `/poteto-mode` for work that needs rigor. A small, obvious edit doesn't.

## Run your first task

Pick something real but small, and describe it the way you'd describe it to a colleague:

```text
/poteto-mode add a --json flag to this command. text output stays byte-identical. verify both.
```

Watch the todo list. Its first items are the matched playbook's steps copied in, the Feature playbook for this prompt. If `/poteto-mode` skips a step, the step stays in the list with `skip: <reason>`, so you can see what it chose not to do.

From here you can type normal follow-ups. Once invoked, the skill's instructions stay in the conversation for the rest of the chat. Start a new chat to turn it off. After a long chat compacts, invoke `/poteto-mode` again with your next task to keep it sharp.

Next: [Route work through `/poteto-mode`](./02-poteto-mode.md).

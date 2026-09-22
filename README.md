# skills

My personal [Claude Code](https://claude.com/claude-code) skills. Written for my own workflow; use at your own risk.

## Skills

| Skill | What it does |
|---|---|
| [`jev`](jev/) | Calls [TypeSafe AI's Jev](https://typesafe.ai) for fast, typed judgments inside a session: safety gates, file triage, step verification, request routing. Needs `TYPESAFE_API_KEY`. |
| [`senior-dev`](senior-dev/) | Orchestrator mode. Plans, delegates to cheaper model tiers (haiku / sonnet / opus), verifies by execution. Modes `auto`, `low`, `high`. Uses `jev` for routing decisions when available. |

## Install

Each folder is a skill. Symlink the ones you want into `~/.claude/skills`:

```bash
ln -s "$PWD/jev" ~/.claude/skills/jev
ln -s "$PWD/senior-dev" ~/.claude/skills/senior-dev
```

Then invoke with `/jev` or `/senior-dev` in Claude Code.

## Jev API key

`jev` is optional. `senior-dev` works without it and falls back to its own judgment.

To enable it, create a key at https://console.typesafe.ai/ and store it:

```bash
~/.claude/skills/jev/bin/jev key <API_KEY>
```

This writes `env.TYPESAFE_API_KEY` into `~/.claude/settings.json`, which Claude Code
loads into every new session. If the key is missing and you invoke `/jev`, Claude will
ask you to paste it and run the same command for you. Note that a key pasted in chat
ends up in the session transcript; run the command yourself if you'd rather keep it out.

To compare Jev against Claude's own judgment on a question, say "compare with jev".
For routing comparisons in orchestrator mode use `/senior-dev auto --compare`.

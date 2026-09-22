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

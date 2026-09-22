---
name: jev
version: 0.1.0
description: |
  Call TypeSafe AI's Jev (a "System One" model) for fast, cheap, typed judgments
  inside a coding session: is this command destructive, is this file relevant,
  did this step succeed, which kind of request is this, which candidate is best.
  Returns a probability, a choice, or a score in ~100-500ms; never generates text.
  Use when asked to "ask jev", "use jev", "gate this with jev", "typesafe",
  or when a decision in the loop needs semantic judgment but not a frontier model.
triggers:
  - ask jev
  - use jev
  - gate with jev
  - typesafe
allowed-tools:
  - Bash
  - Read
---

# /jev — typed judgments for the agent loop

Jev answers **one narrow question over some state** and returns a typed value.
Code owns the workflow; Jev supplies the "gut check" at a fork. It does not write
code, explain, or reason in steps. Use a normal model for that.

Requires `TYPESAFE_API_KEY` in the environment (get one at https://console.typesafe.ai/).
Helper: `${CLAUDE_SKILL_DIR}/bin/jev` (curl + jq, no SDK).

## Primitives

| Need | Type | Returns | Note |
|---|---|---|---|
| Whether a condition holds | `noul` | probability of yes, 0..1 | 0.5 = uncertain, not "medium" |
| One of a defined set | `choice` | option + confidence | max 255 options; add a no-match option |
| Degree on an ordered scale | `score` | weighted level + confidence | 2..10 levels, each a concrete situation |

## CLI

```bash
jev ask --state <file-or-string> --noul "question"
jev ask --state <file-or-string> --choice "question" opt:desc opt:desc ...
jev ask --state <file-or-string> --score "question" level1 level2 ...
jev raw < request.json        # full control; batch many questions in one call
```
`--state` takes a file path (JSON if parseable, else text) or a literal string.
Build state as a JSON object with named fields when there are several parts.

## Recipes for a coding agent

**Safety gate** before a risky shell command. Above ~0.5, stop and ask the user.
```bash
printf '{"task":"%s","command":"%s"}' "$TASK" "$CMD" > /tmp/s.json
jev ask --state /tmp/s.json --noul "Is \`command\` destructive, irreversible, or outside the scope of \`task\`?"
```

**File triage** before reading. Score each candidate, read the top few.
```bash
jev ask --state /tmp/s.json --score "How relevant is \`path\` (with \`head\`) to \`task\`?" \
  "unrelated" "tangential" "directly relevant" "must read"
```

**Step verification** after tests or a tool call.
```bash
jev ask --state /tmp/s.json --noul "Does \`output\` show that \`goal\` is fully met?"
```

**Request routing** to pick effort, plan mode, or a skill.
```bash
jev ask --state "$USER_REQUEST" --choice "What kind of request is this?" \
  bugfix:"fix broken behavior" feature:"add new behavior" refactor:"restructure without behavior change" \
  question:"explain, no code change" docs:"documentation only" other
```

**Rerank candidates** (patches, search queries, file lists): put candidates in state as
named fields and ask a `choice` over their names, or one `score` per candidate.

## Rules

- One narrow judgment per question. Batch independent questions in a single `raw`
  request; they run in parallel and cannot see each other.
- Put the question in `instructions`, the possible answers in `criteria`. Reference
  state fields with backticks like `` `ticket.messages[0].text` ``.
- Thresholds are tuned per use on real cases. Confidence describes the distribution,
  not whether you are allowed to act.
- Typed output guarantees the shape, not the truth. Validate in your domain.
- Never use Jev for: exact lookups, arithmetic, anything a regex or exit code already
  answers, text generation, or multi-step reasoning.

## Live docs

Source of truth: https://docs.typesafe.ai/llms.txt (index), `/api.md`, `/primitives/{choice,noul,score}.md`,
`/confidence.md`, `/concepts/state.md`. Cookbooks: `/cookbooks/*.md`.

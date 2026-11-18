---
name: code-audit
version: 0.1.0
description: Read-only audit of a codebase or module for unnecessary complexity (single-implementation abstractions, pointless indirection, over-general code), duplicated logic, and assumptions that no longer hold (stale feature flags, compat shims, workarounds for fixed dependency bugs, unset config, dead branches, stale comments). Produces an evidence-backed, ranked report and waits for approval before changing anything. Invoke with /code-audit [path | module | "whole repo"]. Use when asked to "audit the codebase", "find dead code", "find duplication", "simplify this module", "tech debt sweep", or "what can we delete".
---

# Code audit

Scope: the argument (a directory, module, or file). No argument → whole repo.
State the scope in your first line.

## Hard rule: report first, no edits

Do NOT modify, create, or delete any project file during the audit. No
formatters, no codemods, no "quick fixes while I'm here". Read-only commands
only (read, grep, git log/blame, running existing tests is fine).

After the report, STOP and wait for the user to approve specific findings
(e.g. "do 1, 3, 4"). Approval covers only the findings named.

## What to look for

1. **Unnecessary complexity**
   - Interfaces / base classes / protocols with exactly one implementation
   - Factories, registries, strategies, or DI wiring for a single product
   - Wrappers and pass-through layers that add no behavior
   - Parameters, options, or generics that every caller passes the same way
2. **Duplicated logic**: the same behavior implemented in more than one place,
   including near-copies that drifted (same intent, slightly different edge
   handling — call out the difference, it may be a latent bug).
3. **Assumptions that no longer hold**
   - Feature flags that are always on/off, or whose rollout finished
   - Compatibility shims for runtimes / APIs / data formats no longer supported
   - Workarounds for dependency bugs fixed in the version now pinned
     (check the lockfile version against the upstream changelog/issue)
   - Config keys and env vars nothing sets
   - Dead branches, unreachable code, unused exports
   - Comments and docstrings that no longer match the code

Skip pure style nitpicks (naming, formatting, import order) entirely.

## How to gather evidence

Use the best search available: CodeGraph (`codegraph_explore`) if the repo has
`.codegraph/`, otherwise grep/ripgrep. For a large scope, fan out read-only
subagents per category or per top-level directory, then verify every finding
they return yourself before it goes in the report.

For every candidate, actively try to disprove it:
- Search for the symbol as a string too (reflection, `getattr`, dynamic
  `import()`, DI containers, route/handler registries, templates, serializers)
- Check whether it is public API: exported from a package entry point,
  documented, versioned, or used by other repos / plugins / scripts / CI / infra
- Check non-code callers: Makefiles, CI workflows, Dockerfiles, cron, migrations,
  `package.json` scripts, deploy config
- For "outdated" claims, cite the proof: `git log -S` / `git blame` showing why
  it was added, plus the lockfile version and changelog/issue showing it's fixed
- For flags/config: show where it's read AND that nothing sets it (env files,
  deploy config, defaults) — if it may be set outside the repo, say so

A finding you couldn't verify is either dropped or reported as low confidence
with what's missing.

## Report format

Open with a one-paragraph summary: scope, what was searched, counts by category.

Then a ranked list — **high impact + low risk first**, then high impact +
higher risk, then the rest. Number findings so the user can approve by number.

For each finding:

```
### N. <short title>  — [complexity | duplication | stale assumption]

- **Location:** path/to/file.ext:LINE (and every other relevant file:line)
- **Evidence:** call sites / "0 references" search with the exact query used /
  duplicated snippets side by side / git history or changelog link
- **Confidence:** high | medium | low
- **False-positive risks:** reflection, dynamic imports, public API, external
  callers, scripts — name the specific ones checked and the ones you couldn't
- **Smallest behavior-preserving change:** exactly what to delete / inline / merge
- **Verification:** existing tests that cover it (name them), or the
  characterization test to add BEFORE the change
- **Impact / Risk / Effort:** low|med|high each, one line of why
```

Keep evidence concrete (paths, line numbers, commands, short snippets). No
speculative findings without evidence.

End with:
- **Not reported:** areas skimmed but not audited, and why
- **Approval prompt:** "Which findings should I apply? (e.g. `1, 3, 5`)"

Then stop.

## After approval

- Apply only the approved findings, one at a time, smallest diff each.
- If a finding needs a characterization test first, add it and run it green
  against the unchanged code before making the change.
- Run the verification named in the report after each change; if it fails,
  revert that change and report instead of patching around it.
- Summarize what changed per finding number, and what was skipped.

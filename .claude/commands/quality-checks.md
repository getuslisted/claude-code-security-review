---
allowed-tools: Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git show:*), Bash(node:*), Bash(python3:*), Bash(rg:*), Bash(grep:*), Bash(bash:*), Bash(quality-checks/scripts/check.sh:*), Read, Glob, Grep, LS, Task
description: Final-line-of-defense quality pipeline. Nine phases. Composes security, design, UI/UX intelligence.
argument-hint: "[phase=N|N,N,N] [target=path]"
---

You are running Quality Checks v1.1, the nine-phase pipeline defined in `quality-checks/`.

## Inputs

```
!`git status`
!`git diff --name-only origin/HEAD...`
!`git log --no-decorate origin/HEAD...`
!`git diff --merge-base origin/HEAD`
```

## Pre-flight: deterministic gate

Run BEFORE LLM judgment:

```
!`bash quality-checks/scripts/check.sh 2>&1 || true`
```

The script catches the regex-detectable subset (11 rules). Treat as fact. If `Status: FAIL`, the gate failed.

## Argument parsing

- `phase=N` runs only phase N.
- `phase=N,M,P` runs only those phases.
- `target=<path>` scopes to a path.
- No arguments runs all nine phases.

## Reference loading

Load `quality-checks/PIPELINE.md`, `RUBRIC.md`, `CHECKLIST.md`, `ANTIPATTERNS.md`, `COPY-DENYLIST.md`. Load `BLOCKS.md` only for paste-replacement.

## Discovery (Phase 0)

1. **Register**. Task cue, surface, then `PRODUCT.md`.
2. **Design system map**.
3. **Stack**.
4. **Industry**. Use ui-ux-pro-max's `search.py` when present.
5. **Anti-references**.

Graceful degradation: missing or trivial `PRODUCT.md` does not halt; register / industry-specific checks are skipped with a nudge.

## Phase execution

Per `PIPELINE.md`. Incorporate pre-flight findings. Score 0-4 where applicable.

Use Task sub-tasks for Phase 1 (security; invoke `/security-review` in that repo, run inline elsewhere; confidence ≥ 8), Phase 4 (a11y, per file), Phase 8 (stack, per file).

## Composite & sign-off

1. Score /20 from five dimensions: Anti-Pattern, Design System, Accessibility, Performance, Resilience.
2. Severity census.
3. Security verdict.
4. Deterministic gate verdict.
5. Final verdict per `RUBRIC.md`.
6. Recommended commands.

## Output

Report per `RUBRIC.md` template. Markdown only.

If **Hold**, end with the next command. If **Ready to ship**, end with `Ready to ship.`

Begin pre-flight, then discovery.

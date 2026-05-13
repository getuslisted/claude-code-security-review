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

Run the deterministic checker BEFORE LLM judgment:

```
!`bash quality-checks/scripts/check.sh 2>&1 || true`
```

The script catches the regex-detectable subset of Phases 2, 4, 7, 8 (11 rules). Treat its output as already-decided fact. If it reports `Status: FAIL`, the deterministic gate has failed and the final verdict cannot be Ready to ship.

## Argument parsing

- `phase=N` runs only phase N (1-9).
- `phase=N,M,P` runs only those phases.
- `target=<path>` scopes the run.
- No arguments runs all nine phases.

## Reference loading

Load before phase work:

1. `quality-checks/PIPELINE.md`
2. `quality-checks/RUBRIC.md`
3. `quality-checks/CHECKLIST.md`
4. `quality-checks/ANTIPATTERNS.md`
5. `quality-checks/COPY-DENYLIST.md`
6. `quality-checks/BLOCKS.md` (only when paste-replacement is needed)

If any reference is missing, halt.

## Discovery (Phase 0)

1. **Register**. Task cue, surface, then `PRODUCT.md`.
2. **Design system map**. Walk for tokens.
3. **Stack**. From configs.
4. **Industry**. Use `ui-ux-pro-max-skill`'s `search.py` when present.
5. **Anti-references**. From `PRODUCT.md`.

### Graceful degradation

- `PRODUCT.md` present and non-trivial: full pipeline.
- `PRODUCT.md` present but trivial: pipeline runs; report nudges.
- `PRODUCT.md` missing: pipeline runs every phase EXCEPT register-specific and industry-specific checks; report nudges.

Never halts at Phase 0.

## Phase execution

For each requested phase:

1. State the phase name.
2. Run the checks per `PIPELINE.md`.
3. Incorporate the deterministic findings from pre-flight.
4. Record additional findings (P0 / P1 / P2 / P3).
5. Score 0-4 where applicable.
6. Note exit criterion pass / fail.

Use Task sub-tasks where parallelism helps:

- Phase 1: in `claude-code-security-review`, invoke `/security-review`. Elsewhere, run inline. Confidence ≥ 8.
- Phase 4: parallel per file.
- Phase 8: per stack-relevant file.

## Composite & sign-off (Phase 9)

1. **Audit Health Score** /20: Anti-Pattern + Design System + Accessibility + Performance + Resilience.
2. **Severity census**.
3. **Security verdict**: pass if zero HIGH, zero MEDIUM ≥ 0.85.
4. **Deterministic gate verdict**: pass if pre-flight exited 0.
5. **Final verdict** per `RUBRIC.md`.
6. **Recommended commands** in priority order.

## Output

Report per `RUBRIC.md` template. Markdown only.

If **Hold**, end with the next command. If **Ready to ship**, end with `Ready to ship.`

## Rules

- Reference loading is mandatory.
- Deterministic gate runs first; its output is fact.
- Denylist enforced verbatim.
- Confidence below threshold = drop.
- The user reviews the verdict; the pipeline does not auto-merge.

## Modes

`phase=2` anti-patterns only. `phase=4` a11y only. `phase=1,3,4` security + design + a11y. `target=src/Hero.tsx` one file.

Begin pre-flight, then discovery.

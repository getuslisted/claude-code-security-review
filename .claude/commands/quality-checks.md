---
allowed-tools: Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git show:*), Bash(git remote show:*), Bash(node:*), Bash(python3:*), Bash(rg:*), Bash(grep:*), Read, Glob, Grep, LS, Task
description: Final-line-of-defense quality pipeline. Nine phases. Composes security, design, and UI/UX intelligence.
argument-hint: "[phase=N|N,N,N] [target=path]"
---

You are running Quality Checks, the nine-phase pipeline defined in this repo's `quality-checks/` folder.

## Inputs from the harness

GIT STATUS:

```
!`git status`
```

FILES MODIFIED:

```
!`git diff --name-only origin/HEAD...`
```

COMMITS:

```
!`git log --no-decorate origin/HEAD...`
```

DIFF CONTENT:

```
!`git diff --merge-base origin/HEAD`
```

The diff above is the primary subject of the run. Read it carefully.

## Argument parsing

Parse `$ARGUMENTS`:

- `phase=N` runs only phase N (1-9).
- `phase=N,M,P` runs only those phases.
- `target=<path>` scopes the run to the path. The path may be a file, directory, or `<paste>` for raw HTML.
- No arguments runs all nine phases on the current branch diff.

If both `phase` and `target` are given, scope each named phase to the target.

## Reference loading

Before phase work, load:

1. `quality-checks/PIPELINE.md` — the phase definitions.
2. `quality-checks/RUBRIC.md` — the scoring and verdict.
3. `quality-checks/CHECKLIST.md` — the flat blocking checklist.
4. `quality-checks/ANTIPATTERNS.md` — the anti-pattern catalog.
5. `quality-checks/COPY-DENYLIST.md` — the editorial denylist.
6. `quality-checks/BLOCKS.md` — load only when the verdict suggests a paste-replacement.

If any reference is missing, halt and tell the user the file is absent. Do not invent rules.

## Discovery (Phase 0)

Before any judgment phase, run discovery:

1. **Register**. Read the task cue, the surface in focus, then `PRODUCT.md`. First match wins. Cache `brand` or `product`.
2. **Design system map**. Walk the project for token files, theme files, CSS variables, design-system documentation. Record presence or absence of: color tokens, type tokens, spacing tokens, radius tokens, shadow tokens, motion tokens, dark-mode variants.
3. **Stack**. Identify from `package.json`, framework configs, and file extensions.
4. **Industry**. Match against the 161 reasoning rules. Use `PRODUCT.md` if explicit; infer from Users / Product Purpose otherwise.
5. **Anti-references**. Pull `PRODUCT.md`'s anti-references list.

Output the discovery JSON to scratch state for the rest of the run.

If `PRODUCT.md` is missing or trivial (< 200 chars, contains `[TODO]`), halt and instruct the user to run `/impeccable teach` first.

## Phase execution

Run each requested phase per `PIPELINE.md`. For each phase:

1. State the phase name as it begins.
2. Run the phase's checks.
3. Record findings with severity (P0 / P1 / P2 / P3) and category.
4. Score the phase's dimension where applicable (0-4).
5. Note exit-criterion pass or fail.

Use sub-tasks (Task tool) for parallel work where the phase splits naturally:

- Phase 1 (Security): one sub-task to identify findings, then parallel sub-tasks per finding to filter false positives.
- Phase 4 (Accessibility): parallel sub-tasks per file in the diff.
- Phase 8 (Stack): sub-task per stack-relevant file.

For Phase 1 specifically, copy the false-positive filtering protocol from `claude-code-security-review/.claude/commands/security-review.md` verbatim into the sub-task prompts. Confidence threshold remains ≥ 8.

## Composite & sign-off (Phase 9)

After phases complete:

1. **Audit Health Score**: sum dimension scores into a /20 total. Map to a band per `RUBRIC.md`.
2. **Severity census**: count P0 / P1 / P2 / P3 across all phases.
3. **Security verdict**: pass if zero HIGH findings and zero MEDIUM with confidence ≥ 0.85.
4. **Verdict**: apply the sign-off rule from `RUBRIC.md`.
5. **Recommended commands**: list `/impeccable <command>` items in priority order. P0 first, then P1, then P2.

## Output

Emit the report in the exact shape from `RUBRIC.md`'s "Report template" section. Markdown only. Nothing else.

Sections, in order:

1. Verdict header (band, score, security, severity census).
2. Verdict paragraph.
3. Composite score table.
4. Top issues, by severity.
5. Drift map.
6. Recommended commands.
7. Positive findings.
8. Skipped phases (if any).

If the verdict is **Hold**, end with a single-line directive telling the user the next command to run. Do not pre-fix. Do not auto-merge. Hand control back.

If the verdict is **Ready to ship**, end with the literal text `Ready to ship.`

## Rules

- Reference loading is mandatory before judgment.
- The denylist is enforced verbatim; do not work around the regex.
- Confidence below the phase threshold means the finding is dropped.
- Skipped phases require a recorded reason.
- The user reviews the verdict; the pipeline does not auto-commit, auto-merge, or auto-publish.
- If the diff is empty (no changed files), state that and exit cleanly.

## Modes

`phase=2` — anti-pattern audit only. Useful for a fresh look at AI-slop tells without re-running security.

`phase=4` — accessibility only. Useful for a pre-launch a11y sweep.

`phase=1,3,4` — security + design system + a11y. Useful for production hotfix branches.

`target=src/components/Hero.tsx` — scope every phase to one file.

`target=<paste>` — read raw HTML from the user's next message and run the pipeline against it. Useful for vetting a block before paste.

Begin discovery now.

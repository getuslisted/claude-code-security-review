---
name: quality-gate
description: "Use as a final pre-ship pass on UI work. Runs eight sequential checks (preflight, security, industry reasoning, design laws, accessibility & performance, hardening, streamline, pre-delivery checklist) against pending changes or a named target. Synthesizes claude-code-security-review, impeccable, and ui-ux-pro-max-skill into one gate. Blocks ship when a P0 or P1 finding is unmitigated. Use after the feature is functionally complete; not a substitute for running impeccable's polish or harden during build."
argument-hint: "[target] [--phase <1-8>] [--strict]"
user-invocable: true
allowed-tools:
  - Bash(git diff:*)
  - Bash(git status:*)
  - Bash(git log:*)
  - Bash(git show:*)
  - Bash(node *)
  - Bash(npx *)
  - Read
  - Glob
  - Grep
license: Apache 2.0. Synthesizes patterns from claude-code-security-review (MIT), impeccable (Apache 2.0), and ui-ux-pro-max-skill (MIT). See each upstream for attribution.
---

A final pre-ship pass. Runs eight phases against the changed code (or a named target), assigns severity and confidence to every finding, and decides ship-or-block. Designed to be the last thing run before a feature ships.

## When to invoke

- After functional completion, before opening the PR.
- Before merging, on every push to a release branch.
- After a security-relevant change (auth, deserialization, file IO, query construction).
- After a UI-visible change (any file under `components/`, `pages/`, `routes/`, `app/`, `src/`, or matching the `*.{tsx,jsx,vue,svelte,astro,html,css}` glob).

Do not invoke in the middle of building. The phases assume the work is complete enough to assess. Use `/impeccable shape`, `/impeccable craft`, `/impeccable polish`, or `/impeccable harden` during build instead.

## Setup gates

Pass these before touching any phase. Skipping produces generic output.

| Gate | Required | If fail |
|---|---|---|
| Context | PRODUCT.md exists, not empty, not `[TODO]` placeholder. | Run `/impeccable teach`, then resume. |
| Register | `register: brand` or `register: product` is set in PRODUCT.md, or inferable from the surface in scope. | Infer from the route or component name; cache for the session. |
| Diff | A diff is available (`git diff --merge-base origin/HEAD` or named target). | Ask the user which target to scope to. |
| Stack | Framework, language, and styling system are known (read `package.json`, `Cargo.toml`, `pyproject.toml`, etc.). | Read manifest before running phase 2 or phase 5. |

State this preflight line before running any phase:

```text
QUALITY_GATE_PREFLIGHT: context=pass register=<brand|product> diff=<sha-range|target> stack=<framework>/<lang>/<styling>
```

## The eight phases

Run sequentially. A phase that finds a P0 stops the gate; later phases don't run until the P0 is acknowledged. Phases 2–8 each load their full reference file before executing.

| # | Phase | What it checks | Reference |
|---|---|---|---|
| 1 | **Preflight** | Context gates, register, stack, target inventory | [reference/phases/1-preflight.md](reference/phases/1-preflight.md) |
| 2 | **Security** | High-confidence vulnerability scan with hard exclusions | [reference/phases/2-security.md](reference/phases/2-security.md) |
| 3 | **Reasoning** | Industry pattern / style / color / typography fit | [reference/phases/3-reasoning.md](reference/phases/3-reasoning.md) |
| 4 | **Design laws** | Shared laws + absolute bans + AI slop test | [reference/phases/4-design-laws.md](reference/phases/4-design-laws.md) |
| 5 | **A11y & performance** | WCAG AA, focus path, Core Web Vitals, responsive | [reference/phases/5-a11y-performance.md](reference/phases/5-a11y-performance.md) |
| 6 | **Harden** | Text overflow, i18n, errors, edge cases, network | [reference/phases/6-harden.md](reference/phases/6-harden.md) |
| 7 | **Streamline** | Design system alignment, drift root-cause, distill | [reference/phases/7-streamline.md](reference/phases/7-streamline.md) |
| 8 | **Pre-delivery checklist** | Final gate; pass or block | [reference/phases/8-checklist.md](reference/phases/8-checklist.md) |

## Severity and confidence

Every finding carries both. The combination drives the ship/block decision.

**Severity (impact on the user):**

- **P0 (Block ship now)**: Exploitable vulnerability with confidence ≥8, broken core flow, complete a11y failure (no keyboard path, contrast 1:1, no focus indicator), data-loss risk.
- **P1 (Block ship before release)**: WCAG AA violation, absolute design ban present, missing required error/empty/loading state on a primary path, drift a user will notice.
- **P2 (Fix in the next pass)**: Token drift, minor responsive break, weak motion, copy inconsistency, dead code, missing memoization on a non-critical path.
- **P3 (Polish)**: Pixel-level alignment, micro-interaction tuning, optical centering, kerning.

**Confidence (evidence quality):**

- **9–10**: Direct evidence cited (file, line, value). Report and act.
- **7–8**: Strong pattern with a clear path. Report.
- **4–6**: Suspicious. Surface as a question to the user; don't claim it as a finding.
- **1–3**: Drop.

The hard rule: never report under confidence 7. False positives erode the signal value of every later finding.

Full rubric: [reference/severity-and-confidence.md](reference/severity-and-confidence.md).

## Output format

A single markdown report. Structured to be skimmable in 30 seconds and actionable in 5 minutes.

```markdown
# Quality Gate Report: <target> @ <sha>

## Verdict
**SHIP** | **BLOCK** (reason)

## Health Score
| Phase | Score | Worst finding |
|---|---|---|
| 2 Security | 4/4 | (none) |
| 3 Reasoning | 3/4 | Color palette is the category-reflex (`category: SaaS → purple gradient`) |
| 4 Design laws | 2/4 | Side-stripe border on `<Alert>` (P1, conf 10) |
| 5 A11y & perf | 3/4 | Focus indicator removed on `.btn-primary` (P1, conf 10) |
| 6 Harden | 3/4 | No empty state for `<ProjectList>` (P1, conf 9) |
| 7 Streamline | 4/4 | (none) |
| 8 Checklist | 6/8 | 2 items unchecked |
| **Total** | **25/32** | |

## Blocking findings (P0/P1)
1. **[P1, conf 10] Absolute ban: side-stripe border** at `components/Alert.tsx:18`
   - Evidence: `border-left: 4px solid var(--color-warning)`
   - Why: One of impeccable's six absolute bans. The colored side-stripe is the most recognizable AI-dashboard tell.
   - Fix: Replace with full 1px border + tinted background, or a leading icon + heading.

2. **[P1, conf 10] WCAG AA: focus removed without replacement** at `components/Button.tsx:42`
   - Evidence: `outline: none` with no `:focus-visible` style.
   - Fix: Add `:focus-visible { outline: 2px solid var(--color-focus); outline-offset: 2px; }`.

## Non-blocking findings
[P2 and P3 grouped by phase, with specific file:line for each]

## Pre-delivery checklist
- [x] PRODUCT.md / DESIGN.md present
- [x] No `#000` or `#fff`
- [ ] All interactive elements ≥44×44px on touch
- [x] `prefers-reduced-motion` respected
- [ ] Empty state for every list
- [x] Error states for every async action
- [x] Tested at 375 / 768 / 1024 / 1440
- [x] No console errors

## Recommended next moves
1. Fix the two P1 findings above (this run blocks on them).
2. Run `/impeccable polish components/Alert.tsx` to address the design law violation.
3. Re-run `/quality-gate` to confirm the gate now passes.
```

## Routing

| Argument | Behavior |
|---|---|
| (none) | Scope to `git diff --merge-base origin/HEAD`. Run all 8 phases. |
| `<target>` | Scope to a path, component name, or feature label. Run all 8 phases. |
| `--phase <n>` | Run only the named phase (1–8). For partial re-runs after a fix. |
| `--strict` | Block on P2 in addition to P0/P1. Default mode for release branches. |
| `--no-security` | Skip phase 2 (when running offline or when claude-code-security-review's GitHub Action already covers it). |

## Block recipes

When a finding's recommendation says "use the canonical pattern," the canonical patterns live in [reference/block-recipes.md](reference/block-recipes.md). Covers buttons, inputs, cards, modals, tables, alerts, navigation, and empty states. Every recipe is register-aware (brand vs product), framework-portable (works in React, Vue, Svelte, plain HTML), and meets all eight phases by construction.

## What this gate is not

- Not a SAST replacement. Phase 2 runs the same prompt as `/security-review`, but the GitHub Action gives full CI coverage and posts inline review comments. Run both.
- Not a design system generator. PRODUCT.md and DESIGN.md must already exist. Use `/impeccable teach` and `/impeccable document` to set them up.
- Not a build/type checker. Run `tsc`, `bun test`, `pnpm lint`, and the framework's preview build separately. The gate assumes those pass.
- Not for backend-only changes. Phases 3–8 are UI-focused. For pure backend, run only phase 2.

## Routing rules

1. **No argument**: scope to the current branch's diff against the merge base.
2. **First word matches a phase number** (e.g. `5`): treat as `--phase 5`.
3. **First word doesn't match a phase**: treat the entire argument as the target.
4. **Multiple targets**: comma-separate, run the gate per-target, aggregate the verdict.

## Anti-pattern catalog

The full list of detectable anti-patterns lives in [reference/anti-patterns-catalog.md](reference/anti-patterns-catalog.md). Consolidated from impeccable's 27 deterministic rules + 12 LLM rules, ui-ux-pro-max's industry exclusions, and the security review's hard-exclusion list. Every entry is paired with its source skill, severity assignment, and reference fix.

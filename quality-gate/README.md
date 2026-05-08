# Quality Gate

A final-line-of-defense skill that runs eight sequential checks against pending changes before they ship. Synthesizes three upstream skills:

- **[claude-code-security-review](https://github.com/getuslisted/claude-code-security-review)**: high-confidence vulnerability scan, hard exclusions, confidence-gated findings.
- **[impeccable](https://github.com/getuslisted/impeccable)**: shared design laws, absolute bans, AI slop test, audit dimensions, hardening.
- **[ui-ux-pro-max-skill](https://github.com/getuslisted/ui-ux-pro-max-skill)**: industry pattern matching, color and typography moods, pre-delivery checklist.

## What it does

Runs in eight phases against the diff (or a named target). Every finding gets a severity (P0–P3) and a confidence score (1–10). Findings under confidence 7 are dropped; under 8 require human review. Anything P0 or P1 blocks ship.

| # | Phase | Source | Blocks ship at |
|---|---|---|---|
| 1 | Preflight | impeccable setup gates | Missing PRODUCT.md/DESIGN.md, ambiguous register |
| 2 | Security | claude-code-security-review | HIGH-severity vuln with confidence ≥8 |
| 3 | Reasoning | ui-ux-pro-max-skill | Pattern/color/type mismatch with industry register |
| 4 | Design laws | impeccable shared laws + bans | Any absolute ban present, failed AI slop test |
| 5 | A11y & performance | impeccable audit + WCAG/CWV | WCAG AA contrast or focus failure, layout thrash |
| 6 | Harden | impeccable harden | Unbounded text, missing error/empty/loading state |
| 7 | Streamline | impeccable polish + distill | Drift unaccounted for, dead code, hard-coded tokens |
| 8 | Pre-delivery checklist | ui-ux-pro-max + synthesis | Any unchecked item |

## Install

Copy this directory to one of:

```bash
# Project-local (Claude Code)
cp -r quality-gate/ your-project/.claude/skills/

# User-wide
cp -r quality-gate/ ~/.claude/skills/
```

Then invoke:

```
/quality-gate                    # Run all 8 phases against the current diff
/quality-gate <target>           # Scope to a file, component, route, or feature name
/quality-gate --phase security   # Run a single phase
/quality-gate --strict           # Block on P2 as well as P0/P1
```

## How findings are scored

Severity is assigned by user impact:

- **P0 (Block ship now)**: Exploitable vuln, broken core flow, complete a11y failure (no keyboard path, contrast 1:1).
- **P1 (Block ship before release)**: WCAG AA violation, absolute ban present, missing required error/empty state, drift the user will notice.
- **P2 (Fix next pass)**: Token drift, minor responsive break, weak motion, copy inconsistency.
- **P3 (Polish)**: Pixel-level alignment, micro-interaction tuning.

Confidence is assigned by evidence:

- **9–10**: Direct evidence in the diff (line cited, value quoted).
- **7–8**: Strong pattern with a clear attack/UX path; report.
- **4–6**: Suspicious; surface as a question, not a finding.
- **1–3**: Drop.

## What this is not

- Not a SAST tool. The security phase delegates to claude-code-security-review's prompt and hard-exclusion list. Run that GitHub Action separately for full PR coverage.
- Not a design system generator. Use `/impeccable teach` and `/impeccable document` to set up DESIGN.md and PRODUCT.md first; this skill assumes they exist.
- Not a replacement for testing. It catches issues in the changed code, not regressions in surfaces it didn't read.

## Files

```
quality-gate/
├── SKILL.md                              # Orchestrator. Loads the right phase reference per invocation.
├── reference/
│   ├── phases/
│   │   ├── 1-preflight.md                # Context gates, register, target inventory
│   │   ├── 2-security.md                 # Synthesized from claude-code-security-review
│   │   ├── 3-reasoning.md                # Industry pattern / style / color / typography match
│   │   ├── 4-design-laws.md              # Shared laws, absolute bans, AI slop test
│   │   ├── 5-a11y-performance.md         # WCAG AA + Core Web Vitals + responsive
│   │   ├── 6-harden.md                   # Text overflow, i18n, errors, edge cases
│   │   ├── 7-streamline.md               # Design system alignment, drift root-cause, distill
│   │   └── 8-checklist.md                # Pre-delivery gate
│   ├── anti-patterns-catalog.md          # Consolidated catalog from all 3 sources
│   ├── severity-and-confidence.md        # Decision rubric
│   └── block-recipes.md                  # Reusable patterns: button, card, form, modal, table
└── README.md                             # This file
```

## Why these three skills

Each one closes a different gap.

- **Security review** has the false-positive discipline. Most LLM security passes flood with theoretical findings; the hard-exclusion list and ≥8 confidence threshold drop noise the way a senior engineer would.
- **Impeccable** has the design taste. Its absolute bans (side-stripe borders, gradient text, glassmorphism default, hero-metric template, identical card grids, modal-first) name the specific tells that mark AI-generated UI.
- **UI UX Pro Max** has the industry calibration. A banking dashboard and a wellness landing page need different palettes, typography moods, and effects; the 161 product-type rules carry that.

Together they form a pre-ship gate that catches what each one alone misses.

## License

Apache 2.0. Synthesizes patterns from three upstream skills under their respective licenses (Apache 2.0 for impeccable, MIT for the other two). See each upstream repo for full attribution.

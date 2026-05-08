# Rubric

How phase outputs combine into a single verdict. Every Quality Checks run produces one report in this exact shape.

## Audit Health Score

Five dimensions, scored 0-4 each. Total /20.

| Dimension | Source phase | 0 | 1 | 2 | 3 | 4 |
|-----------|--------------|---|---|---|---|---|
| Anti-Pattern | 2 | AI slop gallery (5+ tells) | Heavy AI aesthetic (3-4 tells) | Some tells (1-2 noticeable) | Mostly clean (subtle issues) | No AI tells, distinctive |
| Design System | 3 | Hard-coded everything | Mostly hard-coded, few tokens | Tokens used inconsistently | Tokens used, minor drift | Full token system, dark mode works |
| Accessibility | 4 | Fails WCAG A | Major gaps, no keyboard nav | Partial a11y, gaps remain | WCAG AA mostly met | WCAG AA fully met, approaches AAA |
| Performance | 5 | Layout thrash, unoptimized | Major problems, expensive animations | Some optimization, gaps | Mostly optimized | Fast, lean, well-optimized |
| Theming | sub of 3 | No theming | Minimal tokens | Partial token coverage | Tokens used well | Full theme switch, dark perfect |

### Bands

| Total | Band | Meaning |
|-------|------|---------|
| 18-20 | Excellent | Minor polish only |
| 14-17 | Good | Address weak dimensions before shipping |
| 10-13 | Acceptable | Significant work required |
| 6-9 | Poor | Major overhaul |
| 0-5 | Critical | Fundamental issues |

## Severity census

Count issues across all phases at each severity level.

| Severity | Definition | Action |
|----------|------------|--------|
| **P0** | Blocking. Prevents task completion or violates a hard gate | Fix immediately. No merge. |
| **P1** | Major. Significant difficulty or WCAG AA violation | Fix before release |
| **P2** | Minor. Annoyance, workaround exists | Fix in next pass |
| **P3** | Polish. No user impact. Nice to fix | Fix if time permits |

The severity census is reported as a tuple: `P0=0 P1=2 P2=4 P3=7`.

## Security verdict

Binary. Pass or fail.

- **Fail** if any HIGH-severity finding lands in the report.
- **Fail** if any MEDIUM-severity finding has confidence ≥ 0.85.
- **Pass** otherwise.

Security overrides the band. A run with Score 20 / 20 fails sign-off if Security fails.

## Sign-off rule

| Band | Security | P0 count | P1 count | Verdict |
|------|----------|----------|----------|---------|
| Excellent | Pass | 0 | ≤ 5 | **Ready to ship** |
| Good | Pass | 0 | ≤ 5 | **Ready to ship** |
| Good | Pass | 0 | 6-10 | **Ship with documented exception** |
| Acceptable | Pass | 0 | ≤ 5 | **Ship with documented exception** |
| Acceptable | Pass | 0 | 6+ | **Hold** |
| any | Fail | any | any | **Hold** |
| any | any | ≥ 1 | any | **Hold** |
| Poor or Critical | any | any | any | **Hold** |

## Report template

Every Quality Checks run emits this exact markdown shape.

```markdown
# Quality Checks — [branch / target]

**Verdict:** Ready to ship | Ship with exception | Hold
**Audit Health Score:** xx / 20 (Band)
**Security:** Pass | Fail
**Severity census:** P0=x P1=x P2=x P3=x

## 1. Verdict

[One paragraph. Why this verdict. The single most important thing to fix or celebrate.]

## 2. Composite score

| Dimension | Score | Top finding |
|-----------|-------|-------------|
| Anti-Pattern | x / 4 | [...] |
| Design System | x / 4 | [...] |
| Accessibility | x / 4 | [...] |
| Performance | x / 4 | [...] |
| Theming | x / 4 | [...] |

## 3. Top issues

### P0 (blocking)

- **[Title]**
  - Location: file:line
  - Phase: [phase name]
  - Category: [category]
  - Impact: [what breaks for users]
  - Fix: [specific action]
  - Suggested command: `/impeccable polish file.tsx`

### P1 (fix before release)

[...]

### P2 (fix in next pass)

[Truncate after first 5; refer to full report for the rest.]

## 4. Drift map

For each design-system deviation:

- **Location**: file:line
- **Drift class**: missing token | one-off implementation | conceptual misalignment
- **Found**: [the value or component as written]
- **Expected**: [the token, the shared component, the matching flow]
- **Root cause**: [the actual cause, not the symptom]
- **Fix**: [what to change first]

## 5. Recommended commands

In priority order. Address P0 before P1 before P2.

1. `/impeccable harden src/components/Form.tsx` (P0 i18n breaks at long German strings)
2. `/impeccable polish src/components/Hero.tsx` (P1 hard-coded color, two off-scale gaps)
3. `/impeccable clarify src/pages/Pricing.astro` (P1 banned terms in CTA copy)
4. `/impeccable critique src/pages/Index.astro` (P2 triadic auto-pilot in feature section)
5. `/impeccable optimize src/components/AnimatedHero.tsx` (P2 backdrop-filter on full-bleed element)

## 6. Positive findings

- [What's working well. Practices to maintain. Patterns to replicate.]

## 7. Skipped phases

- Phase 5: skipped (no animation, no large assets in diff).
- Phase 8: skipped (CSS-only diff).
```

## Exception protocol

A `Ship with exception` verdict requires a one-paragraph exception note in the PR description:

- What is being shipped despite the finding.
- Why it is acceptable to ship anyway (deadline, dependency on a separate fix, intentional tradeoff).
- A linked issue tracking the follow-up.

The reviewer who approves the PR records the exception in the merge commit message.

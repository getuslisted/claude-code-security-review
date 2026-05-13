# Rubric

How phase outputs combine into a single verdict.

## Audit Health Score

Five dimensions, scored 0-4 each. Total /20. One source phase per dimension; no double-counting.

| Dimension | Source phase | 0 | 1 | 2 | 3 | 4 |
|-----------|--------------|---|---|---|---|---|
| Anti-Pattern | 2 | AI slop gallery | Heavy AI aesthetic | Some tells | Mostly clean | No AI tells |
| Design System | 3 | Hard-coded everything | Mostly hard-coded | Tokens inconsistent | Tokens used, minor drift | Full token system |
| Accessibility | 4 | Fails WCAG A | Major gaps | Partial a11y | WCAG AA mostly | WCAG AA fully met |
| Performance | 5 | Layout thrash | Major problems | Some optimization | Mostly optimized | Fast, lean |
| Resilience | 6 | Happy-path only | Most states missing | Some empty / error states | All states present | Hardened for long text, RTL, errors, offline |

(v1.0 listed Theming as a separate dimension while describing it as a sub-component of Design System. That double-count is fixed in v1.1: Theming is folded into Design System, Resilience takes its slot.)

## Bands

| Total | Band | Meaning |
|-------|------|---------|
| 18-20 | Excellent | Minor polish only |
| 14-17 | Good | Address weak dimensions before shipping |
| 10-13 | Acceptable | Significant work required |
| 6-9 | Poor | Major overhaul |
| 0-5 | Critical | Fundamental issues |

## Severity census

| Severity | Definition | Action |
|----------|------------|--------|
| **P0** | Blocking. Hard gate violated | Fix immediately. No merge. |
| **P1** | Major. WCAG AA violation or significant difficulty | Fix before release |
| **P2** | Minor. Workaround exists | Fix in next pass |
| **P3** | Polish. No user impact | Fix if time permits |

Reported as `P0=0 P1=2 P2=4 P3=7`.

## Security verdict

Binary. Pass or fail.

- **Fail** if any HIGH-severity finding.
- **Fail** if any MEDIUM with confidence ≥ 0.85.
- **Pass** otherwise.

Security overrides the band.

## Deterministic-gate verdict

`scripts/check.sh` produces a pass / fail (P0 = 0 AND P1 = 0). Precondition to LLM judgment.

## Sign-off rule

| Band | Security | Det. gate | P0 | P1 | Verdict |
|------|----------|-----------|----|----|---------|
| Excellent | Pass | Pass | 0 | ≤ 5 | **Ready to ship** |
| Good | Pass | Pass | 0 | ≤ 5 | **Ready to ship** |
| Good | Pass | Pass | 0 | 6-10 | **Ship with documented exception** |
| Acceptable | Pass | Pass | 0 | ≤ 5 | **Ship with documented exception** |
| Acceptable | Pass | Pass | 0 | 6+ | **Hold** |
| any | Fail | any | any | any | **Hold** |
| any | any | Fail | any | any | **Hold** |
| any | any | any | ≥ 1 | any | **Hold** |
| Poor or Critical | any | any | any | any | **Hold** |

## Report template

```markdown
# Quality Checks — [branch / target]

**Verdict:** Ready to ship | Ship with exception | Hold
**Audit Health Score:** xx / 20 (Band)
**Security:** Pass | Fail
**Deterministic gate:** Pass | Fail
**Severity census:** P0=x P1=x P2=x P3=x

## 1. Verdict

[One paragraph.]

## 2. Composite score

| Dimension | Score | Top finding |
|-----------|-------|-------------|
| Anti-Pattern | x / 4 | [...] |
| Design System | x / 4 | [...] |
| Accessibility | x / 4 | [...] |
| Performance | x / 4 | [...] |
| Resilience | x / 4 | [...] |

## 3. Top issues

### P0 (blocking)

- **[Title]**
  - Location: file:line
  - Phase: [phase name]
  - Category: [category]
  - Impact: [what breaks for users]
  - Fix: [specific action]
  - Suggested command: `/impeccable polish file.tsx`

### P1

[...]

### P2

[Truncate after first 5.]

## 4. Drift map

For each design-system deviation:

- **Location**: file:line
- **Drift class**: missing token | one-off | conceptual
- **Found**: [the value as written]
- **Expected**: [the token / shared component / matching flow]
- **Root cause**: [the actual cause]
- **Fix**: [what to change first]

## 5. Recommended commands

1. `/impeccable harden src/Form.tsx` (P0)
2. `/impeccable polish src/Hero.tsx` (P1)

## 6. Positive findings

## 7. Skipped phases
```

## Exception protocol

`Ship with exception` requires a one-paragraph note in the PR description: what's shipping despite the finding, why it's acceptable, a linked follow-up issue.

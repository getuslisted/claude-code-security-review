# Severity and confidence rubric

Every finding the gate reports carries two scores. The combination drives the ship/block decision.

## Severity (impact on the user)

| Severity | Definition | Examples |
|---|---|---|
| **P0** | Blocks ship now. Fix before any further work. | Exploitable HIGH-severity vuln (conf ≥ 8); broken core flow; complete a11y failure (no keyboard path, contrast 1:1, no focus indicator); data-loss risk (form submit losing user input on error); placeholder used as the only label. |
| **P1** | Blocks ship before release. Fix this push. | WCAG AA contrast or focus failure; absolute design ban present (side-stripe border, gradient text, glassmorphism default, hero-metric, identical card grid, modal-as-first-thought); missing required error/empty/loading state on a primary path; drift Category C (conceptual misalignment); banned diction in user-facing copy. |
| **P2** | Fix in next pass. Doesn't block this release. | Token drift (hard-coded color used twice, should be a token); minor responsive break at one breakpoint; weak motion choice; copy inconsistency; dead code; placeholder contrast under 4.5:1. |
| **P3** | Polish. Address if time permits. | Pixel-level alignment; micro-interaction tuning; optical centering; kerning; magic numbers without naming. |

### Severity mapping from upstream skills

| Upstream | Their term | Maps to |
|---|---|---|
| claude-code-security-review | HIGH (conf ≥ 9) | P0 |
| claude-code-security-review | HIGH (conf 8) | P1 |
| claude-code-security-review | MEDIUM (conf ≥ 9, obvious) | P1 |
| claude-code-security-review | MEDIUM (conf 7–8) | P2, report only |
| claude-code-security-review | LOW | drop |
| impeccable | P0 (Blocking) | P0 |
| impeccable | P1 (Major / WCAG AA) | P1 |
| impeccable | P2 (Minor) | P2 |
| impeccable | P3 (Polish) | P3 |
| ui-ux-pro-max-skill | "Avoid" anti-pattern present | P1 |
| ui-ux-pro-max-skill | Pre-delivery checklist item failed | P1 |

## Confidence (evidence quality)

| Confidence | Action | What it looks like |
|---|---|---|
| **9–10** | Report and act. Direct evidence. | The exact code line cited, the exact value quoted. The finding is reproducible from the report alone. |
| **7–8** | Report. Strong pattern. | A clear pattern with a known exploitation or UX path. Reproduction may require one inference step. |
| **4–6** | Surface as a question, not a finding. | Suspicious. Worth asking the user about; not worth claiming as a defect. |
| **1–3** | Drop. | Speculation. The gate's signal value depends on dropping these. |

### The hard rule

**Never report a finding under confidence 7.** Each false positive halves the trust the next true positive gets.

If you find yourself wanting to report a confidence-5 issue, ask a question instead:

> "I noticed `<X>` in `<file:line>`. Is this intentional?"

Questions are free; false positives aren't.

## How severity and confidence interact

```
                            CONFIDENCE
                  1-3       4-6       7-8         9-10
              ┌─────────┬─────────┬───────────┬──────────┐
        P0   │  drop   │ question│ report+act│ block    │
              ├─────────┼─────────┼───────────┼──────────┤
        P1   │  drop   │ question│ report+act│ block    │
SEVERITY      ├─────────┼─────────┼───────────┼──────────┤
        P2   │  drop   │  drop   │ report    │ report   │
              ├─────────┼─────────┼───────────┼──────────┤
        P3   │  drop   │  drop   │ report    │ report   │
              └─────────┴─────────┴───────────┴──────────┘
```

## What block means

| Verdict | Action | Override available |
|---|---|---|
| **SHIP** | All clear. Open the PR. | n/a |
| **SHIP (with override: <reason>)** | The user accepted some failed checklist items. | Yes |
| **BLOCK** | Don't open the PR. Fix the listed P0/P1, re-run. | No (unless --override) |

The gate doesn't have authority to block a human's decision, but it does block by default. The user has to type `--override <reason>` to ship past it. The override gets logged.

## Worked examples

### Example 1: Security HIGH, confidence 10

```
SQL injection in app/api/search.py:42
- Severity: P0 (HIGH × confidence 10 → block)
- Evidence: cursor.execute(f"SELECT * FROM users WHERE name = '{name}'")
- Action: BLOCK
```

### Example 2: Design absolute ban

```
Side-stripe border on components/Alert.tsx:18
- Severity: P1 (absolute ban × direct evidence → block before release)
- Evidence: border-left: 4px solid var(--color-warning);
- Action: BLOCK
```

### Example 3: Suspicious but not certain

```
Possible focus loss when modal closes (components/EditDialog.tsx:91)
- Severity: P1 (a11y impact)
- Confidence: 5/10 (didn't verify in browser)
- Action: SURFACE AS QUESTION. "I wasn't able to verify that focus returns to the trigger after closing this modal. Can you confirm?"
```

### Example 4: Token drift, low impact

```
Hard-coded color in components/Button.tsx:24
- Severity: P2 (minor; not user-visible regression)
- Confidence: 10/10 (literal hex in the file)
- Action: REPORT (non-blocking). Suggest replacing with var(--color-accent).
```

### Example 5: Speculative, drop

```
The gradient on the CTA might be too saturated
- Severity: P3
- Confidence: 3/10 (subjective, no measured ratio, no anti-pattern hit)
- Action: DROP. Don't add to the report.
```

## Confidence calibration

Most LLM passes over-report confidence. The gate is calibrated against this tendency.

Self-check before reporting:

1. **Can I quote the exact line?** If no, confidence drops one tier.
2. **Can I write the exploit/UX scenario in one sentence?** If no, drop another tier.
3. **Could a human disagree with this and have a point?** If yes, drop another tier.
4. **Did I verify or only infer?** Verified → 9–10. Inferred → 7–8 max.

After all four checks, the confidence is what you can defend in PR review.

## What confidence is *not*

Confidence is **not** a measure of importance. A P0 with confidence 7 is still a P0; it gets reported. Confidence is a measure of *evidence quality*, not *issue severity*.

Don't conflate the two. The gate's discipline rests on keeping them separate.

# Phase 8 — Pre-delivery checklist

The final gate. Synthesizes the per-phase checks into a single boolean list. Synthesized from ui-ux-pro-max-skill's pre-delivery checklist plus impeccable's polish checklist.

## How this phase runs

After phases 1–7 complete, walk this list. Every item is a single yes/no. Any unchecked item is a **P1** blocker for ship. The user can choose to override (they're shipping; you don't get to veto), but the override goes on record in the report.

## The checklist

### Foundation

- [ ] **PRODUCT.md exists and is non-trivial** (preflight 1.1)
- [ ] **Register identified** (preflight 1.2)
- [ ] **Build, type check, tests, lint all pass** (preflight 1.5)

### Security

- [ ] **No HIGH-severity vulnerability with confidence ≥ 8** (phase 2)
- [ ] **No hardcoded secrets in the diff** (phase 2)
- [ ] **No `dangerouslySetInnerHTML` / `bypassSecurityTrustHtml` without justification** (phase 2)

### Industry fit

- [ ] **Pattern fit ≥ 3/4** (phase 3.2)
- [ ] **Color mood fit ≥ 3/4** (phase 3.2)
- [ ] **Category-reflex check passes both altitudes** (phase 3.3)

### Design laws

- [ ] **No `#000` or `#fff`** (phase 4)
- [ ] **No side-stripe borders** (phase 4 ban 1)
- [ ] **No gradient text** (phase 4 ban 2)
- [ ] **No glassmorphism without explicit justification** (phase 4 ban 3)
- [ ] **No hero-metric template** (phase 4 ban 4)
- [ ] **No identical card grid** (phase 4 ban 5)
- [ ] **No new modal that could have been inline** (phase 4 ban 6)
- [ ] **No layout-property animation** (phase 4)
- [ ] **No bounce or elastic easing** (phase 4)
- [ ] **No em dashes in user-facing copy** (phase 4)
- [ ] **No nested cards** (phase 4)

### Accessibility

- [ ] **Body text contrast ≥ 4.5:1** (phase 5.1)
- [ ] **Focus indicator visible on every interactive element** (phase 5.2)
- [ ] **No `outline: none` without `:focus-visible` replacement** (phase 5.2)
- [ ] **Every input has a label** (phase 5.3)
- [ ] **Heading hierarchy is sequential, single h1** (phase 5.3)
- [ ] **Icon-only buttons have `aria-label`** (phase 5.3)
- [ ] **Images have meaningful or empty alt** (phase 5.3)
- [ ] **Touch targets ≥ 44×44px on touch surfaces** (phase 5.5)
- [ ] **`prefers-reduced-motion` honored on every animation** (phase 4)

### Performance

- [ ] **`<img>` has explicit `width` and `height`** (phase 5.4)
- [ ] **Below-fold images are `loading="lazy"`** (phase 5.4)
- [ ] **No tree-shaking-broken imports** (phase 5.4)
- [ ] **No layout reads inside layout writes** (phase 5.4)
- [ ] **Bundle size delta is acceptable** (phase 5.4; project-defined budget)

### Responsive

- [ ] **Tested at 375px (mobile)** (phase 5.5)
- [ ] **Tested at 768px (tablet)** (phase 5.5)
- [ ] **Tested at 1024px (laptop)** (phase 5.5)
- [ ] **Tested at 1440px (desktop)** (phase 5.5)
- [ ] **No horizontal scroll at any breakpoint** (phase 5.5)
- [ ] **No fixed widths breaking on mobile** (phase 5.5)

### Hardening

- [ ] **Long text (200+ chars) renders correctly in every text container** (phase 6.1)
- [ ] **Empty state for every list / table / feed** (phase 6.4)
- [ ] **Loading state for every async action** (phase 6.4)
- [ ] **Error state for every async action** (phase 6.3)
- [ ] **Submit button disables during request** (phase 6.4)
- [ ] **User input preserved on validation error** (phase 6.3)
- [ ] **i18n: text expansion budget (30–40%) accommodated** (phase 6.2)
- [ ] **i18n: logical properties used (`padding-inline`, not `padding-left`)** (phase 6.2)
- [ ] **`Intl.*` for dates, numbers, currency** (phase 6.2)

### Streamline

- [ ] **No drift Category C (conceptual misalignment)** (phase 7.2)
- [ ] **Drift Category B (one-off implementation) count ≤ 2** (phase 7.2)
- [ ] **Hard-coded colors / spacing / durations replaced with tokens or noted as P2** (phase 7.2)
- [ ] **No `console.log`** (phase 7.4)
- [ ] **No commented-out code** (phase 7.4)
- [ ] **No `any` types without justification** (phase 7.4)
- [ ] **No unused imports** (phase 7.4)

### Final pass

- [ ] **The build is green** (preflight, re-confirmed)
- [ ] **The feature has been used end-to-end in a browser, not just compiled** (the most-skipped item)
- [ ] **The diff has been read line-by-line by the gate** (this run)

## Output format

```markdown
## Phase 8 — Pre-delivery checklist

| Section | Status | Failed items |
|---|---|---|
| Foundation | <pass|fail (n)> | |
| Security | <pass|fail (n)> | |
| Industry fit | <pass|fail (n)> | |
| Design laws | <pass|fail (n)> | |
| Accessibility | <pass|fail (n)> | |
| Performance | <pass|fail (n)> | |
| Responsive | <pass|fail (n)> | |
| Hardening | <pass|fail (n)> | |
| Streamline | <pass|fail (n)> | |
| Final pass | <pass|fail (n)> | |
| **Total** | <X / Y items pass> | |

### Unchecked items
[every unchecked item, with the phase reference, severity, and what unblocks it]
```

## Block decision

- **All items pass** → **SHIP**.
- **Any item fail** → **BLOCK**. The verdict line names the failed items and the unblock action for each.

## Override

Sometimes the user ships with known gaps. The gate doesn't refuse; it records.

If the user passes `--override <reason>`:

- Items still listed as failed.
- Verdict reads `SHIP (with override: <reason>)`.
- A `KNOWN_GAPS.md` line is written to the project root with the unchecked items, the override reason, and the date.

This makes the next run aware. The gate doesn't keep re-flagging things the user already declined to fix; it tracks them.

## Common mistakes

- **Treating the checklist as a suggestion.** Each item ties back to a specific phase finding. They're not aspirational.
- **Auto-checking items the gate didn't verify.** If phase 5 didn't actually compute the contrast ratio, don't check the contrast item. Be honest about what was and wasn't measured.
- **Skipping the "used end-to-end in a browser" item.** This is the highest-leverage check on the list. A feature can compile, type-check, test, and lint, and still be broken in ways only a real browser session catches.
- **Using --override as a default.** If the team finds itself overriding every run, the checklist is too strict for the project, or the project's bar is too low. Talk about it; don't normalize the override.

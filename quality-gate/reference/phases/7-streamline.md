# Phase 7 — Streamline

Align the diff to the design system, name and resolve drift, and remove what doesn't earn its place. Synthesized from impeccable's polish (design system discovery, drift root-cause classification) and distill (strip to essence) references.

## Why this phase exists

Most code review focuses on what was added. This phase focuses on what was added incorrectly: hard-coded values that should reference tokens, one-off implementations of components that already exist, conceptual misalignment between the new feature and the system around it. Drift compounds. The streamline phase is where it gets caught.

## 7.1 Design system discovery

Before judging drift, find the system. Search for:

- A `tokens.css`, `design-tokens.json`, or `theme.config.{ts,js}`.
- A `components/` or `ui/` directory with shared primitives.
- A storybook (`*.stories.tsx`).
- A DESIGN.md at the project root.
- Tailwind's `tailwind.config.{ts,js}` (the source of truth in Tailwind projects).
- The `:root` CSS variable declarations in the global stylesheet.

Record the resolved system. Phase 4 found the values that violate the laws; phase 7 finds the values that violate *this project's* system.

## 7.2 Drift classification

For every value or component in the diff that doesn't match the system, classify the drift. The fix differs by category.

### Category A: Missing token

The value should exist in the system but doesn't. The diff invented a new value the system was missing.

**Example:** Diff uses `color: #6366f1` (an indigo). The system has `--color-primary` and `--color-primary-light` but not `--color-primary-dim`. The new value is reasonable but undeclared.

**Fix:** Add the token to the system, then reference it. Don't keep it inline.

| Pattern | Severity |
|---|---|
| Hard-coded color that's used 2+ times in the diff | P2 |
| Hard-coded spacing that doesn't match an existing scale step | P2 |
| Hard-coded duration that doesn't match the motion system | P3 |

### Category B: One-off implementation

A shared component already exists; the diff reimplemented it. The fix is to delete the new implementation and use the existing one.

**Example:** The diff has a custom `<button class="btn-blue">`. The system has `<Button variant="primary">`. The diff bypassed the system, probably because the author didn't know it existed.

**Fix:** Replace the one-off with the shared component. If the shared component lacks a needed feature, extend it; don't fork it.

| Pattern | Severity |
|---|---|
| Custom button styled with raw CSS when `<Button>` exists | P1 |
| Custom modal when `<Dialog>` exists | P1 |
| Custom dropdown when `<Select>` / `<Menu>` exists | P1 |
| Custom form input when the form library's input exists | P1 |
| Custom icon when the icon library has it | P2 |

### Category C: Conceptual misalignment

The diff's flow, IA, or hierarchy doesn't match neighboring features. Same conceptual weight gets different visual weight.

**Example:** The system uses inline expansion for "show more" patterns. The diff opens a modal for the same kind of action. The styling is fine; the *shape* is wrong.

**Fix:** Rework the flow to match. This is the most expensive drift to fix; catching it at the gate is much cheaper than after launch.

| Pattern | Severity |
|---|---|
| Multi-step flow as modal when the rest of the app uses full-page routes | P1 |
| Save-on-blur in a feature where the rest of the app uses explicit submit | P1 |
| Progressive disclosure pattern that reveals more or less than neighboring features | P1 |
| New noun (`Workspace` vs the rest of the app's `Project`) | P1 |
| Hierarchy inversion (primary action where the system uses tertiary) | P1 |

## 7.3 Distillation

After resolving drift, strip what doesn't earn its place.

### What to strip

| Pattern | Severity |
|---|---|
| Component that adds a wrapper `<div>` with no layout, style, or semantic role | P2 |
| Container that wraps a single child for no reason | P2 |
| Three lines of utility classes that could be one (`.flex.flex-row.items-center.justify-start.gap-0.flex-nowrap`) | P3 |
| Decorative SVG that adds nothing the layout needs | P2 |
| Headline that restates the title above it | P1 |
| Helper text that's longer than the form field it labels | P2 |
| Card on a card on a card | P1 (also covered in phase 4) |
| Three CTAs of identical weight when only one is the primary action | P1 |
| Trust badges that add no trust ("As seen in: We don't say where") | P2 |
| Sections that exist because the template had them, not because the user needs them | P2 |

### What never to strip

- Loading, error, and empty states. They're mandatory.
- Focus indicators.
- Skip links and landmarks.
- ARIA labels.
- Captions and alt text.
- The footer, even on landing pages.

### The distillation question

For every element in the diff, ask: *if this were removed, what would be lost?*

- **Nothing measurable**: strip it.
- **Visual symmetry only**: strip it. Asymmetric is fine.
- **A user task gets harder**: keep it.
- **A user task disappears**: keep it.
- **Brand recognition / atmosphere on a brand surface**: keep it (brand only; product surfaces don't get this exception).

## 7.4 Code quality cleanup

Beyond design drift, the diff must be clean to ship.

| Pattern | Severity |
|---|---|
| `console.log` left in code | P1 |
| Commented-out code blocks | P2 |
| Unused import | P2 |
| Unused variable / parameter | P3 |
| TypeScript `any` (or `// @ts-ignore`) without justification | P1 |
| `// TODO` without an owner or date | P3 |
| Magic number that should be a named constant | P2 |
| Function or component over 200 lines without a clear reason | P2 |
| Repeated code that should be extracted | P2 |
| Dead code branch (unreachable or never-true condition) | P1 |

## Output format

```markdown
## Phase 7 — Streamline

### Design system discovery
- System found at: <path>
- Token sources: <files>
- Component primitives: <count> in <path>

### Drift report
| Category | Count | Severity range |
|---|---|---|
| A. Missing token | <n> | P2-P3 |
| B. One-off implementation | <n> | P1-P2 |
| C. Conceptual misalignment | <n> | P1 |

### Findings
[entries; for each, label the drift category]

### Distillation report
- Elements that could be removed without measurable loss: <count>
- One-off wrappers: <count>
- Restated headlines: <count>

### Code cleanup
| Issue | Count |
|---|---|
| console.log | <n> |
| Commented code | <n> |
| Unused imports | <n> |
| `any` types | <n> |
| Magic numbers | <n> |
```

## Block decision

| Drift category B count | Decision |
|---|---|
| 0 | Pass |
| 1–2 | P2 |
| 3+ | P1. The diff is bypassing the system enough that it's becoming the new pattern. |

| Drift category C count | Decision |
|---|---|
| 0 | Pass |
| 1+ | P1. Misalignment compounds. |

| Code cleanup P0/P1 count | Decision |
|---|---|
| 0 | Pass |
| 1+ | P1 |

## Common mistakes

- **Calling all drift Category A.** Inventing a new token is easier than admitting you didn't use the existing component. Ask: was there a one-off implementation? was the flow shape different? Don't default to "missing token."
- **Stripping by reflex.** A "redundant" element might be carrying the brand voice. On brand surfaces, the bar for stripping is higher than on product surfaces. Run the distillation question, don't just delete.
- **Treating `any` as P3.** Each `any` weakens the type system for everything downstream. The justification bar is high.
- **Letting "TODO with no owner" slide.** TODOs without owners turn into permanent code in three sprints.

# Phase 5 — Accessibility & performance

WCAG AA compliance, Core Web Vitals, and responsive behavior. Synthesized from impeccable's audit dimensions plus the WCAG and CWV standards directly. This phase is largely measurable; subjective judgment lives in phases 3 and 4.

## Inputs

- The diff or named target.
- The project's accessibility test config if present (`axe-playwright`, `pa11y`, `lighthouse-ci`).
- The component library version (some primitives carry guarantees this phase trusts).

## Methodology

Five sub-checks, scored 0–4 individually, combined at the end.

### 5.1 Contrast

WCAG AA requires:
- 4.5:1 for normal text (under 18.66px regular, or under 24px bold).
- 3:1 for large text (≥ 18.66px regular, ≥ 24px bold).
- 3:1 for UI components and graphical objects (focus rings, icon-only buttons, form borders).

**Detection:**

For every text + background pair in the diff, compute the contrast ratio. Flag pairs below the threshold for their size class. Pay extra attention to:

- Text on colored backgrounds (the most common failure).
- Disabled state colors (often accidentally fail when the disabled style was an afterthought).
- Placeholder text inside inputs.
- Hover and active state colors.
- Error / warning / success colored text on tinted backgrounds.

| Issue | Severity |
|---|---|
| Body text contrast < 4.5:1 | **P0** |
| Heading or large text contrast < 3:1 | P1 |
| Focus ring contrast < 3:1 against background | P1 |
| Form border contrast < 3:1 | P1 |
| Disabled state contrast < 3:1 (and the disabled state must be conveyed by more than color anyway) | P2 |
| Placeholder contrast < 4.5:1 | P2 |

**Don't:**
- Don't claim a failure on a `text-gray-400 on bg-gray-900` pair without computing the actual ratio. Tailwind's gray scale is contrast-tuned; many pairs that look weak measure fine.
- Don't flag if the project has documented an explicit AAA target and the pair clears 7:1 in some other state.

### 5.2 Keyboard path

Every interactive element must be reachable and operable by keyboard.

**Detection:**

| Issue | Severity |
|---|---|
| `outline: none` or `outline: 0` without a `:focus-visible` replacement | **P0** |
| `tabindex` greater than 0 (creates a confusing tab order) | P1 |
| Interactive element built on `<div>` or `<span>` without `role`, `tabindex`, and key handlers | P1 |
| `onClick` on a non-button element without keyboard equivalent | P1 |
| Focus trap in a non-modal context | P1 |
| Skip link missing on a long-form layout | P2 |

**Verify:**

- Tab through the affected surface. Every interactive element receives focus. Order is left-to-right, top-to-bottom in reading order.
- Enter or Space activates the focused element.
- Esc closes modals and popovers.
- Arrow keys navigate within composite widgets (menus, tabs, listbox).

If the project uses Radix, shadcn, Headless UI, Ariakit, or a similar primitive library, the primitives carry these guarantees. Verify they're being used as designed (not styled around with a custom replacement that breaks the contract).

### 5.3 ARIA & semantic HTML

**Heading hierarchy:**

| Issue | Severity |
|---|---|
| Skipped heading levels (h1 → h3) | P1 |
| Multiple `<h1>` on a page | P1 |
| Heading used for visual sizing only (`<h2>` because the text needs to be big) | P1 |

**Landmarks:**

| Issue | Severity |
|---|---|
| Missing `<main>` on a page-level component | P1 |
| Missing `<nav>` on top-level navigation | P2 |
| `role="..."` set on an element that already has the equivalent native semantic | P2 (redundant; remove) |

**Form labels:**

| Issue | Severity |
|---|---|
| Input without `<label>` or `aria-label` | **P0** |
| Required field without `required` and `aria-required` | P1 |
| Error message not connected via `aria-describedby` or `aria-errormessage` | P1 |
| Placeholder used as the only label | **P0** |

**Image alt:**

| Issue | Severity |
|---|---|
| `<img>` without `alt` attribute | P1 |
| Decorative image with non-empty alt | P2 (use `alt=""`) |
| Functional image (icon-button) without `aria-label` | P1 |

### 5.4 Performance

The Core Web Vitals targets:

| Metric | Good | Needs improvement | Poor |
|---|---|---|---|
| LCP (Largest Contentful Paint) | ≤ 2.5s | ≤ 4.0s | > 4.0s |
| INP (Interaction to Next Paint) | ≤ 200ms | ≤ 500ms | > 500ms |
| CLS (Cumulative Layout Shift) | ≤ 0.1 | ≤ 0.25 | > 0.25 |

**Detectable in the diff:**

| Pattern | Severity |
|---|---|
| `<img>` without explicit `width` and `height` (CLS risk) | P1 |
| Hero image without `loading="eager"` and `fetchpriority="high"` | P2 |
| Below-the-fold image without `loading="lazy"` | P2 |
| Large dependency added without tree-shaking imports | P1 |
| Reading layout properties (`offsetHeight`, `getBoundingClientRect`) inside a write loop | P1 |
| Unbounded blur or filter (`backdrop-filter: blur(40px)` on a 100% wide element) | P2 |
| Layout-property transition (covered in phase 4) | P1 |
| Animation without `will-change` on a frequently-animated property | P3 |

**Bundle and dependency:**

| Pattern | Severity |
|---|---|
| `import * as Lib from 'lib'` when tree-shakable named imports exist | P2 |
| Adding a dependency that duplicates an existing one (e.g. lodash + ramda + es-toolkit) | P2 |
| Adding a dependency for a single function that's stdlib in the framework | P2 |

**Render performance (React/Vue/Svelte):**

| Pattern | Severity |
|---|---|
| Inline object/array in props on a frequently-rerendering parent (`<X data={[1,2,3]} />`) | P3 |
| Missing `React.memo` / `Vue.memo` / `$derived` on a component re-rendering on every parent update | P3 |
| `useEffect` that runs every render due to a missing dep array | P1 |
| State update inside a render function (not inside an effect) | P1 |

### 5.5 Responsive

The standard breakpoints to test against: 375px, 768px, 1024px, 1440px.

| Pattern | Severity |
|---|---|
| Fixed `width: <px>` on a top-level layout container | P1 |
| Touch target < 44×44px on a touch surface | **P0** if primary action; P1 otherwise |
| Horizontal scroll at any breakpoint | P1 |
| Text smaller than 14px on mobile | P1 |
| Layout that breaks when `text-zoom: 200%` | P2 |
| Missing media queries on a multi-column layout | P1 |
| `100vh` used (breaks on iOS Safari with the bottom bar) | P2 (use `100dvh`) |

## Output format

```markdown
## Phase 5 — Accessibility & performance

| Sub-check | Score | Worst finding |
|---|---|---|
| 5.1 Contrast | <0-4> | <or "—"> |
| 5.2 Keyboard path | <0-4> | |
| 5.3 ARIA & semantic | <0-4> | |
| 5.4 Performance | <0-4> | |
| 5.5 Responsive | <0-4> | |
| **Total** | **<0-20>** | |

### Findings
[detailed entries; each with file:line, severity, evidence, recommendation, confidence]
```

## Block decision

| Total | Decision |
|---|---|
| 0–9 | **P0**. Block. Fundamental a11y or performance failures. |
| 10–13 | **P1**. Block. Significant gaps in WCAG AA compliance. |
| 14–17 | **P2**. Continue, address in next pass. |
| 18–20 | Pass. |

Plus: any individual P0 listed above blocks regardless of the total.

## Verifying with tools

If the project has these configured, run them and incorporate findings:

| Tool | What it adds |
|---|---|
| `axe-playwright` / `axe-core` | Automated rule check; catches ~30–40% of WCAG issues |
| `lighthouse` (CI) | Performance budgets, CWV scores |
| `pa11y` | CLI a11y audit |
| Browser DevTools "Issues" panel | A11y, performance, deprecation warnings |

These tools augment but don't replace the diff-level reading. Many of the highest-severity issues (placeholder-as-label, nested interactive elements, focus trap escape) need code reading.

## Common mistakes

- **Trusting computed contrast in the design tool.** Figma's contrast plugin computes against the visual canvas; the rendered DOM with overlapping translucent layers can measure differently. Verify in the browser.
- **Treating Radix/shadcn as a guarantee.** It guarantees the underlying primitive's behavior. If the component composes a `<Tooltip>` inside a `<Dialog>` inside a `<Popover>`, focus management can still escape. Test the composition.
- **Reporting "missing memoization" as P1.** Most components don't need it. Premature memoization adds cost. Only flag when there's measurable overhead.
- **Ignoring `prefers-reduced-motion` on hover effects.** Hover transforms are motion. Wrap them in the media query.

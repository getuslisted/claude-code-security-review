# Pre-Delivery Checklist

The flat, copy-pasteable version of the pipeline. One markdown checkbox per blocking item. Run through it for any branch before merge or any block before paste.

## Hard gates (any unchecked = no merge)

### Security

- [ ] Diff scanned for SQL injection, command injection, XXE, NoSQL injection, path traversal, template injection.
- [ ] AuthN / AuthZ checked: no bypass logic, no privilege escalation, no missing IDOR guard.
- [ ] No hardcoded API keys, passwords, or tokens in source.
- [ ] No weak cryptographic algorithms or improper key storage.
- [ ] No deserialization RCE (pickle, YAML, eval injection).
- [ ] No XSS via `dangerouslySetInnerHTML`, `v-html`, `[innerHTML]`, `bypassSecurityTrustHtml`, or `{@html ...}` with user content.
- [ ] No PII or secrets in logs.
- [ ] No HIGH-severity finding. No MEDIUM-severity finding with confidence ≥ 0.85.

### Anti-Pattern absolutes

- [ ] No `border-left` or `border-right` greater than 1px as a colored accent.
- [ ] No `background-clip: text` with a gradient (no gradient text).
- [ ] No glassmorphism applied as default decoration.
- [ ] No hero-metric template (big number + small label + supporting stats + gradient).
- [ ] No identical card grids (4+ same-sized cards with same icon + heading + text shape).
- [ ] No modal as the first thought for an interaction; inline or progressive alternatives considered.
- [ ] No pure black (`#000`) or pure white (`#fff`); every neutral tinted toward a brand hue.
- [ ] No bounce or elastic easing; uses ease-out exponential family.
- [ ] No animation on layout properties (`width`, `height`, `padding`, `margin`); only `transform` and `opacity`.
- [ ] No nested cards.
- [ ] No first-order category reflex (observability ≠ dark blue, banking ≠ navy gold, AI tool ≠ purple-pink).
- [ ] No second-order category reflex (anti-cliché-cliché).

### Accessibility

- [ ] Body text contrast ≥ 4.5:1.
- [ ] Large text contrast ≥ 3:1.
- [ ] Focus indicators visible at ≥ 3:1 against adjacent surfaces.
- [ ] Every interactive element reachable via keyboard.
- [ ] Tab order matches visual order.
- [ ] No keyboard traps (modals trap and release focus).
- [ ] No `outline: none` without a replacement focus ring.
- [ ] Touch targets ≥ 44 × 44 px on touch devices.
- [ ] Adjacent interactive elements have ≥ 8 px separation.
- [ ] Every form input has a programmatic label.
- [ ] Required fields marked visually and via `aria-required`.
- [ ] Error messages associated with inputs via `aria-describedby` or `aria-errormessage`.
- [ ] Validation errors persist in the DOM; not pure visual flash.
- [ ] Heading hierarchy monotone (no skipped levels).
- [ ] Landmarks present: `<header>`, `<nav>`, `<main>`, `<footer>`.
- [ ] Decorative images use `alt=""`; meaningful images describe what they convey.
- [ ] Live regions on dynamic content updates.
- [ ] `<button>` for buttons, `<a>` for links; no `<div onClick>`.
- [ ] `prefers-reduced-motion` collapses non-essential animation.
- [ ] No flashing content above 3 Hz.
- [ ] Color is never the only carrier of meaning.
- [ ] Forced-colors mode (high-contrast) does not break layout.

### Design system conformance

- [ ] Every color in the diff comes from a token (no hex literals outside the token file).
- [ ] New colors declared in OKLCH, with chroma reduced toward 0 and 100 lightness.
- [ ] Every spacing value lives on the project's spacing scale.
- [ ] Every radius matches the controlled vocabulary for the component class.
- [ ] Hierarchy contrast ≥ 1.25 between adjacent type steps.
- [ ] Body line length capped 65-75ch.
- [ ] Italic is voice for display, not emphasis inside paragraphs.
- [ ] Body line-height matches project value.
- [ ] Headings use `clamp()` fluid sizing; body uses fixed `rem`.
- [ ] No gray text on a colored background.
- [ ] Surfaces flat at rest; shadows respond to state.
- [ ] Strongest shadow blur uses ≤ 0.15 alpha.
- [ ] Tinted shadows reserved for the deliberate accent-glow moment.
- [ ] Animation duration matches project scale (150ms color/opacity, 300-400ms transforms).
- [ ] Easing from project curve set; no bounce, no elastic.
- [ ] Shared components used; no one-off reimplementations of existing primitives.

## Should-pass (any unchecked → P1, fix before release)

### Performance

- [ ] No layout thrashing (read-then-write-then-read of layout properties in a loop).
- [ ] `filter`, `backdrop-filter`, `box-shadow` paint areas bounded.
- [ ] Off-screen images use `loading="lazy"`.
- [ ] Hero image preloaded.
- [ ] Above-the-fold critical CSS under 14 KB.
- [ ] Web fonts use `font-display: swap` and a preload directive.
- [ ] Images and embeds carry explicit `width` and `height` (or `aspect-ratio` CSS).
- [ ] Layout shift score below 0.1.
- [ ] Critical API calls run in parallel, not waterfalled.
- [ ] Search inputs debounced (200-400ms).
- [ ] Scroll handlers throttled (50-100ms).
- [ ] React: expensive components and selectors memoized.

### Resilience

- [ ] Long names, descriptions, titles render without breaking layout.
- [ ] Single-line ellipsis or multi-line clamp for overflowing text.
- [ ] Flex / grid items have `min-width: 0` to allow shrink.
- [ ] Empty state present for every list, search result, and dataset.
- [ ] Loading state present for every async action.
- [ ] Error state present for every async action.
- [ ] 4xx and 5xx errors have distinct treatments.
- [ ] Validation errors render near input, preserve user input.
- [ ] Error messages specific (not "Error occurred").
- [ ] Long translations fit (30-40% expansion budget).
- [ ] Logical CSS properties (`margin-inline-start`, not `margin-left`).
- [ ] RTL layout reverses correctly.
- [ ] Direction-implying icons flip via `[dir="rtl"]`.
- [ ] Date and number formatting via `Intl.*`.
- [ ] Pluralization handled by i18n library.
- [ ] Double-submit prevented (button disabled while pending).
- [ ] Concurrent requests handled (request id correlation, abort prior).

### Editorial

- [ ] No em dashes (`—`, `&mdash;`, `&#8212;`, `&#x2014;`).
- [ ] No double-hyphen as em-dash substitute.
- [ ] No `delve`, `delves`, `delving`.
- [ ] No `seamless`, `seamlessly`.
- [ ] No `robust`, `robustness`.
- [ ] No `elevate`, `empower`, `underscore`, `pivotal`, `tapestry`.
- [ ] No `load-bearing`, `highest-leverage`, `biggest unlock`.
- [ ] No `data-driven`.
- [ ] No `in today's`, `gone are the days`, `whether you're`, `let's dive in`.
- [ ] No `in summary`, `in conclusion`.
- [ ] No `moreover`, `furthermore`.
- [ ] Triadic everything checked (vary list count).
- [ ] Five-paragraph essay shape avoided.
- [ ] Synthetic balance (equal pros / cons when one is right) avoided.
- [ ] Hollow confidence ("powerful" without numbers) replaced with concrete fact.
- [ ] Interchangeable-copy test passed (swap product name; if nothing breaks, copy is generic).

### Stack-specific

- [ ] React: Server vs Client components classified correctly.
- [ ] React: `next/image`, `next/font`, `next/link` used in Next.js code.
- [ ] Vue: composables prefixed `use*`. Reactivity primitive chosen correctly.
- [ ] Astro: `client:*` only when needed.
- [ ] SwiftUI: `@StateObject` vs `@ObservedObject` correct.
- [ ] React Native: `Pressable` over `TouchableOpacity`. `FlatList` for long lists.
- [ ] Flutter: `const` constructors. `Semantics` widgets. `MediaQuery.textScaleFactor` respected.
- [ ] Tailwind: class strings under 80 chars or extracted.
- [ ] shadcn: components imported from `@/components/ui/*`, theme variables in `:root` and `.dark`.

## Polish (any unchecked → P2 / P3, fix in next pass if time permits)

- [ ] Pixel-perfect alignment to grid.
- [ ] Optical alignment for icons against text.
- [ ] No widows or orphans.
- [ ] Consistent capitalization (Title Case vs Sentence case).
- [ ] Icons from same family or matching style.
- [ ] No debug `console.log` in production paths.
- [ ] No commented-out dead code.
- [ ] No unused imports.
- [ ] No TypeScript `any` or ignored errors.

## Sign-off

- [ ] All hard gates checked.
- [ ] Composite Audit Health Score documented (≥ 14 / 20 to ship).
- [ ] Security verdict: pass.
- [ ] P0 count: 0.
- [ ] P1 count: ≤ 5.
- [ ] Verdict band recorded.
- [ ] Recommended commands listed in priority order.

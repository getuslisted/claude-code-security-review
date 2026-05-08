# Anti-Patterns Catalog

Consolidated catalog from `claude-code-security-review`, `impeccable`, and `ui-ux-pro-max-skill`. Each entry has an ID, category, detection rule, why it fails, and the fix.

The catalog drives Phase 2 (Anti-Pattern Audit) and feeds Phases 3-8 with rules that touch their domain.

## Categories

- `slop` — AI-generation tells.
- `quality` — concrete design or accessibility issues.
- `security` — security anti-patterns (also covered by Phase 1).
- `stack-react`, `stack-vue`, `stack-react-native`, etc. — stack-specific.
- `industry-fintech`, `industry-healthcare`, etc. — industry-specific.

## Severity

- **P0** — blocking. Pipeline fails.
- **P1** — fix before release.
- **P2** — fix in next pass.
- **P3** — polish only.

---

## Cross-register absolute bans (impeccable)

### `qc-001` — Side-stripe border

- **Category**: slop
- **Severity**: P1
- **Detection**: `border-left` or `border-right` greater than 1px as a colored accent on cards, list items, callouts, or alerts.
- **Why it fails**: The single most recognizable AI-dashboard tell. Almost never intentional.
- **Fix**: full border, background tint, leading number, leading icon, or no articulation at all.

### `qc-002` — Gradient text

- **Category**: slop
- **Severity**: P1
- **Detection**: `background-clip: text` combined with a `linear-gradient` or `radial-gradient` background.
- **Why it fails**: Decorative, never meaningful. Marks generic AI output.
- **Fix**: solid color. Emphasis via weight (500-700) or size, not gradient fill.

### `qc-003` — Glassmorphism by default

- **Category**: slop
- **Severity**: P1
- **Detection**: `backdrop-filter: blur(...)` on cards, panels, headers as default decoration (not state-driven).
- **Why it fails**: Used to fake depth without the design rigor that earned it. Dropped frames, accessibility cost.
- **Fix**: flat surface with hairline border, or earned depth (state shadow, scroll position).

### `qc-004` — Hero-metric template

- **Category**: slop
- **Severity**: P1
- **Detection**: hero region contains a large numeric figure (`~10k+ users`, `99.9%`, `$1B+`) followed by a small label, supporting stats, and a gradient accent.
- **Why it fails**: SaaS cliché. Could belong to any product.
- **Fix**: replace with a sentence that names what the product specifically does. The number, if it appears, lives mid-page near the proof, not in the headline.

### `qc-005` — Identical card grid

- **Category**: slop
- **Severity**: P1
- **Detection**: 4 or more sibling cards with identical dimensions, identical structure (icon + heading + text), identical role.
- **Why it fails**: AI default. Vary scale, role, or shape; identical repetition is monotony, not rhythm.
- **Fix**: vary card size (lead, standard, wide), vary content density, or replace with a list when the cards aren't doing card work.

### `qc-006` — Modal as first thought

- **Category**: quality
- **Severity**: P2
- **Detection**: any non-trivial flow that opens in a modal when an inline edit, panel, or route would do.
- **Why it fails**: Modals interrupt. Most flows are calmer inline.
- **Fix**: exhaust inline / progressive alternatives first. Keep the modal only when it's actually a focus-demanding decision.

### `qc-007` — Pure black or pure white

- **Category**: slop
- **Severity**: P2
- **Detection**: `#000`, `#fff`, `rgb(0,0,0)`, `rgb(255,255,255)` in CSS.
- **Why it fails**: Reads as untuned. Tinted neutrals feel considered.
- **Fix**: tint every neutral toward the brand hue at chroma 0.005-0.01.

### `qc-008` — Bounce or elastic easing

- **Category**: slop
- **Severity**: P2
- **Detection**: easing curves that overshoot then settle (e.g., `cubic-bezier(0.68, -0.55, 0.27, 1.55)`, GSAP `back.out`, `elastic.out`).
- **Why it fails**: Real objects decelerate smoothly. Bounce reads as decorative, not designed.
- **Fix**: use `ease-out-quart` (`cubic-bezier(0.165, 0.84, 0.44, 1)`), `ease-out-quint` (`cubic-bezier(0.23, 1, 0.32, 1)`), or `ease-out-expo` (`cubic-bezier(0.16, 1, 0.3, 1)`).

### `qc-009` — Layout-property animation

- **Category**: quality (performance)
- **Severity**: P1
- **Detection**: `transition` or `animation` targeting `width`, `height`, `padding`, `margin`, `top`, `left`, `right`, `bottom`.
- **Why it fails**: Triggers layout passes. Drops frames. Hostile to compositor.
- **Fix**: use `transform` and `opacity`. For dimension change, animate `scale()` or use `will-change: transform`.

### `qc-010` — Nested cards

- **Category**: slop
- **Severity**: P2
- **Detection**: a card-classed element contains another card-classed element.
- **Why it fails**: Reads as Russian-doll architecture; obscures hierarchy.
- **Fix**: flatten. The inner card becomes a list item or section.

---

## Category-reflex tells (impeccable + ui-ux-pro-max)

### `qc-020` — First-order category reflex

- **Category**: slop
- **Severity**: P1
- **Detection**: theme + palette is the obvious one for the category.
  - Observability / DevOps → dark blue + neon accents
  - Healthcare → white + teal
  - Finance / Banking → navy + gold
  - Crypto → neon on black
  - AI tool → purple-pink gradients on dark
  - Wellness / Spa → soft pink + sage
- **Why it fails**: The first training-data reflex. Marks the design as "could be anyone in the category".
- **Fix**: rework the scene sentence and color strategy until the answer isn't obvious from the domain.

### `qc-021` — Second-order category reflex

- **Category**: slop
- **Severity**: P1
- **Detection**: given an anti-reference, the design ran to the next predictable family.
  - "AI tool that's not purple-pink" → editorial-typographic on warm cream
  - "Fintech that's not navy-gold" → terminal-native dark mode
  - "SaaS that's not gradient-on-dark" → brutalist black-and-white
- **Why it fails**: Anti-cliché reflex is its own cliché.
- **Fix**: name the actual aesthetic the design wants, sourced from real referents (a publication, a film, an artist, a building), not from "not the obvious thing".

---

## Design-system drift (impeccable + ui-ux-pro-max)

### `qc-030` — Hard-coded color

- **Category**: quality (theming)
- **Severity**: P1
- **Detection**: hex literal, `rgb()`, `hsl()`, `oklch()` outside the token file.
- **Why it fails**: Drift compounds. Theme switching breaks. Future updates skip the orphan.
- **Fix**: classify as missing-token (add the token), one-off (use the existing token), or conceptual (the value shouldn't exist at all). Patch accordingly.

### `qc-031` — Off-scale spacing

- **Category**: quality (theming)
- **Severity**: P2
- **Detection**: a `gap`, `margin`, or `padding` value that doesn't match the project's spacing scale (e.g., 13px when the scale is 8/16/24/32).
- **Why it fails**: Reads as unconsidered.
- **Fix**: use the nearest scale value, or add a documented exception.

### `qc-032` — Gray on color

- **Category**: quality (theming)
- **Severity**: P1
- **Detection**: gray text (any neutral) layered over a colored (non-neutral) background.
- **Why it fails**: Always looks accidental. Usually fails contrast.
- **Fix**: a tint or shade of the background color. Or transparency over the same color.

### `qc-033` — Flat type scale

- **Category**: quality (typography)
- **Severity**: P2
- **Detection**: adjacent type sizes have ratio < 1.25 (e.g., 14 / 15 / 16 / 17).
- **Why it fails**: No hierarchy; everything reads equally weighted.
- **Fix**: target ≥ 1.25 contrast (e.g., 14 / 16 / 20 / 26).

### `qc-034` — Italic-as-emphasis

- **Category**: quality (typography)
- **Severity**: P3
- **Detection**: italic applied inside body paragraphs as emphasis (`<em>` rendered italic when the display face is also italic).
- **Why it fails**: Dilutes the display voice when the display family uses italic as character.
- **Fix**: emphasize with weight or by swapping to the mono family.

---

## Accessibility (impeccable + ui-ux-pro-max)

### `qc-040` — Contrast under 4.5:1

- **Category**: quality (a11y)
- **Severity**: P1 (P0 if below 3:1)
- **Detection**: computed text contrast against background below 4.5:1 for body, 3:1 for large text.
- **Fix**: deepen the foreground or lighten the background until the ratio passes.

### `qc-041` — Touch target under 44px

- **Category**: quality (a11y)
- **Severity**: P1
- **Detection**: interactive element width OR height below 44px on touch viewports.
- **Fix**: pad to 44px minimum, or add an invisible hit region.

### `qc-042` — `outline: none` without replacement

- **Category**: quality (a11y)
- **Severity**: P0
- **Detection**: `outline: none` or `outline: 0` without a `:focus-visible` style.
- **Fix**: provide a visible focus ring (border, outline, box-shadow) at ≥ 3:1 contrast.

### `qc-043` — `<div onClick>`

- **Category**: quality (a11y)
- **Severity**: P0
- **Detection**: `<div>` or `<span>` with click handler but no role, no tabindex, no keyboard handling.
- **Fix**: use `<button>` for actions, `<a>` for navigation. If a div is unavoidable, add `role="button"`, `tabindex="0"`, and `keydown` for Space and Enter.

### `qc-044` — Missing form label

- **Category**: quality (a11y)
- **Severity**: P0
- **Detection**: `<input>` without `<label for>`, `aria-label`, or `aria-labelledby`.
- **Fix**: pair every input with a programmatic label.

### `qc-045` — Placeholder as label

- **Category**: quality (a11y)
- **Severity**: P1
- **Detection**: input with placeholder but no label.
- **Fix**: add a real label. Placeholder is a hint, not a label.

### `qc-046` — Skipped heading level

- **Category**: quality (a11y)
- **Severity**: P2
- **Detection**: `<h1>` followed by `<h3>` with no `<h2>` between.
- **Fix**: maintain monotone hierarchy.

---

## Performance (impeccable)

### `qc-050` — Layout thrashing

- **Category**: quality (performance)
- **Severity**: P1
- **Detection**: read of `offsetWidth` / `getBoundingClientRect` followed by a write to `width` / `height` / `transform` followed by another read in the same frame.
- **Fix**: batch reads, then writes. Use `requestAnimationFrame`.

### `qc-051` — Unbounded blur or filter

- **Category**: quality (performance)
- **Severity**: P1
- **Detection**: `filter`, `backdrop-filter`, or `box-shadow` with large blur on a full-bleed (100vh, 100vw) element.
- **Fix**: bound the paint area; apply blur to a smaller layer, or replace with a tinted background.

### `qc-052` — Missing image dimensions

- **Category**: quality (performance)
- **Severity**: P1
- **Detection**: `<img>` without `width` and `height` attributes, and no `aspect-ratio` CSS.
- **Fix**: set explicit dimensions. Layout shift score drops to near zero.

### `qc-053` — `loading="lazy"` missing on off-screen image

- **Category**: quality (performance)
- **Severity**: P2
- **Detection**: `<img>` below the fold without `loading="lazy"`.
- **Fix**: add `loading="lazy"`. Reserve eager loading for the hero image.

---

## Resilience (impeccable harden)

### `qc-060` — Fixed-width text container

- **Category**: quality (i18n)
- **Severity**: P1
- **Detection**: text-bearing element with `width: <fixed>` (e.g., `w-24`).
- **Fix**: use intrinsic sizing or `max-width`. Budget 30-40% expansion for translations.

### `qc-061` — Physical CSS properties in i18n-ready code

- **Category**: quality (i18n)
- **Severity**: P2
- **Detection**: `margin-left`, `padding-right`, `border-left` used where `margin-inline-start`, `padding-inline-end`, `border-inline-start` would adapt to RTL.
- **Fix**: switch to logical properties.

### `qc-062` — Missing empty state

- **Category**: quality (resilience)
- **Severity**: P1
- **Detection**: list, search-results, or dataset-bearing component with no rendering for the zero-item case.
- **Fix**: add an empty state with the next action.

### `qc-063` — Generic error message

- **Category**: quality (resilience)
- **Severity**: P2
- **Detection**: error message reads "Error occurred", "Something went wrong", or equivalent.
- **Fix**: name what failed, what's safe, and what to do next.

---

## Editorial (impeccable STYLE.md)

### `qc-070` — Em dash in user-facing copy

- **Category**: slop (editorial)
- **Severity**: P1 (P0 if in marketing hero)
- **Detection**: `—`, `&mdash;`, `&#8212;`, `&#x2014;`, or ` -- ` in JSX text, markdown, or strings exported to UI.
- **Why it fails**: Decision-avoidance. The writer didn't pick a relationship between clauses.
- **Fix**: comma, colon, semicolon, period, or parentheses. Pick the relationship.

### `qc-071` — Banned diction

- **Category**: slop (editorial)
- **Severity**: P1
- **Detection**: any of: `delve`, `seamless`, `robust`, `elevate`, `empower`, `underscore`, `pivotal`, `tapestry`, `load-bearing`, `highest-leverage`, `biggest unlock`, `data-driven`, `reflex defaults`, `collapses into monoculture` in user-facing copy.
- **Why it fails**: AI-flavor diction.
- **Fix**: see [`COPY-DENYLIST.md`](./COPY-DENYLIST.md) for replacements per term.

### `qc-072` — Throat-clearing opener

- **Category**: slop (editorial)
- **Severity**: P2
- **Detection**: paragraph or section starts with `In today's`, `Gone are the days`, `Whether you're`, `Let's dive in`.
- **Fix**: start at the actual point.

### `qc-073` — Triadic auto-pilot

- **Category**: slop (editorial)
- **Severity**: P3
- **Detection**: lists of three when content fits two or four. Adjective triplets ("fast, simple, and powerful").
- **Fix**: vary the count.

---

## Stack-specific

### `qc-080` — `dangerouslySetInnerHTML` with user content (React)

- **Category**: security + stack-react
- **Severity**: P0
- **Detection**: `dangerouslySetInnerHTML={{ __html: <user-content> }}`.
- **Fix**: don't. Use a sanitization layer (DOMPurify) before injection, or render structured content via React.

### `qc-081` — `v-html` with user content (Vue)

- **Category**: security + stack-vue
- **Severity**: P0
- **Detection**: `v-html="<user-content>"`.
- **Fix**: same as above.

### `qc-082` — Server Component using `useState` (Next.js)

- **Category**: stack-react
- **Severity**: P0
- **Detection**: file without `"use client"` directive contains `useState`, `useEffect`, `useRef`, or other client-only hooks.
- **Fix**: add `"use client"` at the top, or move the stateful logic to a child Client Component.

### `qc-083` — `ScrollView` of mapped long list (React Native)

- **Category**: stack-react-native
- **Severity**: P1
- **Detection**: `<ScrollView>` containing `.map()` over a list expected to grow beyond ~20 items.
- **Fix**: `FlatList` or `SectionList`; both virtualize.

### `qc-084` — `TouchableOpacity` in new code (React Native)

- **Category**: stack-react-native
- **Severity**: P3
- **Detection**: `<TouchableOpacity>` in new components.
- **Fix**: `<Pressable>` is the modern primitive with finer control.

---

## Industry-specific (ui-ux-pro-max reasoning rules)

### `qc-100` — AI-purple gradient on banking surface

- **Category**: industry-fintech
- **Severity**: P1
- **Detection**: industry inferred as banking, fintech, insurance, or invoicing AND a purple-pink gradient appears in the diff.
- **Why it fails**: Banking should not look like an AI startup.
- **Fix**: use a navy / charcoal / accent palette appropriate to financial trust.

### `qc-101` — Brutalism on healthcare surface

- **Category**: industry-healthcare
- **Severity**: P1
- **Detection**: industry inferred as healthcare, dental, mental health, or pharmacy AND brutalist styling (sharp uppercase, raw black-on-white, deliberately rough type) is used in primary surfaces.
- **Why it fails**: Healthcare needs reassurance, not aggression.
- **Fix**: soft, calm, reassuring palette and type. White space.

### `qc-102` — Harsh neon on wellness surface

- **Category**: industry-wellness
- **Severity**: P1
- **Detection**: industry inferred as wellness, spa, beauty, or meditation AND saturated neon accents (>0.25 chroma at typical lightness) are used.
- **Fix**: soft pastels, organic shapes, gentle gradients (if any).

### `qc-103` — Dense data table on consumer mobile

- **Category**: industry-consumer
- **Severity**: P2
- **Detection**: industry inferred as consumer mobile (lifestyle, beauty, food delivery) AND a dense multi-column table is shown on a 375px viewport.
- **Fix**: card or list-row layout that respects the touch surface.

---

## How to add a new anti-pattern

When a new tell or drift gets named in any of the source repos, add an entry here:

1. Reserve the next available ID in the appropriate range:
   - 001-019: cross-register absolute bans (impeccable)
   - 020-029: category-reflex (impeccable + ui-ux-pro-max)
   - 030-039: design-system drift
   - 040-049: accessibility
   - 050-059: performance
   - 060-069: resilience
   - 070-079: editorial
   - 080-099: stack-specific
   - 100-149: industry-specific
2. Document Category, Severity, Detection rule, Why it fails, Fix.
3. Reference the source repo and rule (e.g., "impeccable `audit.md` §A11y").
4. Bump the manifest version in `PIPELINE.md` and `README.md`.

# Anti-patterns catalog

Consolidated catalog of every anti-pattern the gate detects. Pulled from impeccable's 27 deterministic + 12 LLM rules, ui-ux-pro-max-skill's industry exclusions, and the security review's hard-exclusion list (inverted — what *not* to flag).

Each entry has:
- **ID**: stable identifier for cross-referencing
- **Source**: which upstream skill it came from
- **Detect**: machine-readable signal in the diff
- **Severity**: gate severity (P0–P3)
- **Why**: what's wrong
- **Fix**: the canonical replacement

## Visual / design tells

### `side-stripe-border` — P1
- **Source**: impeccable
- **Detect**: `border-(left|right|inline-start|inline-end): \d+px` where `\d+ ≥ 2`, with a non-neutral color
- **Why**: The single most recognizable AI-dashboard tell. Applied as a "category accent" on cards, alerts, callouts, list items.
- **Fix**: Full 1px border + tinted background; or leading icon + heading; or top/bottom 1px accent.

### `gradient-text` — P1
- **Source**: impeccable
- **Detect**: `background-clip: text` (or `-webkit-background-clip: text`) with a `linear-gradient` or `radial-gradient` background and `color: transparent`
- **Why**: Decorative, never meaningful. Decision-avoidance: writer didn't pick a single color.
- **Fix**: A single solid color. Emphasis via weight or size.

### `glassmorphism-default` — P1
- **Source**: impeccable
- **Detect**: `backdrop-filter: blur(\d+px)` combined with `background: rgba(...,< 0.3)` on a card or panel, appearing on more than one surface in the diff or without specific justification
- **Why**: Used decoratively, not purposefully. Reads as 2020-era SaaS.
- **Fix**: Solid surface + hairline border. If depth needed, low-alpha shadow.

### `hero-metric-template` — P1
- **Source**: impeccable
- **Detect**: A grid or flex container with 3–4 cells, each containing a large number element (≥ 48px) followed by a small label element (≤ 16px), often with colored accents
- **Why**: SaaS cliché. Dashboards aren't pages of stats; they're pages of *answers*.
- **Fix**: Embed metrics in sentences. Use one hero KPI with context. Use a chart that shows the metric in motion.

### `identical-card-grid` — P1
- **Source**: impeccable
- **Detect**: A `grid` or `flex` container with 3+ direct children matching the same structure: icon element + heading element + paragraph element, identical sizes
- **Why**: The "feature grid" pattern. Reads as template-filled.
- **Fix**: Vary card sizes (bento). Replace some with sentences in flow. Use a list with leading numbers. Pull strongest into a hero.

### `modal-as-first-thought` — P1
- **Source**: impeccable
- **Detect**: New modal added in the diff
- **Conditional finding**: Only flag if an inline alternative exists (accordion, side panel, separate route, popover-with-confirm)
- **Why**: Modals are usually laziness. They trap focus, they obscure the surrounding context, they require dismiss management.
- **Fix**: The named alternative. Modals reserved for genuinely modal moments (destructive confirm, blocking auth, error preventing continuation).

### `nested-cards` — P1
- **Source**: impeccable
- **Detect**: A card-styled element (background + radius + padding + border or shadow) inside another card-styled element
- **Why**: Visual noise. The hierarchy isn't doing work; the redundancy is.
- **Fix**: Flatten. The outer card is a section; the inner is a row. They don't both need card styling.

### `icon-tile-stack` — P2
- **Source**: impeccable
- **Detect**: Rounded-square colored tile (typically 32–48px, `border-radius: 8–12px`, accent background) above an `<h2>` or `<h3>` heading. Pattern repeats across the page.
- **Why**: Specific AI-marketing-page tell. The icon adds nothing; the heading carries the meaning.
- **Fix**: Skip the tile. Or use the icon at a smaller size, inline with the heading.

### `flat-type-hierarchy` — P2
- **Source**: impeccable
- **Detect**: Heading scale ratios under 1.2 between adjacent levels (e.g. h1 = 32px, h2 = 28px, h3 = 24px)
- **Why**: Weak hierarchy. The eye can't distinguish levels at a glance.
- **Fix**: Ratios ≥ 1.25, ideally 1.333 or 1.5 between display and body.

### `inter-only` — P2
- **Source**: impeccable
- **Detect**: Inter (or system-ui-only) for both display and body type, no second voice
- **Why**: Inter is the most-trained default. Shipping it everywhere makes the surface look like every other AI-generated UI.
- **Fix**: Pair Inter with a display face that has voice (transitional serif, geometric grotesque, mono variable).

### `bouncy-easing` — P2
- **Source**: impeccable
- **Detect**: Cubic-bezier with overshoot, `ease-in-out` with overshoot, named `cubic-bezier(.68,-.55,.27,1.55)`
- **Why**: Real objects decelerate smoothly. Bounce/elastic feels dated.
- **Fix**: Expo-out (`cubic-bezier(0.16, 1, 0.3, 1)`) or quart/quint variants.

### `ai-purple-blue-gradient` — P2
- **Source**: ui-ux-pro-max-skill (industry-anti-pattern)
- **Detect**: Linear gradient from saturated purple (`#7c3aed`, oklch hue ~280–300) to saturated blue (`#3b82f6`, oklch hue ~240–260), particularly on AI products, hero sections, or CTAs
- **Why**: The single most-trained AI-product reflex.
- **Fix**: A single committed color, or a different palette entirely. The category-reflex check (phase 3) catches this.

### `pure-black-white` — P1
- **Source**: impeccable
- **Detect**: `#000`, `#fff`, `rgb(0,0,0)`, `rgb(255,255,255)`, `black`, `white` literal values
- **Why**: Both read as cold and untinted. Real ink isn't pure black; real paper isn't pure white.
- **Fix**: Tinted neutrals — chroma 0.005–0.01 toward the brand hue. `oklch(15% 0.005 270)` instead of `#000`.

### `gray-on-color` — P1
- **Source**: impeccable
- **Detect**: A neutral gray text color on a colored (chroma > 0.05) background
- **Why**: Reads as washed-out, broken contrast tunnel.
- **Fix**: Use a darker tone of the bg color, or a transparent overlay.

### `wave-divider` — P3
- **Source**: impeccable (LLM rule)
- **Detect**: SVG between sections with a sinusoidal or curved path
- **Why**: 2018 marketing-page cliché.
- **Fix**: A 1px hairline rule, or no divider at all.

### `universal-radius-12` — P3
- **Source**: impeccable (LLM rule)
- **Detect**: Every interactive surface uses `border-radius: 12px` (or any single value across all elements)
- **Why**: Radius should be a controlled vocabulary, not a single value.
- **Fix**: Establish a radius scale, apply per element weight.

### `every-section-divided-with-rule-line` — P3
- **Source**: impeccable (LLM rule)
- **Detect**: A `<hr>`-style rule between every section
- **Why**: Whitespace already separates sections.
- **Fix**: Trust whitespace.

## Layout / structure tells

### `card-everywhere` — P2
- **Source**: impeccable
- **Detect**: 80%+ of content blocks are wrapped in a card-styled element
- **Why**: Cards are the lazy answer.
- **Fix**: Use cards only when content genuinely is a discrete object. Let prose flow.

### `centered-narrow-everything` — P3
- **Source**: impeccable (LLM rule)
- **Detect**: Every section uses `max-width: 768–960px` and `margin: 0 auto`
- **Why**: Reads as documentation site, not a product or brand site.
- **Fix**: Vary the layout per section.

### `single-spacing-token` — P2
- **Source**: impeccable
- **Detect**: One spacing value used for nearly every gap on the page
- **Why**: Monotonous rhythm.
- **Fix**: A spacing scale (8/16/24/32/48/80/120). Use them differently per visual weight.

### `wrapped-single-child` — P3
- **Source**: impeccable (LLM rule)
- **Detect**: A `<div>` or `<section>` with one child, no styling beyond layout
- **Why**: Adds DOM nodes for nothing.
- **Fix**: Hoist the child or merge the wrapper's role into it.

## Motion / interaction tells

### `layout-property-animation` — P1
- **Source**: impeccable
- **Detect**: `transition-property` or `@keyframes` targeting `width`, `height`, `padding`, `margin`, `top`, `left`, `right`, `bottom`
- **Why**: Triggers layout on every frame.
- **Fix**: `transform` and `opacity` instead.

### `motion-without-reduced-motion-fallback` — P1
- **Source**: impeccable
- **Detect**: Animation or transition without a `@media (prefers-reduced-motion: reduce)` rule
- **Why**: A11y baseline.
- **Fix**: Wrap motion in the media query.

### `hover-only-affordance` — P1
- **Source**: impeccable
- **Detect**: An interactive surface whose interactive nature is signaled only by hover
- **Why**: Touch users don't hover. Keyboard users don't hover.
- **Fix**: Interactive elements look interactive at rest.

## Copy / content tells

### `em-dash` — P1
- **Source**: impeccable STYLE.md
- **Detect**: `—`, `&mdash;`, `&#8212;`, `&#x2014;` in user-facing copy
- **Why**: Decision-avoidance punctuation.
- **Fix**: Comma, colon, semicolon, period, parentheses.

### `double-hyphen-em-dash-substitute` — P1
- **Source**: impeccable STYLE.md
- **Detect**: ` -- ` in user-facing copy
- **Why**: Worse than the em dash.
- **Fix**: Real punctuation.

### `banned-diction` — P1
- **Source**: impeccable STYLE.md
- **Detect**: `load-bearing`, `highest-leverage`, `biggest unlock`, `seamless`, `seamlessly`, `robust`, `robustness`, `delve`, `delves`, `delving`, `elevate`, `empower`, `underscore`, `pivotal`, `tapestry`, `data-driven`, `reflex defaults`, `collapses into monoculture`, `in today's`, `gone are the days`, `whether you're`, `let's dive in`, `in summary`, `in conclusion`, `moreover`, `furthermore`
- **Why**: Each is in the upstream STYLE.md denylist with a rationale.
- **Fix**: See [STYLE.md in the impeccable repo](https://github.com/getuslisted/impeccable/blob/main/STYLE.md) for per-term replacements.

### `restated-heading` — P2
- **Source**: impeccable STYLE.md
- **Detect**: First sentence of body copy paraphrases the heading directly above it
- **Why**: The heading already said it.
- **Fix**: Open with the strongest specific claim.

### `triadic-everything` — P3
- **Source**: impeccable STYLE.md
- **Detect**: Lists of exactly three items, adjective groups of three
- **Why**: Triads are the AI default rhythm.
- **Fix**: Vary count.

### `placeholder-as-label` — P0
- **Source**: a11y standards
- **Detect**: `<input>` with `placeholder` and no `<label>` or `aria-label`
- **Why**: Placeholder disappears on focus.
- **Fix**: Visible label above the input.

### `error-message-too-generic` — P1
- **Source**: impeccable harden
- **Detect**: User-facing error text matching `/^(error|something went wrong|please try again|invalid input)\.?$/i`
- **Why**: User can't act on it.
- **Fix**: Specific failure + recovery path.

## Accessibility tells

### `outline-none-no-replacement` — P0
- **Source**: WCAG
- **Detect**: `outline: none` or `outline: 0` without a `:focus-visible` style providing equivalent indication
- **Why**: WCAG 2.4.7 failure.
- **Fix**: `:focus-visible { outline: 2px solid var(--color-focus); outline-offset: 2px; }`.

### `tabindex-positive` — P1
- **Source**: WCAG
- **Detect**: `tabindex="1"`, `tabindex="2"`, etc.
- **Why**: Creates a confusing tab order.
- **Fix**: Use the natural DOM order.

### `div-button` — P1
- **Source**: WCAG
- **Detect**: `<div>` or `<span>` with `onClick` and no `role="button"`, `tabindex`, or key handlers
- **Why**: Not focusable, not keyboard-operable, not announced.
- **Fix**: Use `<button>`.

### `image-no-alt` — P1
- **Source**: WCAG
- **Detect**: `<img>` without `alt` attribute
- **Why**: WCAG 1.1.1.
- **Fix**: Meaningful alt for content; `alt=""` for decorative.

### `touch-target-too-small` — P0 (primary action) / P1 (secondary)
- **Source**: WCAG 2.5.5, Apple HIG, Material guidelines
- **Detect**: Interactive element with rendered dimensions < 44×44px on touch surfaces
- **Why**: Mis-tap rate climbs sharply below 44px.
- **Fix**: Minimum 44×44px hit area.

### `low-contrast-body` — P0
- **Source**: WCAG 1.4.3
- **Detect**: Body text contrast < 4.5:1 against its background
- **Why**: WCAG AA hard requirement.
- **Fix**: Increase contrast.

### `low-contrast-large` — P1
- **Source**: WCAG 1.4.3
- **Detect**: Large text (≥ 18.66px regular or ≥ 24px bold) contrast < 3:1
- **Why**: WCAG AA for large text.
- **Fix**: Increase contrast.

### `heading-hierarchy-broken` — P1
- **Source**: WCAG 1.3.1
- **Detect**: Heading levels skipped; multiple h1; or a heading used purely for visual sizing
- **Why**: Screen readers navigate by heading level.
- **Fix**: Sequential levels. Use CSS for sizing, semantics for hierarchy.

## Performance tells

### `image-no-dimensions` — P1
- **Source**: Web Vitals (CLS)
- **Detect**: `<img>` without `width` and `height` attributes
- **Why**: Causes CLS as the image loads.
- **Fix**: Set explicit dimensions, or `aspect-ratio: w / h` on the wrapper.

### `barrel-import` — P2
- **Source**: bundle-size best practice
- **Detect**: `import * as Lib from 'lib'` or `import { ... } from 'large-lib'` where the lib doesn't tree-shake
- **Why**: Pulls the full library into the bundle.
- **Fix**: Direct imports.

### `layout-thrash` — P1
- **Source**: rendering best practice
- **Detect**: Reading layout properties inside a write loop
- **Why**: Forces synchronous layout on every iteration.
- **Fix**: Read first, then write all changes.

### `unbounded-blur` — P2
- **Source**: rendering best practice
- **Detect**: `backdrop-filter: blur(40px)` or higher on a 100vw or 100vh element
- **Why**: Blur is a per-pixel operation.
- **Fix**: Smaller blur radius, smaller surface, or skip the effect.

## What never to flag

These come up often but the gate doesn't report them. Each is on a hard-exclusion list.

### From the security review

- DoS / resource exhaustion of any kind
- Outdated dependencies
- Memory safety in memory-safe languages
- Log spoofing
- Test-file vulnerabilities
- Markdown / docs vulnerabilities
- Lack of audit logs
- Lack of rate limiting
- React/Angular XSS in standard usage
- Path-only SSRF
- Regex injection or ReDoS

### From the design skills

- Subjective taste differences ("I'd use a different shade")
- Industry-standard patterns the user explicitly chose
- Brand guidelines that don't match the gate's defaults (the project's brand wins)
- Performance issues on devices outside the project's stated support matrix

## Source attribution

The catalog draws from:

- **impeccable** ([cli/engine/detect-antipatterns.mjs](https://github.com/getuslisted/impeccable/blob/main/cli/engine/detect-antipatterns.mjs)) — 27 deterministic rules, the AI slop test, the absolute bans
- **claude-code-security-review** ([prompts.py](https://github.com/getuslisted/claude-code-security-review/blob/main/claudecode/prompts.py), [hard-exclusion list](https://github.com/getuslisted/claude-code-security-review/blob/main/.claude/commands/security-review.md)) — what counts as a real security finding
- **ui-ux-pro-max-skill** — industry anti-patterns (per-product-type exclusions in the 161 reasoning rules)
- **WCAG 2.2** — accessibility baselines
- **Core Web Vitals** — performance baselines

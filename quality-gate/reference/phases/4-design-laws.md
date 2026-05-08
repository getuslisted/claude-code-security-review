# Phase 4 — Design laws

The shared design laws and absolute bans. Synthesized from impeccable's SKILL.md shared-laws section and the 27 deterministic anti-pattern rules. Most findings here are pattern-matchable in the diff itself; the rest need a render to verify.

## The shared laws

Every UI surface must satisfy these. They apply to brand and product equally.

### Color

**Laws:**
- Use OKLCH. Reduce chroma as lightness approaches 0 or 100; high chroma at extremes looks garish.
- Never use `#000` or `#fff`. Tint every neutral toward the brand hue (chroma 0.005–0.01 is enough).
- Pick a color strategy before picking colors:
  - **Restrained**: tinted neutrals + one accent ≤ 10%. Product default.
  - **Committed**: one saturated color carries 30–60% of the surface. Brand default.
  - **Full palette**: 3–4 named roles, each used deliberately. Brand campaigns; product data viz.
  - **Drenched**: the surface IS the color. Brand heroes only.

**Detectable in diff:**

| Pattern | Severity | Example |
|---|---|---|
| `#000` or `rgb(0, 0, 0)` | P1 | `color: #000` → `color: oklch(15% 0.005 270)` |
| `#fff` or `rgb(255, 255, 255)` | P1 | `background: #fff` → `background: oklch(98% 0.003 270)` |
| `color: gray` on a colored background | P1 | "Don't put gray on color." Use a tone of the bg color or transparency. |
| Multiple accents in restrained register | P1 | One accent only. Use weight/scale for second emphasis. |
| `linear-gradient` with `background-clip: text` | **P0** | Absolute ban. Gradient text. |

### Theme

**Law:** Dark vs. light is never a default. Write one sentence of physical scene before deciding.

If you can swap the sentence's category and the answer doesn't change ("observability dashboard" vs "personal finance dashboard" → dark either way), the sentence isn't concrete enough. The category-reflex check from phase 3 catches this.

| Pattern | Severity |
|---|---|
| Dark mode chosen with no scene sentence | P1 |
| Dark mode for "calmness" or "focus" without a specific user/environment | P1 |
| Light mode chosen "to be safe" on a brand surface | P2 |

### Typography

**Laws:**
- Cap body line length at 65–75ch.
- Hierarchy through scale + weight contrast. Ratio between steps ≥ 1.25.
- Avoid flat scales; if h1, h2, h3 are within 1.1× of each other, the hierarchy isn't doing work.

| Pattern | Severity |
|---|---|
| Body text without `max-width` (and not in a constrained grid cell) | P1 |
| Heading scale ratio < 1.2 between adjacent levels | P1 |
| Same font for display and body without intentional voice contrast | P2 |
| Inter for everything (the most-trained AI tell) | P2 |

### Layout

**Laws:**
- Vary spacing for rhythm. Same padding everywhere is monotony.
- Cards are the lazy answer. Use them only when they're truly the best affordance.
- **Nested cards are always wrong.**
- Don't wrap everything in a container. Most things don't need one.

| Pattern | Severity |
|---|---|
| Card inside a card | P1 |
| Identical card grid (3+ cards same size, icon + heading + text pattern) | P1 |
| Single spacing token used for every gap on the page | P2 |
| Container wrapping a single child for no layout reason | P3 |

### Motion

**Laws:**
- Don't animate CSS layout properties (`width`, `height`, `padding`, `margin`, `top`, `left`, `right`, `bottom`).
- Ease out with exponential curves: `ease-out-quart`, `ease-out-quint`, `ease-out-expo`. No bounce, no elastic.

| Pattern | Severity |
|---|---|
| `transition-property: width` / `height` / `padding` / `margin` / `top` / `left` | P1 |
| `cubic-bezier(.68,-.55,.27,1.55)` (bounce/back) | P2 |
| Animation without `prefers-reduced-motion` fallback | P1 |
| Transition duration > 500ms on a state change | P2 |
| Transition duration < 100ms on a meaningful state change | P3 |

### Copy

**Laws:**
- Every word earns its place. No restated headings. No intros that repeat the title.
- **No em dashes.** Use commas, colons, semicolons, periods, parentheses. Also not `--`.

| Pattern | Severity |
|---|---|
| Em dash (`—`, `&mdash;`, `&#8212;`, `&#x2014;`) in user-facing copy | P1 |
| `--` (double hyphen as em-dash substitute) in user-facing copy | P1 |
| Heading restated in the first sentence below it | P2 |
| Banned diction (see [anti-patterns-catalog.md](../anti-patterns-catalog.md)) | P1 |

## The six absolute bans

Every one of these is a P0 or P1 finding by itself. Match-and-refuse: if you're about to write any of these, rewrite the element with different structure.

### Ban 1 — Side-stripe borders (P1)

`border-left` or `border-right` greater than 1px as a colored accent on cards, list items, callouts, or alerts. Never intentional. The single most recognizable AI-dashboard tell.

**Detect:**
```css
/* Hits the rule */
border-left: 4px solid var(--color-warning);
border-left: 3px solid #f59e0b;
border-inline-start: 4px solid red;
```

**Fix:**
- Full 1px border + tinted background.
- Leading icon + heading.
- Accent strip top or bottom (`border-top: 1px`), not side.

### Ban 2 — Gradient text (P1)

`background-clip: text` combined with a gradient background. Decorative, never meaningful.

**Detect:**
```css
background: linear-gradient(...);
background-clip: text;
-webkit-background-clip: text;
color: transparent;
```

**Fix:**
- A single solid color.
- Emphasis via weight or size.

### Ban 3 — Glassmorphism as default (P1)

Blurs and glass cards used decoratively. Rare and purposeful, or nothing.

**Detect:**
```css
backdrop-filter: blur(20px);
background: rgba(255, 255, 255, 0.1);
border: 1px solid rgba(255, 255, 255, 0.2);
```

If this pattern appears on more than one surface in the diff, or appears at all without a specific reason in PRODUCT.md, it's a finding.

**Fix:**
- Solid surface with a hairline border.
- If you really need depth, a tinted background + low-alpha shadow.

### Ban 4 — The hero-metric template (P1)

Big number, small label, supporting stats, gradient accent. SaaS cliché.

**Detect:** A grid of 3–4 cells, each with a large number (60–80px) and a small label, often with a colored accent. Look for `.text-6xl + .text-sm` patterns or equivalent.

**Fix:**
- Embed the metric in a sentence.
- Use a single hero KPI with context, not a metric grid.
- Use a chart that shows the metric in motion.

### Ban 5 — Identical card grids (P1)

Same-sized cards with icon + heading + text, repeated endlessly.

**Detect:** A `grid` or `flex` container with 3–6+ direct children that all match the structure: icon element + heading element + paragraph element, all the same size.

**Fix:**
- Vary the card sizes (bento grid pattern).
- Replace some cards with single sentences in the flow.
- Use a list with leading numbers/letters instead.
- Pull the strongest into a hero, fold the rest into prose.

### Ban 6 — Modal as first thought (P1)

Modals are usually laziness. Exhaust inline / progressive alternatives first.

**Detect:** A new modal in the diff. The finding is conditional on the alternatives:

- Could this be inline expansion (accordion, expander)?
- Could this be a side panel that doesn't trap focus?
- Could this be a separate route?
- Could the action be done in place (popover with confirm)?

If yes to any, the modal is a finding.

**Fix:** Rebuild as the inline alternative. Modals are reserved for genuinely modal moments (destructive confirm, blocking auth, error that prevents continuing).

## The AI slop test

If someone could look at this interface and say "AI made that" without doubt, it's failed. Run at two altitudes (described in phase 3, repeated here for completeness because phase 4 is where it blocks):

**First-order**: theme + palette guessable from category alone. Block.

**Second-order**: aesthetic family guessable from category-plus-anti-reference. Block.

The bans above are concrete failure modes the test catches. The test catches more, including:

- Inter as the only font.
- Purple-to-blue gradients on AI products.
- The "rounded square icon tile above every heading" stack.
- Bouncy easing on serious products.
- The wave-shape SVG divider between sections.
- Universal `border-radius: 12px` on every surface.
- The "stat row across the top" layout.

Each of these is in the [anti-patterns-catalog.md](../anti-patterns-catalog.md) with a detection rule.

## Output format

```markdown
## Phase 4 — Design laws

### Shared-law violations
| Law | Status | Evidence |
|---|---|---|
| OKLCH only | <pass|fail> | <file:line> |
| No #000 / #fff | <pass|fail> | <file:line> |
| Color strategy declared | <pass|fail> | <which strategy> |
| Body line length 65–75ch | <pass|fail> | |
| Type scale ratio ≥ 1.25 | <pass|fail> | |
| No nested cards | <pass|fail> | |
| No layout-property animation | <pass|fail> | |
| `prefers-reduced-motion` honored | <pass|fail> | |
| No em dashes | <pass|fail> | |

### Absolute bans
| Ban | Hit | Severity |
|---|---|---|
| 1. Side-stripe border | <yes|no> | P1 |
| 2. Gradient text | <yes|no> | P1 |
| 3. Glassmorphism default | <yes|no> | P1 |
| 4. Hero-metric template | <yes|no> | P1 |
| 5. Identical card grids | <yes|no> | P1 |
| 6. Modal as first thought | <yes|no> | P1 |

### AI slop test
- First-order: <pass | fail (which reflex)>
- Second-order: <pass | fail (which reflex)>

### Findings
[detailed finding entries with file:line, evidence, fix]
```

## Block decision

- Any absolute ban hit → **P1**.
- Either AI slop altitude failed → **P1**.
- 3+ shared-law failures → **P1**.
- 1–2 shared-law failures → **P2**.
- All pass → score 4/4.

## Common mistakes

- **Treating "common" as "acceptable."** A side-stripe border is in 80% of admin-dashboard templates. That doesn't make it acceptable. The bans are bans.
- **Using `#fff` for `transparent`.** Even when the visual result is the same, the token discipline matters. Use the project's neutral-100 token.
- **Animating `padding` for a "smooth expand."** It's a layout property. Use `transform: scaleY()` with `transform-origin: top`, or animate `max-height` if you must, but document why.
- **Reaching for a modal because the design didn't think about progressive disclosure.** Most dialogs in product UIs were the wrong choice. Resist.

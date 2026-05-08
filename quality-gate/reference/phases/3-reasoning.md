# Phase 3 — Industry reasoning

Match the surface to its industry context. Synthesized from ui-ux-pro-max-skill's reasoning engine: 161 product-type rules, 67 UI styles, 161 color palettes, 57 font pairings, and the anti-patterns specific to each industry.

## Why this phase exists

A button that's accessible, fast, and visually clean can still be wrong for the surface. A neon cyber-aesthetic banking dashboard fails before any pixel-level check runs. This phase catches category-level mismatches the other phases miss because they don't read industry context.

## Inputs

- `register` from preflight (brand or product).
- The product type. Read from PRODUCT.md's "Product Purpose" or infer from the codebase (route names, copy, schema).
- The visual evidence: colors, fonts, layout pattern, key effects in the diff.

## Methodology

### 3.1 Identify the product type

Map the project to one of these categories. If the project spans more than one, pick the one in scope for this diff.

| Bucket | Examples |
|---|---|
| Tech & SaaS | SaaS, micro-SaaS, B2B service, developer tool/IDE, AI/chatbot platform, cybersecurity platform |
| Finance | Fintech, crypto, banking, insurance, personal finance, invoice/billing |
| Healthcare | Medical clinic, pharmacy, dental, veterinary, mental health, medication tracking |
| E-commerce | General, luxury, marketplace (P2P), subscription box, food delivery |
| Services | Beauty/spa, restaurant, hotel, legal, home services, booking/appointment |
| Creative | Portfolio, agency, photography, gaming, music streaming, photo/video editor |
| Lifestyle | Habit tracker, recipe, meditation, weather, diary, mood tracker |
| Emerging tech | Web3/NFT, spatial computing, quantum computing, autonomous vehicles |

### 3.2 Score the four reasoning dimensions

For each dimension, score 0–4. The score combines into the phase verdict.

#### Pattern fit (0–4)

Does the page structure match the conversion logic for the product type?

- **Lead-gen / sales pages** (services, healthcare clinics, legal): hero → trust signals → service detail → testimonials → CTA → contact.
- **SaaS landing**: hero → feature showcase → social proof → integration grid → pricing → CTA.
- **E-commerce category page**: filters → grid → load-more → quick-add → recently-viewed.
- **Dashboard**: KPI strip → primary chart → secondary panels → drill-down table.
- **Portfolio**: above-the-fold work → about → contact. No filler.

Score 0 if the page is structured for a different product category (e.g. a SaaS hero-metric layout on a wellness landing). Score 4 if every section earns its place for the user's task.

#### Color mood fit (0–4)

Does the palette match the industry's emotional register?

| Industry | Mood appropriate | Mood inappropriate |
|---|---|---|
| Banking, insurance, legal | Navy, deep green, slate, gold accents, restrained | Neon, aggressive gradients, pastels |
| Wellness, beauty, spa | Warm neutrals, muted earth tones, soft pinks, sage | Sharp neon, harsh primary colors |
| Gaming, crypto, music | Saturated, high-contrast, dark mode acceptable | Pastel, muted, "calm" |
| Healthcare | Calming blues, greens, white space, restrained | Aggressive reds, harsh contrast |
| Children's apps, education | Warm primaries, friendly accents, playful | Dark mode, muted neutrals |
| Editorial, luxury | One decisive accent, lots of paper white, OKLCH-tuned | Multi-color, gradient-heavy |

Critical: this is not a license to converge on the category-default palette. Phase 4's AI slop test catches that. The goal here is **not inappropriate**, not **most expected**.

Score 0 if the palette fights the product (neon banking, dark-mode children's app). Score 4 if the palette serves the product without being the obvious first reach.

#### Typography mood fit (0–4)

Does the type pairing match the voice?

| Voice | Pairing examples |
|---|---|
| Editorial / luxury / wellness | Italic transitional serif (Cormorant, Playfair) + clean sans (Instrument, Söhne, Inter alternative) |
| Tech / developer / SaaS | Variable sans (Geist, IBM Plex, Söhne) + variable mono (JetBrains, Geist Mono, Berkeley) |
| Banking / finance | Geometric sans (Söhne, Aktiv, Akzidenz) at confident sizes + monospace numerics (tabular figures) |
| Children's / education | Friendly humanist (Quicksand, Nunito, DM Sans) at warm weights |
| Editorial print | Old-style serif (Source Serif, Lora) + grotesque sans (Inter, Söhne) |
| Gaming / crypto | Display geometric (Space Grotesk, Cabinet Grotesk) + variable mono |

Score 0 if the type pairing is the AI default for everything (Inter + Inter, system font stack only). Score 4 if the pairing has voice that matches the product.

#### Effect register fit (0–4)

Are the motion and interaction effects appropriate?

- **Brand pages** can carry overdrive: WebGL hero, scroll-triggered orchestration, custom cursors, haptic micro-interactions. The bar is wonder, not utility.
- **Product pages** carry restraint: 150–300ms transitions, hover lift 1–2px, expo-out easing, reduced motion respected. The bar is invisibility.
- **Banking / healthcare** carry maximum restraint: motion communicates state only, no decorative animation.
- **Gaming / entertainment** carry maximum register: kinetic typography, parallax, video backgrounds, 3D transforms.

Score 0 if the effects fight the register (ambient WebGL on a banking app, no motion on a portfolio). Score 4 if effects serve the surface's purpose.

### 3.3 Run the category-reflex check (two altitudes)

Critical. This is what separates "appropriate for the category" from "the AI default for the category."

**First-order reflex**: Could someone guess the theme + palette from the category alone?

| Category | First-order reflex (BLOCK) |
|---|---|
| Observability / monitoring | Dark blue, neon green accents, terminal aesthetic |
| Healthcare | White + teal, rounded corners, soft drop shadow |
| Finance / banking | Navy + gold, geometric sans, conservative |
| Crypto | Neon on black, glitch effects, gradient meshes |
| AI products | Purple-to-blue gradient, glassmorphism, Inter font |
| Wellness | Pastel pink + sage, Cormorant Garamond, soft shadows |
| Children's | Bright primaries, rounded squares, Quicksand |

If the first-order check matches, score reasoning 1/4. The design has reached for the most-trained reflex.

**Second-order reflex**: Could someone guess the aesthetic family from category-plus-anti-references? The first reflex was avoided; the second wasn't.

| Category + anti-reference | Second-order reflex (BLOCK) |
|---|---|
| AI tool that's not SaaS-cream | Editorial-typographic (italic serif + lots of white space) |
| Fintech that's not navy-and-gold | Terminal-native dark mode (black + green monospace) |
| Wellness that's not pastel-pink | Brutalist serif on cream (Cormorant + warm beige) |
| Crypto that's not neon-on-black | Bauhaus geometry + primary colors |
| Healthcare that's not white-teal | Editorial newsprint (warm paper + rule lines) |

If the second-order check matches, score reasoning 2/4. The design avoided the obvious reflex but reached for the next-most-trained pattern.

Score 4 only when the surface is recognizably itself, not a category template.

## Output format

```markdown
## Phase 3 — Reasoning

| Dimension | Score | Note |
|---|---|---|
| Pattern fit | <0-4> | <one-line evidence> |
| Color mood fit | <0-4> | <one-line evidence> |
| Typography mood fit | <0-4> | <one-line evidence> |
| Effect register fit | <0-4> | <one-line evidence> |
| **Total** | **<0-16>** | |

### Category reflex check
- First-order: <pass | fail (specific match)>
- Second-order: <pass | fail (specific match)>

### Findings
- **[P<0-2>, conf <7-10>] <name>**: <evidence>. <recommendation>.
```

## Block decision

- Either reflex check fails → **P1**.
- Total score < 8/16 → **P1**.
- Total score 8–11/16 → **P2**.
- Total score ≥ 12/16 → pass.

## Common mistakes

- **Treating the industry default as correct.** Banking apps that look like every other banking app are reasoning failures, not successes. Score by what serves *this* product, not by category averages.
- **Using "ui-ux-pro-max says category X uses palette Y" as a license.** It's a starting hypothesis, not a verdict. Run the category-reflex check before accepting it.
- **Confusing register with category.** A banking *brand site* can be more visually committed than a banking *product*. Both are banking; the register sets the bar.

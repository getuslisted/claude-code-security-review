# Blocks

A small library of verified UI primitives. Each block already passes the nine-phase pipeline. Paste, then tune to the project's tokens.

Each entry carries: the HTML, the CSS, the rationale baked in, and what it deliberately avoids. Tokens use OKLCH; substitute your project's tokens in place of the placeholders.

## Header

```html
<header class="qc-header">
  <a href="/" class="qc-header__brand" aria-label="Home">
    <span class="qc-header__mark" aria-hidden="true"></span>
    <span class="qc-header__wordmark">Brand</span>
  </a>
  <nav class="qc-header__nav" aria-label="Primary">
    <a href="/work">Work</a>
    <a href="/about">About</a>
    <a href="/contact">Contact</a>
  </nav>
  <a href="/start" class="qc-button qc-button--primary">Start</a>
</header>
```

```css
.qc-header {
  display: flex;
  align-items: center;
  gap: var(--space-md);
  height: 62px;
  padding-inline: var(--space-lg);
  background: var(--color-paper);
  border-bottom: 1px solid var(--color-mist);
}
.qc-header__brand { display: inline-flex; align-items: center; gap: var(--space-xs); text-decoration: none; }
.qc-header__nav { display: flex; gap: var(--space-md); margin-inline-start: auto; }
.qc-header__nav a {
  color: var(--color-ink);
  font-weight: 500;
  text-decoration: none;
  transition: color 200ms cubic-bezier(0.16, 1, 0.3, 1);
}
.qc-header__nav a:hover { color: var(--color-accent); }
.qc-header__nav a:focus-visible {
  outline: 2px solid var(--color-accent);
  outline-offset: 4px;
  border-radius: 2px;
}
@media (max-width: 720px) {
  .qc-header__nav { display: none; }
}
```

**Why it passes**:

- Logical CSS properties (`padding-inline`, `margin-inline-start`) work in RTL.
- Active focus indicator at ≥ 3:1 contrast.
- 62px header height clears the ≥ 44px touch-target rule with margin.
- Hairline border (1px Mist), no side stripes anywhere.
- No glass blur. No gradient. No animation on layout properties.

**What it avoids**: dropdown menus on hover (touch-hostile), oversized 80px+ headers (steal scroll), centered-everything navigation (poor scannability), `outline: none` without replacement.

---

## Hero (no metric template)

```html
<section class="qc-hero">
  <div class="qc-hero__copy">
    <p class="qc-hero__eyebrow">Releases</p>
    <h1 class="qc-hero__title">Ship with the same care a print editor gives a final proof.</h1>
    <p class="qc-hero__lead">A nine-phase pipeline reads your branch, scores the work, and tells you the one thing to fix first.</p>
    <a href="/install" class="qc-button qc-button--primary">Install</a>
  </div>
  <figure class="qc-hero__figure">
    <img src="/hero.webp" alt="Editor reviewing a paper proof on a worn wood desk." width="800" height="600" />
  </figure>
</section>
```

```css
.qc-hero {
  display: grid;
  grid-template-columns: minmax(0, 5fr) minmax(0, 4fr);
  gap: var(--space-2xl);
  padding-block: var(--space-3xl);
  align-items: end;
}
.qc-hero__eyebrow {
  font-size: 0.6875rem;
  font-weight: 500;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--color-ash);
  margin-block-end: var(--space-sm);
}
.qc-hero__title {
  font-family: var(--font-display);
  font-size: clamp(2.5rem, 7vw, 4.5rem);
  font-weight: 300;
  font-style: italic;
  line-height: 1;
  letter-spacing: -0.01em;
  color: var(--color-ink);
}
.qc-hero__lead {
  max-width: 60ch;
  margin-block: var(--space-lg);
  font-size: 1.0625rem;
  line-height: 1.6;
  color: var(--color-charcoal);
}
.qc-hero__figure img {
  width: 100%;
  height: auto;
  display: block;
}
@media (max-width: 720px) {
  .qc-hero { grid-template-columns: 1fr; gap: var(--space-xl); }
}
```

**Why it passes**:

- Asymmetric two-column grid; deliberately avoids the centered-everything hero.
- Italic display type; one decisive accent reserved for the CTA, used at < 10% of screen area.
- `clamp()` fluid heading; fixed body type at 1.0625rem.
- Body line length ~60ch.
- `width` and `height` on the image prevent layout shift.
- Image alt describes the scene, not "image of".

**What it avoids**: hero-metric template (big number + small label + supporting stats), centered single-column hero with gradient, glass-card hero panel, "lightning fast" hollow-confidence headline.

---

## Feature section (varied, not identical card grid)

```html
<section class="qc-features">
  <h2 class="qc-features__title">What it catches</h2>
  <div class="qc-features__grid">
    <article class="qc-feature qc-feature--lead">
      <h3>Security</h3>
      <p>Concrete attack paths only. Confidence ≥ 0.8 to surface. The rest is noise.</p>
    </article>
    <article class="qc-feature">
      <h3>Anti-patterns</h3>
      <p>Side-stripe borders, gradient text, hero-metric templates, modal-as-first-thought.</p>
    </article>
    <article class="qc-feature qc-feature--wide">
      <h3>Design-system drift</h3>
      <p>Every hard-coded color, off-scale gap, and one-off component, classed by root cause.</p>
    </article>
    <article class="qc-feature">
      <h3>WCAG AA</h3>
      <p>Contrast under 4.5:1, touch targets under 44px, missing focus rings, illogical tab order.</p>
    </article>
  </div>
</section>
```

```css
.qc-features__grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-auto-rows: minmax(220px, auto);
  gap: var(--space-lg);
}
.qc-feature {
  padding: var(--space-lg);
  background: var(--color-paper);
  border: 1px solid var(--color-mist);
}
.qc-feature--lead {
  grid-column: span 2;
  background: var(--color-paper-warm);
  padding: var(--space-xl);
}
.qc-feature--wide { grid-column: span 2; }
.qc-feature h3 {
  font-family: var(--font-display);
  font-size: 1.5rem;
  font-weight: 400;
  margin-block-end: var(--space-sm);
}
@media (max-width: 720px) {
  .qc-features__grid { grid-template-columns: 1fr; }
  .qc-feature--lead, .qc-feature--wide { grid-column: span 1; }
}
```

**Why it passes**:

- Cards vary in size and role (lead, standard, wide), passing the no-identical-card-grid check.
- Hairline 1px border for articulation, no side stripes.
- Cards are flat at rest. No shadow.
- Grid degrades cleanly to single-column under 720px.

**What it avoids**: four identical 1-of-4 columns of icon + heading + text, decorative gradient borders, nested cards.

---

## Card (no nested, no side stripe)

```html
<article class="qc-card">
  <header class="qc-card__header">
    <span class="qc-card__eyebrow">Phase 4</span>
    <h3 class="qc-card__title">Accessibility hardening</h3>
  </header>
  <p class="qc-card__body">Twelve checks. WCAG AA is the floor. Contrast, focus, keyboard, semantics, ARIA, motion.</p>
  <a href="/phases/04-accessibility" class="qc-card__link">Read the checks <span aria-hidden="true">→</span></a>
</article>
```

```css
.qc-card {
  display: flex;
  flex-direction: column;
  gap: var(--space-sm);
  padding: var(--space-lg);
  background: var(--color-paper);
  border: 1px solid var(--color-mist);
  border-radius: 8px;
  min-width: 0;
  transition: transform 200ms cubic-bezier(0.16, 1, 0.3, 1),
              box-shadow 200ms cubic-bezier(0.16, 1, 0.3, 1);
}
.qc-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 24px -4px rgba(0,0,0,0.12), 0 1px 3px rgba(0,0,0,0.06);
}
.qc-card__eyebrow {
  font-size: 0.6875rem;
  font-weight: 500;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--color-ash);
}
.qc-card__title {
  font-family: var(--font-display);
  font-size: 1.5rem;
  font-weight: 400;
}
.qc-card__body { color: var(--color-charcoal); line-height: 1.6; max-width: 60ch; }
.qc-card__link {
  margin-top: auto;
  font-weight: 500;
  color: var(--color-ink);
  text-decoration: none;
  transition: color 200ms cubic-bezier(0.16, 1, 0.3, 1);
}
.qc-card__link:hover { color: var(--color-accent); }
.qc-card__link:focus-visible { outline: 2px solid var(--color-accent); outline-offset: 4px; }
@media (prefers-reduced-motion: reduce) {
  .qc-card { transition: none; }
  .qc-card:hover { transform: none; }
}
```

**Why it passes**:

- Flat at rest. Shadow appears only on hover. Strongest blur uses 0.12 alpha.
- 8px radius from controlled vocabulary. No oversized rounded-lg default.
- `min-width: 0` allows shrinking inside flex / grid parents.
- Hover translateY uses `transform`, not layout property.
- `prefers-reduced-motion` disables the lift.

**What it avoids**: nested cards, gradient backgrounds, side-stripe borders, `outline: none`, hover effects that animate `padding`.

---

## Button (sharp and considered)

```html
<button class="qc-button qc-button--primary">Submit</button>
<button class="qc-button qc-button--ghost">Cancel</button>
```

```css
.qc-button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-xs);
  min-height: 44px;
  padding: var(--space-sm) var(--space-xl);
  font-family: inherit;
  font-size: 0.9rem;
  font-weight: 500;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  border: 1px solid transparent;
  border-radius: 0;
  cursor: pointer;
  transition: background 200ms cubic-bezier(0.16, 1, 0.3, 1),
              color 200ms cubic-bezier(0.16, 1, 0.3, 1),
              transform 200ms cubic-bezier(0.16, 1, 0.3, 1);
}
.qc-button--primary { background: var(--color-ink); color: var(--color-paper); }
.qc-button--primary:hover { background: var(--color-accent); transform: translateY(-2px); }
.qc-button--primary:focus-visible { outline: 2px solid var(--color-accent); outline-offset: 3px; }
.qc-button--primary:active { transform: translateY(0); }
.qc-button--primary[disabled] { background: var(--color-ash); cursor: not-allowed; transform: none; }

.qc-button--ghost { background: transparent; color: var(--color-ink); border-color: var(--color-mist); }
.qc-button--ghost:hover { color: var(--color-accent); border-color: var(--color-accent); }

@media (prefers-reduced-motion: reduce) {
  .qc-button { transition: none; }
  .qc-button:hover { transform: none; }
}
```

**Why it passes**:

- Min-height 44px clears touch target on mobile.
- Sharp corners (`border-radius: 0`) signal restraint.
- Three states present: default, hover, focus-visible, active, disabled.
- Disabled state communicates non-interactivity (cursor, color).
- Reduced motion respected.

**What it avoids**: rounded-rectangle-with-drop-shadow default, gradient fill, bouncy click animation, hover that animates `padding`.

---

## Form (label, error, help)

```html
<form class="qc-form" novalidate>
  <div class="qc-field">
    <label for="email" class="qc-field__label">Email</label>
    <input
      id="email"
      name="email"
      type="email"
      class="qc-field__input"
      aria-describedby="email-help email-error"
      aria-invalid="false"
      autocomplete="email"
      required
    />
    <p id="email-help" class="qc-field__help">We'll only use this to send the verdict report.</p>
    <p id="email-error" class="qc-field__error" hidden>Enter a valid email.</p>
  </div>

  <button type="submit" class="qc-button qc-button--primary">Send report</button>
</form>
```

```css
.qc-form { display: flex; flex-direction: column; gap: var(--space-md); max-width: 480px; }
.qc-field { display: flex; flex-direction: column; gap: var(--space-xs); }
.qc-field__label { font-size: 0.875rem; font-weight: 500; color: var(--color-ink); }
.qc-field__input {
  font: inherit;
  padding: var(--space-sm) var(--space-md);
  background: transparent;
  color: var(--color-ink);
  border: 1px solid var(--color-mist);
  border-radius: 4px;
  transition: border-color 200ms cubic-bezier(0.16, 1, 0.3, 1),
              box-shadow 200ms cubic-bezier(0.16, 1, 0.3, 1);
}
.qc-field__input:focus-visible {
  border-color: var(--color-accent);
  box-shadow: 0 0 0 3px var(--color-accent-veil);
  outline: none;
}
.qc-field__input[aria-invalid="true"] {
  border-color: var(--color-error);
}
.qc-field__help { font-size: 0.875rem; color: var(--color-ash); }
.qc-field__error { font-size: 0.875rem; color: var(--color-error); }
```

**Why it passes**:

- Programmatic label via `for` / `id`.
- Help text and error text linked via `aria-describedby`.
- `aria-invalid` toggles based on validation state.
- Error message preserved in DOM (not flash).
- Focus ring uses border-color + box-shadow, not `outline: none` alone.

**What it avoids**: placeholder-as-label, error message that disappears on next keystroke, validation that blocks submission without explanation, no autocomplete attribute.

---

## Empty state

```html
<div class="qc-empty">
  <svg class="qc-empty__icon" aria-hidden="true" viewBox="0 0 64 64"><!-- … --></svg>
  <h3 class="qc-empty__title">No findings yet.</h3>
  <p class="qc-empty__body">Run Quality Checks against a branch or a block to see the verdict.</p>
  <a href="/run" class="qc-button qc-button--primary">Run Quality Checks</a>
</div>
```

```css
.qc-empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  gap: var(--space-md);
  padding: var(--space-2xl);
  background: var(--color-paper);
  border: 1px dashed var(--color-mist);
  border-radius: 8px;
}
.qc-empty__icon { width: 48px; height: 48px; color: var(--color-ash); }
.qc-empty__title { font-family: var(--font-display); font-size: 1.5rem; font-weight: 400; }
.qc-empty__body { max-width: 40ch; color: var(--color-charcoal); }
```

**Why it passes**:

- Empty state offers the next action (CTA), not a dead end.
- Dashed border distinguishes it from a content card.
- Icon is decorative (`aria-hidden`); copy carries the meaning.

**What it avoids**: blank screen, "No data" with no recovery path, animated 404-style mascot drained of context.

---

## Loading state (skeleton)

```html
<div class="qc-skeleton" role="status" aria-label="Loading verdict report">
  <div class="qc-skeleton__line qc-skeleton__line--lg"></div>
  <div class="qc-skeleton__line qc-skeleton__line--md"></div>
  <div class="qc-skeleton__line qc-skeleton__line--sm"></div>
</div>
```

```css
.qc-skeleton { display: flex; flex-direction: column; gap: var(--space-sm); padding: var(--space-md); }
.qc-skeleton__line {
  height: 14px;
  background: var(--color-mist);
  border-radius: 2px;
  animation: qc-skeleton-pulse 1200ms ease-in-out infinite;
}
.qc-skeleton__line--lg { width: 80%; }
.qc-skeleton__line--md { width: 60%; }
.qc-skeleton__line--sm { width: 40%; }
@keyframes qc-skeleton-pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}
@media (prefers-reduced-motion: reduce) {
  .qc-skeleton__line { animation: none; }
}
```

**Why it passes**:

- Animates `opacity`, not layout.
- `role="status"` with an accessible name announces to assistive tech.
- Skeleton dimensions match the loaded content's size, preventing layout shift on reveal.
- `prefers-reduced-motion` stops the pulse.

**What it avoids**: spinner overlay that blocks the entire screen, animated gradient (drains battery, drops frames), content that jumps in size when loaded.

---

## Error state (recoverable)

```html
<div class="qc-error" role="alert">
  <h3 class="qc-error__title">Couldn't load the verdict.</h3>
  <p class="qc-error__body">The report service didn't respond. The branch and your local report are unaffected.</p>
  <div class="qc-error__actions">
    <button type="button" class="qc-button qc-button--primary" data-action="retry">Try again</button>
    <a href="/status" class="qc-error__link">Check service status</a>
  </div>
</div>
```

```css
.qc-error {
  display: flex;
  flex-direction: column;
  gap: var(--space-md);
  padding: var(--space-lg);
  background: var(--color-paper);
  border: 1px solid var(--color-error);
  border-radius: 8px;
}
.qc-error__title { font-family: var(--font-display); font-size: 1.25rem; font-weight: 400; color: var(--color-ink); }
.qc-error__body { color: var(--color-charcoal); max-width: 60ch; }
.qc-error__actions { display: flex; gap: var(--space-md); align-items: center; flex-wrap: wrap; }
.qc-error__link { color: var(--color-ink); text-decoration: underline; text-underline-offset: 3px; }
```

**Why it passes**:

- `role="alert"` announces to assistive tech.
- Error explains what failed, what's safe, and what to do next.
- Two recovery paths (retry, status link).
- 1px solid border, not a colored side stripe.

**What it avoids**: red modal dialog as first thought, "Error occurred. Please try again later." with no detail, error toast that vanishes before it can be read, error state that loses user input.

---

## Footer

```html
<footer class="qc-footer">
  <div class="qc-footer__cols">
    <section>
      <h4>Pipeline</h4>
      <ul>
        <li><a href="/phases/01-security">Security</a></li>
        <li><a href="/phases/02-anti-patterns">Anti-patterns</a></li>
        <li><a href="/phases/04-accessibility">Accessibility</a></li>
      </ul>
    </section>
    <section>
      <h4>Reference</h4>
      <ul>
        <li><a href="/checklist">Checklist</a></li>
        <li><a href="/rubric">Rubric</a></li>
        <li><a href="/blocks">Blocks</a></li>
      </ul>
    </section>
    <section>
      <h4>Source</h4>
      <ul>
        <li><a href="https://github.com/getuslisted/claude-code-security-review">claude-code-security-review</a></li>
        <li><a href="https://github.com/getuslisted/impeccable">impeccable</a></li>
        <li><a href="https://github.com/getuslisted/ui-ux-pro-max-skill">ui-ux-pro-max-skill</a></li>
      </ul>
    </section>
  </div>
  <p class="qc-footer__meta">v1.0.0 · Quality Checks</p>
</footer>
```

```css
.qc-footer { padding: var(--space-2xl) var(--space-lg); border-top: 1px solid var(--color-mist); }
.qc-footer__cols { display: grid; grid-template-columns: repeat(3, 1fr); gap: var(--space-xl); }
.qc-footer h4 { font-size: 0.6875rem; text-transform: uppercase; letter-spacing: 0.1em; color: var(--color-ash); margin-block-end: var(--space-sm); }
.qc-footer ul { list-style: none; padding: 0; margin: 0; display: flex; flex-direction: column; gap: var(--space-xs); }
.qc-footer a { color: var(--color-ink); text-decoration: none; }
.qc-footer a:hover { color: var(--color-accent); }
.qc-footer__meta { margin-top: var(--space-xl); font-size: 0.75rem; color: var(--color-ash); }
@media (max-width: 720px) {
  .qc-footer__cols { grid-template-columns: 1fr; }
}
```

**Why it passes**: lists wrap related items in `<ul>`, columns degrade cleanly, micro-label uses uppercase+tracked, hairline border-top only.

**What it avoids**: oversized footer with social-proof carousel, dark-mode-by-default footer that breaks the page, decorative gradient header on the footer block.

---

## Token surface (drop into `:root`)

The blocks reference these tokens. Substitute the project's equivalents.

```css
:root {
  /* Color (OKLCH; chroma reduced toward extremes) */
  --color-ink: oklch(10% 0.005 280);
  --color-charcoal: oklch(25% 0.005 280);
  --color-ash: oklch(55% 0.005 280);
  --color-mist: oklch(92% 0.005 280);
  --color-paper: oklch(98% 0.003 280);
  --color-paper-warm: oklch(96% 0.005 350);
  --color-accent: oklch(60% 0.20 350);
  --color-accent-veil: oklch(60% 0.20 350 / 0.25);
  --color-error: oklch(50% 0.18 25);

  /* Type */
  --font-display: "Cormorant Garamond", Georgia, serif;
  --font-body: "Instrument Sans", system-ui, sans-serif;
  --font-mono: "Space Grotesk", monospace;

  /* Spacing (no 4px step; editorial scale) */
  --space-xs: 8px;
  --space-sm: 16px;
  --space-md: 24px;
  --space-lg: 32px;
  --space-xl: 48px;
  --space-2xl: 80px;
  --space-3xl: 120px;
}
```

These tokens are starting values; tune them to your `DESIGN.md` or `design-system/MASTER.md`. The blocks will track once your token names match.

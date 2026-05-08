# Block recipes

Reusable component patterns that pass all eight phases by construction. When a finding's recommendation says "use the canonical pattern," it's pointing at one of these.

Every recipe is:
- **Register-aware**: brand and product variants where they differ.
- **Framework-portable**: HTML + CSS + ARIA. Adapt to React/Vue/Svelte/Astro by hand; the structure stays the same.
- **Token-driven**: every value references a CSS custom property. The recipes assume the project has a tokens layer.
- **Accessibility-complete**: focus, keyboard, screen reader, reduced-motion all handled.

## The token foundation

Every recipe references these. The names match impeccable's DESIGN.md conventions; rename to match your project.

```css
:root {
  /* Color: tinted neutrals, OKLCH */
  --color-ink: oklch(15% 0.005 270);          /* primary text */
  --color-charcoal: oklch(28% 0.005 270);     /* headings, large body */
  --color-ash: oklch(55% 0.005 270);          /* secondary text */
  --color-mist: oklch(92% 0.003 270);         /* hairline borders */
  --color-paper: oklch(98% 0.003 270);        /* page background */
  --color-cream: oklch(96% 0.005 270);        /* warm surface */
  --color-accent: oklch(60% 0.18 25);         /* one decisive accent */
  --color-accent-deep: oklch(52% 0.20 25);    /* hover */
  --color-focus: oklch(60% 0.20 250);         /* focus ring */
  --color-error: oklch(55% 0.20 25);
  --color-success: oklch(55% 0.18 145);

  /* Spacing: 8/16/24/32/48/80/120 */
  --space-xs: 8px;
  --space-sm: 16px;
  --space-md: 24px;
  --space-lg: 32px;
  --space-xl: 48px;
  --space-2xl: 80px;
  --space-3xl: 120px;

  /* Type */
  --font-display: "Cormorant Garamond", Georgia, serif;
  --font-body: "Inter", system-ui, sans-serif;
  --font-mono: "JetBrains Mono", "SF Mono", monospace;

  /* Radius */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 12px;
  --radius-xl: 16px;

  /* Motion */
  --ease-out: cubic-bezier(0.16, 1, 0.3, 1);
  --ease-out-quint: cubic-bezier(0.22, 1, 0.36, 1);
  --duration-fast: 150ms;
  --duration-base: 200ms;
  --duration-slow: 300ms;
}

@media (prefers-reduced-motion: reduce) {
  :root {
    --duration-fast: 0.01ms;
    --duration-base: 0.01ms;
    --duration-slow: 0.01ms;
  }
}
```

## Button

The squared, sharp, letter-tracked CTA. The single most over-templated component on the web; the recipe rejects the rounded-rectangle-with-drop-shadow default.

```html
<button type="button" class="btn btn-primary">Continue</button>
<button type="button" class="btn btn-secondary">Cancel</button>
<button type="button" class="btn btn-ghost" aria-label="Close">
  <svg aria-hidden="true" viewBox="0 0 24 24" width="16" height="16">
    <path d="M6 6L18 18M18 6L6 18" stroke="currentColor" stroke-width="2"/>
  </svg>
</button>
```

```css
.btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: var(--space-xs);
  font-family: var(--font-body);
  font-weight: 500;
  font-size: 0.9rem;
  letter-spacing: 0.05em;
  text-transform: uppercase;
  padding: var(--space-sm) var(--space-xl);
  border: 0;
  border-radius: 0;
  cursor: pointer;
  transition: transform var(--duration-base) var(--ease-out),
              background-color var(--duration-base) var(--ease-out);
  min-height: 44px;
}
.btn-primary { background: var(--color-ink); color: var(--color-paper); }
.btn-primary:hover { background: var(--color-accent); transform: translateY(-2px); }
.btn-secondary { background: transparent; color: var(--color-ink); border: 1px solid var(--color-mist); }
.btn-secondary:hover { border-color: var(--color-ink); }
.btn-ghost { background: transparent; color: var(--color-ash); padding: var(--space-xs); min-width: 44px; }
.btn-ghost:hover { color: var(--color-ink); }
.btn:focus-visible { outline: 2px solid var(--color-focus); outline-offset: 2px; }
.btn:disabled { opacity: 0.5; cursor: not-allowed; }
```

Avoids: rounded-rectangle-with-drop-shadow, bouncy easing, layout-property animation, missing focus indicator, touch target under 44px, color-only disabled state.

## Input

```html
<div class="field">
  <label for="email" class="field-label">Email address</label>
  <input id="email" type="email" name="email" class="field-input" autocomplete="email" required
         aria-describedby="email-hint email-error" aria-invalid="false" />
  <p id="email-hint" class="field-hint">We use this only to send your receipt.</p>
  <p id="email-error" class="field-error" hidden>Enter a valid email like name@example.com.</p>
</div>
```

```css
.field { display: flex; flex-direction: column; gap: var(--space-xs); min-width: 0; }
.field-label { font-family: var(--font-body); font-size: 0.875rem; font-weight: 500; color: var(--color-charcoal); }
.field-input {
  font-family: var(--font-body); font-size: 1rem; color: var(--color-ink);
  background: transparent; border: 1px solid var(--color-mist); border-radius: var(--radius-sm);
  padding: var(--space-sm); min-height: 44px;
  transition: border-color var(--duration-base) var(--ease-out), box-shadow var(--duration-base) var(--ease-out);
}
.field-input:hover { border-color: var(--color-ash); }
.field-input:focus-visible {
  outline: 0; border-color: var(--color-focus);
  box-shadow: 0 0 0 3px oklch(60% 0.20 250 / 0.2);
}
.field-input[aria-invalid="true"] { border-color: var(--color-error); }
.field-hint { font-size: 0.8125rem; color: var(--color-ash); }
.field-error { font-size: 0.8125rem; color: var(--color-error); }
.field-error[hidden] { display: none; }
```

Avoids: placeholder-as-label, missing required attributes, error not connected via aria-describedby, font-size under 16px (iOS zoom), missing focus replacement.

## Card

```html
<article class="card">
  <h3 class="card-title">Quarterly summary</h3>
  <p class="card-body">Revenue is up 18% from last quarter, on the back of expanded enterprise contracts.</p>
  <footer class="card-meta">Last updated 2 hours ago</footer>
</article>

<a href="/projects/atlas" class="card card-interactive">
  <h3 class="card-title">Atlas, observability platform</h3>
  <p class="card-body">147 services. 99.94% uptime this week.</p>
</a>
```

```css
.card {
  display: flex; flex-direction: column; gap: var(--space-sm);
  background: var(--color-paper); padding: var(--space-md);
  border: 1px solid var(--color-mist); border-radius: var(--radius-md);
}
.card-interactive {
  text-decoration: none; color: inherit;
  transition: transform var(--duration-base) var(--ease-out),
              border-color var(--duration-base) var(--ease-out),
              box-shadow var(--duration-base) var(--ease-out);
}
.card-interactive:hover {
  transform: translateY(-2px); border-color: var(--color-ash);
  box-shadow: 0 4px 24px -4px oklch(0% 0 0 / 0.10), 0 1px 3px oklch(0% 0 0 / 0.05);
}
.card-interactive:focus-visible { outline: 2px solid var(--color-focus); outline-offset: 4px; }
.card-title { font-family: var(--font-display); font-weight: 400; font-size: 1.25rem; line-height: 1.3; color: var(--color-ink); margin: 0; }
.card-body { font-family: var(--font-body); font-size: 1rem; line-height: 1.6; color: var(--color-charcoal); margin: 0; }
.card-meta { font-family: var(--font-body); font-size: 0.8125rem; color: var(--color-ash); margin-top: auto; }
```

Avoids: nested cards, side-stripe accent border, shadow at rest, touch target failure.

## Empty state

```html
<div class="empty">
  <div class="empty-graphic" aria-hidden="true">
    <svg viewBox="0 0 64 64" width="64" height="64">
      <circle cx="32" cy="32" r="28" stroke="currentColor" fill="none" stroke-width="1"/>
      <path d="M22 32 H42 M32 22 V42" stroke="currentColor" stroke-width="1.5"/>
    </svg>
  </div>
  <h3 class="empty-title">No projects yet</h3>
  <p class="empty-body">Projects group your work and unlock collaboration. Start by creating one for your current sprint.</p>
  <button type="button" class="btn btn-primary">Create your first project</button>
</div>
```

```css
.empty {
  display: flex; flex-direction: column; align-items: center; text-align: center;
  gap: var(--space-md); padding: var(--space-2xl) var(--space-md);
  max-width: 480px; margin: 0 auto;
}
.empty-graphic { color: var(--color-ash); margin-bottom: var(--space-sm); }
.empty-title { font-family: var(--font-display); font-weight: 400; font-size: 1.5rem; margin: 0; color: var(--color-ink); }
.empty-body { font-family: var(--font-body); font-size: 1rem; line-height: 1.6; color: var(--color-charcoal); margin: 0; }
```

Three required variants: empty (no data yet), filtered to zero, permission denied. Don't ship one for all three.

## Error state (network)

```html
<div class="error-state" role="alert" aria-live="polite">
  <h3 class="error-title">Couldn't load your projects</h3>
  <p class="error-body">The server didn't respond in time. This usually clears up within a minute.</p>
  <div class="error-actions">
    <button type="button" class="btn btn-primary" data-action="retry">Try again</button>
    <a class="btn btn-secondary" href="/help/connectivity">Troubleshoot</a>
  </div>
</div>
```

```css
.error-state {
  display: flex; flex-direction: column; gap: var(--space-md);
  padding: var(--space-lg);
  background: oklch(96% 0.02 25);
  border: 1px solid oklch(80% 0.10 25);
  border-radius: var(--radius-md);
  /* Full border, NEVER border-left: Xpx solid. */
}
.error-title { font-family: var(--font-display); font-size: 1.125rem; color: var(--color-ink); margin: 0; }
.error-body { font-family: var(--font-body); font-size: 1rem; line-height: 1.6; color: var(--color-charcoal); margin: 0; }
.error-actions { display: flex; gap: var(--space-sm); flex-wrap: wrap; }
```

Avoids: side-stripe border, generic "Error occurred" copy, no retry path.

## Loading state (skeleton)

```html
<ul class="list" aria-busy="true" aria-label="Loading projects">
  <li class="list-row skeleton" aria-hidden="true">
    <div class="skeleton-line skeleton-title"></div>
    <div class="skeleton-line skeleton-meta"></div>
  </li>
  <li class="list-row skeleton" aria-hidden="true">
    <div class="skeleton-line skeleton-title"></div>
    <div class="skeleton-line skeleton-meta"></div>
  </li>
</ul>
```

```css
.skeleton { padding: var(--space-md); border-bottom: 1px solid var(--color-mist); }
.skeleton-line {
  height: 1em; border-radius: var(--radius-sm);
  background: linear-gradient(90deg, var(--color-mist) 0%, var(--color-cream) 50%, var(--color-mist) 100%);
  background-size: 200% 100%;
  animation: skeleton-shimmer 1.6s infinite var(--ease-out);
}
.skeleton-title { width: 60%; margin-bottom: var(--space-xs); }
.skeleton-meta { width: 30%; height: 0.875em; }
@keyframes skeleton-shimmer { 0% { background-position: 100% 0; } 100% { background-position: -100% 0; } }
@media (prefers-reduced-motion: reduce) { .skeleton-line { animation: none; background: var(--color-mist); } }
```

The skeleton must match the loaded layout exactly. Mismatch causes CLS.

## Modal (only when nothing else fits)

If you reached for this, double-check that none of these alternatives work:
- Inline expansion (accordion / disclosure)
- Side panel that doesn't trap focus
- Separate route
- Popover with confirm

If genuinely required, use a primitive library (Radix Dialog, Headless UI Dialog, Ariakit Dialog). Keyboard, focus management, and ARIA semantics are guaranteed.

```tsx
import * as Dialog from '@radix-ui/react-dialog';

<Dialog.Root>
  <Dialog.Trigger asChild>
    <button className="btn btn-primary">Delete project</button>
  </Dialog.Trigger>
  <Dialog.Portal>
    <Dialog.Overlay className="dialog-overlay" />
    <Dialog.Content className="dialog-content">
      <Dialog.Title className="dialog-title">Delete this project?</Dialog.Title>
      <Dialog.Description className="dialog-description">
        This removes 47 files and 3 collaborators. It can't be undone.
      </Dialog.Description>
      <div className="dialog-actions">
        <Dialog.Close asChild>
          <button className="btn btn-secondary">Cancel</button>
        </Dialog.Close>
        <button className="btn btn-primary btn-destructive" onClick={handleDelete}>
          Delete project
        </button>
      </div>
    </Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>
```

Animate `opacity` on overlay and `transform: scale(0.96 → 1)` on content; never animate width/height.

## Alert (the one that catches the side-stripe ban)

```html
<div class="alert alert-warning" role="alert">
  <svg class="alert-icon" aria-hidden="true" viewBox="0 0 24 24" width="20" height="20">
    <path d="M12 9v4M12 17h.01" stroke="currentColor" stroke-width="2" fill="none"/>
    <circle cx="12" cy="12" r="9" stroke="currentColor" stroke-width="1.5" fill="none"/>
  </svg>
  <div class="alert-body">
    <h4 class="alert-title">Two-factor authentication is off</h4>
    <p class="alert-text">Your account uses password-only login. Enable 2FA to protect against credential leaks.</p>
  </div>
  <a href="/settings/security" class="alert-action">Set up 2FA</a>
</div>
```

```css
.alert {
  display: flex; align-items: flex-start; gap: var(--space-sm);
  padding: var(--space-md); border-radius: var(--radius-md);
  border: 1px solid;
  /* Critical: NO border-left: Xpx solid. The full border carries the meaning. */
}
.alert-warning { background: oklch(96% 0.04 75); border-color: oklch(75% 0.12 75); color: oklch(35% 0.10 75); }
.alert-error { background: oklch(96% 0.03 25); border-color: oklch(75% 0.12 25); color: oklch(35% 0.12 25); }
.alert-info { background: oklch(96% 0.02 250); border-color: oklch(75% 0.08 250); color: oklch(35% 0.08 250); }
.alert-success { background: oklch(96% 0.02 145); border-color: oklch(75% 0.08 145); color: oklch(30% 0.08 145); }
.alert-icon { flex-shrink: 0; margin-top: 2px; }
.alert-body { flex: 1; min-width: 0; }
.alert-title { font-family: var(--font-body); font-weight: 600; font-size: 0.9375rem; margin: 0 0 var(--space-xs) 0; color: inherit; }
.alert-text { font-family: var(--font-body); font-size: 0.875rem; line-height: 1.55; margin: 0; color: inherit; }
.alert-action { flex-shrink: 0; font-weight: 500; text-decoration: none; color: inherit; border-bottom: 1px solid currentColor; padding-bottom: 1px; align-self: flex-start; }
.alert-action:hover { border-bottom-width: 2px; padding-bottom: 0; }
```

The cardinal sin this avoids: `border-left: 4px solid var(--color-warning)`. Full border only.

## Form

```html
<form class="form" novalidate>
  <fieldset class="form-section">
    <legend class="form-section-title">Account</legend>
    <div class="field">
      <label for="name" class="field-label">Full name</label>
      <input id="name" name="name" type="text" class="field-input" required autocomplete="name" />
    </div>
    <div class="field">
      <label for="email" class="field-label">Email address</label>
      <input id="email" name="email" type="email" class="field-input" required autocomplete="email" />
      <p class="field-hint">We send only receipts and security notifications.</p>
    </div>
  </fieldset>
  <div class="form-actions">
    <button type="submit" class="btn btn-primary">Create account</button>
    <a href="/login" class="form-link">Already have one? Sign in.</a>
  </div>
</form>
```

```css
.form { display: flex; flex-direction: column; gap: var(--space-xl); max-width: 480px; }
.form-section { display: flex; flex-direction: column; gap: var(--space-md); border: 0; padding: 0; margin: 0; }
.form-section-title { font-family: var(--font-display); font-size: 1.25rem; margin-bottom: var(--space-sm); color: var(--color-ink); }
.form-actions { display: flex; align-items: center; gap: var(--space-md); flex-wrap: wrap; }
.form-link { color: var(--color-ash); text-decoration: underline; text-underline-offset: 2px; }
.form-link:hover { color: var(--color-ink); }
```

## Navigation (header)

```html
<header class="site-header">
  <a href="/" class="brand-mark">
    <svg aria-hidden="true" width="24" height="24" viewBox="0 0 24 24"><!-- mark --></svg>
    <span class="brand-name">Atlas</span>
  </a>
  <nav class="primary-nav" aria-label="Primary">
    <ul class="nav-list">
      <li><a href="/projects" class="nav-link">Projects</a></li>
      <li><a href="/team" class="nav-link">Team</a></li>
      <li><a href="/billing" class="nav-link">Billing</a></li>
    </ul>
  </nav>
  <div class="account-area">
    <button type="button" class="btn btn-ghost" aria-label="Notifications"><!-- icon --></button>
    <button type="button" class="account-menu" aria-haspopup="menu">
      <span aria-hidden="true">JD</span>
      <span class="visually-hidden">Account menu</span>
    </button>
  </div>
</header>
```

```css
.site-header { display: flex; align-items: center; gap: var(--space-lg); padding: var(--space-sm) var(--space-md); border-bottom: 1px solid var(--color-mist); }
.primary-nav { flex: 1; }
.nav-list { display: flex; gap: var(--space-md); list-style: none; padding: 0; margin: 0; }
.nav-link { font-family: var(--font-body); font-weight: 500; font-size: 0.9375rem; color: var(--color-ink); text-decoration: none; padding: var(--space-xs) var(--space-sm); transition: color var(--duration-base) var(--ease-out); }
.nav-link:hover { color: var(--color-accent); }
.nav-link:focus-visible { outline: 2px solid var(--color-focus); outline-offset: 4px; border-radius: var(--radius-sm); }
.nav-link[aria-current="page"] { color: var(--color-accent); }
.visually-hidden { position: absolute; width: 1px; height: 1px; padding: 0; margin: -1px; overflow: hidden; clip: rect(0, 0, 0, 0); white-space: nowrap; border: 0; }
```

## How the gate uses these

When a finding lands on a component covered here, the recommendation links to this file by section. Example:

```markdown
**[P1, conf 10] Side-stripe border on `<Alert>`** — `components/Alert.tsx:18`
- Evidence: `border-left: 4px solid var(--color-warning)`
- Recommendation: Replace with the canonical Alert pattern in [block-recipes.md § Alert](./block-recipes.md#alert-the-one-that-catches-the-side-stripe-ban). The recipe uses a full border, not a side-stripe.
```

This is the "minimal edits needed" path the gate is designed to enable. The block already exists, already passes all eight phases, already works in every framework that compiles HTML.

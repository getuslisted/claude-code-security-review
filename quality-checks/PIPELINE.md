# Pipeline

Nine phases. Run them in order. Each phase has explicit inputs, checks, outputs, and exit criteria. Skipping a phase is allowed only when its exit criterion already passes from prior context.

Version: **v1.0.0**.

---

## Phase 0 — Discovery & Context

The pipeline cannot grade what it cannot read. Phase 0 collects every artifact the later phases depend on, in one pass, before any judgment runs.

### Inputs

- The current branch diff (`git diff --merge-base origin/HEAD`).
- The repo root, scanned for: `PRODUCT.md`, `DESIGN.md`, `design-system/MASTER.md`, `design-system/pages/*`, `package.json`, framework config (`astro.config.*`, `next.config.*`, `nuxt.config.*`, `vite.config.*`, `vue.config.*`, `svelte.config.*`, `app.json`, `Info.plist`, `pubspec.yaml`).
- The list of changed files, classified by extension into source, style, content, config, test.

### Checks

1. **Register identification**. Read the task cue first ("landing page", "dashboard"), then the surface in focus, then the `register` field in `PRODUCT.md`. First match wins. Cache `brand` or `product` for the rest of the run.
2. **Design system map**. Walk the project for tokens (CSS variables, theme files, design tokens JSON). Record every token bucket: color, type, spacing, radius, shadow, motion. Note presence or absence of dark-mode variants.
3. **Stack identification**. From `package.json`, frame configs, and file extensions. Record one of: `html-tailwind`, `react`, `nextjs`, `astro`, `vue`, `nuxtjs`, `svelte`, `swiftui`, `react-native`, `flutter`, `shadcn`, `jetpack-compose`, `angular`, `laravel`. The Phase 8 stack-specific check pivots on this value.
4. **Industry classification**. Match against the 161 reasoning rules from ui-ux-pro-max (Tech & SaaS, Finance, Healthcare, E-commerce, Services, Creative, Lifestyle, Emerging Tech). If `PRODUCT.md` carries an explicit category, use it. Otherwise infer from product title and Users section. The classification feeds Phase 2's industry-specific anti-patterns.
5. **Anti-references**. Pull `PRODUCT.md`'s anti-references list. These become Phase 2 second-order checks ("don't be the obvious thing for this category").

### Output

A discovery JSON, kept in scope for every later phase:

```json
{
  "register": "brand|product",
  "stack": "react",
  "industry": "saas|fintech|healthcare|...",
  "design_system": {
    "tokens": { "color": true, "type": true, "spacing": true, "motion": false },
    "dark_mode": false,
    "master": "design-system/MASTER.md"
  },
  "anti_references": ["dark mode with purple gradients", "hero metric template"],
  "diff_files": [ ... ]
}
```

### Exit criterion

`PRODUCT.md` exists and is non-trivial (≥ 200 chars, no `[TODO]` markers). If it fails, halt and run `impeccable teach`. Never synthesize PRODUCT.md from the user's prompt alone.

---

## Phase 1 — Security Lockdown

Borrowed verbatim from `claude-code-security-review`. The model conducts a focused security review of the diff and reports HIGH-confidence vulnerabilities only. Defensive findings, theoretical concerns, and rate-limiting issues do not appear.

### Inputs

- The diff from Phase 0.
- The full file content for any file in the diff (read on demand).
- `PRODUCT.md` for trust boundaries (who is the user, what is the threat model).

### Checks

Three sub-phases.

**Sub-phase 1A — Repository Context Research**. Identify existing security frameworks and libraries. Look for established sanitization and validation patterns. Understand the project's security model.

**Sub-phase 1B — Comparative Analysis**. Compare new code against existing security patterns. Identify deviations. Flag code that introduces a new attack surface.

**Sub-phase 1C — Vulnerability Assessment** across these categories:

- **Input Validation**: SQL injection, command injection, XXE, template injection, NoSQL injection, path traversal.
- **AuthN / AuthZ**: bypass logic, privilege escalation, session flaws, JWT vulnerabilities, IDOR.
- **Crypto & Secrets**: hardcoded keys, weak algorithms, improper key storage, randomness issues, certificate validation bypass.
- **Injection & Code Execution**: deserialization RCE, pickle injection, YAML deserialization, eval injection, XSS (reflected, stored, DOM).
- **Data Exposure**: sensitive data logging, PII handling violations, API endpoint leakage, debug exposure.

### False-positive filter (hard exclusions)

Do not report any of:

1. Denial-of-service or resource exhaustion.
2. Secrets stored on disk if otherwise secured.
3. Rate-limiting concerns or service overload.
4. Memory or CPU exhaustion.
5. Lack of input validation on non-security-critical fields without proven impact.
6. GitHub Action workflow input sanitization unless clearly triggerable from untrusted input.
7. General lack of hardening; only flag concrete vulnerabilities.
8. Theoretical race conditions.
9. Outdated third-party libraries (managed elsewhere).
10. Memory safety in memory-safe languages (Rust, Go, etc.).
11. Test files.
12. Log spoofing from un-sanitized user input.
13. SSRF that only controls path (not host or protocol).
14. User-controlled content in AI system prompts.
15. Regex injection or regex DoS.
16. Findings in markdown or other documentation files.
17. Lack of audit logs.

### Precedents

- Logging URLs is safe. Logging secrets in plaintext is a vulnerability.
- UUIDs are unguessable; do not require validation.
- Environment variables and CLI flags are trusted.
- React, Angular, and Vue are XSS-safe by default. Only flag XSS in these frameworks for `dangerouslySetInnerHTML`, `bypassSecurityTrustHtml`, `v-html`, or equivalent.
- Most GitHub Action workflow vulnerabilities are not exploitable in practice. Require a concrete attack path.
- Client-side permission or auth checks are not vulnerabilities; the server is the trust boundary.
- Logging non-PII data is not a vulnerability even if the data is sensitive.
- Command injection in shell scripts is generally not exploitable. Require a concrete path for untrusted input.

### Confidence threshold

Confidence 1-10 per finding.

- **8-10**: report.
- **4-7**: investigate, report only with a concrete attack path documented.
- **1-3**: drop.

### Output (per finding)

```json
{
  "file": "path/to/file.ext",
  "line": 42,
  "severity": "HIGH|MEDIUM|LOW",
  "category": "sql_injection",
  "description": "...",
  "exploit_scenario": "...",
  "recommendation": "...",
  "confidence": 0.95
}
```

### Exit criterion

Zero HIGH findings and zero MEDIUM findings with confidence ≥ 0.85. A single qualifying finding fails the entire pipeline regardless of later phase scores. Security is a hard gate.

---

## Phase 2 — Anti-Pattern Audit

The AI-slop test, run at two altitudes. Cross-register failures (the absolute bans) plus register-specific reflexes plus industry-specific reflexes.

### Inputs

- Discovery JSON from Phase 0.
- All HTML, JSX, TSX, Vue, Svelte, Astro, and CSS in the diff.

### Checks

**Cross-register absolute bans** (any one fails the dimension):

- **Side-stripe borders**. `border-left` or `border-right` greater than 1px as a colored accent on cards, list items, callouts, or alerts. Never intentional. Rewrite with a full border, a background tint, a leading number or icon, or nothing.
- **Gradient text**. `background-clip: text` plus a gradient background. Decorative, never meaningful. Use a single solid color; emphasis via weight or size.
- **Glassmorphism by default**. Blurred translucent cards used decoratively. Rare and purposeful, or nothing.
- **Hero-metric template**. Big number, small label, supporting stats, gradient accent. SaaS cliché.
- **Identical card grids**. Same-sized cards with icon + heading + text, repeated four or more times. Vary scale, role, or shape.
- **Modal as first thought**. Reach for inline or progressive alternatives first.
- **Pure black or pure white** (`#000`, `#fff`). Tint every neutral toward the brand hue.
- **Bounce or elastic easing**. Use `ease-out-quart`, `ease-out-quint`, or `ease-out-expo`.
- **Animating layout properties** (`width`, `height`, `padding`, `margin`). Use `transform` and `opacity` only.
- **Nested cards**. Cards inside cards. Flatten the hierarchy.

**First-order category reflex**. Could someone guess the theme + palette from the category alone?

| Category | Reflex | What it signals |
|---|---|---|
| Observability / DevOps | Dark blue + neon accents | Generic |
| Healthcare | White + teal | Generic |
| Finance / Banking | Navy + gold | Generic |
| Crypto | Neon on black | Generic |
| AI tool | Purple-pink gradients on dark | Generic |
| Wellness / Spa | Soft pink + sage | Generic |

If the answer is "yes, that's exactly what they did", the design failed the first-order check. Rework the scene sentence and color strategy.

**Second-order category reflex**. Given the anti-references, could someone guess the *next* aesthetic family the design ran to?

| First reflex avoided | Common second reflex | What it still signals |
|---|---|---|
| AI tool, not purple-pink | Editorial-typographic on warm cream | "Trying not to look like AI" |
| Fintech, not navy-gold | Terminal-native dark mode | Cliché-from-anti-cliché |
| SaaS, not gradient-on-dark | Brutalist black-and-white | Anti-cliché reflex |

**Industry-specific anti-patterns** (from ui-ux-pro-max's 161 reasoning rules). For the industry detected in Phase 0, load the corresponding anti-references and check each one. Banking should not use AI purple-pink gradients. Healthcare should not use brutalism. Wellness should not use harsh animations.

### Scoring

Score the dimension 0-4:

- **0** — AI slop gallery (5+ tells).
- **1** — Heavy AI aesthetic (3-4 tells).
- **2** — Some tells (1-2 noticeable).
- **3** — Mostly clean (subtle issues only).
- **4** — No AI tells. Distinctive, intentional design.

### Exit criterion

Dimension score ≥ 3, and zero absolute-ban hits. Anything else routes the run to `impeccable bolder` (if the verdict is bland) or `impeccable distill` (if the verdict is overstuffed) or `impeccable critique` (if the diagnosis itself needs more rigor) before re-running Phase 2.

---

## Phase 3 — Design System Conformance

Drift kills design systems quietly. Phase 3 is the loudest siren in the pipeline.

### Inputs

- Design system map from Phase 0.
- All style declarations in the diff (CSS, inline styles, Tailwind classes, theme references, styled-components, etc.).

### Checks

**Color**:

- All colors come from tokens. Hex literals, RGB strings, and HSL strings outside the token file are drift.
- New colors declared in OKLCH, not hex, unless extending a fenced legacy palette.
- Chroma reduces as lightness approaches 0 or 100.
- No `#000` or `#fff` anywhere. Even tinted-neutral tokens carry chroma 0.005-0.01 toward the brand hue.
- Never gray text on a colored background. Use a shade of that color or transparency.

**Typography**:

- Hierarchy contrast ratio ≥ 1.25 between adjacent steps. No flat scales.
- Body line length capped 65-75ch.
- Body line-height fixed at the project value (impeccable's reference is 1.6).
- Headings use `clamp()` fluid sizing. Body uses fixed `rem`.
- Italic is voice for display type, not emphasis inside paragraphs.

**Spacing**:

- All spacing values come from the scale (e.g., 8 / 16 / 24 / 32 / 48 / 80 / 120 px).
- Gaps not on the scale (a stray 13px) are drift unless the project documents them.
- Card internal padding matches visual weight, not applied uniformly.

**Radius**:

- Radius vocabulary controlled. No single rounded-lg default. Pick per component weight.
- 0 for editorial CTAs that signal restraint. 4-12px for chips, cards, frames as the system dictates.

**Shadow / elevation**:

- Flat by default. Shadows respond to state (hover, elevation, focus).
- Strongest blur uses ≤ 0.15 alpha. Higher reads as 2014 Material Design.
- Tinted shadows only for the deliberate accent-glow moment, never decorative.

**Motion**:

- Durations from the scale: 150ms color/opacity, 300-400ms transforms, 600-1200ms orchestrated entrances.
- Easing from the project's curve set. `ease-out-quart` / `quint` / `expo`. No bounce, no elastic.
- `prefers-reduced-motion` collapses every non-essential transition.

**Component reuse**:

- Find shared components in the codebase. Verify the diff uses them, not one-off reimplementations.
- Three categories of drift, each with a different fix:
  - **Missing token**: the value should exist in the system but doesn't. Patch the token file.
  - **One-off implementation**: a shared component already exists but wasn't used. Swap to the shared version.
  - **Conceptual misalignment**: the feature's flow, IA, or hierarchy doesn't match neighboring features. Rework the flow.

### Scoring

Score the dimension 0-4:

- **0** — No theming. Hard-coded everything.
- **1** — Minimal tokens. Mostly hard-coded.
- **2** — Tokens exist but inconsistently used.
- **3** — Tokens used. Minor hard-coded values.
- **4** — Full token system. Dark mode works perfectly.

### Output (per drift)

```
[P1] Hard-coded color in src/components/Hero.tsx:12
  Category: Theming / drift / one-off implementation
  Found: background: #f5f5f5
  Should: background: var(--color-mist), or token equivalent.
  Root cause: missing token (no --color-mist defined). Patch tokens.css to add it.
  Suggested command: /impeccable polish src/components/Hero.tsx
```

### Exit criterion

Dimension score ≥ 3, zero P0 drift, ≤ 3 P1 drift items. Otherwise route to `impeccable polish` to close drift before sign-off.

---

## Phase 4 — Accessibility Hardening

WCAG AA is the floor, not the ceiling. Phase 4 measures floor compliance and reports anything that pushes toward AAA.

### Inputs

- Rendered DOM (or the JSX/HTML source as proxy when rendering is not available).
- Computed styles (color values, font sizes, dimensions).

### Checks

**Contrast**:

- All text contrast ≥ 4.5:1 against its background (WCAG AA).
- Large text (18px+ or 14px+ bold) ≥ 3:1.
- Icon contrast against background ≥ 3:1 when the icon carries meaning.
- Focus indicators ≥ 3:1 against adjacent surfaces.

**Semantic HTML**:

- `<button>` for buttons, `<a>` for links. No `<div onClick>`.
- Heading hierarchy is monotone (h1 → h2 → h3, no skipping).
- Landmarks present: `<header>`, `<nav>`, `<main>`, `<footer>`.
- Lists wrap related items in `<ul>` or `<ol>`, not stacked `<div>`.

**ARIA**:

- Interactive elements have an accessible name (label, aria-label, aria-labelledby).
- Decorative images use `alt=""`. Meaningful images describe what they convey.
- Live regions on dynamic content (notifications, search results, validation messages).
- States announced (`aria-expanded`, `aria-selected`, `aria-pressed`) where applicable.

**Keyboard**:

- Every interactive element is reachable via Tab.
- Tab order is logical (visual order, with skip links for long content).
- No keyboard traps. Modals trap focus internally and release on close.
- Focus indicators always visible (never `outline: none` without replacement).

**Touch targets**:

- ≥ 44 × 44 px on touch devices.
- Adjacent interactive elements have ≥ 8px separation.

**Forms**:

- Every input has a programmatic label.
- Required fields marked both visually and via `aria-required`.
- Errors associated with inputs via `aria-describedby` or `aria-errormessage`.
- Validation messages preserved in the DOM (not pure visual flash).

**Motion**:

- `prefers-reduced-motion: reduce` collapses non-essential animation.
- No flashing content > 3 Hz (seizure risk).

**Color independence**:

- Color is never the only carrier of meaning. Pair color with icon, text, or shape.
- High-contrast mode (forced colors) does not break the layout.

### Severity

- **P0**: WCAG A failures (no labels, no alt, keyboard inaccessible, contrast < 3:1).
- **P1**: WCAG AA failures (contrast 3:1-4.5:1, missing focus indicator, touch target < 44px).
- **P2**: minor a11y polish (small focus indicator, ambiguous label).
- **P3**: AAA-level enhancement (contrast 4.5:1-7:1).

### Scoring

Score the dimension 0-4:

- **0** — Inaccessible (fails WCAG A).
- **1** — Major gaps. Few ARIA labels. No keyboard navigation.
- **2** — Partial. Some a11y effort. Significant gaps.
- **3** — WCAG AA mostly met. Minor gaps.
- **4** — WCAG AA fully met. Approaches AAA.

### Exit criterion

Dimension score ≥ 3 and zero P0. Route P1+ to `impeccable harden` and `impeccable polish`.

---

## Phase 5 — Performance Audit

Smooth feels designed. Janky feels prototyped.

### Inputs

- The diff plus enough surrounding code to understand render frequency.
- For runtime checks: Lighthouse or Chrome DevTools traces when available.

### Checks

**Animation**:

- No animation of layout properties (`width`, `height`, `top`, `left`, `padding`, `margin`). Use `transform` and `opacity`.
- Bound `filter`, `backdrop-filter`, and `box-shadow` paint areas. An unbounded blur on a 100vh element drops frames.
- Casual layout-property animation that visibly drops frames is a P1.

**Render**:

- React: memoize expensive components and selectors. Avoid creating new object literals in render paths that downstream effects depend on.
- Vue: avoid reactive deep watchers on large objects.
- Svelte: avoid running heavy logic inside reactive statements that fire on every keystroke.
- Avoid layout thrashing (read-then-write-then-read of layout properties in a loop).

**Loading**:

- Images use `loading="lazy"` for off-screen content.
- Largest hero image preloaded.
- Above-the-fold CSS critical path under 14 KB.
- Web fonts use `font-display: swap` and a preload directive.

**Bundle**:

- No unused dependencies in the diff (importing `lodash` for `_.get` is drift; use optional chaining).
- Code-split routes when feasible.
- Avoid deep barrel imports that drag in unrelated modules.

**Layout shift**:

- Images and embeds carry explicit `width` and `height` attributes (or `aspect-ratio` CSS).
- Fonts swap without size jump where possible (`size-adjust`, fallback metric matching).
- Skeletons match the dimension of the loaded content.

**Network**:

- Critical API calls happen in parallel, not waterfalled.
- Optimistic updates rollback on failure.
- Debounce search inputs (200-400ms). Throttle scroll handlers (50-100ms).

### Severity

- **P0**: site-breaking jank, frame drops below 30fps on standard hardware.
- **P1**: ≥ 100ms response delay on key interactions, layout shift score ≥ 0.1.
- **P2**: 50-100ms response delay, suboptimal lazy-loading.
- **P3**: micro-optimizations (memoization opportunities, redundant re-renders).

### Scoring

Score the dimension 0-4:

- **0** — Severe issues. Layout thrash. Unoptimized everything.
- **1** — Major problems. No lazy loading. Expensive animations.
- **2** — Partial. Some optimization. Gaps remain.
- **3** — Mostly optimized. Minor improvements possible.
- **4** — Fast, lean, well-optimized.

### Exit criterion

Dimension score ≥ 3 and zero P0. Route P1 issues to `impeccable optimize`.

---

## Phase 6 — Resilience & Edge Cases

The interface that only renders the happy path is a demo, not a product.

### Inputs

- Component source.
- The data model (TypeScript types, schema, API contracts).

### Checks

**Text overflow**:

- Long names, descriptions, titles render without breaking layout.
- Single-line ellipsis or multi-line clamp as appropriate (`-webkit-line-clamp`).
- Word-wrap and overflow-wrap allow break on overflow.
- Flex and grid items have `min-width: 0` to allow shrinking.

**Empty states**:

- No items in list — provide helpful empty state with next action.
- No search results — explain what was searched, suggest alternatives.
- No data on load — distinguish from loading state.

**Error states**:

- Network failure shows clear error and a retry button.
- 4xx and 5xx each have a distinct treatment.
- Validation errors render near the input, preserve user input.
- Error messages are specific and actionable (not "Error occurred").

**Loading states**:

- Skeleton screens for primary content.
- Inline spinners for actions (buttons disable while pending).
- Time estimates for long operations (> 3 seconds).

**i18n**:

- Text containers expand to fit translated content. Avoid fixed widths.
- Budget 30-40% extra space for the longest language (often German).
- Logical CSS properties: `margin-inline-start`, not `margin-left`.
- RTL layout reverses correctly. Icons that imply direction flip via `[dir="rtl"]`.
- Date and number formatting through `Intl.DateTimeFormat` and `Intl.NumberFormat`.
- Pluralization handled by the i18n library, not template literals.

**Concurrency**:

- Double-submit prevented (button disabled while pending).
- Race conditions handled (request `id` correlation, abort prior requests).
- Optimistic updates rollback on failure.

**Permission states**:

- No permission to view shows the right empty state.
- Read-only mode is visually distinct from edit mode.

**Browser compatibility**:

- Modern features have polyfills or graceful fallbacks.
- Feature detection, not browser detection.

### Severity

- **P0**: empty state, error state, or loading state missing for a critical flow.
- **P1**: long text breaks layout. RTL collapse. i18n widths fixed.
- **P2**: minor edge cases (single-character names render weirdly).
- **P3**: AAA polish (perfect-data tests pass with extreme inputs).

### Scoring

Combined into the Resilience portion of the final report, not a separate /4 dimension. Each P0 docks the composite score by 1.

### Exit criterion

Zero P0. Route P1 issues to `impeccable harden`.

---

## Phase 7 — Editorial & Copy

Copy is part of the interface. AI flavor in copy taints the design no matter how well the visuals score.

### Inputs

- All user-facing strings. JSX text nodes. `placeholder`, `alt`, `title`, `aria-label`. Markdown content. README sections that ship to users.

### Checks (denylist — see [`COPY-DENYLIST.md`](./COPY-DENYLIST.md) for the full list)

**Stolen-engineer diction** (banned outright):

`load-bearing`, `highest-leverage`, `biggest unlock`.

**Internal jargon** (banned outright):

`reflex defaults`, `collapses into monoculture`, `data-driven`.

**Marketing voice** (banned outright):

`seamless`, `seamlessly`, `robust`, `robustness`, `elevate`, `elevates`, `empower`, `empowers`, `underscore`, `underscores`, `pivotal`, `tapestry`.

**AI tells** (banned outright):

`delve`, `delves`, `delved`, `delving`.

**Throat-clearing** (banned outright):

`in today's …`, `gone are the days`, `whether you're …`, `let's dive in`.

**Closers** (banned outright):

`in summary`, `in conclusion`.

**Transitions** (banned outright):

`moreover`, `furthermore`.

**Punctuation**:

- Em dash (`—`) and HTML entities `&mdash;`, `&#8212;`, `&#x2014;`. Banned. Use comma, colon, semicolon, period, parentheses.
- Double-hyphen as em-dash substitute (` -- `). Banned.

### Structural patterns (require human judgment)

- **Negation pivot**: "It's not just X, it's Y." Use sparingly, replace most with a direct positive claim.
- **Triadic everything**: every list of three, every "fast, simple, and powerful". Vary the count.
- **Five-paragraph essay shape**: intro / 3 sections / conclusion on every page. Mix it up.
- **Uniform paragraph length**: insert a 4-word sentence; insert a one-line paragraph.
- **Synthetic balance**: pros and cons of equal length when one is right.
- **Hollow confidence**: "Powerful" without numbers. Replace with a concrete fact.
- **Hedging stacks**: "It might potentially be useful to consider …".
- **Interchangeable copy**: swap the product name for a competitor; if nothing becomes false, the copy is generic.

### Severity

- **P0**: a banned term in production marketing copy or in a hero string.
- **P1**: a banned term elsewhere in user-facing content.
- **P2**: structural pattern (triadic auto-pilot, uniform rhythm).
- **P3**: tonal nudge (could be tightened, not wrong).

### Exit criterion

Zero P0. ≤ 3 P1. Route P1+ to `impeccable clarify`.

---

## Phase 8 — Cross-Stack Verification

The right pattern in the wrong stack is the wrong pattern. Phase 8 verifies stack-correct usage.

### Inputs

- The stack identifier from Phase 0.
- The component source.

### Checks per stack

**React / Next.js**:

- No `dangerouslySetInnerHTML` with user-controlled content (Phase 1 also catches this; Phase 8 catches the design-time anti-pattern).
- Server components vs client components classified correctly. Don't ship a `useState` hook in a server component.
- `next/image` for images. `next/font` for fonts. `next/link` for client-side navigation.
- Route segments respect the routing convention.
- No layout effects in components rendered as Server Components.

**Vue / Nuxt**:

- No `v-html` with user-controlled content.
- `<NuxtLink>` for routing.
- Composables prefixed `use*`.
- Reactivity primitives chosen correctly (`ref` vs `reactive` vs `shallowRef`).

**Astro**:

- Page islands hydrated via `client:*` directives only when needed.
- Image component used for content images.
- Frontmatter imports resolved server-side.
- Content collections typed via the schema.

**Svelte / SvelteKit**:

- `{@html ...}` only with sanitized content.
- Stores derived correctly. No store leaks.
- `+page.server.ts` for server-only logic.

**SwiftUI**:

- No layout in `body` that depends on state read inside `body` (causes infinite layout pass).
- `@StateObject` vs `@ObservedObject` chosen correctly.
- Dynamic Type respected (`.dynamicTypeSize` modifiers).
- `Accessibility*` modifiers applied to custom controls.

**React Native**:

- Layout uses Flexbox. No fixed widths that don't reflow.
- `accessibilityLabel`, `accessibilityRole`, `accessibilityHint` on custom touchables.
- `Pressable` instead of `TouchableOpacity` for new code.
- Lists use `FlatList` or `SectionList`, not `ScrollView` of mapped items.

**Flutter**:

- `const` constructors used where possible.
- `Semantics` widgets on custom interactive elements.
- `MediaQuery.textScaleFactor` respected.
- `ThemeData` carries dark and light variants.

**HTML + Tailwind**:

- Class strings under 80 chars, otherwise extract to a component or `@apply`.
- Arbitrary values (`[#3a3a3a]`) only when no token fits. Otherwise use the design-system token.
- Custom CSS via `@layer components`, not unscoped overrides.

**shadcn/ui**:

- Components imported from `@/components/ui/*`, customized via the slot props the primitive exposes.
- Theme variables in `:root` and `.dark`. No hard-coded HSL channels in the diff.
- `cn()` helper used for className merging, not string concatenation.

**Angular**:

- No template injection via `[innerHTML]` with user content.
- `OnPush` change detection where feasible.
- Reactive forms preferred over template-driven for non-trivial forms.

**Laravel** (Blade, Livewire, Inertia):

- `{{ }}` for escaped output. `{!! !!}` only with explicitly sanitized content.
- Livewire components don't expose unsigned URLs of properties.
- CSRF tokens present on forms.

### Severity

- **P0**: stack-specific security or correctness issue (`dangerouslySetInnerHTML` with user content, missing CSRF token).
- **P1**: stack-specific anti-pattern (Server Component using `useState`, `ScrollView` of long list).
- **P2**: stack-specific drift (custom CSS overriding shadcn, manual className concatenation).
- **P3**: idiomatic improvement.

### Exit criterion

Zero P0. ≤ 3 P1.

---

## Phase 9 — Sign-Off

Composite the prior phases into a single verdict. The verdict drives the next action.

### Inputs

All prior phase outputs.

### Composite

**Audit Health Score** (out of 20):

- Anti-Pattern (Phase 2): 0-4
- Design System (Phase 3): 0-4
- Accessibility (Phase 4): 0-4
- Performance (Phase 5): 0-4
- Theming (sub-component of Phase 3): 0-4

**Bands**:

- 18-20: Excellent. Minor polish only.
- 14-17: Good. Address the weak dimensions before shipping.
- 10-13: Acceptable. Significant work required.
- 6-9: Poor. Major overhaul.
- 0-5: Critical. Fundamental issues.

**Severity census** (across all phases): count of P0, P1, P2, P3.

**Security verdict**: pass or fail. Any HIGH finding fails. Any MEDIUM with confidence ≥ 0.85 fails.

### Verdict logic

- **Ready to ship**: Excellent or Good band, Security passes, zero P0, ≤ 5 P1.
- **Ship with exception**: Acceptable band, Security passes, zero P0, exception documented.
- **Hold**: Poor or Critical band, OR Security fails, OR any P0.

### Output

A single markdown report. See [`RUBRIC.md`](./RUBRIC.md) for the report template.

The report ends with a numbered list of recommended `impeccable` sub-commands (or other actions) to run, in priority order. P0 items first.

### Exit criterion

The user reads the verdict and decides. No automatic ship.

---

## Phase ordering rationale

Why this order:

1. Discovery before judgment. Without context, every check is generic.
2. Security before everything else. A leaking app is unshippable regardless of polish.
3. Anti-Patterns before Design System. AI slop is louder than drift; fix the loudness first so drift becomes visible.
4. Design System before Accessibility. Tokens often carry contrast guarantees; fixing tokens fixes a11y in bulk.
5. Accessibility before Performance. A fast, inaccessible page is still inaccessible.
6. Performance before Resilience. Performance is observable on the happy path. Resilience tests degrade.
7. Resilience before Editorial. Working features first, polished prose second.
8. Editorial before Stack. Copy lives in components; verifying copy doesn't depend on stack idioms.
9. Stack before Sign-Off. Stack issues might be the last surprise; catch them right before the verdict.

## Skipping phases

Phases 1, 2, 3, 4 are mandatory. Skipping any of them invalidates the run.

Phases 5, 6, 7, 8 are skippable when the diff doesn't touch their domain:

- Skip Phase 5 if the diff has no animation, no large assets, no render-frequency changes.
- Skip Phase 6 if the diff is text-only or config-only.
- Skip Phase 7 if the diff has no user-facing strings.
- Skip Phase 8 if the diff is in a stack-agnostic file (CSS-only, JSON config, doc).

The pipeline records the skip reason. A skipped phase still appears in the final report, marked Skipped with rationale.

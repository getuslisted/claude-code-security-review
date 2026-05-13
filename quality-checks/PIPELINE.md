# Pipeline

Nine phases. Each phase has explicit inputs, checks, outputs, exit criteria.

Version: **v1.1.0** (2026-05-13).

Changelog from v1.0:
- Fixed Rubric dimension double-count (Theming folded into Design System, Resilience promoted).
- `PRODUCT.md` no longer hard-gates the pipeline.
- Removed undefined `target=<paste>` mode.
- Phase 1 composes with `/security-review` when available.
- Added deterministic gate (`scripts/check.sh`).

---

## Phase 0 — Discovery & Context

### Inputs

- Branch diff.
- Repo root scanned for `PRODUCT.md`, `DESIGN.md`, `design-system/MASTER.md`, `package.json`, framework configs.

### Checks

1. **Register**. Task cue, surface in focus, then `register` field in `PRODUCT.md`.
2. **Design system map**. Walk for tokens.
3. **Stack**. Identify from configs.
4. **Industry**. When `ui-ux-pro-max-skill`'s `search.py` is present, invoke it. Otherwise infer from `PRODUCT.md`.
5. **Anti-references**. Pull from `PRODUCT.md`.

### Graceful degradation

- `PRODUCT.md` present and non-trivial: full pipeline.
- `PRODUCT.md` present but trivial: pipeline runs; register inferred from cue; report nudges.
- `PRODUCT.md` missing: pipeline runs every phase EXCEPT register-specific and industry-specific checks. Report nudges.

Never halts at Phase 0 in v1.1.

---

## Phase 1 — Security Lockdown

**Preferred** (in `claude-code-security-review`): invoke `/security-review` directly.

**Fallback**: inline prompt below.

### Sub-phases

**1A** Repository Context Research.
**1B** Comparative Analysis.
**1C** Vulnerability Assessment across Input Validation, AuthN / AuthZ, Crypto & Secrets, Injection & Code Execution, Data Exposure.

### False-positive filter

Do not report: DoS, on-disk secrets if otherwise secured, rate-limiting, memory / CPU exhaustion, generic input-validation gaps without proven impact, GitHub Action workflow issues without concrete trigger, general lack of hardening, theoretical race conditions, outdated libraries, memory safety in memory-safe languages, test files, log spoofing from un-sanitized input, path-only SSRF, user content in AI prompts, regex injection / regex DoS, doc files, lack of audit logs.

### Confidence threshold

8-10: report. 4-7: report only with concrete attack path. 1-3: drop.

### Exit

Zero HIGH. Zero MEDIUM ≥ 0.85 confidence.

---

## Phase 2 — Anti-Pattern Audit

### Deterministic subset

`scripts/check.sh` catches `qc-001` (side-stripe), `qc-002` (gradient text), `qc-007` (pure black/white), `qc-009` (layout-property animation).

### LLM-judgment subset

`qc-003` (glassmorphism default), `qc-004` (hero-metric template), `qc-005` (identical card grids), `qc-006` (modal as first thought), `qc-008` (bounce easing), `qc-010` (nested cards). Plus first-order and second-order category reflexes (`qc-020`, `qc-021`) and industry-specific (`qc-100`+).

### Scoring

0 — AI slop gallery. 4 — No AI tells.

### Exit

Score ≥ 3, zero absolute-ban hits.

---

## Phase 3 — Design System Conformance

### Checks

**Color**: tokens only. OKLCH for new colors. No `#000` / `#fff` (deterministic). No gray on color.

**Typography**: hierarchy contrast ≥ 1.25 between steps. Body 65-75ch. Headings `clamp()`, body fixed `rem`.

**Spacing**: scale-only.

**Radius**: controlled vocabulary.

**Shadow**: flat at rest. Blur ≤ 0.15 alpha. Tinted only for accent-glow.

**Motion**: durations from scale. Ease-out exponential family. Honour `prefers-reduced-motion`.

**Component reuse**: shared primitives, not one-off reimplementations.

### Drift classification

Missing token / one-off implementation / conceptual misalignment.

### Scoring

0 — Hard-coded everything. 4 — Full token system, dark mode works.

### Exit

Score ≥ 3, zero P0 drift, ≤ 3 P1 drift.

---

## Phase 4 — Accessibility Hardening

### Deterministic subset

`qc-042` (`outline: none`), `qc-043` (`<div onClick>`).

### LLM-judgment subset

Contrast ≥ 4.5:1. Semantic HTML. ARIA names. Keyboard reachable. Touch targets ≥ 44 × 44 px. Form labels. `prefers-reduced-motion`. Color independence.

### Scoring

0 — Inaccessible. 4 — WCAG AA fully met.

### Exit

Score ≥ 3, zero P0.

---

## Phase 5 — Performance Audit

Animation: `transform` and `opacity` only. Render: memoize. Loading: lazy. Bundle: no unused deps. Layout shift: explicit dimensions. Network: parallel, debounced, throttled.

### Scoring

0 — Severe issues. 4 — Fast, lean.

### Exit

Score ≥ 3, zero P0.

---

## Phase 6 — Resilience & Edge Cases

Text overflow: clamp / ellipsis / wrap. Empty states. Error states (4xx, 5xx distinct). Loading states. i18n (30-40% expansion, logical CSS, RTL, `Intl.*`). Concurrency. Permission states.

### Scoring

0 — Happy-path only. 4 — Hardened for long text, RTL, errors, offline.

(v1.1 promotes Resilience to its own dimension. Replaces v1.0's Theming.)

### Exit

Zero P0.

---

## Phase 7 — Editorial & Copy

### Deterministic subset

`qc-070` em dashes, `qc-071` banned diction, `qc-072` throat-clearing, banned closers and transitions.

### LLM-judgment subset

Negation pivot. Triadic everything. Five-paragraph essay shape. Uniform paragraph length. Synthetic balance. Hollow confidence. Hedging stacks. Interchangeable copy.

### Exit

Zero P0. ≤ 3 P1.

---

## Phase 8 — Cross-Stack Verification

### Deterministic subset

`qc-080` `dangerouslySetInnerHTML`, `qc-081` `v-html` / Svelte `{@html}`. Always P0.

### LLM-judgment subset

React / Next.js, Vue / Nuxt, Astro, Svelte, SwiftUI, React Native, Flutter, HTML + Tailwind, shadcn/ui, Angular, Laravel, Jetpack Compose.

### Exit

Zero P0. ≤ 3 P1.

---

## Phase 9 — Sign-Off

### Audit Health Score (v1.1)

Five dimensions, 0-4 each. Total /20:

- Anti-Pattern (Phase 2)
- Design System (Phase 3)
- Accessibility (Phase 4)
- Performance (Phase 5)
- Resilience (Phase 6)

### Bands

18-20 Excellent. 14-17 Good. 10-13 Acceptable. 6-9 Poor. 0-5 Critical.

### Verdict

- **Ready to ship**: Excellent or Good, Security pass, deterministic gate pass, zero P0, ≤ 5 P1.
- **Ship with exception**: Acceptable, Security pass, deterministic gate pass, zero P0, exception documented.
- **Hold**: Poor or Critical, OR Security fail, OR deterministic gate fail, OR any P0.

### Output

Markdown report per [`RUBRIC.md`](./RUBRIC.md).

---

## Deterministic gate timing

`scripts/check.sh` runs BEFORE Phase 1 in two modes:

1. **CI mode**: from `.github/workflows/quality-checks.yml`. Exit code drives PR comment.
2. **Slash command mode**: from `commands/quality-checks.md` before LLM judgment. Findings feed phase scores.

Idempotent. Both modes share the same script.

## Skipping phases

Phases 1, 2, 3, 4 are mandatory. Phases 5, 6, 7, 8 are skippable when the diff doesn't touch their domain.

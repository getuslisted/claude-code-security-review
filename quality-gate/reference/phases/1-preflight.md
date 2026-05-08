# Phase 1 — Preflight

Establish the context the rest of the gate runs in. Skipping this produces generic findings that ignore the project's register, stack, and intent.

## Why this exists

The other seven phases all assume answers to questions phase 1 asks. A "this color isn't accessible" finding means nothing if you don't know whether the surface is brand (where rare contrast tradeoffs are deliberate) or product (where they're never acceptable). A "this query is vulnerable to injection" finding needs the framework context to know whether the framework auto-parameterizes.

## Inputs

- `PRODUCT.md` — strategic intent, register, anti-references
- `DESIGN.md` — color tokens, typography scale, motion conventions
- `package.json` / `Cargo.toml` / `pyproject.toml` — stack signal
- The diff or named target — what's actually changed

## Checks

### 1.1 Context files exist and are non-trivial

Read PRODUCT.md and DESIGN.md from the project root, falling back to `.agents/context/` and `docs/`. Override with `IMPECCABLE_CONTEXT_DIR`.

| Condition | Verdict |
|---|---|
| PRODUCT.md missing or empty | **P0**. Block. Run `/impeccable teach`, then resume. |
| PRODUCT.md contains `[TODO]` markers or is under 200 chars | **P0**. Block. Same fix. |
| DESIGN.md missing | **P2**. Continue with a single-session nudge: "DESIGN.md is missing. Run `/impeccable document` for on-brand output." |

Never synthesize PRODUCT.md from the user's prompt alone. The gate is precisely the wrong moment to invent strategic context.

### 1.2 Register is determined

Every UI surface is one of two registers:

- **Brand** — design IS the product. Marketing pages, landing pages, brand sites, campaign surfaces, portfolios, long-form content. Distinctiveness is the bar.
- **Product** — design SERVES the product. App UI, admin, dashboards, tools. Earned familiarity is the bar.

Resolution order, first match wins:

1. Cue in the task itself ("landing page" → brand, "dashboard" → product).
2. The surface in scope (`pages/marketing/*` → brand, `pages/app/*` → product).
3. The `register` field in PRODUCT.md.
4. Inferred from PRODUCT.md's "Users" and "Product Purpose" sections, cached for the session.

Record the chosen register in the preflight line. Phases 3, 4, and 8 read it.

### 1.3 Stack is identified

Read the manifest. Record:

- **Framework** — react, vue, svelte, astro, next, nuxt, sveltekit, solid, qwik, remix, plain HTML
- **Language** — typescript, javascript, python, rust, go, ruby, php
- **Styling** — tailwind, css-modules, vanilla-extract, styled-components, emotion, plain CSS, sass, less
- **Component library** — shadcn, radix, headless-ui, ariakit, mantine, chakra, mui, antd, none

Stack signal changes the rules:

| Stack signal | Affects |
|---|---|
| React or Angular | Phase 2: don't report XSS unless `dangerouslySetInnerHTML` / `bypassSecurityTrustHtml` is used. |
| Rust | Phase 2: drop memory-safety findings. |
| Tailwind | Phase 4: token enforcement reads `tailwind.config` for the source of truth. |
| shadcn/Radix | Phase 5: keyboard interaction and ARIA patterns are guaranteed by the primitive. Verify usage, not implementation. |
| Plain CSS | Phase 7: design tokens live as CSS variables; check for hard-coded values that should reference them. |

### 1.4 Diff is bounded

Get the changed file list:

```bash
# For PR review
git diff --name-only --merge-base origin/HEAD

# For a target name
# Resolve the target to a file glob: component name → src/components/<Name>.*; route → pages/**/<route>.*
```

Record the list. Every later phase scopes to it. Don't audit unchanged files; that's drift, and it's noise.

If the diff is empty and no target is named, ask the user what to scope to. Don't audit the whole repo unless explicitly asked.

### 1.5 Build state is healthy

Before running phases 2–8, confirm the basics:

| Check | Command | If fail |
|---|---|---|
| Type check passes | `tsc --noEmit` (or framework equivalent) | P0. Fix before gating. |
| Tests pass | `bun test` / `pnpm test` / `pytest` | P0. Fix before gating. |
| Linter passes | `bun lint` / `pnpm lint` / `ruff check` | P1. Note in report. |
| Build succeeds | `bun run build` (or framework equivalent) | P0. Fix before gating. |

The gate isn't a substitute for these. If they fail, the gate hasn't run; record the failures and stop.

## Output

A single block, written before any phase 2 work:

```text
QUALITY_GATE_PREFLIGHT:
  context_dir: <resolved path>
  product_md: pass
  design_md: <pass|nudge>
  register: <brand|product>
  stack: <framework>/<lang>/<styling>/<component-lib>
  target: <diff sha range | named target>
  files_in_scope: <count>
  build_health: typecheck=<pass|fail> tests=<pass|fail> lint=<pass|fail> build=<pass|fail>
```

If any field is `fail`, stop. The gate cannot run on a broken foundation.

## Common mistakes

- **Synthesizing PRODUCT.md from the prompt.** The whole point of the gate is independent verification. Inventing the strategic context defeats it.
- **Inferring register from one keyword.** "Dashboard" can be brand (a design tool's marketing site demo) or product (an actual operations dashboard). Read the surface; don't trust the noun.
- **Skipping the build-health gate.** Running quality checks on broken code is noise. Findings that disappear on the next compile aren't findings.
- **Auditing the whole repo by reflex.** Scope to the diff. The gate is a pre-ship pass, not a refactor pass.

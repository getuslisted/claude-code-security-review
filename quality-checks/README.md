# Quality Checks v1.1

The final line of defense for code that ships. A nine-phase pipeline composing security analysis (`claude-code-security-review`), design rigour (`impeccable`), and industry-specific UI intelligence (`ui-ux-pro-max-skill`) into one pass.

**v1.1 makes the pipeline actually executable.** The deterministic subset runs as a shell script in CI; the subjective subset runs as a Claude Code slash command. Tests are bundled. The dimension double-count in v1.0's rubric is fixed.

## How to use it

### 1. Deterministic gate (CI-friendly)

```bash
quality-checks/scripts/check.sh
```

Exit 0: pass. Exit 1: P0 or P1 finding. 11 rules covered:

| ID | Rule | Severity |
|----|------|----------|
| qc-001 | Side-stripe border > 1px | P1 |
| qc-002 | Gradient text (`background-clip: text` + gradient) | P1 |
| qc-007 | Pure `#000` / `#fff` | P2 |
| qc-009 | Layout-property animation | P1 |
| qc-042 | `outline: none` directive | P0 |
| qc-043 | `<div onClick>` | P0 |
| qc-070 | Em dash in user copy | P1 |
| qc-071 | Banned diction (`delve`, `seamless`, `robust`, etc.) | P1 |
| qc-072 | Throat-clearing openers | P2 |
| qc-080 | `dangerouslySetInnerHTML` | P0 |
| qc-081 | `v-html`, Svelte `{@html}` | P0 |

Self-test (must pass on install):

```bash
quality-checks/tests/run-tests.sh
# 13 passed, 0 failed
```

### 2. Slash command (LLM judgment)

```
/quality-checks                        # all nine phases
/quality-checks phase=2                # one phase
/quality-checks phase=1,3,4            # multiple
/quality-checks target=src/Hero.tsx    # scope to a path
```

Loads the references, runs the deterministic gate as pre-flight, then executes the subjective phases. Emits one markdown report per [`RUBRIC.md`](./RUBRIC.md)'s template.

## Pipeline at a glance

| Phase | Name | Source | Hard gate |
|-------|------|--------|-----------|
| 0 | Discovery & Context | impeccable + ui-ux-pro-max | required |
| 1 | Security Lockdown | claude-code-security-review (composed) | yes |
| 2 | Anti-Pattern Audit | impeccable + ui-ux-pro-max | yes |
| 3 | Design System | impeccable + ui-ux-pro-max | yes |
| 4 | Accessibility | WCAG AA floor | yes |
| 5 | Performance | impeccable optimize | no |
| 6 | Resilience | impeccable harden | no |
| 7 | Editorial & Copy | impeccable STYLE.md | no |
| 8 | Cross-Stack | ui-ux-pro-max stacks | no |
| 9 | Sign-Off | composite | n/a |

Full definitions in [`PIPELINE.md`](./PIPELINE.md). Checklist in [`CHECKLIST.md`](./CHECKLIST.md). Scoring in [`RUBRIC.md`](./RUBRIC.md).

## v1.1 changes

- **Deterministic gate**: new `scripts/check.sh` runs 11 regex-detectable rules. Self-tested against 13 fixtures.
- **CI integration**: `.github/workflows/quality-checks.yml` runs on PRs, posts a comment.
- **Rubric fix**: v1.0 listed Theming as both a sub-component of Design System and a separate dimension. v1.1 folds Theming into Design System, promotes Resilience to its own dimension.
- **Graceful degradation**: `PRODUCT.md` no longer hard-gates the pipeline.
- **Phase 1 composes**: when running inside `claude-code-security-review`, invokes `/security-review` directly.
- **Removed `target=<paste>`**.
- **Bundled manifest**: `manifest.json` declares version + source references.

## What you can paste in

[`BLOCKS.md`](./BLOCKS.md) ships pre-verified UI primitives: header, hero, card, button, form, footer, empty / loading / error states.

## Installation

See [`INSTALL.md`](./INSTALL.md). Three steps: copy, self-test, run.

## Source repos

- `claude-code-security-review` — three-phase security analysis with false-positive filtering.
- `impeccable` — 23-command design skill: shared design laws, registers, anti-pattern catalog, editorial denylist.
- `ui-ux-pro-max-skill` — 161 industry rules, 67 styles, 161 palettes, 57 font pairings, 99 UX guidelines, 15 stack guides.

## Versioning

v1.1.0 (2026-05-13). Bundled `manifest.json` declares the version and source-repo references.

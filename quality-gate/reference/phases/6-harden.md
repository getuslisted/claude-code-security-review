# Phase 6 — Hardening

Designs that only work with perfect data aren't production-ready. Synthesized from impeccable's harden reference. Five dimensions: text overflow, internationalization, error handling, edge cases, and network resilience.

## Why this phase exists

Real users will:
- Type 200 character names.
- Use a language where every word is 30% longer than English.
- Lose connection mid-submit.
- Have permissions revoked between page loads.
- See an empty state on day one with no data, never having seen a populated state.

The build phase usually tests the happy path. This phase tests reality.

## 6.1 Text overflow & wrapping

Every text container in the diff needs a behavior for content longer than designed.

### Detection patterns

| Pattern | Severity | Fix |
|---|---|---|
| Fixed `width` on a text container with no overflow handling | P1 | Add `overflow: hidden; text-overflow: ellipsis; white-space: nowrap;` for single-line; `display: -webkit-box; -webkit-line-clamp: N; -webkit-box-orient: vertical; overflow: hidden;` for multi-line. |
| Flex item showing text without `min-width: 0` | P1 | Flex items default to `min-width: auto` (content size); long content forces overflow. Add `min-width: 0`. |
| Grid item with long text, no `min-width: 0` | P1 | Same root cause as flex. |
| Email or URL display without `overflow-wrap: break-word` or `word-break: break-all` | P2 | Long URLs wrap onto themselves. |
| Heading without overflow handling at narrow breakpoints | P2 | Long product names break the layout on mobile. |

### Test inputs

For every text field in the diff, manually substitute:

- A 200-character lorem ipsum string.
- A single word with 50 characters (URL or hash).
- Three or four CJK characters (different glyph metrics from Latin).
- A string with 5+ emoji (2–4 byte glyphs).
- A 10-line paragraph in a single-line container.

Each substitution should render correctly. If it doesn't, the container is a finding.

## 6.2 Internationalization

### Text expansion

Allow 30–40% more space than the English string requires. German is the canonical worst case (`Settings` → `Einstellungen`, 8 → 13 chars). Some Slavic languages can run 50%+.

| Pattern | Severity |
|---|---|
| Fixed-width button or label sized to fit the English string exactly | P1 |
| Truncation that hides UI primary action label | P1 |
| Two-line button label allowed in Latin scripts but not in others | P2 |

### RTL support

If the project ships in any RTL language (Arabic, Hebrew, Persian, Urdu):

| Pattern | Severity |
|---|---|
| `margin-left` / `margin-right` instead of `margin-inline-start` / `margin-inline-end` | P1 |
| `padding-left` / `padding-right` instead of `padding-inline-*` | P1 |
| Directional icons (arrows, chevrons) without `[dir="rtl"]` mirror | P1 |
| `text-align: left` instead of `text-align: start` | P2 |

### Date, number, currency

| Pattern | Severity |
|---|---|
| `Date.toLocaleDateString()` without explicit locale | P2 |
| Currency formatted by string concatenation (`"$" + amount`) | P1 |
| Number formatted by `.toFixed()` only (no thousands separator, wrong for many locales) | P2 |
| Plurals via `count !== 1 ? 's' : ''` (English-only) | P2 |

Use `Intl.DateTimeFormat`, `Intl.NumberFormat`, `Intl.PluralRules`, and the project's i18n library (i18next, react-intl, vue-i18n) for plurals.

## 6.3 Error handling

Every async action and every external call needs an error path.

### Network errors

For every `fetch`, `axios`, or framework data-loader call in the diff:

| Pattern | Severity |
|---|---|
| No `.catch` / try/catch around an async call | **P0** |
| Catch that swallows the error silently | P1 |
| Error UI that says only "Error occurred" | P1 |
| No retry option on a transient failure | P2 |
| No timeout set on a long-running request | P2 |

### API status codes

| Status | Required UX |
|---|---|
| 400 | Show validation errors inline. Preserve user input. |
| 401 | Redirect to login. Preserve return URL. |
| 403 | Show a permission error with what the user can do (request access, contact admin). |
| 404 | Show a not-found state with primary action to go back. |
| 429 | Show rate limit message with retry-after time. |
| 500 | Show generic error with support contact. Don't leak the stack trace. |

| Pattern | Severity |
|---|---|
| Single error path for all status codes | P1 |
| Stack trace shown in production UI | **P0** (data exposure) |

### Validation errors

| Pattern | Severity |
|---|---|
| Submit blocked but no inline error message | P1 |
| Error message not adjacent to the field | P1 |
| User input lost on error | **P0** (data loss; a real user-rage moment) |
| Generic "Please correct the errors" without specifying which | P1 |

### Optimistic updates

When the diff introduces an optimistic update:

| Pattern | Severity |
|---|---|
| Optimistic update without rollback on failure | **P0** (data appears saved but isn't) |
| No conflict resolution when concurrent edits happen | P1 |
| Disabling the form during the request leaves the UI in limbo if the request hangs | P2 |

## 6.4 Edge cases

### Empty states

Every list, table, grid, or feed needs an empty state that explains:

1. Why is it empty? (no data yet vs. filtered to zero vs. permission)
2. What's the next action? (create one, clear filter, request access)

| Pattern | Severity |
|---|---|
| List component without an empty state | P1 |
| Empty state that's just blank space | P1 |
| Empty state without a primary action | P2 |
| Empty state used when the user filtered to zero (should show a "no results" state) | P2 |

### Loading states

| Pattern | Severity |
|---|---|
| Async action without any loading indicator | P1 |
| Loading state that looks identical to the empty state | P2 |
| Skeleton that mismatches the loaded layout (causes CLS) | P1 |
| Loading on every interaction, including locally cached data | P2 |

### Permission states

| Pattern | Severity |
|---|---|
| UI that shows actions the user can't perform (instead of hiding or disabling them) | P1 |
| Disabled action without a tooltip explaining why | P2 |
| Read-only mode that looks identical to edit mode | P1 |

### Concurrent operations

| Pattern | Severity |
|---|---|
| Submit button doesn't disable on click (allows double-submit) | P1 |
| Race condition between rapid clicks (e.g. delete then undo) without proper state management | P1 |

### Large datasets

| Pattern | Severity |
|---|---|
| `<List>` rendering 1000+ items without virtualization | P1 |
| No pagination on an unbounded server-side query | P1 |
| Search/filter without debouncing | P2 |

### Browser compatibility

| Pattern | Severity |
|---|---|
| CSS feature without fallback (no `@supports` for `subgrid`, `:has()`, container queries) | P2 |
| Browser-detection sniffing (use feature detection) | P2 |
| Polyfill missing for a feature the project's browserslist requires | P1 |

## 6.5 Network resilience

| Pattern | Severity |
|---|---|
| No offline state for a tool the user might use offline | P2 |
| No service worker for a PWA-claimed app | P1 |
| No skeleton or progress indicator on slow 3G | P2 |
| No request cancellation on component unmount (memory leak) | P2 |
| Image without `loading="lazy"` below the fold (covered also in phase 5) | P2 |

## Output format

```markdown
## Phase 6 — Harden

| Dimension | Score | Worst finding |
|---|---|---|
| 6.1 Text overflow | <0-4> | |
| 6.2 i18n | <0-4> | |
| 6.3 Error handling | <0-4> | |
| 6.4 Edge cases | <0-4> | |
| 6.5 Network resilience | <0-4> | |
| **Total** | **<0-20>** | |

### Findings
[entries with file:line, severity, evidence, fix, confidence]

### Edge case test matrix
- Long text (200 chars): <pass | fail location>
- CJK characters: <pass | fail location>
- Emoji (5+): <pass | fail location>
- RTL (if applicable): <pass | fail location | n/a>
- Empty state: <pass | fail location>
- Loading state: <pass | fail location>
- Network error: <pass | fail location>
- Permission denied: <pass | fail location>
- Slow connection: <pass | fail location>
```

## Block decision

| Total | Decision |
|---|---|
| 0–9 | **P0**. Major hardening gaps. |
| 10–13 | **P1**. Significant gaps. |
| 14–17 | **P2**. Continue. |
| 18–20 | Pass. |

Plus: any individual P0 above blocks regardless.

## Common mistakes

- **Designing only for English.** Even if the v1 ships in English, the components will be reused across surfaces and eventually localized. Build with `padding-inline` from the start; retrofitting is harder than starting right.
- **Skeleton that doesn't match the real layout.** A 4-row skeleton followed by a 5-row table is CLS. Match exactly.
- **Empty states that look like errors.** "No projects yet" with a smiling icon is empty; "We couldn't load your projects" with a warning icon is an error. Don't conflate.
- **Toast notifications for inline errors.** Form errors belong next to the field. Toasts are for global state changes (saved, deleted, undone).
- **Disabling submit forever after one failure.** Users want to retry. Disable for the duration of the request, not after a failure.

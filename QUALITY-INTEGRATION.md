# Quality integration

This repo (`claude-code-security-review`) is **Phase 1** of the unified `.quality/` pipeline at [`getuslisted-website-v3/.quality/`](https://github.com/getuslisted/getuslisted-website-v3/tree/claude/code-review-quality-checks-1deaY/.quality). When a PR touches the website, the pipeline runs a fast deterministic subset of these security rules locally; the full LLM-driven review still runs as a GitHub Action on the PR.

## What's mirrored, what's not (v1.2 manifest)

| Capability | Lives here | Mirrored in `.quality/` runner | Why |
| --- | --- | --- | --- |
| Full LLM diff review with Claude | [`claudecode/`](claudecode/) + [`.claude/commands/security-review.md`](.claude/commands/security-review.md) | No | Requires API key + diff context. Stays as a workflow. |
| `innerHTML =` with template literal | LLM finding | Yes - `sec.innerhtml-template` (P1) | Cheap, deterministic, single-quote-safe regex. |
| `eval(...)` / `new Function(...)` | LLM finding | Yes - `sec.eval` (P0) | |
| `document.write` / `document.writeln` | LLM finding | Yes - `sec.document-write` (P0) | |
| React `dangerouslySetInnerHTML`, Angular `bypassSecurityTrustHtml` | LLM finding | Yes - `sec.dangerously-set-inner-html` (P0) | |
| `<iframe srcdoc=>` | LLM finding | Yes - `sec.iframe-srcdoc` (P1) | |
| `javascript:` URL in `href`/`src`/`action` | LLM finding | Yes - `sec.javascript-url` (P0) | |
| `postMessage` listener missing origin check | LLM finding | Yes - `sec.postmessage-no-origin` (P1; uses regex requireSibling for `e.origin`, `event.origin`, etc.) | |
| Hardcoded secrets (Stripe / AWS / GitHub / Slack / JWT) | LLM finding | Yes - `sec.hardcoded-secret` (P0) | Pattern-based; rotation-prompting message. |
| External `<script src>` without SRI | LLM finding | Yes - `sec.script-no-sri` (P2; Google/FB tag scripts allowlisted) | |
| Page without CSP meta or header | LLM finding | Yes - `design.no-csp-meta` (P3; promote to P1 when CSP lands) | |
| `<a target="_blank">` missing `rel="noopener"` | LLM finding | Yes - `a11y.target-blank-no-noopener` (P1) | Tabnabbing. |
| `fetch()` without `AbortController` | LLM finding | Yes - `sec.fetch-no-timeout` (P2; sibling regex matches the import) | |
| SQL / command / template injection | LLM finding | No | Static site; no server-rendered queries. Re-enable if a backend lands. |
| Site-wide watchlist: `links.config.js` GTM/FB Pixel injection | Reviewed manually + hardened | Hardened at source (`isValidGtmId` + `isValidFbPixelId`) AND flagged by `sec.innerhtml-template` | The runner cannot reason about provenance; validation runs before the innerHTML write. |

## Hard exclusions (kept identical to this repo's [`security-review.md`](.claude/commands/security-review.md))

- DoS, rate-limiting, memory/CPU exhaustion.
- Secrets stored on disk (handled separately).
- Lack of permission checking in client-side JS.
- Log spoofing, regex DoS, SSRF that only controls the path.
- React/Angular XSS unless `dangerouslySetInnerHTML` or equivalent is present.

The LLM workflow and the local heuristics must agree on what counts. Otherwise the local run flags what the workflow ignores and the team starts ignoring the local run.

## Operational surfaces

- **Per-phase CI status check**: `quality / security` runs every PR. A failure surfaces as a single red check (no need to open logs).
- **Nightly drift detection**: `.github/workflows/quality-nightly.yml` runs `--strict` at 06:00 UTC and opens a single `quality-drift` issue when totals exceed baseline. The issue updates in place rather than spamming.
- **Admin dashboard**: `.quality/admin/` surfaces total findings, severity / phase breakdown, and a 30-day trend. Phase-1 (security) findings appear in the "By phase" bar.

## Updating the integration

When this repo's prompt or filter list changes:

1. Update [`.claude/commands/security-review.md`](.claude/commands/security-review.md) here.
2. Mirror the change in `getuslisted-website-v3/.quality/PHASES.md` Phase 1.
3. If the change is a deterministic rule (regex): add it to `getuslisted-website-v3/.quality/checks/manifest.json` AND add a fixture under `.quality/tests/fixtures/` AND add a case in `.quality/tests/cases.json`. The runner refuses to load a manifest with a duplicate id or bad regex; the selftest refuses to pass without paired fixtures.
4. If the change is a hard exclusion: drop the corresponding rule from the manifest. Document the exclusion in the LLM prompt's filter list AND in `PHASES.md` so both stay aligned.

Branch convention: `claude/code-review-quality-checks-*` so the two diffs land together.

## Local use during this repo's own development

This repo eats its own dog food via the `/security-review` slash command. The unified `.quality/` pipeline at the website is downstream; nothing in this repo needs to depend on `.quality/`. The integration is one-way: rules originate here, get mirrored as regex over there.

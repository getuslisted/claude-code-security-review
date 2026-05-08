# Quality integration

This repo (`claude-code-security-review`) is **Phase 1** of the unified `.quality/` pipeline that lives in [`getuslisted-website-v3/.quality/`](https://github.com/getuslisted/getuslisted-website-v3/tree/claude/code-review-quality-checks-1deaY/.quality). When a PR touches the website, the pipeline runs a fast subset of these security rules locally; the full LLM-driven review still runs as a GitHub Action on the PR itself.

## What's mirrored, what's not

| Capability | Lives here | Mirrored in `.quality/` runner | Why |
| --- | --- | --- | --- |
| Full LLM diff review with Claude | [`claudecode/`](claudecode/) + [`.claude/commands/security-review.md`](.claude/commands/security-review.md) | No | Requires API key + diff context. Stays as a workflow. |
| `innerHTML = \`...${...}\`` XSS sink | implicit in prompt | Yes — `sec.innerhtml-string` regex | Cheap, deterministic, catches regressions immediately. |
| `eval(...)` / `new Function(...)` | implicit in prompt | Yes — `sec.eval` regex | Same reason. |
| Hardcoded API keys / secrets | LLM finding | No | Handled by GitHub secret scanning + dotenv hygiene; CCSR explicitly excludes secrets-on-disk. |
| `fetch()` without timeout | LLM finding (P2) | Yes — `sec.fetch-no-timeout` heuristic | Surfaces hung-form risk in client JS. |
| SQL / command / template injection | LLM finding | No | Site is static; no server-rendered queries. Re-enable if a backend lands. |
| GTM / FB Pixel injection from config | reviewed manually | Hardened by tokens, audited by Phase 1 of `PHASES.md` | The runner cannot reason about provenance, so a human owns the call. |

## Hard exclusions (kept identical)

The unified runner inherits the same exclusion list this repo's [`security-review.md`](.claude/commands/security-review.md) command defines:

- DoS, rate-limiting, memory/CPU exhaustion.
- Secrets stored on disk (handled separately).
- Lack of permission checking in client-side JS.
- Log spoofing, regex DoS, SSRF that only controls the path.
- React/Angular XSS unless `dangerouslySetInnerHTML` or equivalent is present.

If a new exclusion lands here, mirror it in `.quality/PHASES.md` Phase 1 and the runner's allow logic. The LLM workflow and the local heuristics must agree on what counts as a finding; otherwise the local run will flag what the workflow ignores and the team starts ignoring the local run too.

## Updating the integration

When this repo's prompt or filter list changes:

1. Update [`.claude/commands/security-review.md`](.claude/commands/security-review.md) here.
2. Mirror the change in `getuslisted-website-v3/.quality/PHASES.md` Phase 1.
3. If the change is a new deterministic rule (regex), add it to `getuslisted-website-v3/.quality/checks/manifest.json`.
4. If the change is a new hard exclusion, drop the corresponding rule from the manifest.

The branch convention for these cross-repo updates is `claude/code-review-quality-checks-*` so the two diffs land together.

## Local use during this repo's own development

This repo eats its own dog food via the `/security-review` slash command. The unified `.quality/` pipeline at the website is downstream of that; nothing in this repo needs to depend on `.quality/`.

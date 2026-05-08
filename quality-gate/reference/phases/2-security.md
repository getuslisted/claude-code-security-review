# Phase 2 — Security

A high-confidence vulnerability scan. Synthesized from claude-code-security-review's audit prompt and false-positive filter. The discipline of this phase is what makes the gate trustworthy: report only what a senior security engineer would confidently raise in PR review.

## The bar

> Better to miss some theoretical issues than flood the report with false positives.

Each finding must clear three filters:

1. **Concrete attack path.** Specific input, specific code path, specific impact.
2. **Confidence ≥ 8 (out of 10).** Below that, drop.
3. **Not on the hard-exclusion list.** See below.

If you can't write a one-sentence exploit scenario that names the input and the impact, the finding isn't ready.

## Methodology

Three sub-phases. Each is a separate read of the diff.

### 2.1 Repository context research

Before judging the diff, understand the codebase:

- What security frameworks are in use? (helmet, csurf, django CSRF middleware, Rails strong params, etc.)
- What sanitization patterns exist? (zod schemas, pydantic models, joi, validator libraries)
- What auth model? (JWT, session, OAuth, mTLS)
- What's the threat model? (PRODUCT.md may say; otherwise infer from the data being handled)

Without this, every finding is speculative. With it, you can spot deviations from the project's own patterns, which are the highest-signal finds.

### 2.2 Comparative analysis

Compare the diff against the patterns from 2.1:

- Does the new code use the same sanitization layer as the rest of the codebase?
- Does it cross the same privilege boundaries the same way?
- Does it introduce a new attack surface (a new endpoint, a new IPC channel, a new file write)?

Deviations are the high-signal finds. New patterns introduced by the diff are higher signal than findings on patterns that have lived in the codebase for years.

### 2.3 Vulnerability assessment

Walk each modified file. For each user-influenced value, trace the data flow to its sink. The sinks that matter:

- SQL / NoSQL queries — injection
- `exec` / `spawn` / `system` — command injection
- Template rendering with `unsafe`/`raw`/`dangerouslySetInnerHTML` — XSS
- Deserialization (pickle, YAML.load, marshal, eval) — RCE
- File path construction — path traversal
- HTTP redirects with user-controlled URLs — open redirect (only HIGH if host/protocol controlled)
- Crypto primitives — algorithm weakness, IV reuse, missing AEAD
- Auth checks — bypass logic, missing checks on privileged routes

## Categories

The categories from the upstream prompt, copied verbatim because the wording matters:

**Input Validation Vulnerabilities:**
- SQL injection via unsanitized user input
- Command injection in system calls or subprocesses
- XXE injection in XML parsing
- Template injection in templating engines
- NoSQL injection in database queries
- Path traversal in file operations

**Authentication & Authorization Issues:**
- Authentication bypass logic
- Privilege escalation paths
- Session management flaws
- JWT token vulnerabilities (none algorithm, key confusion, weak secret)
- Authorization logic bypasses (IDOR, missing role check)

**Crypto & Secrets Management:**
- Hardcoded API keys, passwords, or tokens
- Weak cryptographic algorithms (MD5/SHA1 for security, DES, RC4)
- Improper key storage or management
- Cryptographic randomness issues (`Math.random` for tokens, predictable seeds)
- Certificate validation bypasses

**Injection & Code Execution:**
- RCE via deserialization
- Pickle injection in Python
- YAML deserialization (`yaml.load` without `SafeLoader`)
- Eval injection (`eval`, `Function`, `setTimeout(string)`)
- XSS in web applications (reflected, stored, DOM-based)

**Data Exposure:**
- Sensitive data logging or storage
- PII handling violations
- API endpoint data leakage (returning more than needed)
- Debug information exposure (stack traces in 500 responses)

## Hard exclusions

Drop these categories on sight. They are noise for this gate, even when technically true.

1. **Denial of Service** of any kind, including resource exhaustion.
2. **Secrets stored on disk** if they are otherwise secured. Handled by separate processes.
3. **Rate limiting concerns** or service overload scenarios.
4. **Memory consumption / CPU exhaustion** issues.
5. **Lack of input validation** on non-security-critical fields without proven impact.
6. **GitHub Actions input sanitization** unless clearly triggerable via untrusted input.
7. **Lack of hardening measures.** Code is not expected to implement every best practice; only flag concrete vulns.
8. **Theoretical race conditions / timing attacks.** Only report if concretely problematic.
9. **Outdated third-party libraries.** Managed separately.
10. **Memory safety issues in memory-safe languages** (Rust, Go, JVM, .NET, JS, Python). Drop entirely.
11. **Findings in test files.**
12. **Log spoofing.** Outputting unsanitized user input to logs is not a vulnerability.
13. **SSRF that only controls the path.** SSRF is only HIGH if host or protocol is controllable.
14. **User-controlled content in AI system prompts.** Not a vulnerability for this gate.
15. **Regex injection.** Not a vulnerability.
16. **Regex DoS (ReDoS).** Falls under DoS exclusion.
17. **Findings in markdown / documentation files.**
18. **Lack of audit logs.** Not a vulnerability.

## Precedents

When in doubt, apply these:

1. Logging high-value secrets (passwords, API keys, session tokens) in plaintext is a vuln. Logging URLs is assumed safe.
2. UUIDs are unguessable. Don't require validation.
3. Environment variables and CLI flags are trusted. Attacks that rely on controlling them are invalid.
4. Resource leaks (memory, file descriptors) are not security findings.
5. Subtle web vulns (tabnabbing, XS-Leaks, prototype pollution, open redirects) drop unless extremely high confidence.
6. **React and Angular auto-escape.** Drop XSS findings on `.tsx` / Angular components unless `dangerouslySetInnerHTML`, `bypassSecurityTrustHtml`, or equivalent is used.
7. Most GitHub Action workflow vulns are not exploitable in practice. Verify a specific attack path before reporting.
8. Client-side permission/auth checks are not vulnerabilities. The server is responsible.
9. Only include MEDIUM findings if obvious and concrete.
10. Most notebook (`*.ipynb`) vulns are not exploitable. Require a specific untrusted-input path.
11. Logging non-PII data is not a vulnerability even if "sensitive."
12. Command injection in shell scripts is only a vuln when the script accepts untrusted input.

## Severity (this phase)

| This-phase severity | Maps to gate severity | Definition |
|---|---|---|
| HIGH | P0 | Directly exploitable. RCE, data breach, auth bypass. Confidence ≥ 8 to report; ≥ 9 to ship-block. |
| MEDIUM | P1 | Exploitable under specific conditions, with significant impact. Only report if obvious and concrete. |
| LOW | drop | Defense-in-depth. Drop unless the project explicitly asked for hardening review. |

## Confidence scale

- **0.9–1.0 (9–10/10)**: Certain exploit path identified; reproduction steps clear.
- **0.8–0.9 (8/10)**: Clear vulnerability pattern with known exploitation methods.
- **0.7–0.8 (7/10)**: Suspicious pattern requiring specific conditions. Borderline; report only if HIGH severity.
- **Below 0.7**: Drop. Too speculative.

## Output format

Every finding follows this template:

```markdown
### [HIGH | MEDIUM] <category>: <file>:<line>

- **Severity**: HIGH | MEDIUM
- **Confidence**: <8|9|10>/10
- **Evidence**: `<code snippet, ≤120 chars>`
- **Exploit scenario**: <one sentence naming the input, the path, the impact>
- **Recommendation**: <specific fix, prefer existing project patterns>
```

Example:

```markdown
### HIGH SQL Injection: `app/api/search.py:42`

- **Severity**: HIGH
- **Confidence**: 10/10
- **Evidence**: `cursor.execute(f"SELECT * FROM products WHERE name LIKE '%{query}%'")`
- **Exploit scenario**: Attacker passes `'; DROP TABLE products; --` to the `/api/search?q=` endpoint, executing arbitrary SQL with the API user's privileges.
- **Recommendation**: Use the same parameterized pattern as `app/api/users.py:88`: `cursor.execute("SELECT * FROM products WHERE name LIKE %s", (f"%{query}%",))`.
```

## Block decision

- Any HIGH with confidence ≥ 9 → **P0, block ship now.**
- Any HIGH with confidence 8 → **P1, block ship before release.**
- Any MEDIUM with confidence ≥ 9 and obvious impact → **P1.**
- Everything else → not blocking. Report under "Non-blocking" only if confidence ≥ 7.

## Coordinating with the GitHub Action

If the project runs `claude-code-security-review` as a GitHub Action, this phase is partial coverage. The Action does the full PR sweep with inline review comments. This phase covers the same prompt against the working tree before the PR opens. Run both for defense in depth.

When `--no-security` is passed, this phase is skipped. The verdict line records: `phase_2: skipped (--no-security; covered by GitHub Action)`.

---
name: owasp-web-top10
description: Audit a web application against OWASP Top 10:2025. Runs parallel subagents for each category, produces a prioritized findings report with severity ratings, and optionally implements fixes. Use before launch or after major changes.
argument-hint: [scope: full | quick | A01..A10] [output: report-only | report-and-fix | fix-critical]
---

# OWASP Web Top 10:2025 Audit

You are conducting a security audit against the OWASP Top 10:2025. Work through the phases in order.

**Args:** {{args}}

If `{{args}}` already includes a scope and output mode (e.g. `quick report-only`, `A03 report-and-fix`), skip the matching interview questions in Phase 1. The Stack question is always required.


## Phase 1: Interview

Use `AskUserQuestion` for each question below, one at a time. Skip questions whose answer was already provided in `{{args}}`.

**Question 1: Scope**
| Option | Description |
|--------|-------------|
| `full` | All 10 categories (thorough, 10+ min) |
| `quick` | A01, A02, A03, A04, A05 only - the five categories ranked highest by OWASP 2025 exploit + impact data |
| `single` | One specific category - follow up with a free-text question asking which (answer must match `A01`..`A10`; this is the same form accepted directly in `{{args}}`) |

**Question 2: Stack** - free text:
> What is your stack? (e.g. "Next.js, PostgreSQL, Drizzle, Vercel") Include auth provider, ORM, cloud host, and any CI/CD platform.

**Question 3: Output**
| Option | Description |
|--------|-------------|
| `report-only` | Findings report, no code changes |
| `report-and-fix` | Findings report + implement all critical/high fixes |
| `fix-critical` | Implement critical fixes only, abbreviated report (critical findings + remediation log) |


## Phase 2: Explore

Spawn **3 parallel subagents** to map the codebase before auditing. Each subagent must return findings in this structured format so Phase 3 subagents can consume them:

```
AREA: <name>
FILES:
  - <path>: <one-line role description>
TECH:
  - <library/framework/version>
ENTRY_POINTS:
  - <route or handler>: <auth required? yes/no>
NOTABLE:
  - <anything unusual: TODO comments, disabled checks, custom crypto, etc.>
```

| Subagent | Focus |
|----------|-------|
| 1 | Auth, session handling, JWT/cookie config, role/permission checks, CSRF token middleware, middleware order |
| 2 | Database queries, input handling, API routes, file uploads, error responses, output rendering |
| 3 | Dependencies (package.json/pyproject.toml/Cargo.toml/go.mod), lockfiles, env config, secrets handling, logging, CI workflows, Docker/infra files |

Collect findings before proceeding. Pass the combined map to every Phase 3 subagent as context.


## Phase 3: Parallel Audit

Spawn **one subagent per active category** (all 10 for full scope, A01..A05 for quick, just the chosen one for single). Pass the combined Phase 2 map to every subagent as context so they don't re-explore. Each subagent reads the codebase and reports findings in this format:

```
CATEGORY: <ID> - <Name>
SEVERITY: Critical | High | Medium | Low | Pass
FINDINGS:
  - <specific file:line or pattern>: <what is wrong>
  - ...
FIXES:
  - <specific change needed>
  - ...
```

---

### A01:2025 - Broken Access Control

Check for:
- Missing authorization checks on routes/endpoints (especially POST/PUT/DELETE/PATCH). Grep for route definitions and verify each has either an explicit auth middleware or a documented public reason.
- Insecure direct object references: any route taking a numeric/uuid ID parameter must scope the query to the current user. Grep: `findUnique`, `findFirst`, `findById`, `SELECT * FROM .* WHERE id =`, `params.id`, `params\.[a-zA-Z]+Id`.
- Privilege escalation: search for role/permission checks (`isAdmin`, `role ===`, `hasRole`, `requireRole`); confirm they are enforced server-side, not just hidden in the UI.
- CORS misconfiguration: grep for `Access-Control-Allow-Origin: *` with `Allow-Credentials: true`, or reflective origin echoing without an allowlist. Check `cors()` config in Express/Fastify/Hono.
- CSRF protection: state-changing routes (POST/PUT/DELETE/PATCH) reached via browser cookies must verify a CSRF token, SameSite=Strict/Lax cookies, or use a non-cookie auth scheme (Authorization header). Grep for `csurf`, `csrf`, `SameSite`.
- Force browsing: search for client-only `if (user.isAdmin)` guards with no matching server check.
- JWT/session token validation gaps: grep for `jwt.verify` / `jwt.decode` - `decode` without `verify` is a critical bug. Check `algorithms: ['HS256']` is pinned (no `alg:none`, no algorithm confusion).
- Missing `deny by default`: framework-level catch-all middleware should require auth unless a route opts out.
- File path traversal in static serving and download routes (`../`, absolute paths, symlink following).

Look in: route handlers, middleware, auth guards, API endpoints, `middleware.ts` / `middleware.py`, Next.js `(auth)` route groups, framework auth plugins.

---

### A02:2025 - Security Misconfiguration

Check for:
- Default credentials in config files, env examples, or seed scripts. Grep: `admin/admin`, `password123`, `changeme`, `DEFAULT_PASSWORD`.
- Unnecessary features enabled in production: debug routes (`/debug`, `/__debug__`), directory listing, Swagger/OpenAPI UI (`/api-docs`, `/swagger`), Spring `/actuator`, GraphQL introspection, Next.js source maps, dev-only endpoints.
- Missing security headers: CSP, HSTS, X-Frame-Options (or `frame-ancestors`), X-Content-Type-Options, Referrer-Policy, Permissions-Policy. Check `next.config.js` headers, `helmet()`, middleware, reverse proxy config.
- Verbose error messages leaking stack traces, SQL strings, internal paths to users. Search for `error.stack`, `err.message` rendered to response, framework default error pages in prod.
- Insecure cloud permissions: grep configs/IaC for `"Effect": "Allow", "Principal": "*"`, `"acl": "public-read"`, `0.0.0.0/0` ingress on non-HTTP ports, public S3/R2/GCS buckets.
- Development config leaking into production: missing `NODE_ENV` checks, dev-only middleware mounted unconditionally, source maps deployed.
- Exposed `.env`, `.env.local`, `config.json`, `.git/`, `.DS_Store` reachable via web root. Confirm `.gitignore` covers them and the deployed static dir excludes them.
- CORS configured globally rather than per-route (covered in A01, cross-reference here).
- TLS / reverse proxy: HSTS preload, redirect HTTP to HTTPS, modern cipher suites only.

Look in: server config, framework config (`next.config.js`, `vite.config.ts`, `nginx.conf`), environment files, error handlers, IaC (`terraform/`, `pulumi/`, CloudFormation), Dockerfiles.

---

### A03:2025 - Software Supply Chain Failures

Check for:
- Dependencies with known CVEs. Run the project's package manager audit, falling back across them:
  - JS/TS: `bun audit` || `pnpm audit --prod` || `npm audit --omit=dev`
  - Python: `pip-audit` || `safety check`
  - Go: `govulncheck ./...`
  - Rust: `cargo audit`
  - Ruby: `bundle audit check --update`
- Unpinned versions using broad ranges (`^`, `~`, `*`, `latest`) for security-critical packages (auth libs, crypto, parsers, framework core). Lockfile must exist and be committed.
- Packages from unusual scopes or typosquatted names (e.g. `lodahs`, `crossenv`, `react-doom`). Cross-check against the well-known list.
- Absence of lockfile integrity checks in CI: `npm ci` / `bun install --frozen-lockfile` / `pip install --require-hashes` should be used, not `npm install`.
- Third-party scripts loaded without Subresource Integrity (SRI). Grep HTML/JSX for `<script src="https://` without `integrity=`.
- Unverified GitHub Actions: `uses:` references must be pinned to a commit SHA, not a mutable tag (`@v3`, `@main`). Grep `.github/workflows/*.yml` for `uses: .*@v\d+$`.
- Docker base images pinned by digest (`image@sha256:...`) not tag (`image:latest`).
- Direct `curl | bash` or `wget | sh` patterns in Dockerfiles, install scripts, or CI.

Look in: `package.json`, lockfiles (`bun.lockb`, `pnpm-lock.yaml`, `package-lock.json`, `poetry.lock`, `go.sum`, `Cargo.lock`), `.github/workflows/`, `Dockerfile`, CDN script tags in HTML/JSX templates.

---

### A04:2025 - Cryptographic Failures

Check for:
- Sensitive data transmitted or stored without encryption: `http://` links to internal services, plaintext DB columns holding PII / tokens / payment data, plaintext backups.
- Weak algorithms: grep for `md5`, `sha1`, `des`, `rc4`, `ECB`, `Math.random()` used for tokens, `RSA.*1024`. Also flag `crypto.createCipher` (deprecated, use `createCipheriv`).
- TLS verification disabled: grep for `rejectUnauthorized: false`, `verify=False` (Python requests), `InsecureSkipVerify: true` (Go), `NSAllowsArbitraryLoads`, `--no-check-certificate`.
- Weak TLS versions allowed: TLS 1.0 / 1.1 enabled in server config; missing TLS termination on internal services.
- Hardcoded keys, secrets, or IVs in source code. Grep: `-----BEGIN`, `sk_live_`, `pk_live_`, `AKIA`, `ghp_`, `xoxb-`, `eyJ` (JWTs), `secret\s*=\s*["']`, `apiKey\s*=\s*["']`.
- Scan git history for committed secrets:
  - `git log -p -S "BEGIN PRIVATE KEY" --all`
  - `git log -p -S "AKIA" --all`
  - Or use `gitleaks detect --source . --log-opts="--all"` / `trufflehog git file://.` if installed.
- Missing `Secure` / `HttpOnly` / `SameSite` flags on session/auth cookies. Verify cookie config in auth library.
- Insufficient key management: keys in `.env` files committed to git, no rotation policy, prod secrets in dev `.env.local` examples.
- Password hashing with non-adaptive algorithms: plain SHA-256, MD5, unsalted hashes. Require bcrypt (cost >= 10), argon2id, or scrypt.
- Random number generation: `Math.random()` / `random.random()` used for tokens, session IDs, password resets. Must use CSPRNG (`crypto.randomBytes`, `secrets.token_urlsafe`).
- Encryption without authentication (CBC without HMAC, AES without GCM/CCM).

Look in: crypto utilities, password hashing, cookie config, env files, `.env.example`, git history, HTTP client config (axios/fetch/requests), TLS config in reverse proxy.

---

### A05:2025 - Injection

Note: in 2025 SSRF (formerly A10:2021) was merged into this category. Audit it explicitly.

Check for:
- SQL injection via string concatenation. Grep: `` `SELECT.*\${` ``, `"SELECT " +`, `f"SELECT ... {`, `query(` with template interpolation. Drizzle/Prisma raw query helpers (`sql\`\`` `, `$queryRaw`) must use parameter bindings, not template-interpolated user data.
- NoSQL injection: MongoDB queries passing raw request body as filter (`User.find(req.body)`); user input becoming operator keys (`{ $ne: null }`, `{ $gt: "" }`). Grep: `Model\.find\(req\.`, `findOne\(.*req\.body`.
- OS command injection: grep `exec(`, `execSync(`, `spawn(.*shell: true`, `os.system(`, `subprocess.*shell=True`, backticks. Any user input reaching these is critical.
- SSRF (Server-Side Request Forgery): server-side `fetch` / `axios` / `requests.get` / `http.get` calls with user-controlled URLs. Verify URL allowlist, block private IP ranges (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `127.0.0.0/8`, `169.254.0.0/16`, `::1`, IPv4-mapped IPv6), block `file://`, `gopher://`, `dict://` schemes. Includes webhooks, URL previews, image proxies, PDF generators (Puppeteer/Playwright pointed at user URLs).
- LDAP injection: user input in LDAP filters without escaping.
- XML injection / XXE: XML parsers must disable external entity resolution (`libxml.set_default_parser`, `XMLParser(resolve_entities=False)`, `DocumentBuilderFactory.setFeature`).
- Template injection (SSTI): user input passed into template engines as the template itself (`Handlebars.compile(userInput)`, `Jinja2(...).render(userInput)`, `eval`, `new Function`).
- Cross-Site Scripting (XSS):
  - Reflected/stored: user content rendered via `dangerouslySetInnerHTML`, `v-html`, `innerHTML`, `document.write`, `eval`, untrusted `href="javascript:..."`.
  - DOM-based: `location.hash` / `location.search` flowing to sinks above.
  - Verify output encoding before HTML rendering and a strict CSP as defense-in-depth.
- Header injection / CRLF: user input in response headers, redirects (`res.redirect(userUrl)` with no allowlist).
- Open redirect: `redirect_uri`, `next`, `returnTo` params not validated against an allowlist.
- Prototype pollution: `Object.assign({}, req.body)`, lodash `_.merge` / `_.set` with user keys.

For each query/command: verify user input is never concatenated directly. Verify ORM queries use parameterized bindings. Verify output encoding before HTML rendering.

Look in: route handlers, ORM/raw query usage, shell/process spawning, HTTP clients making outbound requests, redirect handlers, template renderers.

---

### A06:2025 - Insecure Design

Check for:
- Missing rate limiting on login, registration, password reset, OTP/MFA, email send, payment, and any expensive endpoint (search, export, AI calls). Verify rate limit is keyed on something the attacker can't trivially rotate (IP + account, not IP alone behind a CDN).
- No account lockout, exponential backoff, or CAPTCHA on brute-force targets.
- Sensitive workflows missing multi-step verification (email change, password change, payout, role change should require re-auth or email confirm).
- Business logic flaws: search handlers for `quantity`, `price`, `amount`, `discount` - confirm server validates against authoritative DB values, not trusts client-submitted values. Check for negative numbers, integer overflow, race conditions on balance changes.
- Trust boundary violations: client-side validation only (price computed in JS, then submitted as a number).
- Missing tenant isolation in multi-tenant apps (workspace/org ID must come from the session, not the request body).
- Workflow bypass: can users skip required steps by calling endpoints out of order? (e.g. confirm payment before paying, mark order shipped without auth).
- Missing threat modeling artifacts or abuse case documentation (look for `THREAT_MODEL.md`, `SECURITY.md`).
- Anti-automation controls absent on high-value actions.
- Resource enumeration via sequential IDs; consider opaque IDs/UUIDs for shared resources.

Look in: auth flows, payment flows, API rate limit config (`@upstash/ratelimit`, `express-rate-limit`, framework middleware), business logic handlers, checkout/order processing, admin actions.

---

### A07:2025 - Authentication Failures

Check for:
- Weak password policy: minimum 8 chars (NIST), block top-10k common passwords, no max-length cap below 64, allow unicode. No mandatory periodic rotation.
- Credential stuffing exposure: rate limit + MFA option + breach-password check (e.g. HaveIBeenPwned k-anonymity API).
- Insecure password reset: tokens must be CSPRNG-generated (>= 128 bits entropy), single-use, time-limited (<= 1 hour), invalidated after use, scoped to one account.
- Session tokens not invalidated on logout, password change, MFA change, or role downgrade. Check for server-side session revocation (not just client cookie clear).
- Session fixation: session ID must rotate on login (`req.session.regenerate`, `session.cycle`).
- Remember-me tokens with excessive lifetimes (> 30 days) or no device binding.
- MFA: required for admin/privileged accounts; recovery codes single-use; TOTP secrets stored encrypted at rest.
- Plaintext credential transmission anywhere in the flow (HTTP login form, query string).
- User enumeration via:
  - Different error messages ("user not found" vs "wrong password")
  - Different response times (timing attack on user lookup)
  - Different status codes on `/register` (409 if exists vs 201 if new)
  - Password reset that says "no such email"
- OAuth/OIDC: state parameter required, PKCE for public clients, `redirect_uri` exact-match against allowlist, ID token signature verified.
- WebAuthn/passkey storage handled by vetted library, not custom.

Look in: auth routes, session config, password reset flow, logout handler, OAuth callback, MFA setup/verify, `auth.config.ts` / Auth.js / Clerk / Lucia / better-auth config.

---

### A08:2025 - Software or Data Integrity Failures

Check for:
- Deserializing untrusted data without validation. Grep:
  - Python: `pickle.loads`, `yaml.load` (without `SafeLoader`), `marshal.loads`, `shelve.open`
  - Node: `node-serialize`, `serialize-javascript` with `eval`, `vm.runInNewContext` with user input
  - Java: `ObjectInputStream`, Jackson polymorphic deserialization with `enableDefaultTyping`
  - PHP: `unserialize` on user input
  - Ruby: `Marshal.load`, `YAML.load`
- Missing integrity checks on software updates, plugins, themes, or config fetched at runtime over the network.
- CI/CD pipeline steps pulling from unverified or mutable sources: unpinned actions, third-party scripts curled at build time, dependencies installed without lockfile freeze.
- Auto-update mechanisms without cryptographic signature verification.
- Unsigned JWT with algorithm confusion risk:
  - `jwt.verify(token, secret)` without `algorithms: [...]` pinned (allows `alg: none` or HS256/RS256 confusion).
  - JWT secret reused across environments.
  - JWT used for session without revocation list.
- Webhook payloads accepted without HMAC signature verification (Stripe, GitHub, Slack, etc.).
- Cache poisoning: response varies on header not in `Vary`; user-controllable cache key.
- File upload integrity: missing content-type validation, missing magic byte check, executable extensions allowed (`.html`, `.svg`, `.htm`, `.xhtml`, `.php`, `.exe`).

Look in: deserialization code, update mechanisms, CI/CD configs, JWT configuration, webhook handlers, file upload handlers, CDN/cache config.

---

### A09:2025 - Logging and Alerting Failures

Check for:
- No logging of authentication events (login success, login failure, logout, password change, MFA setup/use, account lockout).
- No logging of authorization failures or access control denials (403s on protected resources).
- No logging of input validation failures, server-side errors, or high-value transactions.
- Log injection: user input written to logs without sanitization (newlines that forge log lines, ANSI escape sequences, `\r\n` CRLF).
- Sensitive data written to logs: passwords, tokens, session IDs, full JWTs, full card numbers, full SSNs, full email body content. Grep log statements for variable names like `password`, `token`, `secret`, `authorization`, `cookie`.
- Logs stored only locally on a single host with no centralization, no retention policy, no monitoring or alerting.
- No structured logging (JSON) - makes correlation impossible.
- No correlation/trace ID propagated across services.
- No audit trail for admin actions, role changes, data exports, or high-value data access. Audit log should be append-only / tamper-evident.
- No alerting on suspicious patterns: spike in 401/403, login from new geo, MFA disabled, mass data export.
- Log retention below the minimum required by your compliance regime (typically 90 days minimum; GDPR/SOC2/HIPAA each have specifics).

Look in: logging configuration (`pino`, `winston`, `logging` module, `slog`), auth handlers, error handlers, audit log code, alerting/monitoring config (Datadog, Sentry, BetterStack, Grafana), middleware that logs requests.

---

### A10:2025 - Mishandling of Exceptional Conditions

Check for:
- Unhandled exceptions revealing internal state, stack traces, or SQL to users.
- Silent error swallowing: empty `catch {}`, `except: pass`, `.catch(() => {})`, errors logged but not surfaced. Grep for `catch\s*\(\s*[^)]*\)\s*\{\s*\}` and `except.*:\s*pass`.
- Missing input boundary checks causing integer overflow, off-by-one, null dereference.
- Race conditions / TOCTOU: check-then-use patterns on balances, inventory, unique-constraint races (`findFirst` then `create` without transaction or unique index).
- Incomplete error handling leaving transactions or state inconsistent. Verify DB writes spanning multiple statements use transactions and roll back on error.
- Crash-on-invalid-input in parsing routines (JSON, XML, image, PDF). All parsers must handle malformed input as expected exceptions, not panics.
- ReDoS (Regex Denial of Service): catastrophic backtracking in regex applied to user input. Grep regex patterns for nested quantifiers `(a+)+`, `(a*)*`, `(a|a)*`. Tools: `safe-regex` for Node.
- Resource exhaustion:
  - No request body size limit (`body-parser` default, `express.json()` without `limit`).
  - No file upload size limit.
  - No pagination cap on list endpoints (user can request `limit=999999`).
  - No timeout on outbound HTTP calls (default Node `fetch` has no timeout).
  - No query timeout at the DB level.
  - Unbounded recursion, unbounded loops driven by user input.
- Zip bomb / billion laughs on archive or XML parsing.
- Memory leaks from event listener / interval not cleaned up on error path.

Look in: error handlers, async code, file upload handlers, parsing utilities, regex usage, DB transaction boundaries, HTTP client config.


## Phase 4: Report

Skip this phase if the user selected `fix-critical` - in that mode, write only an abbreviated `security/owasp-critical-fixes.md` log instead (Critical findings + what was fixed).

Generate `security/owasp-top10-report.md` (project-relative path; create the `security/` directory if needed) with:

```markdown
# OWASP Top 10:2025 Security Audit

**Date:** <today>
**Scope:** <stack and categories audited>
**Overall Risk:** Critical | High | Medium | Low

## Summary

| Category | Severity | Findings |
|----------|----------|----------|
| A01 Broken Access Control | <sev> | N issues |
| A02 Security Misconfiguration | <sev> | N issues |
| A03 Software Supply Chain Failures | <sev> | N issues |
| A04 Cryptographic Failures | <sev> | N issues |
| A05 Injection | <sev> | N issues |
| A06 Insecure Design | <sev> | N issues |
| A07 Authentication Failures | <sev> | N issues |
| A08 Software or Data Integrity Failures | <sev> | N issues |
| A09 Logging and Alerting Failures | <sev> | N issues |
| A10 Mishandling of Exceptional Conditions | <sev> | N issues |

## Critical Findings (fix immediately)

### <CAT-ID>: <short title>
- **File:** `<path>:<line>`
- **Impact:** <what an attacker gains>
- **Evidence:** <code snippet or grep result>
- **Fix:** <concrete change>
- **Status:** Open | Fixed in commit `<sha>`

## High Findings (fix before next release)
<same structure>

## Medium Findings (fix within 30 days)
<same structure>

## Low Findings (track and monitor)
<same structure>

## Passing Controls

Categories where no issues were found, with one-line note on what was verified.

## Remediation Priority
1. <CAT-ID>: <short title> - <reason it is first>
2. ...

## Out-of-Scope / Infra Findings

Findings that require cloud, DNS, certificate, or org policy changes outside the codebase.
```


## Phase 5: Fix (if report-and-fix or fix-critical selected)

For each Critical (and also High if `report-and-fix`) finding:
1. Implement the specific fix identified in Phase 3.
2. Add a test or assertion verifying the fix where possible (regression test on the vulnerable code path).
3. Update the Phase 4 deliverable (the full report, or the abbreviated log under `fix-critical`) - set status to `Fixed in commit <sha>` (or `Fixed in working tree` if not yet committed).
4. Re-run the relevant Phase 3 check against the changed code to confirm the fix holds.

Do not refactor beyond what is needed to fix the specific vulnerability. Do not bundle unrelated changes. Do not auto-commit unless the user explicitly asks.


## Completion

List every file changed. Note any findings that require infrastructure changes (cloud config, DNS, certificates, IAM policy, WAF rules) that cannot be fixed in code - these belong under "Out-of-Scope / Infra Findings" in the report.

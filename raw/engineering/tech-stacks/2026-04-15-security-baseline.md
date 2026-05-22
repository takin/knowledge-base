# Security Baseline: Web Applications

Source URL: Internal draft
Collected: 2026-04-15
Published: 2026-04-15
Status: Draft
Scope: Cross-stack web application security baseline

---

## 12. Security Hardening

Security hardening is mandatory from day one. It is not a pre-launch checklist item.

### 12.1 HTTP security headers

Every HTTP surface must include the applicable security headers. Configure these at the server/middleware or Nginx level, not per-route.

| Header | Value | Purpose |
|---|---|---|
| `Content-Security-Policy` | Required for HTML/web responses; restrictive nonce-based policy, at minimum `default-src 'self'` plus `script-src 'self' 'nonce-$csp_nonce'` | Prevents XSS by blocking unauthorized script sources |
| `X-Content-Type-Options` | `nosniff` | Prevents MIME-type sniffing |
| `X-Frame-Options` | `DENY` or `SAMEORIGIN` | Prevents clickjacking |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Controls referrer leakage |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Enforces HTTPS (production only) |
| `Cross-Origin-Opener-Policy` | `same-origin` | Prevents Spectre-family attacks by isolating browsing context |
| `Permissions-Policy` | Restrict camera, mic, geolocation to needed origins only | Limits browser feature access |
| `Cross-Origin-Resource-Policy` | `same-origin` unless cross-origin embeds are explicitly required | Reduces cross-origin data exposure |

Backend JSON APIs that do not serve HTML do not require nonce CSP in the API Nginx template. They still require the non-CSP security headers above. If a backend endpoint serves HTML, CSP applies to that response.

### 12.1.1 Third-party CSP source rules
- Baseline CSP must include `default-src 'self'`, `object-src 'none'`, `base-uri 'self'`, `frame-ancestors 'none'`, and `form-action 'self'` unless an ADR documents a narrower or broader policy.
- Production CSP must not contain wildcard `*` sources.
- Production `script-src` must not use `unsafe-inline` unless a security review documents why a nonce/hash-based policy cannot work.
- `connect-src` must list only approved API, monitoring, analytics, storage, or websocket endpoints actually used by the product.
- The default CSP must not hardcode third-party analytics, payment, storage, iframe, or regional collection endpoints.
- Add third-party sources only when the project actually uses that provider.
- Google Analytics may require `https://www.google-analytics.com`.
- Some Google Analytics deployments may require a regional endpoint such as `https://region1.google-analytics.com`, but this varies by project and must be added explicitly after verification.
- Google Tag Manager requires script permission for `https://www.googletagmanager.com` when used.
- Every third-party CSP source must be justified in the project ADR or security review.

### 12.2 CSRF protection

JWT bearer authentication is the default for standalone Bun + Elysia backend APIs. CSRF protection is mandatory only when a project uses cookie-based credentials for state-changing requests.

For public-facing forms or non-auth mutation endpoints, verify the `Origin` header server-side and reject requests from unexpected origins.

If cookie-based auth is used, every state-changing backend endpoint must enforce CSRF protection. Client-side route guards are UX only and never replace server-side authorization or CSRF checks.

If bearer JWT auth is used, clients must send tokens in `Authorization: Bearer <token>` and state-changing requests still require normal authorization, RBAC/scope checks, Origin validation where relevant, and rate limiting.

### 12.2.0 JWT, JWK, and JWKS rules

Standalone backend APIs use JWT-first authentication with asymmetric JOSE keys.

Rules:
- Use `jose` for JWT signing, verification, JWK handling, and JWKS handling.
- Production SaaS/public API JWTs must use asymmetric signing keys with `kid` headers.
- Prefer `EdDSA` where runtime support is clean; use `RS256` as the compatibility fallback.
- JWT protected headers include `alg`, `kid`, and `typ: "JWT"`.
- Expose public verification keys at `/.well-known/jwks.json`.
- Validate JWT signature, `iss`, `aud`, `exp`, `nbf` when present, and required claims.
- `HS256` is banned for production SaaS/public API JWTs unless an ADR approves it.
- Access tokens are short-lived.
- Refresh tokens use rotation and revocation.
- Machine tokens are scoped, revocable, and expire.
- Private signing keys must come from Vault or the approved secrets manager, never source code or Docker images.
- Old public keys remain in JWKS until maximum token lifetime plus safety window has elapsed.
- API keys and refresh tokens must be hashed at rest; show API key plaintext only once at creation.
- JWT claims are not sufficient authorization. Every protected action still checks workspace membership, RBAC permissions or scopes, and resource ownership.

### 12.2.1 CORS rules

CORS must be explicit and environment-specific.

Rules:
- Allow only known dashboard and landing origins for each environment.
- Do not use `Access-Control-Allow-Origin: *` with credentials.
- Do not reflect arbitrary `Origin` values without allowlist validation.
- Keep staging, preview, and production origin allowlists separate.
- If cookies are used cross-site, document `SameSite`, `Secure`, and CSRF behavior in the auth/security review.
- Preflight responses must not expose internal headers, debug headers, or implementation details.

### 12.3 XSS prevention

- Never use `dangerouslySetInnerHTML`. If you must render HTML from an external source, sanitize it with DOMPurify before rendering.
- Stack-standard route schemas on every mutation input prevent injection via unvalidated user data. For Bun + Elysia, use Elysia/TypeBox at the HTTP boundary and Zod for service/domain validation.
- CSP headers (§12.1) are the last line of defense — do not rely on them as the primary mitigation.
- Do not use `eval()`, `new Function()`, or dynamic `import()` with user-controlled paths.
- Never render raw backend errors, SQL errors, stack traces, Zod internals, provider diagnostics, or exception messages directly to users.
- Markdown or HTML rendering from external sources requires an explicit sanitizer and security review.
- JSON-LD helpers must escape `<` as `\u003c` before injection.

### 12.4 SQL injection

Drizzle's parameterized query builder prevents SQL injection by construction. Raw SQL strings in application code are banned (see §8.2). If a raw query is unavoidable, use Drizzle's `sql` tagged template literal, which parameterizes values automatically.

### 12.5 Rate limiting

All authentication endpoints (login, signup, password reset, OTP) must be rate-limited. General API endpoints must have a baseline rate limit.

- Standalone Bun + Elysia APIs must use a Redis-backed rate limiter because Redis is mandatory in that stack.
- If Redis is unavailable, protected production endpoints should fail closed or degrade according to a documented project policy; do not silently fall back to per-process limits in production.

Rate limit responses must return HTTP `429 Too Many Requests` with a `Retry-After` header.

Rate limits must cover:
- IP address.
- User.
- Workspace or tenant.
- API client.
- Authentication and token refresh endpoints.
- API key creation and token exchange endpoints.
- Upload intent endpoints.
- Public API route groups.
- Webhook endpoints only when provider retry behavior allows rate limiting safely.

### 12.6 Input sanitization & output encoding

- Validate all inputs with the stack-standard schema system before processing. For Bun + Elysia, route boundaries use Elysia/TypeBox schemas and service/domain rules may use Zod.
- Encode user-generated content when rendering it back to the page (React's JSX does this by default for text nodes — do not bypass it).
- File uploads: validate MIME type server-side (not just by extension), enforce size limits, and store outside the web root.
- Never store uploaded blob bytes or base64 payloads in PostgreSQL. Store files in S3-compatible object storage and store only metadata/object keys in the database.
- Presigned upload/download URLs must be short-lived and issued only after authorization checks.
- Object storage buckets are private by default; public-read buckets require an explicit ADR.

### 12.7 Sensitive data handling

- Never log PII (names, emails, phone numbers, payment info) to any logging service including Sentry.
- Never store plaintext passwords.
- Never store plaintext API keys or refresh tokens. Store only hashed values after initial issuance.
- Mask sensitive fields (card numbers, tokens) in all log output.
- Tokens and secrets in `.env` must never appear in source code, API responses, browser bundles, logs, or plaintext database rows. Hashed API keys, hashed refresh tokens, and revocation records may be stored server-side.
- JWT private keys, refresh-token signing secrets, webhook secrets, storage credentials, and telemetry write tokens are secrets and must not be browser-exposed.
- Session replay is disabled by default. It requires explicit approval, masking rules, and proof that sensitive fields are redacted before production.
- Error monitoring may be enabled before production, but PII scrubbing is mandatory.
- Health and readiness endpoints must not expose environment variables, secret names, dependency credentials, internal hostnames, image tags, or detailed config.

### 12.7.0 Authorization and tenant isolation

RBAC is mandatory for SaaS APIs.

Rules:
- Every protected backend action checks authentication, workspace membership, RBAC permission or API scope, and resource ownership.
- Tenant-owned database tables must include a tenant/workspace identifier.
- Tenant-owned queries must filter by tenant/workspace identifier in the service layer.
- System-admin or cross-tenant actions require explicit system roles and audit logging.
- Authorization denials should return safe `403` errors and emit audit/security telemetry without exposing internal policy details.

### 12.7.1 Browser-exposed environment variables

Only intentionally public configuration may be exposed to browser code.

Rules:
- Browser-exposed values must use an explicit public prefix such as `VITE_PUBLIC_*` or the framework's equivalent convention.
- Secrets, private API keys, DSNs with write privileges, tokens, internal hostnames, database URLs, and private endpoints must never use public prefixes.
- `.env.example` must document all required public and server-only variables.
- Builds must fail when required public variables are missing, malformed, or point to an invalid environment.
- Do not use build-time env injection for values that must vary between environments unless the deployment intentionally builds one immutable image per environment.

### 12.8 Dependency security

- Run `bun audit` (or equivalent) in the CI pipeline on every PR.
- Pin exact dependency versions in `package.json` for production dependencies. Use `^` only for dev dependencies.
- Review Dependabot / Renovate alerts within one sprint for high/critical severity.
- Commit and review the Bun lockfile on every dependency change.
- Do not use `latest` tags for Docker images in production or CI release paths.
- New third-party scripts, analytics, chat widgets, payment embeds, and session replay tools require explicit approval before adding CSP sources.
- Do not allow lifecycle/postinstall script exceptions during an active supply-chain incident without security review.
- If a dependency adds install scripts, binary downloads, credential access, or telemetry, review it before merge.

### 12.8.1 TanStack supply-chain advisory handling

All projects adopting this standard using `@tanstack/*` packages must follow TanStack Router advisory `GHSA-g7cv-rxg3-hmpx` / `CVE-2026-45321`. This advisory covers a critical supply-chain compromise where malicious `@tanstack/*` package versions exfiltrated cloud credentials, GitHub tokens, npm tokens, SSH keys, Vault tokens, and other secrets during install.

Mandatory rules:
- Never install affected `@tanstack/*` versions listed in `GHSA-g7cv-rxg3-hmpx`.
- Pin all production `@tanstack/*` dependencies to explicit known-good or patched versions.
- Do not use loose production dependency ranges for `@tanstack/*` packages.
- If an affected version may have been installed, delete `node_modules` and the lockfile, then reinstall from a clean lockfile.
- Treat any CI runner or developer machine that installed affected versions during the advisory window as compromised.
- Rotate all credentials accessible to the install process after suspected exposure, including cloud, GitHub, npm, SSH, Vault, and CI secrets.
- Audit cloud logs, GitHub logs, npm publish/access logs, SSH usage, Vault logs, and CI logs after suspected exposure.
- During an active supply-chain incident, CI may temporarily install with lifecycle scripts disabled using `bun install --ignore-scripts`. Re-enable scripts only after dependency provenance has been reviewed.

Minimum patched versions for relevant stack packages:

| Package | Minimum patched version |
|---|---:|
| `@tanstack/react-router` | `1.169.9` |
| `@tanstack/router-core` | `1.169.9` |
| `@tanstack/react-start` | `1.167.72` |
| `@tanstack/router-cli` | `1.166.50` |
| `@tanstack/router-plugin` | `1.167.42` |
| `@tanstack/router-vite-plugin` | `1.166.57` |
| `@tanstack/start-plugin-core` | `1.169.27` |
| `@tanstack/eslint-plugin-router` | `1.161.13` |
| `@tanstack/eslint-plugin-start` | `0.0.8` |

If a project uses any other `@tanstack/*` package, check the advisory table and pin to its patched version or newer.

Indicators that must trigger incident response:
- `optionalDependencies` containing `@tanstack/setup`.
- Git dependency `github:tanstack/router#79ac49eedf774dd4b0cfa308722bc463cfe5885c`.
- Payload file `router_init.js` in a package tarball or installed package.
- Helper file `tanstack_runner.js`.
- Network calls to `filev2.getsession.org`, `seed1.getsession.org`, `seed2.getsession.org`, or `seed3.getsession.org`.

CI must fail if:
- `package.json` or the lockfile resolves to an affected `@tanstack/*` version.
- A resolved package manifest contains `@tanstack/setup`.
- `router_init.js` appears inside a fetched `@tanstack/*` tarball or installed package.

Forensic exception: `npm pack <package>@<version>` is allowed only for security inspection because it downloads a package tarball without running lifecycle scripts. It must not be used for normal installation or project workflows.

### 12.8.2 Source maps

Production source maps must not be publicly served by default.

Rules:
- If production source maps are required, upload them privately to the error monitoring provider during CI/deploy.
- Remove source maps from the public static artifact after private upload.
- Source map upload tokens must be CI secrets, never browser-exposed variables.
- Public `.map` files require an explicit security exception.

### 12.9 Observability security

Observability must not become a data exfiltration channel.

Rules:
- Error monitoring is recommended before production for authenticated dashboards and backend APIs.
- PII scrubbing is mandatory before sending events to third-party monitoring providers.
- Scrub request bodies, auth headers, cookies, tokens, card data, phone numbers, email addresses, and free-text fields that may contain user data.
- Session replay is opt-in only and requires field masking, DOM text masking where appropriate, and explicit product approval.
- Monitoring clients must not be loaded in the authenticated app shell unless the project has approved telemetry requirements and CSP sources.
- Certificate expiry monitoring is required for self-hosted TLS deployments.
- OpenTelemetry spans, metrics, and logs must not include PII, bearer tokens, refresh tokens, API keys, cookies, raw request bodies, card data, emails, phone numbers, or free-text user content unless an explicit allowlist and scrubber exists.
- Metrics must avoid high-cardinality labels such as `user_id`, `workspace_id`, `request_id`, `resource_id`, raw URL paths containing IDs, email, or phone.
- Pino logs must include request and trace correlation fields, but sensitive headers and bodies are redacted by default.
- Audit logs are not disposable telemetry. They are product/security records stored in PostgreSQL with an explicit retention policy.

### 12.10 Idempotency and replay protection

Unsafe public API writes and external side effects must support idempotency.

Rules:
- Require `Idempotency-Key` for public API writes that create irreversible or externally visible side effects.
- Store request hash, actor, route, and result so safe retries return the original response.
- Reusing the same key with a different payload returns a conflict.
- Webhook handlers must store provider event IDs before queueing work and must not process duplicate event IDs twice.
- Signed webhook payloads must include timestamp/replay protection when the provider supports it.

### 12.11 Queue and worker security

Background jobs are part of the security boundary. Treat BullMQ payloads and worker dashboards as sensitive backend surfaces.

Rules:
- Job payloads must be schema-validated before enqueue and before execution.
- Do not put bearer tokens, refresh tokens, API keys, raw cookies, plaintext secrets, or unredacted PII in job payloads.
- Store sensitive small records in PostgreSQL; store large/binary data in object storage; pass only IDs or object keys in jobs.
- Queue UIs such as Bull Board must be admin RBAC-gated, network-restricted where possible, and hidden from public OpenAPI specs.
- Manual job replay requires admin RBAC, idempotency preservation, and an audit log entry.
- Worker logs and metrics follow the same redaction and high-cardinality label rules as API logs and metrics.

---

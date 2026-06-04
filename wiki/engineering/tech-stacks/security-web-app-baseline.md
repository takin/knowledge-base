# Security Baseline: Web Applications

Updated: 2026-06-04
Status: Draft
Sources: Internal Tech Stacks draft (2026-04-15, updated 2026-05-22); Internal Dashboard Stack draft (2026-05-22); Internal Infrastructure draft (2026-04-15, updated 2026-06-04)
Platform: Cross-stack web application security
Runtime: N/A
Framework: N/A
Primary Use Case: HTTP security headers, CSP, CSRF, CORS, JWT/JWKS, rate limiting, input validation, sensitive data, source maps, observability security, queue security, and supply-chain controls
Raw: [2026-04-15-security-baseline.md](../../../raw/engineering/tech-stacks/2026-04-15-security-baseline.md); [2026-05-22-web-react-vite-dashboard-stack.md](../../../raw/engineering/tech-stacks/2026-05-22-web-react-vite-dashboard-stack.md); [2026-04-15-infra-docker-compose-nginx-stack.md](../../../raw/engineering/tech-stacks/2026-04-15-infra-docker-compose-nginx-stack.md)

## Summary

Security hardening is mandatory from day one. This baseline scopes controls across frontend dashboards, backend JSON APIs, infrastructure, workers, queues, observability, and CI supply chain.

## HTTP Security Headers And CSP

Every HTTP surface includes applicable security headers at the server, gateway, or middleware layer.

| Header | Scope | Purpose |
| --- | --- | --- |
| `Content-Security-Policy` | Required for HTML/web responses | XSS and exfiltration reduction |
| `X-Content-Type-Options: nosniff` | All HTTP surfaces | Prevent MIME sniffing |
| `X-Frame-Options` | All browser-facing surfaces | Clickjacking defense |
| `Referrer-Policy` | Browser-facing surfaces | Referrer leakage control |
| `Strict-Transport-Security` | Production HTTPS | HTTPS enforcement |
| `Cross-Origin-Opener-Policy` | Browser-facing surfaces | Browsing context isolation |
| `Permissions-Policy` | Browser-facing surfaces | Restrict browser feature access |

Backend JSON APIs that do not serve HTML do not require nonce CSP in the API Nginx template. They still require non-CSP security headers. If a backend endpoint serves HTML, CSP applies to that response.

CSP rules for HTML/web responses:
- Baseline CSP includes `default-src 'self'`, `object-src 'none'`, `base-uri 'self'`, `frame-ancestors 'none'`, and `form-action 'self'` unless an ADR says otherwise.
- Production CSP must not contain wildcard `*` sources.
- Production `script-src` must not use `unsafe-inline` unless reviewed.
- `connect-src` lists only approved API, monitoring, analytics, storage, or websocket endpoints actually used.
- Third-party CSP sources are added only when the project uses that provider and are justified in ADR/security review.

## CSRF, CORS, JWT, And JWKS

JWT bearer authentication is the default for standalone Bun + Elysia APIs. CSRF protection is mandatory only when browser cookies authenticate state-changing requests.

CSRF and CORS rules:
- Cookie-authenticated unsafe methods require CSRF protection.
- Bearer-token-only APIs do not require CSRF tokens, but still require authz, origin checks where relevant, and rate limiting.
- CORS allowlists are explicit and environment-specific.
- Do not use wildcard origins with credentials.
- Do not reflect arbitrary `Origin` values.
- Preflight responses must not expose internal/debug headers.

JWT/JWKS rules:
- Use `jose` for JWT signing, verification, JWK, and JWKS handling.
- Production SaaS/public API JWTs use asymmetric signing keys with `kid` headers.
- Prefer `EdDSA`; use `RS256` as compatibility fallback.
- JWT protected headers include `alg`, `kid`, and `typ: "JWT"`.
- Expose public keys at `/.well-known/jwks.json`.
- Validate signature, `iss`, `aud`, `exp`, `nbf` when present, and required claims.
- `HS256` is banned for production SaaS/public API JWTs unless an ADR approves it.
- Access tokens are short-lived.
- Refresh tokens use rotation and revocation.
- Machine tokens are scoped, revocable, and expire.
- Private signing keys come from Vault or approved secrets manager.
- API keys and refresh tokens are hashed at rest; API key plaintext is shown only once.
- JWT claims are not sufficient authorization; every protected action checks workspace membership, RBAC/scopes, and resource ownership.

## Input, XSS, SQL, And Uploads

Rules:
- Validate inputs with the stack-standard schema system before processing. For Bun + Elysia, route boundaries use Elysia/TypeBox and service/domain rules may use Zod.
- Encode user-generated content when rendering it back to the page.
- Never use unsanitized `dangerouslySetInnerHTML`.
- Do not use `eval()`, `new Function()`, or dynamic imports with user-controlled paths.
- Never render raw backend errors, SQL errors, stack traces, schema internals, provider diagnostics, or exception messages directly to users.
- Use Drizzle's parameterized query builder by default.
- Raw SQL strings are banned unless using parameterized helpers.
- File uploads validate MIME type server-side, enforce size limits, and store outside the web root.
- Never store uploaded blob bytes or base64 payloads in PostgreSQL.
- Store files in S3-compatible object storage and only metadata/object keys in the database.
- Presigned upload/download URLs are short-lived and issued only after authorization.
- Object storage buckets are private by default; public-read buckets require an ADR.

## Rate Limiting

Auth endpoints and general API endpoints must be rate-limited. Standalone Bun + Elysia APIs use Redis-backed rate limiting because Redis is mandatory in that stack.

Rate limits cover IP, user, workspace/tenant, API client, auth/token endpoints, API key creation/token exchange, upload intent endpoints, public API route groups, and webhook endpoints where provider retry behavior allows it.

Rate-limited responses return HTTP `429` with `Retry-After`. Protected production endpoints fail closed or degrade according to documented project policy when Redis is unavailable; do not silently fall back to per-process production limits.

## Sensitive Data And Browser Environment

Rules:
- Never log PII to monitoring or logs.
- Mask tokens, card numbers, and sensitive fields in all log output.
- Tokens/secrets from `.env` must never appear in source code, API responses, browser bundles, logs, or plaintext database rows.
- Hashed API keys, hashed refresh tokens, and revocation records may be stored server-side.
- JWT private keys, refresh-token secrets, webhook secrets, storage credentials, and telemetry write tokens are secrets and must not be browser-exposed.
- Session replay is disabled by default and requires explicit approval plus masking proof.
- Health/readiness endpoints must not expose env vars, secret names, credentials, internal hostnames, image tags, or detailed config.
- Browser-exposed values use explicit public prefixes such as `VITE_PUBLIC_*`; secrets and private endpoints never use public prefixes.

## Dependency, Source Map, And Observability Security

Supply-chain rules:
- Run `bun audit` or equivalent in CI.
- Pin exact production dependencies and Docker tags.
- Commit and review Bun lockfile changes.
- Review dependencies that add install scripts, binary downloads, credential access, or telemetry.
- Prefer Alpine, slim, distroless, or the smallest production-suitable Docker image variants.
- Large non-minimal base images require an ADR or documented implementation note.
- Final JS/TS runtime images must not include development dependencies or development `node_modules`.
- Runtime images must not include package manager caches, test artifacts, Playwright browsers, coverage, `.env*`, `.git`, or development-only files.
- CI must review final image contents for dev dependency leakage and oversized/non-minimal base images.
- During active supply-chain incidents, CI may temporarily install with lifecycle scripts disabled.

Source map rules:
- Production source maps are not publicly served by default.
- If needed, upload privately to monitoring during CI/deploy and remove from public static artifact.
- Source map upload tokens are CI secrets.

Observability rules:
- Error monitoring is recommended before production for authenticated dashboards and backend APIs.
- PII scrubbing is mandatory before sending events to providers.
- OpenTelemetry spans, metrics, and logs must not include PII, bearer tokens, refresh tokens, API keys, cookies, raw request bodies, card data, emails, phone numbers, or free-text user content unless explicitly allowlisted and scrubbed.
- Metrics avoid high-cardinality labels such as user ID, workspace ID, request ID, resource ID, raw URL path, email, or phone.
- Audit logs are product/security records stored in PostgreSQL with retention policy.

## Queue And Worker Security

Background jobs are part of the security boundary.

Rules:
- Job payloads are schema-validated before enqueue and execution.
- Do not put bearer tokens, refresh tokens, API keys, raw cookies, plaintext secrets, or unredacted PII in job payloads.
- Store sensitive small records in PostgreSQL; store large/binary data in object storage; pass only IDs or object keys in jobs.
- Queue UIs such as Bull Board are admin RBAC-gated, network-restricted where possible, and hidden from public OpenAPI specs.
- Manual job replay requires admin RBAC, idempotency preservation, and an audit log entry.
- Worker logs and metrics follow API redaction and high-cardinality label rules.

## Idempotency And Replay Protection

Unsafe public API writes and external side effects support idempotency.

Rules:
- Require `Idempotency-Key` for public API writes that create irreversible or externally visible side effects.
- Store request hash, actor, route, and result so safe retries return the original response.
- Reusing the same key with different payload returns conflict.
- Webhook handlers store provider event IDs before queueing work and must not process duplicate event IDs twice.
- Signed webhook payloads use timestamp/replay protection when the provider supports it.

## See Also

- [Backend: Bun + Elysia](backend-bun-elysia.md)
- [Infrastructure: Docker Compose + Nginx](infra-docker-compose-nginx.md)
- [CI and Testing: TypeScript + React](ci-testing-typescript-react.md)
- [React + Vite Dashboard](web-react-vite-dashboard.md)

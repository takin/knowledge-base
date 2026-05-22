---
name: backend-bun-elysia
description: "Use when building, reviewing, or modifying standalone Backend API projects using Bun + Elysia, OpenAPI, JWT/JWKS, Drizzle/PostgreSQL, Redis/BullMQ, workers, webhooks, object storage, or Docker Compose API deployments."
---

# Backend API: Bun + Elysia

This skill applies the repository wiki standard for standalone backend APIs. Use it before writing or reviewing code for Bun services, Elysia HTTP routes, OpenAPI contracts, auth, PostgreSQL/Drizzle, Redis, BullMQ workers, webhooks, media flows, observability, and Docker Compose API deployments.

Primary source of truth: `wiki/engineering/tech-stacks/backend-bun-elysia.md`.

Related standards:
- `wiki/engineering/tech-stacks/security-web-app-baseline.md`
- `wiki/engineering/tech-stacks/infra-docker-compose-nginx.md`
- `wiki/engineering/tech-stacks/ci-testing-typescript-react.md`
- `wiki/engineering/tech-stacks/agent-skills-typescript-product.md`

## Required Skill Loading

Before implementing backend code, load and apply these skills when available:

- `typescript-best-practices` for all TypeScript and JavaScript code.
- `elysiajs` for API routes, route schemas, OpenAPI metadata, middleware, guards, and plugin patterns.
- `drizzle-orm` for schema, migrations, and queries.
- `postgresql-table-design` for table design, constraints, indexes, tenant modeling, and RLS decisions.

If one of these skills is unavailable, continue using the rules in this skill and state the gap in the final response.

## Stack Invariants

- Runtime is Bun.
- HTTP framework is Elysia.
- APIs expose REST JSON under versioned prefixes such as `/api/v1`.
- OpenAPI 3.1 is the official contract for dashboards, mobile apps, public SDKs, third parties, and CI drift checks.
- Use `@elysiajs/openapi`; do not use deprecated Swagger plugins.
- Scalar UI may be mounted at `/docs` when docs are enabled.
- Raw OpenAPI JSON may be exposed at `/docs/json` only when docs are enabled.
- Disable or auth-gate `/docs` and `/docs/json` in production.
- Eden Treaty is allowed only for internal TypeScript-only tooling. It must not be the official dashboard, mobile, or public API contract.
- Hide health, readiness, metrics, admin, internal worker, and consumer-irrelevant webhook endpoints from consumer OpenAPI specs.
- Every controller group defines OpenAPI tags and security metadata.

## Response Envelope

All successful responses use this shape:

```json
{
  "success": true,
  "data": {},
  "meta": {
    "page": 1,
    "pageSize": 20,
    "total": 450,
    "totalPages": 23
  }
}
```

All error responses use this shape:

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Please check the highlighted fields.",
    "details": [
      { "field": "email", "message": "Invalid email address" }
    ]
  }
}
```

Rules:
- Error codes are stable `SCREAMING_SNAKE_CASE` constants.
- Public API clients branch on `error.code`, not `message`.
- Never return raw Elysia, TypeBox, Zod, SQL, provider errors, exception messages, or stack traces.
- Every response includes or correlates with `X-Request-ID`.
- Baseline error codes include `BAD_REQUEST`, `VALIDATION_ERROR`, `UNAUTHENTICATED`, `FORBIDDEN`, `NOT_FOUND`, `CONFLICT`, `IDEMPOTENCY_CONFLICT`, `PAYLOAD_TOO_LARGE`, `UNSUPPORTED_MEDIA_TYPE`, `UNPROCESSABLE_ENTITY`, `RATE_LIMITED`, `INTERNAL_ERROR`, `SERVICE_UNAVAILABLE`, and `SERVICE_SHUTTING_DOWN`.

## Auth, CORS, And CSRF

- JWT is the standard auth mechanism for dashboard, mobile, machine-to-machine, and public API access.
- Use `jose` for JWT, JWK, and JWKS handling.
- Use asymmetric JWT signing in production.
- Prefer `EdDSA`; use `RS256` as compatibility fallback.
- Ban `HS256` for production SaaS/public APIs unless an ADR approves it.
- Every JWT protected header includes `alg`, `kid`, and `typ: "JWT"`.
- Verify issuer, audience, expiration, signature, and required claims on every protected request.
- Access tokens are short-lived.
- Refresh tokens use rotation and revocation.
- Machine tokens are scoped, revocable, and expire.
- API keys are hashed at rest and shown only once at creation.
- Expose public verification keys at `/.well-known/jwks.json`.

CORS rules:
- `CORS_ALLOWED_ORIGINS` is required for browser clients.
- Do not use wildcard `Access-Control-Allow-Origin: *` for authenticated APIs.
- Never combine wildcard origins with credentials.
- Production CORS origins must be exact origins, not broad suffixes.
- Do not reflect arbitrary `Origin` values.
- Preflight responses must not expose internal or debug headers.

Token transport rules:
- Machine and public API clients use `Authorization: Bearer <token>` or hashed API keys where specified.
- Dashboard and mobile clients use short-lived bearer access tokens plus refresh rotation.
- Do not store bearer access tokens in browser `localStorage` unless a security ADR accepts the risk.
- If cookies are used, they must be `HttpOnly`, `Secure` outside local development, and `SameSite=Lax` or `SameSite=Strict` unless an ADR approves `SameSite=None`.

CSRF rules:
- Bearer-token-only APIs that do not authenticate via ambient cookies do not require CSRF tokens.
- Cookie-authenticated unsafe methods require CSRF protection.
- CORS is not a CSRF defense.

## Authorization And Tenant Isolation

- RBAC is mandatory for SaaS APIs.
- JWT claims are identity hints, not the authorization source of truth.
- JWT verification proves token authenticity and basic identity.
- Workspace membership proves tenant context.
- RBAC authorizes user actions through roles and permissions.
- Scopes authorize machine and public API client actions.
- Resource ownership checks prevent cross-tenant access.
- Built-in roles are `owner`, `admin`, `member`, and `viewer`.
- Support custom roles in schema from day one, even if UI starts with built-ins.
- Tenant-owned tables include `workspace_id` or an equivalent tenant identifier.
- Tenant-owned service queries must filter by tenant or workspace identifier.
- Redis may cache resolved permissions only with short TTL and explicit invalidation.

## Validation, Database, And Migrations

- Elysia/TypeBox handles HTTP boundary validation.
- Zod may handle business or domain validation inside services.
- PostgreSQL is mandatory.
- Drizzle is the mandatory ORM/query builder.
- Use PgBouncer in transaction mode for staging and production.
- API and worker `DATABASE_URL` values point to PgBouncer, not directly to Postgres.
- Drizzle/Bun SQL pools are bounded per API or worker instance.
- Raw SQL strings are banned unless using Drizzle's parameterized `sql` helper with a documented reason.
- Every schema change is a committed Drizzle migration.
- Do not edit migrations after they have been applied to staging or production.
- Destructive migrations require backup, rollback plan, and approval.
- Use expand/contract for incompatible schema changes.
- Seed data must be deterministic and synthetic or anonymized.
- Never seed real PII, production secrets, real API keys, or real webhook secrets.

## Rate Limiting

- Redis-backed rate limiting is mandatory.
- Nginx performs coarse edge protection against abusive IP bursts.
- Elysia applies authoritative product and API policies because it sees user, workspace, API client, scopes, route group, and request cost.
- Controllers call an internal `src/lib/rate-limit.ts` abstraction, not `elysia-rate-limit` directly.
- Default algorithm is sliding window counter.
- Token bucket is allowed for burst-tolerant public or mobile APIs.
- Strict auth throttling applies to login, token refresh, password reset, API key creation, and upload intent endpoints.
- Expensive endpoints use weighted points or cost-based limits.
- Rate limit responses use HTTP `429`, `Retry-After`, `RateLimit-Limit`, `RateLimit-Remaining`, and `RateLimit-Reset` when feasible.
- Error code is `RATE_LIMITED`.
- Auth, token refresh, API key creation, unsafe writes, upload intent, and public API limits fail closed on Redis failure.
- Low-risk reads may fail open only when documented in a project ADR.

## Idempotency, Pagination, And Timeouts

- Idempotency is mandatory for unsafe writes and external or public API side effects.
- Use `Idempotency-Key` for side-effecting public `POST` requests.
- Same key plus same payload returns the original result.
- Same key plus different payload returns `IDEMPOTENCY_CONFLICT`.
- All list endpoints are paginated.
- Offset pagination is acceptable for bounded master data.
- Cursor pagination is required for fast-growing transactional data.
- Enforce a server-side maximum page size.
- JSON/form body size defaults to `1mb` unless an endpoint explicitly needs more.
- Direct binary upload through the API is banned by default.
- Normal HTTP request handling timeout is at most `30s`.
- DB query timeout defaults to `10s`.
- DB transaction timeout defaults to `20s` for HTTP handlers.
- External provider call timeout defaults to `5s`.
- Webhook synchronous handling timeout is `5s`; long work is queued.
- Use object-storage presigned uploads for media and large files.
- Propagate `AbortSignal` or equivalent cancellation into provider and database helpers where supported.

## Redis, BullMQ, And Workers

- Redis and BullMQ are mandatory from day one.
- Use BullMQ for email, webhooks, push fanout, media processing, imports/exports, provider retries, and scheduled background work.
- Queue names and job names are constants.
- Job payloads are schema-validated before enqueue and execution.
- Workers call the same `services/` layer as HTTP controllers.
- Jobs are idempotent.
- Trace context is propagated from HTTP requests into job metadata.
- Bull Board and queue UIs are admin-gated and hidden from public OpenAPI.
- Do not put bearer tokens, API keys, refresh tokens, raw cookies, plaintext secrets, or unredacted PII in job payloads.
- Store sensitive small records in PostgreSQL.
- Store large or binary data in object storage.
- Pass only IDs or object keys in jobs.

Queue taxonomy:
- `critical` for audit-sensitive internal work, billing finalization, and security notifications.
- `default` for normal async product work.
- `bulk` for imports, exports, media processing, and large fanout.
- `outbound` for email, push, customer webhooks, and third-party provider calls.

Job naming uses `domain.action.v1`, for example `media.process.v1` or `webhook.deliver.v1`.

Retry defaults:
- Attempts: `5`.
- Backoff: exponential.
- Initial delay: `1s`.
- Maximum delay: `5m`.
- Jitter: required.
- Failed terminal jobs are retained until operational retention expires.

Retry transient infrastructure or provider failures such as timeouts, `408`, `429`, `5xx`, temporary DNS failures, and safe serialization conflicts. Do not retry permanent validation errors, missing required records, invalid recipients, unsupported media types, or schema-version mismatches. Manual replay requires admin RBAC, audit log entry, and preserved idempotency.

## Media, Webhooks, Realtime, And Push

- Never store blob/file bytes in PostgreSQL.
- Store binary files in S3-compatible object storage such as MinIO, Cloudflare R2, or S3.
- PostgreSQL stores metadata, bucket, object key/path, status, checksum, and ownership.
- Buckets are private by default.
- Use presigned uploads and short-lived presigned downloads by default.
- Validate MIME type, size, purpose, quota, and permission before issuing upload URLs.
- File scanning, processing, and transcoding run in BullMQ workers.
- Inbound webhooks verify raw body signatures before JSON parsing.
- Store raw provider event metadata and provider event IDs for idempotency.
- Enqueue async webhook processing and return quickly.
- Outbound webhooks are delivered by workers, signed with HMAC, retried with exponential backoff, and tracked by delivery attempt.
- SSE is default for notifications, job progress, and dashboard live updates.
- WebSocket is by exception for bidirectional or high-frequency interactions.
- Push notifications use FCM, APNs, or Web Push through workers, not SSE/WebSocket.

## Observability, Logging, And Audit

- OpenTelemetry and Pino are mandatory for API and worker processes.
- Start telemetry before route and worker registration.
- Trace request, job, DB, Redis, storage, and provider timing.
- Emit metrics for request throughput, latency, errors, rate limits, Redis, DB pool, BullMQ jobs, queue lag, webhooks, and push failures.
- Use low-cardinality labels only.
- Never label metrics with user ID, workspace ID, API key, raw IP, raw URL, email, or phone number.
- Never log PII, bearer tokens, refresh tokens, API keys, cookies, card data, or raw request bodies by default.
- Audit logs are product/security records stored in PostgreSQL.
- Mandatory audit events include login/logout/session refresh, suspicious auth events, RBAC changes, member changes, API client/key lifecycle, media URL issuance/deletion, destructive actions, billing-sensitive actions, public API auth failures, and rate limit denials.

## Health, Shutdown, Deployment, And Docker

Required endpoints:

| Endpoint | Purpose | Auth |
| --- | --- | --- |
| `GET /health` | Liveness, process responds | Public to Nginx/load balancer |
| `GET /ready` | Dependency readiness | Internal/Nginx/load balancer |
| `GET /metrics` | Prometheus scrape if used | Internal only |
| `GET /docs` | Scalar docs | Dev/staging or auth-gated |
| `GET /.well-known/jwks.json` | Public JWT verification keys | Public |

Rules:
- `/health` must not check dependencies.
- `/ready` checks PostgreSQL, Redis, and mandatory storage.
- `/ready` returns `503` during graceful shutdown.
- Deployment uses Docker Compose behind Nginx on VPS by default.
- Baseline services are `nginx`, `api`, `worker`, `postgres`, `pgbouncer`, `redis`, `otel-collector`, `prometheus`, `loki`, `tempo`, `grafana`, and `certbot`.
- One Bun application image is used for both API and worker.
- API uses the default command; worker uses `command: ["bun", "run", "worker"]`.
- Use pinned `oven/bun:<version>`, never `latest`.
- Runtime containers run as non-root.
- Never bake secrets into images.
- Require `bun.lock` and `bun install --frozen-lockfile`.
- `.dockerignore` excludes `.env*`, `.git`, `node_modules`, `.venv`, coverage, test reports, Playwright reports, caches, and local artifacts.
- Nginx is the only host-facing service; do not expose API container ports to the host.
- Workers run as separate processes/containers, not inside the API process.

Graceful shutdown order:
- Set `isShuttingDown = true`.
- `/ready` returns `503`.
- Stop accepting new HTTP requests.
- Drain in-flight handlers.
- Reject new job enqueue attempts unless shutdown-safe.
- Stop BullMQ workers from taking new jobs.
- Let active jobs finish until worker timeout.
- Ensure DB transactions finish, commit, or rollback.
- Close SQL pool, BullMQ, Redis, logs, and telemetry.

## Public API Versioning

- Breaking changes require a new version prefix unless an explicit customer migration plan is approved.
- Default public API migration window is 90 days unless a project ADR changes it.
- Deprecated endpoints should return `Deprecation` and `Sunset` headers when practical.
- Public API CI blocks accidental breaking OpenAPI changes.

## Testing And CI

Required checks for backend API repositories:

- `bun install --frozen-lockfile`.
- Formatting check.
- Lint check.
- TypeScript typecheck.
- Unit tests for services and utilities.
- Integration tests with PostgreSQL and Redis.
- OpenAPI generation and drift check.
- Drizzle migration check.
- Docker build.
- API and worker command smoke tests.
- Auth, JWT, and JWKS tests.
- CORS and CSRF tests where applicable.
- RBAC/scope tests.
- Tenant isolation tests.
- Idempotency tests.
- Rate limit tests.
- Cache invalidation tests.
- Timeout and body-size tests.
- Media flow tests.
- Webhook signature and idempotency tests.
- Queue retry/backoff/DLQ/manual replay tests.
- Worker retry and failure tests.

## Anti-Patterns

Do not introduce these patterns:

- Eden Treaty as official dashboard, mobile, or public contract.
- `HS256` for production SaaS/public API JWTs without an ADR.
- JWT auth without RBAC, scopes, and resource ownership checks.
- Tenant-owned queries without `workspace_id` or equivalent filters.
- Raw Elysia, Zod, SQL, provider, or stack-trace errors in responses.
- Blob or base64 files in PostgreSQL.
- Public-read buckets by default.
- Redis as durable source of truth.
- Blocking HTTP on email, push, provider calls, or long webhook processing.
- Webhook JSON parsing before raw body signature verification.
- Missing OpenTelemetry/Pino from day one.
- Exposing API container ports to the host.
- Workers inside the API process in production.
- `oven/bun:latest`.
- Infinite retries or unbounded backoff.

## Implementation Workflow

When modifying or creating backend code:

1. Identify whether the surface is HTTP route, service, database schema, worker job, webhook, media flow, auth, infra, or observability.
2. Check the relevant invariant sections above before editing.
3. Keep route handlers thin; push business behavior into `services/` so HTTP and workers can share it.
4. Define route boundary schemas and OpenAPI metadata with the route.
5. Preserve the standard response envelope and error codes.
6. Add authorization checks at the service/resource boundary, not only middleware.
7. Add or update Drizzle migrations for schema changes.
8. Add tests for authz, tenant isolation, idempotency, rate limiting, webhooks, queues, and migrations when touched.
9. Run the smallest relevant verification commands available in the project.
10. In the final response, report which backend gates were verified and which were unavailable.

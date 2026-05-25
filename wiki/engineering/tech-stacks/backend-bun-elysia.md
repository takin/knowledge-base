# Backend Stack: Bun + Elysia

Updated: 2026-05-25
Status: Draft
Sources: Internal Tech Stacks draft (2026-04-15, updated 2026-05-23)
Platform: Backend API
Runtime: Bun
Framework: Elysia
Primary Use Case: Standalone SaaS APIs, SaaS administration, public APIs, mobile APIs, webhooks, async workers, media workflows, and independently deployable backend surfaces
Raw: [2026-04-15-backend-bun-elysia-stack.md](../../../raw/engineering/tech-stacks/2026-04-15-backend-bun-elysia-stack.md)

## Summary

This is the current backend API standard for standalone Bun services. It uses Elysia for HTTP, Oxc/Oxlint/Oxfmt for JavaScript/TypeScript tooling, OpenAPI 3.1 as the official client contract, JWT/JWK/JWKS with `jose` for auth, RBAC/scopes for authorization, backend-owned SaaS administration for tenants/users/subscriptions/soft locks, PostgreSQL 18+/Drizzle/PgBouncer for durable data, Redis 8+/BullMQ for distributed coordination and workers, S3-compatible storage for media, and OpenTelemetry/Pino for observability.

The stack has two product profiles:
- `internal-api` for dashboards, mobile apps, internal operators, and first-party machine clients.
- `public-api` for external developers, partners, customer systems, and public machine clients.

Both profiles expose REST JSON under `/api/v1`, generate OpenAPI 3.1, use JWT/JWKS/RBAC/scopes, require Redis 8+ and BullMQ from day one, and deploy behind Nginx with Docker Compose.

## Runtime And Toolchain

Backend API projects use Bun for installs, scripts, runtime, tests, and Docker builds. The JavaScript/TypeScript quality baseline is the Oxc toolchain family.

Rules:
- Use `oxc` as the baseline compiler/tooling family for JavaScript and TypeScript backend tooling.
- Use `oxlint` as the default linter for Elysia backend projects.
- Use `oxfmt` as the default formatter for Elysia backend projects.
- `oxlint` does not replace TypeScript typechecking; `tsc --noEmit` remains mandatory.
- ESLint is not the default backend linter. Add it only as a targeted exception when `oxlint` cannot cover a concrete risk.
- Prettier and Biome are not the default backend formatters. Add them only with a documented project exception.
- Oxfmt configuration must live in project formatter config, not ad hoc CI-only CLI flags.

Required script contract:

```json
{
  "scripts": {
    "typecheck": "tsc --noEmit",
    "lint": "oxlint",
    "lint:fix": "oxlint --fix",
    "format": "oxfmt",
    "format:check": "oxfmt --check"
  }
}
```

## API Contract

OpenAPI is the official contract for dashboard, mobile, public SDK, third-party, and CI drift workflows. Eden Treaty is allowed only for internal TypeScript-only tooling and must not be the official dashboard/mobile/public contract.

Rules:
- Use `@elysiajs/openapi`, not deprecated Swagger plugins.
- Mount Scalar UI at `/docs` when docs are enabled.
- Raw OpenAPI JSON is exposed at `/docs/json` only when docs are enabled.
- Disable or auth-gate `/docs` and `/docs/json` in production.
- Version APIs by URL prefix: `/api/v1`, `/api/v2`.
- Hide health, readiness, metrics, admin, internal worker, and consumer-irrelevant webhook endpoints from consumer OpenAPI specs.
- Every controller group defines OpenAPI tags and security metadata.

Public API versioning:
- Breaking changes require a new version prefix unless an explicit customer migration plan is approved.
- The default public API migration window is 90 days unless a project ADR changes it.
- Deprecated endpoints should return `Deprecation` and `Sunset` headers when practical.
- Public API CI blocks accidental breaking OpenAPI changes.

## Response Envelope And Errors

All responses use a standard envelope.

Success:

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

Error:

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

Baseline error codes include `BAD_REQUEST`, `VALIDATION_ERROR`, `UNAUTHENTICATED`, `FORBIDDEN`, `NOT_FOUND`, `CONFLICT`, `IDEMPOTENCY_CONFLICT`, `PAYLOAD_TOO_LARGE`, `UNSUPPORTED_MEDIA_TYPE`, `UNPROCESSABLE_ENTITY`, `RATE_LIMITED`, `INTERNAL_ERROR`, `SERVICE_UNAVAILABLE`, and `SERVICE_SHUTTING_DOWN`.

Rules:
- Error codes are stable `SCREAMING_SNAKE_CASE` constants.
- Public API clients branch on `error.code`, not `message`.
- Never return raw Elysia/TypeBox/Zod/SQL/provider errors or stack traces.
- Every response includes or correlates with `X-Request-ID`.

## Authentication, CORS, And CSRF

JWT is the standard auth mechanism for dashboard, mobile, machine-to-machine, and public API access. Use `jose` for JWT/JWK/JWKS.

Rules:
- Use asymmetric JWT signing in production.
- Prefer `EdDSA`; use `RS256` as compatibility fallback.
- Ban `HS256` for production SaaS/public API JWTs unless an ADR approves it.
- Every JWT includes `alg`, `kid`, and `typ: "JWT"` in the protected header.
- Verify issuer, audience, expiration, signature, and required claims on every protected request.
- Access tokens are short-lived.
- Refresh tokens use rotation and revocation.
- Machine tokens are scoped, revocable, and expire.
- API keys are hashed at rest and shown only once at creation.

CORS rules:
- `CORS_ALLOWED_ORIGINS` is required for browser clients.
- Do not use wildcard `Access-Control-Allow-Origin: *` for authenticated APIs.
- Never combine wildcard origins with credentials.
- Production CORS origins must be exact origins, not broad suffixes.

Token transport rules:
- Machine/public API clients use `Authorization: Bearer <token>` or hashed API keys where specified.
- Dashboard/mobile clients use short-lived bearer access tokens plus refresh rotation.
- Do not store bearer access tokens in browser localStorage unless a security ADR explicitly accepts the risk.
- If cookies are used, they must be `HttpOnly`, `Secure` outside local development, and `SameSite=Lax` or `SameSite=Strict` unless an ADR approves `SameSite=None`.

CSRF rules:
- Bearer-token-only APIs that do not authenticate via ambient cookies do not require CSRF tokens.
- Cookie-authenticated unsafe methods require CSRF protection.
- CORS is not a CSRF defense.

## Authorization And Tenant Isolation

RBAC is mandatory for SaaS APIs. JWT claims are identity hints, not authorization source of truth.

Authorization layers:
- JWT verification proves token authenticity and basic identity.
- Workspace membership proves tenant context.
- RBAC authorizes user actions through roles and permissions.
- Scopes authorize machine/public API client actions.
- Resource ownership checks prevent cross-tenant access.

Rules:
- Built-in roles are `owner`, `admin`, `member`, and `viewer`.
- Support custom roles in schema from day one, even if UI starts with built-ins.
- Tenant-owned tables include `workspace_id` or equivalent tenant identifier.
- Tenant-owned service queries must filter by tenant/workspace identifier.
- Redis may cache resolved permissions with short TTL and explicit invalidation.

## SaaS Administration Baseline

SaaS backend APIs include a generic product administration system for tenant self-service and platform operator administration. The dashboard renders administration UI, but the backend is authoritative for tenant state, tenant membership, subscription state, entitlements, soft locks, billing remediation, operator overrides, and audit records.

Administration surfaces:

| Surface | Actor | Scope | Boundary |
| --- | --- | --- | --- |
| Tenant self-service admin | Tenant owners and tenant admins | Their own tenant/workspace only | Normal authenticated API under tenant RBAC |
| Platform operator admin | Internal support, operations, finance, and platform admins | Cross-tenant operations | Separate admin route group with stronger RBAC and audit requirements |

Tenant self-service admin owns tenant settings, member lists, invitations, member removal, role changes, ownership transfer, subscription/plan visibility, entitlement visibility, billing remediation, and soft-lock recovery instructions.

Platform operator admin owns cross-tenant search/view, subscription and entitlement inspection, manual soft locks, suspension/restoration, billing recovery support, and approved operator overrides.

Rules:
- Platform operator routes live under a separate route group such as `/api/v1/admin/*`.
- Platform operator routes require stronger RBAC than tenant admin routes and are hidden from consumer OpenAPI specs unless intentionally documented.
- Platform operator changes require an audit reason, actor identity, timestamp, and before/after state where feasible.
- Tenant self-service routes operate on the resolved current tenant, not arbitrary tenant IDs.
- Billing provider events update subscription records through idempotent webhook processing and must not bypass application state transition rules.
- The backend, not the dashboard, decides whether an action is blocked by subscription state, entitlement limits, soft lock, suspension, or RBAC.

Core schema concepts are `tenants`, `tenant_members`, `tenant_invitations`, `subscription_plans`, `tenant_subscriptions`, `tenant_entitlements`, `tenant_soft_locks`, `billing_events`, and `audit_logs`.

Terminology rules:
- `tenant` is the architecture concept.
- `workspace_id` is acceptable as the default tenant key when product UX calls tenants "workspaces".
- A product must choose one tenant key convention, such as `workspace_id` or `tenant_id`, and use it consistently.
- Every tenant-owned business table includes the chosen tenant key.
- Tenant-scoped uniqueness includes the chosen tenant key.

Tenant state is product access posture; subscription state is billing lifecycle. Soft lock is an enforced access posture, not merely a subscription status.

Tenant states: `active`, `soft_locked`, `suspended`, `archived`, and `deleted_pending`.

Subscription states: `trialing`, `active`, `past_due`, `unpaid`, `expired`, and `cancelled`.

Soft lock default behavior is read-mostly and write-restricted.

Allowed during soft lock:
- login.
- view tenant, billing, subscription, invoice, usage, and entitlement state.
- update payment method, pay invoice, resume subscription, or open approved billing/support flows.
- export critical data if product policy allows.

Blocked during soft lock:
- creating product resources.
- inviting tenant users.
- using paid features.
- public API writes.
- background jobs that consume paid quota.
- webhook deliveries that represent paid usage.
- plan changes except approved recovery flows.

Stable error codes include `TENANT_SOFT_LOCKED`, `SUBSCRIPTION_PAST_DUE`, `SUBSCRIPTION_UNPAID`, `SUBSCRIPTION_EXPIRED`, and `PLAN_LIMIT_EXCEEDED`.

Tenant self-service API surface:

```text
GET    /api/v1/tenants/current
PATCH  /api/v1/tenants/current
GET    /api/v1/tenants/current/members
POST   /api/v1/tenants/current/invitations
PATCH  /api/v1/tenants/current/members/:memberId/role
DELETE /api/v1/tenants/current/members/:memberId
GET    /api/v1/tenants/current/subscription
GET    /api/v1/tenants/current/entitlements
POST   /api/v1/tenants/current/billing-portal-session
```

Platform operator API surface:

```text
GET    /api/v1/admin/tenants
GET    /api/v1/admin/tenants/:tenantId
GET    /api/v1/admin/tenants/:tenantId/subscription
POST   /api/v1/admin/tenants/:tenantId/soft-lock
DELETE /api/v1/admin/tenants/:tenantId/soft-lock
POST   /api/v1/admin/tenants/:tenantId/suspend
POST   /api/v1/admin/tenants/:tenantId/restore
```

Soft lock and entitlement enforcement happens in HTTP route guards, service methods, worker enqueue paths, worker execution paths, public API client checks, outbound webhook delivery jobs, and plan-aware rate limit policies.

Mandatory audit events include tenant lifecycle changes, member invitations/removals/role changes, ownership transfer, subscription plan/status changes, entitlement changes, soft-lock apply/clear, operator overrides, and billing recovery actions.

## Validation, Database, And Migrations

Elysia/TypeBox handles HTTP boundary validation. Zod handles business/domain validation inside services.

Database rules:
- PostgreSQL 18+ is mandatory.
- Drizzle is the mandatory ORM/query builder.
- Use PgBouncer in transaction mode for staging and production.
- API and worker `DATABASE_URL` values point to PgBouncer, not directly to Postgres.
- Drizzle/Bun SQL pools are bounded per API/worker instance.
- Raw SQL strings are banned unless using Drizzle's parameterized `sql` helper with a documented reason.

Drizzle schema organization:
- Small prototypes with only a few tables may use a single `src/db/schema.ts`.
- Backend API projects with multiple business domains split schema definitions by domain under `src/db/schema/`.
- Use `src/db/schema/index.ts` as the public schema entrypoint and re-export each domain file from it.
- Configure Drizzle Kit with `schema: "./src/db/schema"` or an explicit glob such as `schema: "./src/db/schema/*.ts"`.
- Initialize Drizzle with the merged schema object from the schema entrypoint, for example `import * as schema from "./schema"`.
- Split by business domain, not by object type. Prefer `auth.ts`, `workspaces.ts`, `billing.ts`, `media.ts`, `webhooks.ts`, and `audit.ts` over folders such as `tables/`, `relations/`, and `indexes/`.
- Keep each domain's tables, relations, indexes, and constraints close together unless the file becomes genuinely large.
- Tables that are tightly coupled should stay in the same domain file.
- Cross-domain foreign keys are allowed, but the ownership boundary should remain clear.
- Shared enums, reusable column helpers, and common timestamp/tenant columns may live in `common.ts` or `enums.ts`.

Primary key strategy:
- Choose primary keys by workload, exposure, and relational fan-out.
- Default to UUID v7 for entity tables that are not write-hot, especially when IDs are exposed through APIs, URLs, webhooks, exports, or cross-system integrations.
- Use `BIGINT GENERATED ALWAYS AS IDENTITY` for high-ingestion, append-heavy, transaction-heavy, or large fan-out tables where storage, index size, FK cost, and insert locality matter.
- UUID v7 keys must use PostgreSQL 18+'s built-in `uuidv7()` as the database-side default.
- Application code must not generate primary keys by default unless an ADR approves it.
- Do not use `serial`, `bigserial`, or UUID v4 for new primary keys unless an ADR documents the reason.
- High-ingestion tables that need public IDs use an internal BIGINT primary key plus `public_id UUID NOT NULL DEFAULT uuidv7() UNIQUE`.

Default UUID v7 candidates include users, workspaces, organizations, roles, API clients, products, customers, configuration tables, and reference/master data that is not write-hot. Default BIGINT identity candidates include orders, order items, order events, ledger entries, audit logs, webhook events, outbox messages, notification deliveries, job runs, metrics events, and append-only/high-volume logs.

Migration rules:
- Every schema change is a committed Drizzle migration.
- Do not edit migrations after they have been applied to staging or production.
- Destructive migrations require backup, rollback plan, and approval.
- Use expand/contract: add compatible structures, dual-write/backfill as needed, switch reads, then remove old structures only after old versions are gone.
- Seed data must be deterministic and synthetic/anonymized; never use real PII, production secrets, real API keys, or real webhook secrets.

## Rate Limiting

Redis-backed rate limiting is mandatory. The model is layered:
- Nginx performs coarse edge protection against abusive IP bursts.
- Elysia applies authoritative product/API policies because it sees user, workspace, API client, scopes, route group, and request cost.
- Redis stores distributed limiter state across API containers.

Implementation standard:
- Controllers call an internal `src/lib/rate-limit.ts` abstraction, not `elysia-rate-limit` directly.
- Default algorithm is sliding window counter.
- Token bucket is allowed for burst-tolerant public/mobile APIs.
- Strict auth throttling applies to login, token refresh, password reset, API key creation, and upload intent endpoints.
- Expensive endpoints use weighted points/cost-based limits.

Rate limit responses use `429`, `Retry-After`, `RateLimit-Limit`, `RateLimit-Remaining`, and `RateLimit-Reset` when feasible. Error code is `RATE_LIMITED`.

Redis failure policy:
- Auth, token refresh, API key creation, unsafe writes, upload intent, and public API limits fail closed.
- Low-risk reads may fail open only when documented in a project ADR.

Telemetry labels must stay low-cardinality. Do not label metrics with user ID, workspace ID, API key, raw IP, raw URL, email, or phone number.

## Idempotency, Pagination, And Timeouts

Idempotency is mandatory for unsafe writes and external/public API side effects. Use `Idempotency-Key` for side-effecting public `POST` requests. Same key plus same payload returns the original result; same key plus different payload returns conflict.

Pagination rules:
- All list endpoints are paginated.
- Offset pagination is acceptable for bounded master data.
- Cursor pagination is required for fast-growing transactional data.
- Enforce server-side maximum page size.

Timeout and body limit defaults:
- JSON/form body size: `1mb` unless endpoint requires more.
- Direct binary upload through API: banned by default.
- HTTP request handling timeout: `30s` maximum for normal endpoints.
- DB query timeout: `10s` default.
- DB transaction timeout: `20s` default for HTTP handlers.
- External provider call timeout: `5s` default.
- Webhook synchronous handling timeout: `5s`; long work is queued.

Use object-storage presigned uploads for media and large files. Propagate `AbortSignal` or equivalent cancellation into provider/database helpers where supported.

## Redis, BullMQ, And Queue Handling

Redis 8+ and BullMQ are mandatory from day one. Use BullMQ for email, webhooks, push fanout, media processing, imports/exports, provider retries, and scheduled background work.

Rules:
- Queue names and job names are constants.
- Job payloads are schema-validated before enqueue and execution.
- Workers call the same `services/` layer as HTTP controllers.
- Jobs are idempotent.
- Trace context is propagated from HTTP requests into job metadata.
- Bull Board or queue UIs are admin-gated and hidden from public OpenAPI.

Queue taxonomy:
- `critical`: audit-sensitive internal work, billing finalization, security notifications.
- `default`: normal async product work.
- `bulk`: imports, exports, media processing, large fanout.
- `outbound`: email, push, customer webhooks, third-party provider calls.

Job naming uses `domain.action.v1`, for example `media.process.v1` or `webhook.deliver.v1`. Payloads include `schema_version`, tenant/workspace context where applicable, actor context, idempotency key where applicable, and trace context. Do not put bearer tokens, API keys, refresh tokens, raw cookies, or unredacted PII in job payloads. Store sensitive small records in PostgreSQL; store large/binary data in object storage; pass only IDs or object keys in jobs.

Retry defaults:
- Attempts: `5`.
- Backoff: exponential.
- Initial delay: `1s`.
- Maximum delay: `5m`.
- Jitter: required.
- Failed terminal jobs are retained until operational retention expires.

Retry transient infrastructure/provider failures such as timeouts, `408`, `429`, `5xx`, temporary DNS failures, and safe serialization conflicts. Do not retry permanent validation errors, missing required records, invalid recipients, unsupported media types, or schema-version mismatches. Manual replay requires admin RBAC, audit log entry, and preserved idempotency.

## Media, Webhooks, Realtime, And Push

Never store blob/file bytes in PostgreSQL. Store binary files in S3-compatible object storage such as MinIO, Cloudflare R2, or S3. PostgreSQL stores metadata, bucket, object key/path, status, checksum, and ownership.

Media rules:
- Buckets are private by default.
- Use presigned uploads and short-lived presigned downloads by default.
- Validate MIME type, size, purpose, quota, and permission before issuing upload URLs.
- File scanning/processing/transcoding runs in BullMQ workers.

Webhook rules:
- Inbound webhooks verify raw body signatures before JSON parsing.
- Store raw provider event metadata and provider event IDs for idempotency.
- Enqueue async processing and return quickly.
- Outbound webhooks are delivered by workers, signed with HMAC, retried with exponential backoff, and tracked by delivery attempt.

Realtime rules:
- SSE is default for notifications, job progress, and dashboard live updates.
- WebSocket is by exception for bidirectional/high-frequency interactions.
- Push notifications use FCM/APNs/Web Push through workers, not SSE/WebSocket.

## Observability, Logging, And Audit

OpenTelemetry and Pino are mandatory for API and worker processes.

Observability covers:
- Pino JSON logs.
- OpenTelemetry traces for request, job, DB, Redis, storage, and provider timing.
- Metrics for request throughput/latency/errors, rate limit checks, Redis, DB pool, BullMQ jobs, queue lag, webhooks, and push failures.

Rules:
- Start telemetry before route and worker registration.
- Propagate trace context into BullMQ jobs.
- Use low-cardinality labels only.
- Never log PII, bearer tokens, refresh tokens, API keys, cookies, card data, or raw request bodies by default.

Audit logs are product/security records stored in PostgreSQL. Mandatory audit events include login/logout/session refresh, suspicious auth events, RBAC changes, member changes, API client/key lifecycle, media URL issuance/deletion, destructive actions, billing-sensitive actions, public API auth failures, and rate limit denials.

## Health, Shutdown, Deployment, And Docker

Required endpoints:

| Endpoint | Purpose | Auth |
| --- | --- | --- |
| `GET /health` | Liveness, process responds | Public to Nginx/load balancer |
| `GET /ready` | Dependency readiness | Internal/Nginx/load balancer |
| `GET /metrics` | Prometheus scrape if used | Internal only |
| `GET /docs` | Scalar docs | Dev/staging or auth-gated |
| `GET /.well-known/jwks.json` | Public JWT verification keys | Public |

`/health` must not check dependencies. `/ready` checks PostgreSQL 18+, Redis 8+, and mandatory storage. `/ready` returns `503` during graceful shutdown.

Graceful shutdown order:
1. Set `isShuttingDown = true`.
2. `/ready` returns `503`.
3. Stop accepting new HTTP requests.
4. Drain in-flight handlers.
5. Reject new job enqueue attempts unless shutdown-safe.
6. Stop BullMQ workers from taking new jobs.
7. Let active jobs finish until worker timeout.
8. Ensure DB transactions finish, commit, or rollback.
9. Close SQL pool, BullMQ, Redis, logs, and telemetry.

Deployment uses Docker Compose behind Nginx on VPS. Baseline services are `nginx`, `api`, `worker`, `postgres 18+`, `pgbouncer`, `redis 8+`, `otel-collector`, `prometheus`, `loki`, `tempo`, `grafana`, `certbot`, and `certbot-renew`.

Dockerfile standard:
- One Bun application image is used for both API and worker.
- Use pinned `oven/bun:<version>`, never `latest`.
- API uses the default command; worker uses `command: ["bun", "run", "worker"]`.
- The Nginx gateway image must be based on `fholzer/nginx-brotli:<pinned-version>` with Brotli enabled by default.
- Let's Encrypt issuance and renewal must be handled by Certbot sidecars with shared certificate volumes; do not install Certbot into the API or Nginx image.
- Runtime containers run as non-root.
- Never bake secrets into images.
- Require `bun.lock` and `bun install --frozen-lockfile`.
- `.dockerignore` excludes `.env*`, `.git`, `node_modules`, `.venv`, coverage, test reports, Playwright reports, caches, and local artifacts.

## Testing And CI

Required backend checks:
- `bun install --frozen-lockfile`.
- Oxfmt formatting check: `oxfmt --check`.
- Oxlint lint check: `oxlint`.
- TypeScript typecheck: `tsc --noEmit`.
- unit tests for services, utilities, schemas, and isolated domain logic under `tests/unit/`.
- integration tests with PostgreSQL 18+, Redis 8+, queues, workers, and HTTP routes under `tests/integration/`.
- E2E tests for critical user, webhook, and transactional flows under `tests/e2e/`.
- OpenAPI generation/drift check.
- Drizzle migration check.
- Docker build plus API/worker command smoke tests.
- auth/JWT/JWKS, CORS/CSRF where applicable, RBAC/scope, tenant isolation, tenant membership/role, tenant self-service admin RBAC, platform operator admin RBAC, subscription state transition, entitlement enforcement, soft lock allowed/blocked action, audit log, idempotency, rate limit, cache invalidation, timeout/body-size, media flow, webhook, queue retry/backoff/DLQ/manual replay, and worker retry/failure tests.

Test placement rules:
- `src/` contains production implementation code only.
- Do not colocate tests with implementation files in `src/`.
- Do not create adjacent `__tests__/` directories under `src/`.
- Do not place `*.test.ts`, `*.spec.ts`, `*.test.tsx`, or `*.spec.tsx` beside implementation files.
- Shared fixtures, factories, mocks, test containers, app harnesses, and custom assertions live under `tests/fixtures/`, `tests/factories/`, or `tests/helpers/`.

## Anti-Patterns

| Anti-pattern | Alternative |
| --- | --- |
| Eden Treaty as official dashboard/mobile/public contract | OpenAPI 3.1 + generated clients |
| `HS256` for production SaaS/public API JWTs | Asymmetric `jose` signing with JWK/JWKS and `kid` |
| JWT auth without RBAC/scope/resource checks | JWT identity + workspace membership + RBAC/scopes + ownership |
| Missing `workspace_id` filters | Tenant-scoped service queries |
| Dashboard-only tenant administration rules | Backend-owned tenant admin services and RBAC |
| Treating subscription state as tenant access state | Separate subscription lifecycle from tenant access posture |
| Soft lock only in frontend route guards | Backend guard, service, worker, public API, and webhook enforcement |
| Operator tenant changes without audit reason | Operator RBAC plus mandatory audit log reason |
| Mixing `tenant_id` and `workspace_id` inconsistently | One project-wide tenant key convention |
| Raw Elysia/Zod/SQL errors | Standard safe error envelope |
| ESLint as the default backend linter | Oxlint |
| Prettier or Biome as the default backend formatter | Oxfmt |
| Treating Oxlint as TypeScript typecheck | `tsc --noEmit` |
| Test files colocated with implementation code in `src/` | Dedicated top-level `tests/unit/`, `tests/integration/`, and `tests/e2e/` directories |
| Adjacent `__tests__/` directories under `src/` | Top-level `tests/` hierarchy with mirrored domain subfolders |
| Blob/base64 files in PostgreSQL | S3-compatible object storage + metadata/object keys |
| Public-read buckets by default | Private bucket + presigned URLs |
| Redis as durable source of truth | PostgreSQL source of truth, Redis cache/queue only |
| Blocking HTTP on email/push/provider calls | Enqueue BullMQ job and respond promptly |
| Webhook JSON parse before signature verification | Verify raw body first |
| Missing telemetry | OpenTelemetry from day one |
| Exposing API container ports to host | Nginx is the only host-facing service |
| Workers inside API process | Separate worker process/container |
| `oven/bun:latest` | Pinned Bun image |
| Infinite retries or unbounded backoff | Bounded attempts, capped exponential backoff, DLQ, alerting |

## See Also

- [Infrastructure: Docker Compose + Nginx](infra-docker-compose-nginx.md)
- [Security Baseline: Web Applications](security-web-app-baseline.md)
- [CI and Testing: TypeScript + React](ci-testing-typescript-react.md)
- [React + Vite Dashboard](web-react-vite-dashboard.md)
- [Mobile: React Native + Expo](mobile-react-native-expo.md)

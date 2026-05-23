# Tech Stacks Overview

Source URL: Internal draft
Collected: 2026-04-15
Published: 2026-04-15
Updated: 2026-05-23
Status: Draft
Scope: Generic taxonomy, adoption model, registry, repository structure, anti-patterns, and open questions

---

## Source Bundle Purpose

Tech Stacks is a generic, reusable engineering standards bundle. It is intentionally not tied to any company name. Teams can adopt one or more stack articles as binding standards for a product, and future stacks can coexist under the same taxonomy.

## Naming Convention

Stack source files and wiki articles use names that encode platform plus runtime or framework identity. Avoid generic file names such as `frontend.md` or `backend.md` because multiple frontend and backend stacks can coexist.

Examples:

| Stack Family | Current Stack | Future Examples |
|---|---|---|
| Web | `web-astro-landing`, `web-react-vite-dashboard` | `web-svelte-sveltekit`, `web-solid-solidstart`, `web-react-nextjs` |
| Backend API | `backend-bun-elysia` | `backend-go-chi`, `backend-go-fiber`, `backend-rust-axum` |
| Mobile | `mobile-react-native-expo` | `mobile-flutter` |
| Infrastructure | `infra-docker-compose-nginx` | `infra-kubernetes-nginx`, `infra-flyio` |

## Current Stack Registry

| Stack | Platform | Runtime | Framework / Core Tools | Source |
|---|---|---|---|---|
| Web: Astro Landing | Public marketing / SEO landing | Bun | Astro, MDX, Content Collections, Tailwind, Cloudflare Pages | `2026-05-22-web-astro-landing-stack.md` |
| Web: React + Vite Dashboard | Authenticated SaaS dashboard / API-only SPA | Bun | React, Vite, TanStack Router/Query/Form/Table, Zustand, OpenAPI client, Docker Compose, Nginx static runtime, Let's Encrypt via Certbot sidecar | `2026-05-22-web-react-vite-dashboard-stack.md` |
| Backend: Bun + Elysia | SaaS backend API / public API / mobile API / async workers | Bun | Elysia, OpenAPI, JWT/JWKS, RBAC, Drizzle, PostgreSQL 18+, PgBouncer, Redis 8+, BullMQ, S3-compatible storage, OpenTelemetry | `2026-04-15-backend-bun-elysia-stack.md` |
| Mobile: React Native + Expo | Mobile | Expo / JS | React Native, Expo Router, NativeWind, Maestro | `2026-04-15-mobile-react-native-expo-stack.md` |
| Infrastructure: Docker Compose + Nginx | Infrastructure | Docker | Docker Compose, `fholzer/nginx-brotli`, Let's Encrypt Certbot sidecar, PgBouncer, Vault, Kubernetes tier | `2026-04-15-infra-docker-compose-nginx-stack.md` |
| Security Baseline | Cross-stack | N/A | CSP, CSRF, XSS, SQL injection, rate limiting, dependency security | `2026-04-15-security-baseline.md` |
| CI and Testing: TypeScript + React | Delivery | Bun / GitHub Actions | Oxlint, Oxfmt, TypeScript strict, Vitest, Playwright, React Doctor | `2026-04-15-ci-testing-typescript-react.md` |
| Agent Skills | AI implementation workflow | N/A | Required coding-agent skills per stack | `2026-04-15-agent-skills.md` |

## 1. Philosophy

This document defines reusable technical stack standards. A team or company may adopt one or more stacks as binding standards for a product. Deviations from an adopted stack require an explicit Architecture Decision Record (ADR) and sign-off.

Three principles shape every choice here:

**Async-first.** The UI should never block waiting for data. Interactions feel instant through optimistic updates, background refetches, and progressive loading. Blocking spinners are a failure mode, not a loading pattern.

**Type-safe from edge to edge.** Schema → validation → ORM → Elysia route/OpenAPI contract → generated client — every boundary is typed. Runtime errors that a compiler could have caught are team failures, not bad luck.

**Minimal magic.** Prefer explicit over implicit. Avoid frameworks that hide what they are doing. If you cannot explain how a library works in five sentences, it probably does not belong in this stack.

---

## 18. Repository Structure

### 18.1 One repo per product

Each product adopting this standard lives in its own independent repository. There is no cross-product monorepo.

### 18.2 Monorepo within a product

A single product may contain a monorepo if it has distinct web, API, and/or mobile workloads. In that case:

```
apps/
  landing/      — Astro landing app when public marketing/SEO pages are part of the product repo
  dashboard/    — Vite React dashboard app when the product uses a separate backend API
  api/          — standalone API server when the product owns a backend API
  mobile/       — React Native / Expo app
packages/
  ui/           — shared Shadcn preset and base components
  schema/       — shared Zod schemas
  db/           — shared Drizzle schema and migrations
  jobs/         — shared BullMQ queue definitions when the product includes a backend API or worker workload
```

Use **Turborepo** for monorepo task orchestration within a product repo.

### 18.3 Standard directory layout (standalone backend API)

```
src/
  index.ts         — process entrypoint, listen(), SIGTERM handling, telemetry shutdown
  app.ts           — Elysia app composition, plugin registration, controller mounting
  controllers/     — Elysia route groups with prefix, OpenAPI metadata, auth/RBAC/webhook guards
  services/        — business logic; called by controllers and workers
  lib/
    db.ts          — Drizzle client instance (Bun native driver, via PgBouncer)
    redis.ts       — ioredis client
    logger.ts      — Pino logger setup
    telemetry.ts   — OpenTelemetry setup for API and workers
    env.ts         — env parsing / config normalization
    jwt.ts         — jose JWT signing and verification helpers
    jwks.ts        — JWK/JWKS key loading and public key export
    rbac.ts        — RBAC/scope authorization helpers
    rate-limit.ts  — Redis-backed rate limiting helpers
    idempotency.ts — idempotency-key storage and replay helpers
    storage.ts     — S3-compatible storage wrapper for MinIO/R2/S3
    email.ts       — Resend client + React Email render helper
    webhook-verify.ts — HMAC verification, replay protection helpers
  db/
    schema.ts      — Drizzle schema
    migrations/    — drizzle-kit migration files
  jobs/
    queues.ts      — BullMQ queue definitions
    workers/       — BullMQ worker files
  emails/          — React Email templates
  types/
    errors.ts      — shared API error codes (`SCREAMING_SNAKE_CASE`)
    api.ts         — response envelope / pagination types
    auth.ts        — principal, JWT claim, RBAC, and scope types
  utils/           — pure utility functions
tests/
  unit/            — services, pure utilities, schemas, and isolated domain logic
  integration/     — API, service, worker, and database integration tests
  e2e/             — end-to-end or webhook flow tests
  fixtures/        — reusable test data and static fixtures
  factories/       — deterministic test data builders
  helpers/         — test harnesses, dependency setup, and assertions
Dockerfile
docker-compose.yml
.env.example
```

Rules:
- Keep HTTP concerns in `controllers/` and business logic in `services/`. Do not embed business logic directly in Elysia route handlers.
- OpenAPI is the official client contract for dashboard, mobile, public, and machine clients. Eden Treaty is optional only for internal TypeScript tooling.
- JWT uses asymmetric JOSE keys with `kid` and a public JWKS endpoint. `HS256` is banned for production SaaS/public API JWTs unless an ADR approves it.
- RBAC is mandatory for SaaS APIs; public and machine clients use scopes.
- Tenant-owned service queries must filter by `workspace_id` or the equivalent tenant identifier.
- Webhook handlers live in a dedicated controller module and share verification helpers from `src/lib/webhook-verify.ts`.
- BullMQ workers call the same `services/` layer as the HTTP controllers. Do not duplicate business logic inside workers.
- OpenAPI tags, route metadata, and auth/security declarations are defined at the controller group level whenever possible.
- Health endpoints (`/health`, `/ready`) are defined close to the app bootstrap in `src/index.ts` or a dedicated system controller, but must remain outside auth middleware.
- `src/` contains production implementation code only; test files must not be colocated with real implementation code.
- Do not use adjacent `__tests__/` directories inside `src/` and do not place `*.test.*` or `*.spec.*` files beside implementation files.
- Unit, integration, and E2E tests live only under the top-level `tests/unit/`, `tests/integration/`, and `tests/e2e/` directories.
- Shared test helpers, fixtures, factories, mocks, and harnesses live under `tests/helpers/`, `tests/fixtures/`, or `tests/factories/`, not under `src/`.
- PostgreSQL 18+ is the baseline relational database and Redis 8+ is the baseline cache/queue coordination service.
- Primary keys are selected by workload, exposure, and relational fan-out: UUID v7 for normal API-facing entity tables, `BIGINT GENERATED ALWAYS AS IDENTITY` for high-ingestion/append-heavy/large fan-out tables.
- UUID v7 primary keys use PostgreSQL 18+'s built-in `uuidv7()` as the database-side default; application code does not generate primary keys unless an ADR approves it.
- High-ingestion tables that need public IDs use an internal BIGINT primary key plus `public_id UUID NOT NULL DEFAULT uuidv7() UNIQUE`.
- Media blobs are never stored in PostgreSQL. Store media in S3-compatible object storage and save only metadata/object keys in the database.
- API and worker processes must start OpenTelemetry and flush telemetry on shutdown.

Example tree for a webhook-heavy backend:

```
src/
  index.ts
  controllers/
    auth.controller.ts
    workspace.controller.ts
    webhook.controller.ts
    health.controller.ts
  services/
    auth.service.ts
    workspace.service.ts
    webhook.service.ts
    provider-sync.service.ts
  routes/
    auth.routes.ts
    workspace.routes.ts
    webhook.routes.ts
  jobs/
    queues.ts
    workers/
      webhook.worker.ts
      email.worker.ts
      provider-sync.worker.ts
  lib/
    db.ts
    redis.ts
    logger.ts
    telemetry.ts
    env.ts
    jwt.ts
    jwks.ts
    rbac.ts
    rate-limit.ts
    idempotency.ts
    email.ts
    storage.ts
    webhook-verify.ts
  db/
    schema.ts
    migrations/
  types/
    api.ts
    errors.ts
    webhook.ts
  utils/
    pagination.ts
tests/
  unit/
    services/
      webhook.service.test.ts
    utils/
      pagination.test.ts
  integration/
    api/
      webhook.test.ts
      workspace.test.ts
    workers/
      webhook.worker.test.ts
  e2e/
    flows/
      webhook-flow.test.ts
  fixtures/
  factories/
  helpers/
```

In this structure:
- `webhook.controller.ts` handles the HTTP boundary: verify source, validate headers/signature, persist raw event, enqueue job, return `200`.
- `webhook.worker.ts` performs the asynchronous business logic and calls `webhook.service.ts`.
- `provider-sync.service.ts` contains connector-specific logic that may be reused by both webhook processing and scheduled/background jobs.
- `idempotency.ts` centralizes duplicate-event protection so providers follow one consistent pattern.

Example tree for a CRUD-heavy backend:

```
src/
  index.ts
  controllers/
    auth.controller.ts
    customer.controller.ts
    order.controller.ts
    product.controller.ts
    health.controller.ts
  services/
    auth.service.ts
    customer.service.ts
    order.service.ts
    product.service.ts
    inventory.service.ts
  routes/
    auth.routes.ts
    customer.routes.ts
    order.routes.ts
    product.routes.ts
  lib/
    db.ts
    redis.ts
    logger.ts
    telemetry.ts
    env.ts
    jwt.ts
    jwks.ts
    rbac.ts
    rate-limit.ts
    idempotency.ts
    storage.ts
    email.ts
  db/
    schema.ts
    migrations/
  types/
    api.ts
    errors.ts
    customer.ts
    order.ts
    product.ts
  utils/
    pagination.ts
    cursor.ts
tests/
  unit/
    services/
      customer.service.test.ts
      order.service.test.ts
      product.service.test.ts
    utils/
      cursor.test.ts
  integration/
    api/
      customer.test.ts
      order.test.ts
      product.test.ts
    db/
      order-repository.test.ts
  e2e/
    flows/
      order-lifecycle.test.ts
  fixtures/
  factories/
  helpers/
```

In this structure:
- each controller owns the HTTP contract for one domain resource and delegates to its matching service
- each service owns validation-adjacent business rules, transactional writes, and orchestration across related entities
- shared cross-domain concerns such as inventory reservation stay in a dedicated service instead of being duplicated in `order.service.ts` and `product.service.ts`
- pagination helpers stay in `utils/` so every list endpoint uses the same cursor/page conventions and response envelope shape

---

## 21. Anti-Patterns (Banned)

The following are explicitly prohibited in all products adopting this standard:

| Anti-pattern | Why banned | Alternative |
|---|---|---|
| `useEffect` for data fetching | Creates race conditions, stale state, waterfalls | TanStack Query |
| `useEffect` to sync derived state | Creates infinite loops and unnecessary renders | `useMemo`, selectors |
| Flat dotted TanStack route files like `store.$slug.tsx` | Harder to navigate as route trees grow; weakens domain grouping | Use folder/sub-folder route files like `store/$slug.tsx` |
| Defining page components inline in route modules | Couples route wiring to page implementation and can hurt code splitting | Keep route modules thin; import pages from `src/pages/**` |
| Importing `Route` from route modules into page components | Creates circular imports and can hurt code splitting | Use `getRouteApi('/route/path')` in page files |
| Hardcoded secrets or API keys | Security risk | `.env` |
| Baking secrets into Docker image | Secrets leak via `docker inspect` | Runtime env injection |
| Custom project-specific Dockerfile shape | Makes deployments, CI, debugging, and agent-generated changes inconsistent | Use the standard multi-stage Bun Dockerfile contract |
| Installing only production dependencies before build | Vite, TypeScript, and adapter tooling often live in `devDependencies`, causing builds to fail | Install full dependencies before build, then production dependencies in runtime images when applicable |
| Copying `.env` into Docker image | Bakes secrets into immutable artifacts | Inject env at runtime via Compose or Vault |
| Running `vite preview` in production | Preview server is not production-grade | Nginx static runtime |
| Running Bun/Node static server in production for dashboards | Adds runtime surface the SPA does not need | Nginx serves `dist/` directly |
| Deploying dashboard without Docker Compose | Makes setup and deployment inconsistent | Standard Docker Compose contract |
| Deploying dashboard without Nginx | Skips the standard TLS, compression, headers, and cache baseline | Nginx static runtime |
| Proxying Nginx to a dashboard app server | Adds an unnecessary runtime hop for static SPA | Nginx serves `dist/` directly |
| Missing dashboard SPA fallback | Deep links 404 on refresh | `try_files $uri $uri/ /index.html` |
| Immutable caching for dashboard `index.html` | Users keep stale app shells after deploy | no-cache / must-revalidate |
| Public production source maps | Exposes source code and implementation details | Private source map upload only |
| Importing charts/editors/upload clients in dashboard app shell | Bloats initial JavaScript and hurts startup | Route-level or component-level code splitting |
| Ignoring dashboard bundle budgets | Performance regressions accumulate silently | Bundle analysis and CI budget checks |
| Baking TLS certificates into Docker images | Cert rotation requires image rebuild and leaks secrets | Let's Encrypt volume + Certbot sidecar |
| Installing Certbot into dashboard image | Bloats runtime and mixes TLS renewal with static serving | Certbot sidecar |
| Installing Certbot into the main Nginx/API image | Mixes TLS lifecycle with runtime serving and causes rebuild-driven renewal | Certbot issuance/renewal sidecars with shared volumes |
| Using stock `nginx` for the main gateway | Brotli is missing or inconsistently configured | `fholzer/nginx-brotli:<pinned-version>` |
| Disabling Brotli by default | Larger payloads and inconsistent compression baseline | `NGINX_BROTLI_ENABLED=on` with gzip fallback |
| Serving production dashboard app traffic over HTTP | Exposes sessions and user data to network interception | Port 80 ACME challenge + HTTPS redirect only |
| Missing HTTPS redirect from port 80 to 443 | Users can remain on insecure HTTP | Always return 301 to HTTPS except ACME challenge path |
| Manual-only certificate renewal | Certificates expire during normal operation | Cron/systemd renewal script |
| Renewing certificates without reloading Nginx | Nginx keeps serving the old certificate | Reload Nginx after successful renewal |
| `any` type without comment | Defeats TypeScript | `unknown` + type narrowing |
| Blocking full-page spinners | Poor UX | Suspense + skeletons |
| Ad hoc auth implementation | Security risk, inconsistent token lifecycle and revocation | Standard JWT/JWKS auth module with `jose`, refresh rotation, RBAC, and scopes |
| Eden Treaty as the official dashboard/mobile/public API contract | TypeScript-only contract does not serve mobile, public, or CI drift needs | OpenAPI 3.1 + generated clients |
| `HS256` for production SaaS/public API JWTs | Verifiers can forge tokens if they hold the shared secret | Asymmetric `jose` signing with JWK/JWKS and `kid` |
| JWT checks without RBAC/scope/resource ownership | Authenticated identity is not sufficient authorization | JWT identity + workspace membership + RBAC/scopes + tenant resource checks |
| Missing `workspace_id` on tenant-owned tables | Cross-tenant data leakage risk | Tenant-scoped tables and queries |
| Storing blob files or base64 in PostgreSQL | Bloats DB, breaks backups, and bypasses object storage controls | S3-compatible object storage + DB metadata/object keys |
| Public-read media bucket by default | Bypasses API authorization | Private buckets + short-lived presigned URLs |
| Missing OpenTelemetry in backend API or workers | Production incidents lack trace and metric evidence | OpenTelemetry from day one |
| High-cardinality metric labels | Explodes Prometheus/telemetry storage | Use low-cardinality route/status/job/provider labels |
| Running BullMQ workers inside the API process | API scaling and worker concurrency become coupled | Separate worker process/container |
| Third-party component library over Shadcn | Stack fragmentation | Shadcn UI |
| Custom toast/snackbar implementation | Fragments transient feedback behavior and accessibility | Sonner via Shadcn |
| Raw SQL strings in application code | Injection risk, loses type safety | Drizzle query builder |
| Postgres driver other than `bun:sql` without reason | Adds unnecessary dependency | Default to Bun native driver |
| Using UUID v4 for new primary keys | Random UUIDs fragment indexes and lose time-ordering | UUID v7 via PostgreSQL 18+ `uuidv7()` |
| Using UUID primary keys for high-ingestion tables by default | Larger PK/FK indexes and higher write amplification | BIGINT identity PK plus optional UUID v7 `public_id` |
| Using `serial` or `bigserial` for new primary keys | Legacy sequence shorthand, less explicit than SQL-standard identity | `BIGINT GENERATED ALWAYS AS IDENTITY` when BIGINT is chosen |
| Node.js / npm / pnpm in any form | Stack inconsistency | BunJS |
| `dangerouslySetInnerHTML` with unsanitized input | XSS vector | DOMPurify + CSP headers |
| Wildcard credentialed CORS | Allows unintended origins to make authenticated requests | Explicit per-environment origin allowlist |
| Production CSP with wildcard sources | Makes XSS and data exfiltration easier | Restrictive CSP with reviewed provider sources |
| Raw backend errors in UI | Leaks internals and confuses users | Map machine codes to safe user messages |
| Logging PII to Sentry or console | Privacy risk, compliance violation | Scrub before logging |
| Session replay by default | Captures sensitive user behavior and data | Explicit approval plus masking rules |
| Animations longer than 400ms | Feels sluggish, not lightweight | 150–300ms target range |
| Scaffolding a project from scratch or copying from another | Non-standard structure, onboarding friction | `bunx --bun @tanstack/cli@latest create` |
| `docker compose restart api` or `docker compose up api` | Zero-instance gap during restart → dropped requests | Use the deterministic deployment script/blue-green flow (§20.11) |
| Defining `add_header` in a child Nginx location without repeating security headers | Parent security headers disappear because Nginx does not inherit them | Repeat the complete security header set in each header-setting location |
| Adding a service worker by default | Stale-cache bugs and broken deploys are common when offline behavior is not intentionally designed | Add PWA only when installability or offline behavior is a product requirement |
| Ad hoc frontend-only auth/session logic in frontend-owned products | Creates inconsistent auth boundaries and security assumptions | Delegate to the external backend auth boundary and generated OpenAPI client |
| `maxUnavailable: 1` in K8s | K8s removes old pod before new one is proven healthy | Set `maxUnavailable: 0` always |
| Missing `preStop: sleep 5` in K8s | 502s during the 1–5s deregister propagation gap | Mandatory on every K8s container spec |
| `terminationGracePeriodSeconds` shorter than drain time | SIGKILL kills in-flight requests mid-response | Set to 60s minimum |
| `/ready` returning 200 during shutdown | Nginx keeps routing to a draining pod | Set `isShuttingDown = true` → return 503 on SIGTERM |
| Missing `proxy_next_upstream` in Nginx | One 502 from a deregistering pod reaches the user | Always configure retry on 502/503/504 |
| No load test during staged deploy | Zero-downtime not actually verified | Run `vegeta` at 100 RPS during every staged deploy |
| `kubectl rollout restart` without PodDisruptionBudget | Multiple pods restart simultaneously → capacity drops | Set PDB `minAvailable: 2` alongside `maxUnavailable: 0` |
| Connecting API or worker directly to PostgreSQL 18+ (bypassing PgBouncer) | Exhausts Postgres connection limit under load | Route all DB traffic through PgBouncer |
| Unbounded Drizzle connection pool | Multiple instances × unlimited = Postgres crash | Set `DB_POOL_MAX` per instance |
| PgBouncer in session or statement mode | Negates pooling benefits at high concurrency | Use `transaction` mode only |
| Returning raw Zod or Elysia errors to the client | Leaks internal schema, inconsistent DX | Use the standard error envelope (§20.2) |
| Blocking HTTP response on email send | Adds latency, fails request if email provider is down | Queue via BullMQ, respond immediately |
| Serving files directly from storage bucket (public read) | Security risk, no access control | Generate presigned URLs server-side |
| Caching transactional records in Redis | Stale data for financial/order records | Never cache transactional data — query fresh |
| Hardcoding compression settings in Nginx image | Cannot tune per environment | All Nginx params via `.env` + envsubst |
| Exposing API container port to host (using `ports:`) | Bypasses Nginx, exposes uncompressed/unsecured traffic | Use `expose:` for internal only; Nginx owns `ports:` |
| Processing webhook payloads synchronously | Exceeds provider timeout → provider retries → duplicates | Queue via BullMQ, respond 200 within 500ms |
| Loose `@tanstack/*` production dependency ranges | Can resolve to compromised versions during supply-chain incidents | Pin explicit patched versions |
| Ignoring `GHSA-g7cv-rxg3-hmpx` indicators | Malware can exfiltrate cloud, GitHub, npm, SSH, and Vault credentials | Check for `@tanstack/setup`, `router_init.js`, and affected versions |
| Reusing CI secrets after suspected compromised install | Install-time malware may have stolen accessible secrets | Rotate credentials and audit logs |
| Running lifecycle scripts during active supply-chain incident | Malicious install scripts execute before code review catches them | Temporarily use `bun install --ignore-scripts` |
| Accepting webhook payloads without signature verification | Allows forged events from any caller | Layer 3: HMAC-SHA256 verification (§20.16.4) |
| Re-queueing a webhook event ID that was already received | Duplicate processing of payment/order events | Idempotency check against `webhookEvents` table |
| Using `===` to compare HMAC signatures | Timing attack allows signature forgery | `crypto.timingSafeEqual` always |
| Reading JSON body before HMAC verification | HMAC must be computed over raw bytes | Read raw body first, parse JSON after signature passes |
| Sharing HMAC secrets or API keys across providers | One compromise exposes all webhook integrations | One secret per provider, namespaced in `.env` |
| Skipping replay protection | Valid captured request can be replayed hours later | Reject events older than 5 minutes via timestamp check |
| Using `useState` or Zustand for filter/search/pagination state | State lost on hard refresh, not shareable via URL | `nuqs` with TanStack Router adapter |
| Reading raw `searchParams` strings and casting manually | Type-unsafe, no coercion, breaks on missing params | `nuqs` `parseAs*` typed parsers |

---

## 22. Open Questions

All architectural decisions are resolved. This document is ready for wiki ingestion and ratification as the binding standard.

---

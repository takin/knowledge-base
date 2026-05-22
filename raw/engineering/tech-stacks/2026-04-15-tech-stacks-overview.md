# Tech Stacks Overview

Source URL: Internal draft
Collected: 2026-04-15
Published: 2026-04-15
Updated: 2026-05-22
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
| Web | `web-react-tanstack-start` | `web-svelte-sveltekit`, `web-solid-solidstart`, `web-react-nextjs` |
| Backend API | `backend-bun-elysia` | `backend-go-chi`, `backend-go-fiber`, `backend-rust-axum` |
| Mobile | `mobile-react-native-expo` | `mobile-flutter` |
| Infrastructure | `infra-docker-compose-nginx` | `infra-kubernetes-nginx`, `infra-flyio` |

## Current Stack Registry

| Stack | Platform | Runtime | Framework / Core Tools | Source |
|---|---|---|---|---|
| Web: React + TanStack Start | Web / frontend-owned or fullstack | Bun | React, TanStack Start, TanStack Router/Query/Form/Table/Virtual | `2026-04-15-web-react-tanstack-start-stack.md` |
| Backend: Bun + Elysia | Backend API | Bun | Elysia, Eden Treaty, OpenAPI, BullMQ, PostgreSQL | `2026-04-15-backend-bun-elysia-stack.md` |
| Mobile: React Native + Expo | Mobile | Expo / JS | React Native, Expo Router, NativeWind, Maestro | `2026-04-15-mobile-react-native-expo-stack.md` |
| Infrastructure: Docker Compose + Nginx | Infrastructure | Docker | Docker Compose, Nginx Brotli, PgBouncer, Vault, Kubernetes tier | `2026-04-15-infra-docker-compose-nginx-stack.md` |
| Security Baseline | Cross-stack | N/A | CSP, CSRF, XSS, SQL injection, rate limiting, dependency security | `2026-04-15-security-baseline.md` |
| CI and Testing: TypeScript + React | Delivery | Bun / GitHub Actions | ESLint, TypeScript strict, Vitest, Playwright, React Doctor | `2026-04-15-ci-testing-typescript-react.md` |
| Agent Skills | AI implementation workflow | N/A | Required coding-agent skills per stack | `2026-04-15-agent-skills.md` |

## 1. Philosophy

This document defines reusable technical stack standards. A team or company may adopt one or more stacks as binding standards for a product. Deviations from an adopted stack require an explicit Architecture Decision Record (ADR) and sign-off.

Three principles shape every choice here:

**Async-first.** The UI should never block waiting for data. Interactions feel instant through optimistic updates, background refetches, and progressive loading. Blocking spinners are a failure mode, not a loading pattern.

**Type-safe from edge to edge.** Schema → validation → ORM → server function → client — every boundary is typed. Runtime errors that a compiler could have caught are team failures, not bad luck.

**Minimal magic.** Prefer explicit over implicit. Avoid frameworks that hide what they are doing. If you cannot explain how a library works in five sentences, it probably does not belong in this stack.

---

## 18. Repository Structure

### 18.1 One repo per product

Each product adopting this standard lives in its own independent repository. There is no cross-product monorepo.

### 18.2 Monorepo within a product

A single product may contain a monorepo if it has distinct web, API, and/or mobile workloads. In that case:

```
apps/
  web/          — TanStack Start app (frontend-owned or fullstack)
  api/          — standalone API server if explicitly needed (rare — prefer server functions for fullstack web apps)
  mobile/       — React Native / Expo app
packages/
  ui/           — shared Shadcn preset and base components
  schema/       — shared Zod schemas
  db/           — shared Drizzle schema and migrations
  jobs/         — shared BullMQ queue definitions (if background jobs are used)
```

Use **Turborepo** for monorepo task orchestration within a product repo.

### 18.3 Standard directory layout (single TanStack Start app)

The web stack supports both fullstack products and frontend-owned products. "Frontend-owned" means a TanStack Start app that may still run SSR, prerendered routes, BFF-style integration points, and the Nitro app server, but does not own a product database, auth server, or server-side domain model. It does not mean static-only hosting unless the web stack later adds a separate static export deployment contract.

```
src/
  routes/         — TanStack Router route modules and route-level wiring only
    __root.tsx
    index.tsx
    <segment>/
      route.tsx
      index.tsx
      $param.tsx
    _<group>/
      route.tsx
      index.tsx
    $.tsx
  pages/          — page components imported by route modules
    HomePage.tsx
    products/
      ProductsPage.tsx
      ProductDetailPage.tsx
  components/
    ui/           — Shadcn components
    layout/       — app layout and shell components
    shared/       — cross-feature shared components
  features/
    <domain>/
      components/
      hooks/
      queries/
      mutations/
      schema.ts
      types.ts
  lib/
    auth.ts       — Better Auth server config for fullstack products
    auth-client.ts — Better Auth client config or delegated auth client boundary
    csp.ts        — CSP nonce accessor
    env.ts        — environment parsing / config normalization
    utils.ts      — shared utilities
  db/             — fullstack products only
    index.ts
    schema.ts
    migrations/
  server/         — fullstack server functions and services
    functions/
    services/
  styles/
    app.css       — global CSS and Tailwind theme entry
  router.tsx
  routeTree.gen.ts — generated by TanStack Router; never hand-edit
public/
nginx/
  Dockerfile
  nginx.conf.template
  entrypoint.sh
tests/
  e2e/            — Playwright tests
Dockerfile
docker-compose.yml
.env.example
```

Rules:
- `src/routes/**` contains route modules only: route declarations, loaders, guards, search validation, head metadata, server route handlers, and route-level wiring.
- Page components live under `src/pages/**`. Route modules import page components; they do not define page components inline.
- Page files use `getRouteApi('/route/path')` for typed params, search, loader data, and route context. Do not import `Route` from the route module into a page component.
- Use folder/sub-folder route files such as `src/routes/store/$slug.tsx`; flat dotted route files such as `store.$slug.tsx` are banned.
- Domain-specific reusable code lives in `src/features/<domain>/`. Pages compose features; features must not depend on pages.
- Fullstack database code lives in `src/db/`; frontend-owned products with no owned persistence omit database services.
- Server-only functions and services live in `src/server/` unless they are TanStack Start server route handlers owned by `src/routes/**`.
- `public/` is required even when empty so the standard Dockerfile remains identical across products.
- Deployed TanStack Start apps use the Nitro app-server baseline behind Nginx. Static export is not part of this baseline.

### 18.4 Standard directory layout (standalone backend API)

```
src/
  index.ts         — Elysia app bootstrap, plugin registration, listen(), SIGTERM handling
  routes/          — route modules grouped by resource/domain
  controllers/     — Elysia route groups with prefix, detail tags, auth/webhook guards
  services/        — business logic; called by controllers and workers
  lib/
    db.ts          — Drizzle client instance (Bun native driver, via PgBouncer)
    redis.ts       — ioredis client
    logger.ts      — Pino logger setup
    env.ts         — env parsing / config normalization
    storage.ts     — MinIO / R2 client wrapper
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
  utils/           — pure utility functions
tests/
  integration/     — API, service, and database integration tests
  e2e/             — end-to-end or webhook flow tests
Dockerfile
docker-compose.yml
.env.example
```

Rules:
- Keep HTTP concerns in `controllers/` and business logic in `services/`. Do not embed business logic directly in Elysia route handlers.
- Webhook handlers live in a dedicated controller module and share verification helpers from `src/lib/webhook-verify.ts`.
- BullMQ workers call the same `services/` layer as the HTTP controllers. Do not duplicate business logic inside workers.
- OpenAPI tags, route metadata, and auth/security declarations are defined at the controller group level whenever possible.
- Health endpoints (`/health`, `/ready`) are defined close to the app bootstrap in `src/index.ts` or a dedicated system controller, but must remain outside auth middleware.

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
    env.ts
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
    idempotency.ts
tests/
  integration/
    webhook.test.ts
    workspace.test.ts
  e2e/
    webhook-flow.test.ts
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
    env.ts
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
  integration/
    customer.test.ts
    order.test.ts
    product.test.ts
  e2e/
    order-lifecycle.test.ts
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
| REST API layer for fullstack TanStack Start app | Adds a roundtrip and indirection | TanStack Start server functions unless a standalone API is explicitly required |
| Flat dotted TanStack route files like `store.$slug.tsx` | Harder to navigate as route trees grow; weakens domain grouping | Use folder/sub-folder route files like `store/$slug.tsx` |
| Defining page components inline in route modules | Couples route wiring to page implementation and can hurt code splitting | Keep route modules thin; import pages from `src/pages/**` |
| Importing `Route` from route modules into page components | Creates circular imports and can hurt code splitting | Use `getRouteApi('/route/path')` in page files |
| Hydrating every SSR component immediately | Wastes startup JavaScript and hydration work on non-critical UI | Use `Hydrate` boundaries for below-the-fold or intent-gated UI |
| Deferring primary navigation, checkout, search, or accessibility-critical controls | Makes expected interactions late or broken | Keep immediate-interaction UI hydrated normally |
| Hiding `Hydrate` behind wrapper components | Compiler may not split child chunks | Render imported `Hydrate` directly or use `split={false}` |
| Calling hooks directly inside extracted `Hydrate` JSX | Compiler extraction can move hook execution incorrectly | Move hook logic into a child component |
| Treating frontend-owned TanStack Start apps as static-only by default | The baseline still assumes SSR/prerendered routes served by the Nitro app server | Use the standard Nitro app-server deployment unless a static export contract is added |
| Hardcoded secrets or API keys | Security risk | `.env` |
| Baking secrets into Docker image | Secrets leak via `docker inspect` | Runtime env injection |
| Custom project-specific Dockerfile shape | Makes deployments, CI, debugging, and agent-generated changes inconsistent | Use the standard multi-stage Bun Dockerfile contract |
| Installing only production dependencies before build | TanStack Start, Vite, TypeScript, and adapter tooling often live in `devDependencies`, causing builds to fail | Install full dependencies in `deps`/`build`, then production dependencies in `runtime` |
| Copying `.env` into Docker image | Bakes secrets into immutable artifacts | Inject env at runtime via Compose or Vault |
| `any` type without comment | Defeats TypeScript | `unknown` + type narrowing |
| Blocking full-page spinners | Poor UX | Suspense + skeletons |
| Custom auth implementation | Security risk, maintenance burden | Better Auth |
| Third-party component library over Shadcn | Stack fragmentation | Shadcn UI |
| Custom toast/snackbar implementation | Fragments transient feedback behavior and accessibility | Sonner via Shadcn |
| `"use client"` in Shadcn components | Next.js directive, incorrect in TanStack Start | Remove after every `shadcn add` |
| Raw SQL strings in application code | Injection risk, loses type safety | Drizzle query builder |
| Postgres driver other than `bun:sql` without reason | Adds unnecessary dependency | Default to Bun native driver |
| Node.js / npm / pnpm in any form | Stack inconsistency | BunJS |
| `dangerouslySetInnerHTML` with unsanitized input | XSS vector | DOMPurify + CSP headers |
| Logging PII to Sentry or console | Privacy risk, compliance violation | Scrub before logging |
| Animations longer than 400ms | Feels sluggish, not lightweight | 150–300ms target range |
| Scaffolding a project from scratch or copying from another | Non-standard structure, onboarding friction | `bunx --bun @tanstack/cli@latest create` |
| `docker compose restart app` or `docker compose up app` | Zero-instance gap during restart → dropped requests | Use `--scale` + `--no-recreate` sequence (§20.11) |
| Missing `X-CSP-Nonce` proxy header | App and CSP header use different or missing nonce values | Always forward `$csp_nonce` to the app |
| Missing `ssr.nonce` in TanStack Router | SSR scripts/styles may not receive the request CSP nonce | Pass `getCspNonce()` into router SSR config |
| Returning `""` for missing client CSP nonce | Causes SSR/client hydration mismatch | Return `undefined` from the client nonce reader |
| Defining `add_header` in a child Nginx location without repeating security headers | Parent security headers disappear because Nginx does not inherit them | Repeat the complete security header set in each header-setting location |
| Exposing TanStack Start directly to the host with `ports:` | Bypasses Nginx security headers, CSP nonce propagation, compression, caching, rate limiting, and upstream retry behavior | App uses `expose:` only; Nginx is the only host-facing gateway |
| Adding a service worker by default | Stale-cache bugs and broken deploys are common when offline behavior is not intentionally designed | Add PWA only when installability or offline behavior is a product requirement |
| Ad hoc frontend-only auth/session logic in frontend-owned products | Creates inconsistent auth boundaries and security assumptions | Delegate to the external backend auth boundary or use Better Auth in fullstack products |
| `maxUnavailable: 1` in K8s | K8s removes old pod before new one is proven healthy | Set `maxUnavailable: 0` always |
| Missing `preStop: sleep 5` in K8s | 502s during the 1–5s deregister propagation gap | Mandatory on every K8s container spec |
| `terminationGracePeriodSeconds` shorter than drain time | SIGKILL kills in-flight requests mid-response | Set to 60s minimum |
| `/ready` returning 200 during shutdown | Nginx keeps routing to a draining pod | Set `isShuttingDown = true` → return 503 on SIGTERM |
| Missing `proxy_next_upstream` in Nginx | One 502 from a deregistering pod reaches the user | Always configure retry on 502/503/504 |
| No load test during staged deploy | Zero-downtime not actually verified | Run `vegeta` at 100 RPS during every staged deploy |
| `kubectl rollout restart` without PodDisruptionBudget | Multiple pods restart simultaneously → capacity drops | Set PDB `minAvailable: 2` alongside `maxUnavailable: 0` |
| Connecting app directly to Postgres (bypassing PgBouncer) | Exhausts Postgres connection limit under load | Route all DB traffic through PgBouncer |
| Unbounded Drizzle connection pool | Multiple instances × unlimited = Postgres crash | Set `DB_POOL_MAX` per instance |
| PgBouncer in session or statement mode | Negates pooling benefits at high concurrency | Use `transaction` mode only |
| Returning raw Zod or Elysia errors to the client | Leaks internal schema, inconsistent DX | Use the standard error envelope (§20.2) |
| Blocking HTTP response on email send | Adds latency, fails request if email provider is down | Queue via BullMQ, respond immediately |
| Serving files directly from storage bucket (public read) | Security risk, no access control | Generate presigned URLs server-side |
| Caching transactional records in Redis | Stale data for financial/order records | Never cache transactional data — query fresh |
| Hardcoding compression settings in Nginx image | Cannot tune per environment | All Nginx params via `.env` + envsubst |
| Exposing app container port to host (using `ports:`) | Bypasses Nginx, exposes uncompressed/unsecured traffic | Use `expose:` for internal only; Nginx owns `ports:` |
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

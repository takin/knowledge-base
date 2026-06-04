# Web Stack: TanStack Start OAuth/OIDC App

Updated: 2026-06-04
Status: Draft
Sources: Internal TanStack Start OAuth/OIDC stack draft (2026-06-04)
Platform: Web / authenticated SaaS app
Runtime: Bun
Framework: TanStack Start
Primary Use Case: Authenticated SaaS web applications that require OAuth2/OIDC, SSO, server-managed sessions, SSR, protected server functions, BFF behavior, or first-party app-server resource management
Raw: [2026-06-04-web-tanstack-start-oauth-oidc-stack.md](../../../raw/engineering/tech-stacks/2026-06-04-web-tanstack-start-oauth-oidc-stack.md)

## Summary

TanStack Start is the default web frontend stack for SaaS applications that require OAuth2 or OIDC authentication through a separate authorization service.

Use this stack when the web app must initiate OAuth/OIDC login, receive callback routes, exchange authorization codes server-side, manage a secure app session, protect server functions, act as a Backend-for-Frontend, or directly own product resources from a server runtime.

Better Auth is the current self-hosted auth service standard, but this stack stays provider-neutral and can use any standards-compliant OAuth2/OIDC provider. The auth service owns login, SSO, credential handling, consent, issuer metadata, token issuance, JWKS, token revocation, and account-level auth policy. The TanStack Start app owns product session, tenant resolution, route protection, app authorization, server functions, and resource access.

React + Vite remains the dashboard standard for static API-only SPAs. The dividing line is runtime ownership: React + Vite Dashboard is static and consumes a backend API; TanStack Start OAuth/OIDC App has a production server runtime and participates in the authentication/resource boundary.

## When To Use This Stack

Use TanStack Start OAuth/OIDC App when:

- OAuth2 Authorization Code Flow with PKCE is required.
- OIDC Authorization Code Flow with PKCE is required.
- SSO is delegated to a separate auth service.
- The app must handle OAuth/OIDC callback routes.
- The app must exchange authorization codes server-side.
- The browser should not store access tokens or refresh tokens.
- The app needs Redis-backed HttpOnly cookie sessions.
- The app needs authenticated server functions or server routes.
- The app needs SSR with session-aware route loading.
- The app should own its product database and business logic directly.
- The app should act as a BFF for an existing backend API.

Do not use this stack for public marketing/SEO landing pages, static API-only dashboards, mobile apps, or standalone public backend APIs unless the product intentionally consolidates those surfaces into this app server.

## Relationship To React + Vite Dashboard

React + Vite Dashboard is still the default for static authenticated dashboards with a mandatory separate backend API.

Use React + Vite Dashboard when:

- The dashboard is a static SPA.
- The backend API is mandatory and authoritative.
- The dashboard consumes generated OpenAPI clients.
- The dashboard does not need SSR.
- The dashboard does not need server functions.
- The dashboard does not need OAuth/OIDC callback handling.
- The dashboard does not manage server-side app sessions.
- Production should be a static `dist/` artifact served directly by Nginx.

Use TanStack Start OAuth/OIDC App when the app needs server-side authentication flow handling, server-managed session state, BFF behavior, or full-stack resource ownership.

## Runtime And Toolchain

Core stack:

- Bun for package management, scripts, builds, tests, and runtime.
- TanStack Start as the full-stack React framework.
- React 19+ with React Compiler enabled for production builds.
- TanStack Router for routing.
- TanStack Query for server state.
- TanStack Form with Zod for submit-style forms.
- TanStack Table for interactive tables.
- TanStack Store or a minimal client store only for client-owned UI state.
- PostgreSQL and Drizzle when the app owns product data.
- Redis as a mandatory runtime dependency for live sessions, OAuth callback state, PKCE verifier storage, CSRF state, rate limiting, distributed invalidation, and BullMQ queues.
- BullMQ for long-running async processes and retryable background jobs.
- Tailwind CSS v4+ and Shadcn UI for styling and components.
- Radix UI primitives through Shadcn, Sonner for toasts, Recharts for chart patterns, and `lucide-react` for default icons unless the design preset chooses otherwise.

Tooling target:

- Strict TypeScript.
- Oxlint for linting.
- Oxfmt for formatting.
- Vitest for unit and integration tests.
- Testing Library for component tests.
- Playwright for E2E tests.
- React Doctor or equivalent React health diagnostics when available.

Required script contract:

```json
{
  "scripts": {
    "dev": "vinxi dev",
    "build": "vinxi build",
    "start": "vinxi start",
    "worker": "tsx src/server/worker.ts",
    "typecheck": "tsc --noEmit",
    "lint": "oxlint .",
    "format": "oxfmt",
    "format:check": "oxfmt --check",
    "test": "vitest run",
    "test:e2e": "playwright test"
  }
}
```

If the TanStack CLI scaffold emits ESLint defaults, migrate to the Oxc baseline unless a project ADR keeps ESLint for a concrete plugin requirement.

## OAuth2/OIDC Model

The default authentication model is Authorization Code Flow with PKCE.

Rules:

- The browser does not handle credentials.
- Login starts from the TanStack Start app and redirects to the auth service.
- The auth service authenticates the user and applies SSO policy.
- The auth service redirects back to a registered callback route in the TanStack Start app.
- The callback route validates `state`.
- The callback route exchanges the authorization code server-side.
- The app creates its own application session.
- The browser receives only an application session cookie.
- Access tokens and refresh tokens are not exposed to browser JavaScript.

Login flow:

```text
Browser -> TanStack Start app
TanStack Start app -> create state + PKCE verifier
TanStack Start app -> redirect to auth service authorization endpoint
Auth service -> login / SSO / consent
Auth service -> redirect back to app callback with code
TanStack Start callback route -> validate state
TanStack Start callback route -> exchange code server-side
TanStack Start app -> create app session cookie
Browser -> authenticated app route
```

Logout flow:

```text
Browser -> TanStack Start logout route
TanStack Start app -> clear app session
TanStack Start app -> optionally revoke provider token
TanStack Start app -> optionally redirect to provider logout
TanStack Start app -> redirect to public/login route
```

## Better Auth Provider Assumptions

Better Auth is the current self-hosted auth service standard. The TanStack Start app is not the credential-login app; it is an OAuth/OIDC client of the central auth service.

The auth service is expected to provide:

- OAuth2/OIDC authorization server capability.
- Authorization endpoint and token endpoint.
- Stable issuer URL.
- OIDC discovery document where applicable.
- JWKS endpoint for asymmetric token verification.
- Registered OAuth client for each SaaS app.
- Explicit redirect URI allowlist.
- Login page and consent page when required.
- Short-lived access tokens.
- Refresh token rotation when refresh tokens are issued.
- Token revocation support.
- Token introspection support where required.
- Rate limiting for auth and OAuth endpoints.

The stack can use another OAuth2/OIDC provider if it supports the same authorization code, issuer, JWKS, client registration, token, and revocation requirements.

## Session Model

The default session model is a Redis-backed server-managed application session with an HttpOnly cookie.

Redis is mandatory from day one because live session invalidation, logout, callback state, PKCE verifier storage, CSRF state, rate limiting, and distributed runtime support are part of the authentication boundary.

PostgreSQL may store session audit trails, session history, device/session lists, or long-lived security investigation records, but Redis is the primary live session store.

Rules:

- Do not store access tokens or refresh tokens in `localStorage`.
- Do not store access tokens or refresh tokens in `sessionStorage`.
- Avoid browser-held bearer tokens for this stack.
- Store OAuth/OIDC tokens server-side only when the app needs them.
- Browser receives only an app session cookie.
- Cookies are `HttpOnly`.
- Cookies are `Secure` outside local development.
- Cookies use `SameSite=Lax` by default.
- `SameSite=Strict` is allowed where OAuth callback behavior and product UX permit it.
- `SameSite=None` requires a documented cross-site need and an ADR.
- Session data may include stable user ID, provider subject, tenant/workspace ID, role summary, session version, and safe display fields.
- Session data must not expose raw access tokens, refresh tokens, API keys, provider secrets, or database credentials to client JavaScript.
- Session invalidation must support logout and emergency revocation.
- Cookie-only sessions require an ADR and security review.
- Database-backed live sessions are not the default for this stack.

Server functions and server routes must treat the app session as the authentication boundary for browser users.

## Resource Ownership Profiles

This stack supports two resource ownership profiles.

| Profile | Default | Use Case | Resource Boundary |
| --- | --- | --- | --- |
| Full-stack app server | Yes for new SaaS products | The web app owns product data and business logic | TanStack Start accesses DB/services directly server-side |
| BFF | Yes when backend already exists | A separate backend API remains authoritative | TanStack Start holds app session and calls backend server-side |

Full-stack app server profile:

- TanStack Start owns database access.
- TanStack Start owns product services.
- TanStack Start owns tenant-scoped business rules.
- TanStack Start owns server functions for reads and mutations.
- TanStack Start owns SaaS administration surfaces unless separated by product decision.
- PostgreSQL and Drizzle are the default data layer.
- Redis owns live sessions, callback state, PKCE verifier storage, CSRF state, rate limiting state, distributed invalidation, and BullMQ queues.
- BullMQ workers own long-running async processing for product workflows that belong to the app.

BFF profile:

- Standalone backend API remains authoritative.
- TanStack Start owns OAuth/OIDC callback and app session.
- TanStack Start calls backend API from the server.
- Browser does not receive backend access tokens.
- Backend API still enforces authorization, tenant isolation, validation, and business rules.
- Generated OpenAPI clients may still be used server-side or shared with client-safe wrappers.
- TanStack Start may enqueue local app jobs for app-owned workflows, but backend-owned long processes should use backend-owned job APIs.

Do not mix these profiles accidentally. If both are used, document which domains are owned by TanStack Start and which domains are owned by the backend API.

## Async Processing: BullMQ And Redis

BullMQ backed by Redis is mandatory for long-running work.

Server functions and server routes may enqueue jobs, but long-running work must run in separate worker processes. HTTP requests should return quickly with a `jobId`, operation handle, or accepted status.

Use BullMQ for bulk imports and exports, report generation, file processing, image/document transformation, notification fanout, billing reconciliation, webhook retry/delivery, slow or retryable third-party API calls, AI/LLM generation, large data sync jobs, scheduled maintenance jobs, and any process that needs retry, backoff, deduplication, progress tracking, or durable failure handling.

Request lifecycle rules:

- Server functions validate input, check session, check tenant/RBAC/entitlement, create durable job records when needed, enqueue BullMQ jobs, and return quickly.
- Server functions must not run long processes inline.
- Server functions must not hold HTTP requests open while waiting for long jobs.
- Server functions must not run retry loops for third-party side effects inline.
- Server functions must not depend on the browser tab staying open.

Job rules:

- Jobs are tenant/user scoped.
- Job payloads contain IDs and small metadata, not secrets, raw cookies, access tokens, refresh tokens, large blobs, or unredacted PII.
- Workers re-check product authorization when feasible, or operate on an authorization snapshot created by the enqueueing server function.
- Externally visible side effects require idempotency keys or deduplication.
- Retries use bounded attempts and backoff.
- Worker failures are normalized into product-level job status.
- Users see job status and progress through app-owned APIs, not BullMQ internals.
- Worker logs and metrics follow the same redaction rules as server functions.

Standard async flow:

```text
Client -> TanStack Start server function
Server function -> validate + authz + create job record
Server function -> enqueue BullMQ job in Redis
Server function -> return jobId
Worker -> process job
Worker -> update job status/progress
Client -> poll job status or receive realtime update
```

## Project Structure

Recommended structure:

```text
src/
  routes/
    __root.tsx
    index.tsx
    auth/
      login.tsx
      callback.tsx
      logout.tsx
    app/
      route.tsx
      dashboard.tsx
  components/
    ui/
    layout/
    shared/
  features/
    <domain>/
  lib/
    auth/
      oauth.ts
      session.ts
      csrf.ts
    db/
      client.ts
      schema/
    env.ts
    query-client.ts
    errors.ts
  server/
    middleware/
    jobs/
    worker.ts
    services/
    repositories/
    security/
  styles/
    app.css
public/
tests/
  unit/
  integration/
  e2e/
  fixtures/
  factories/
  helpers/
Dockerfile
.dockerignore
docker-compose.yml
.env.example
```

Rules:

- `src/routes/**` owns routing, route-level loading, callback routes, and route composition.
- `src/lib/auth/**` owns OAuth/OIDC client logic, session helpers, token storage, and CSRF helpers.
- `src/server/**` owns server-only product logic.
- `src/server/jobs/**` owns BullMQ queue definitions, processors, and job payload schemas.
- Database code is allowed only in server-only modules.
- Client components must not import server-only modules.
- Generated API code, when used, lives under `src/lib/api/generated/**` and is never hand-edited.
- Tests live in the top-level `tests/` hierarchy, not beside implementation files.
- Do not create adjacent `__tests__/` directories inside `src/`.

## Shadcn Initialization

Shadcn UI is the default component baseline, but it must use the TanStack Start template and an approved preset.

Default scaffold intent:

```bash
bunx @tanstack/cli@latest create --no-examples \
  --add-ons=nitro,table,store,form,compiler,drizzle \
  --package-manager=bun \
  --deployment=nitro \
  <project_name>
```

Default Shadcn intent:

```bash
bunx --bun shadcn@latest init --preset <approved-preset> --template start
```

Rules:

- Always ask the human for the preset code before Shadcn initialization.
- Do not run plain `shadcn init`.
- Use `--template start` for TanStack Start projects.
- Never decode, fetch, or inspect preset codes manually; pass preset codes directly to the Shadcn CLI.

## Data And State

Rules:

- TanStack Query owns server state.
- TanStack Router owns route state and route params.
- URL search params own filters, search, sorting, pagination, selected tabs, and shareable views.
- TanStack Form owns form state.
- TanStack Store or Zustand may be used only for client-owned UI state.
- `useState` is allowed only for strictly local, ephemeral component UI state such as open/closed toggles, transient input affordances, hover/pressed state, and one-component visual details.
- Do not use scattered component-level `useState` for state that is shared across components, route-relevant, URL-shareable, server-derived, form-owned, workflow-owned, async-job-owned, or persisted.
- Do not store API responses in global client state.
- Do not use `useEffect` for primary data fetching.
- Query keys include every input that changes returned data.
- Mutations invalidate the narrowest affected query keys.
- Mutations that can duplicate side effects must not retry blindly.
- Use server-side pagination for unbounded transactional data.
- Do not use client-side filtering or sorting for unbounded transactional data.

State ownership rules:

- Server data belongs to TanStack Query.
- URL/shareable view state belongs to route search params.
- Submit-style form state belongs to TanStack Form.
- Route and session context belongs to TanStack Router route context.
- Cross-component client UI state belongs to TanStack Store or Zustand.
- Async job status belongs to app-owned job status APIs backed by job records, not component state.
- Local ephemeral visual state may use `useState`.

## Authorization And Tenant Isolation

Authentication proves who the actor is. It does not authorize product actions by itself.

Rules:

- JWT claims and ID token claims are identity hints, not the source of truth for product authorization.
- Resolve tenant/workspace context server-side.
- Check tenant membership before tenant-scoped reads.
- Check RBAC before mutations.
- Check subscription, entitlement, soft-lock, and suspension state before protected feature use.
- Tenant-owned tables include a consistent tenant key such as `tenant_id` or `workspace_id`.
- Tenant-scoped uniqueness includes the chosen tenant key.
- Platform operator routes require stronger RBAC and audit logging.
- Cross-tenant operator actions require reason, actor identity, timestamp, and before/after state where feasible.
- Session display fields are not authorization proof.
- Provider tokens are not authorization proof beyond identity and scope verification.

Built-in product roles should include `owner`, `admin`, `member`, and `viewer`. Support custom roles in schema from day one if the SaaS product may need enterprise permissions.

The auth service is the source of truth for identity. Product database is the source of truth for tenant membership, product RBAC, entitlements, subscription state, soft locks, and operator overrides. Better Auth organization data may bootstrap or mirror tenant identity, but product authorization resolves in the app database by default.

## Security

Authentication and token handling:

- Never expose provider access tokens or refresh tokens to browser JavaScript.
- Never hardcode OAuth client secrets or provider secrets.
- Never bake secrets into Docker images.
- Store hashed refresh-token references or encrypted token material server-side according to product policy.
- Access tokens are short-lived.
- Refresh tokens use rotation and revocation when issued.
- Verify issuer, audience, expiration, signature, and required claims when validating provider tokens.
- Use JWKS with `kid` support for asymmetric token verification.

Cookie and CSRF rules:

- Cookie-authenticated unsafe methods require CSRF protection.
- CORS is not a CSRF defense.
- CORS allowlists are explicit and environment-specific.
- Do not use wildcard origins with credentials.
- Do not reflect arbitrary `Origin` values.

Browser and rendering rules:

- CSP is required for HTML responses.
- Production CSP must not contain wildcard sources.
- Production `script-src` must not use `unsafe-inline` unless reviewed.
- Do not use unsanitized `dangerouslySetInnerHTML`.
- Encode user-generated content before rendering.
- Do not render raw backend, provider, SQL, Zod, OAuth, or stack errors to users.
- Public browser config uses a clear public prefix.
- Private auth URLs, client secrets, token secrets, database URLs, and telemetry write keys remain server-only.

Observability rules:

- Never log tokens, cookies, API keys, raw provider responses, or credential material.
- PII scrubbing is mandatory before sending events to monitoring providers.
- Session replay is disabled by default and requires explicit approval plus masking proof.
- Production source maps are not publicly served.
- If source maps are needed, upload them privately during CI/deploy and remove them from public artifacts.

## Deployment: Docker Compose And Nginx

TanStack Start apps have a production server runtime. They are not static SPA artifacts.

Rules:

- Do not deploy this stack as static-only Nginx `dist/`.
- Do not use the React + Vite dashboard static Nginx profile.
- Production uses Docker Compose for VPS-first deployments unless the project has a different approved deployment platform.
- Nginx is the host-facing gateway.
- TanStack Start runtime runs behind Nginx.
- Redis is mandatory in every environment.
- Worker runtime is mandatory when the app has any long-running or retryable async workflows.
- TLS terminates at Nginx.
- Certbot sidecar owns Let's Encrypt issuance and renewal for VPS deployments.
- Runtime container receives secrets via environment or approved secrets manager, not image layers.
- Health and readiness endpoints must not expose env vars, secret names, credentials, internal hostnames, image tags, or detailed config.
- Runtime images must not include source files, tests, caches, local `.env` files, or public source maps unless explicitly required and reviewed.

Baseline Docker Compose services:

- Nginx gateway.
- TanStack Start app runtime.
- Worker runtime using the app image with a worker command.
- Redis.
- PostgreSQL when the app owns product data.
- Certbot sidecar for VPS Let's Encrypt deployments.

Default deployment is Docker Compose with Nginx gateway and the standard TanStack Start/Nitro-compatible production runtime. Platform-specific adapters are allowed by ADR with documented runtime constraints, secret handling, session behavior, and rollback path.

## Performance Budget

| Asset / Metric | Budget | Rule |
| --- | ---: | --- |
| Authenticated app shell JavaScript | <= 250 KB gzip | Initial shell only; heavy routes split |
| Initial CSS | <= 80 KB gzip | Tailwind and component baseline included |
| SSR blocking work | Minimal | Only session, route-critical data, and required bootstrap |
| Route chunks | Project-specific | Heavy modules lazy-loaded |

Rules:

- Do not import charts, rich editors, upload clients, analytics, chat, session replay, or marketing widgets into the root app shell unless required.
- Use route-level code splitting for heavy product areas.
- Use streaming or deferred loading where appropriate.
- Avoid blocking SSR on non-critical dashboard widgets.
- Run bundle analysis before production launch and after adding large dependencies.
- Fonts must use `font-display: swap`.
- Public production source maps are banned.

## Testing And CI

Required checks:

- `bun install --frozen-lockfile`.
- `bun run format:check`.
- `bun run lint`.
- `bun run typecheck`.
- Vitest unit and component tests.
- Integration tests for auth callback, session helpers, CSRF, server functions, and data access.
- Integration tests for BullMQ enqueueing, worker processors, retry behavior, idempotency, and failure normalization.
- Playwright E2E tests.
- React Doctor or equivalent React health scan when available.
- Bundle budget or bundle analysis before production launch.
- `bun run build`.
- Docker build.
- Security artifact scan.

Required E2E coverage before production:

- Login redirect.
- OAuth/OIDC callback success.
- Invalid callback state rejection.
- Logout.
- Protected route redirect when unauthenticated.
- Protected server function rejection when unauthenticated.
- Primary authenticated dashboard happy path.
- One representative tenant-scoped data flow.
- One representative form mutation.
- One representative async job flow from enqueue to completed/failed status.
- Any money, destructive, billing, tenant membership, role-change, or soft-lock flow.

Test placement rules:

- `src/` contains implementation code only.
- Do not colocate test files with routes, components, server functions, services, repositories, utilities, generated clients, or stores.
- Unit and component tests live under `tests/unit/`.
- Integration tests live under `tests/integration/`.
- Playwright tests live under `tests/e2e/`.
- Shared fixtures, factories, and helpers live under `tests/fixtures/`, `tests/factories/`, and `tests/helpers/`.

## Anti-Patterns

| Anti-pattern | Why banned | Alternative |
| --- | --- | --- |
| React + Vite SPA for OAuth/OIDC BFF app | Browser tends to manage token/session material | TanStack Start with server-managed app session |
| Storing refresh tokens in browser storage | Token theft risk under XSS | Server-side token storage |
| `localStorage` bearer tokens | Persistent XSS exfiltration risk | HttpOnly app session cookie |
| `sessionStorage` refresh tokens | Still exposed to injected JavaScript | Server-side token storage |
| Treating route guards as authorization | Direct server function calls can bypass UX guards | Auth middleware/checks in every protected server function |
| Exposing provider tokens to client components | Leaks auth boundary | Server-only token handling |
| Mixing BFF and full-stack ownership accidentally | Unclear authority and duplicated business rules | Document resource ownership profile |
| Static Nginx deployment | TanStack Start needs server runtime | Runtime container behind Nginx |
| `useEffect` data fetching | Race conditions and stale state | TanStack Query and route loaders |
| API response data in global store | Duplicates server cache | TanStack Query |
| Scattered `useState` for shared or workflow state | State becomes duplicated, inconsistent, and hard to reason about across routes/components | Use URL state, TanStack Query, TanStack Form, TanStack Store/Zustand, route context, or job status APIs based on state ownership |
| Plain `shadcn init` | Loses design-system preset | Approved preset with `--template start` |
| Public source maps | Exposes source code | Private upload only |
| Wildcard credentialed CORS | Allows unintended origins | Explicit origins only |
| Raw provider errors in UI | Leaks internals and confuses users | Normalized product errors |
| Long-running work inside server functions | Blocks request lifecycle and fails under retries/timeouts | Enqueue BullMQ job and return `jobId` |
| Inline retry loops for third-party side effects | Duplicates side effects and ties retries to browser requests | BullMQ retry/backoff with idempotency |
| Job payloads with tokens, cookies, or secrets | Queue storage and logs become credential exposure surfaces | Store IDs only and resolve secrets server-side |
| Browser polling BullMQ internals | Leaks infrastructure details and bypasses product auth | App-owned job status API |
| Workers bypassing tenant/RBAC/product state checks | Async work can cross tenant or entitlement boundaries | Re-check authorization or use enqueue-time authorization snapshot |

## See Also

- [React + Vite Dashboard](web-react-vite-dashboard.md)
- [Astro Landing](web-astro-landing.md)
- [Backend: Bun + Elysia](backend-bun-elysia.md)
- [Infrastructure: Docker Compose + Nginx](infra-docker-compose-nginx.md)
- [Security Baseline: Web Applications](security-web-app-baseline.md)
- [CI and Testing: TypeScript + React](ci-testing-typescript-react.md)

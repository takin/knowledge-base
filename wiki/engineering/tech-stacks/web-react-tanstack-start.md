# Web Stack: React + TanStack Start

Updated: 2026-05-22
Status: Draft
Sources: Internal Tech Stacks draft (2026-04-15, updated 2026-05-22)
Platform: Web / frontend-owned or fullstack
Runtime: Bun
Framework: React + TanStack Start
Primary Use Case: Frontend-owned or fullstack React SSR applications using TanStack Start, TanStack Router, TanStack Query/Form/Table/Virtual, Tailwind, Shadcn, Docker Compose, and Nginx; fullstack products add Better Auth, Drizzle, PostgreSQL, and server functions
Raw: [2026-04-15-web-react-tanstack-start-stack.md](../../../raw/engineering/tech-stacks/2026-04-15-web-react-tanstack-start-stack.md)

## Summary

This is the current React web application stack standard. It covers fullstack products and frontend-owned products that do not own server-side persistence or authentication. "Frontend-owned" does not mean static-only hosting: a frontend-owned product may still run TanStack Start SSR, prerendered routes, BFF-style integration points, and a Nitro app server. Static-only export is outside the baseline until a separate static deployment contract exists.

The stack is Bun-first, TanStack Start-first, Dockerized, and Nginx-fronted. Production traffic flows through Nginx before reaching the internal TanStack Start container. Production containers run generated Nitro output directly with `bun --bun run .output/server/index.mjs`, not `bun run start`.

API examples touching TanStack Start server entry, deferred hydration, Router SSR nonce handling, TanStack Form validators, or `nuqs` TanStack Router integration are version-sensitive and must be verified against each product's pinned package versions before production use.

## Runtime And Deployment

All products adopting this standard run on **Bun** as runtime, package manager, script runner, and baseline test runner.

- Use `bun` for script execution, dependency installs, and builds.
- Do not use Node.js as the application runtime.
- Do not use npm, yarn, or pnpm as package managers.
- `bun test` is the baseline test runner; Vitest may be added only when richer React/component utilities are required.
- Use `oven/bun:<pinned-version>` in Docker images; never use `latest`.
- Use a multi-stage app Dockerfile with `deps`, `build`, and `runtime` stages.
- `deps` and `build` install full dependencies because build tooling usually lives in `devDependencies`.
- `runtime` installs production dependencies only with `bun install --production --frozen-lockfile`.
- The final runtime image must not include dev dependencies, `.env`, secrets, local caches, test reports, or source control metadata.
- Runtime configuration is injected through environment variables.
- The runtime container runs as a non-root user.
- `public/` stays in the repo even when empty so the Dockerfile contract stays identical across products.

Production Docker command contract:

```dockerfile
RUN bun run build
CMD ["bun", "--bun", "run", ".output/server/index.mjs"]
```

Do not use `bun run start` as the Docker runtime command. Script indirection can hide dev-server behavior, source TypeScript transpilation, framework-specific assumptions, or dependencies installed only in `devDependencies`. A `start` script may exist for local/manual production-mode runs, but if present it must match `bun --bun run .output/server/index.mjs`.

Every running Nitro app-server product exposes internal `GET /ready`. It returns `2xx` only when the app process and required local serving dependencies are available. Keep the body minimal, for example `ok`; do not include secrets, build metadata, dependency details, stack traces, or environment dumps.

## Nginx Gateway

TanStack Start must never be exposed directly to outside traffic in staging or production. Nginx is the required outside-facing gateway.

- The app container listens only on the internal Docker network.
- Compose uses `expose:` for the app container, not host-facing `ports:`.
- Nginx is the only service binding host ports `80` / `443`.
- Browser traffic flows through Nginx before TanStack Start.
- Nginx owns TLS termination, HTTP/2, Brotli/Gzip compression, baseline security headers, CSP nonce generation/forwarding, static asset caching, rate limiting, and upstream proxy retry behavior.
- Baseline deployment proxies app and asset requests to the internal app container unless the standard Nginx image contract is extended to copy `.output/public` or equivalent built assets into the Nginx image.
- Do not invent per-project asset-copy flows in the application Dockerfile.

## Framework

TanStack Start is the required React web framework.

- File-based routing is handled by TanStack Router.
- SSR with streaming is the default.
- Fullstack apps use server functions instead of REST API controllers.
- Do not build a separate Express/Hono API layer unless there is an explicit integration reason.
- Frontend-owned apps may use Start for routing, SSR, prerendered routes, and BFF integration points without owning a database or auth server.
- TanStack Router handles client-side navigation; do not use React Router or Next.js.

New projects are initialized through the TanStack CLI:

```bash
bunx --bun @tanstack/cli@latest create my-app \
  --framework react \
  --package-manager bun \
  --toolchain eslint \
  --no-examples \
  -y
```

`@latest` is allowed only for one-time generation commands such as TanStack CLI and Shadcn CLI. Generated files and lockfile changes must be reviewed before merge. Production dependencies stay pinned by supply-chain rules.

## Project Structure

Every product uses this folder structure unless a product-level ADR documents an exception:

```text
src/
  routes/
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
  pages/
    HomePage.tsx
    products/
      ProductsPage.tsx
      ProductDetailPage.tsx
  components/
    ui/
    layout/
    shared/
  features/
    <domain>/
      components/
      hooks/
      queries/
      mutations/
      schema.ts
      types.ts
  lib/
    auth.ts
    auth-client.ts
    csp.ts
    env.ts
    utils.ts
  db/
    index.ts
    schema.ts
    migrations/
  server/
    functions/
    services/
  styles/
    app.css
  router.tsx
  routeTree.gen.ts
public/
nginx/
  Dockerfile
  nginx.conf.template
  entrypoint.sh
```

Rules:

- `src/routes/**` contains route modules only: `createFileRoute`, `createRootRoute`, loaders, `beforeLoad`, `validateSearch`, `head`, route metadata, server route handlers, and route-level wiring.
- Page components live under `src/pages/**`.
- Route modules import page components from `src/pages/**`; do not define page components inline in route files.
- Page files use `getRouteApi('/route/path')` for typed params, search, loader data, and route context.
- Page files must not import `Route` from route modules because that creates circular imports and can hurt code splitting.
- Tiny `__root.tsx` document-shell wiring may stay in the root route file because it is application shell, not a page component.
- Shadcn primitives live in `src/components/ui/`; app layout components live in `src/components/layout/`; cross-feature shared components live in `src/components/shared/`.
- Domain-specific reusable code lives in `src/features/<domain>/`. Pages compose features; features must not depend on pages.
- Shared app helpers live in `src/lib/`; database code lives in `src/db/`; server-only functions/services live in `src/server/` unless they are Start server route handlers owned by `src/routes/**`.
- Global CSS and Tailwind theme entry live in `src/styles/app.css` or the scaffold's equivalent CSS entry.
- `src/routeTree.gen.ts` is generated and must never be hand-edited.

## Routing

Use folder/sub-folder route files, not flat dotted route files.

- Use `src/routes/<segment>/<child>.tsx` for nested URL segments.
- Use `src/routes/<segment>/route.tsx` for directory layout routes that wrap child routes.
- Use `$paramName.tsx` for dynamic path params inside the matching folder.
- Do not use flat dotted route files such as `store.$slug.tsx` or `dashboard.orders.$orderId.tsx`.
- The path passed to `createFileRoute()` must match the folder-based route location generated by TanStack Router.
- Keep `__root.tsx` and top-level `index.tsx` at the route root.

Route module example:

```typescript
// File: src/routes/store/$slug.tsx
import { createFileRoute } from '@tanstack/react-router'
import { StorePage } from '../../pages/store/StorePage'

export const Route = createFileRoute('/store/$slug')({
  component: StorePage,
})
```

Page module example:

```tsx
// File: src/pages/store/StorePage.tsx
import { getRouteApi } from '@tanstack/react-router'

const routeApi = getRouteApi('/store/$slug')

export function StorePage() {
  const { slug } = routeApi.useParams()
  return <main>{slug}</main>
}
```

## Deferred Hydration

Deferred hydration is required where applicable for TanStack Start pages. Use it for non-critical SSR content and heavy widgets that should be visible, styled, and indexable immediately but do not need immediate interactivity.

Good candidates include below-the-fold reviews/comments/FAQs, heavy widgets such as charts/maps/editors/video players, intent-gated panels, and static server-rendered content that need not hydrate immediately. Poor candidates include primary navigation, route chrome, consent controls, above-the-fold forms, money/data-loss controls, interactive LCP content, accessibility-critical controls, and components whose props/context/shared state must update immediately after startup.

Rules:

- Verify `Hydrate` and hydration strategy imports against the pinned TanStack Start version.
- Render the imported `Hydrate` tag directly or with an import rename.
- Keep the component to split directly inside `Hydrate`.
- Do not hide split boundaries behind wrapper components if child chunk splitting is required.
- Move hook calls into named child components inside the boundary.
- Use `split={false}` only when the child code is small, already in the startup bundle, or cannot be safely extracted.
- Measure each boundary; it must reduce startup JavaScript or hydration work without delaying expected interactions.
- Every page with heavy, below-the-fold, or intent-gated UI needs a deferred hydration review before merge.

## CSP Nonce Plumbing

Every TanStack Start app includes app-level CSP nonce plumbing for staging and production nonce-based CSP.

- `src/server.ts` reads `x-csp-nonce` and passes `{ nonce }` into Start context.
- `src/lib/csp.ts` exposes `getCspNonce()` as the only app nonce accessor.
- `src/router.tsx` configures TanStack Router with `ssr.nonce: getCspNonce()`.
- The root document emits `<meta property="csp-nonce" content={nonce} />` only when a nonce exists and when the client helper depends on DOM lookup.
- The client nonce reader returns `undefined`, not an empty string, when no nonce meta tag exists.
- Do not hardcode, reuse, or separately generate client nonce values.
- Do not generate an app fallback nonce in staging/production because it will not match the CSP header.
- Every inline executable `<script>` receives `nonce={nonce}`.
- The pre-hydration theme bootstrap script is the only approved executable inline `dangerouslySetInnerHTML` baseline exception.
- JSON-LD is the only approved non-executable data-script `dangerouslySetInnerHTML` exception; escape `<` as `\u003c`.

The exact Start server entry/context APIs and Router SSR nonce APIs are version-sensitive and must be verified during each stack refresh.

## TanStack Ecosystem

| Library | Purpose | Priority |
| --- | --- | --- |
| TanStack Query | Server state, data fetching, caching, background sync | Primary; exhaust this before Zustand |
| TanStack Router | Type-safe file-based routing | Required, bundled with Start |
| TanStack Form | Submit-style form state, paired with Zod | Required for submit-style forms |
| TanStack Table | Headless table / datagrid | Required for interactive or data tables |
| TanStack Virtual | List/grid virtualization | Required for long scrolling lists rendering more than 100 items at once |

TanStack Query is the default for all server state. If state is fetched from or synchronized with the server, it belongs in Query rather than Zustand.

## Tables And URL State

Interactive data tables should implement column filtering, global search, column sorting, accessible column visibility when useful, and accessible empty/loading/error/no-results states by default. Simple display tables, key-value summaries, static comparison tables, and compact action/detail tables may omit features that do not fit their purpose.

Use client-side pagination for bounded master data. Use server-side pagination for fast-growing transactional data. If a table will contain more than about 5,000 rows within 12 months, use server-side pagination. URL `page` is one-based; TanStack Table `pageIndex` is zero-based.

`nuqs` is required for shareable URL search param state.

- Use URL state for filter, search, sort, pagination, list/detail pages, data tables, and other shareable views.
- Mount `NuqsAdapter` from `nuqs/adapters/tanstack-router` once around `<Outlet />`, after verifying the adapter import path against the pinned `nuqs` version.
- Use parser maps with `createStandardSchemaV1` in TanStack Router `validateSearch`, after verifying the helper against the pinned `nuqs` version.
- Pin `nuqs` versions and test filter/search/pagination URL behavior.
- Use `parseAs*` parsers; do not cast raw `searchParams` strings manually.
- For high-frequency search boxes, debounce URL writes.
- If route loaders or server functions re-run from search-param changes, separate immediate input state from committed URL state.

## State Management

| Layer | Tool | What belongs here |
| --- | --- | --- |
| Server state | TanStack Query | Anything fetched from or synchronized with the server |
| Form state | TanStack Form | Submit-style form inputs, field errors, submission state |
| Global client state | Zustand | UI-only state that must persist across routes, such as sidebar state, multi-step flow progress, or hydrated mirror of theme preference |
| Local component state | `useState` / `useReducer` | Ephemeral UI state scoped to one component |

Restricted `useEffect` policy:

- Do not use `useEffect` for data fetching, derived/computed state, cross-component communication, or state mirroring.
- Use Query, route loaders, event handlers, selectors, or derived values instead.
- `useEffect` is allowed for imperative synchronization with external systems such as DOM APIs, third-party SDKs, browser subscription cleanup, page/view analytics, and imperative widgets.
- Effects that bridge React to external systems must name the external system in code review.

## Async-First UI

- Mutations that change user-visible data should use TanStack Query optimistic updates when rollback is safe and predictable.
- Use explicit confirmation UI for destructive, payment, irreversible, inventory-finalization, compliance-sensitive, external-provider-authoritative, or otherwise unsafe-to-rollback actions.
- Full-page loading spinners are banned as the default loading pattern.
- Minimal blocking initialization states are allowed only for documented app bootstrap boundaries.
- Use Suspense boundaries with skeleton placeholders for initial data loads.
- Use subtle background refetch indicators for stale-while-revalidate patterns.
- Route transitions use TanStack Router `pendingComponent`.
- Default server-pushed data to Server-Sent Events; use WebSocket only for genuinely bidirectional low-latency cases.
- Production deployments serve browser traffic over HTTP/2 at the Nginx gateway.

## Styling And UI

Tailwind CSS v4 is the required styling system. No CSS Modules, styled-components, Emotion, or other CSS-in-JS. Configure CSS-first tokens in `app.css` via `@theme`. Use Tailwind v4 custom variant syntax for class-based dark mode. Reference design tokens via `var(--token-name)`.

Shadcn UI is the primary component library. Components are copied into `src/components/ui/` and customized in place. Third-party component libraries are banned unless a required feature is absent from Shadcn/Radix. The base Shadcn preset or registry is maintained in the organization's shared repository, currently `minia/ui`; replace this name before publishing externally.

Configure Shadcn for TanStack Start / non-RSC output and audit generated files for `"use client"` directives. TanStack Start does not use the Next.js directive model. Browser-only components must be guarded, deferred, or isolated rather than blindly SSR-rendered.

React Sonner is the required toast library. Render `<Toaster />` once in the root layout and use `toast()` from `sonner`. Use success toasts for important completed actions and non-obvious confirmations; do not toast every routine optimistic mutation. Error toasts are required on mutation failure. Prefer inline validation for fixable form problems.

Dark and light mode support is required in every product. Persist canonical browser preference as raw `localStorage.theme` with value `light`, `dark`, or `system`. Zustand may mirror it after hydration, but default JSON persist format must not overwrite the raw string contract needed by the pre-hydration bootstrap script.

Recharts is the required charting library for standard product charts. Use CSS variables for colors, wrap charts in responsive containers, and consider deferred hydration for browser-layout-dependent or heavy charts. D3 is allowed only for complex custom visualizations that Recharts cannot express cleanly.

Motion should be simple and purposeful. Prefer Tailwind transitions; use Motion for React (`motion/react`) only for route-level transitions, gestures, choreography, or exit transitions. Respect `prefers-reduced-motion`; do not use animation as the only cue for state, validation, success, failure, or progress.

## Authentication, Database, And Forms

Better Auth is required for fullstack products. Frontend-owned products that delegate authentication to an external backend must document the boundary and must not implement ad hoc client-only session logic. Configure server auth in `src/lib/auth.ts`, client auth in `src/lib/auth-client.ts`, and enforce protected routes with TanStack Router `beforeLoad` guards backed by server-side session checks.

PostgreSQL is the only supported database for fullstack products. Drizzle ORM is required for products that access PostgreSQL from app code. Prefer Bun's native `bun:sql` driver after verifying production needs such as pooling, TLS, PgBouncer, prepared statements, migrations, and observability. Use `postgres` when production requirements are unsupported or insufficiently proven with `bun:sql`; document the reason. Other drivers are banned unless there is an explicit infrastructure reason.

RLS is the preferred end-state for multi-tenant products but is not mandatory in Phase 1. Every tenant-scoped table needs a non-null tenant identifier. Multi-tenant products must explicitly choose Phase 1 application-level isolation with leakage tests or RLS mode with migration-defined policies. In RLS mode, set tenant context with transaction-local PostgreSQL state such as `SET LOCAL app.current_tenant_id = '...'`; plain session-level `SET` is banned for pooled request tenant context unless the connection is dedicated and reset before reuse.

All submit-style forms use TanStack Form. Filter/search controls that only update URL state may use `nuqs` directly. Zod schemas are the source of truth for submitted or persisted input and are reused server-side. Verify the TanStack Form validator shape against the pinned version before using direct Standard Schema-compatible validators.

## PWA And Observability

PWA support is optional by default. Add it only when the product benefits from installability, offline behavior, background sync, push notifications, or mobile-app-like distribution. Do not add a service worker by default because stale-cache bugs and broken deploys are common failure modes.

Every web product must be mobile-responsive, mobile-first, include a mobile viewport meta tag, use touch-friendly primary targets, and behave safely on poor or intermittent networks.

Sentry is the standard error and performance monitoring tool but remains optional and runtime-toggled. Check `process.env.SENTRY_ENABLED === "true"` before initialization. If enabled, `SENTRY_DSN` is required in production. TanStack Start has server and browser execution paths, so products must document where server-side and client-side Sentry initialization happen and verify compatible SDK entry points for pinned runtime/framework versions.

Production deployments emit structured logs to stdout/stderr, include request or trace IDs in server logs, keep `/ready` minimal and unauthenticated for internal health checks, and record security-sensitive events without logging secrets or full payloads.

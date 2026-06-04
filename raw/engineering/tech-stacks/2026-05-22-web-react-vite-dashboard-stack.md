# Web Stack: React + Vite Dashboard

Source URL: Internal draft
Collected: 2026-05-22
Published: 2026-05-22
Updated: 2026-05-23
Status: Draft
Scope: Authenticated SaaS dashboards, API-only frontend apps, no-SEO React SPAs, and self-hosted static deployment

---

## 1. Runtime & Toolchain

Dashboard projects use **Bun** as package manager, script runner, build runner, and baseline test runner.

Rules:
- Use `bun` for dependency installs, scripts, builds, and tests.
- Do not use npm, yarn, or pnpm.
- Use TypeScript in strict mode.
- Use Vite as the build tool.
- Use Oxlint as the default linter.
- Use Oxfmt as the default formatter.
- Build output is static `dist/`.
- The dashboard has no production frontend server runtime.
- Do not add SSR, server functions, API routes, or database access to the dashboard repo.
- Production deployment must use Docker Compose and Nginx serving the built `dist/` directory.
- Production dependencies must be pinned to exact versions. Dev dependencies may use controlled ranges if lockfile changes are reviewed.

Required `package.json` script contract:

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc --noEmit && vite build",
    "preview": "vite preview",
    "typecheck": "tsc --noEmit",
    "lint": "oxlint .",
    "format": "oxfmt",
    "format:check": "oxfmt --check",
    "test": "vitest run",
    "test:e2e": "playwright test",
    "api:generate": "openapi-generator-command-here"
  }
}
```

The exact OpenAPI generator command is project-specific, but every dashboard project must provide a single script that regenerates the API client from the backend OpenAPI spec. Generation must use a committed/CI-generated OpenAPI artifact or an explicitly docs-enabled/auth-gated environment. Do not depend on unauthenticated production `/docs/json`.

## 2. Development Tooling

Dashboard projects use an Oxc-first development toolchain for fast local feedback and lightweight CI.

Rules:
- Oxlint is the required linter.
- Oxfmt is the required formatter.
- `tsc --noEmit` remains required for TypeScript typechecking; Oxlint does not replace the TypeScript compiler.
- Do not use ESLint as the default linter for new dashboard projects.
- Do not use Prettier or Biome as the default formatter for new dashboard projects.
- If a rule is not available in Oxlint, do not add ESLint by default. Add a targeted exception only when a concrete risk justifies the slower toolchain.
- Oxfmt configuration must live in the project formatter config, not in ad hoc CLI flags.
- If formatter behavior must change, document the reason in the project README or design notes.

Required development dependencies:
- `oxlint`
- `oxfmt`
- `typescript`
- `@vitejs/plugin-react`
- `babel-plugin-react-compiler`

## 3. Framework & Rendering Model

**Vite + React SPA** is the required dashboard framework model.

Rules:
- Dashboard is a single-page application.
- Dashboard uses React 19+.
- Dashboard does not need SEO.
- Dashboard must not use SSR.
- Dashboard must not use fullstack SSR frameworks.
- Dashboard must not use Next.js.
- Dashboard must not use server actions or route handlers.
- All product data comes from the separate backend API.
- Client-side route protection is UX only. Backend API authorization is the security boundary.

## 4. React Compiler And Performance Baseline

Dashboard projects must enable React Compiler for production builds.

Rules:
- Use React 19+.
- Install and configure `babel-plugin-react-compiler`.
- Configure React Compiler in Vite through `@vitejs/plugin-react`.
- Use `compilationMode: "infer"` and `panicThreshold: "critical_errors"` unless a project-level ADR documents a different compiler rollout strategy.
- Do not add manual `useMemo`, `useCallback`, or `React.memo` by default.
- Manual memoization is allowed only for semantic identity requirements, third-party integration stability, expensive non-React computations not handled by the compiler, or measured regressions.
- Prefer clear component code and let React Compiler optimize render paths.
- Keep components pure; compiler effectiveness depends on React rules being followed.
- Fix React Compiler diagnostics instead of silencing them.
- Use `startTransition` for non-urgent state transitions that would otherwise block input.
- Use `useDeferredValue` for high-frequency UI inputs such as search or filter text when immediate updates make the UI sluggish.

Required Vite configuration shape:

```ts
import { defineConfig } from "vite"
import react from "@vitejs/plugin-react"

const ReactCompilerConfig = {
  compilationMode: "infer",
  panicThreshold: "critical_errors",
}

export default defineConfig({
  plugins: [
    react({
      babel: {
        plugins: [["babel-plugin-react-compiler", ReactCompilerConfig]],
      },
    }),
  ],
})
```

## 5. Backend Boundary

The backend API is mandatory and lives in a separate repo/project.

Rules:
- API owns authentication, authorization, validation, persistence, and business rules.
- Dashboard owns UI rendering, user interaction, client-side routing, cache orchestration, and form UX.
- Dashboard does not access databases, queues, object storage, email providers, payment providers, or secrets directly.
- Dashboard does not duplicate backend business rules as authoritative logic.
- Frontend validation improves UX only; backend validation remains mandatory.
- Every protected backend action must enforce authorization server-side, even when the dashboard hides the UI.
- API must expose OpenAPI for dashboard client generation.

## 6. OpenAPI Client Contract

OpenAPI is the contract between the backend API and dashboard.

Rules:
- Dashboard consumes the backend OpenAPI spec through code generation.
- Generated files live under `src/lib/api/generated/`.
- Generated files are never hand-edited.
- The handwritten API wrapper lives in `src/lib/api/client.ts`.
- The API wrapper owns base URL, credentials mode, auth header behavior when needed, request IDs when needed, and normalized error mapping.
- Do not call `fetch()` directly from feature code unless the generated client cannot express the endpoint and an inline comment documents why.
- API response envelopes must be normalized into predictable success and error handling.
- Regenerate the client whenever the backend OpenAPI spec changes.
- OpenAPI generation must run in CI or be checked by CI so generated client drift is caught before merge.
- Client generation must not depend on unauthenticated production `/docs/json`; use a local/CI generated spec artifact or an authenticated docs-enabled staging endpoint.

Expected generated-client structure:

```text
src/lib/api/
  client.ts
  errors.ts
  generated/
    ... generated files, never hand-edit ...
```

## 7. Project Initialization

New dashboard projects are initialized with Vite React TypeScript:

```bash
bun create vite my-dashboard --template react-ts
```

Post-init rules:
- Add TanStack Router.
- Add TanStack Query.
- Add TanStack Table when interactive data tables are needed.
- Add TanStack Form and Zod before building submit-style forms.
- Add `nuqs` for shareable URL search params.
- Add Oxlint and Oxfmt before configuring CI.
- Add React Compiler before production launch.
- Add Tailwind CSS v4+.
- Add Shadcn UI as the default component baseline using the custom preset initialization flow below.
- Add Playwright before production launch.
- Review generated files and lockfile changes before merge.

### 7.1 Shadcn Initialization

Dashboard projects use Shadcn UI as the default component baseline, but Shadcn must be initialized with a custom preset so theme, styling, component defaults, and tokens match the intended product design system.

Before running Shadcn init, always ask the human for the preset code.

Fallback default when the human gives no preset code or responds with an empty value:

```bash
bunx --bun shadcn@latest init --preset b6Z8CJysK --template react-router
```

Rules:
- Do not run plain `shadcn init`.
- Do not initialize Shadcn without asking the human for the preset code first.
- Use `bunx --bun`, not `npx`, `pnpm dlx`, or `yarn dlx`.
- Use `shadcn@latest` only for the CLI invocation; committed runtime dependencies must still be reviewed and pinned.
- Use `--template react-router` for Vite React dashboard projects.
- Use the human-provided preset when one is provided.
- Use fallback preset `b6Z8CJysK` only when the human provides no preset or responds with an empty value.
- Never decode, fetch, or inspect preset codes manually; pass preset codes directly to the Shadcn CLI.
- Document any project-specific approved preset in the project README or design notes.

## 8. Standard Directory Layout

Every dashboard project uses this structure unless a product-level ADR documents an exception:

```text
src/
  routes/
    __root.tsx
    index.tsx
    login.tsx
    dashboard/
      route.tsx
      index.tsx
      settings.tsx
      users/
        index.tsx
        $userId.tsx
  pages/
    LoginPage.tsx
    DashboardHomePage.tsx
    users/
      UsersPage.tsx
      UserDetailPage.tsx
  features/
    auth/
      auth-client.ts
      auth-queries.ts
      auth-store.ts
    users/
      components/
      queries.ts
      mutations.ts
      searchParams.ts
      schema.ts
      types.ts
  components/
    ui/
    layout/
    shared/
  lib/
    api/
      client.ts
      errors.ts
      generated/
    env.ts
    query-client.ts
    router.tsx
    utils.ts
  styles/
    app.css
  main.tsx
public/
nginx/
  nginx.conf.template
  entrypoint.sh
infra/
  scripts/
    renew-certs.sh
Dockerfile
.dockerignore
docker-compose.yml
.env.example
vite.config.ts
```

Rules:
- `src/routes/**` contains route declarations, guards, search validation, loaders, and route-level wiring only.
- Page components live under `src/pages/**`.
- Route modules import page components; they do not define page components inline.
- Domain-specific reusable code lives in `src/features/<domain>/`.
- Feature modules may depend on `src/lib/**` and `src/components/**`; they must not depend on page modules.
- Shadcn primitives live in `src/components/ui/**`.
- App shell and navigation components live in `src/components/layout/**`.
- Cross-feature shared components live in `src/components/shared/**`.
- Generated API code lives only under `src/lib/api/generated/**`.
- Global CSS and Tailwind theme entry live in `src/styles/app.css`.
- `Dockerfile`, `.dockerignore`, and `docker-compose.yml` are required for production deployment.
- `nginx/nginx.conf.template` and `nginx/entrypoint.sh` are required; do not add `nginx/Dockerfile`.
- `infra/scripts/renew-certs.sh` is required for Let's Encrypt renewal from host cron or systemd timers.

## 9. Routing: TanStack Router

TanStack Router is the required router.

Rules:
- Do not use React Router.
- Use file-based routing where practical.
- Use folder/sub-folder route files, not flat dotted route files.
- Use route-level guards for authenticated dashboard sections.
- Use route-level search validation for pages with URL state.
- Keep route modules thin.
- Route protection in the dashboard is UX only; backend API remains the authorization boundary.
- Unknown authenticated routes must render a useful not-found state.

Correct route file examples:

```text
src/routes/dashboard/users/index.tsx
src/routes/dashboard/users/$userId.tsx
src/routes/dashboard/settings.tsx
```

Banned route file examples:

```text
src/routes/dashboard.users.index.tsx
src/routes/dashboard.users.$userId.tsx
```

## 10. Server State: TanStack Query

TanStack Query owns all server state.

Rules:
- Use TanStack Query for all API data fetching.
- Do not use `useEffect` for data fetching.
- Do not store API response data in Zustand.
- Query keys must include every input that changes the returned data: tenant, route params, filters, search, sort, page, page size, and feature flags when relevant.
- Use explicit `staleTime` for low-volatility data such as workspace settings, feature flags, static lookup tables, and current user metadata.
- Prefer background refetch over blocking page reloads when stale data can be safely shown while updating.
- Avoid refetch storms after mutations; invalidate the narrowest affected query keys instead of broad global invalidation.
- Disable or customize retries for unsafe or non-idempotent operations. Mutations must not be retried blindly when duplicate writes could create side effects.
- Mutations must invalidate or update affected query caches.
- Use optimistic updates only when rollback is safe and predictable.
- Use explicit confirmation UI for destructive, payment, irreversible, compliance-sensitive, external-provider-authoritative, or otherwise unsafe-to-rollback actions.
- Errors from API calls must surface through accessible inline states or toasts.
- Never silently swallow mutation failures.

## 11. URL State: nuqs

`nuqs` is required for shareable URL search param state.

Rules:
- Use URL state for filter, search, sort, pagination, selected tabs, list/detail views, and other states users may refresh or share.
- Do not use Zustand for filter/search/pagination state that belongs in the URL.
- Do not use `useState` for shareable table filters.
- Use typed `parseAs*` parsers.
- Do not cast raw `searchParams` strings manually.
- Debounce high-frequency URL writes such as search input.
- Pass normalized URL state into TanStack Query keys so refetching is automatic.
- URL `page` is one-based for humans; TanStack Table `pageIndex` is zero-based.

## 12. Tables: TanStack Table

TanStack Table is required for interactive data tables.

Default interactive table features:
- Column filtering.
- Global search.
- Column sorting.
- Column visibility menu when useful.
- Accessible empty, loading, error, and no-results states.
- Pagination appropriate to data growth.

Pagination rules:
- Use client-side pagination for bounded master data.
- Use server-side pagination for fast-growing transactional data.
- If a table can exceed about 5,000 rows within 12 months, use server-side pagination.
- Do not use client-side filtering or sorting for unbounded transactional data.
- Server-side pagination uses `manualPagination: true` and query keys that include normalized page and filter state.
- Do not virtualize server-paginated small visible pages just because total dataset is large.
- Use TanStack Virtual for long scrolling lists that render more than 100 items at once.

## 13. Forms & Validation

TanStack Form + Zod is the required submit-form stack.

Rules:
- Use TanStack Form for submit-style forms.
- Use Zod schemas for form validation.
- Reuse or mirror backend validation schemas where practical, but backend remains authoritative.
- Do not use React Hook Form, Formik, or custom form state frameworks.
- Filter/search controls that only update URL state may use `nuqs` directly and do not need TanStack Form.
- Show field-level errors for fixable validation problems.
- Use global toasts only for transient, cross-page, or non-field-specific outcomes.

## 14. State Management

Layered state model:

| State type | Tool | Examples |
|---|---|---|
| Server state | TanStack Query | users, invoices, settings, workspace data, API responses |
| URL/shareable state | nuqs + TanStack Router | search, filters, sort, page, tab, selected view |
| Form state | TanStack Form + Zod | create user, billing, profile, settings forms |
| Global client/UI state | Zustand | sidebar, command palette, theme mirror, onboarding step |
| Local ephemeral state | `useState` / `useReducer` | modal open, dropdown state, temporary input |

Zustand rules:
- Zustand is allowed only for client-owned UI state.
- Do not store API response data in Zustand.
- Do not duplicate TanStack Query cache into Zustand.
- Do not use Zustand for filter/search/pagination state that should survive refresh or be shareable by URL.
- Use one store per domain or UI slice.
- Do not create a single global mega-store.
- Persist Zustand state only when UX requires persistence.
- Persisted state must be serializable.
- Do not persist server-derived data.
- Actions may be functions, but persisted state must not contain functions, class instances, DOM nodes, or non-serializable values.
- Prefer selectors over reading entire stores.
- Prefer derived values computed in selectors or render over redundant stored state.

Good Zustand use cases:
- `useSidebarStore`
- `useCommandPaletteStore`
- `useThemeStore`
- `useOnboardingStore`
- `useWorkspaceSwitcherUiStore`

Bad Zustand use cases:
- `useUsersStore` when users come from API.
- `useInvoicesStore` when invoices come from API.
- `useAuthUserStore` as authoritative session state.
- `useTableFilterStore` for shareable filters.
- `useApiCacheStore`.

Restricted `useEffect` policy:
- Do not use `useEffect` for data fetching.
- Do not use `useEffect` for derived/computed state.
- Do not use `useEffect` for cross-component communication.
- Use Query, route loaders, event handlers, selectors, or derived values instead.
- `useEffect` is allowed for imperative synchronization with external systems such as DOM APIs, third-party SDKs, browser subscriptions, analytics, and imperative widgets.

## 15. Async-First UI

Rules:
- Full-page blocking spinners are banned as the default loading pattern.
- Minimal blocking initialization states are allowed only for documented app bootstrap boundaries such as session handoff.
- Use skeletons for initial data loads.
- Use subtle background refetch indicators for stale-while-revalidate behavior.
- Use optimistic updates only when rollback is safe.
- Route transitions should show lightweight pending UI.
- Every async page must define loading, empty, error, and success states.
- Destructive actions require clear confirmation and post-action feedback.

## 16. UI/UX Framework And Library Standard

Dashboard projects use a fixed UI/UX baseline so every product starts with the same component primitives, styling model, feedback patterns, and visualization defaults.

| Area | Standard | Requirement |
|---|---|---|
| Styling | Tailwind CSS v4+ | Required styling system. Use CSS theme tokens and utilities. |
| Component baseline | Shadcn UI | Required default. Initialize with the custom preset flow in §7.1. |
| Primitive layer | Radix UI through Shadcn | Required default for dialogs, menus, popovers, sheets, and accessible primitives. |
| Toasts | Sonner | Required toast library. |
| Charts | Recharts through Shadcn chart patterns | Default charting library for dashboard charts. |
| Icons | `lucide-react` | Default icon library unless the Shadcn preset explicitly chooses another icon library. |
| Tables | TanStack Table + Shadcn table presentation | TanStack owns table logic; Shadcn owns visual shell and controls. |
| Forms | TanStack Form + Zod + Shadcn field components | TanStack owns form state; Zod owns validation; Shadcn owns field UI. |
| Command palette | Shadcn Command / `cmdk` | Recommended for complex dashboards with global navigation or actions. |
| Date and calendar | Shadcn Calendar + `date-fns` | Recommended default for date pickers and date formatting. |
| Async feedback | Shadcn Skeleton, Progress, Alert, Empty, and Sonner | Required primitives for loading, empty, error, and transient feedback states. |
| Motion | Tailwind transitions | Default motion layer. Add Framer Motion only when a concrete interaction cannot be expressed cleanly with Tailwind/Radix transitions. |

Rules:
- Use Tailwind utilities and CSS theme tokens.
- Tailwind v4+ is the baseline; do not create a Tailwind v3-style config unless a project-level ADR explicitly allows it.
- Do not use CSS Modules, styled-components, Emotion, or other CSS-in-JS.
- Shadcn UI is the default component baseline.
- Shadcn components live in `src/components/ui/**` and are customized in place.
- Use Shadcn components before writing custom markup for buttons, forms, dialogs, sheets, cards, tables, empty states, alerts, badges, separators, skeletons, and toasts.
- Third-party component libraries are banned unless a required feature is absent from Shadcn/Radix.
- Do not add MUI, Chakra, Mantine, Ant Design, DaisyUI, Flowbite, or Headless UI as the dashboard component baseline.
- Sonner is the required toast library.
- Use global toasts only for transient, cross-page, or non-field-specific outcomes.
- Do not toast every routine optimistic mutation if the UI already confirms success.
- Error toasts are required for mutation failures that are not fully handled inline.
- Dark and light mode are required for dashboards.
- Store canonical browser theme preference as raw `localStorage.theme` with `light`, `dark`, or `system`.
- Motion should be simple and purposeful; prefer Tailwind transitions.
- Respect `prefers-reduced-motion`.

## 17. Authentication Boundary

Auth/session source of truth is the backend API.

Rules:
- Dashboard does not implement custom auth server logic.
- Current session is fetched from the backend API through TanStack Query.
- Route protection uses TanStack Router guards for UX.
- Backend API enforces every protected endpoint.
- Frontend auth state is a convenience cache, not an authorization boundary.
- Do not store session secrets in localStorage.
- Do not store bearer access tokens in localStorage unless a security ADR explicitly accepts the risk and mitigations.
- Do not expose HttpOnly cookie contents to JavaScript.
- If token-based auth is used, storage and refresh behavior must be documented in a security review.

## 18. API Error Handling

Rules:
- Backend API returns a consistent response envelope.
- Dashboard maps machine-readable error codes to user-facing messages.
- Do not show raw Zod, Elysia, stack trace, SQL, or internal error messages to users.
- Field-level API errors map back to fields where possible.
- Global errors use accessible alert regions or Sonner toasts.
- Authorization failures route to login or access-denied states as appropriate.
- Network failures get retry affordances when retry is safe.

## 19. Charts & Data Visualization

Recharts is the default charting library for dashboard charts, preferably through Shadcn chart composition patterns.

Rules:
- Use CSS variables for chart colors.
- Charts must support dark and light mode.
- Charts must be responsive.
- Use Shadcn chart wrappers/patterns when available so tooltip, legend, color token, and dark-mode behavior stay consistent.
- Do not use Chart.js, Victory, Nivo, or D3 for standard charts.
- D3 is allowed only for custom visualizations that Recharts cannot express cleanly; document the reason inline.
- Heavy chart routes should use code splitting.

## 20. Performance Budget And Bundle Splitting

Dashboard performance is a product requirement, not a late optimization pass.

Baseline budgets:

| Asset / Metric | Budget | Rule |
|---|---:|---|
| Authenticated app shell JavaScript | <= 250 KB gzip | Initial dashboard shell only; feature routes load separately. |
| Initial CSS | <= 80 KB gzip | Includes Tailwind output and component baseline. |
| Route chunks | Project-specific | Heavy domain routes must be lazy-loaded and measured. |
| Blocking app initialization | Minimal | Only session handoff and required bootstrap config may block first render. |

Rules:
- Every domain dashboard route should be lazy-loaded unless it is part of the always-visible app shell.
- Heavy charts, data tables, rich text editors, upload flows, reporting modules, and admin modules must be route-level or component-level split.
- Do not import Recharts, rich editors, upload clients, or analytics widgets into the root app shell.
- Run a bundle analysis before production launch and after adding large dependencies.
- CI should fail or warn when the authenticated shell exceeds the agreed JavaScript or CSS budget.
- Track the largest dependency contributors during review when bundle size grows materially.
- Do not add a large UI or utility dependency when a Shadcn, TanStack, browser, or small local implementation already covers the use case.
- Fonts must be self-hosted or intentionally CDN-hosted with a documented reason.
- Use `font-display: swap` for web fonts.
- Prefer SVG icons from the approved icon library over large image sprites.
- Do not load autoplay video by default.
- Do not load analytics, chat, session replay, or marketing widgets in the authenticated app shell unless the product explicitly requires them.
- Source maps must not be publicly served in production. If production source maps are needed, upload them privately to the error monitoring provider and remove them from served static assets.

## 21. Deployment: Docker Compose + Nginx Static Runtime

Dashboard production artifact is static `dist/`, served directly by Nginx from an immutable Docker image.

Rules:
- `bun run build` must produce the deployable `dist/` directory.
- Do not run `vite preview` in production.
- Do not ship a Node/Bun app server for the dashboard.
- Production deployment must use Docker Compose.
- Production runtime must use Nginx, not Caddy or a generic static server.
- Nginx serves `dist/` directly from `/usr/share/nginx/html`.
- Nginx is the only host-facing dashboard service.
- There is no internal dashboard app server and no `proxy_pass` to dashboard runtime.
- Configure SPA fallback so unknown dashboard paths serve `index.html`.
- Static hashed assets use immutable cache headers.
- `index.html` uses no-cache or must-revalidate headers.
- Runtime environment configuration must not require rebuilding images for every environment unless the deployment consciously chooses build-time env injection.
- The dashboard image must not contain secrets, certificates, `.env`, source control metadata, test reports, or local caches.

Required deployment files:

```text
Dockerfile
.dockerignore
docker-compose.yml
nginx/
  nginx.conf.template
  entrypoint.sh
infra/
  scripts/
    renew-certs.sh
.env.example
```

### 21.1 Dockerfile Contract

Dashboard Dockerfiles use a Bun build stage and an Nginx runtime stage. The Nginx runtime uses the existing proven Brotli-enabled image directly; do not create a separate `nginx/Dockerfile`.

```dockerfile
FROM oven/bun:<pinned-version> AS deps
WORKDIR /app
COPY package.json bun.lock ./
RUN bun install --frozen-lockfile

FROM deps AS build
WORKDIR /app
COPY . .
RUN bun run build

FROM fholzer/nginx-brotli:<pinned-version> AS runtime
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx/nginx.conf.template /etc/nginx/nginx.conf.template
COPY nginx/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

EXPOSE 80
EXPOSE 443
ENTRYPOINT ["/entrypoint.sh"]
```

Rules:
- Use a pinned smallest production-suitable Bun image in build stages, preferably Alpine or slim if available and compatible; never use `latest`.
- Use `fholzer/nginx-brotli:<pinned-version>` or the approved smallest production-suitable Brotli Nginx runtime image in the runtime stage; never use `latest`.
- The runtime stage contains only built static assets and Nginx config files.
- The runtime stage must contain zero `node_modules`.
- Do not copy development `node_modules` into the final image.
- Do not install Certbot into the dashboard image.
- Do not run `bun`, `node`, `vite`, or `vite preview` in the runtime image.
- Do not mount `dist/` from the host as the production deployment mechanism.
- Do not copy production source maps into the served static directory unless they are access-controlled and explicitly approved.
- Follow the infrastructure Docker image size and final runtime policy.
- Every dashboard Docker build requires a final-image review confirming the image contains only `dist/`, Nginx config, entrypoint, and minimal runtime files.

Required `.dockerignore`:

```dockerignore
.git
.github
node_modules
dist
.cache
.DS_Store
.env
.env.*
!.env.example
coverage
playwright-report
test-results
*.log
*.map
```

If source maps are uploaded privately to an error monitoring provider, upload them before the final runtime image is produced and remove public `.map` files from the served static artifact.

### 21.2 entrypoint.sh Contract

`entrypoint.sh` renders `nginx.conf.template` into `nginx.conf` using environment variables, then starts Nginx in the foreground.

```sh
#!/bin/sh
set -e

envsubst '${NGINX_SERVER_NAME}
          ${NGINX_BROTLI_ENABLED}
          ${NGINX_GZIP_ENABLED}
          ${NGINX_WORKER_PROCESSES}
          ${NGINX_WORKER_CONNECTIONS}
          ${NGINX_KEEPALIVE_TIMEOUT}
          ${NGINX_CLIENT_MAX_BODY_SIZE}
          ${NGINX_RATE_LIMIT_RPS}
          ${CSP_CONNECT_SRC}
          ${CSP_SCRIPT_SRC}
          ${CSP_IMG_SRC}
          ${CSP_FRAME_SRC}' \
  < /etc/nginx/nginx.conf.template > /etc/nginx/nginx.conf

exec nginx -g 'daemon off;'
```

Rules:
- `entrypoint.sh` exists only to render runtime-configurable Nginx config and start Nginx.
- Do not put certificate issuance, certificate renewal, application build steps, API checks, or database checks in `entrypoint.sh`.
- `entrypoint.sh` must fail fast if template rendering fails.

### 21.3 Docker Compose Contract

Dashboard Docker Compose runs one dashboard Nginx service and one Certbot sidecar. Nginx serves static files and terminates TLS. Certbot owns certificate issuance and renewal through shared volumes.

```yaml
services:
  dashboard:
    image: example-dashboard:${TAG}
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - letsencrypt:/etc/letsencrypt:ro
      - certbot-webroot:/var/www/certbot:ro
    read_only: true
    tmpfs:
      - /var/cache/nginx
      - /var/run
      - /tmp
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    environment:
      NGINX_SERVER_NAME: "dashboard.example.com"
      NGINX_BROTLI_ENABLED: "on"
      NGINX_GZIP_ENABLED: "on"
      NGINX_WORKER_PROCESSES: "auto"
      NGINX_WORKER_CONNECTIONS: "1024"
      NGINX_KEEPALIVE_TIMEOUT: "65"
      NGINX_CLIENT_MAX_BODY_SIZE: "1m"
      NGINX_RATE_LIMIT_RPS: "20"
      CSP_CONNECT_SRC: "https://api.example.com"
      CSP_SCRIPT_SRC: ""
      CSP_IMG_SRC: ""
      CSP_FRAME_SRC: ""

  certbot:
    image: certbot/certbot:<pinned-version>
    volumes:
      - letsencrypt:/etc/letsencrypt
      - certbot-webroot:/var/www/certbot

volumes:
  letsencrypt:
  certbot-webroot:
```

Rules:
- `dashboard` binds host ports `80` and `443`.
- `certbot` does not bind host ports.
- Certificates live in Docker volumes, never in the image layer.
- Certificate renewal must not require rebuilding the dashboard image.
- Do not add an `app` service for the dashboard runtime.
- Do not expose a dashboard app server on the internal Docker network unless a product-level ADR adopts SSR or runtime HTML generation.
- Use `read_only: true` where compatible with the selected Nginx image.
- Use `tmpfs` for Nginx writable runtime paths such as `/var/cache/nginx`, `/var/run`, and `/tmp`.
- Drop all Linux capabilities by default and add back only `NET_BIND_SERVICE` when binding privileged ports requires it.

### 21.4 Nginx Static SPA Contract

Nginx has two server blocks in production: port 80 for ACME challenge and HTTPS redirect, and port 443 for serving the dashboard.

Port 80 rules:
- Port 80 must always redirect to port 443.
- Port 80 may only serve ACME HTTP-01 challenge files and HTTPS redirects.
- Dashboard application traffic must not be served over plain HTTP in production.

```nginx
server {
    listen 80;
    server_name ${NGINX_SERVER_NAME};

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}
```

Port 443 rules:
- Nginx terminates TLS using Let's Encrypt certificates from `/etc/letsencrypt`.
- Nginx serves Vite assets directly from `/usr/share/nginx/html`.
- SPA fallback is mandatory.
- `index.html` must not be cached immutably.
- Hashed assets must be cached immutably.
- Every `location` that sets custom `add_header` must repeat the full security header set because Nginx does not inherit parent `add_header` directives in that case.

```nginx
server {
    listen 443 ssl http2;
    server_name ${NGINX_SERVER_NAME};

    ssl_certificate /etc/letsencrypt/live/${NGINX_SERVER_NAME}/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/${NGINX_SERVER_NAME}/privkey.pem;

    root /usr/share/nginx/html;
    index index.html;

    location ~* \.(js|css|woff2|woff|ttf|svg|png|jpg|jpeg|webp|avif|ico)$ {
        try_files $uri =404;
        add_header Cache-Control "public, max-age=31536000, immutable" always;
    }

    location / {
        try_files $uri $uri/ /index.html;
        add_header Cache-Control "public, max-age=0, must-revalidate" always;
    }
}
```

### 21.5 Let's Encrypt Certificate Lifecycle

Production dashboard deployments must provision and renew Let's Encrypt certificates.

Rules:
- Nginx must terminate TLS with Let's Encrypt certificates.
- Self-signed certificates are banned in production.
- Certificate files must live in Docker volumes.
- Certificate issuance and renewal are handled by the Certbot sidecar, not by the dashboard image.
- Auto-renewal must be configured through host cron, systemd timer, or an approved equivalent scheduler.
- Nginx must reload after successful certificate renewal.
- Expired certificates are a production-blocking failure.
- Port 80 must remain available for ACME HTTP-01 challenges and HTTPS redirects.

Required renewal script path:

```text
infra/scripts/renew-certs.sh
```

Required script shape:

```bash
#!/usr/bin/env bash
set -euo pipefail

COMPOSE_FILE="${COMPOSE_FILE:-docker-compose.yml}"
PROJECT_DIR="${PROJECT_DIR:-/opt/example-dashboard}"
CERTBOT_SERVICE="${CERTBOT_SERVICE:-certbot}"
NGINX_SERVICE="${NGINX_SERVICE:-dashboard}"
WEBROOT="${WEBROOT:-/var/www/certbot}"

cd "$PROJECT_DIR"

echo "[ssl-renew] $(date -Is) starting certificate renewal"

docker compose -f "$COMPOSE_FILE" run --rm \
  "$CERTBOT_SERVICE" renew \
  --webroot \
  -w "$WEBROOT" \
  --quiet

echo "[ssl-renew] $(date -Is) renewal command completed"

docker compose -f "$COMPOSE_FILE" exec -T \
  "$NGINX_SERVICE" nginx -s reload

echo "[ssl-renew] $(date -Is) nginx reloaded"
echo "[ssl-renew] $(date -Is) done"
```

Required cron setup documentation:

```cron
SHELL=/bin/bash
PATH=/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin

17 3,15 * * * PROJECT_DIR=/opt/example-dashboard COMPOSE_FILE=docker-compose.yml /opt/example-dashboard/infra/scripts/renew-certs.sh >> /var/log/example-dashboard-ssl-renew.log 2>&1
```

Rules:
- Run renewal checks at least twice daily.
- Use `exec -T` for cron compatibility.
- Log renewal output to a persistent host log file.
- The renewal script must exit non-zero if Certbot renewal or Nginx reload fails.
- The deployment runbook must include a manual renewal test before launch.

Manual test command:

```bash
PROJECT_DIR=/opt/example-dashboard COMPOSE_FILE=docker-compose.yml /opt/example-dashboard/infra/scripts/renew-certs.sh
```

## 22. Self-Hosting Rules

Self-hosted dashboard deployments use the Docker Compose + Nginx static runtime defined above.

Rules:
- Nginx owns TLS termination, compression, security headers, and static asset caching.
- The dashboard container or static server must not expose secrets.
- The final image contains only static assets and server config.
- API requests go to the external API base URL. Do not proxy to a hidden dashboard backend.
- Configure CORS on the backend API intentionally for dashboard origins.
- Health checks may verify static server availability but must not expose environment details.

Required static caching behavior:
- Hashed JS/CSS/assets: `Cache-Control: public, max-age=31536000, immutable`.
- `index.html`: `Cache-Control: no-cache` or `public, max-age=0, must-revalidate`.
- `robots.txt` and manifest assets, if any: short or explicit cache policy.

## 23. Security Baseline

Rules:
- Never hardcode secrets or API keys.
- Never bake secrets into dashboard images.
- Only public environment values may be exposed to the browser.
- Public browser configuration must be clearly named, such as `VITE_PUBLIC_*`, and validated before build or startup.
- The build must fail when required public configuration is missing or malformed.
- Configure CSP/security headers at the static gateway.
- Add third-party script/connect/image/frame sources only when the project uses them and after review.
- Do not use unsanitized `dangerouslySetInnerHTML`.
- Do not render raw backend error messages, stack traces, SQL errors, Zod internals, or provider diagnostics to users.
- If cookie auth is used, the backend must enforce CSRF protection and CORS must allow only explicit dashboard origins.
- No `Access-Control-Allow-Origin: *` with credentials.
- Do not add a service worker by default.
- Sanitize any external HTML before rendering.
- Sentry or analytics must scrub PII before capture.
- Session replay is disabled by default and requires explicit approval.
- Error monitoring is recommended before production, but PII scrubbing is mandatory when enabled.
- API credentials and auth enforcement belong to the backend API.

## 24. Testing & CI

Required CI checks:
- `bun install --frozen-lockfile`
- OpenAPI client generation drift check.
- `bun run format:check`.
- `bun run lint` with Oxlint.
- `bun run typecheck` with `tsc --noEmit`.
- Vitest unit/component tests.
- React Doctor or equivalent React health scan when available.
- Bundle budget or bundle analysis check before production launch.
- `bun run build`.

Test placement rules:
- `src/` contains dashboard implementation code only.
- Do not colocate test files with components, routes, stores, hooks, utilities, or generated clients.
- Do not use adjacent `__tests__/` directories inside `src/`.
- Do not place `*.test.ts`, `*.spec.ts`, `*.test.tsx`, or `*.spec.tsx` beside implementation files.
- Unit and component tests live under `tests/unit/`.
- Integration tests for forms, stores, routing behavior, and API-client wiring live under `tests/integration/`.
- Playwright E2E tests live under `tests/e2e/`.
- Shared test render helpers, MSW handlers, fixtures, and factories live under `tests/helpers/`, `tests/fixtures/`, or `tests/factories/`.

Dashboard test directory shape:

```text
tests/
  unit/
    components/
    stores/
    utils/
  integration/
    forms/
    api-client/
    routing/
  e2e/
    auth.spec.ts
    onboarding.spec.ts
    core-flow.spec.ts
  fixtures/
  factories/
  helpers/
```

Required E2E coverage before production:
- Login/logout or session handoff.
- Primary dashboard happy path.
- One representative data table flow with filters/search/pagination.
- One representative form submission flow.
- Any flow involving money, destructive actions, or data loss.

## 25. Agent Implementation Rules

When an AI agent implements a dashboard project:
- Load TypeScript best-practices before editing TypeScript.
- Load or consult shadcn guidance before adding UI components.
- Before running Shadcn init, ask the human for the preset code; if the human provides no preset or responds with an empty value, use `bunx --bun shadcn@latest init --preset b6Z8CJysK --template react-router`.
- Consult current docs for Vite, TanStack Router, TanStack Query, TanStack Table, TanStack Form, nuqs, and the chosen OpenAPI generator before framework-specific code.
- Do not add Next.js, SSR, server functions, API routes, or database code.
- Keep dashboard, landing, and API repositories separate.
- Generate the API client from OpenAPI; do not hand-write endpoint types when a spec exists.
- Add or update `.env.example` whenever environment variables change.
- Verify format check, lint, typecheck, tests, and build before declaring work complete.

## 26. Anti-Patterns

The following are banned:

| Anti-pattern | Why banned | Alternative |
|---|---|---|
| Next.js for dashboard by default | Adds SSR/server features the dashboard does not need | Vite React SPA |
| Server functions in the dashboard repo | Violates mandatory separate backend API boundary | Backend API + OpenAPI client |
| API routes in dashboard | Blurs ownership and deployment boundaries | Separate API repo |
| Database access in dashboard | Security and architecture violation | Backend API owns persistence |
| `useEffect` data fetching | Race conditions, stale state, waterfalls | TanStack Query |
| API response data in Zustand | Duplicates cache and causes stale state | TanStack Query |
| Filter/search/page state in Zustand | Not refreshable or shareable | nuqs + URL state |
| React Hook Form/Formik | Fragments form standard | TanStack Form + Zod |
| ESLint as default linter | Slower and heavier than the Oxc baseline | Oxlint |
| Prettier or Biome as default formatter | Fragments the Oxc-first toolchain | Oxfmt |
| Missing Oxfmt check in CI | Formatting drift reaches review and production | `bun run format:check` |
| Treating Oxlint as typecheck | Oxlint does not replace TypeScript compiler correctness | `tsc --noEmit` |
| React dashboard without React Compiler | Leaves render optimization on manual memoization | React 19+ with React Compiler enabled |
| Manual `useMemo`/`useCallback` everywhere | Adds noise and fights compiler-first optimization | Clear code; memoize only for measured or semantic identity needs |
| `React.memo` by default | Premature manual optimization | Let React Compiler optimize render paths |
| Ignoring React Compiler diagnostics | Compiler skips unsafe components and performance degrades silently | Fix diagnostics or document an ADR exception |
| Importing charts/editors/upload clients in app shell | Bloats initial JavaScript | Route-level or component-level code splitting |
| Public production source maps | Exposes source code and implementation details | Private upload to monitoring provider only |
| Wildcard credentialed CORS | Allows unintended origins to make authenticated requests | Explicit dashboard origins only |
| Raw backend errors in UI | Leaks internals and confuses users | Map machine codes to safe messages |
| Session replay by default | Captures sensitive user behavior and data | Explicit approval plus PII masking |
| Plain `shadcn init` | Loses the custom theme/style baseline | Ask for preset, fallback to `b6Z8CJysK` |
| Initializing Shadcn without asking for preset | Can silently apply the wrong design system | Ask the human before init |
| Using `npx shadcn`, `pnpm dlx shadcn`, or `yarn dlx shadcn` | Violates Bun toolchain standard | `bunx --bun shadcn@latest ...` |
| Tailwind v3 baseline for new dashboards | Diverges from current styling standard | Tailwind CSS v4+ |
| MUI/Chakra/Mantine/Ant Design/DaisyUI as baseline | Fragments component, token, accessibility, and design-system behavior | Shadcn UI + Radix primitives |
| Custom modal/table/form/toast primitives by default | Rebuilds solved UI infrastructure and fragments accessibility | Shadcn + TanStack + Sonner |
| Component-only auth protection | Not a security boundary | Backend authorization + route UX guard |
| Editing generated OpenAPI files | Changes are overwritten and drift from spec | Regenerate from API spec |
| Hardcoded API URL | Breaks environments and previews | Environment config |
| Running `vite preview` in production | Preview server is not production-grade | Nginx static runtime |
| Running Bun/Node static server in production | Adds runtime surface the SPA does not need | Nginx serves `dist/` directly |
| Dashboard runtime image contains `node_modules` | Static runtime needs only built assets and Nginx | Final Nginx image with `dist/` only |
| Large dashboard runtime image without ADR | Bloats image and increases attack surface | Approved minimal Brotli Nginx runtime image |
| Deploying without Docker Compose | Makes setup and deployment inconsistent | Standard Docker Compose contract |
| Deploying without Nginx | Skips the standard TLS, compression, headers, and cache baseline | Nginx static runtime |
| Proxying to a dashboard app server | Adds an unnecessary runtime hop for static SPA | Nginx serves `dist/` directly |
| Missing SPA fallback | Deep links 404 on refresh | `try_files $uri $uri/ /index.html` |
| Immutable caching for `index.html` | Users keep stale app shells after deploy | no-cache / must-revalidate |
| Baking certificates into Docker images | Cert rotation requires image rebuild and leaks secrets | Let's Encrypt volume + Certbot sidecar |
| Installing Certbot in dashboard image | Bloats runtime and mixes concerns | Certbot sidecar |
| Serving app traffic over HTTP in production | Exposes sessions and user data to network interception | Port 80 ACME + HTTPS redirect only |
| Manual-only certificate renewal | Certificates expire during normal operation | Cron/systemd renewal script |
| Renewing certificates without Nginx reload | Nginx keeps serving the old certificate | Reload Nginx after renewal |
| Full-page blocking spinners | Poor dashboard UX | Skeletons and async states |
| Custom toast system | Fragments feedback and accessibility | Sonner |
| Service worker by default | Stale-cache deploy bugs | Add PWA only with explicit requirement |

## 27. Open Questions

All current architectural decisions are resolved for the baseline dashboard stack.

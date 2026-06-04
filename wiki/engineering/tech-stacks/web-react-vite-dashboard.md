# Web Stack: React + Vite Dashboard

Updated: 2026-06-04
Status: Draft
Sources: Internal Tech Stacks draft (2026-05-22, updated 2026-05-23); Internal Backend Stack draft (2026-04-15, updated 2026-05-23); Internal Infrastructure draft (2026-04-15, updated 2026-05-23); Internal Security Baseline draft (2026-04-15, updated 2026-05-22); Internal CI and Testing draft (2026-04-15, updated 2026-05-23); Internal TanStack Start OAuth/OIDC stack draft (2026-06-04)
Platform: Web / authenticated SaaS dashboard
Runtime: Bun
Framework: React + Vite SPA
Primary Use Case: Authenticated SaaS dashboards consuming a mandatory separate backend API through generated OpenAPI clients, with no SEO or SSR requirement, deployed as static Vite output through Docker Compose and Nginx
Raw: [2026-05-22-web-react-vite-dashboard-stack.md](../../../raw/engineering/tech-stacks/2026-05-22-web-react-vite-dashboard-stack.md); [2026-04-15-backend-bun-elysia-stack.md](../../../raw/engineering/tech-stacks/2026-04-15-backend-bun-elysia-stack.md); [2026-04-15-infra-docker-compose-nginx-stack.md](../../../raw/engineering/tech-stacks/2026-04-15-infra-docker-compose-nginx-stack.md); [2026-04-15-security-baseline.md](../../../raw/engineering/tech-stacks/2026-04-15-security-baseline.md); [2026-04-15-ci-testing-typescript-react.md](../../../raw/engineering/tech-stacks/2026-04-15-ci-testing-typescript-react.md); [2026-04-15-tech-stacks-overview.md](../../../raw/engineering/tech-stacks/2026-04-15-tech-stacks-overview.md); [2026-06-04-web-tanstack-start-oauth-oidc-stack.md](../../../raw/engineering/tech-stacks/2026-06-04-web-tanstack-start-oauth-oidc-stack.md)

## Summary

React + Vite SPA is the standard for authenticated SaaS dashboards. Dashboard repositories are standalone and separate from landing and API repositories. The dashboard does not need SEO, must not use SSR, and must not own backend logic.

The backend API is mandatory and owns authentication, authorization, validation, persistence, and business rules. The dashboard consumes the backend through generated OpenAPI clients and treats route guards as UX only.

SaaS administration screens for tenant settings, tenant users, subscriptions, entitlements, billing recovery, and soft-lock remediation are dashboard UI concerns only. The backend owns the administration rules, tenant state, subscription state, soft-lock enforcement, operator overrides, and audit records.

This article is the source of truth for dashboard-specific static deployment. Infrastructure docs retain generic Docker Compose and Nginx rules, but dashboard deployment details live here.

For SaaS apps that require OAuth2/OIDC callback handling, server-side code exchange, Redis-backed HttpOnly app sessions, protected server functions, BFF behavior, or full-stack app-server resource ownership, use [TanStack Start OAuth/OIDC App](web-tanstack-start-oauth-oidc.md) instead of this static SPA stack.

## Runtime And Toolchain

- Use Bun for installs, scripts, builds, and tests.
- Use Vite as the build tool.
- Use TypeScript strict mode.
- Use Oxlint as the required linter.
- Use Oxfmt as the required formatter.
- Use React 19+ with React Compiler enabled.
- Build output is static `dist/`.
- Dashboard has no production frontend server runtime.
- Do not add SSR, server functions, API routes, or database access.
- Production deployment must use Docker Compose and Nginx serving the built `dist/` directory.

Required script contract:

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

## React Compiler

React Compiler is mandatory for production builds.

- Install and configure `babel-plugin-react-compiler`.
- Configure React Compiler in Vite through `@vitejs/plugin-react`.
- Use `compilationMode: "infer"` and `panicThreshold: "critical_errors"` unless an ADR documents a different rollout.
- Do not add manual `useMemo`, `useCallback`, or `React.memo` by default.
- Manual memoization is allowed only for semantic identity requirements, third-party integration stability, expensive non-React computation, or measured regressions.
- Keep components pure and fix compiler diagnostics instead of silencing them.
- Use `startTransition` and `useDeferredValue` only where high-frequency or non-urgent UI work would otherwise block input.

Required Vite shape:

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

## OpenAPI Boundary

- Dashboard consumes backend OpenAPI through code generation.
- Generated files live under `src/lib/api/generated/` and are never hand-edited.
- The handwritten wrapper lives in `src/lib/api/client.ts`.
- The wrapper owns base URL, credentials mode, auth header behavior, request IDs when needed, and normalized errors.
- Do not call `fetch()` directly from feature code unless the generated client cannot express the endpoint and an inline comment documents why.
- Regenerate the client whenever backend OpenAPI changes.
- CI must catch generated-client drift.
- Client generation uses a committed/CI-generated OpenAPI artifact or an authenticated docs-enabled environment.
- Do not depend on unauthenticated production `/docs/json`.

## Project Structure

```text
src/
  routes/
  pages/
  features/
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
  fixtures/
  factories/
  helpers/
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
- Feature reusable code lives in `src/features/<domain>/`.
- Generated API code lives only under `src/lib/api/generated/**`.
- `src/` contains dashboard implementation code only; tests do not live beside implementation files.
- Unit/component tests live under `tests/unit/`, integration tests under `tests/integration/`, and Playwright tests under `tests/e2e/`.
- Shared test render helpers, MSW handlers, fixtures, and factories live under `tests/helpers/`, `tests/fixtures/`, or `tests/factories/`.
- `Dockerfile`, `.dockerignore`, `docker-compose.yml`, `nginx/nginx.conf.template`, `nginx/entrypoint.sh`, and `infra/scripts/renew-certs.sh` are required.
- Do not add `nginx/Dockerfile` for dashboard projects; use the approved Nginx runtime image directly.

## Shadcn Initialization

Shadcn UI is the default component baseline, but it must be initialized with a custom preset so theme, styling, component defaults, and tokens match the intended product design system.

Before running Shadcn init, always ask the human for the preset code.

Fallback default when the human gives no preset code or responds with an empty value:

```bash
bunx --bun shadcn@latest init --preset b6Z8CJysK --template react-router
```

Rules:
- Do not run plain `shadcn init`.
- Use `bunx --bun`, not `npx`, `pnpm dlx`, or `yarn dlx`.
- Use `--template react-router` for Vite React dashboard projects.
- Never decode, fetch, or inspect preset codes manually; pass preset codes directly to the Shadcn CLI.

## Routing, Query, Tables, And State

- TanStack Router is required; do not use React Router for app routing.
- Use folder/sub-folder route files, not flat dotted route files.
- Route protection is UX only; backend API remains the authorization boundary.
- Use `nuqs` for shareable URL search param state.
- URL state owns filters, search, sort, pagination, selected tabs, and shareable views.
- TanStack Query owns all server state.
- Current session is fetched from the backend API; frontend auth state is a convenience cache, not an authorization boundary.
- Do not store bearer access tokens in localStorage unless a security ADR explicitly accepts the risk and mitigations.
- Do not expose HttpOnly cookie contents to JavaScript.
- Do not use `useEffect` for data fetching.
- Query keys must include every input that changes returned data.
- Use explicit `staleTime` for low-volatility data.
- Avoid refetch storms; invalidate the narrowest affected query keys.
- Mutations must not be retried blindly when duplicate writes could create side effects.
- TanStack Table is required for interactive data tables.
- Use server-side pagination for fast-growing or unbounded transactional data.
- Do not use client-side filtering or sorting for unbounded transactional data.
- Zustand is allowed only for client-owned UI state, not API responses or URL-shareable state.

## Forms And UI

TanStack Form + Zod is required for submit-style forms. Backend validation remains authoritative.

| Area | Standard | Requirement |
| --- | --- | --- |
| Styling | Tailwind CSS v4+ | Required styling system. |
| Components | Shadcn UI | Required baseline. |
| Primitives | Radix UI through Shadcn | Required for accessible overlays and menus. |
| Toasts | Sonner | Required toast library. |
| Charts | Recharts through Shadcn chart patterns | Default charting library. |
| Icons | `lucide-react` | Default unless preset chooses another library. |
| Tables | TanStack Table + Shadcn presentation | Logic and visual shell split. |
| Forms | TanStack Form + Zod + Shadcn fields | State, validation, and field UI. |
| Command palette | Shadcn Command / `cmdk` | Recommended for complex dashboards. |
| Date/calendar | Shadcn Calendar + `date-fns` | Recommended default. |
| Motion | Tailwind transitions | Default; Framer Motion only for concrete needs. |

Rules:
- Do not use CSS Modules, styled-components, Emotion, or other CSS-in-JS.
- Do not add MUI, Chakra, Mantine, Ant Design, DaisyUI, Flowbite, or Headless UI as the dashboard component baseline.
- Dark and light mode are required.
- Store canonical theme preference as raw `localStorage.theme` with `light`, `dark`, or `system`.
- Prefer Shadcn Skeleton, Progress, Alert, Empty, and Sonner for async feedback.

## Performance Budget

| Asset / Metric | Budget | Rule |
| --- | ---: | --- |
| Authenticated app shell JavaScript | <= 250 KB gzip | Initial dashboard shell only; feature routes load separately. |
| Initial CSS | <= 80 KB gzip | Includes Tailwind output and component baseline. |
| Route chunks | Project-specific | Heavy domain routes must be lazy-loaded and measured. |
| Blocking app initialization | Minimal | Only session handoff and required bootstrap config may block first render. |

Rules:
- Every domain dashboard route should be lazy-loaded unless it is part of the always-visible app shell.
- Heavy charts, tables, editors, uploads, reporting, and admin modules must be split.
- Do not import Recharts, rich editors, upload clients, analytics, chat, session replay, or marketing widgets into the root app shell unless explicitly required.
- Run bundle analysis before production launch and after adding large dependencies.
- Fonts must be self-hosted or intentionally CDN-hosted with documented reason, and use `font-display: swap`.
- Public production source maps are banned. Upload source maps privately to monitoring providers if needed, then remove them from served static assets.

## Deployment: Docker Compose And Nginx

Dashboard production artifact is static `dist/`, served directly by Nginx from an immutable Docker image. This dashboard article is the source of truth for this deployment profile; infra's generic backend API proxy template is not used for dashboards.

Runtime rules:
- `bun run build` produces the deployable `dist/` directory.
- Production deployment must use Docker Compose.
- Production runtime must use Nginx, not Caddy or a generic static server.
- Nginx serves `dist/` directly from `/usr/share/nginx/html`.
- Nginx is the only host-facing dashboard service.
- Do not run `vite preview`, Bun, Node, Vite, or a custom static server in production.
- There is no internal dashboard app server and no `proxy_pass` to dashboard runtime.
- Hashed assets use immutable cache headers; `index.html` uses no-cache or must-revalidate.
- API requests go to the external API base URL.

Dockerfile contract:

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

Compose shape:

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
      CSP_CONNECT_SRC: "https://api.example.com"

  certbot:
    image: certbot/certbot:<pinned-version>
    volumes:
      - letsencrypt:/etc/letsencrypt
      - certbot-webroot:/var/www/certbot

volumes:
  letsencrypt:
  certbot-webroot:
```

TLS rules:
- Port `80` serves only ACME HTTP-01 challenge files and redirects all other traffic to HTTPS.
- Port `443` terminates TLS with Let's Encrypt certificates and serves static files.
- Certificates live in Docker volumes, never image layers.
- Certbot sidecar owns issuance and renewal.
- `infra/scripts/renew-certs.sh` runs renewal through cron/systemd or an approved scheduler and reloads Nginx after success.
- Certificate expiry monitoring is required.

Artifact hardening:
- Runtime stage contains only built static assets and Nginx config files.
- Runtime stage contains zero `node_modules`.
- Do not copy development `node_modules` into the final image.
- Do not install Certbot in the dashboard image.
- Do not mount `dist/` from host as the production deployment mechanism.
- Do not copy public source maps into served assets.
- `.dockerignore` must exclude `.git`, `.github`, `.env`, `.env.*`, caches, coverage, Playwright reports, test results, logs, and `*.map` unless a private source-map upload flow removes them before final image.
- Use pinned smallest production-suitable build and runtime image variants where available and compatible; never use `latest`.
- Every dashboard Docker build requires a final-image review confirming the image contains only `dist/`, Nginx config, entrypoint, and minimal runtime files.

## Security

- Never hardcode secrets or API keys.
- Never bake secrets into dashboard images.
- Only public environment values may be exposed to browser code, with a clear public prefix such as `VITE_PUBLIC_*`.
- Build must fail when required public configuration is missing or malformed.
- Configure CSP/security headers at the static gateway.
- Add third-party script/connect/image/frame sources only when actually used and reviewed.
- Do not use unsanitized `dangerouslySetInnerHTML`.
- Do not render raw backend errors, stack traces, SQL errors, Zod internals, or provider diagnostics to users.
- If cookie auth is used, backend must enforce CSRF protection and CORS must allow only explicit dashboard origins.
- No `Access-Control-Allow-Origin: *` with credentials.
- Session replay is disabled by default and requires explicit approval.
- Error monitoring is recommended before production, but PII scrubbing is mandatory.

## Testing And CI

Required checks:
- `bun install --frozen-lockfile`.
- OpenAPI client generation drift check.
- `bun run format:check` with Oxfmt.
- `bun run lint` with Oxlint.
- `bun run typecheck` with `tsc --noEmit`.
- Vitest unit/component tests.
- React Doctor or equivalent React health scan when available.
- Bundle budget or bundle analysis before production launch.
- `bun run build` with React Compiler enabled.
- Docker build and static artifact scan.

Test placement rules:
- `src/` contains dashboard implementation code only.
- Do not colocate test files with components, routes, stores, hooks, utilities, or generated clients.
- Do not use adjacent `__tests__/` directories inside `src/`.
- Do not place `*.test.ts`, `*.spec.ts`, `*.test.tsx`, or `*.spec.tsx` beside implementation files.
- Unit and component tests live under `tests/unit/`.
- Integration tests for forms, stores, routing behavior, and API-client wiring live under `tests/integration/`.
- Playwright E2E tests live under `tests/e2e/`.
- Shared test render helpers, MSW handlers, fixtures, and factories live under `tests/helpers/`, `tests/fixtures/`, or `tests/factories/`.

Required E2E coverage before production:
- Login/logout or session handoff.
- Primary dashboard happy path.
- One representative data table flow with filters/search/pagination.
- One representative form submission flow.
- Any flow involving money, destructive actions, or data loss.

## Anti-Patterns

| Anti-pattern | Why banned | Alternative |
| --- | --- | --- |
| Next.js for dashboard by default | Adds SSR/server features the dashboard does not need | Vite React SPA |
| API routes or server functions in dashboard | Violates backend API boundary | Backend API + OpenAPI client |
| Database access in dashboard | Security and ownership violation | Backend API owns persistence |
| `useEffect` data fetching | Race conditions and stale state | TanStack Query |
| API response data in Zustand | Duplicates server cache | TanStack Query |
| React Hook Form/Formik | Fragments form standard | TanStack Form + Zod |
| ESLint as default linter | Slower than Oxc baseline | Oxlint |
| Prettier or Biome as default formatter | Fragments Oxc-first tooling | Oxfmt |
| React without React Compiler | Leaves render optimization manual | React 19+ with Compiler |
| Manual `useMemo`/`useCallback` everywhere | Noise and premature optimization | Compiler-friendly pure code |
| Plain `shadcn init` | Loses custom theme baseline | Ask for preset, fallback to `b6Z8CJysK` |
| Importing charts/editors/upload clients in app shell | Bloats initial JS | Code splitting |
| Public source maps | Exposes source code | Private upload only |
| Colocated tests or adjacent `__tests__/` directories inside `src/` | Mixes implementation and test concerns | Top-level `tests/` hierarchy |
| Running `vite preview` in production | Not production-grade | Nginx static runtime |
| Proxying to a dashboard app server | Unnecessary runtime hop | Nginx serves `dist/` directly |
| Dashboard runtime image contains `node_modules` | Static runtime needs only built assets and Nginx | Final Nginx image with `dist/` only |
| Large dashboard runtime image without ADR | Bloats image and increases attack surface | Approved minimal Brotli Nginx runtime image |
| Missing SPA fallback | Deep links 404 | `try_files $uri $uri/ /index.html` |
| Baking certificates into images | Leaks secrets and blocks rotation | Let's Encrypt volume + Certbot sidecar |
| Wildcard credentialed CORS | Allows unintended origins | Explicit origins only |
| Session replay by default | Sensitive data risk | Explicit approval and masking |

## See Also

- [Astro Landing](web-astro-landing.md)
- [TanStack Start OAuth/OIDC App](web-tanstack-start-oauth-oidc.md)
- [Infrastructure: Docker Compose + Nginx](infra-docker-compose-nginx.md)
- [Security Baseline: Web Applications](security-web-app-baseline.md)
- [CI and Testing: TypeScript + React](ci-testing-typescript-react.md)
- [Backend: Bun + Elysia](backend-bun-elysia.md)

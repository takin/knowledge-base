---
name: web-react-vite-dashboard
description: "Use when building, reviewing, or modifying authenticated SaaS dashboard projects using React + Vite, Bun, TanStack Router/Query/Form/Table, Shadcn UI, Tailwind CSS, generated OpenAPI clients, static Nginx deployment, or dashboard CI/testing."
---

# Web Dashboard: React + Vite

This skill applies the repository wiki standard for authenticated SaaS dashboards. Use it before writing or reviewing code for React + Vite dashboard projects, dashboard routing, state management, generated OpenAPI clients, Shadcn UI, forms, tables, static deployment, Nginx runtime configuration, and dashboard CI checks.

Primary source of truth: `wiki/engineering/tech-stacks/web-react-vite-dashboard.md`.

Related standards:
- `wiki/engineering/tech-stacks/security-web-app-baseline.md`
- `wiki/engineering/tech-stacks/infra-docker-compose-nginx.md`
- `wiki/engineering/tech-stacks/ci-testing-typescript-react.md`
- `wiki/engineering/tech-stacks/backend-bun-elysia.md`
- `wiki/engineering/tech-stacks/agent-skills-typescript-product.md`

## Required Skill Loading

Before implementing dashboard code, load and apply these skills when available:

- `typescript-best-practices` for all TypeScript and JavaScript code.
- `shadcn` for UI component usage, theming, and customization.
- `react-state-management` for TanStack Query, TanStack Form, Zustand, URL state, and local state layering.

If one of these skills is unavailable, continue using the rules in this skill and state the gap in the final response.

## Stack Invariants

- Runtime and package manager is Bun.
- Build tool is Vite.
- App framework is React 19+.
- TypeScript strict mode is mandatory.
- React Compiler is mandatory for production builds.
- Oxlint is the required linter.
- Oxfmt is the required formatter.
- Build output is static `dist/`.
- Dashboard has no production frontend server runtime.
- Do not add SSR, server functions, API routes, or database access.
- Dashboard repositories are standalone and separate from landing and API repositories.
- A separate backend API is mandatory and owns authentication, authorization, validation, persistence, and business rules.
- Route guards are UX only; backend API remains the authorization boundary.
- Production deployment uses Docker Compose and Nginx serving the built `dist/` directory.

## Required Script Contract

Dashboard projects expose this baseline script shape:

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

Use Bun to run installs, scripts, builds, and tests.

## React Compiler

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
- Client generation uses a committed or CI-generated OpenAPI artifact, or an authenticated docs-enabled environment.
- Do not depend on unauthenticated production `/docs/json`.

## Project Structure

Use this baseline shape unless an existing project already has an equivalent convention:

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
- `Dockerfile`, `.dockerignore`, `docker-compose.yml`, `nginx/nginx.conf.template`, `nginx/entrypoint.sh`, and `infra/scripts/renew-certs.sh` are required for production dashboard projects.
- Do not add `nginx/Dockerfile`; use the approved Nginx runtime image directly.

## Shadcn Initialization

Shadcn UI is the default component baseline. It must be initialized with a custom preset so theme, styling, component defaults, and tokens match the intended product design system.

Before running Shadcn init, ask the human for the preset code.

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
- Use folder or sub-folder route files, not flat dotted route files.
- Use `nuqs` for shareable URL search param state.
- URL state owns filters, search, sort, pagination, selected tabs, and shareable views.
- TanStack Query owns all server state.
- Current session is fetched from the backend API.
- Frontend auth state is a convenience cache, not an authorization boundary.
- Do not store bearer access tokens in `localStorage` unless a security ADR explicitly accepts the risk and mitigations.
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
- Public production source maps are banned.
- If source maps are needed, upload them privately to monitoring providers and remove them from served static assets.

## Deployment: Docker Compose And Nginx

Dashboard production artifact is static `dist/`, served directly by Nginx from an immutable Docker image.

Runtime rules:
- `bun run build` produces the deployable `dist/` directory.
- Production deployment must use Docker Compose.
- Production runtime must use Nginx, not Caddy or a generic static server.
- Nginx serves `dist/` directly from `/usr/share/nginx/html`.
- Nginx is the only host-facing dashboard service.
- Do not run `vite preview`, Bun, Node, Vite, or a custom static server in production.
- There is no internal dashboard app server and no `proxy_pass` to dashboard runtime.
- Hashed assets use immutable cache headers.
- `index.html` uses no-cache or must-revalidate.
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

Compose baseline:

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

  certbot-renew:
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
- Certbot sidecars own issuance and renewal.
- `infra/scripts/renew-certs.sh` runs renewal through cron/systemd or an approved scheduler and reloads Nginx after success.
- Certificate expiry monitoring is required.

Artifact hardening:
- Runtime stage contains only built static assets and Nginx config files.
- Do not install Certbot in the dashboard image.
- Do not mount `dist/` from host as the production deployment mechanism.
- Do not copy public source maps into served assets.
- `.dockerignore` excludes `.git`, `.github`, `.env`, `.env.*`, caches, coverage, Playwright reports, test results, logs, and `*.map` unless a private source-map upload flow removes maps before the final image.

## Security

- Never hardcode secrets or API keys.
- Never bake secrets into dashboard images.
- Only public environment values may be exposed to browser code, with a clear public prefix such as `VITE_PUBLIC_*`.
- Build must fail when required public configuration is missing or malformed.
- Configure CSP and security headers at the static gateway.
- Add third-party script, connect, image, and frame sources only when actually used and reviewed.
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
- Vitest unit and component tests.
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

Do not introduce these patterns:

- Next.js for dashboard by default.
- API routes or server functions in dashboard.
- Database access in dashboard.
- `useEffect` data fetching.
- API response data in Zustand.
- React Hook Form or Formik.
- ESLint as default linter.
- Prettier or Biome as default formatter.
- React without React Compiler.
- Manual `useMemo`, `useCallback`, or `React.memo` everywhere.
- Plain `shadcn init`.
- Importing charts, editors, upload clients, analytics, chat, session replay, or marketing widgets into the app shell.
- Public source maps.
- Colocated tests or adjacent `__tests__/` directories inside `src/`.
- Running `vite preview` in production.
- Proxying to a dashboard app server.
- Missing SPA fallback such as `try_files $uri $uri/ /index.html`.
- Baking certificates into images.
- Wildcard credentialed CORS.
- Session replay by default.

## Implementation Workflow

When modifying or creating dashboard code:

1. Identify whether the change touches routing, API access, server state, forms, tables, UI components, auth/session handling, build tooling, Nginx, Docker, or CI.
2. Check the relevant invariant sections above before editing.
3. Keep routes thin; route files own route declarations, guards, search validation, loaders, and route-level wiring.
4. Put reusable domain behavior under `src/features/<domain>/`.
5. Use generated OpenAPI clients through the handwritten API wrapper.
6. Put server data in TanStack Query, URL-shareable state in URL search params, and only client-owned UI state in Zustand or local component state.
7. Use TanStack Form + Zod for submit-style forms.
8. Preserve dark and light mode behavior.
9. Avoid bloating the app shell; lazy-load heavy dashboard domains.
10. Add or update tests for affected routing, forms, tables, auth/session, and critical business flows.
11. Run the smallest relevant verification commands available in the project.
12. In the final response, report which dashboard gates were verified and which were unavailable.

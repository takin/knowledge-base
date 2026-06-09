# Repository Structure: Standalone Repos

Updated: 2026-06-09
Status: Draft
Sources: Internal standalone repository structure draft (2026-06-09)
Platform: Cross-stack repository structure
Runtime: N/A
Framework: N/A
Primary Use Case: Standard repository boundaries for products split across backend API, dashboard, landing page, and mobile deployables
Raw: [2026-06-09-repository-structure-standalone.md](../../../raw/engineering/tech-stacks/2026-06-09-repository-structure-standalone.md)

## Summary

Standalone repositories are the default structure for new products. Each major deployable surface owns its own repository, CI pipeline, deployment boundary, environment configuration, and release process. Monorepos are allowed only when an Architecture Decision Record proves stronger coupling, shared release cadence, or mature shared packages make the monorepo simpler than cross-repository contract management.

Default repositories:

| Surface | Repository | Stack |
| --- | --- | --- |
| Backend API | `<product>-api` | Bun + Elysia |
| Dashboard | `<product>-dashboard` | React + Vite SPA |
| Landing page | `<product>-landing` | Astro |
| Mobile app | `<product>-mobile` | React Native + Expo |

## Decision

Use one standalone repository per deployable product surface by default:

```text
product-api/
product-dashboard/
product-landing/
product-mobile/
```

Do not default to a single product monorepo with `apps/api`, `apps/dashboard`, `apps/landing`, `apps/mobile`, and shared `packages/*`. That layout requires an explicit product ADR.

## Naming Convention

Repository names use strict surface suffixes:

| Surface | Required Naming Pattern |
| --- | --- |
| Backend API | `<product>-api` |
| Dashboard | `<product>-dashboard` |
| Landing page | `<product>-landing` |
| Mobile app | `<product>-mobile` |

Prefer `<product>-api` over `<product>-backend`. `api` names the external contract and primary service boundary. `backend` is too broad because it can ambiguously include workers, admin jobs, internal tools, integrations, and infrastructure scripts. A repository that owns API plus workers is still `<product>-api` because the API contract is the primary boundary.

Avoid alternatives such as `<product>-frontend`, `<product>-web`, `<product>-app`, or `<product>-server` unless an existing product naming convention already requires them.

## Ownership Boundaries

### Backend API Repository

The API repository owns HTTP API routes, authentication, authorization, business rules, database schema, migrations, workers, queues, webhooks, OpenAPI contract generation, the API Docker image, runtime deployment, observability, and audit logs.

The API repository is authoritative for product behavior. Dashboard and mobile repositories consume it through the published OpenAPI contract.

### Dashboard Repository

The dashboard repository owns authenticated product UI, dashboard routes and layouts, generated OpenAPI client code, the handwritten API client wrapper, UI state, route state, forms, tables, charts, static dashboard builds, and dashboard Docker/Nginx deployment.

The dashboard must not own backend business logic, database code, API routes, workers, or webhooks. Dashboard route guards are user experience only; the API remains the authorization boundary.

### Landing Repository

The landing repository owns public marketing pages, SEO pages, blog/content pages, static Astro builds, sitemap, RSS, robots.txt, metadata, and public lead/contact forms that submit to the API or approved external services.

The landing repository must not contain dashboard routes, authenticated app routes, backend routes, or database code.

### Mobile Repository

The mobile repository owns the React Native / Expo app, Expo Router navigation, native UI, NativeWind styling, generated OpenAPI client code, secure token storage, mobile tests, EAS build configuration, and app store release configuration.

The mobile repository must not be embedded in the dashboard repository by default. Mobile has a slower release and adoption cycle than web, so it needs an independent repository, CI pipeline, and release process.

## Standard Root Layouts

Backend API:

```text
src/
  index.ts
  app.ts
  controllers/
  services/
  lib/
  db/
    schema/
    migrations/
  jobs/
  emails/
  types/
  utils/
tests/
  unit/
  integration/
  e2e/
  fixtures/
  factories/
  helpers/
Dockerfile
docker-compose.yml
.env.example
drizzle.config.ts
```

Dashboard:

```text
src/
  routes/
  pages/
  features/
  components/
  lib/
    api/
      client.ts
      errors.ts
      generated/
  styles/
public/
nginx/
tests/
  unit/
  integration/
  e2e/
  fixtures/
  factories/
  helpers/
infra/
Dockerfile
docker-compose.yml
.env.example
vite.config.ts
```

Landing:

```text
src/
  content/
  pages/
  layouts/
  components/
  lib/
  styles/
public/
astro.config.mjs
.env.example
```

Mobile:

```text
app/
src/
  features/
  components/
  lib/
    api/
      client.ts
      errors.ts
      generated/
  styles/
assets/
tests/
  unit/
  integration/
  e2e/
  fixtures/
  factories/
  helpers/
app.json
eas.json
.env.example
```

## Cross-Repository Contracts

OpenAPI is the official contract between backend, dashboard, mobile, public SDKs, partner integrations, and machine clients.

Rules:

- The backend API repository generates the OpenAPI artifact.
- Dashboard and mobile repositories consume generated clients from the OpenAPI artifact.
- Generated client code is never hand-edited.
- API contract changes must be reviewed before dashboard or mobile updates.
- CI should detect generated-client drift.
- Public or partner-facing breaking changes require versioning, a migration plan, and a deprecation window.

Preferred contract flow:

```text
api repo -> OpenAPI JSON artifact -> dashboard/mobile client generation
```

## OpenAPI Publishing

Publish OpenAPI as a versioned contract artifact from the API repository.

Default publishing model:

- The API repository publishes `openapi.json` to GitHub Releases on every API release.
- Release asset file names use `openapi.v<version>.json`.
- Dashboard and mobile CI download the pinned version or latest compatible version.
- The OpenAPI artifact includes a checksum.
- Specs are never hand-copied between repositories.

Recommended release asset shape:

```text
openapi.v1.4.0.json
openapi.v1.4.0.sha256
openapi.v1.4.0.changelog.md
```

Dashboard and mobile repositories should pin the consumed API contract version in a committed config file or package script variable so upgrades are explicit in pull requests.

An optional package-registry contract may be added later when GitHub Release assets become painful:

```text
@org/<product>-api-contract
```

If created, this package contains OpenAPI JSON, checksum, contract changelog, and optional generated metadata. It must not contain handwritten frontend types or business logic. OpenAPI remains the source of truth.

## Shared Types And Design Tokens

Do not create a separate shared package repository by default. Prefer OpenAPI for API request and response types, duplicated small UI constants only when stable and harmless, and product-specific SDK packages only when there is a real external consumer or repeated internal pain.

Avoid copying backend domain types into frontend repositories, importing backend source directly from dashboard or mobile, or creating shared packages before boundaries are stable.

Design tokens may be shared across dashboard, landing, and mobile, but they should not force a monorepo.

Default token policy:

- Start by duplicating a small token baseline across repositories while the product is early.
- Create a shared token package only after repeated cross-repository need appears.
- Prefer sharing tokens before sharing full UI components.
- Avoid creating a shared UI package before component boundaries and design language stabilize.

Optional future token package:

```text
@org/<product>-design-tokens
```

## Environment Variables

Each repository owns its own `.env.example`.

Rules:

- Never share `.env` files across repositories.
- Browser-exposed variables must use public prefixes such as `VITE_PUBLIC_*` or framework equivalent.
- Backend secrets stay only in the API repository and API deployment environment.
- Landing repositories may expose only public runtime/build configuration.
- Mobile public config and secret handling must follow Expo/EAS conventions.
- Updating an environment variable requires updating that repository's `.env.example` in the same change.

## CI And Deployment Boundaries

Each repository owns its own CI pipeline.

| Repository | Deployment |
| --- | --- |
| `<product>-api` | Docker Compose + Nginx gateway |
| `<product>-dashboard` | Static Vite build served by Nginx/Docker Compose |
| `<product>-landing` | Cloudflare Pages static deployment |
| `<product>-mobile` | Expo/EAS builds and app store release flow |

Each repository should deploy independently. Coordinated releases are allowed, but repositories should not require lockstep deployment unless the API contract change is breaking and an approved migration plan requires coordination.

Baseline CI expectations:

- Backend API CI runs Bun install, format, lint, typecheck, unit/integration/E2E tests, OpenAPI generation/drift, migration checks, Docker build/smoke, and GitHub Release artifact publishing on release.
- Dashboard CI runs Bun install, format, lint, typecheck, generated-client drift, unit/integration/E2E tests, production build, static artifact scan, and Docker build.
- Landing CI runs Bun install, typecheck, lint, tests where configured, Astro build, and link/SEO smoke checks when configured.
- Mobile CI runs install, typecheck, lint, unit tests, integration tests where practical, Maestro E2E where practical, generated-client drift, and EAS build checks before release.

## Versioning And Compatibility

Backend breaking API changes require versioning or a migration plan. Dashboard and mobile should support compatible API changes independently. Landing deploys should not be blocked by backend, dashboard, or mobile releases.

Recommended compatibility model:

```text
backend supports current dashboard + supported mobile app versions
dashboard tracks latest stable backend API
mobile supports backend compatibility window
landing is mostly independent
```

Minimum compatibility rule:

- Backend API must support the current dashboard plus supported mobile app versions for at least 90 days after a breaking change is released.
- For mobile, support at least the latest released app version plus the previous supported version during the 90-day window.
- Breaking API changes should produce clear deprecation notes in the OpenAPI changelog.

## Monorepo Exception

A monorepo may be approved when at least one is true:

- The product has one tightly coupled release train.
- Shared packages are numerous, stable, and genuinely reduce duplication.
- The same team owns all surfaces and deployment is coordinated.
- Cross-repository contract management becomes more expensive than monorepo orchestration.
- The product is an internal tool where repository separation adds no operational value.

If approved, the monorepo should still preserve clear boundaries:

```text
apps/
  api/
  dashboard/
  landing/
  mobile/
packages/
  ui/
  schema/
  config/
```

Use Turborepo or equivalent task orchestration only inside approved monorepos.

## Anti-Patterns

| Anti-pattern | Why banned | Alternative |
| --- | --- | --- |
| Defaulting every product to monorepo | Couples unrelated deployables | Standalone repositories |
| Dashboard and API in one repository by default | Blurs frontend/backend ownership | Separate `dashboard` and `api` repositories |
| Landing pages inside dashboard repository | Weakens SEO/static publishing boundary | Separate Astro landing repository |
| Mobile app inside dashboard repository by default | Couples web and mobile release cycles | Separate mobile repository |
| Frontend importing backend source | Breaks service boundary | OpenAPI-generated client |
| Copying backend types into frontend | Drifts silently | OpenAPI contract |
| Shared package before stability | Adds release/version overhead | Duplicate small stable constants temporarily |
| Shared UI package before design maturity | Freezes unstable components too early | Share tokens first, package UI later |
| One CI pipeline for all surfaces | Slower and noisier feedback | Per-repository CI |
| Lockstep deployment for non-breaking changes | Slows delivery | Backward-compatible API contracts |
| Backend secrets in dashboard, landing, or mobile repositories | Security risk | API-only secret ownership |
| Hand-copying OpenAPI specs across repositories | Manual drift risk | Published GitHub Release artifacts |

## See Also

- [Backend: Bun + Elysia](backend-bun-elysia.md)
- [React + Vite Dashboard](web-react-vite-dashboard.md)
- [Astro Landing](web-astro-landing.md)
- [Mobile: React Native + Expo](mobile-react-native-expo.md)
- [CI and Testing: TypeScript + React](ci-testing-typescript-react.md)

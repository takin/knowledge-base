# Repository Structure Stack: Standalone Repos

Source URL: Internal draft
Collected: 2026-06-09
Published: 2026-06-09
Updated: 2026-06-09
Status: Draft
Scope: Engineering repository structure standard for products split across backend API, dashboard, landing page, and mobile repositories.

---

## Source Bundle Purpose

This draft defines the standalone repository standard for product engineering. It is intentionally focused on repository boundaries, ownership, contract publishing, and cross-repo coordination. Stack-specific implementation rules remain in their own backend, dashboard, landing, mobile, infrastructure, security, and CI standards.

## Summary

Standalone repositories are the default structure for new products.

Each major deployable surface owns its own repository:

| Surface | Repository | Stack |
| --- | --- | --- |
| Backend API | `<product>-api` | Bun + Elysia |
| Dashboard | `<product>-dashboard` | React + Vite SPA |
| Landing page | `<product>-landing` | Astro |
| Mobile app | `<product>-mobile` | React Native + Expo |

The monorepo model is no longer the default. Use a monorepo only when a product has strong coupling, shared release cadence, or shared internal packages that materially reduce complexity. Otherwise, separate repositories keep ownership, CI, deployment, access control, and release responsibility clearer.

## Decision

Default to standalone repositories per deployable product surface.

A product with backend, dashboard, landing, and mobile surfaces should normally use:

```text
product-api/
product-dashboard/
product-landing/
product-mobile/
```

Do not create a single repository containing:

```text
apps/
  api/
  dashboard/
  landing/
  mobile/
packages/
  ...
```

unless an Architecture Decision Record approves a monorepo for that product.

## Naming Convention

Use these repository names as the strict default:

| Surface | Required Naming Pattern |
| --- | --- |
| Backend API | `<product>-api` |
| Dashboard | `<product>-dashboard` |
| Landing page | `<product>-landing` |
| Mobile app | `<product>-mobile` |

Prefer `<product>-api`, not `<product>-backend`.

Rationale:

- `api` names the external contract and primary service boundary.
- `backend` is broader and can ambiguously include workers, admin jobs, internal tools, integrations, and infrastructure scripts.
- If the repository owns API plus workers, it is still `<product>-api` because the API contract is the primary boundary.

Avoid alternatives such as `<product>-frontend`, `<product>-web`, `<product>-app`, or `<product>-server` unless an existing product naming convention already requires them.

## Repository Ownership

### Backend API Repository

The API repository owns:

- HTTP API routes.
- Authentication and authorization.
- Business rules.
- Database schema and migrations.
- Workers and queues.
- Webhooks.
- OpenAPI contract generation.
- API Docker image and runtime deployment.
- Backend observability and audit logs.

Standard repository name:

```text
<product>-api
```

The API repository is authoritative for product behavior. Dashboard and mobile repositories consume it through the published OpenAPI contract.

### Dashboard Repository

The dashboard repository owns:

- Authenticated product UI.
- Dashboard routes and layouts.
- Generated OpenAPI client.
- API client wrapper.
- UI state, route state, forms, tables, and charts.
- Static dashboard build.
- Dashboard Docker/Nginx deployment.

Standard repository name:

```text
<product>-dashboard
```

The dashboard must not own backend business logic, database code, API routes, workers, or webhooks. Dashboard route guards are user experience only; the API remains the authorization boundary.

### Landing Repository

The landing repository owns:

- Public marketing pages.
- SEO pages.
- Blog or content pages.
- Static Astro build.
- Sitemap, RSS, robots.txt, and metadata.
- Public lead/contact forms that submit to the API or approved external services.

Standard repository name:

```text
<product>-landing
```

The landing repository must not contain dashboard routes, authenticated app routes, backend routes, or database code.

### Mobile Repository

The mobile repository owns:

- React Native / Expo app.
- Expo Router navigation.
- Native UI and NativeWind styling.
- Generated OpenAPI client.
- Secure token storage.
- Mobile-specific tests.
- EAS build and app store release configuration.

Standard repository name:

```text
<product>-mobile
```

The mobile repository must not be embedded in the dashboard repository by default. Mobile has a slower release and adoption cycle than web, so it needs an independent repository, CI pipeline, and release process.

## Standard Root Layouts

### Backend API

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

### Dashboard

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

### Landing

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

### Mobile

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

### API Contract

OpenAPI is the official contract between backend, dashboard, mobile, public SDKs, partner integrations, and machine clients.

Rules:

- Backend API repository generates the OpenAPI artifact.
- Dashboard and mobile repositories consume generated clients from the OpenAPI artifact.
- Generated client code is never hand-edited.
- API contract changes must be reviewed before dashboard or mobile updates.
- CI should detect generated-client drift.
- Public or partner-facing breaking changes require versioning, a migration plan, and a deprecation window.

Preferred contract flow:

```text
api repo -> OpenAPI JSON artifact -> dashboard/mobile client generation
```

### OpenAPI Publishing

Publish OpenAPI as a versioned contract artifact.

Default publishing model:

- The API repository publishes `openapi.json` to GitHub Releases on every API release.
- Release asset file name uses `openapi.v<version>.json`.
- Dashboard and mobile CI download the pinned version or latest compatible version.
- The OpenAPI artifact includes a checksum.
- Do not hand-copy OpenAPI specs between repositories.

Recommended release asset shape:

```text
openapi.v1.4.0.json
openapi.v1.4.0.sha256
openapi.v1.4.0.changelog.md
```

Dashboard and mobile repositories should pin the consumed API contract version in a committed config file or package script variable so upgrades are explicit in pull requests.

### Optional API Contract Package

Add a package-registry contract only when GitHub Release assets become painful.

Optional package name:

```text
@org/<product>-api-contract
```

If created, the package should contain:

- OpenAPI JSON.
- Checksum.
- Contract changelog.
- Optional generated metadata.

The package should not contain handwritten frontend types or business logic. OpenAPI remains the source of truth.

### Shared Types

Do not create a separate shared package repository by default.

Prefer:

- OpenAPI for API request and response types.
- Duplicated small UI constants only when stable and harmless.
- Product-specific SDK package only when there is a real external consumer or repeated internal pain.

Avoid:

- Copy-pasting backend domain types into frontend repositories.
- Importing backend source directly from dashboard or mobile.
- Creating shared packages before the boundaries are stable.

### Design Tokens

Design tokens may be shared across dashboard, landing, and mobile, but they should not force a monorepo.

Default policy:

- Start by duplicating a small token baseline across repos while the product is early.
- Create a shared token package only after repeated cross-repo need appears.
- Prefer sharing tokens before sharing full UI components.
- Avoid creating a shared UI package before component boundaries and design language stabilize.

Preferred options:

| Option | Use When |
| --- | --- |
| Documented duplicated baseline | Product is early and design system is not stable |
| Generated token artifact | Tokens are machine-generated from a source of truth |
| Published internal package | Multiple repos need the same stable design system |

Optional future token package:

```text
@org/<product>-design-tokens
```

Do not introduce a shared design-system package until the product has enough repeated UI to justify it.

### Environment Variables

Each repository owns its own `.env.example`.

Rules:

- Never share `.env` files across repositories.
- Browser-exposed variables must use public prefixes such as `VITE_PUBLIC_*` or framework equivalent.
- Backend secrets stay only in the API repository and API deployment environment.
- Landing repositories may expose only public runtime/build configuration.
- Mobile public config and secret handling must follow Expo/EAS conventions.
- Updating an environment variable requires updating that repository's `.env.example` in the same change.

## CI Expectations

Each repository owns its own CI pipeline.

### Backend API CI

Required checks:

- Install with Bun.
- Format check.
- Lint.
- Typecheck.
- Unit tests.
- Integration tests.
- End-to-end tests for critical flows.
- OpenAPI generation and drift check.
- Migration check.
- Docker build and smoke test.
- GitHub Release artifact publishing on release.

### Dashboard CI

Required checks:

- Install with Bun.
- Format check.
- Lint.
- Typecheck.
- Generated API client drift check.
- Unit tests.
- Integration tests.
- End-to-end tests.
- Production build.
- Static artifact scan.
- Docker build.

### Landing CI

Required checks:

- Install with Bun.
- Typecheck.
- Lint.
- Tests where configured.
- Astro build.
- Link and SEO smoke checks when configured.

### Mobile CI

Required checks:

- Install with the approved mobile package/runtime workflow.
- Typecheck.
- Lint.
- Unit tests.
- Integration tests where practical.
- Maestro E2E flows where practical.
- Generated API client drift check.
- EAS build checks before release.

## Deployment Boundaries

| Repository | Deployment |
| --- | --- |
| `<product>-api` | Docker Compose + Nginx gateway |
| `<product>-dashboard` | Static Vite build served by Nginx/Docker Compose |
| `<product>-landing` | Cloudflare Pages static deployment |
| `<product>-mobile` | Expo/EAS builds and app store release flow |

Each repository should deploy independently.

Coordinated releases are allowed, but repositories should not require lockstep deployment unless the API contract change is breaking and an approved migration plan requires coordination.

## Versioning And Release Coordination

Rules:

- Backend breaking API changes require versioning or a migration plan.
- Dashboard and mobile should support compatible API changes independently.
- Landing deploys should not be blocked by backend, dashboard, or mobile releases.
- Mobile releases need longer compatibility windows because users may not update immediately.
- Backend must preserve mobile-compatible API behavior across supported app versions.

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

## When Monorepo Is Allowed

A monorepo may be approved when at least one is true:

- The product has one tightly coupled release train.
- Shared packages are numerous, stable, and genuinely reduce duplication.
- The same team owns all surfaces and deployment is coordinated.
- Cross-repo contract management becomes more expensive than monorepo orchestration.
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
| Copy-pasting backend types into frontend | Drifts silently | OpenAPI contract |
| Shared package before stability | Adds release/version overhead | Duplicate small stable constants temporarily |
| Shared UI package before design maturity | Freezes unstable components too early | Share tokens first, package UI later |
| One CI pipeline for all surfaces | Slower and noisier feedback | Per-repository CI |
| Lockstep deployment for non-breaking changes | Slows delivery | Backward-compatible API contracts |
| Backend secrets in dashboard, landing, or mobile repositories | Security risk | API-only secret ownership |
| Hand-copying OpenAPI specs across repositories | Manual drift risk | Published GitHub Release artifacts |

## Resolved Decisions

- Repository names use `<product>-api`, `<product>-dashboard`, `<product>-landing`, and `<product>-mobile`.
- `<product>-api` is preferred over `<product>-backend`.
- OpenAPI artifacts are published from the API repository as versioned GitHub Release assets.
- A package-registry API contract package is optional and added only when release-asset consumption becomes painful.
- Design tokens start duplicated and become a shared package only after repeated cross-repo need.
- Backend API compatibility must support the current dashboard plus supported mobile versions for at least 90 days.

## Open Questions

No open questions remain for this draft. Future implementation details should be handled in stack-specific standards or product-level Architecture Decision Records.

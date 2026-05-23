# CI and Testing Stack: TypeScript + React

Updated: 2026-05-24
Status: Draft
Sources: Internal Tech Stacks draft (2026-04-15, updated 2026-05-23)
Platform: Delivery / quality
Runtime: Bun
Framework: TypeScript + React
Primary Use Case: Oxc linting/formatting, type checking, unit/integration/component/E2E tests, React diagnostics, React Compiler checks, bundle budgets, static artifact scans, and GitHub Actions pipelines
Raw: [2026-04-15-ci-testing-typescript-react.md](../../../raw/engineering/tech-stacks/2026-04-15-ci-testing-typescript-react.md); [2026-05-22-web-react-vite-dashboard-stack.md](../../../raw/engineering/tech-stacks/2026-05-22-web-react-vite-dashboard-stack.md)

## Summary

This stack defines quality gates for TypeScript React products. New dashboard projects use Oxlint, Oxfmt, strict TypeScript, Vitest, Testing Library, Playwright, React Doctor, React Compiler diagnostics, dependency audit, bundle analysis, Docker build, and static artifact scans.

## Linting And Formatting

- Oxlint is the default linter for new TypeScript + React dashboard projects.
- Oxfmt is the default formatter for new TypeScript + React dashboard projects.
- ESLint is not the default linter for new dashboard projects. Add it only as a targeted exception when Oxlint cannot cover a concrete risk.
- Prettier and Biome are not the default formatter for new dashboard projects.
- Oxfmt formatting must be checked with `oxfmt --check`.
- Oxlint does not replace TypeScript typechecking; `tsc --noEmit` remains mandatory.

## TypeScript

- Strict mode is mandatory.
- No `any` types without an inline lint suppression comment and a short explanation.
- Prefer `unknown` over `any` when the type is genuinely unknown.
- Backend API boundaries are typed through route schemas and generated OpenAPI clients. Dashboard-only code must not define server functions.

## Testing

| Layer | Tool | Scope |
| --- | --- | --- |
| Unit / integration | Vitest | Pure functions, schemas, backend services/controllers where present, stores |
| Component | Vitest + Testing Library | Shadcn/custom component rendering and interaction |
| End-to-end | Playwright | Auth, onboarding, core feature happy path, destructive/payment flows |
| React health scan | React Doctor | React correctness, performance, security, and architecture diagnostics |
| Bundle analysis | Project-approved Vite bundle analyzer or size-limit check | Initial app shell, route chunks, dependency growth |

Test placement rules:
- Test files live under the top-level `tests/` directory, never adjacent to production implementation files.
- Unit tests live in `tests/unit/`.
- Integration tests live in `tests/integration/`.
- E2E tests live in `tests/e2e/`.
- Shared test helpers, fixtures, factories, and mocks live in `tests/helpers/`, `tests/fixtures/`, or `tests/factories/`.
- Do not use adjacent `__tests__/` directories inside `src/`.
- Do not place `*.test.ts`, `*.spec.ts`, `*.test.tsx`, or `*.spec.tsx` beside implementation files.
- Vitest and Playwright config must target the top-level test directories instead of scanning colocated tests in `src/`.

## React Doctor

React Doctor complements Oxlint, TypeScript, Vitest, Testing Library, and Playwright with React-specific diagnostics.

- React Doctor runs in CI on every PR that touches React/frontend code.
- Error-level diagnostics block merge.
- Release candidates must pass a full React Doctor scan.
- Use Bun-first commands.

Recommended PR/CI command:

```bash
bunx --bun react-doctor@latest . --yes --fail-on error --offline --annotations
```

Recommended release-candidate command:

```bash
bunx --bun react-doctor@latest . --yes --full --fail-on error --offline
```

## React Compiler CI

React Compiler is mandatory for dashboard production builds.

- Production build must run with React Compiler enabled.
- CI must fail if the configured build fails because of React Compiler critical diagnostics.
- Do not silence compiler diagnostics without an ADR explaining the unsafe pattern and remediation plan.
- Avoid adding `useMemo`, `useCallback`, or `React.memo` as a reflexive fix. Prefer pure, compiler-friendly code first.

## Bundle, Source Map, And Static Artifact Checks

- CI should fail or warn when the authenticated dashboard shell exceeds the project budget.
- Bundle analysis must identify top dependency contributors when bundle size changes materially.
- Static production artifacts must not include public `.map` files unless a security exception explicitly allows them.
- If source maps are needed for monitoring, CI uploads them privately to the monitoring provider and removes them from served assets.
- CI must verify the Docker image does not contain `.env`, `.git`, test reports, coverage, Playwright artifacts, local caches, or dependency install caches.

## Pull Request Pipeline

1. `bun install`.
2. Oxfmt formatting check: `oxfmt --check`.
3. Oxlint lint check.
4. TypeScript type check: `tsc --noEmit`.
5. React Compiler production build diagnostics.
6. React Doctor frontend health scan.
7. `bun audit` dependency vulnerability scan.
8. TanStack supply-chain guard for affected `@tanstack/*` versions and advisory indicators.
9. Vitest unit and integration tests.
10. Bundle budget or bundle analysis for frontend changes.
11. Docker build.
12. Static artifact scan for source maps, secrets, source control metadata, reports, coverage, Playwright artifacts, and caches.

When a repository includes a standalone backend API, CI also runs backend checks from [Backend: Bun + Elysia](backend-bun-elysia.md): OpenAPI generation/drift, JWT/JWKS auth tests, RBAC/scope tests, tenant isolation tests, rate limit tests, idempotency tests, CORS/cookie/CSRF tests where applicable, webhook tests, worker/queue retry tests, and API/worker Docker smoke tests.

## Merge To Main Pipeline

1. All PR checks.
2. Playwright E2E tests.
3. Docker image build and push to registry.
4. Deploy to staging environment.

## Rules

- All secrets used in Actions are stored as GitHub Actions Secrets, never in workflow YAML.
- Production deployments require a passing staging deployment as a prerequisite.
- Do not use force pushes to `main` or `production` branches.
- During active supply-chain incidents, CI may temporarily use `bun install --ignore-scripts`; document the incident link and restore normal installs after provenance review.

## See Also

- [React + Vite Dashboard](web-react-vite-dashboard.md)
- [Security Baseline: Web Applications](security-web-app-baseline.md)
- [Infrastructure: Docker Compose + Nginx](infra-docker-compose-nginx.md)

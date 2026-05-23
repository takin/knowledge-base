# CI and Testing Stack: TypeScript + React

Source URL: Internal draft
Collected: 2026-04-15
Published: 2026-04-15
Updated: 2026-05-23
Status: Draft
Scope: TypeScript, React, CI, testing, linting, type checking, and release quality gates

---

## 15. Code Quality

### 15.1 Linting And Formatting: Oxc Toolchain

- Oxlint is the default linter for new TypeScript + React dashboard projects.
- Oxfmt is the default formatter for new TypeScript + React dashboard projects.
- ESLint is not the default linter for new dashboard projects. Add it only as a targeted exception when Oxlint cannot cover a concrete risk.
- Prettier and Biome are not the default formatter for new dashboard projects.
- Lint and format checks run in CI on every pull request. A failing lint or format check blocks merge.
- Oxfmt formatting must be checked with `oxfmt --check`.
- Oxlint does not replace TypeScript typechecking; `tsc --noEmit` remains mandatory.

### 15.2 TypeScript

- **Strict mode** is mandatory (`"strict": true` in `tsconfig.json`).
- No `any` types without an inline lint suppression comment and a short explanation.
- Prefer `unknown` over `any` when the type is genuinely unknown.
- All backend API boundaries are typed through route schemas and generated OpenAPI clients. Dashboard-only code must not define server functions.

### 15.3 Testing

| Layer | Tool | Scope |
|---|---|---|
| Unit / integration | **Vitest** | Pure functions, Zod/domain schemas, backend services/controllers where present, Zustand stores |
| Component | **Vitest + Testing Library** | Shadcn/custom component rendering and interaction |
| End-to-end | **Playwright** | Critical user flows (auth, onboarding, core feature happy path) |
| React health scan | **React Doctor** | React correctness, performance, security, and architecture diagnostics |
| Bundle analysis | **Project-approved Vite bundle analyzer or size-limit check** | Initial app shell, route chunks, and dependency growth |

Rules:
- Every backend controller/service that touches PostgreSQL, Redis, queues, auth, or storage must have integration coverage against real test dependencies where feasible — no mocks for database calls in backend integration tests.
- E2E tests cover: login/logout, the single most important user action per product, and any flow that handles money or data loss.
- Test files live under the top-level `tests/` directory, never adjacent to production implementation files.
- Unit tests live in `tests/unit/`.
- Integration tests live in `tests/integration/`.
- E2E tests live in `tests/e2e/`.
- Shared test helpers, fixtures, factories, and mocks live in `tests/helpers/`, `tests/fixtures/`, or `tests/factories/`.
- Do not use adjacent `__tests__/` directories inside `src/`.
- Do not place `*.test.ts`, `*.spec.ts`, `*.test.tsx`, or `*.spec.tsx` beside implementation files.
- Vitest and Playwright config must target the top-level test directories instead of scanning colocated tests in `src/`.

### 15.4 React Doctor

**React Doctor** is mandatory for all React frontend and fullstack applications.

React Doctor complements Oxlint, TypeScript, Vitest, Testing Library, and Playwright with React-specific diagnostics. It is used to catch correctness, performance, security, and architecture issues that generic linting and tests may miss.

Rules:
- React Doctor runs in CI on every pull request that touches React/frontend code.
- Error-level diagnostics block merge.
- Warning-level diagnostics must be reviewed and may be tracked as improvement backlog, but they do not block early product development unless explicitly promoted later.
- Release candidates must pass a full React Doctor scan.
- Use Bun-first commands; do not use `npx`, `npm`, `yarn`, or `pnpm`.

Recommended PR/CI command:

```bash
bunx --bun react-doctor@latest . --yes --fail-on error --offline --annotations
```

Recommended release-candidate command:

```bash
bunx --bun react-doctor@latest . --yes --full --fail-on error --offline
```

Recommended changed-files command when a repository supports base-branch diffing:

```bash
bunx --bun react-doctor@latest . --diff main --fail-on error --offline --annotations
```

### 15.5 React Compiler CI

React Compiler is mandatory for dashboard production builds. Compiler diagnostics are treated as correctness and performance signals, not optional noise.

Rules:
- Production build must run with React Compiler enabled.
- CI must fail if the configured build fails because of React Compiler critical diagnostics.
- Do not silence compiler diagnostics without an ADR explaining the unsafe pattern and remediation plan.
- Avoid adding `useMemo`, `useCallback`, or `React.memo` as a reflexive fix for render warnings. Prefer pure components and compiler-friendly code first.

### 15.6 Bundle, Source Map, And Static Artifact Checks

Performance and static artifact checks are required before production launch.

Rules:
- CI should fail or warn when the authenticated dashboard shell exceeds the project budget.
- Bundle analysis must identify top dependency contributors when bundle size changes materially.
- Static production artifacts must not include public `.map` files unless a security exception explicitly allows public source maps.
- If source maps are needed for monitoring, CI uploads them privately to the monitoring provider and removes them from served assets.
- CI must verify the Docker image does not contain `.env`, `.git`, test reports, coverage, Playwright artifacts, local caches, or dependency install caches.

---

## 16. CI/CD

**GitHub Actions** is the mandatory CI/CD platform.

### 16.1 Pull Request pipeline (runs on every PR)

1. `bun install` — install dependencies
2. Oxfmt — formatting check (`oxfmt --check`)
3. Oxlint — lint check
4. TypeScript — type check (`tsc --noEmit`)
5. React Compiler production build diagnostics
6. React Doctor — React frontend health scan
7. `bun audit` — dependency vulnerability scan
8. TanStack supply-chain guard — fail on affected `@tanstack/*` versions and advisory indicators (§12.8.1)
9. Vitest — unit and integration tests
10. Bundle budget or bundle analysis check for frontend changes
11. Docker build — verify the image builds without error
12. Static artifact scan — fail on public source maps, `.env`, `.git`, test reports, coverage, Playwright artifacts, and local caches in the runtime image

When the repository includes a standalone backend API, CI also runs the backend checks from the backend stack: OpenAPI generation/drift, JWT/JWKS auth tests, RBAC/scope tests, tenant isolation tests, rate limit tests, idempotency tests, CORS/cookie/CSRF tests where applicable, webhook tests, worker/queue retry tests, and API/worker Docker smoke tests.

### 16.2 Merge to `main` pipeline

1. All PR checks (above)
2. Playwright E2E tests
3. Docker image build and push to registry
4. Deploy to staging environment

### 16.3 Rules

- All secrets used in Actions are stored as **GitHub Actions Secrets**, never in workflow YAML.
- Production deployments require a passing staging deployment as a prerequisite.
- Do not use `--force` pushes to `main` or `production` branches.
- During active supply-chain incidents, CI may temporarily use `bun install --ignore-scripts`; document the incident link and restore normal installs after provenance review.

---

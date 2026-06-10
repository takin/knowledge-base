# CI/CD: TypeScript

Source URL: Internal draft
Collected: 2026-04-15
Published: 2026-04-15
Updated: 2026-06-10
Status: Draft
Scope: CI/CD pipelines for TypeScript projects

---

## 16. CI/CD

**GitHub Actions** is the mandatory CI/CD platform.

### 16.1 Pull Request Pipeline (runs on every PR)

1. `bun install` — install dependencies
2. Oxfmt — formatting check (`oxfmt --check`)
3. Oxlint — lint check
4. TypeScript — type check (`tsc --noEmit`)
5. React Compiler production build diagnostics (when repository includes React)
6. React Doctor — React frontend health scan (when repository includes React)
7. `bun audit` — dependency vulnerability scan
8. TanStack supply-chain guard — fail on affected `@tanstack/*` versions and advisory indicators (§12.8.1)
9. Vitest — unit and integration tests
10. Bundle budget or bundle analysis check for frontend changes
11. Docker build — verify the image builds without error
12. Docker image size and base-image review — fail when the runtime image exceeds `200 MB` without an ADR, uses `latest`, or uses a large non-minimal base image without ADR
13. Final JS dependency review — fail when final runtime `node_modules` contains dev dependencies, development-only packages, install caches, or packages copied from a build/dev stage
14. Static artifact scan — fail on public source maps, `.env`, `.git`, source directories, test reports, coverage, Playwright artifacts, temporary files, and local caches in the runtime image

When the repository includes a standalone backend API, CI also runs the backend checks from the backend stack: OpenAPI generation/drift, JWT/JWKS auth tests, RBAC/scope tests, tenant isolation tests, rate limit tests, idempotency tests, CORS/cookie/CSRF tests where applicable, webhook tests, worker/queue retry tests, and API/worker Docker smoke tests.

### 16.2 Merge to `main` Pipeline

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

## 15.6 Bundle, Source Map, And Static Artifact Checks

Performance and static artifact checks are required before production launch.

Rules:
- CI should fail or warn when the authenticated dashboard shell exceeds the project budget.
- Bundle analysis must identify top dependency contributors when bundle size changes materially.
- Static production artifacts must not include public `.map` files unless a security exception explicitly allows public source maps.
- If source maps are needed for monitoring, CI uploads them privately to the monitoring provider and removes them from served assets.
- CI must verify the Docker image does not contain `.env`, `.git`, test reports, coverage, Playwright artifacts, local caches, or dependency install caches.
- CI must verify final Docker images use pinned Alpine, slim, distroless, or the smallest production-suitable image variants unless an ADR documents why a larger base image is required.
- CI must fail when final runtime images exceed the `200 MB` target unless an ADR-approved exception documents the measured size and why a larger image is required.
- CI must inspect final JS/TS runtime images to verify development dependencies are absent from final `node_modules`.
- CI must verify final JS/TS runtime images do not contain development `node_modules`, test runners, linters, formatters, TypeScript compilers, Playwright browser bundles, local test utilities, codegen-only packages, or package manager caches unless an ADR documents a runtime need.
- CI must verify final JS/TS runtime `node_modules` came from a production-only install stage, not from a build/dev dependency stage.
- CI must verify final runtime images do not contain source directories, build caches, install caches, temporary files, `.env*`, `.git`, test reports, coverage, Playwright artifacts, or public source maps unless explicitly approved.
- Static dashboard runtime images must contain zero `node_modules`.

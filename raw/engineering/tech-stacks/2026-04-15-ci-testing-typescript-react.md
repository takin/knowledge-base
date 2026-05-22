# CI and Testing Stack: TypeScript + React

Source URL: Internal draft
Collected: 2026-04-15
Published: 2026-04-15
Status: Draft
Scope: TypeScript, React, CI, testing, linting, type checking, and release quality gates

---

## 15. Code Quality

### 15.1 Linting: ESLint

- ESLint with TypeScript support (`@typescript-eslint`) is mandatory.
- Extend from the project's shared ESLint config. Do not configure per-file exceptions without a comment explaining why.
- Lint runs in CI on every pull request. A failing lint check blocks merge.

### 15.2 TypeScript

- **Strict mode** is mandatory (`"strict": true` in `tsconfig.json`).
- No `any` types without an inline `// eslint-disable-next-line` comment and a short explanation.
- Prefer `unknown` over `any` when the type is genuinely unknown.
- All server function inputs and outputs are typed via Zod inference (`z.infer<typeof schema>`).

### 15.3 Testing

| Layer | Tool | Scope |
|---|---|---|
| Unit / integration | **Vitest** | Pure functions, Zod schemas, server function logic, Zustand stores |
| Component | **Vitest + Testing Library** | Shadcn/custom component rendering and interaction |
| End-to-end | **Playwright** | Critical user flows (auth, onboarding, core feature happy path) |
| React health scan | **React Doctor** | React correctness, performance, security, and architecture diagnostics |

Rules:
- Every server function must have at least one integration test hitting a real (test) database — no mocks of database calls.
- E2E tests cover: login/logout, the single most important user action per product, and any flow that handles money or data loss.
- Test files live in `__tests__/` adjacent to the code they test, or in a top-level `tests/` folder for E2E.

### 15.4 React Doctor

**React Doctor** is mandatory for all React frontend and fullstack applications.

React Doctor complements ESLint, TypeScript, Vitest, Testing Library, and Playwright with React-specific diagnostics. It is used to catch correctness, performance, security, and architecture issues that generic linting and tests may miss.

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

---

## 16. CI/CD

**GitHub Actions** is the mandatory CI/CD platform.

### 16.1 Pull Request pipeline (runs on every PR)

1. `bun install` — install dependencies
2. ESLint — lint check
3. TypeScript — type check (`tsc --noEmit`)
4. React Doctor — React frontend health scan
5. `bun audit` — dependency vulnerability scan
6. TanStack supply-chain guard — fail on affected `@tanstack/*` versions and advisory indicators (§12.8.1)
7. Vitest — unit and integration tests
8. Docker build — verify the image builds without error

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

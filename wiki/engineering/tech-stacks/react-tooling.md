# React Tooling: Compiler, Linting, Formatting, and Doctor

Updated: 2026-06-10
Status: Draft
Sources: Internal Tech Stacks draft (2026-04-15, updated 2026-06-10)
Platform: Delivery / quality
Runtime: Bun
Framework: React 19 + TypeScript
Primary Use Case: React Compiler diagnostics, React Doctor health scans, React-specific linting/formatting, and CI quality gates
Raw: [2026-04-15-react-tooling.md](../../../raw/engineering/tech-stacks/2026-04-15-react-tooling.md)

## Summary

This article defines React-specific tooling standards for dashboard and fullstack applications. React Compiler is mandatory for production builds. React Doctor runs in CI to catch correctness, performance, security, and architecture issues. Oxlint and Oxfmt remain the default linter and formatter.

## React Compiler

React Compiler is mandatory for dashboard production builds. Compiler diagnostics are treated as correctness and performance signals, not optional noise.

Rules:
- Production build must run with React Compiler enabled.
- CI must fail if the configured build fails because of React Compiler critical diagnostics.
- Do not silence compiler diagnostics without an ADR explaining the unsafe pattern and remediation plan.
- Avoid adding `useMemo`, `useCallback`, or `React.memo` as a reflexive fix for render warnings. Prefer pure components and compiler-friendly code first.

## React Doctor

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

## React Linting and Formatting

- Oxlint is the default linter for React projects; it covers React-specific rules.
- Oxfmt is the default formatter for React projects.
- ESLint is added only as a targeted exception when Oxlint cannot cover a concrete React-specific risk.
- Lint and format checks run in CI on every pull request and block merge on failure.

## See Also

- [TypeScript Linting and Formatting](typescript-linting-formatting.md)
- [CI/CD: TypeScript](ci-cd-typescript.md)
- [Web: React + Vite](web-react-vite.md)
- [Web: TanStack Start](web-tanstack-start.md)

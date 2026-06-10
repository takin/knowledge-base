# TypeScript Linting and Formatting

Updated: 2026-06-10
Status: Draft
Sources: Internal Tech Stacks draft (2026-04-15, updated 2026-06-10)
Platform: Delivery / quality
Runtime: Bun
Framework: TypeScript
Primary Use Case: Oxc linting/formatting, strict TypeScript type checking, and code quality gates
Raw: [2026-04-15-typescript-linting-formatting.md](../../../raw/engineering/tech-stacks/2026-04-15-typescript-linting-formatting.md)

## Summary

This article defines the linting, formatting, and type-checking standards for TypeScript projects. Oxlint and Oxfmt are the default linter and formatter. Strict TypeScript mode is mandatory. These checks run in CI on every pull request and block merge on failure.

## Linting And Formatting: Oxc Toolchain

- **Oxlint** is the default linter for new TypeScript + React dashboard projects.
- **Oxfmt** is the default formatter for new TypeScript + React dashboard projects.
- ESLint is not the default linter for new dashboard projects. Add it only as a targeted exception when Oxlint cannot cover a concrete risk.
- Prettier and Biome are not the default formatter for new dashboard projects.
- Lint and format checks run in CI on every pull request. A failing lint or format check blocks merge.
- Oxfmt formatting must be checked with `oxfmt --check`.
- Oxlint does not replace TypeScript typechecking; `tsc --noEmit` remains mandatory.

## TypeScript

- **Strict mode** is mandatory (`"strict": true` in `tsconfig.json`).
- No `any` types without an inline lint suppression comment and a short explanation.
- Prefer `unknown` over `any` when the type is genuinely unknown.
- All backend API boundaries are typed through route schemas and generated OpenAPI clients. Dashboard-only code must not define server functions.

## See Also

- [React Tooling](react-tooling.md)
- [CI/CD: TypeScript](ci-cd-typescript.md)
- [Web: React + Vite](web-react-vite.md)
- [Web: TanStack Start](web-tanstack-start.md)

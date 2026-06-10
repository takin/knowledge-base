# TypeScript Linting and Formatting

Source URL: Internal draft
Collected: 2026-04-15
Published: 2026-04-15
Updated: 2026-06-10
Status: Draft
Scope: TypeScript linting, formatting, and type checking

---

## 15.1 Linting And Formatting: Oxc Toolchain

- **Oxlint** is the default linter for new TypeScript and Javascript projects.
- **Oxfmt** is the default formatter for new TypeScript and Javascript projects.
- ESLint is not the default linter for new dashboard projects. Add it only as a targeted exception when Oxlint cannot cover a concrete risk.
- Prettier and Biome are not the default formatter for any Typescript and Javascript projects.
- Lint and format checks run in CI on every pull request. A failing lint or format check blocks merge.
- Oxfmt formatting must be checked with `oxfmt --check`.
- Oxlint does not replace TypeScript typechecking; `tsc --noEmit` remains mandatory.

## 15.2 TypeScript

- **Strict mode** is mandatory (`"strict": true` in `tsconfig.json`).
- No `any` types - non negotiable. Create new type if there is no known types library.
- Prefer `unknown` over `any` when the type is genuinely unknown.
- All backend API boundaries are typed through route schemas and generated OpenAPI clients. Dashboard-only code must not define server functions.

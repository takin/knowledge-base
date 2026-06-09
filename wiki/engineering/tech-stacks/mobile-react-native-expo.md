# Mobile Stack: React Native + Expo

Updated: 2026-06-09
Status: Draft
Sources: Internal Tech Stacks draft (2026-04-15, updated 2026-05-23); Internal standalone repository structure draft (2026-06-09)
Platform: Mobile
Runtime: Expo / JavaScript
Framework: React Native + Expo Router
Primary Use Case: Standalone mobile applications consuming the backend API through generated OpenAPI clients, with independent CI, EAS builds, and app store release flow
Raw: [2026-04-15-mobile-react-native-expo-stack.md](../../../raw/engineering/tech-stacks/2026-04-15-mobile-react-native-expo-stack.md); [2026-06-09-repository-structure-standalone.md](../../../raw/engineering/tech-stacks/2026-06-09-repository-structure-standalone.md)

## Summary

This stack maps web product standards to React Native with Expo Router, NativeWind, TanStack Query/Form, Zustand, JWT access tokens with refresh rotation, an OpenAPI-generated API client, secure native token storage, Sentry, and Maestro. Mobile repositories are standalone by default under `<product>-mobile` because mobile release and adoption cycles are independent from dashboard, landing, and API deploys.

## Standard Mappings

| Web standard | Mobile equivalent |
| --- | --- |
| Web routing | Expo Router |
| TanStack Query | TanStack Query |
| TanStack Form + Zod | TanStack Form + Zod |
| Zustand | Zustand |
| Tailwind CSS v4 | NativeWind v4 |
| Shadcn UI | Shadcn/RN community port or shared internal preset |
| Backend auth | JWT access tokens, refresh rotation, OpenAPI-generated API client, secure native token storage |
| Sentry | `@sentry/react-native` with runtime toggle |
| Vitest + Playwright | Vitest + Maestro |
| Bun runtime | Not applicable; Expo/Metro handles the mobile bundle |

## Rules

- Mobile apps live in standalone `<product>-mobile` repositories by default.
- `apps/mobile/` is allowed only inside a product monorepo approved by Architecture Decision Record.
- Dark/light mode is mandatory using Expo color scheme integration plus NativeWind `dark:` variants.
- The React `"use client"` directive rule does not apply to React Native.
- Do not use `useEffect` for data fetching; use TanStack Query.
- Use secure native token storage for refresh/access token handling.
- The OpenAPI-generated mobile API client must be regenerated or checked in CI whenever the backend OpenAPI contract changes.
- Mobile client generation uses the API repository's versioned OpenAPI release artifact or an approved authenticated docs-enabled environment, not unauthenticated production `/docs/json`.
- Design tokens may be duplicated early and moved into a shared token package only after repeated cross-repository need.
- NativeWind v4 requires Babel preset configuration.
- Do not use `StyleSheet.create()` for normal product UI; use NativeWind utility classes. `StyleSheet` is allowed for performance-critical animations or third-party integration.
- Maestro is the standard mobile E2E runner and flows live in `tests/e2e/`.
- Mobile test files live in the top-level `tests/` hierarchy, not beside implementation files under `src/` or `app/`.
- Unit tests live in `tests/unit/`, integration tests in `tests/integration/`, and Maestro E2E flows in `tests/e2e/`.
- Shared mobile test helpers, fixtures, and factories live under `tests/helpers/`, `tests/fixtures/`, or `tests/factories/`.
- Do not use adjacent `__tests__/` directories or colocated `*.test.*` / `*.spec.*` files inside implementation folders.

## NativeWind Rationale

NativeWind v4 preserves the Tailwind mental model across web and mobile. It lets developers reuse class names, token thinking, and CSS variable conventions. Tamagui and Unistyles can be appropriate for specialized performance or design-system needs, but NativeWind is the pragmatic default for cross-platform teams that want shared product language.

## See Also

- [Backend: Bun + Elysia](backend-bun-elysia.md)
- [React + Vite](web-react-vite.md)
- [CI and Testing: TypeScript + React](ci-testing-typescript-react.md)
- [Repository Structure: Standalone Repos](repository-structure-standalone.md)

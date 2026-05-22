# Mobile Stack: React Native + Expo

Updated: 2026-05-22
Status: Draft
Sources: Internal Tech Stacks draft (2026-04-15, updated 2026-05-22)
Platform: Mobile
Runtime: Expo / JavaScript
Framework: React Native + Expo Router
Primary Use Case: Mobile applications sharing product architecture with the web stack and consuming the backend API through generated OpenAPI clients
Raw: [2026-04-15-mobile-react-native-expo-stack.md](../../../raw/engineering/tech-stacks/2026-04-15-mobile-react-native-expo-stack.md)

## Summary

This stack maps web product standards to React Native with Expo Router, NativeWind, TanStack Query/Form, Zustand, JWT access tokens with refresh rotation, an OpenAPI-generated API client, secure native token storage, Sentry, and Maestro.

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

- Mobile apps live in the product repo under `apps/mobile/` when the product uses a monorepo.
- Dark/light mode is mandatory using Expo color scheme integration plus NativeWind `dark:` variants.
- The React `"use client"` directive rule does not apply to React Native.
- Do not use `useEffect` for data fetching; use TanStack Query.
- Use secure native token storage for refresh/access token handling.
- The OpenAPI-generated mobile API client must be regenerated or checked in CI whenever the backend OpenAPI contract changes.
- Mobile client generation uses a committed/CI-generated OpenAPI artifact or an authenticated docs-enabled environment, not unauthenticated production `/docs/json`.
- Design tokens are shared from the same source as web where possible.
- NativeWind v4 requires Babel preset configuration.
- Do not use `StyleSheet.create()` for normal product UI; use NativeWind utility classes. `StyleSheet` is allowed for performance-critical animations or third-party integration.
- Maestro is the standard mobile E2E runner and flows live in `tests/e2e/`.

## NativeWind Rationale

NativeWind v4 preserves the Tailwind mental model across web and mobile. It lets developers reuse class names, token thinking, and CSS variable conventions. Tamagui and Unistyles can be appropriate for specialized performance or design-system needs, but NativeWind is the pragmatic default for cross-platform teams that want shared product language.

## See Also

- [Backend: Bun + Elysia](backend-bun-elysia.md)
- [React + Vite Dashboard](web-react-vite-dashboard.md)
- [CI and Testing: TypeScript + React](ci-testing-typescript-react.md)

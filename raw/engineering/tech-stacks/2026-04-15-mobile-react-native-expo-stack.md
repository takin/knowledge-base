# Mobile Stack: React Native + Expo

Source URL: Internal draft
Collected: 2026-04-15
Published: 2026-04-15
Status: Draft
Scope: React Native mobile applications using Expo Router and NativeWind

---

## 17. Mobile: React Native (Expo)

The same principles and standards that govern web products apply to mobile products built with **React Native (Expo)**.

Specific mappings:

| Web standard | Mobile equivalent |
|---|---|
| Web routing | Expo Router (file-based routing) |
| TanStack Query | TanStack Query (same library, React Native compatible) |
| TanStack Form + Zod | TanStack Form + Zod (same) |
| Zustand | Zustand (same) |
| Tailwind CSS v4 | **NativeWind v4** (Tailwind for React Native) |
| Shadcn UI | **Shadcn/RN** — community port of Shadcn components for React Native, built on NativeWind v4 and Radix primitives adapted for mobile. Pull from the shared internal Shadcn preset repo as the baseline. |
| Backend auth | JWT access tokens, refresh rotation, OpenAPI-generated API client, and secure native token storage |
| Sentry | `@sentry/react-native` (same optional/runtime-toggle pattern) |
| Vitest + Playwright | Vitest + Maestro (E2E for mobile) |
| BunJS runtime | Not applicable — Expo/Metro handles the mobile bundle |

**Why NativeWind v4 over alternatives (Tamagui, Unistyles):**
NativeWind v4 uses the same Tailwind CSS class names and CSS variable token system as the web stack. A developer who knows the web product can work on mobile without learning a new styling paradigm. Tamagui is more performant at compile time but introduces its own design system and component primitives that would diverge from the web tokens and the shared Shadcn preset. Unistyles v3 is excellent for performance-critical cases but lacks the Tailwind mental model that unifies the team. NativeWind is the pragmatic choice for a cross-platform team building products with a shared design language.

Rules:
- Mobile apps live in the same product repo under `apps/mobile/` (see §18.2).
- Dark/Light mode is mandatory on mobile (use Expo's `useColorScheme` + NativeWind's `dark:` variant).
- The `"use client"` directive rule does not apply to React Native.
- No `useEffect` for data fetching on mobile — same policy as web.
- The OpenAPI-generated mobile API client must be regenerated or checked in CI whenever the backend OpenAPI contract changes.
- Mobile client generation uses a committed/CI-generated OpenAPI artifact or an authenticated docs-enabled environment, not unauthenticated production `/docs/json`.
- Design tokens (colors, spacing, typography) are shared from the same source as the web product. In a monorepo, the `packages/ui/` shared package exports the token definitions used by both web (Tailwind CSS variables) and mobile (NativeWind CSS variables).
- NativeWind v4 requires the Babel preset — ensure `babel.config.js` is configured per the NativeWind v4 setup guide.
- Do not use React Native's `StyleSheet.create()` for product UI — use NativeWind utility classes. `StyleSheet` is permitted only for performance-critical animations or third-party library integration.
- **Maestro** is the standard mobile E2E test runner. It uses a declarative YAML flow DSL (`- tapOn:`, `- assertVisible:`) that AI coding agents generate accurately and consistently. Detox requires native build coupling and imperative JavaScript that is harder for agents to produce correctly. Maestro flows live in `tests/e2e/` alongside Playwright web tests.

---

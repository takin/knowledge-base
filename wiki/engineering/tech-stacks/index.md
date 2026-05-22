# Tech Stacks Index

Updated: 2026-05-22
Status: Draft
Sources: Internal Tech Stacks draft (2026-04-15)
Raw: [2026-04-15-tech-stacks-overview.md](../../../raw/engineering/tech-stacks/2026-04-15-tech-stacks-overview.md)

Generic, reusable engineering standards for application stacks. Stack names encode platform plus runtime or framework identity so current standards can coexist with future alternatives.

| Stack | Platform | Runtime | Framework / Core Tools | Status | Article |
| --- | --- | --- | --- | --- | --- |
| Web: React + TanStack Start | Web / frontend-owned or fullstack | Bun | React + TanStack Start | Current draft standard | [web-react-tanstack-start.md](web-react-tanstack-start.md) |
| Backend: Bun + Elysia | Backend API | Bun | Elysia | Current draft standard | [backend-bun-elysia.md](backend-bun-elysia.md) |
| Mobile: React Native + Expo | Mobile | Expo / JavaScript | React Native + Expo Router | Current draft standard | [mobile-react-native-expo.md](mobile-react-native-expo.md) |
| Infrastructure: Docker Compose + Nginx | Infrastructure | Docker | Docker Compose + Nginx | Current draft standard | [infra-docker-compose-nginx.md](infra-docker-compose-nginx.md) |
| Security Baseline: Web Applications | Cross-stack security | N/A | CSP, CSRF, XSS, rate limiting, dependency security | Current draft standard | [security-web-app-baseline.md](security-web-app-baseline.md) |
| CI and Testing: TypeScript + React | Delivery / quality | Bun | ESLint, TypeScript, Vitest, Playwright, React Doctor | Current draft standard | [ci-testing-typescript-react.md](ci-testing-typescript-react.md) |
| Agent Skills: TypeScript Product Development | AI implementation workflow | N/A | Required coding-agent skills by stack | Current draft standard | [agent-skills-typescript-product.md](agent-skills-typescript-product.md) |

## Future Stack Naming Examples

| Stack Family | Example Names |
| --- | --- |
| Web | `web-svelte-sveltekit`, `web-solid-solidstart`, `web-react-nextjs` |
| Backend API | `backend-go-chi`, `backend-go-fiber`, `backend-rust-axum` |
| Mobile | `mobile-flutter` |
| Infrastructure | `infra-kubernetes-nginx`, `infra-flyio` |

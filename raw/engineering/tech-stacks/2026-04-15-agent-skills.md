# Agent Skills: TypeScript Product Development

Source URL: Internal draft
Collected: 2026-04-15
Published: 2026-04-15
Status: Draft
Scope: Required AI coding-agent skills for implementing products that adopt these stacks

---

## 23. Required Agent Skills

When an AI agent implements a product adopting this standard, it **must** load and apply the following skills before writing any code. Skills are scoped per stack. Loading a skill gives the agent the detailed standards for that domain — skipping one risks inconsistent or non-compliant output.

### 23.1 All TypeScript projects (every stack)

| Skill | Scope | Install Command |
|-------|-------|-----------------|
| `typescript-best-practice` | All TypeScript files across frontend, fullstack, backend, and mobile | `npx skills add https://github.com/0xbigboss/claude-code --skill typescript-best-practices` |

### 23.2 Frontend & Fullstack (TanStack Start)

| Skill | Scope | Install Command |
|-------|-------|-----------------|
| `better-auth-best-practice` | Auth configuration and session handling | `npx skills add https://github.com/better-auth/skills --skill better-auth-best-practices` |
| `drizzle-orm` | Database schema, migrations, and queries | `npx skills add https://github.com/bobmatnyc/claude-mpm-skills --skill drizzle-orm` |
| `shadcn` | UI component usage and customization | `npx skills add https://github.com/shadcn/ui --skill shadcn` |
| `shadcn ui` | UI component usage and customization (Stitch Skills) | `npx skills add https://github.com/google-labs-code/stitch-skills --skill shadcn-ui` |
| `react-state-management` | State layering (TanStack Query / Form / Zustand / useState) | `npx skills add https://github.com/wshobson/agents --skill react-state-management` |
| `tanstack-start-best-practice` | Server functions, routing, and SSR patterns | `npx skills add https://github.com/deckardger/tanstack-agent-skills --skill tanstack-start-best-practices` |
| `tanstack-start-project-init` | New project bootstrapping via TanStack CLI | `cd ~/.agents/skills && git clone https://github.com/takin/tanstack-start-init-project .` |

**Exception — Landing page projects:** `shadcn` is optional. Before loading the `shadcn` skill, the agent must ask the user: _"Will this landing page use Shadcn UI, or will you build with custom components / a different component library?"_ Load `shadcn` only if the user confirms yes.

### 23.3 Mobile (React Native / Expo)

| Skill | Scope | Install Command |
|-------|-------|-----------------|
| `better-auth-best-practice` | Auth client configuration and session handling | `npx skills add https://github.com/better-auth/skills --skill better-auth-best-practices` |
| `react-state-management` | State layering (TanStack Query / Zustand / useState) on mobile | `npx skills add https://github.com/wshobson/agents --skill react-state-management` |

### 23.4 Backend API (Elysia + Bun)

| Skill | Scope | Install Command |
|-------|-------|-----------------|
| `elysiajs` | API routes, Eden Treaty type export, middleware, and plugin patterns | `npx skills add https://github.com/elysiajs/skills --skill elysiajs` |
| `drizzle-orm` | Database schema, migrations, and queries | `npx skills add https://github.com/bobmatnyc/claude-mpm-skills --skill drizzle-orm` |
| `postgresql-table-design` | Table design, RLS policies, and index strategy | `npx skills add https://github.com/wshobson/agents --skill postgresql-table-design` |

---

*This source bundle is a draft. Upon approval, it is ingested into `wiki/engineering/tech-stacks/` as reusable stack standards.*

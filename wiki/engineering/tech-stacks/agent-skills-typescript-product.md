# Agent Skills: TypeScript Product Development

Updated: 2026-05-22
Status: Draft
Sources: Internal Tech Stacks draft (2026-04-15)
Platform: AI implementation workflow
Runtime: N/A
Framework: N/A
Primary Use Case: Required coding-agent skill loading rules for products adopting these stacks
Raw: [2026-04-15-agent-skills.md](../../../raw/engineering/tech-stacks/2026-04-15-agent-skills.md)

## Summary

This article lists required AI coding-agent skills by stack so generated code follows the same standards for TypeScript, Shadcn, Elysia, PostgreSQL, and mobile work.

## Standard

## 23. Required Agent Skills

When an AI agent implements a product adopting this standard, it **must** load and apply the following skills before writing any code. Skills are scoped per stack. Loading a skill gives the agent the detailed standards for that domain — skipping one risks inconsistent or non-compliant output.

### 23.1 All TypeScript projects (every stack)

| Skill | Scope | Install Command |
|-------|-------|-----------------|
| `typescript-best-practice` | All TypeScript files across frontend, fullstack, backend, and mobile | `npx skills add https://github.com/0xbigboss/claude-code --skill typescript-best-practices` |

### 23.2 Frontend Web Projects

| Skill | Scope | Install Command |
|-------|-------|-----------------|
| `shadcn` | UI component usage and customization | `npx skills add https://github.com/shadcn/ui --skill shadcn` |
| `shadcn ui` | UI component usage and customization (Stitch Skills) | `npx skills add https://github.com/google-labs-code/stitch-skills --skill shadcn-ui` |
| `react-state-management` | State layering (TanStack Query / Form / Zustand / useState) | `npx skills add https://github.com/wshobson/agents --skill react-state-management` |

**Exception — Landing page projects:** `shadcn` is optional. Before loading the `shadcn` skill, the agent must ask the user: _"Will this landing page use Shadcn UI, or will you build with custom components / a different component library?"_ Load `shadcn` only if the user confirms yes.

### 23.3 Mobile (React Native / Expo)

| Skill | Scope | Install Command |
|-------|-------|-----------------|
| `react-state-management` | State layering (TanStack Query / Zustand / useState) on mobile | `npx skills add https://github.com/wshobson/agents --skill react-state-management` |

### 23.4 Backend API (Elysia + Bun)

| Skill | Scope | Install Command |
|-------|-------|-----------------|
| `elysiajs` | API routes, OpenAPI metadata, middleware, guards, and plugin patterns | `npx skills add https://github.com/elysiajs/skills --skill elysiajs` |
| `drizzle-orm` | Database schema, migrations, and queries | `npx skills add https://github.com/bobmatnyc/claude-mpm-skills --skill drizzle-orm` |
| `postgresql-table-design` | Table design, RLS policies, and index strategy | `npx skills add https://github.com/wshobson/agents --skill postgresql-table-design` |

---

*This source bundle is a draft. Upon approval, it is ingested into `wiki/engineering/tech-stacks/` as reusable stack standards.*

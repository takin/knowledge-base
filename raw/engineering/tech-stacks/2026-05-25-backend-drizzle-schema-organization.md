# Backend Drizzle Schema Organization

Source URL: Internal decision
Collected: 2026-05-25
Published: 2026-05-25
Status: Draft
Scope: Drizzle schema layout standard for production SaaS backend API projects

---

## Decision

Production SaaS APIs and backend API projects with multiple business domains split Drizzle schema definitions by business domain under `src/db/schema/`.

Small prototypes with only a few tables may use a single `src/db/schema.ts`, but this is not the production SaaS default.

## Rules

- Use `src/db/schema/index.ts` as the public schema entrypoint and re-export each domain file from it.
- Configure Drizzle Kit with an explicit glob such as `schema: "./src/db/schema/*.ts"` or an explicit array of domain schema files.
- Initialize Drizzle with the merged schema object from the schema entrypoint using `drizzle(sql, { schema })`.
- Split by business domain, not by object type.
- Prefer `auth.ts`, `workspaces.ts`, `billing.ts`, `media.ts`, `webhooks.ts`, and `audit.ts` over folders such as `tables/`, `relations/`, and `indexes/`.
- Keep each domain's tables, relations, indexes, foreign keys, and constraints close together unless the file becomes genuinely large.
- Tables that are tightly coupled should stay in the same domain file.
- Cross-domain foreign keys are allowed, but the ownership boundary should remain clear.
- Shared enums, reusable column helpers, and common timestamp/tenant columns may live in `common.ts` or `enums.ts`.

## Standard Schema Layout

```text
src/
  db/
    schema/
      index.ts
      common.ts
      auth.ts
      workspaces.ts
      billing.ts
      media.ts
      webhooks.ts
      audit.ts
    migrations/
  lib/
    db.ts
drizzle.config.ts
```

## Domain Ownership Defaults

- `auth.ts` owns users, sessions, refresh tokens, API keys, roles, and permissions.
- `workspaces.ts` owns tenants/workspaces, memberships, invitations, and workspace settings.
- `billing.ts` owns subscriptions, plans, entitlements, invoices, and payment provider references.
- `media.ts` owns files, upload intents, and object storage metadata.
- `webhooks.ts` owns webhook endpoints, received events, provider sync state, and outbox messages.
- `audit.ts` owns audit logs, operator actions, and security events.
- `common.ts` owns shared enums, timestamp column helpers, tenant column helpers, and reusable ID helpers.

## Schema Entrypoint

```ts
export * from "./auth"
export * from "./workspaces"
export * from "./billing"
export * from "./media"
export * from "./webhooks"
export * from "./audit"
```

## Drizzle Kit Config Shape

```ts
import { defineConfig } from "drizzle-kit"

export default defineConfig({
  dialect: "postgresql",
  schema: "./src/db/schema/*.ts",
  out: "./src/db/migrations",
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
})
```

## Database Client Shape

```ts
import { SQL } from "bun"
import { drizzle } from "drizzle-orm/bun-sql"
import * as schema from "../db/schema"

const sql = new SQL({
  url: process.env.DATABASE_URL,
  max: Number(process.env.DB_POOL_MAX ?? 20),
  idleTimeout: 30,
  connectionTimeout: 10,
})

export const db = drizzle(sql, { schema })
```

## Schema Definition Examples

```ts
import { sql } from "drizzle-orm"
import { bigint, index, pgTable, text, uniqueIndex, uuid } from "drizzle-orm/pg-core"

export const workspaces = pgTable(
  "workspaces",
  {
    id: uuid("id").default(sql`uuidv7()`).primaryKey(),
    slug: text("slug").notNull(),
    name: text("name").notNull(),
  },
  (table) => [
    uniqueIndex("workspaces_slug_unique").on(table.slug),
  ],
)

export const auditLogs = pgTable(
  "audit_logs",
  {
    id: bigint("id", { mode: "number" }).primaryKey().generatedAlwaysAsIdentity(),
    workspaceId: uuid("workspace_id").notNull().references(() => workspaces.id),
    action: text("action").notNull(),
  },
  (table) => [
    index("audit_logs_workspace_id_idx").on(table.workspaceId),
  ],
)
```

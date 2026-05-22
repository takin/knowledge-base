# Backend Stack: Bun + Elysia

Source URL: Internal draft
Collected: 2026-04-15
Published: 2026-04-15
Status: Draft
Scope: Standalone Bun + Elysia backend APIs, webhooks, and async services

---

## 20. Backend API Stack

### 20.1 Framework: Elysia

**Elysia** is the mandatory HTTP framework for standalone backend API servers.

Use Elysia-first architecture when one or more of these are true:
- the product depends on long-lived SSE connections as a core interaction model
- the product coordinates external connectors/sessions that maintain in-memory runtime state
- the product is webhook-heavy and async-job-heavy enough that a dedicated backend is operationally cleaner than server functions
- the product must expose a separately deployable backend surface to mobile apps or third-party systems

Elysia is built specifically for Bun. It is the fastest TypeScript HTTP framework in the Bun ecosystem and ships **Eden Treaty** — a type-safe RPC client generated directly from the Elysia server type definition. Mobile and web clients import the Eden Treaty client and get full end-to-end TypeScript types without a separate schema, codegen step, or GraphQL overhead.

```typescript
// server — src/index.ts
const app = new Elysia()
  .get('/users/:id', ({ params }) => getUserById(params.id))
  .post('/orders', ({ body }) => createOrder(body), {
    body: t.Object({ productId: t.String(), qty: t.Number() })
  })
export type App = typeof app

// mobile/web client — no codegen needed
import { treaty } from '@elysiajs/eden'
import type { App } from '@app/api'
const client = treaty<App>(process.env.API_URL)
const { data, error } = await client.users({ id: '123' }).get()
// data and error are fully typed
```

Do not use Express, Hono, Fastify, or Koa.

### 20.2 API design

**Style:** RESTful with an OpenAPI 3.1 spec generated automatically by Elysia's `@elysiajs/openapi` plugin.

> **Note:** `@elysiajs/swagger` is deprecated and no longer maintained. Use `@elysiajs/openapi` instead — it ships Scalar UI by default and has a cleaner API.

**Versioning:** URL-prefix versioning (`/api/v1/`, `/api/v2/`). Never version via headers or query params.

**API documentation setup:**

Install:

```bash
bun add @elysiajs/openapi
```

Mount the plugin at the top of the app, before any routes:

```typescript
import { Elysia } from 'elysia'
import { openapi } from '@elysiajs/openapi'

new Elysia()
  .use(
    openapi({
      path: '/docs',                        // serves Scalar UI at /docs
      documentation: {
        info: {
          title: 'Example API',
          version: '1.0.0',
          description: 'Internal REST API for an example product phase'
        },
        tags: [
          { name: 'Auth',          description: 'Session management' },
          { name: 'Workspace',     description: 'Tenant workspace operations' },
          { name: 'Contact',       description: 'Recipient contact management' },
          { name: 'Blast',         description: 'Blast creation and execution' },
          { name: 'Invoice',       description: 'Invoice lifecycle and PDF' },
          { name: 'Disbursement',  description: 'Payment and settlement tracking' },
          { name: 'Webhook',       description: 'Inbound Xendit and Meta webhooks' },
        ],
        components: {
          securitySchemes: {
            sessionCookie: {
              type: 'apiKey',
              in: 'cookie',
              name: 'better_auth_session',   // match Better Auth session cookie name
              description: 'Better Auth session cookie (HttpOnly)'
            }
          }
        }
      }
    })
  )
```

Docs UI is at `/docs`. Raw OpenAPI JSON spec is at `/docs/json`.

**Tagging routes:**

Assign a tag to a single route via `detail`:

```typescript
.get('/workspaces', listWorkspaces, {
  detail: { tags: ['Workspace'], summary: 'List all workspaces for the authenticated tenant' }
})
```

Apply tags to an entire controller group using instance-level `detail`:

```typescript
export const workspaceController = new Elysia({
  prefix: '/workspaces',
  detail: {
    tags: ['Workspace'],
    security: [{ sessionCookie: [] }]   // marks all routes as requiring session cookie
  }
})
```

**Hiding internal or sensitive routes:**

Webhook endpoints and internal-only routes should be excluded from the public spec:

```typescript
.post('/webhooks/xendit', handleXendit, {
  detail: { hide: true }
})
```

Health check endpoints (`/health`, `/ready`) should also be hidden — they are not consumer API surface.

**Access control in production:**

The `/docs` and `/docs/json` endpoints should be disabled or access-controlled in production. Mount the plugin conditionally:

```typescript
.use(
  openapi({
    enabled: process.env.APP_ENV !== 'production',
    path: '/docs',
    // ... rest of config
  })
)
```

Alternatively, gate the docs path behind an admin-only middleware and leave `enabled: true` in all environments for internal use.

**Env var:**

```env
API_DOCS_ENABLED=true   # false in production (or gate behind auth middleware)
```

**Rules:**
- Use `@elysiajs/openapi`, not the deprecated `@elysiajs/swagger`.
- Mount at `/docs` (not the default `/openapi`) to avoid path collision with OpenAPI spec tooling.
- Every resource group gets its own tag — makes the Scalar UI navigable.
- All protected routes explicitly declare `security: [{ sessionCookie: [] }]` at the controller level, not per-route.
- Webhook handlers always use `detail: { hide: true }` — they are not consumer-facing API.
- Health check endpoints (`/health`, `/ready`) always hidden.
- Disable or auth-gate `/docs` in production.

**Consistent response envelope:**

All API responses use a standard envelope. This makes client error handling predictable across all products.

```typescript
// success
{
  "success": true,
  "data": { ... },
  "meta": {                    // present only on paginated responses
    "page": 1,
    "pageSize": 20,
    "total": 450,
    "totalPages": 23
  }
}

// error
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",   // machine-readable constant, never changes
    "message": "...",             // human-readable, may change
    "details": [                  // field-level errors from Zod, optional
      { "field": "email", "message": "Invalid email address" }
    ]
  }
}
```

Error codes are `SCREAMING_SNAKE_CASE` constants defined in a shared `src/types/errors.ts`. Never return raw Zod error objects or Elysia internals to the client.

**Pagination:** All list endpoints are paginated. Master data uses `?page=&pageSize=` (client controls). Transactional data uses cursor-based pagination (`?cursor=&limit=`) to avoid offset drift on fast-growing tables.

### 20.3 Validation

Elysia's built-in schema validation (based on TypeBox) handles request-level shape validation. **Zod** handles business-logic validation (cross-field rules, database constraint checks, domain rules).

Rule: TypeBox at the route boundary, Zod inside service functions. Never skip either layer.

### 20.4 Logging: Pino

**Pino** is the mandatory logger for all backend services.

- Structured JSON output — every log line is a parseable JSON object.
- Log levels in order: `error`, `warn`, `info`, `debug`. Production runs at `info`; debug mode toggles via `LOG_LEVEL=debug` in `.env`.
- Never log PII (email, phone, payment info). Scrub before logging.
- Request logging uses `pino-http` middleware — logs method, path, status code, response time on every request. Sensitive headers (`Authorization`, `Cookie`) are redacted automatically.

```env
LOG_LEVEL=info        # error | warn | info | debug
LOG_PRETTY=false      # true for local dev (human-readable), false for production (JSON)
```

### 20.5 Database connection: PgBouncer + Drizzle pool

**This is the primary scalability control for 2K+ concurrent API calls.**

**PgBouncer runs in `transaction` mode.** This is the only mode that allows many application connections to share a small number of actual Postgres connections. Session mode and statement mode are banned.

In transaction mode: a Postgres connection is held only for the duration of a single transaction, then returned to the pool. 2,000 concurrent app connections can share ~50–100 real Postgres connections.

Each app instance configures Drizzle with a **bounded connection pool** to PgBouncer:

```typescript
// src/lib/db.ts
import { drizzle } from 'drizzle-orm/bun-sql'
import { SQL } from 'bun'

const sql = new SQL({
  url: process.env.DATABASE_URL,  // points to PgBouncer, not Postgres directly
  max: Number(process.env.DB_POOL_MAX ?? 20),   // connections per instance to PgBouncer
  idleTimeout: 30,
  connectionTimeout: 10,
})
export const db = drizzle(sql)
```

```env
DATABASE_URL=postgresql://user:pass@pgbouncer:5432/dbname
DB_POOL_MAX=20       # tune per instance: (total PgBouncer max_client_conn) / (number of app instances)
```

PgBouncer runs as a service in Docker Compose:

```yaml
pgbouncer:
  image: bitnami/pgbouncer:<pinned-version>
  environment:
    POSTGRESQL_HOST: postgres
    POSTGRESQL_PORT: 5432
    PGBOUNCER_DATABASE: ${POSTGRES_DB}
    PGBOUNCER_POOL_MODE: transaction
    PGBOUNCER_MAX_CLIENT_CONN: ${PGBOUNCER_MAX_CLIENT_CONN:-500}
    PGBOUNCER_DEFAULT_POOL_SIZE: ${PGBOUNCER_DEFAULT_POOL_SIZE:-25}
  depends_on:
    - postgres
```

All PgBouncer limits are runtime-configurable via `.env`.

### 20.6 Redis query cache

Redis is used as a **read-through cache** for data that is read frequently and written infrequently. This reduces Postgres load at high concurrency.

Caching rules:
- Cache only data that is safe to serve slightly stale (configurable TTL per resource).
- Never cache responses that include PII without encrypting the cached value.
- Use structured cache keys: `<product>:<resource>:<id>` (e.g., `example:product:abc123`).
- On write/update/delete, **invalidate** the affected cache key immediately — do not rely solely on TTL expiry.

```typescript
// pattern: cache-aside
async function getProduct(id: string) {
  const cached = await redis.get(`example:product:${id}`)
  if (cached) return JSON.parse(cached)
  const product = await db.query.products.findFirst({ where: eq(products.id, id) })
  await redis.setex(`example:product:${id}`, 300, JSON.stringify(product)) // TTL: 5 min
  return product
}
```

Cache TTL defaults by resource type:

| Resource type | Default TTL | Rationale |
|---|---|---|
| Master data (products, categories) | 5 minutes | Low write frequency |
| User session metadata | 15 minutes | Balance freshness vs DB load |
| Aggregated stats / dashboards | 1 minute | Acceptable staleness for analytics |
| Transactional records | Do not cache | Must always be fresh |

### 20.7 File storage

| Environment | Storage | Protocol |
|---|---|---|
| Local development | **MinIO** (Docker Compose) | S3-compatible API |
| Staging / Production | **Cloudflare R2** | S3-compatible API, zero egress fees |

Use the AWS SDK (`@aws-sdk/client-s3`) pointed at the appropriate endpoint — same code works for both MinIO and R2.

Rules:
- Never serve files directly from the bucket. Always generate **presigned URLs** with a short expiry (15 minutes for downloads, 5 minutes for uploads).
- Validate MIME type and file size server-side before accepting uploads. Do not trust the `Content-Type` header alone.
- Store files outside the web root. The bucket must not be publicly readable.
- File metadata (original name, size, MIME type, owner) is stored in Postgres; the bucket stores only the binary.

```env
STORAGE_ENDPOINT=http://localhost:9000     # MinIO locally, R2 endpoint in prod
STORAGE_BUCKET=minia-uploads
STORAGE_ACCESS_KEY=...
STORAGE_SECRET_KEY=...
STORAGE_PRESIGN_EXPIRY=900                 # seconds (15 min)
```

### 20.8 Email: Resend + React Email

**Resend** is the mandatory transactional email provider. **React Email** is the template engine.

- Define email templates as React components in `src/emails/`.
- Render to HTML server-side using `@react-email/render` before sending.
- All sends go through a BullMQ job queue (never block an HTTP response on email delivery).
- Store a record of every sent email in Postgres (recipient, template name, sent_at, status) for auditability.

```env
RESEND_API_KEY=...
EMAIL_FROM=no-reply@minia.id
EMAIL_ENABLED=true     # set to false in local dev to suppress sends
```

### 20.9 Health checks

Every backend service exposes two health endpoints — required by Docker and load balancers for readiness and liveness probes.

| Endpoint | Purpose | Response |
|---|---|---|
| `GET /health` | **Liveness** — is the process alive? | `200 { "status": "ok" }` always (if the process responds, it's alive) |
| `GET /ready` | **Readiness** — are dependencies connected? | `200` if Postgres + Redis reachable; `503` if either is down |

The `/ready` endpoint checks actual connectivity (a lightweight `SELECT 1` on Postgres, a `PING` on Redis). It does not check business logic.

Both endpoints are excluded from authentication middleware and rate limiting.

### 20.10 Graceful shutdown

Bun receives `SIGTERM` from Docker on container stop. The backend must handle it cleanly to avoid dropping in-flight requests.

```typescript
// src/index.ts
const server = app.listen(3000)

process.on('SIGTERM', async () => {
  server.stop(true)          // stop accepting new connections; wait for in-flight
  await db.$client.end()     // close Drizzle / PgBouncer connections
  await redis.quit()         // close Redis connection
  await bullQueue.close()    // drain BullMQ queue gracefully
  process.exit(0)
})
```

Docker Compose `stop_grace_period` must be set to at least `30s` to give in-flight requests time to complete before Docker sends `SIGKILL`.

---

### 20.15 Backend API vs Server Functions

A standalone backend API is necessary when:

- The **mobile app** needs a typed, versioned API endpoint it can call directly
- A **third-party service** sends webhook callbacks to a fixed endpoint
- A feature needs to be **independently deployable** and scaled (e.g., a heavy processing service)
- **Background job workers** need to run in a separate process or container
- You need **explicit API versioning** (`/api/v1/`, `/api/v2/`) for external consumers

Use **TanStack Start server functions** for:

- Fullstack web product features (the web app is the only consumer)
- Features that share the same deployment unit as the web app
- Internal APIs where you control both client and server versions

The rule: **prefer server functions until you have a specific reason not to.** A separate backend API adds versioning overhead, a separate deployment pipeline, and an Eden Treaty client dependency. Only pay that cost when the use case demands it.

---

### 20.16 Webhook Endpoints

Webhook endpoints receive inbound calls from external services — payment providers, logistics partners, notification platforms. These providers have strict timeout windows (typically 3–10 seconds) and aggressive retry strategies (up to 25 retries over 72 hours for some providers). Any processing that exceeds the timeout causes the provider to mark the delivery as failed and retry, leading to duplicate events.

This section defines two things: the **async processing pattern** (how to handle webhooks without timing out) and the **security model** (how to authenticate and validate every inbound webhook). Both are mandatory and uniform across all third-party integrations.

---

#### 20.16.1 Async Processing Pattern — Mermaid Diagram

```mermaid
flowchart TD
    P[External Provider] -->|POST webhook| WH[Webhook Handler]
    
    WH --> L1{Layer 1<br/>IP Whitelist?}
    L1 -- Fail --> R1[Reject 403]
    L1 -- Pass --> L2{Layer 2<br/>API Key?}
    
    L2 -- Fail --> R2[Reject 403]
    L2 -- Pass --> L3{Layer 3<br/>HMAC Signature?}
    
    L3 -- Fail --> R3[Reject 400]
    L3 -- Pass --> L4{Replay<br/>Check?}
    
    L4 -- Fail --> R4[Reject 400]
    L4 -- Pass --> IC{Idempotency<br/>Check}
    
    IC -- Duplicate --> R5[Return 200<br/>Skip Queue]
    IC -- New --> PERSIST[Persist Raw Event<br/>webhook_events table]
    
    PERSIST --> ENQUEUE[Enqueue BullMQ Job]
    ENQUEUE --> RESPOND[Return 200]
    
    RESPOND --> WK[BullMQ Worker]
    WK --> BL[Business Logic]
    WK --> UPDATE[Update Status<br/>processed / failed]
    
    style R1 fill:#e74c3c,color:#fff
    style R2 fill:#e74c3c,color:#fff
    style R3 fill:#e74c3c,color:#fff
    style R4 fill:#e74c3c,color:#fff
    style R5 fill:#f39c12,color:#fff
    style RESPOND fill:#27ae60,color:#fff
```

---

#### 20.16.2 Async Processing Pattern — Text

**The rule: receive immediately, process asynchronously.** The HTTP handler does only four things synchronously — verify, check, persist, enqueue — then returns `200`. All business logic runs in a BullMQ worker.

```
[External Provider] → POST /api/v1/webhooks/:provider
                           │
                    ┌──────▼────────────────────────────┐
                    │  SYNCHRONOUS — must complete       │
                    │  within provider timeout (~500ms)  │
                    │                                    │
                    │  1. IP whitelist check             │
                    │  2. API Key check (if supported)   │
                    │  3. HMAC signature verification    │
                    │  4. Timestamp replay check         │
                    │  5. Idempotency check              │
                    │  6. Persist raw event (pending)    │
                    │  7. Enqueue BullMQ job             │
                    │  8. Return HTTP 200                │
                    └──────┬────────────────────────────┘
                           │
                    ┌──────▼────────────────────────────┐
                    │  ASYNCHRONOUS — BullMQ worker      │
                    │                                    │
                    │  Parse + validate payload          │
                    │  Execute business logic            │
                    │  Update event status               │
                    │  Retry on failure (exp. backoff)   │
                    └───────────────────────────────────┘
```

**Idempotency.** Store the provider's event ID in Postgres before queueing. If the same event ID arrives again (provider retry), return `200` immediately without re-queueing. This is the primary guard against duplicate processing.

**BullMQ retry strategy for webhook workers:**
- 5 attempts with exponential backoff: 2s → 4s → 8s → 16s → 32s.
- After all retries exhausted, move to the dead-letter queue — never discard silently.
- Update `webhookEvents.status`: `pending` → `processed` on success, `failed` after terminal failure.
- Failed events surface in Bull Board (§13) and trigger a Sentry alert.

---

#### 20.16.3 Webhook Security Model — Mermaid Diagram

```mermaid
flowchart TD
    IN[Inbound Request] --> IPW{IP Whitelist<br/>Layer 1}
    
    IPW -->|Not in CIDR| R1[403<br/>Reject]
    IPW -->|In CIDR| AK{API Key<br/>Layer 2}
    
    AK -->|Missing/Invalid| R2[403<br/>Reject]
    AK -->|Valid| HMAC{HMAC-SHA256<br/>Layer 3}
    
    HMAC -->|Invalid| R3[400<br/>Reject]
    HMAC -->|Valid| TS{Timestamp<br/>Replay Check}
    
    TS -->|Older than 5min| R4[400<br/>Reject]
    TS -->|Valid| PASS[Pass → Idempotency → Queue]
    
    style R1 fill:#e74c3c,color:#fff
    style R2 fill:#e74c3c,color:#fff
    style R3 fill:#e74c3c,color:#fff
    style R4 fill:#e74c3c,color:#fff
    style PASS fill:#27ae60,color:#fff
```

---

#### 20.16.4 Webhook Security Model — Text

Every webhook endpoint in every product adopting this standard must implement the same **three-layer security model**, applied in order from cheapest to most expensive. A request is rejected at the first layer it fails — later layers are not evaluated.

```
Inbound request
      │
      ▼
┌─────────────────────────────────────────────────────────────┐
│  Layer 1 — IP Whitelist                                     │
│  Cheapest. Nginx-level or application-level check.          │
│  Reject with 403 if source IP is not in provider's range.   │
└─────────────────┬───────────────────────────────────────────┘
                  │ pass
                  ▼
┌─────────────────────────────────────────────────────────────┐
│  Layer 2 — API Key Header                                   │
│  Mid-cost. Header present + value matches shared secret.    │
│  Reject with 403 if missing or incorrect.                   │
│  Apply only when the provider supports it.                  │
└─────────────────┬───────────────────────────────────────────┘
                  │ pass
                  ▼
┌─────────────────────────────────────────────────────────────┐
│  Layer 3 — HMAC-SHA256 Signature                            │
│  Most expensive. Cryptographic proof the body is authentic. │
│  Reject with 400 if signature does not match.               │
│  Always present — this is the definitive verification.      │
└─────────────────┬───────────────────────────────────────────┘
                  │ pass
                  ▼
┌─────────────────────────────────────────────────────────────┐
│  Bonus — Timestamp / Replay Protection                      │
│  Reject with 400 if event timestamp is older than 5 min.    │
│  Prevents captured-and-replayed valid requests.             │
└─────────────────┬───────────────────────────────────────────┘
                  │ pass → proceed to idempotency check + queue
```

---

**Layer 1 — IP Whitelist**

Every major provider publishes its webhook sender IP ranges. Check the inbound IP against these ranges before doing anything else. This eliminates random internet traffic at zero cryptographic cost.

- IP is read from the `X-Forwarded-For` header set by Nginx (trust only Nginx's header, not the raw socket IP — the app container is behind the proxy).
- IP ranges are stored in `.env` as comma-separated CIDR blocks per provider.
- If a provider does not publish IP ranges, skip this layer — do not fabricate a fake whitelist.
- IP ranges change occasionally. Monitor provider release notes and update `.env` accordingly. This layer is defense-in-depth, not the primary control.

```env
WEBHOOK_IP_WHITELIST_PAYMENT=103.4.55.0/24,103.4.56.0/24   # provider's published ranges
WEBHOOK_IP_WHITELIST_LOGISTICS=202.80.0.0/20
```

---

**Layer 2 — API Key Header**

A shared static secret sent in a custom request header. Some providers call it `X-Api-Key`, others `X-Webhook-Token` — use whatever the provider specifies. This is a pre-filter before the more expensive HMAC computation.

- Apply only when the provider explicitly supports sending a static API key header.
- Use a randomly generated secret (minimum 32 bytes, hex or base64), not a human-memorable password.
- Compare using timing-safe equality (`crypto.timingSafeEqual`) — never `===`.

```env
WEBHOOK_API_KEY_PAYMENT=<random-32-byte-hex>    # set in Vault for staging/prod
WEBHOOK_API_KEY_HEADER_PAYMENT=X-Payment-Token  # the header name the provider sends
```

---

**Layer 3 — HMAC-SHA256 Signature (mandatory)**

The provider signs the raw request body with a shared secret using HMAC-SHA256 and sends the result in a signature header. The application recomputes the HMAC independently and compares the two values.

Critical implementation rules:
- **Use the raw request body** (bytes before JSON parsing). Some providers compute the HMAC over the raw body including whitespace. Parse JSON only after the signature check passes.
- **Use `crypto.timingSafeEqual`** for comparison — a standard string `===` check leaks timing information that can be used to forge signatures.
- The signature format varies by provider (`sha256=<hex>`, `v1=<hex>`, etc.). Strip the prefix before comparing.

```typescript
// src/lib/webhook-verify.ts

import { createHmac, timingSafeEqual } from 'crypto'

export function verifyHmac(
  rawBody: string,
  receivedSig: string,     // the full header value, e.g. "sha256=abc123"
  secret: string,
  prefix = 'sha256='       // provider-specific prefix to strip
): boolean {
  const clean = receivedSig.startsWith(prefix)
    ? receivedSig.slice(prefix.length)
    : receivedSig

  const expected = createHmac('sha256', secret)
    .update(rawBody, 'utf8')
    .digest('hex')

  // timing-safe comparison — never use ===
  try {
    return timingSafeEqual(Buffer.from(clean), Buffer.from(expected))
  } catch {
    return false  // buffers of different length — definite mismatch
  }
}
```

```env
WEBHOOK_SECRET_PAYMENT=<random-32-byte-hex>        # HMAC signing secret
WEBHOOK_SIG_HEADER_PAYMENT=X-Payment-Signature     # header name carrying the signature
WEBHOOK_SIG_PREFIX_PAYMENT=sha256=                 # prefix to strip before comparing
```

---

**Replay protection**

Many providers embed a timestamp in the payload or signature header. Reject any event with a timestamp older than 5 minutes. This prevents a valid captured request from being replayed hours or days later.

```typescript
const eventTimestamp = body.created_at  // provider-specific field name
const ageMs = Date.now() - new Date(eventTimestamp).getTime()
if (ageMs > 5 * 60 * 1000) {
  return error(400, 'Webhook event too old — possible replay attack')
}
```

If the provider does not include a timestamp, skip this check — do not fabricate one.

---

**Uniform `.env` convention for all webhook integrations**

All three layers follow the same naming pattern across every provider, making it trivially auditable which providers have which layers configured.

```env
# Pattern: WEBHOOK_<LAYER>_<PROVIDER>=<value>

# Layer 1 — IP Whitelist (CIDR ranges, comma-separated)
WEBHOOK_IP_WHITELIST_PAYMENT=103.4.55.0/24,103.4.56.0/24
WEBHOOK_IP_WHITELIST_LOGISTICS=202.80.0.0/20

# Layer 2 — API Key (if provider supports it)
WEBHOOK_API_KEY_PAYMENT=<hex>
WEBHOOK_API_KEY_HEADER_PAYMENT=X-Payment-Token
# WEBHOOK_API_KEY_LOGISTICS — omitted: this provider does not send an API key

# Layer 3 — HMAC Signature (always present)
WEBHOOK_SECRET_PAYMENT=<hex>
WEBHOOK_SIG_HEADER_PAYMENT=X-Payment-Signature
WEBHOOK_SIG_PREFIX_PAYMENT=sha256=

WEBHOOK_SECRET_LOGISTICS=<hex>
WEBHOOK_SIG_HEADER_LOGISTICS=X-Logistics-Hmac
WEBHOOK_SIG_PREFIX_LOGISTICS=

# General
WEBHOOK_TIMEOUT_MS=5000        # abort handler if synchronous steps exceed this
WEBHOOK_MAX_AGE_MS=300000      # 5 minutes — reject events older than this
```

Rules:
- One set of secrets per provider. Never share secrets across providers.
- All secrets are stored in Vault (§14.2) for staging/production. Never in source code.
- If a provider does not support a layer (e.g., no IP list, no API key), omit that variable — do not set a dummy value.
- Adding a new third-party integration requires configuring all three layers and documenting which layers the provider supports in the integration's ADR.

---

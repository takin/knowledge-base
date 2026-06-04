# Infrastructure Stack: Docker Compose + Nginx

Updated: 2026-06-04
Status: Draft
Sources: Internal Tech Stacks draft (2026-04-15, updated 2026-05-23); Internal Dashboard Stack draft (2026-05-22, updated 2026-05-23); Internal TanStack Start OAuth/OIDC stack draft (2026-06-04)
Platform: Infrastructure
Runtime: Docker
Framework: Docker Compose + Nginx
Primary Use Case: Backend API deployment, Nginx gatewaying, Vault-backed secrets, TLS, compression, deterministic deploys, observability stack, scale tiers, and Kubernetes upgrade path
Raw: [2026-04-15-infra-docker-compose-nginx-stack.md](../../../raw/engineering/tech-stacks/2026-04-15-infra-docker-compose-nginx-stack.md); [2026-05-22-web-react-vite-dashboard-stack.md](../../../raw/engineering/tech-stacks/2026-05-22-web-react-vite-dashboard-stack.md); [2026-06-04-web-tanstack-start-oauth-oidc-stack.md](../../../raw/engineering/tech-stacks/2026-06-04-web-tanstack-start-oauth-oidc-stack.md)

## Summary

This stack defines VPS-first infrastructure for backend APIs, TanStack Start app runtimes, and approved runtime services. Nginx is the only host-facing HTTP service; API, app, and worker containers stay internal. Main Nginx gateways use `fholzer/nginx-brotli:<pinned-version>` with Brotli enabled by default. Static Vite dashboard deployment is a separate specialized profile documented in [React + Vite Dashboard](web-react-vite-dashboard.md).

## Environment And Secrets

Secrets are tiered by environment:

| Environment | Secrets source | Mechanism |
| --- | --- | --- |
| Local development | `.env` | Docker Compose `env_file` |
| Staging | HashiCorp Vault | Vault Agent or CI injection |
| Production | HashiCorp Vault | Vault Agent or CI injection |

Rules:
- Never hardcode secrets, API keys, DSNs, or database URLs.
- Never bake secrets into Docker images through `ENV` or build-time `ARG`.
- Vault OSS is the mandatory secrets manager for staging and production.
- `.env.example` is committed and documents all required variables.
- Staging/production `DATABASE_URL` points to PgBouncer, not directly to PostgreSQL 18+.

Baseline env categories include app config, database, JWT/JWKS, CORS, Redis/BullMQ, object storage, OpenTelemetry, and monitoring.

## Docker Image Size And Final Runtime Policy

Use Alpine, slim, distroless, or the smallest production-suitable image variant by default. Avoid large general-purpose Docker images when a smaller runtime image is available.

Rules:
- Pin exact image versions; never use `latest`.
- Prefer Alpine, slim, distroless, or smallest official production-suitable image variants.
- Avoid large general-purpose images unless required by native dependencies, libc compatibility, debugging, or vendor constraints.
- Non-minimal base image usage requires an ADR or documented implementation note.
- Final runtime images contain only runtime artifacts, production dependencies, required OS packages, and runtime config.
- Build tools, package manager caches, test artifacts, Playwright browsers, coverage, source maps meant for private upload, local files, `.env*`, `.git`, and development-only files must not remain in final runtime images.

JavaScript and TypeScript runtime images:
- Final runtime images for JavaScript and TypeScript applications must not include development dependencies or development `node_modules`.
- Build stages may install development dependencies.
- Runtime stages must install or receive production dependencies only.
- Do not copy development `node_modules` into the final runtime image.
- Do not run final runtime with `node_modules` produced by a development install.
- Use `bun install --frozen-lockfile --production` or the package-manager equivalent in the runtime dependency stage.
- If a dependency is needed at runtime, it belongs in production dependencies.
- If a dependency is needed only for build, test, typecheck, linting, formatting, codegen, or local development, it must not be present in the final image.
- Every JS/TS Docker build requires a final-image dependency review step.
- Review must verify that dev dependencies are absent from final `node_modules`.
- Review must verify package manager caches and development-only artifacts are absent.
- CI should fail when final images contain known dev-only packages such as test runners, linters, formatters, TypeScript compilers, Playwright browser bundles, local test utilities, or codegen-only packages unless an ADR documents a runtime need.

Stack-specific final image rules:
- React + Vite Dashboard final Nginx image contains zero `node_modules`; only `dist/`, Nginx config, entrypoint, and minimal runtime files are allowed.
- TanStack Start OAuth/OIDC App final app/worker image may contain runtime JS dependencies, but production dependencies only.
- Backend Bun + Elysia final API/worker image may contain runtime JS dependencies, but production dependencies only.
- Infrastructure services prefer pinned Alpine/slim/minimal variants for Redis, Nginx, Certbot, and other service images where production-suitable.

## Backend API Runtime Profile

Standalone Bun + Elysia backend APIs deploy as separate API and worker processes behind Nginx.

Required baseline services:
- `nginx`
- `api`
- `worker`
- `postgres 18+`
- `pgbouncer`
- `redis 8+`
- `otel-collector`
- `prometheus`
- `loki`
- `tempo`
- `grafana`
- `certbot`
- `certbot-renew`

Optional services include `minio`, `mailpit`, and `bull-board`.

Compose shape:

```yaml
services:
  api:
    image: example-api:${TAG}
    expose:
      - "3000"
    stop_grace_period: 30s
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL

  worker:
    image: example-api:${TAG}
    command: ["bun", "run", "worker"]
    stop_grace_period: 60s
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL

  nginx:
    build:
      context: ./nginx
    image: example-api-nginx:${TAG}
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - letsencrypt:/etc/letsencrypt:ro
      - certbot-webroot:/var/www/certbot:ro
    environment:
      NGINX_UPSTREAM_HOST: api
      NGINX_UPSTREAM_PORT: 3000
      NGINX_SERVER_NAME: api.example.com
      NGINX_BROTLI_ENABLED: "on"
      NGINX_GZIP_ENABLED: "on"
    depends_on:
      - api

  certbot:
    image: certbot/certbot:<pinned-version>
    volumes:
      - letsencrypt:/etc/letsencrypt
      - certbot-webroot:/var/www/certbot

  certbot-renew:
    image: certbot/certbot:<pinned-version>
    volumes:
      - letsencrypt:/etc/letsencrypt
      - certbot-webroot:/var/www/certbot

volumes:
  letsencrypt:
  certbot-webroot:
```

Rules:
- `api` and `worker` use the same application image with different commands.
- `api` handles HTTP traffic only.
- `worker` processes BullMQ jobs only.
- Do not run BullMQ workers inside the API process in production.
- `api` and `worker` both connect to PgBouncer, Redis 8+, object storage, and OpenTelemetry Collector.
- PostgreSQL 18+ is the baseline durable database service.
- Redis 8+ is mandatory for BullMQ, rate limiting, permission cache, and short-lived coordination.
- PgBouncer runs in transaction mode and connects to PostgreSQL 18+.
- The Nginx gateway image must be based on `fholzer/nginx-brotli:<pinned-version>` with Brotli enabled by default.
- Certbot sidecars own Let's Encrypt issuance and renewal; do not install Certbot into Nginx or API images.
- `api` `stop_grace_period` is at least `30s`; `worker` is at least `60s`.
- Production media storage should prefer managed S3-compatible storage; self-hosted MinIO requires backup/restore procedures.

## Nginx Backend API Template

The backend API Nginx template is for JSON APIs, not static dashboard SPAs. It does not include CSP nonce propagation, robots/sitemap handling, static asset caching, or SSR cache rules.

Key rules:
- Nginx applies baseline non-CSP security headers to API responses.
- Backend API CSP nonce handling is omitted by default because JSON APIs do not serve HTML.
- `/health` may be public to Nginx/load balancers.
- `/ready` is internal-only.
- `/metrics` is internal-only.
- `/docs` and `/docs/json` are blocked by baseline public Nginx unless the project intentionally auth-gates them.
- `limit_req` is coarse per-IP edge protection only; Elysia/Redis owns product rate limits.
- `NGINX_CLIENT_MAX_BODY_SIZE` defaults to `1m`; media uses presigned object-storage uploads.
- Set upstream timeouts such as `proxy_connect_timeout 5s`, `proxy_send_timeout 30s`, and `proxy_read_timeout 30s`.

Nginx runtime variables include `NGINX_SERVER_NAME`, `NGINX_UPSTREAM_HOST=api`, `NGINX_UPSTREAM_PORT=3000`, Brotli/Gzip toggles, worker/connection limits, body size, rate limit RPS, and TLS/HSTS switch. Port `80` must serve `/.well-known/acme-challenge/` from the shared Certbot webroot for ACME HTTP-01 validation.

## TanStack Start OAuth/OIDC Runtime Profile

TanStack Start OAuth/OIDC apps deploy as app and worker runtime processes behind Nginx. They are not static SPA artifacts and must not use the static Vite dashboard profile.

Required baseline services:
- `nginx`
- `app`
- `worker`
- `redis 8+`
- `postgres 18+` when the app owns product data
- `pgbouncer` when PostgreSQL is part of the deployment
- `certbot`
- `certbot-renew`

Optional services include observability stack services, `minio`, `mailpit`, and `bull-board` when admin-gated and network-restricted.

Compose shape:

```yaml
services:
  app:
    image: example-app:${TAG}
    command: ["bun", "run", "start"]
    expose:
      - "3000"
    stop_grace_period: 30s
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL

  worker:
    image: example-app:${TAG}
    command: ["bun", "run", "worker"]
    stop_grace_period: 60s
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL

  redis:
    image: redis:<pinned-version>
    expose:
      - "6379"

  nginx:
    build:
      context: ./nginx
    image: example-app-nginx:${TAG}
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - letsencrypt:/etc/letsencrypt:ro
      - certbot-webroot:/var/www/certbot:ro
    environment:
      NGINX_UPSTREAM_HOST: app
      NGINX_UPSTREAM_PORT: 3000
      NGINX_SERVER_NAME: app.example.com
      NGINX_BROTLI_ENABLED: "on"
      NGINX_GZIP_ENABLED: "on"
    depends_on:
      - app
```

Rules:
- `app` and `worker` use the same application image with different commands.
- `app` handles HTTP traffic, SSR, OAuth/OIDC callback routes, server functions, session handling, and job enqueueing.
- `worker` processes BullMQ jobs only.
- Do not run BullMQ workers inside the app process in production.
- Redis 8+ is mandatory for live sessions, OAuth callback state, PKCE verifier storage, CSRF state, rate limiting, distributed invalidation, and BullMQ queues.
- PostgreSQL is required when the app owns product data; app and worker connect through PgBouncer in staging and production.
- Nginx proxies to the `app` runtime and applies HTML-capable security headers, including CSP for browser responses.
- Set upstream timeouts for normal request latency. Long-running work must be enqueued through BullMQ instead of extending Nginx timeouts.
- `app` `stop_grace_period` is at least `30s`; `worker` is at least `60s`.
- Platform-specific adapters are allowed only by ADR documenting runtime constraints, secret handling, session behavior, and rollback path.

## Let's Encrypt Certbot Sidecars

Production deployments use Certbot sidecars for Let's Encrypt certificate issuance and renewal. Nginx terminates TLS; Certbot writes certificates and ACME challenge files into shared Docker volumes.

Rules:
- `nginx` mounts `letsencrypt:/etc/letsencrypt:ro` and `certbot-webroot:/var/www/certbot:ro`.
- `certbot` and `certbot-renew` mount both volumes read-write.
- Certbot containers do not bind host ports and use `certbot/certbot:<pinned-version>`, never `latest`.
- Certificate files live in Docker volumes, never in Git, image layers, or copied application assets.
- Certificate issuance and renewal must not require rebuilding API, worker, dashboard, or Nginx images.
- Nginx must reload after successful renewal.
- Do not generate certificates in `nginx/entrypoint.sh`.
- Self-signed certificates are banned in production, and expired certificates are production-blocking failures.

## Static SPA Dashboard Profile

Static Vite dashboards do not use the generic proxy-to-API template. They serve `dist/` directly from Nginx and follow [React + Vite Dashboard](web-react-vite-dashboard.md).

Dashboard invariants:
- Docker Compose production deployment.
- Nginx serves Vite `dist/` directly from `/usr/share/nginx/html`.
- No Bun, Node, Vite preview, custom static server, or internal dashboard app server in production runtime.
- For OAuth/OIDC callback handling, Redis-backed app sessions, protected server functions, BFF behavior, or app-owned resources, use [TanStack Start OAuth/OIDC App](web-tanstack-start-oauth-oidc.md) instead of this static profile.
- TLS terminates at Nginx with Let's Encrypt certificates in Docker volumes.
- Certbot sidecars handle issuance and renewal.
- Static hashed assets use immutable cache; `index.html` uses no-cache or must-revalidate.

## Deterministic Deployments

Deployment procedures must be deterministic. A script that checks only `http://localhost/ready` through Nginx is insufficient because the old instance can make that health check pass while the new instance is unhealthy.

Acceptable strategies:
- Blue/green services such as `api_blue` and `api_green`, switching Nginx only after the target color is healthy.
- Container-specific readiness checks against the newly-created container before old containers are removed.
- Kubernetes rolling updates at Tier 3.

Rules:
- Never run `docker compose restart` or direct `docker compose up` as the production deploy procedure.
- Use the project deployment script and test it in staging.
- The script must prove the new container/target is ready before removing the old target.
- API and worker processes must handle SIGTERM and drain cleanly.

## Graceful Shutdown

API shutdown sequence:
1. `api` receives SIGTERM.
2. `isShuttingDown = true`.
3. `/ready` returns `503`.
4. Nginx stops routing new requests.
5. API drains in-flight requests.
6. Active DB transactions commit or rollback.
7. SQL, Redis, BullMQ, logs, and telemetry close before `stop_grace_period` expires.

Worker shutdown sequence:
1. `worker` receives SIGTERM.
2. Worker stops taking new BullMQ jobs.
3. Active jobs finish or checkpoint within timeout.
4. BullMQ workers/queues close before shared Redis connections close.
5. SQL, Redis, and telemetry close before `stop_grace_period` expires.

## Scale Tiers

| Tier   | Concurrency | Orchestration               | Notes                                                                                     |
| ------ | ----------: | --------------------------- | ----------------------------------------------------------------------------------------- |
| Tier 0 |         100 | Single VM Docker Compose    | MVP/prototype, co-located services                                                        |
| Tier 1 |          2K | Docker Compose              | Nginx, API, worker, PgBouncer, PostgreSQL 18+, Redis 8+, Grafana stack                    |
| Tier 2 |         10K | Docker Compose or light K8s | More API instances, workers, DB read replicas, observability stack                        |
| Tier 3 |        100K | Kubernetes                  | API HPA, worker HPA, PDB, Redis 8+ Cluster, read replicas, OS tuning, observability stack |

Tier 3 requirements:
- API Deployment with `maxUnavailable: 0`, `minReadySeconds`, readiness/liveness probes, `preStop` sleep, and sufficient `terminationGracePeriodSeconds`.
- Worker Deployment using the same image and `command: ["bun", "run", "worker"]`.
- API and worker HPAs/PDBs sized independently.
- Redis 8+ Cluster for 100K+ concurrency.
- OS file descriptor tuning on every host running API containers.
- Tier 3 architecture routes API traffic to Redis/BullMQ, and workers consume from Redis/BullMQ. API pods do not call worker pods directly.

## See Also

- [Backend: Bun + Elysia](backend-bun-elysia.md)
- [TanStack Start OAuth/OIDC App](web-tanstack-start-oauth-oidc.md)
- [React + Vite Dashboard](web-react-vite-dashboard.md)
- [Security Baseline: Web Applications](security-web-app-baseline.md)
- [CI and Testing: TypeScript + React](ci-testing-typescript-react.md)

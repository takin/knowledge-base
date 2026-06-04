# Infrastructure Stack: Docker Compose + Nginx

Updated: 2026-05-24
Status: Draft
Sources: Internal Tech Stacks draft (2026-04-15, updated 2026-05-23); Internal Dashboard Stack draft (2026-05-22, updated 2026-05-23)
Platform: Infrastructure
Runtime: Docker
Framework: Docker Compose + Nginx
Primary Use Case: Backend API deployment, Nginx gatewaying, Vault-backed secrets, TLS, compression, deterministic deploys, observability stack, scale tiers, and Kubernetes upgrade path
Raw: [2026-04-15-infra-docker-compose-nginx-stack.md](../../../raw/engineering/tech-stacks/2026-04-15-infra-docker-compose-nginx-stack.md); [2026-05-22-web-react-vite-dashboard-stack.md](../../../raw/engineering/tech-stacks/2026-05-22-web-react-vite-dashboard-stack.md)

## Summary

This stack defines VPS-first infrastructure for backend APIs and approved runtime services. Nginx is the only host-facing HTTP service; API and worker containers stay internal. Main Nginx gateways use `fholzer/nginx-brotli:<pinned-version>` with Brotli enabled by default. Static Vite dashboard deployment is a separate specialized profile documented in [React + Vite Dashboard](web-react-vite-dashboard.md).

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
- [React + Vite Dashboard](web-react-vite-dashboard.md)
- [Security Baseline: Web Applications](security-web-app-baseline.md)
- [CI and Testing: TypeScript + React](ci-testing-typescript-react.md)

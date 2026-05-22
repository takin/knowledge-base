# Security Baseline: Web Applications

Source URL: Internal draft
Collected: 2026-04-15
Published: 2026-04-15
Status: Draft
Scope: Cross-stack web application security baseline

---

## 12. Security Hardening

Security hardening is mandatory from day one. It is not a pre-launch checklist item.

### 12.1 HTTP security headers

Every response must include the following headers. Configure these at the server/middleware level, not per-route:

| Header | Value | Purpose |
|---|---|---|
| `Content-Security-Policy` | Restrictive nonce-based policy, at minimum `default-src 'self'` plus `script-src 'self' 'nonce-$csp_nonce'` | Prevents XSS by blocking unauthorized script sources |
| `X-Content-Type-Options` | `nosniff` | Prevents MIME-type sniffing |
| `X-Frame-Options` | `DENY` or `SAMEORIGIN` | Prevents clickjacking |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Controls referrer leakage |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` | Enforces HTTPS (production only) |
| `Cross-Origin-Opener-Policy` | `same-origin` | Prevents Spectre-family attacks by isolating browsing context |
| `Permissions-Policy` | Restrict camera, mic, geolocation to needed origins only | Limits browser feature access |

### 12.1.1 CSP nonce for TanStack Start SSR

All TanStack Start applications must implement request-scoped CSP nonce support. The nonce must be generated at the HTTP edge, forwarded into the app, passed through TanStack Start server context, and used by TanStack Router SSR.

Mandatory rules:
- Generate or receive one nonce per request at the Nginx/server-entry boundary.
- Nginx forwards the nonce to the app with `X-CSP-Nonce`.
- The app server entry reads `x-csp-nonce` and passes it into TanStack Start context.
- Expose a shared isomorphic `getCspNonce()` helper.
- Configure TanStack Router with `ssr.nonce: getCspNonce()`.
- The root document emits `<meta property="csp-nonce" content={nonce} />` when the client helper depends on DOM lookup.
- The client-side nonce reader must return `undefined`, not an empty string, when no nonce meta tag exists. This prevents SSR/client hydration mismatches.
- Product-specific CSP sources may be added for analytics, payment, storage, frames, or media CDNs, but the nonce pattern must not be removed.
- Every inline script, including JSON-LD, analytics bootstrap, and theme bootstrapping scripts, must receive `nonce={nonce}`.
- JSON-LD scripts must escape `<` as `\\u003c` before injecting serialized JSON.

Third-party CSP source rules:
- The default CSP must not hardcode third-party analytics, payment, storage, iframe, or regional collection endpoints.
- Add third-party sources only when the project actually uses that provider.
- Google Analytics may require `https://www.google-analytics.com`.
- Some Google Analytics deployments may require a regional endpoint such as `https://region1.google-analytics.com`, but this varies by project and must be added explicitly after verification.
- Google Tag Manager requires script permission for `https://www.googletagmanager.com` when used.
- Every third-party CSP source must be justified in the project ADR or security review.

Reference implementation shape:

`src/server.ts`

```typescript
import handler, { createServerEntry } from '@tanstack/react-start/server-entry'

export default createServerEntry({
  fetch(request) {
    const nonce =
      request.headers.get('x-csp-nonce') ?? crypto.randomUUID().replaceAll('-', '')

    return handler.fetch(request, {
      context: { nonce },
    })
  },
})
```

`src/lib/csp.ts`

```typescript
import {
  createIsomorphicFn,
  getGlobalStartContext,
} from '@tanstack/react-start'

export const getCspNonce = createIsomorphicFn()
  .server(() => {
    const ctx = getGlobalStartContext()
    return (ctx as { nonce?: string } | undefined)?.nonce
  })
  .client(() => {
    const el = document.querySelector<HTMLMetaElement>(
      'meta[property="csp-nonce"]',
    )

    return el?.content || undefined
  })
```

`src/router.tsx`

```typescript
import { createRouter as createTanStackRouter } from '@tanstack/react-router'
import { getCspNonce } from './lib/csp'
import { routeTree } from './routeTree.gen'

export function getRouter() {
  const router = createTanStackRouter({
    ssr: {
      nonce: getCspNonce(),
    },
    routeTree,
    scrollRestoration: true,
    defaultPreload: 'intent',
    defaultPreloadStaleTime: 0,
  })

  return router
}
```

`src/routes/__root.tsx` root document requirements:

```tsx
function RootDocument({ children }: { children: React.ReactNode }) {
  const nonce = getCspNonce()

  return (
    <html suppressHydrationWarning>
      <head>
        <meta property="csp-nonce" content={nonce} />
        <HeadContent />
      </head>
      <body>
        {children}
        <Scripts />
      </body>
    </html>
  )
}
```

Optional JSON-LD helper:

```tsx
export function JsonLd({ data }: { data: JsonLdValue }) {
  const nonce = getCspNonce()

  return (
    <script
      nonce={nonce}
      type="application/ld+json"
      suppressHydrationWarning
      dangerouslySetInnerHTML={{
        __html: JSON.stringify(data).replaceAll('<', '\\u003c'),
      }}
    />
  )
}
```

### 12.2 CSRF protection

Better Auth handles CSRF protection via `SameSite=Strict` or `SameSite=Lax` session cookies combined with a CSRF token header check. Do not implement custom CSRF logic on top of Better Auth.

For server functions that mutate state and are called from non-auth contexts (e.g., public-facing forms), verify the `Origin` header server-side and reject requests from unexpected origins.

### 12.3 XSS prevention

- Never use `dangerouslySetInnerHTML`. If you must render HTML from an external source, sanitize it with DOMPurify before rendering.
- Zod validation on every server function input prevents injection via unvalidated user data.
- CSP headers (§12.1) are the last line of defense — do not rely on them as the primary mitigation.
- Do not use `eval()`, `new Function()`, or dynamic `import()` with user-controlled paths.

### 12.4 SQL injection

Drizzle's parameterized query builder prevents SQL injection by construction. Raw SQL strings in application code are banned (see §8.2). If a raw query is unavoidable, use Drizzle's `sql` tagged template literal, which parameterizes values automatically.

### 12.5 Rate limiting

All authentication endpoints (login, signup, password reset, OTP) must be rate-limited. General API endpoints must have a baseline rate limit.

- If Redis is available (see §13): use a Redis-backed rate limiter (e.g., `@upstash/ratelimit` or a custom sliding window implementation with `ioredis`).
- If Redis is not available: use an in-memory rate limiter as a fallback, with the understanding that it does not work across multiple instances.

Rate limit responses must return HTTP `429 Too Many Requests` with a `Retry-After` header.

### 12.6 Input sanitization & output encoding

- Validate all inputs with Zod before processing (mandatory, §9.2).
- Encode user-generated content when rendering it back to the page (React's JSX does this by default for text nodes — do not bypass it).
- File uploads: validate MIME type server-side (not just by extension), enforce size limits, and store outside the web root.

### 12.7 Sensitive data handling

- Never log PII (names, emails, phone numbers, payment info) to any logging service including Sentry.
- Never store plaintext passwords. Better Auth handles hashing; do not touch passwords directly.
- Mask sensitive fields (card numbers, tokens) in all log output.
- Tokens and secrets in `.env` — never in source code, database, or API responses.

### 12.8 Dependency security

- Run `bun audit` (or equivalent) in the CI pipeline on every PR.
- Pin exact dependency versions in `package.json` for production dependencies. Use `^` only for dev dependencies.
- Review Dependabot / Renovate alerts within one sprint for high/critical severity.

### 12.8.1 TanStack supply-chain advisory handling

All projects adopting this standard using `@tanstack/*` packages must follow TanStack Router advisory `GHSA-g7cv-rxg3-hmpx` / `CVE-2026-45321`. This advisory covers a critical supply-chain compromise where malicious `@tanstack/*` package versions exfiltrated cloud credentials, GitHub tokens, npm tokens, SSH keys, Vault tokens, and other secrets during install.

Mandatory rules:
- Never install affected `@tanstack/*` versions listed in `GHSA-g7cv-rxg3-hmpx`.
- Pin all production `@tanstack/*` dependencies to explicit known-good or patched versions.
- Do not use loose production dependency ranges for `@tanstack/*` packages.
- If an affected version may have been installed, delete `node_modules` and the lockfile, then reinstall from a clean lockfile.
- Treat any CI runner or developer machine that installed affected versions during the advisory window as compromised.
- Rotate all credentials accessible to the install process after suspected exposure, including cloud, GitHub, npm, SSH, Vault, and CI secrets.
- Audit cloud logs, GitHub logs, npm publish/access logs, SSH usage, Vault logs, and CI logs after suspected exposure.
- During an active supply-chain incident, CI may temporarily install with lifecycle scripts disabled using `bun install --ignore-scripts`. Re-enable scripts only after dependency provenance has been reviewed.

Minimum patched versions for relevant stack packages:

| Package | Minimum patched version |
|---|---:|
| `@tanstack/react-router` | `1.169.9` |
| `@tanstack/router-core` | `1.169.9` |
| `@tanstack/react-start` | `1.167.72` |
| `@tanstack/router-cli` | `1.166.50` |
| `@tanstack/router-plugin` | `1.167.42` |
| `@tanstack/router-vite-plugin` | `1.166.57` |
| `@tanstack/start-plugin-core` | `1.169.27` |
| `@tanstack/eslint-plugin-router` | `1.161.13` |
| `@tanstack/eslint-plugin-start` | `0.0.8` |

If a project uses any other `@tanstack/*` package, check the advisory table and pin to its patched version or newer.

Indicators that must trigger incident response:
- `optionalDependencies` containing `@tanstack/setup`.
- Git dependency `github:tanstack/router#79ac49eedf774dd4b0cfa308722bc463cfe5885c`.
- Payload file `router_init.js` in a package tarball or installed package.
- Helper file `tanstack_runner.js`.
- Network calls to `filev2.getsession.org`, `seed1.getsession.org`, `seed2.getsession.org`, or `seed3.getsession.org`.

CI must fail if:
- `package.json` or the lockfile resolves to an affected `@tanstack/*` version.
- A resolved package manifest contains `@tanstack/setup`.
- `router_init.js` appears inside a fetched `@tanstack/*` tarball or installed package.

Forensic exception: `npm pack <package>@<version>` is allowed only for security inspection because it downloads a package tarball without running lifecycle scripts. It must not be used for normal installation or project workflows.

---

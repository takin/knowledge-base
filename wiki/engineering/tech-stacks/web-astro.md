# Web Stack: Astro

Updated: 2026-06-09
Status: Draft
Sources: Internal Tech Stacks draft (2026-05-22, updated 2026-06-09); Internal standalone repository structure draft (2026-06-09)
Platform: Web / public landing
Runtime: Bun
Framework: Astro
Primary Use Case: Astro web projects, commonly public marketing websites, SEO pages, MDX-in-repo blogs, static-first landing pages, and Cloudflare Pages deployment
Raw: [2026-05-22-web-astro-stack.md](../../../raw/engineering/tech-stacks/2026-05-22-web-astro-stack.md); [2026-06-09-repository-structure-standalone.md](../../../raw/engineering/tech-stacks/2026-06-09-repository-structure-standalone.md)

## Summary

Astro is the standard for Astro web projects. It is commonly the best fit for landing pages, public marketing sites, SEO pages, and MDX-in-repo content. The baseline is static-first, MDX-in-repo, SEO-oriented, and deployed to Cloudflare Pages as static `dist/` output.

Astro SSR is not part of the baseline. It requires a product-level ADR naming the freshness requirement, cache strategy, and deployment adapter. React is allowed only for isolated islands, not for hydrating full marketing pages.

## Runtime And Toolchain

- Use Bun for installs, scripts, builds, and tests.
- Use TypeScript strict mode.
- Build output is static `dist/` by default.
- Do not add a server runtime unless SSR is explicitly approved.
- Do not use npm, yarn, or pnpm.
- Production dependencies must be pinned to exact versions.

Required script contract:

```json
{
  "scripts": {
    "dev": "astro dev",
    "build": "astro check && astro build",
    "preview": "astro preview",
    "typecheck": "astro check",
    "lint": "eslint .",
    "test": "bun test"
  }
}
```

## Framework And Rendering

- Astro is mandatory for landing projects.
- Landing is static-first by default.
- Blog pages are generated at build time.
- Do not build landing pages as React SPAs.
- Do not use fullstack SSR frameworks for this landing standard.
- React islands are allowed only for isolated interactivity.
- Use the Astro Cloudflare adapter only when SSR is approved.

## Project Structure

```text
src/
  content/
    blog/
      example-post.mdx
    config.ts
  pages/
    index.astro
    pricing.astro
    features.astro
    about.astro
    contact.astro
    blog/
      index.astro
      [slug].astro
    rss.xml.ts
  layouts/
    BaseLayout.astro
    MarketingLayout.astro
    BlogPostLayout.astro
  components/
    marketing/
    blog/
    islands/
    seo/
  lib/
    blog.ts
    seo.ts
    site.ts
  styles/
    app.css
public/
  images/
  robots.txt
astro.config.mjs
```

Rules:
- `src/pages/**` owns routes only.
- Static marketing components live in `src/components/marketing/**`.
- Blog components live in `src/components/blog/**`.
- Interactive React components live in `src/components/islands/**`.
- SEO helpers live in `src/components/seo/**` or `src/lib/seo.ts`.
- Site constants live in `src/lib/site.ts`.
- Do not place dashboard, admin, or authenticated product-app routes in the landing repo.

## MDX Content Model

Blog content lives in repo as MDX and uses Astro Content Collections.

Required frontmatter:

```yaml
---
title: "Post title"
description: "SEO description"
publishedAt: "2026-05-22"
updatedAt: "2026-05-22"
author: "Team"
tags: ["saas", "product"]
draft: false
image: "/images/blog/post-cover.jpg"
---
```

Rules:
- Blog posts live in `src/content/blog/*.mdx`.
- Use `src/content/config.ts` to validate schema.
- Do not use an external CMS by default.
- Draft posts are excluded from production builds.
- Every published post must be reviewable in a pull request.
- Do not fetch MDX blog content from the backend API at runtime when static generation is enough.

## Blog Routing

- `/blog` lists published posts.
- `/blog/[slug]` renders individual posts using `getStaticPaths()`.
- `/rss.xml` emits the RSS feed when blog is enabled.
- `/sitemap.xml` includes static pages and published blog posts.
- Use `getCollection('blog')` to read posts.
- Filter `draft: true` in production.
- Sort posts by `publishedAt` descending unless a page states otherwise.
- Return 404 for unknown slugs.
- Do not rely on client-side routing for blog pages.

## SEO

SEO is a first-class requirement.

- Every public page defines title, description, canonical URL, Open Graph metadata, social image, and Twitter card metadata.
- Blog posts derive metadata from frontmatter unless overridden.
- Canonical URLs use the production site URL from config.
- The home page, pricing page, feature pages, and blog posts need unique titles and descriptions.
- Do not ship placeholder metadata.
- Use semantic headings with one meaningful `h1` per page.
- Images that convey content require meaningful `alt` text.
- JSON-LD is allowed only through a helper that escapes `<` as `\u003c` before injection.

## Sitemap, RSS, And Robots

- Generate `sitemap.xml` before launch.
- Include published posts in sitemap.
- Exclude drafts, preview-only routes, and internal pages.
- Provide `robots.txt` in `public/`.
- `robots.txt` references the sitemap URL.
- Generate RSS when `/blog` exists.
- RSS entries include title, link, description, publication date, and stable GUID.

## Styling And Islands

Tailwind CSS is required. Use Tailwind utilities and CSS theme tokens. Do not use CSS Modules, styled-components, Emotion, or other CSS-in-JS libraries. Shadcn UI is optional for landing projects and requires user confirmation before adoption.

React islands are allowed for pricing calculators, ROI calculators, interactive demos, newsletter enhancements, lightweight toggles, and client-only embeds. Poor candidates include static hero sections, feature cards, testimonials, FAQ content that can use HTML details/summary, blog body, and full-page shells.

Island rules:
- Default to Astro components and HTML.
- Add React integration only for real interactivity.
- Hydrate with the narrowest Astro client directive that satisfies UX.
- Avoid `client:load` for below-the-fold or non-critical islands.
- Prefer `client:visible` or `client:idle` for non-critical islands.
- Measure JavaScript added by each island.

## Forms And API Usage

- Landing forms may submit to the external backend API, a webhook endpoint, or an approved form provider.
- Landing repo does not own backend business logic.
- Landing repo does not access a database.
- Public forms validate client-side for UX and server-side in the backend API.
- Do not trust client-side validation as security.
- API base URLs are environment-configured.
- Do not expose secrets in public environment variables.
- Public forms need accessible success, loading, and error states.
- Spam protection is required for public lead/contact forms before production launch.

## Cloudflare Pages Deployment

Cloudflare Pages is the default target.

- Production deploys publish `dist/`.
- Use Cloudflare preview deployments for pull requests.
- Configure production environment variables in Cloudflare.
- Do not commit `.env` or `.env.*` except `.env.example`.
- If SSR is approved, use the Astro Cloudflare adapter and document route-level caching.

Baseline build:

```bash
bun install --frozen-lockfile
bun run build
```

## Optional Docker Runtime

Cloudflare Pages remains the default deployment target for static landing projects. When a landing project explicitly adopts VPS/Docker deployment, every Docker image must use pinned Alpine, slim, distroless, or smallest production-suitable variants when compatible, and final runtime images target below `200 MB`.

Static Astro output should use a minimal pinned Nginx or Brotli-enabled Nginx runtime that serves `dist/` directly and contains zero `node_modules`.

Approved Astro SSR or Node-compatible server output uses a Bun Alpine build stage, a production-only `runtime-deps` stage, and a pinned Node Alpine runtime when the output is Node-compatible. The runtime dependency stage sets `NODE_ENV=production`, runs `bun install --frozen-lockfile --production --linker hoisted`, and removes Bun install cache and `/tmp/*` before the final image copies `node_modules`.

Rules:
- Use the Node Alpine runtime only when the build output is Node-compatible.
- Use a Bun Alpine/slim runtime when runtime behavior depends on Bun APIs.
- Do not copy `node_modules` from build/dev stages into the final runtime image.
- Final SSR runtime `node_modules` must come only from a dedicated production-only `runtime-deps` stage.
- In monorepos, copy only package manifests needed by the Astro app or use a minimal generated runtime `package.json` so unrelated workspace packages are not installed.
- Do not copy source files, development `node_modules`, test artifacts, coverage, build caches, install caches, temporary files, `.env*`, `.git`, or public source maps into the final runtime image.
- Upload private source maps before producing the final runtime image and remove public `.map` files from served assets.

## Security

- Never use unsanitized `set:html` with untrusted content.
- MDX content in repo is trusted only after PR review.
- Sanitize external HTML if rendering it becomes necessary.
- JSON-LD helpers escape `<` as `\u003c`.
- Third-party scripts require explicit review.
- Analytics domains are added intentionally to CSP or Cloudflare headers.
- Do not put secrets, API keys, DSNs, tokens, or private endpoints in source code.
- Do not add a service worker by default.

## Testing And CI

Required checks:
- `bun install --frozen-lockfile`
- `bun run typecheck`
- `bun run lint`
- `bun run test`
- `bun run build`

Recommended checks:
- Link checking for public pages.
- Accessibility checks for marketing and blog templates.
- Smoke tests for `/`, `/blog`, one `/blog/[slug]`, `/sitemap.xml`, and `/rss.xml`.
- Lighthouse or equivalent checks before launch.

## Agent Rules

- Load TypeScript best-practices guidance before editing TypeScript.
- Consult current docs for Astro, MDX, Tailwind, and Cloudflare Pages before setup or framework-specific code.
- Ask before adding Shadcn UI.
- Keep landing, dashboard, and API repositories separate.
- Do not introduce backend routes, database code, or dashboard app logic.
- Preserve static generation unless SSR is explicitly approved.
- Update `.env.example` when environment variables change.
- Verify production build before declaring work ready.

## Anti-Patterns

| Anti-pattern | Why banned | Alternative |
| --- | --- | --- |
| Landing as React SPA | Adds JavaScript and weakens SEO for static content | Astro static pages |
| Runtime blog fetch for MDX | Slower and unnecessary for repo content | Content Collections + static generation |
| CMS by default | Adds workflow and integration cost before need exists | MDX in repo |
| Dashboard routes in landing repo | Blurs ownership and deployment | Separate dashboard repo |
| Backend routes in landing repo | Violates API boundary | Separate API repo |
| Database access from landing | Security and operational risk | Backend API owns persistence |
| Hydrating full static pages | Wastes client JavaScript | Astro components + targeted islands |
| `client:load` everywhere | Forces non-critical JS into startup | `client:visible`, `client:idle`, or no island |
| Unsanitized `set:html` | XSS risk | MDX review or sanitizer |
| Publishing drafts | Leaks unfinished content | Filter `draft: true` in production |
| Service worker by default | Stale-cache deploy bugs | Add PWA only with explicit requirement |

## See Also

- [React + Vite](web-react-vite.md)
- [Security Baseline: Web Applications](security-web-app-baseline.md)
- [CI and Testing: TypeScript + React](ci-testing-typescript-react.md)
- [Repository Structure: Standalone Repos](repository-structure-standalone.md)

# Web Stack: Astro Landing

Source URL: Internal draft
Collected: 2026-05-22
Published: 2026-05-22
Updated: 2026-05-22
Status: Draft
Scope: Public marketing websites, SEO pages, MDX blogs, static-first landing pages, and Cloudflare Pages deployment

---

## 1. Runtime & Toolchain

Astro landing projects use **Bun** as the package manager, script runner, and baseline test runner.

Rules:
- Use `bun` for dependency installs, scripts, builds, and test commands.
- Do not use npm, yarn, or pnpm in projects adopting this standard.
- Use TypeScript in strict mode.
- Build output is static by default. The standard deploy artifact is the generated `dist/` directory.
- Do not add a server runtime unless Astro SSR is explicitly approved by product-level ADR.
- Production dependencies must be pinned to exact versions. Dev dependencies may use normal controlled ranges if lockfile changes are reviewed.

Required `package.json` script contract:

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

## 2. Framework & Rendering Model

**Astro** is the required framework for landing page projects.

Rules:
- Landing projects are static-first.
- Use Astro's default static output for production unless real-time content requires SSR.
- Astro SSR is not part of the baseline. It requires a product-level ADR naming the freshness requirement, cache strategy, and deployment adapter.
- Do not build landing pages as React SPAs.
- Do not use fullstack SSR frameworks for projects adopting this landing standard.
- React is allowed only as an island framework for isolated interactive components.
- Do not hydrate full marketing pages as React applications.

SSR exception:
- SSR may be adopted only when content must appear without rebuild, preview must be request-time, or personalization is required.
- If SSR is adopted on Cloudflare, use the official Astro Cloudflare adapter and document cache behavior per route.
- SSR routes must still preserve SEO metadata, sitemap behavior, security headers, and safe fallback states.

## 3. Project Initialization

New landing projects are initialized with Astro and Bun:

```bash
bun create astro@latest my-landing
```

Post-init rules:
- Select TypeScript.
- Add Tailwind CSS.
- Add MDX support.
- Add Astro Content Collections for blog schema validation.
- Add sitemap support before production launch.
- Add React integration only if interactive islands are needed.
- Review generated files and lockfile changes before merge.

Generation-tool rule: `@latest` is allowed only for one-time project generation commands. Runtime dependencies committed to `package.json` must be reviewed and pinned according to the supply-chain policy.

## 4. Standard Directory Layout

Every Astro landing project uses this structure unless a product-level ADR documents an exception:

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
- Layouts live in `src/layouts/**` and own repeated page chrome, SEO slots, and document structure.
- Static marketing components live in `src/components/marketing/**`.
- Blog-specific components live in `src/components/blog/**`.
- Interactive React components live in `src/components/islands/**`.
- SEO helpers and JSON-LD helpers live in `src/components/seo/**` or `src/lib/seo.ts`.
- Blog helper functions live in `src/lib/blog.ts`.
- Site constants such as `siteUrl`, default title, default description, and social image live in `src/lib/site.ts`.
- Global CSS and Tailwind theme entry live in `src/styles/app.css`.
- Do not place dashboard, admin, or authenticated product-app routes in the landing repo.

## 5. Content Model: MDX In Repo

Landing blog content lives in the repo as MDX.

Rules:
- Use Astro Content Collections for blog content.
- Blog posts live in `src/content/blog/*.mdx`.
- Do not use an external CMS by default.
- Do not fetch blog content from the backend API at runtime when static generation is sufficient.
- Every content schema change must update `src/content/config.ts`.
- Every published post must be reviewable in a pull request.
- Draft posts must be excluded from production builds.

Required blog frontmatter:

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

Frontmatter rules:
- `title` is required and must be human-readable.
- `description` is required and should be usable as the meta description.
- `publishedAt` is required for every published post.
- `updatedAt` is required when content changes materially.
- `author` is required.
- `tags` is required and must be an array.
- `draft` is required.
- `image` is required for Open Graph and social sharing.

## 6. Blog Routing

Blog routes are generated statically.

Required routes:
- `/blog` lists published posts.
- `/blog/[slug]` renders individual posts.
- `/rss.xml` emits the blog RSS feed when blog is enabled.
- `/sitemap.xml` includes all published static pages and published blog posts.

Rules:
- Use `getCollection('blog')` to read posts.
- Filter out `draft: true` in production.
- Sort posts by `publishedAt` descending unless a page explicitly states another order.
- Slugs come from content collection IDs or explicit slug fields. Pick one convention and keep it consistent.
- Use `getStaticPaths()` for `/blog/[slug]`.
- Return 404 for unknown slugs.
- Do not rely on client-side routing for blog pages.

## 7. SEO Requirements

SEO is a first-class requirement for landing projects.

Rules:
- Every public page defines `title`, `description`, canonical URL, Open Graph title, Open Graph description, Open Graph image, and Twitter card metadata.
- Blog posts derive metadata from frontmatter unless explicitly overridden.
- Canonical URLs must use the production site URL from config.
- The home page, pricing page, feature pages, and blog posts must have unique titles and descriptions.
- Do not ship placeholder metadata.
- JSON-LD is allowed only through an approved helper that escapes `<` as `\u003c` before injection.
- Do not inject untrusted HTML into metadata or JSON-LD.
- Use semantic headings. Each page has one meaningful `h1`.
- Images that convey content require meaningful `alt` text.

## 8. Sitemap, RSS, And Robots

Rules:
- Generate `sitemap.xml` before production launch.
- Include published blog posts in the sitemap.
- Exclude drafts, preview-only routes, and internal pages.
- Provide `robots.txt` in `public/`.
- `robots.txt` must explicitly reference the sitemap URL.
- Generate RSS when `/blog` exists.
- RSS entries must include title, link, description, publication date, and stable GUID.

## 9. Styling & UI

Tailwind CSS is the required styling system.

Rules:
- Use Tailwind utilities and CSS theme tokens.
- Do not use CSS Modules, styled-components, Emotion, or other CSS-in-JS libraries.
- Keep custom CSS limited to global tokens, typography defaults, and rare third-party integration fixes.
- Design mobile-first and test from 375px width upward.
- Dark mode is optional for pure marketing sites unless the brand/product requires it.
- If dark mode is adopted, implement it consistently across pages and blog content.
- Shadcn UI is optional for landing projects. Ask before adding it.
- Avoid generic SaaS template layout unless the product's brand intentionally calls for it.

## 10. Interactive Islands

React islands are allowed for isolated interactivity.

Good candidates:
- Pricing calculator.
- ROI calculator.
- Interactive demo preview.
- Newsletter form enhancement.
- Lightweight comparison toggles.
- Client-only embeds that cannot render safely at build time.

Poor candidates:
- Static hero sections.
- Feature cards.
- Testimonials.
- FAQ accordions that can be implemented with HTML details/summary.
- Blog article body.
- Primary navigation unless a mobile menu requires it.

Rules:
- Default to Astro components and HTML.
- Add React integration only when a real interactive island exists.
- Hydrate islands with the narrowest Astro client directive that satisfies UX.
- Do not use `client:load` for below-the-fold or non-critical islands by default.
- Prefer `client:visible` or `client:idle` for non-critical islands.
- Measure JavaScript added by each island.

## 11. Forms & Backend API Usage

Landing forms may submit to the external backend API, a webhook endpoint, or an approved form provider.

Rules:
- Landing repo does not own backend business logic.
- Landing repo does not access a database.
- Public forms must validate input client-side for UX and server-side in the backend API.
- Never trust client-side validation as security.
- Form submissions that call the backend API must use environment-configured API base URLs.
- Do not hardcode production API URLs inside components.
- Do not expose secrets in `PUBLIC_*` environment variables.
- Public forms must have accessible success, loading, and error states.
- Spam protection is required for public lead/contact forms before production launch.

## 12. Images, Assets, And Performance

Rules:
- Use Astro image optimization where applicable.
- Prefer AVIF/WebP for large marketing images.
- Keep above-the-fold images dimensioned to avoid layout shift.
- Use explicit width/height for images where possible.
- Lazy-load below-the-fold media.
- Do not ship autoplay video by default.
- Use local assets for brand-critical images unless a CDN is explicitly required.
- Avoid large client-side animation libraries unless the page's core concept requires them.
- Target excellent Lighthouse SEO, accessibility, and best-practices scores before launch.

## 13. Deployment: Cloudflare Pages

Cloudflare Pages is the default deployment target for landing projects.

Rules:
- Production deploys build the static Astro site and publish `dist/`.
- Use Cloudflare preview deployments for pull requests.
- Configure production environment variables in Cloudflare, not in source code.
- Do not commit `.env` or `.env.*` files except `.env.example`.
- If SSR is approved, use the Astro Cloudflare adapter and document the route-level caching strategy.
- Do not use Cloudflare Workers runtime features in a static project unless the project explicitly adopts SSR or edge functions.

Baseline build command:

```bash
bun install --frozen-lockfile
bun run build
```

Baseline output directory:

```text
dist
```

## 14. Environment Variables

Rules:
- `.env` is local-only and ignored by Git.
- `.env.example` documents all variables and is committed.
- Public browser-exposed variables must use Astro's public env convention.
- Secrets must never be exposed to client-side code.
- API base URLs, analytics IDs, and feature flags must be environment-configured.

Baseline variables:

```env
PUBLIC_SITE_URL=https://example.com
PUBLIC_API_BASE_URL=https://api.example.com
PUBLIC_ANALYTICS_ENABLED=false
```

## 15. Security Baseline

Rules:
- Never use unsanitized `set:html` with untrusted content.
- MDX content in repo is trusted only after PR review.
- If rendering external HTML becomes necessary, sanitize it first and document the source.
- JSON-LD helpers must escape `<` as `\u003c`.
- Third-party scripts require explicit review.
- Analytics domains must be added intentionally to CSP or Cloudflare headers, not copied into every project by default.
- Do not put secrets, API keys, DSNs, tokens, or private endpoints in source code.
- Do not add a service worker by default.

## 16. Testing & CI

Required CI checks:
- `bun install --frozen-lockfile`
- `bun run typecheck`
- `bun run lint`
- `bun run test`
- `bun run build`

Recommended checks:
- Link checking for public pages.
- Accessibility checks for marketing and blog templates.
- Smoke test that `/`, `/blog`, at least one `/blog/[slug]`, `/sitemap.xml`, and `/rss.xml` build successfully.
- Lighthouse or equivalent checks before launch.

## 17. Agent Implementation Rules

When an AI agent implements an Astro landing project:
- Load TypeScript best-practices guidance before editing TypeScript files.
- Use Context7 or current docs for Astro, MDX, Tailwind, and Cloudflare Pages before project setup or framework-specific code.
- Ask before adding Shadcn UI.
- Keep landing, dashboard, and API repositories separate.
- Do not introduce backend routes, database code, or dashboard app logic.
- Preserve static generation unless the user explicitly approves SSR.
- Add or update `.env.example` whenever environment variables change.
- Verify production build before declaring the project ready.

## 18. Anti-Patterns

The following are banned:

| Anti-pattern | Why banned | Alternative |
|---|---|---|
| Building landing as a React SPA | Adds JavaScript and weakens SEO for static content | Astro static pages |
| Runtime blog fetch for MDX content | Slower, more failure modes, unnecessary for repo content | Astro Content Collections + static generation |
| Adding a CMS by default | Adds workflow and integration cost before need exists | MDX in repo |
| Dashboard routes in landing repo | Blurs product boundary and deployment ownership | Separate dashboard repo |
| Backend API routes in landing repo | Violates API ownership boundary | Separate API repo |
| Database access from landing | Security and operational risk | Backend API owns persistence |
| Hydrating full static pages | Wastes client JavaScript | Astro components + islands only where needed |
| `client:load` everywhere | Forces non-critical JS into startup | `client:visible`, `client:idle`, or no island |
| Unsanitized `set:html` | XSS risk | MDX review or sanitizer |
| Missing metadata | SEO and social sharing regressions | Required SEO helper/layout |
| Publishing drafts | Leaks unfinished content | Filter `draft: true` in production |
| Service worker by default | Stale-cache deploy bugs | Add PWA only with explicit product requirement |

## 19. Open Questions

All current architectural decisions are resolved for the baseline landing stack.

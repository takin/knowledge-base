---
name: web-astro-landing
description: "Use when building, reviewing, or modifying public marketing/SEO landing projects using Astro, Bun, MDX Content Collections, Tailwind CSS, static generation, React islands, forms, sitemap/RSS/robots, Cloudflare Pages, or landing-page CI/testing."
---

# Web Landing: Astro

This skill applies the repository wiki standard for public marketing and SEO landing projects. Use it before writing or reviewing code for Astro landing pages, MDX blogs, SEO metadata, sitemap/RSS/robots, Tailwind styling, React islands, public forms, Cloudflare Pages deployment, and landing-page CI checks.

Primary source of truth: `wiki/engineering/tech-stacks/web-astro-landing.md`.

Related standards:
- `wiki/engineering/tech-stacks/security-web-app-baseline.md`
- `wiki/engineering/tech-stacks/ci-testing-typescript-react.md`
- `wiki/engineering/tech-stacks/web-react-vite-dashboard.md`
- `wiki/engineering/tech-stacks/agent-skills-typescript-product.md`

## Required Skill And Docs Loading

Before implementing landing code, load and apply these skills when available:

- `typescript-best-practices` for TypeScript and JavaScript code.
- `shadcn` only if the human confirms the landing page will use Shadcn UI.

Shadcn UI is optional for landing projects. Before loading or initializing Shadcn, ask: "Will this landing page use Shadcn UI, or will you build with custom components / a different component library?"

For setup or framework-specific code, consult current documentation for Astro, MDX/Astro Content Collections, Tailwind CSS, and Cloudflare Pages before implementing.

If a skill or documentation source is unavailable, continue using the rules in this skill and state the gap in the final response.

## Stack Invariants

- Astro is mandatory for landing projects.
- Runtime and package manager is Bun.
- Use Bun for installs, scripts, builds, and tests.
- Use TypeScript strict mode.
- Landing is static-first by default.
- Build output is static `dist/` by default.
- Landing repositories are standalone and separate from dashboard and API repositories.
- Do not build landing pages as React SPAs.
- Do not use fullstack SSR frameworks for this landing standard.
- Do not add a server runtime unless SSR is explicitly approved.
- Astro SSR is not part of the baseline.
- SSR requires a product-level ADR naming the freshness requirement, cache strategy, and deployment adapter.
- Use the Astro Cloudflare adapter only when SSR is approved.
- React is allowed only for isolated islands, not for hydrating full marketing pages.
- Do not place dashboard, admin, or authenticated product-app routes in the landing repo.
- Do not introduce backend routes, database code, or dashboard app logic.
- Production dependencies must be pinned to exact versions.
- Do not use npm, yarn, or pnpm.

## Required Script Contract

Landing projects expose this baseline script shape:

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

Baseline build command:

```bash
bun install --frozen-lockfile
bun run build
```

## Project Structure

Use this baseline shape unless an existing project already has an equivalent convention:

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

Blog content lives in the repository as MDX and uses Astro Content Collections.

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
- Use `getCollection("blog")` to read posts.
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

- Tailwind CSS is required.
- Use Tailwind utilities and CSS theme tokens.
- Do not use CSS Modules, styled-components, Emotion, or other CSS-in-JS libraries.
- Shadcn UI is optional and requires user confirmation before adoption.
- Default to Astro components and HTML.
- Add React integration only for real interactivity.
- React islands are allowed for pricing calculators, ROI calculators, interactive demos, newsletter enhancements, lightweight toggles, and client-only embeds.
- Poor React island candidates include static hero sections, feature cards, testimonials, FAQ content that can use HTML `details`/`summary`, blog body, and full-page shells.
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
- Update `.env.example` when environment variables change.

## Security

- Never use unsanitized `set:html` with untrusted content.
- MDX content in repo is trusted only after PR review.
- Sanitize external HTML if rendering it becomes necessary.
- JSON-LD helpers escape `<` as `\u003c`.
- Third-party scripts require explicit review.
- Analytics domains are added intentionally to CSP or Cloudflare headers.
- Do not put secrets, API keys, DSNs, tokens, or private endpoints in source code.
- Do not expose secrets in public environment variables.
- Do not add a service worker by default.

## Testing And CI

Required checks:

- `bun install --frozen-lockfile`.
- `bun run typecheck`.
- `bun run lint`.
- `bun run test`.
- `bun run build`.

Recommended checks:
- Link checking for public pages.
- Accessibility checks for marketing and blog templates.
- Smoke tests for `/`, `/blog`, one `/blog/[slug]`, `/sitemap.xml`, and `/rss.xml`.
- Lighthouse or equivalent checks before launch.

Verify production build before declaring work ready.

## Anti-Patterns

Do not introduce these patterns:

- Landing as React SPA.
- Runtime blog fetch for MDX.
- CMS by default.
- Dashboard routes in landing repo.
- Backend routes in landing repo.
- Database access from landing.
- Hydrating full static pages.
- `client:load` everywhere.
- Unsanitized `set:html`.
- Publishing drafts.
- Service worker by default.
- Fullstack SSR framework by default.
- React islands for static hero sections, testimonials, feature cards, FAQ content, blog body, or full-page shells.
- Secrets, API keys, DSNs, tokens, or private endpoints in source code.

## Implementation Workflow

When modifying or creating landing code:

1. Identify whether the change touches Astro pages, layouts, marketing components, blog content, SEO, sitemap/RSS/robots, islands, forms, Cloudflare deployment, or CI.
2. Check the relevant invariant sections above before editing.
3. Keep pages route-focused; put reusable marketing UI under `src/components/marketing/**`.
4. Use Astro components and HTML by default.
5. Add React only as a targeted island for real interactivity.
6. Use the narrowest client directive for every island.
7. Keep blog content in MDX Content Collections and validate frontmatter.
8. Preserve SEO metadata, canonical URLs, semantic headings, and image alt text.
9. Keep forms accessible and route submissions to the backend API, webhook, or approved provider.
10. Maintain `.env.example` whenever environment variables change.
11. Add or update checks for touched pages, blog routes, RSS, sitemap, links, accessibility, and public form behavior when applicable.
12. Run the smallest relevant verification commands available in the project.
13. In the final response, report which landing gates were verified and which were unavailable.

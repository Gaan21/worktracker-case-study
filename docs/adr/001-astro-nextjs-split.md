# ADR-001: Astro for Landing + Next.js for Admin Panel

## Status

Accepted

## Context

WorkTracker needs two distinct web surfaces:
1. A marketing landing page (static, fast, SEO-optimized)
2. An admin portal (dynamic, authenticated, interactive)

Using a single framework for both introduces trade-offs — either over-engineering the landing or under-serving the portal.

## Decision

- Use **Astro** for the marketing landing page. Astro excels at static content, SEO, and performance with zero client-side JS by default.
- Use **Next.js** for the admin panel. Next.js provides SSR, API routes, and rich client-side interactivity needed for dashboard workflows.

## Consequences

- Two separate codebases to maintain, but each is simpler within its scope.
- Landing page loads fast and ranks well on Core Web Vitals.
- Admin panel has full React ecosystem access for complex UIs.
- Shared types and utilities can be extracted into a shared package if needed.

## References

- [Astro documentation](https://docs.astro.build)
- [Next.js documentation](https://nextjs.org/docs)

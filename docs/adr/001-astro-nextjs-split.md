# ADR-001: Astro for the Public Site, Next.js for the Panel

## Status

Accepted

## Context

Two web surfaces exist with almost nothing in common.

The public site is marketing, pricing, the legal set and public help. It is read by people who
have never logged in and by search engines. It needs to be fast, indexable and mostly static.

The administration panel is authenticated, interactive and data-heavy: review queues, wide
tables, filters, configuration forms and export. It needs a server runtime, because
authorization decisions are made server-side before markup is produced.

## Decision

Astro for the public site. Next.js with the App Router for the panel.

## Consequences

The public site ships close to zero client-side JavaScript by default, which is the right
default for content whose job is to load fast and rank well.

The panel has the full React ecosystem for interactive administrative work, and Server
Components let authorization run before any markup reaches the browser.

Two codebases, two dependency sets, two deployment paths. Each is simpler within its scope, but
shared concerns — design tokens, copy, translation keys — have to be deliberately kept
consistent rather than being consistent by construction.

The two surfaces also deploy to different Cloudflare targets, since static output and a server
runtime are not the same kind of artefact. See [ADR-003](003-cloudflare-deploy.md).

## Trade-offs considered

**Next.js for both** would have unified the toolchain, at the cost of shipping a React runtime
and its build complexity to serve a marketing page. The public site's performance and SEO are a
commercial concern for a product with no sales team.

**Astro for both** was not viable: the panel's authorization model depends on a server runtime
doing real work per request.

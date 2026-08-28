# ADR-003: Cloudflare for Delivery, With Two Different Hosting Targets

## Status

Accepted

## Context

Three deployable web surfaces exist with different runtime needs. The Astro site is static
output. The Next.js panel needs a server runtime for Server Components, route handlers and
server-side authorization. Both need CDN, DNS, TLS and WAF, and the product needs a staging
environment that is not production.

Edge compute for business logic is already settled: it belongs to Supabase Edge Functions, so
that server-side logic lives in one place rather than being split by which vendor happened to
be convenient. See [ADR-002](002-supabase-postgresql.md).

## Decision

Use Cloudflare for delivery across all web surfaces, with the hosting target chosen per
surface:

- **Astro site → Cloudflare Pages.** Static output deployed directly.
- **Next.js panel → Cloudflare Workers, via OpenNext.** The panel is built with the OpenNext
  Cloudflare adapter and deployed as a Worker, with separate staging and production workers
  behind distinct deployment gates.
- **CDN, DNS, TLS and WAF → Cloudflare**, for both.
- **Edge compute for business logic → Supabase**, not Cloudflare Workers.

## Consequences

One vendor for delivery and network-level protection, one vendor for data and server-side
logic, and a clear line between them.

The panel runs on a Workers runtime rather than Node, which constrains available APIs and makes
dependency choices matter more than they would on a conventional Node host. Build and deploy go
through the adapter, so an adapter regression is a deployment risk that a plain Node deployment
would not carry.

Staging and production are separate workers, so a deployment can be validated against real
infrastructure before it reaches users.

Deployment configuration is vendor-specific, but the applications themselves are standard Astro
and Next.js. Moving hosts would be a configuration and pipeline exercise, not a rewrite.

## Trade-offs considered

Hosting the panel on Vercel would have been the path of least resistance for Next.js. It was
rejected to avoid a third vendor in the delivery path and to keep DNS, WAF, CDN and hosting
under one account with one bill and one place to look during an incident.

Running everything as static output was not possible: the panel's authorization decisions
happen in Server Components before markup is produced, which is a deliberate security property
described in [security and multi-tenancy](../security-and-multitenancy.md).

## Note

An earlier version of this record stated that Cloudflare Pages hosted both surfaces. That was
never true of the panel. Corrected in August 2026.

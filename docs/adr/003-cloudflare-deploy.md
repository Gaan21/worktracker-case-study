# ADR-003: Cloudflare for CDN, DNS, WAF, and Deployment

## Status

Accepted

## Context

WorkTracker needs fast global delivery, DDoS protection, and a simple deployment pipeline. Edge computing (serverless functions) is handled by Supabase Edge Functions, not Cloudflare Workers. The deployment pipeline should be simple and cost-effective for an independent SaaS.

## Decision

Use **Cloudflare** for infrastructure, excluding edge computing:
- Cloudflare Pages for hosting and deployment
- Cloudflare CDN for static asset delivery
- Cloudflare as DNS and SSL provider
- Cloudflare WAF for DDoS protection

Edge Functions are handled by **Supabase Edge Functions** (see ADR-002).

## Consequences

- Global CDN network for low-latency static delivery.
- Built-in DDoS protection and WAF.
- Simple deployment via Git integration.
- Cost-effective for an independent product.
- Clear separation: Cloudflare for infra/CDN, Supabase for edge compute.
- Vendor dependency on Cloudflare, but deployment configs are portable.

## References

- [Cloudflare Pages documentation](https://developers.cloudflare.com/pages)
- [Supabase Edge Functions documentation](https://supabase.com/docs/guides/functions)

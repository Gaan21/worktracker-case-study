# ADR-003: Cloudflare for CDN, Edge, and Deployment

## Status

Accepted

## Context

WorkTracker needs fast global delivery, DDoS protection, and edge computing capabilities. The deployment pipeline should be simple and cost-effective for an independent SaaS.

## Decision

Use **Cloudflare** as the infrastructure layer:
- Cloudflare Pages/Workers for hosting and edge computing
- Cloudflare CDN for static asset delivery
- Cloudflare as DNS and SSL provider

## Consequences

- Global edge network for low-latency delivery.
- Built-in DDoS protection and WAF.
- Simple deployment via Git integration.
- Cost-effective for an independent product.
- Vendor dependency on Cloudflare, but deployment configs are portable.

## References

- [Cloudflare Pages documentation](https://developers.cloudflare.com/pages)

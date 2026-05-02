# System Overview

## Architecture Diagram

```
                         ┌─────────────────────────────────┐
                         │         Cloudflare              │
                         │   (CDN + DNS + WAF)             │
                         └──────────────┬──────────────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                    │
          ┌─────────▼──────┐  ┌─────────▼──────┐  ┌─────────▼──────┐
          │  Astro Landing  │  │  Next.js Panel │  │  (Future:      │
          │  (marketing)    │  │  (admin portal) │  │   Flutter App) │
          └────────────────┘  └─────────┬──────┘  └────────────────┘
                                        │
                               ┌────────▼─────────┐
                               │     Supabase      │
                               │   (Auth + Data)   │
                               │   PostgreSQL      │
                               │   Edge Functions  │
                               └──────────────────┘
                                        │
                               ┌────────▼─────────┐
                               │   CI/CD Pipeline  │
                               │   (guardrails)    │
                               └──────────────────┘
```

## Data Flow

1. Users visit the Astro landing for product information
2. Authenticated users access the Next.js admin panel
3. Panel communicates with Supabase (REST + Realtime)
4. Cloudflare handles DNS, CDN, and WAF
5. Supabase Edge Functions handle serverless compute
6. CI/CD pipeline enforces quality gates on every push

## Offline-First (Future)

When Flutter mobile ships:
- Local SQLite as primary data store on device
- Background sync when connectivity returns
- Conflict resolution via last-write-wins with audit trail

# WorkTracker — Architecture Case Study

> An independent SaaS for time tracking, compliance workflows, and auditability for Spanish SMBs.

This repository is a **public architecture case study** for [WorkTracker](https://worktracker.es).
It contains **no private source code** — only design documents, Architecture Decision Records (ADRs), and diagrams.

## Why this repo exists

As an independent builder, I can't open-source the full product. But the architectural decisions behind it — offline-first design, auth strategies, boundary design, CI guardrails — are worth sharing. This repo makes those decisions visible to collaborators, recruiters, and anyone interested in pragmatic architecture for small SaaS products.

## System overview

```
┌─────────────────┐     ┌───────────────────┐     ┌──────────────────┐
│  Astro Landing   │     │  Next.js Web Panel │     │  Supabase        │
│  (marketing)     │     │  (admin portal)    │────▶│  PostgreSQL      │
└─────────────────┘     └───────────────────┘     │  Edge Functions  │
                               │                  └──────────────────┘
                               ▼
                        ┌───────────────────┐
                        │  Cloudflare       │
                        │  (CDN + DNS + WAF)│
                        └───────────────────┘
                               │
                               ▼
                        ┌───────────────────┐
                        │  CI/CD Pipeline   │
                        │  (guardrails)     │
                        └───────────────────┘
```

**Flutter mobile** is part of the broader product ecosystem and is planned for future expansion. It is not currently the primary client for the shipped product.

## Key architecture decisions

| # | Decision | Status | ADR |
|---|----------|--------|-----|
| 1 | Astro for marketing landing + Next.js for admin panel | Accepted | [ADR-001](docs/adr/001-astro-nextjs-split.md) |
| 2 | Supabase/PostgreSQL as primary data layer | Accepted | [ADR-002](docs/adr/002-supabase-postgresql.md) |
| 3 | Cloudflare for CDN, DNS, WAF, and deployment | Accepted | [ADR-003](docs/adr/003-cloudflare-deploy.md) |
| 4 | CI/CD guardrails with versioned guidelines and lint enforcement | Accepted | [ADR-004](docs/adr/004-ci-guardrails.md) |
| 5 | Offline-first as a product principle (Flutter planned) | Accepted | [ADR-005](docs/adr/005-offline-first-principle.md) |

## Current stack

| Layer | Technology |
|-------|-----------|
| **Landing** | Astro |
| **Web Panel** | Next.js, TypeScript |
| **Data** | Supabase, PostgreSQL, Edge Functions |
| **Infra** | Cloudflare (CDN, DNS, WAF, deployment) |
| **Mobile** | Flutter (planned, part of broader ecosystem) |
| **Delivery** | CI/CD pipelines, AI-assisted delivery guardrails, versioned guidelines |

## Directory structure

```
worktracker-case-study/
├── README.md
├── LICENSE
└── docs/
    ├── adr/                    # Architecture Decision Records
    │   ├── 001-astro-nextjs-split.md
    │   ├── 002-supabase-postgresql.md
    │   ├── 003-cloudflare-deploy.md
    │   ├── 004-ci-guardrails.md
    │   └── 005-offline-first-principle.md
    ├── diagrams/               # Architecture diagrams
    │   └── system-overview.md
    └── stack.md                # Full technology breakdown
```

## What you won't find here

- Private source code (Flutter, Next.js, Astro, or any other)
- Business logic or domain models with real data
- Deployment configurations or infrastructure secrets
- Customer data or PII

## Links

- **Product:** [worktracker.es](https://worktracker.es)
- **Author:** [Luciano Plaza](https://linkedin.com/in/luciano-plaza-grueso)
- **GitHub:** [Gaan21](https://github.com/Gaan21)

---

<sub>This is an architecture documentation repository, not a runnable codebase.</sub>

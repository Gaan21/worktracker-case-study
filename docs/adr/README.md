# Architecture Decision Records

Decisions with a real trade-off, recorded with what was given up.

| # | Decision | Status |
|---|---|---|
| [001](001-astro-nextjs-split.md) | Astro for the public site, Next.js for the panel | Accepted |
| [002](002-supabase-postgresql.md) | Supabase and PostgreSQL as the data and auth layer | Accepted |
| [003](003-cloudflare-deploy.md) | Cloudflare for delivery, with two different hosting targets | Accepted |
| [004](004-ai-delivery-guardrails.md) | Guardrails for AI-assisted delivery | Accepted |
| [005](005-offline-first-outbox.md) | Offline-first clocking with an idempotent outbox | Accepted, implemented |
| [006](006-capability-based-authorization.md) | Capability-based authorization instead of role-name checks | Accepted |
| [007](007-panel-scope-admin-only.md) | The panel is scoped to administration | Accepted |
| [008](008-single-time-authority.md) | One cross-surface authority for time | Accepted, partially implemented |
| [009](009-passwordless-no-recovery.md) | Passwordless authentication with no recovery path | Accepted |

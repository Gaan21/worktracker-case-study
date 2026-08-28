# ADR-002: Supabase and PostgreSQL as the Data and Auth Layer

## Status

Accepted

## Context

The product needs a relational database, authentication, server-side compute, and row-level
authorization that holds regardless of which client is asking. It is built and operated by one
engineer, so anything self-hosted is time not spent on the product.

The domain is relational in a way that is not incidental. Companies have employees, employees
have workdays, workdays have ordered movements, movements have audit history, corrections
reference the records they correct. Referential integrity and transactional guarantees are the
requirement, not a preference.

Records are also legal evidence retained for four years, which means constraints that must hold
regardless of application behaviour: append-only history, immutable snapshots, scheduled
retention.

## Decision

Supabase as the data, auth and server-side compute layer, on PostgreSQL.

- **PostgreSQL** for relational data, with Row Level Security scoping rows to the caller's
  company at the database level.
- **Supabase Auth** for identity. Authorization stays in the product, because that is where the
  rules live.
- **`SECURITY DEFINER` database functions** for operations that must validate and audit in one
  transaction.
- **Edge Functions** for server-side operations that need to run before or around the database.
- **`pg_cron`** for scheduled maintenance, including the retention job.
- **Migrations** as the only path to schema change, with pgTAP tests for behaviour that has to
  be proven in the database.

## Consequences

Isolation guarantees live where they cannot be bypassed by a client mistake, and the same
guarantees apply to the mobile app, the panel and any future surface.

Compliance properties — append-only history, per-record immutable snapshots, four-year
retention — are enforced by the schema rather than by convention. That is only possible on a
database that takes constraints and scheduled work seriously.

Both clients call the same database functions for reporting, so figures are computed once
rather than derived twice and drifting.

There is vendor dependency. It is bounded by the fact that the substance is PostgreSQL:
migrations, policies, functions and tests are standard SQL. Auth and Edge Functions are the
genuinely Supabase-shaped parts, and they are the parts that would need work on a move.

Writing meaningful policies and `SECURITY DEFINER` functions requires real PostgreSQL knowledge.
This is not a layer that can be treated as a managed black box, which is why database behaviour
carries its own test suite.

## Trade-offs considered

**A conventional API server** over a managed database was rejected on operational cost for a
single engineer, and because it would make the API the only enforcement point. Row Level
Security holds even when a query is wrong.

**Firebase or another document store** was rejected as a poor fit for a domain that is
relational and audited, and for retention and integrity requirements that are natural in SQL
and awkward outside it.

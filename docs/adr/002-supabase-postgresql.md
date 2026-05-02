# ADR-002: Supabase/PostgreSQL as Primary Data Layer

## Status

Accepted

## Context

WorkTracker needs a reliable, managed data layer with built-in auth, real-time capabilities, and PostgreSQL's relational model for compliance and auditability requirements.

## Decision

Use **Supabase** as the primary data layer:
- PostgreSQL for relational data (time entries, users, compliance records)
- Supabase Auth for authentication
- Supabase client libraries for web and (future) mobile integration

## Consequences

- Managed PostgreSQL with automated backups and RLS (Row Level Security).
- Built-in auth reduces custom auth implementation burden.
- Real-time subscriptions available for live updates.
- Vendor dependency on Supabase, but PostgreSQL remains portable via standard SQL.
- RLS policies enforce data access boundaries at the database level.

## References

- [Supabase documentation](https://supabase.com/docs)

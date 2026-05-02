# ADR-005: Offline-First as a Product Principle

## Status

Accepted

## Context

WorkTracker's users — Spanish SMBs — often work in environments with unreliable connectivity (warehouses, construction sites, field operations). Clock-ins and time tracking must work without the network.

## Decision

Adopt **offline-first as a product principle**:
- All critical user flows must function without network connectivity
- Local data persistence as the source of truth, synced when connectivity returns
- Conflict resolution strategy for concurrent offline edits
- Flutter is the planned mobile client for offline-first native experience

**Current state:** The web panel is the shipped product. Flutter mobile with offline-first capabilities is planned for future expansion.

## Consequences

- Users can clock in and track time regardless of network conditions.
- Sync logic adds implementation complexity but is essential for the target market.
- Data integrity guarantees require careful conflict resolution design.
- Flutter's SQLite support and local persistence make it a strong fit for offline-first mobile.

## References

- [Offline-first architecture patterns](https://developer.android.com/topic/architecture)

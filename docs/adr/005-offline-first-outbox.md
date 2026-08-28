# ADR-005: Offline-First Clocking With an Idempotent Outbox

## Status

Accepted. Implemented and shipped in the mobile app.

## Context

Employees clock in from warehouses, basements, construction sites and vehicles. Connectivity is
unreliable at exactly the moments the product is used.

A clock-in is a labour record that the employer must retain for four years and produce on
request. If registering one depends on a network round trip, then a bad signal produces either
no record or a record with the wrong time, and the employee carries the consequences.

The system therefore has to guarantee two things that pull against each other: the record
exists the moment the user acts, and the same real-world action never becomes two records.

## Decision

Write locally first, synchronize afterwards, and deduplicate rather than resolve conflicts.

A punch is written to local SQLite as an outbox row before any network call, and the user's
confirmation comes from the local commit. Every row carries an idempotency key generated once
when the punch intent is created, held under a local uniqueness constraint, and sent with every
retry. The server honours that key for 72 hours: a repeated key returns the original result
without creating a second movement.

Synchronization runs in order, in batches, with concurrent requests collapsed into the run
already in flight, triggered by connectivity recovery and by app resume. Failures are
classified before they are retried: transient failures back off exponentially with jitter, up
to a bounded number of attempts within a bounded window; non-recoverable failures stop
immediately.

Rows reach one of two terminal states — dead-lettered, or needing manual action — and both are
surfaced to the user with an explicit discard action.

Offline feature availability is declared in a single map, read at three enforcement points: the
shell, the router, and a gate widget.

## Consequences

Clocking is never blocked by connectivity, and retries are safe because an uncertain outcome
can always be repeated.

The local database is a real database with a versioned schema and explicit upgrade paths.
Every schema change must migrate devices holding unsynced records, and a mistake destroys data
that exists nowhere else.

Records uploaded long after the fact carry a large gap between the device timestamp and server
receipt. That is legitimate, so the system flags large gaps for manager review rather than
rejecting them. See [ADR-008](008-single-time-authority.md).

Offline support is a permanent constraint on the product. Every new employee-facing feature has
to answer whether it works offline.

## Trade-offs considered

**Last-write-wins conflict resolution was rejected.** Clock movements are append-only facts,
not documents two devices edit into different states. Applying a last-write-wins policy to
labour records would silently discard evidence, and it addresses a problem this domain does not
have. The real risk is duplication, which deduplication solves directly.

**Carrying the idempotency key as a first-class function parameter was rejected** in favour of
carrying it in the punch metadata. A separate parameter would create a second source of truth
for a value that belongs to the punch, and would force a signature change on a function several
paths call.

**Unbounded retry was rejected.** A system that retries forever hides a stuck record behind an
optimistic indicator. Terminal states are a deliberate admission that automation is finished
and a person has to look.

**Scattered `if (offline)` checks were rejected** in favour of the declarative map. The
scattered version is easier to write for the first screen and worse permanently after, because
the policy stops being inspectable and every new screen can forget it.

## See also

[Offline-first](../offline-first.md) for the full model, states, and user-visible behaviour.

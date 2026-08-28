# ADR-007: The Panel Is Scoped to Administration

## Status

Accepted

## Context

Both the mobile app and the web panel authenticate the same users against the same backend.
The obvious expectation is that an employee can do employee things in either one, including
clocking in from a browser.

The two surfaces have genuinely different strengths. The panel has screen space for wide tables,
filters, review queues and export. The app has the device: location, biometrics, push
notifications, and local storage that works without a network.

## Decision

The panel is for administration. Employees who reach it see their account and the help section,
and are pointed to the mobile app.

Clocking, personal history, incidents, change requests and weekly confirmation exist in the app
only. A server-side guard redirects employees away from administrative routes rather than
hiding menu entries.

Manager and administrator functions exist on both surfaces, at parity, because that work
happens in both contexts: approvals from a phone between sites, reports and configuration at a
desk.

## Consequences

Employees must install the app. That is a real adoption cost, and the panel mitigates it by
surfacing download paths for both platforms, including an explanation of the beta distribution
channels.

The most correctness-sensitive logic in the product — the clocking state machine, the offline
outbox, idempotency — has one implementation rather than two. A browser implementation could
not work offline, so the two versions would differ in exactly the behaviour that matters most.

Manager and administrator features carry an ongoing parity obligation across two clients. This
is managed by computing report figures once in shared database functions and by recording
behaviour in a shared product knowledge base rather than in either codebase.

## Trade-offs considered

**Full parity for employees on both surfaces** was rejected on the grounds above: two
implementations of the rules with legal consequences, one of which is structurally weaker.

**A read-only employee view in the panel**, showing history without allowing clocking, was
considered and deferred. It is coherent, but it splits the employee's mental model of where
their data lives for a benefit that has not been asked for.

**Making the app the only surface** was rejected because administrative work — reviewing a
month of records, exporting for an inspection, configuring a schedule template — is
unreasonable on a phone.

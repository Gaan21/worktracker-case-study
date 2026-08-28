# ADR-006: Capability-Based Authorization Instead of Role-Name Checks

## Status

Accepted

## Context

The product has three roles: administrator, manager and employee. The mobile app originally
carried an enum of user types and made routing and permission decisions by comparing against it.

That approach fails in predictable ways. Renaming a role breaks comparisons scattered across
clients. Adding a fourth role requires finding every comparison. A customer asking for a
manager who cannot see reports has no representation at all, because the role name is the
permission. And with two clients, the same comparison exists twice and can drift.

Permission logic embedded in string comparisons is also invisible to review. A missing check
looks exactly like code that was never written.

## Decision

Grant access by capabilities held against a user's membership, resolved server-side.

Capabilities are discrete and named for what they permit: managing the company, managing roles,
managing security, managing team members, inviting people, reviewing incidents, viewing
reports. They are exposed through a single view that resolves the user's access state and
capability set together.

Route guards, menu visibility and server-side authorization read capabilities. Nothing in
routing compares a string to a role label. The legacy enum was removed from the mobile app.

Roles remain as a product-facing concept, because customers reason about roles rather than
capability sets. A role is a named bundle of capabilities, not a permission in itself.

## Consequences

The manager role is defined by the capabilities it does not hold, expressed once rather than as
a set of exceptions repeated per client.

Both clients resolve permissions from the same source, so the mobile app and the panel cannot
drift into different answers about who may do what.

Changing what a role can do is a data change, not a code change across two clients.

The indirection has a cost: reading the code no longer tells you what an administrator can do,
because that answer lives in data. It has to be documented, which is why role capabilities are
recorded in the product knowledge base rather than inferred from source.

## Trade-offs considered

A single permission bitmask was rejected as harder to read in review and in logs than named
capabilities.

Deriving capabilities from role names at the boundary, keeping role checks internally, was
rejected because it preserves the coupling that caused the problem while adding a translation
layer.

One known weakness is tracked privately in the risk register: part of the capability grant
still relies on matching by role name in one place, which is exactly the pattern this decision
set out to remove. It is registered as an open item rather than described as finished.

## See also

[Security and multi-tenancy](../security-and-multitenancy.md).

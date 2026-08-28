# ADR-008: One Cross-Surface Authority for Time

## Status

Accepted. Partially implemented.

## Context

A production defect produced times that were wrong by one or two hours, inconsistently, across
different screens showing the same record. The root cause was that four surfaces each derived
local time independently, from different inputs, with no artefact stating what the correct
derivation was.

A workday belongs to a calendar date. Which date determines which month it is reported in and
what appears in an export produced for an inspection. This is a correctness property of the
product's primary output, not a formatting concern.

## Decision

One contract, owned outside any implementation repository, that every surface inherits.

Real instants are stored as `timestamptz` in true UTC and never manually shifted. Local display
time and the workday's civil date are projected server-side, never computed by a client.
Company timezone accepts only supported geographic IANA identifiers, enforced by a database
trigger; technical fallbacks such as `UTC`, `GMT`, `Etc/*` and fixed offsets are rejected.
Filter ranges are computed in company time, not UTC and not device time.

Each workday freezes its resolved timezone as an immutable snapshot at creation, and all read
paths use the snapshot rather than the live company setting. A per-user timezone override that
had been an earlier partial answer was removed.

The contract names its anti-patterns explicitly, and the common violations are enforced by
custom lint rules in both the Dart and the TypeScript stacks.

## Consequences

A disagreement between surfaces is now a contradiction against a document rather than an
argument between two plausible implementations.

Historical records carry the interpretation they were created under. A company correcting its
timezone setting does not retroactively change the meaning of four years of records.

There is one more artefact to keep current, and it lives outside the repositories that consume
it. That is the point, and it is also the cost: a contract nobody reads is worse than no
contract, so it is referenced from the task template rather than left to be discovered.

The lint rules carry path scopes and approved-wrapper lists, so they can be adopted
incrementally. That also means code outside their scope is not covered.

## Current limitation

The contract currently assumes all workers in a company share one timezone. That is already
wrong for a company operating in both mainland Spain and the Canary Islands.

The approved target model resolves timezone per work center, with a remote worker modelled as
their own work center. The workday snapshot, the read-path repointing, the work centers table
and the removal of the per-user override have landed. The full resolution model has not. It is
recorded as in-progress.

## See also

[The failure case](../failure-case-timezone.md), which documents the defect and all six changes
that followed.

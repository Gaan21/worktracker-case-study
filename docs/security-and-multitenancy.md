# Security and multi-tenancy

This document describes boundaries and patterns. It does not reproduce policy definitions,
function signatures or schema, and it does not describe known weaknesses. Those are tracked
privately, for reasons explained at the end.

## The isolation problem

Every company on WorkTracker holds records about its employees' working hours, and in some
configurations their location at the moment they clocked in. A leak across two companies is
not a degraded experience. It is a personal data breach involving employment records.

Isolation therefore cannot depend on a query being written correctly. It has to hold even when
the query is wrong.

## Three layers, in order of trust

**PostgreSQL Row Level Security** scopes rows to the caller's company at the database level.
It is declared in migrations, versioned with the schema, and it applies regardless of which
client, function or ad-hoc query is asking. Across 344 migrations there are 159 policy
statements in 40 of them, which counts definitions over time rather than distinct live
policies, since policies get revised.

The recurring patterns are unified access selects that resolve visibility per role in a single
policy rather than one policy per role, self-service insert policies that let an employee write
only their own records, manager-level update policies for review workflows, service-role-only
write access on system tables such as notification logs, and guards that prevent an
administrator from editing identity or company assignment even on records they otherwise
control.

**Server-side operations** handle every write that carries consequences: creating a clock
movement, changing a membership status, resolving an incident, issuing or accepting an
invitation, registering a company. These run in Edge Functions and `SECURITY DEFINER` database
functions that validate the caller, the target, the company relationship, the sequence and the
timing, and write the audit trail in the same transaction as the change.

**Application guards** resolve the caller's state before rendering. In the panel this happens
in Server Components with a server-only module boundary, so an unauthorized route never
reaches the browser as markup that some client check is expected to hide.

The ordering is deliberate. If Row Level Security were the only mechanism, one mistaken policy
would be a breach. If application guards were the only mechanism, any direct database access
would bypass everything.

## Capabilities, not role names

The product has three roles: administrator, manager and employee. Authorization does not check
them.

Access is granted by capabilities held against the user's membership, resolved server-side and
exposed through a single view: whether this user can manage the company, manage roles, manage
security, manage team members, invite people, review incidents, view reports. Route guards and
menu visibility read capabilities. Nothing in routing compares a string to a role label.

The mobile app previously carried a legacy enum of user types and made decisions from it. It
was removed. The reason is that role names are a presentation concern that drifts: a role gets
renamed, a fourth role is introduced, a customer wants a manager who cannot see reports, and
every string comparison scattered through the client becomes a silent authorization bug. A
capability set is data, and it can change without touching a guard.

The practical consequence: the manager role differs from the administrator role in exactly the
capabilities it does not hold. It cannot manage the company, roles or security. That
difference is expressed once.

## Access states

Authentication answers whether someone is who they say they are. It does not answer whether
they should be let in.

A session resolves to one of a small number of states: unauthenticated, onboarding incomplete,
blocked, or active. Blocked and incomplete states route to dedicated walls rather than to a
partially functional panel, so there is no path where a user with an unfinished company reaches
a screen that assumes one exists.

Company-level access is gated by a single provider-agnostic state on the company record. There
is deliberately one gate rather than several overlapping ones, because overlapping gates are
how a user ends up locked out by a condition nobody remembers adding.

## Auditability

Time records are evidence. That imposes properties the schema has to guarantee rather than
conventions the application is expected to follow.

**History is append-only.** The clock movement history table rejects every update and delete
through a trigger. Each monthly partition additionally revokes insert, update, delete and
truncate from the public, anonymous and authenticated roles, granting select only. There is
exactly one exception, the scheduled retention purge, and it runs under a session flag rather
than through any client path.

**Changes are attributable.** Dedicated audit tables record role changes, work-center changes
and membership changes, each capturing the actor, the target, the company, the previous and new
values, and the timestamp.

**Some changes require a stated reason.** Activating or deactivating a team member cannot be
confirmed without entering a non-empty reason. The reason is persisted and shown in a
per-member history that lists every transition, when it happened in company-local time, who
made it, and why. The history is read-only from every client.

This last one is a product decision as much as a security one. The technical work was small.
The reason it exists is that "why was I deactivated on the 14th" is a question that gets asked
months later, when nobody remembers, and the honest answer needs to have been recorded at the
time.

**Corrections are traceable rather than silent.** Employees request changes to their own
records; managers approve or reject them; the resolver is recorded and surfaced in review. A
correction is a reviewable event with a name attached to it, not an edit.

## Authentication

Authentication is passwordless. There is no password to phish, reuse or leak, and consequently
no password recovery flow, which is the most commonly abused path in a small product's auth
surface. One-time code requests are rate limited server-side, with the limiter as its own
enforced boundary rather than a client-side delay.

Access is gated on legal acceptance: a user must acknowledge the applicable terms before
reaching any authenticated route, and the acknowledgement is recorded. Removing or bypassing
that gate is treated as a compliance regression rather than a UX improvement.

Sensitive values are kept out of logs by a lint rule that fails the build when a logging call
carries a token, password, authorization header, cookie, secret or API key, whether as a
metadata key or inside a message string. Log sanitation is a discipline that erodes under
deadline; a rule does not get tired.

## The risk register

Eight security and correctness weaknesses are currently tracked as named entries in the private
product knowledge base. Each records what the weakness is, what it affects, the evidence
establishing it, and its status.

They are not published here, and that is a deliberate decision rather than an omission. They
describe specific, currently unmitigated properties of a system that is live. Publishing them
would be publishing an exploitation path against real companies' employment data. The maturity
signal is that the register exists, is maintained, and distinguishes hardening gaps from
breaches — not that its contents are visible.

They are available to discuss in an interview, in a setting where context can go with them.

## What is deliberately not documented publicly

Policy bodies and the conditions they evaluate. Database function signatures and the arguments
they accept. The table inventory. Edge function internals and validation sequences. The
specific enforcement paths behind each boundary described above.

The patterns are useful to a reader trying to understand how the system reasons about
security. The details are only useful to someone probing it.

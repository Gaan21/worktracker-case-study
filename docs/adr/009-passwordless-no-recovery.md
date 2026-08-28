# ADR-009: Passwordless Authentication With No Recovery Path

## Status

Accepted

## Context

WorkTracker holds employment records. The users are employees of small companies, on shared and
sometimes older devices, who use the product for a few seconds a day and will not tolerate
friction at the point of clocking in.

Password authentication brings a predictable set of problems at this scale: reuse across
services, phishing, and a recovery flow that is the most commonly abused part of a small
product's auth surface. A password reset is, by construction, a way to take over an account
using only an email address.

## Decision

Authenticate with one-time codes delivered by email. No password is ever set, so there is
nothing to recover.

Code requests are rate limited server-side, as an enforced boundary rather than a client-side
delay. The mobile app adds a biometric lock over the authenticated session, so a shared or
unattended device does not expose records to whoever picks it up.

Access is additionally gated on recorded acceptance of the applicable legal terms before any
authenticated route is reachable.

## Consequences

There is no password database, no reset flow, and no credential-stuffing surface.

Authentication depends on email delivery. If a user cannot receive mail, they cannot log in,
and the mitigations are operational rather than technical. Delivery is therefore treated as
part of the auth path rather than as a notification concern.

Rate limiting is a genuine availability trade-off: a limit tight enough to stop abuse can lock
out a legitimate user who is retrying, which is registered as a known risk rather than assumed
away.

Losing access to the email account means losing access, with recovery handled by a company
administrator through membership management rather than by a self-service flow. For a B2B
product where every user belongs to a company that can vouch for them, that is the right place
for the decision to sit. It would not be for a consumer product.

## Trade-offs considered

**Passwords with a reset flow** were rejected: it reintroduces the phishing and reuse surface
and adds the recovery path as an attack vector.

**Social login** was rejected as a dependency on a consumer identity provider for a workforce
product, where employees may not have or want to use a personal account for work.

**Magic links instead of codes** were considered. Codes were chosen because they work when the
link opens in a different browser than the one the user started in, which on shared and
mobile-managed devices is common.

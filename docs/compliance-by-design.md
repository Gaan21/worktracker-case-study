# Compliance by design

WorkTracker is built for Spanish companies, so it operates inside a body of working-time
recordkeeping law and under the GDPR. This document is about an engineering problem: how to
turn legal obligations into requirements a schema and a product can actually satisfy.

It is not legal advice, and nothing here claims that the product is compliant or certified.
Compliance is a property of an employer's practice. What a product can do is make the
employer's obligations easier to meet and harder to accidentally break.

## Which duties actually reach the product

Spanish working-time recordkeeping sits across several sources, and they do not impose the
same thing.

The general baseline requires a daily record of start and end times, retained for four years,
available to the worker, their representatives and the Labour Inspectorate. Separate provisions
cover part-time work, where hours are recorded daily and totalled monthly with a copy delivered
alongside the payslip, and overtime, which carries its own daily recording and period
totalisation duty. A further provision requires the employer to inform worker representatives
monthly about overtime performed.

Beyond that, a draft Royal Decree developing the recordkeeping regime has been through public
hearing but is not enacted.

The first engineering task was separating these, because they have different consequences. A
retention duty is a schema property. A monthly delivery duty is a reporting feature that only
some companies need. A duty that lands on the employer's process rather than on their tooling
may correctly produce no product work at all. And a draft is not a requirement yet.

## Rules for reasoning about regulation

The internal analysis carries an explicit evidence taxonomy, and every requirement statement is
tagged with one:

- **Current official source.** Enacted law.
- **Official draft.** Published draft text. Anything derived from it is recorded as assumed,
  never as confirmed, however likely enactment looks.
- **Verified against the product.** Checked on a stated date against migrations, source, tests
  or the database catalogue, with the date recorded so the claim expires visibly.
- **Interpretation.** A conclusion the source text does not settle unambiguously.
- **Decision required.** A product or policy choice that no amount of evidence will resolve,
  which needs a person to decide.

Two rules govern how those become work.

**A draft is never recorded as an enacted requirement.** It is easy, and tempting, to build
against a draft and describe the result as compliant. That produces a product making a claim
that is false until the day it happens to become true, and possibly never, since drafts change
in the hearing process. The register keeps the distinction so that a future reader can tell
which requirements were real when the code was written.

**An employer duty is not a product defect unless the product undertakes to provide that
capability.** Without this rule, every obligation that touches working time becomes a gap in a
backlog, and the backlog stops describing the product. With it, each duty gets an explicit
decision: we cover this, we deliberately do not, or someone needs to decide.

The most recent revision of that analysis produced concrete outcomes rather than a document: it
corrected business rules that had been recorded too confidently, added entries to the risk
register where the product's behaviour did not match what a requirement implied, and left
several items marked as needing a decision instead of resolving them by guesswork.

## Where the constraints show up in the system

**Four-year retention is enforced by the database, not by a policy document.** A scheduled job
runs monthly, creates upcoming partitions of the audit history and drops those outside the
four-year window. The retention period is a parameter with a 48-month default rather than a
hardcoded constant, so a different jurisdiction or a changed duty is a configuration change.

Workdays still referenced by an open change request are preserved past the cutoff, because
deleting the record a dispute is about would destroy the evidence the dispute needs.

**Availability of records drove the export work.** A duty to produce records on request means
export is a first-class feature, not a convenience. Reports export to PDF and CSV with the same
fields and layout on both the mobile app and the panel, including an annexe of rectifications,
so a company can hand over a coherent document regardless of which surface produced it.

**Attributable corrections drove the incident and change-request model.** A record that can be
edited without trace is worth less as evidence than one that cannot be edited at all. Employees
request corrections, managers resolve them, and the resolver is recorded and surfaced.

**Append-only history drove the schema.** The movement history rejects updates and deletes at
the database level, with partitions granting select only. This is described in
[security and multi-tenancy](security-and-multitenancy.md).

**Time correctness is a compliance property, not a display detail.** A workday belongs to a
calendar date, and which date it belongs to determines which month it is reported in. Getting
that wrong is a wrong record, not a cosmetic bug. That constraint is what makes the timezone
work in [the failure case](failure-case-timezone.md) a compliance issue rather than a UI one.

## Data protection

The GDPR position is that the customer company is the controller of its employees' working-time
data and WorkTracker is the processor. The public legal set on the site sets out that framing,
the categories of data processed, the retention constraints and the deletion path.

Two product decisions follow from privacy by design rather than from a checklist.

**Geolocation is off by default and binary.** A company either does not capture location, which
is the default, or requires it on every punch. There is no partial or per-employee mode. That
looks less flexible and is deliberately so: intermediate modes produce a system where neither
the employee nor the employer can state plainly whether location is being recorded, and consent
and transparency both depend on being able to state it plainly.

**Sensitive values never reach logs.** Enforced by a lint rule rather than a convention, as
described in the security document.

## What is not documented here

The gap analysis between the product's current behaviour and the draft Royal Decree names
specific places where they do not align. That analysis is not published, for the same reason
the risk register is not: it is a precise list of where a live system falls short of a rule
that may become binding, and it belongs in a conversation with context rather than on a public
page.

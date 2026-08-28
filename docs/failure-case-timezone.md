# Failure case: the UTC display regression

This is a real defect from this codebase, with the changes that followed it. It is included
because how a system responds to a failure says more than the failure does.

## The symptom

The app displayed times that were wrong by an hour, or by two, depending on the season.

Screens showed a workday's clock-in at 08:00 in one place and 06:00 in another. The incident
review flow was where it became undeniable: a manager reviewing a correction request saw one
time in the list and a different time in the detail view of the same record.

## Root cause

There was no single owner of the question "what time is it, for this company, for this record".

The database stored real instants correctly, as UTC. Everything downstream re-derived local
time independently, and the derivations disagreed:

- Some UI code formatted a UTC instant directly, rendering server time and labelling it as the
  employee's time.
- Some code used the **device** timezone, which is right for a phone in Madrid, wrong for a
  phone in the Canary Islands, and wrong for anyone whose phone is set to something else.
- Some paths applied a timezone conversion to a value that had already been converted server
  side, shifting it twice.
- Filter ranges — this week, this month, today — were computed in a mix of UTC and local time,
  so a workday could fall inside a report on one surface and outside it on another.

Each of these is individually defensible in the file it lives in. Collectively they are a
system with no answer to the question.

The deeper problem was that the rule existed only as knowledge in my head. There was a
convention. There was no artefact that stated it, nothing that enforced it, and no place a new
piece of code was obliged to consult.

## Why this was not a display bug

A workday belongs to a calendar date, and which date it belongs to determines which month it
is reported in, which week's hours it counts toward, and what appears in an export produced for
an inspection.

A one-hour display error at 23:30 is a workday recorded against the wrong day. The record is
wrong, not the rendering. That reclassified the defect from cosmetic to a correctness issue in
the product's primary output, and it justified the size of the response.

## What changed

Six changes, across three layers. The fix itself was the smallest of them.

**1. The rule became an artefact, in a repository that outlives the app.**

A cross-surface time contract was written and moved out of the mobile repository into the
product knowledge base, so that the mobile app, the panel, the database and any future surface
inherit it instead of each deriving its own answer.

It specifies the SQL type for each category of value: real instants as `timestamptz`, always
true UTC, never manually shifted; local display time derived server-side; the workday's civil
date projected server-side and never computed in a client. It defines the rule for each shape
of database function — one that takes a civil date, one that takes an instant, one that returns
an already-projected local time — so a client can tell from the signature what it is obliged to
do.

Most usefully, it names the anti-patterns explicitly: using the device timezone for display or
filters, re-shifting a value that already carries a real instant, treating a picker's local
value as UTC, computing filter ranges in UTC, and mixing endpoints where one returns real UTC
and another returns a shifted value. Naming a mistake is what lets a reviewer point at it
without re-deriving the argument.

**2. The rule became enforceable at author time.**

Three custom Dart lint rules were added:

- One detects time formatting applied to a value that is not wrapped by an approved
  company-time helper. Its changelog entry names this exact regression as the reason it exists.
  It ships with fourteen tests covering direct calls, named constructors, date-only patterns,
  approved wrappers, and configuration.
- One detects mixing a civil date with a UTC instant without going through an approved helper.
- One detects device-local time reaching the presentation layer.

All three take configurable path globs and approved-wrapper lists, so they can be adopted
incrementally instead of failing an entire existing codebase on day one, which is the usual
reason a good rule never gets turned on.

An equivalent rule was added to the web stack's ESLint plugin, because the same class of
mistake does not care what language it happens in.

**3. The company timezone became a validated value.**

Only supported geographic IANA identifiers are accepted, enforced by a database trigger.
Technical fallbacks such as `UTC`, `GMT`, `Etc/*` and fixed offsets are rejected as company
timezones. They are the values that make a bug look like a configuration choice.

**4. Each workday freezes its own timezone.**

A migration added an immutable timezone snapshot to the workday record, populated at creation
and backfilled for existing rows, with a defensive trigger that fills it from the company if an
internal path inserts a workday without one.

This is the change that closes the class rather than the instance. Without it, a company that
corrects its timezone setting retroactively rewrites the meaning of every historical record.
With it, a record carries the interpretation it was created under, permanently. For records
kept as legal evidence for four years, that difference is the whole point.

**5. Every read path was repointed to the snapshot.**

A second migration moved all the reporting and history functions — clock history, workday
detail, user state, incidents, change requests — onto the frozen value rather than the live
company setting. A per-user timezone override that had been an earlier partial answer was
dropped, after checking for live dependencies, because two mechanisms for the same question is
how the original bug happened.

**6. The device clock became evidence rather than truth.**

A further migration records the server transaction time for each punch and stores the signed
difference against the client's timestamp, flagging outliers for review without rejecting them.
This is described in [offline-first](offline-first.md). It belongs to this story because the
audit made explicit that the client's notion of time is an input to be attested, not a fact to
be trusted.

Each schema change carries pgTAP coverage, so the guarantees are asserted in the database
rather than in a client that happens to behave.

## Why recurrence is less likely

The class of defect is now checked at author time in two languages, so occurrences within the
rules' scope are blocked before they reach review.

The invariant lives in one file with one owner, so a disagreement between surfaces is a
contradiction against a document rather than an argument between two plausible implementations.

The historical interpretation is frozen in the data, so the same bug occurring again would be a
present-tense display error rather than a retroactive corruption of four years of records.

The anti-patterns are named, so review can point at a rule instead of re-litigating.

I would not claim it cannot happen again. Lint rules have path scopes and approved-wrapper
lists, and something outside them will eventually be written. The claim is narrower: the
cheapest path is now the correct one, and the expensive path is visible.

## What is still open

The contract currently assumes all workers in a company share one timezone. That assumption is
already wrong for a company operating in both mainland Spain and the Canary Islands.

The approved target model resolves timezone per work center, with a remote worker modelled as
their own work center. It is partially landed: the workday snapshot, the read-path repointing,
the work centers table and the removal of the per-user override are done. The full resolution
model is not.

It is documented as in-progress rather than described as finished, which is the same discipline
the rest of this repository tries to hold to.

## What I took from it

The fix was the least valuable output. Formatting a few dates correctly took an afternoon.

What was worth the time was noticing that the rule had never existed anywhere outside my head,
and that a rule which lives only there scales to exactly one person paying attention. Writing
it down as a contract, then making it enforceable, then making the data carry its own
interpretation, is what turned one afternoon's fix into a class of bug the system now resists.

The general form: when something breaks, fixing it is table stakes. The question worth asking
is what property of the environment allowed it, and that question usually has an answer that
can be encoded.

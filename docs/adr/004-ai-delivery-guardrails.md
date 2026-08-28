# ADR-004: Guardrails for AI-Assisted Delivery

## Status

Accepted

## Context

AI agents write a large share of the code. A single engineer cannot review generated code at
the rate it can be generated, so review by attention does not scale and is the wrong control to
rely on.

The specific failure modes are known and repeatable. An agent violates an architectural rule
established in a document it did not read. It suppresses a lint error rather than satisfying it,
and the suppression is invisible in a large diff. It expands scope past the task while
refactoring something it passed on the way. It leaves documentation behind while changing
behaviour.

## Decision

Constrain agents with artefacts rather than instructions, at four levels.

**Versioned guidelines**, distributed as a git submodule so each repository pins a known
revision, authored once and generated into per-runtime instruction files. Separately, a product
knowledge base owns what the product does and why, and is authoritative over technical
guidelines when the two disagree about behaviour.

**Scoped tasks** as files with a frontmatter contract: `files_scope`, `non_goals`, `must_read`
for product context, `preload` naming concrete guideline files, acceptance criteria and a
definition of done. Task state derives from folder location, never from a status field.

**Runtime guards** as pre-tool hooks that inspect an intended action and can refuse it. Lint
suppressions without a traceable approval tag are blocked before the file is written. Write git
operations are blocked entirely, with commits reachable only through audited scripts and no
environment-variable bypass. Both policies are implemented once per agent runtime from a single
stated policy.

**Automated gates**: custom lint rules encoding project-specific defect classes, zero tolerated
warnings, type checks, unit tests, browser tests, database tests, and internationalization
consistency checks, with deployment behind a separate validation step.

Every task closes with an explicit statement of whether product knowledge was updated, needed
no update, or needs review.

## Consequences

Generated code is held to the same standard as hand-written code, and the standard is
enforceable rather than aspirational.

Each rule is institutional memory. A defect class that produced a rule does not reach review a
second time.

The cost is real: writing a task file is slower than describing a change, guidelines and the
knowledge base need maintenance that ships no features, custom lint rules need their own tests
and will produce false positives requiring carve-outs, and runtime guards occasionally block
something legitimate.

## Trade-offs considered

Relying on review alone was rejected because it fails exactly when volume is high, which is
when agents are most useful.

Documentation-only guidance was rejected because an agent that has read a rule can still break
it. Rules that matter are enforced where the action happens.

Blocking agents from git entirely, rather than allowing careful use, was chosen because
partial restrictions on destructive operations are hard to specify correctly and easy to work
around. A small set of audited scripts is a smaller surface to reason about.

## See also

[The full workflow](../ai-engineering-workflow.md), including three real failures and the
changes each produced.

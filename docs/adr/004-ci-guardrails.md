# ADR-004: CI/CD Guardrails with Versioned Guidelines and Lint Enforcement

## Status

Accepted

## Context

As a solo builder using AI-assisted delivery, I need automated quality enforcement that doesn't depend on manual review alone. CI guardrails ensure that every change — whether human or AI-generated — meets the same standards.

## Decision

Implement **CI/CD guardrails** as the primary quality enforcement mechanism:
- Versioned guidelines in the repository (not just documentation — enforced rules)
- Custom lint rules that catch project-specific anti-patterns
- CI pipeline that fails on lint violations, type errors, or guideline breaches
- Structured workflow for AI-assisted delivery: prompts → code → guardrails → review → merge

## Consequences

- Quality gates are automated and non-negotiable.
- AI-generated code is held to the same standard as human-written code.
- Guidelines are versioned alongside code, preventing drift.
- Onboarding is faster — the CI teaches the rules.

## References

- [Effective CI guardrails for AI-assisted delivery](https://github.com/Gaan21/setup-codex-per-repo) — related DevEx tooling

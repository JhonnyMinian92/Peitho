# Architecture overview

Describe the system boundaries, components, dependencies and integration points.

# Reused Architecture from Central Memory

## architectures/hexagonal.md

# Hexagonal Architecture

## Intent
Protect domain and application logic from infrastructure and external interfaces through ports and adapters.

## Applicability
Use when a system has multiple integrations, requires strong testability, or must preserve domain independence.

## Non-applicability
Avoid for disposable prototypes or trivial components where interfaces would add more complexity than value.

## Structure
Define ports for required/provided capabilities and adapters for HTTP, persistence, messaging and other external mechanisms.

## Implementation guidance
Keep domain code unaware of concrete adapters. Select adapters through composition/configuration rather than embedding infrastructure decisions in business logic.

## Validation
Verify dependency direction, replaceability of adapters, isolated domain tests and explicit integration boundaries.

## Security
Treat adapters as trust boundaries; validate input, enforce authorization and avoid leaking infrastructure errors into public contracts.

## Trade-offs
Strong boundaries and testability at the cost of more interfaces and indirection.

## Related
Skill: `architecture.system_design`. Often complements Repository and Adapter patterns.

## architectures/layered.md

# Layered Architecture

## Intent
Separate presentation, application, domain and infrastructure responsibilities behind explicit boundaries.

## Applicability
Use when responsibilities are distinct and the system has moderate complexity or multiple contributors.

## Non-applicability
Avoid adding layers solely for ceremony in very small components or when extra boundaries create unacceptable latency/complexity.

## Structure
Presentation handles delivery concerns; application coordinates use cases; domain owns business rules; infrastructure implements external concerns.

## Implementation guidance
Keep dependencies directed toward stable inner responsibilities. Do not let controllers contain business rules or infrastructure details leak into the domain.

## Validation
Check dependency direction, testability of domain logic, clear ownership and absence of duplicated cross-layer rules.

## Security
Validate untrusted input at boundaries and keep authentication/authorization decisions explicit in application boundaries.

## Trade-offs
Simple mental model and separation, but excessive layering can create indirection and accidental coupling.

## Related
Skill: `architecture.system_design`. Consider `architecture.hexagonal` when external adapters become numerous or domain isolation is critical.

## architectures/adr.md

# Architecture Decision Record

## Intent
Preserve the context, decision and consequences behind an important architectural choice.

## Applicability
Use when alternatives exist, the decision has durable consequences, or future agents need the rationale.

## Non-applicability
Do not create an ADR for routine implementation details with no meaningful architectural consequence.

## Structure
Record context, constraints, alternatives considered, decision, consequences, risks and status.

## Implementation guidance
Reference affected contracts, components and constraints. Keep the ADR immutable in intent; supersede it with a new decision when the architecture changes.

## Validation
A reviewer should be able to understand why the decision was made without reconstructing the original discussion.

## Security
Record relevant security implications and rejected alternatives when security is a material decision factor.

## Trade-offs
Preserves rationale and reduces repeated debate, but requires maintenance and disciplined updates.

## Related
Skill: `architecture.system_design`. Project SSOT for architectural decisions is `architecture/decisions/`.

Obsoleto: usar content en su lugar.


# Project-specific architecture decisions

These accepted ADRs override generic examples when a conflict exists:
- [ADR-0001 — Dedicated logical replication from Inventary](architecture/decisions/ADR-0001-inventary-logical-replication.md): dedicated logical replication to Peitho PostgreSQL and a read-only API for point-in-time validation.
- [ADR-0002 — Local authentication first](architecture/decisions/ADR-0002-autenticacion-local.md): local accounts, Argon2id, TOTP/passkeys, secure sessions and audit; no initial external federation.
- [ADR-0003 — Self-hosted AI gateway](architecture/decisions/ADR-0003-pasarela-ia-autogestionada.md): vLLM for language models and Python/Sentence Transformers for embeddings; Ollama is development-only.

The machine-readable summary is [decisions.json](architecture/decisions/decisions.json). These records describe intended architecture, not proof of a production deployment.

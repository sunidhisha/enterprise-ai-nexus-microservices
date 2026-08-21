# ADR-002: Manufacturing Core as a Modular Monolith

## Status
Accepted

## Context

Manufacturing functionality in Version 1 spans product definition, production planning, execution, quality, genealogy, and asset/maintenance concerns. These capabilities are closely coupled and should not be prematurely split into independently deployed services.

## Decision

Manufacturing Core will remain a single deployable modular monolith in Version 1, with internal module boundaries but no premature split into independent networked services.

Manufacturing Core's internal modules retain explicit business ownership, domain terminology, invariants, and internal boundaries.

Co-location within one deployable does not permit unrestricted sharing of domain models or authoritative state, and one module must not bypass another module's invariants or ownership.

Future extraction may be considered when there is demonstrated need for different:
- scaling characteristics
- business/operational ownership
- security boundaries
- availability requirements
- runtime requirements
- deployment lifecycle
- failure-isolation requirements

Future extraction is evidence-driven rather than automatic, and it does not imply that every bounded context will eventually become a microservice.

## Alternatives Considered

- Split Manufacturing Core into multiple services immediately.
- Keep all manufacturing logic in a single unstructured application.
- Treat business modules as autonomous runtime services from the start.

## Rationale

The approved architecture favors a focused Phase 0 model with low operational complexity and strong transactional coherence. A modular monolith preserves business cohesion while still enabling internal ownership boundaries.

## Consequences

### Positive

- Lower operational and deployment complexity.
- Clear internal module boundaries without unnecessary distribution.
- Better fit for the tightly coupled manufacturing workflows in V1.

### Negative / Trade-offs

- The system is not yet fully decomposed for maximum scale.
- Future growth may require additional service boundaries.

## Constraints / Guardrails

- Internal modules remain logical boundaries, not independent runtime services.
- Cross-module communication remains within the same deployable service.
- The modular monolith does not change authoritative ownership elsewhere in the platform.
- Manufacturing Core's internal modules retain explicit business ownership, domain terminology, invariants, and internal boundaries.
- Co-location within one deployable does not permit unrestricted sharing of domain models or authoritative state, and one module must not bypass another module's invariants or ownership.
- Future extraction is evidence-driven rather than automatic and must be justified by demonstrated need for different scaling characteristics, business/operational ownership, security boundaries, availability requirements, runtime requirements, deployment lifecycle, or failure-isolation requirements.
- Future extraction does not imply that every bounded context will eventually become a microservice.

## Future Reconsideration Triggers

- The modular monolith becomes a scaling or release bottleneck.
- Different modules require independent operational ownership.
- A significant part of the manufacturing domain becomes functionally independent and separately governed.

## Related Architecture

- Microservices Design
- Domain Context Design
- Data Ownership
- Integration Design
- Platform Architecture

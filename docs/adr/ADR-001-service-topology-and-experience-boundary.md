# ADR-001: Version 1 Service Topology and Experience Boundary

## Status
Accepted

## Context

The Phase 0 architecture must support a single-facility manufacturing platform with clear business ownership, low operational overhead, and a clean experience boundary. The design must keep authoritative business logic within the owning business/domain boundaries while allowing a separate composition layer for operational user experiences.

## Decision

Version 1 business services:
- Manufacturing Core
- Inventory Service
- Industrial Operations Service

Supporting independently deployable components:
- Operational Analytics
- AI Decision Support

Experience layer:
- Angular
- GraphQL BFF

The GraphQL BFF provides composition and presentation only and owns no authoritative domain state or business rules.

## Alternatives Considered

- Keep manufacturing behavior in a single monolith without service boundaries.
- Split all manufacturing functions into independent networked services immediately.
- Put business responsibility into the GraphQL BFF.

## Rationale

The approved design separates authoritative business domains from the experience layer. This preserves ownership, keeps service boundaries understandable, and avoids premature distribution of tightly coupled manufacturing workflows.

## Consequences

### Positive

- Clear ownership boundaries for business responsibility.
- A simpler Phase 0 runtime model.
- Better separation between user experience and authoritative state.

### Negative / Trade-offs

- Some cross-service coordination is required.
- The platform is intentionally simpler than a fully distributed future-state design.

## Constraints / Guardrails

- The BFF owns composition and experience concerns only.
- The BFF does not own persistent domain state.
- Authoritative business decisions remain in domain-owned services.

## Future Reconsideration Triggers

- New business flows require materially different service boundaries.
- Performance or scale constraints exceed the approved V1 model.
- The platform shows repeated cross-service coupling that justifies new service decomposition.

## Related Architecture

- Product Vision
- Solution Architecture
- Microservices Design
- Integration Design
- Platform Architecture

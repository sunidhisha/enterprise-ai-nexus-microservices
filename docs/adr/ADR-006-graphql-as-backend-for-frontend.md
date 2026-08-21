# ADR-006: GraphQL as Backend-for-Frontend

## Status
Accepted

## Context

The user experience layer requires efficient composition across multiple business domains, while the architecture must preserve separation between user experience and authoritative business logic.

## Decision

GraphQL will be the Version 1 backend-for-frontend composition layer, used to aggregate experience-oriented data without owning persistent domain state or authoritative business logic.

## Alternatives Considered

- Use a REST BFF for screen aggregation.
- Put business orchestration directly in the UI.
- Let the BFF own authoritative state or domain logic.

## Rationale

The approved design treats GraphQL as an experience layer. It is well suited to screen-level composition and data shaping without becoming the owner of manufacturing decisions or persistent domain state.

## Consequences

### Positive

- Better experience composition across multiple services.
- Clear separation between UI shaping and business authority.
- Simpler front-end data assembly for operational workflows.

### Negative / Trade-offs

- Additional composition logic is required in the BFF.
- The BFF must remain disciplined to avoid becoming a hidden authority layer.

## Constraints / Guardrails

- GraphQL BFF is primarily an experience-oriented composition layer for the Angular operational application.
- It may perform screen-oriented composition and read aggregation.
- It must not own authoritative manufacturing state.
- It must not become the owner of domain invariants or manufacturing business rules.
- Authoritative business validation remains within the owning business/domain boundary.
- Business authorization decisions remain with the authoritative domain boundary where the protected business action is performed.
- The BFF must not bypass Manufacturing Core, Inventory Service, Industrial Operations Service, or their ownership boundaries.
- GraphQL is not the universal service-to-service communication mechanism.
- REST/synchronous service communication remains a separate architectural concern as already established in the approved architecture.
- Cross-domain composition in the BFF must not turn the BFF into an authoritative workflow or transaction coordinator.

## Future Reconsideration Triggers

- Experience needs require a different aggregation model.
- The BFF begins to absorb domain logic beyond composition.
- A different experience technology becomes the better fit for the product.

## Related Architecture

- Solution Architecture
- Platform Architecture
- Integration Design
- Security Architecture

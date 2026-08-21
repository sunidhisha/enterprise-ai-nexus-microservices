# ADR-007: Keycloak as Version 1 Identity Provider and IAM Boundary

## Status
Accepted

## Context

The platform needs a centralized identity model for human users and service interactions without folding identity concerns into manufacturing domain logic. Authentication, authorization, and business validation must remain distinct concerns.

## Decision

Keycloak will provide the Version 1 identity provider and IAM boundary, while business authorization and business validation remain owned by the domain services that hold the authoritative decision.

## Alternatives Considered

- Embed identity and authorization logic in each domain service.
- Use a different IAM platform without clear domain boundaries.
- Allow business services to treat identity as a direct business authority.

## Rationale

The approved security model separates identity from business authority. Keycloak is the identity authority for token, session, and identity context, but it is not the authority for production, inventory, quality, or AI decisions.

## Consequences

### Positive

- Clear separation between identity and domain responsibility.
- Consistent authentication and session model across services.
- Better alignment with enterprise IAM governance.

### Negative / Trade-offs

- Identity governance requires coordination across platform and domain teams.
- Domain services must still enforce business authorization and validation.

## Constraints / Guardrails

- Keycloak owns authentication and identity context, not manufacturing business authority.
- Authentication, authorization, and business validation are distinct concerns.
- Authoritative domain services make the final authorization and business-validation decisions for protected business actions.
- Keycloak must not determine production-order validity, inventory availability decisions, formal Quality Hold decisions, maintenance decisions, or AI recommendation authority.
- The GraphQL BFF may enforce experience-level access controls, but it must not become the sole authorization authority for authoritative domain operations.
- Human identities, service identities, and machine/device identities must remain conceptually distinct.
- Facility/plant scope and business role may participate in authorization decisions, but exact authorization policies remain detailed design.
- Keycloak unavailability must never cause authentication or authorization to be silently bypassed.
- The effect of Keycloak unavailability on already authenticated requests depends on the later token-validation design.

## Future Reconsideration Triggers

- A different enterprise identity model becomes required.
- The platform expands to a significantly different IAM architecture.
- Domain authorization responsibilities become mismatched with the current identity boundary.

## Related Architecture

- Security Architecture
- Platform Architecture
- Solution Architecture
- Identity and access governance design

# ADR-012: Dependency Criticality and Health/Readiness Semantics

## Status
Accepted

## Context

The platform depends on infrastructure and service components with different operational impact. Health signals and readiness semantics must be explicit so the platform can distinguish critical dependencies from optional ones and degrade gracefully without misrepresenting system state.

## Decision

Dependencies will be classified by operational criticality in the context of the consuming component’s business responsibility and workflow. The platform will use clear health and readiness semantics that distinguish critical dependencies, optional dependencies, and degraded capability without treating every dependency as globally critical or every partial failure as total unavailability.

## Alternatives Considered

- Treat every dependency as equally critical.
- Use a single binary health model without dependency nuance.
- Ignore dependency criticality in runtime semantics.

## Rationale

The architecture requires clear operational semantics. Some dependencies are critical to core business readiness; others are optional or degraded. This distinction is essential for correct platform behavior, resilience, and operational clarity.

## Consequences

### Positive

- Clear operational understanding of dependency impact.
- Better degrade-and-recover behavior in partial outages.
- Stronger platform readiness semantics and incident response.

### Negative / Trade-offs

- Dependency taxonomy adds operational rules and design discipline.
- Narrow or partial failures need explicit handling and reporting.

## Constraints / Guardrails

- Dependency criticality is contextual to the business responsibility and workflow of the consuming component. A dependency is not automatically critical everywhere.
- Critical dependency: the consuming component cannot perform its required business responsibility without the dependency.
- Optional dependency: the consuming component can continue its core business responsibility without the dependency.
- Degraded capability: some functionality is unavailable or reduced, but the component or platform must not automatically be treated as completely unavailable.
- Health and readiness signals must represent actual capability state rather than simply reporting whether every dependency is reachable.
- Failure of an optional dependency must not automatically make an otherwise functional service unready.
- AI Decision Support unavailable: recommendations and AI findings may be unavailable, but core manufacturing continues where AI is optional.
- Operational Analytics unavailable: reporting and analytical capabilities degrade, while authoritative manufacturing transactions continue.
- Industrial connectivity degraded: affected operational capabilities report degradation; unrelated platform capabilities do not automatically become unavailable.
- Keycloak unavailable: new authentication, token acquisition, or session refresh may fail. Authentication and authorization must never be silently bypassed. The effect on already-authenticated API requests depends on the later token-validation design.
- This ADR does not define health endpoint paths, readiness endpoint paths, Kubernetes probes, timeout values, retry counts, circuit-breaker configuration, monitoring products, or implementation classes.

## Future Reconsideration Triggers

- Dependency semantics become inconsistent with the platform’s operational model.
- New service patterns require a different classification model.
- Stronger resilience or cross-service dependency contracts are required by design.
- A dependency’s criticality needs to be reinterpreted because the consuming component’s business responsibility changes materially.

## Related Architecture

- Platform Architecture
- Security Architecture
- Integration Design
- Operational resilience and monitoring design

# ADR-004: PostgreSQL as Version 1 Persistence Platform

## Status
Accepted

## Context

The Phase 0 platform needs a reliable relational persistence platform for authoritative transactional data. The decision must be explicit about the difference between physical database infrastructure and logical ownership.

## Decision

PostgreSQL will be the Version 1 relational persistence platform, while physical database infrastructure remains separate from logical and domain ownership and shared infrastructure will not permit cross-service table access.

## Alternatives Considered

- Use a different relational database platform.
- Treat a shared persistence layer as a shared logical domain store.
- Allow cross-service access to each other’s tables for convenience.

## Rationale

PostgreSQL is the approved V1 relational persistence platform for the platform’s authoritative transactional needs. It provides the physical store, but it does not change who owns the business meaning or runtime truth of a fact. Shared infrastructure can be operationally convenient without implying shared ownership or shared domain tables.

## Consequences

### Positive

- Clear relational fit for Version 1 transactional services.
- Operational convenience through shared infrastructure without logical coupling.
- A stable foundation for MVC-style domain services and service-owned schemas.

### Negative / Trade-offs

- Shared infrastructure still requires strong ownership discipline.
- Exact database and schema topology remains a detailed design concern.

## Constraints / Guardrails

- PostgreSQL is the V1 relational persistence platform.
- Physical database infrastructure is not the same as logical/domain ownership.
- A shared PostgreSQL server/instance may be used in Version 1 for operational simplicity, but shared physical infrastructure does not create shared domain ownership.
- Shared infrastructure does not permit cross-service table access.
- Exact database names, schema names, and physical database topology remain deferred to detailed design.
- Each service retains responsibility for its authoritative persistence boundary.
- PostgreSQL does not change the ownership rules established by ADR-003.

## Future Reconsideration Triggers

- New workload patterns require a different persistence model.
- The platform expands into materially different data ownership or latency patterns.
- A future architecture requires a deliberately different storage strategy than relational persistence.

## Related Architecture

- Platform Architecture
- Data Ownership
- Solution Architecture
- Microservices Design
- Database and migration design decisions

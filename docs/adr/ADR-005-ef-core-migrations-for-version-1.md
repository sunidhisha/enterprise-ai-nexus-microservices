# ADR-005: EF Core Migrations for Version 1

## Status
Accepted

## Context

The Version 1 transactional services are primarily ASP.NET Core services. The architecture must keep schema evolution disciplined, reviewable, and aligned with service ownership without introducing competing migration systems in Phase 0.

## Decision

.NET-owned transactional schemas will use EF Core Migrations as the Version 1 migration mechanism, with migration ownership remaining with the service that owns the schema.

## Alternatives Considered

- Use a separate SQL-first migration system from the start.
- Require all services to adopt a different migration process.
- Allow ad hoc schema changes without formal migration ownership.

## Rationale

EF Core is the natural fit for .NET-owned transactional services in Version 1. It keeps schema evolution within the owning service, aligns with service ownership, and avoids operational duplication that would not add value in Phase 0.

## Consequences

### Positive

- Consistent migration process for .NET-owned services.
- Clear service ownership of schema evolution.
- Lower operational complexity than supporting multiple migration approaches in V1.

### Negative / Trade-offs

- This is not the best fit for every future scenario.
- SQL-first or DBA-led governance may become attractive later.

## Constraints / Guardrails

- Each service owns migrations for its own authoritative schema.
- A service must never execute migrations against another service’s authoritative persistence boundary.
- Migration artifacts must be version controlled.
- Migrations must be code reviewed.
- CI/CD must validate migrations before deployment.
- Production schema changes are controlled deployment activities.
- Application startup must not blindly or automatically mutate production schemas.
- Prefer backward-compatible schema evolution and forward-fix strategies for production changes.
- Schema evolution must remain coordinated with compatible application deployment.
- Flyway is deferred from Version 1.
- Flyway may be reconsidered only if demonstrated requirements emerge, such as SQL-first database governance, heterogeneous or non-.NET transactional services, independent DBA-controlled migration workflows, or operational requirements not served well by EF Core Migrations.
- AI Decision Support does not own authoritative manufacturing transactional schemas and does not automatically justify Flyway.
- This ADR does not imply that Python/FastAPI or AI components are authoritative for manufacturing transactional persistence.

## Future Reconsideration Triggers

- A future architecture requires SQL-first governance or cross-technology migration ownership.
- The platform expands to service patterns not well served by EF Core.
- DBA-controlled migration workflows become a clear requirement.
- Operational needs emerge that are not adequately served by EF Core Migrations.

## Related Architecture

- Platform Architecture
- Solution Architecture
- Data Ownership
- Microservices Design

# ADR-008: Docker Compose for Local Development

## Status
Accepted

## Context

The platform includes multiple services and shared dependencies. The team needs a predictable local developer environment without confusing local infrastructure setup with production orchestration.

## Decision

Docker Compose will provide the Version 1 local development runtime baseline for shared dependencies and developer consistency, without being treated as the production orchestration model.

## Alternatives Considered

- Use Kubernetes for local development.
- Require every service to run in containers for normal local dev.
- Treat Docker Compose as production orchestration by default.

## Rationale

Docker Compose is the right local-development tool for Version 1 because it provides dependency consistency and startup simplicity while keeping the operational model aligned with the architecture. It is not the same as a production orchestration model.

## Consequences

### Positive

- Easier local setup for shared infrastructure.
- Lower developer environment drift.
- A clear separation between local developer tooling and production topology.

### Negative / Trade-offs

- Docker Compose is not a full production orchestration system.
- Local tooling needs to remain disciplined and not be mistaken for production deployment design.

## Constraints / Guardrails

- Docker Compose is for local development infrastructure.
- It is not production orchestration.
- Application runtimes may still be run directly for troubleshooting and debugging.

## Future Reconsideration Triggers

- Local development needs outgrow Compose.
- Production orchestration needs become part of the Phase 0 runtime model.
- Shared dependency startup becomes materially more complex than local environment support.

## Related Architecture

- Platform Architecture
- Security Architecture
- Observability and runtime design

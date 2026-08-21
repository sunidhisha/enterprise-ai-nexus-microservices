# ADR-003: Authoritative Data Ownership Boundaries

## Status
Accepted

## Context

In a distributed manufacturing platform, the architecture must prevent hidden ambiguity about who owns business truth. Without explicit ownership, local copies and read models can create conflicting state or stale decisions.

## Decision

Each authoritative fact will have one owning business boundary, no direct cross-service table access will be allowed, local copies and read models will not transfer ownership, and stale local data will never override authoritative state.

## Alternatives Considered

- Allow local services to mutate or override shared state based on local copies.
- Treat all service data as equally authoritative.
- Allow direct cross-service database access for convenience.

## Rationale

Authoritative ownership is a domain and distributed-systems governance decision. The approved architecture requires a single owner for each authoritative fact and makes local copies non-authoritative. This is essential for correctness, safety, and auditability across manufacturing, inventory, quality, maintenance, genealogy, analytics, and AI support.

## Consequences

### Positive

- Clear accountability for manufacturing facts.
- Reduced risk of stale or conflicting state.
- Stronger domain boundaries and safer downstream decision support.

### Negative / Trade-offs

- More explicit integration and governance work is required.
- Local views may need deliberate synchronization or read-model strategy.

## Constraints / Guardrails

- Authoritative facts have one owning business boundary.
- No direct cross-service table access.
- Local copies and read models do not transfer ownership.
- Stale local data cannot override authoritative state.
- Cross-boundary changes occur through explicit domain/integration boundaries rather than shared persistence.
- Inventory owns authoritative inventory quantity, availability, reservation state, location, and inventory-level lot/serial state.
- Asset & Maintenance owns equipment master data and EquipmentId.
- Industrial Operations owns interpreted operational state, machine observations, alarms, and operational events.
- Industrial Operations may maintain the minimum local equipment reference or identity mapping needed to process machine observations, but this does not transfer equipment-master ownership.
- Manufacturing Genealogy & Traceability owns manufacturing trace relationships between materials, production execution, equipment, and finished output.
- Quality Management owns formal Quality Holds, dispositions, releases, non-conformance decisions, and quality authority.
- Production Execution may stop or contain production when a problem is detected, but it cannot independently release a formal Quality Hold.
- Operational Analytics is non-authoritative and consumes governed analytical facts.
- AI Decision Support is non-authoritative and cannot directly mutate authoritative manufacturing state.
- GraphQL BFF owns no persistent authoritative domain state.

## Future Reconsideration Triggers

- A business concept is found to have multiple competing owners.
- A new domain requires a formal state authority change.
- The platform introduces a shared data pattern that conflicts with the ownership model.

## Related Architecture

- Data Ownership
- Domain Context Design
- Integration Design
- Security Architecture
- Solution Architecture

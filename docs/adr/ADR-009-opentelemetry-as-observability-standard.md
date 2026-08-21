# ADR-009: OpenTelemetry as Observability Standard

## Status
Accepted

## Context

The platform needs consistent metrics, logs, and tracing across services, while keeping the observability model vendor-neutral and architecture-level rather than tied to a specific commercial monitoring product.

## Decision

OpenTelemetry will be the Version 1 observability instrumentation standard, providing consistent tracing, metrics, logs, and correlation context across the platform. It is not the complete monitoring platform and does not replace platform-specific telemetry storage, visualization, alerting, or monitoring backends.

## Alternatives Considered

- Use proprietary monitoring tooling as the default standard.
- Leave observability unstandardized across services.
- Treat OpenTelemetry as an implementation detail rather than a platform standard.

## Rationale

OpenTelemetry is the approved instrumentation standard for the platform. It provides cross-service consistency and supports structured observability without hardwiring the architecture to a specific vendor.

## Consequences

### Positive

- Consistent instrumentation model across services.
- Better operational visibility and correlation.
- Vendor-neutral observability standard for the platform.

### Negative / Trade-offs

- Additional instrumentation discipline is required.
- The platform still needs clear business and operational meaning for signals.

## Constraints / Guardrails

- OpenTelemetry provides consistent instrumentation for traces, metrics, logs, and correlation context.
- The architecture remains vendor-neutral regarding telemetry storage, visualization, alerting, and monitoring backends.
- OpenTelemetry does not replace domain-level business monitoring.
- OpenTelemetry does not replace business audit trails.
- Business-operation auditability remains distinct from technical telemetry.
- OpenTelemetry is the Version 1 observability instrumentation standard, not a complete monitoring platform selection.
- This ADR does not select a monitoring vendor or monitoring backend.

## Future Reconsideration Triggers

- A different observability standard becomes required by a platform mandate.
- Vendor lock-in or product strategy changes conflict with the current model.
- The platform adopts materially different telemetry or tracing requirements.

## Related Architecture

- Platform Architecture
- Integration Design
- Security Architecture
- Operational monitoring and runtime design

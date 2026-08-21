# ADR-010: Kafka Deferred from Version 1

## Status
Accepted

## Context

The platform needs event-driven patterns, but the Phase 0 design must remain operationally focused and avoid unnecessary distributed middleware dependencies before the architecture proves the need.

## Decision

Kafka will remain deferred from Version 1, with business events preserved conceptually but not adopted as a required dependency until the platform shows a clear need for event-driven scale or integration complexity.

## Alternatives Considered

- Adopt Kafka as a default integration backbone in V1.
- Ignore event-driven patterns and avoid business-event semantics.
- Add middleware early without demonstrated need.

## Rationale

The approved architecture explicitly defers Kafka in Version 1. This keeps the platform simpler while preserving the concept of business events and eventual downstream awareness without introducing unnecessary operational complexity.

## Consequences

### Positive

- Lower operational complexity in Phase 0.
- Clear event intent without requiring middleware as a V1 dependency.
- Focus on correctness and governance before event-driven expansion.

### Negative / Trade-offs

- Event-driven scalability is deferred.
- Some future integration patterns may need to be re-evaluated later.

## Constraints / Guardrails

- Kafka is not a Version 1 platform dependency.
- Deferring Kafka does not reject event-driven architecture or business-event semantics.
- Business facts and events may still be modeled conceptually without committing Version 1 to Kafka infrastructure.
- Kafka must not be introduced merely because the architecture contains independently deployable services.
- Asynchronous communication does not transfer authoritative ownership of business state.
- Consumers of business events must not treat replicated or derived data as more authoritative than the owning business boundary.
- The architecture must not equate asynchronous intent with a Kafka dependency.
- This decision does not define Kafka topics, partitions, brokers, consumer groups, schemas, retention policies, deployment configuration, or libraries.

## Future Reconsideration Triggers

Kafka may be reconsidered when demonstrated requirements emerge, including:

- sustained event volume
- multiple independent consumers
- replay requirements
- durable event-history requirements
- independent producer/consumer scaling
- enterprise integration fan-out
- operational decoupling that cannot be handled adequately by the simpler Version 1 integration model

This decision does not require all Version 1 communication to be synchronous.

## Related Architecture

- Solution Architecture
- Integration Design
- Platform Architecture
- Data Ownership

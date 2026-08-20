# Enterprise AI Nexus — Integration Design

## 1. Purpose
This document defines the proposed Version 1 integration architecture for Enterprise AI Nexus. It establishes the conceptual communication model between the approved business services and supporting components without prematurely locking in implementation contracts.

The design is aligned to the approved product vision, business capability map, solution architecture, bounded context design, microservices topology, and data ownership boundaries. The purpose is to keep the integration model simple, governable, and consistent with Version 1 business realities.

This proposal intentionally avoids:
- direct cross-service database access
- implementation-class design
- specific endpoint contracts
- schema or table design
- Kafka adoption in Version 1
- premature decomposition of Manufacturing Core internal modules into separate networked services

---

## 2. Integration Principles
The Version 1 integration model is governed by the following principles:

- Data ownership remains authoritative.
- A consumer must not change another service’s authoritative state through a local copy.
- The authoritative owner of a business concept remains the only source of truth for that concept.
- GraphQL BFF is an experience-layer aggregator only.
- GraphQL BFF owns no persistent business state.
- Manufacturing Core internal module communication stays inside the modular monolith unless there is a compelling reason to create a separate runtime boundary.
- REST is used for synchronous communication only where immediate response is required.
- Not every flow must be synchronous.
- Business events remain valuable even when Version 1 does not yet use Kafka.
- Analytics consumes governed operational facts, not direct transactional database access.
- AI consumes governed context and produces recommendations or anomaly findings, but does not mutate authoritative manufacturing state.
- Industrial telemetry and manufacturing transactional data remain conceptually distinct.
- Raw telemetry is not authoritative manufacturing transactional data.
- Local read models are allowed for performance and experience, but they are never authoritative.
- Manufacturing Core remains a Version 1 modular monolith.
- Inventory Service owns inventory state.
- Industrial Operations owns interpreted operational state.
- Operational Analytics is non-transactional.
- AI Decision Support is advisory.
- GraphQL BFF is experience/composition only.
- Kafka remains deferred.
- Local read models remain non-authoritative.

---

## 3. Integration Participants
The approved Version 1 topology has the following participants:

1. Manufacturing Core
   - Modular monolith containing:
     - Product & Manufacturing Definition
     - Manufacturing Engineering
     - Production Planning & Scheduling
     - Production Execution
     - Quality
     - Manufacturing Genealogy & Traceability
     - Asset & Maintenance

2. Inventory Service
   - Authoritative for inventory-level state

3. Industrial Operations Service
   - Authoritative for interpreted machine state, alarms, operational events, and equipment-condition observations

4. Operational Analytics
   - Supporting component for KPI definitions, calculations, aggregates, and operational projections

5. AI Decision Support
   - Supporting component for recommendation records, anomaly findings, and recommendation context

6. Angular + GraphQL BFF
   - Experience layer
   - No domain transactional ownership
   - No persistent business state

This is the set of allowed integration participants for Version 1.

---

## 4. Synchronous Communication
Synchronous communication is used where the business flow requires an immediate answer or a controlled transactional decision.

Conceptual use cases in Version 1:
- Production Execution asks Inventory Service for current material availability
- Production Execution requests reservation or allocation confirmation
- Manufacturing Core asks Industrial Operations for current machine-state context
- GraphQL BFF queries multiple services to assemble a plant operations screen
- A manufacturing workflow may require a confirmation before proceeding

Synchronous integration should be:
- focused
- bounded
- response-driven
- used only where immediate coordination is required

Synchronous communication is not the default mechanism for all integration patterns. It is reserved for interactions needing immediate, authoritative business response.

---

## 5. Asynchronous Communication
Asynchronous communication is appropriate where the interaction is informational, eventual, or decoupled from the immediate workflow.

Conceptual use cases:
- inventory balance changes observed by downstream consumers
- production progress or completion facts for operational awareness
- alarms and operational events
- quality disposition or hold updates for downstream visibility
- analytics consumption of governed facts
- AI context updates from governed manufacturing and operational data

Asynchronous patterns in Version 1 should be conceptual and event-oriented, not Kafka-dependent. The architecture can preserve business event intent without requiring a broker as a Version 1 dependency.

Important principle:
- “asynchronous” does not mean “unreliable”
- it means response is not required in the same request flow
- it may still carry business meaning, auditability, and eventual consistency

Business events represent facts that have already occurred at the authoritative owner. For example:

Inventory Service changes authoritative inventory state
        ↓
MaterialConsumed
        ↓
Analytics / AI / other downstream consumers

The event describes the result; it does not transfer ownership.

---

## 6. Manufacturing Core ↔ Inventory
This is one of the highest-value integration relationships in Version 1.

### Source
Production Execution inside Manufacturing Core

### Target
Inventory Service

### Business purpose
To reserve, validate, and consume material as production work progresses.

### Communication style
Primary conceptual pattern:
- synchronous for availability and reservation confirmation
- synchronous for authoritative material consumption confirmation
- asynchronous for downstream business facts after the authoritative inventory state has already been updated

### Consistency
- strong/immediate for reservation and availability decisions
- strong/immediate for authoritative material consumption
- eventual for downstream observational updates

### Failure behavior
If Inventory is unavailable:
- production flow must fail safe
- the system should avoid continuing with an assumed reservation
- the user or workflow should be blocked or re-routed to a constrained state
- local stale copies must never override authoritative inventory state

For Version 1, authoritative material consumption is a synchronous interaction:

Production Execution
    → requests Inventory Service to record/confirm consumption
    → Inventory Service performs the authoritative inventory decision
    → successful consumption may then be published as a business fact for downstream consumers

Therefore:
- authoritative material consumption is a synchronous interaction in Version 1
- failure to confirm consumption must not be treated as successful
- MaterialConsumed may exist as an asynchronous business fact after authoritative inventory state has been changed
- an asynchronous event must not independently establish authoritative inventory state in Version 1

### Important scenarios addressed
1. Production Execution needs to reserve material from Inventory.
   - Source: Manufacturing Core / Production Execution
   - Target: Inventory Service
   - Purpose: reserve material for a work order or production step
   - Style: synchronous request/response
   - Consistency: strong/immediate
   - Failure: no reservation is treated as no commitment

2. Production Execution needs current material availability.
   - Source: Manufacturing Core / Production Execution
   - Target: Inventory Service
   - Purpose: check on-hand and available quantities before execution
   - Style: synchronous read/query
   - Consistency: immediate for current availability
   - Failure: constrained or degraded operation; do not proceed with assumed stock

3. Production Execution consumes material.
   - Source: Manufacturing Core / Production Execution
   - Target: Inventory Service
   - Purpose: request authoritative inventory consumption confirmation
   - Style: synchronous authoritative consumption interaction
   - Consistency: immediate and authoritative
   - Failure: if confirmation fails, the consumption is not considered successful
   - After success, MaterialConsumed may be published asynchronously as a business fact for downstream awareness

4. Inventory publishes material-consumption facts.
   - Source: Inventory Service
   - Target: downstream consumers such as Analytics, AI, and operational consumers
   - Purpose: communicate that authoritative inventory state has changed
   - Style: asynchronous business fact
   - Consistency: eventual
   - Failure: the event does not create or modify authoritative inventory state

### Ownership rule
Inventory owns inventory-level quantity, reservation, location, availability, and movement state. Manufacturing Core owns the production intent and execution state, but not the inventory state itself.

### Interaction / decision matrix
| Scenario | Source | Target | Business purpose | Communication style | Authority | Failure handling |
|---|---|---|---|---|---|---|
| Reserve material | Production Execution | Inventory Service | secure reservation for production | synchronous | authoritative inventory decision | no reservation without confirmation |
| Check availability | Production Execution | Inventory Service | determine material readiness | synchronous | authoritative current state | constrained or degraded operation |
| Material consumption | Production Execution | Inventory Service | request authoritative consumption confirmation | synchronous | authoritative inventory state change | no success without confirmation |
| MaterialConsumed fact | Inventory Service | downstream consumers | describe changed inventory state | asynchronous | informational fact only | event cannot establish missing authoritative state |

---

## 7. Manufacturing Core ↔ Industrial Operations
This integration establishes operational context for manufacturing decisions and equipment-state awareness.

### Source
Industrial Operations Service

### Target
Manufacturing Core

### Business purpose
Provide machine-state and operational-condition context to production execution and maintenance workflows.

### Communication style
- synchronous where front-line production or maintenance workflows need current machine condition
- asynchronous for alarms, operational events, and condition observations

### Consistency
- immediate for machine-state checks affecting execution-readiness
- eventual for historical or aggregate operational summaries

### Failure behavior
If Industrial Operations is unavailable:
- Manufacturing Core should continue with last-known non-authoritative context only for read-only decision support
- it must not treat stale operational state as authoritative manufacturing state
- execution-readiness decisions must degrade to a constrained operating state

This architecture is not the authoritative industrial safety-control layer. Industrial Operations provides operational machine-state context and operational gating information, but it does not define the safety-control domain for the plant.

### Important scenarios addressed
1. Production Execution consumes machine-state information.
   - Source: Industrial Operations Service
   - Target: Manufacturing Core / Production Execution
   - Purpose: determine whether equipment is available or operating within expected operating conditions
   - Style: synchronous operational read
   - Consistency: immediate where execution-readiness depends on machine condition
   - Failure: constrained or degraded operation; do not assume readiness from stale information

2. Industrial Operations reports alarms and equipment-condition observations.
   - Source: Industrial Operations Service
   - Target: Manufacturing Core / Asset & Maintenance and Production Execution
   - Purpose: surface equipment-condition data and alarm conditions
   - Style: asynchronous business fact/event
   - Consistency: eventual but timely
   - Failure: alarms remain in the operational stream, but the system should preserve visibility and not assume resolution without confirmation

### Ownership rule
Industrial Operations owns interpreted machine state and alarms. Asset & Maintenance owns EquipmentId and equipment master identity. Raw telemetry remains distinct and not authoritative manufacturing transactional data.

---

## 8. Manufacturing Core ↔ Operational Analytics
This is a governed read/analytics integration. It is not a transactional source-of-truth relationship.

### Source
Manufacturing Core and Inventory / Industrial Operations facts

### Target
Operational Analytics

### Business purpose
Provide manufacturing and operational facts for KPI and trending calculations.

### Communication style
- asynchronous business-fact publication or periodic data push of governed facts
- analytical reads from governed facts rather than direct transactional database access

### Consistency
- eventual for analytical projection
- immediate for operationally sensitive reporting when required

### Failure behavior
If Analytics is unavailable:
- core manufacturing flow continues
- analytics should degrade gracefully without blocking production operations
- dashboards may lag, but business operations must not depend on analytics as a transactional system of record

### Important scenario addressed
1. Analytics receives operational manufacturing facts.
   - Source: Manufacturing Core, Inventory Service, Industrial Operations Service
   - Target: Operational Analytics
   - Purpose: generate KPI definitions, production summaries, quality trends, machine performance
   - Style: asynchronous governed-fact flow
   - Consistency: eventual
   - Failure: analytical lag is acceptable; analytics cannot become a transactional dependency

---

## 9. Manufacturing Core ↔ AI Decision Support
This is a decision-support relationship, not a transactional ownership relationship.

### Source
Manufacturing Core and relevant operational context providers

### Target
AI Decision Support

### Business purpose
Provide governed operational and manufacturing context for anomaly detection and recommendation generation.

### Communication style
- asynchronous context publication or event-based delivery
- synchronous only if a live decision-support response is necessary, and only as advisory

### Consistency
- eventual when delivering context
- advisory and non-authoritative by design

### Failure behavior
If AI is unavailable:
- the workflow continues
- humans remain in control
- AI results are absent, not treated as a required execution gate
- AI does not directly mutate authoritative manufacturing state

### Important scenario addressed
1. AI consumes governed context and publishes recommendations.
   - Source: Manufacturing Core, Inventory Service, Industrial Operations Service, Operational Analytics
   - Target: AI Decision Support
   - Purpose: detect anomalies and generate recommendations
   - Style: asynchronous governed-context intake; recommendation output
   - Consistency: eventual
   - Failure: decision support is degraded, not blocking the business flow

### Ownership rule
AI does not own manufacturing transactions. It owns recommendation records and anomaly findings, but not authoritative PLM, inventory, execution, quality, or asset state.

---

## 10. Inventory ↔ Analytics / AI
This integration is derived and read-oriented.

### Source
Inventory Service

### Target
Operational Analytics and AI Decision Support

### Business purpose
Provide inventory-level facts relevant to availability, material movements, reservations, and operational health.

### Communication style
- asynchronous business facts
- local read models for analytical display and AI context

### Consistency
- eventual and governed
- not authoritative for inventory state

### Failure behavior
If analytics or AI is unavailable:
- inventory state still remains under Inventory Service control
- downstream consumers may not receive updates until recovery
- stale copies must not override inventory status

---

## 11. Industrial Operations ↔ Analytics / AI
This integration provides equipment and operational context used for performance analysis and decision support.

### Source
Industrial Operations Service

### Target
Operational Analytics and AI Decision Support

### Business purpose
Provide alarms, events, machine-state transitions, equipment-condition observations, and process insight for predictive analysis.

### Communication style
- asynchronous event stream
- derived context for analytical and AI use

### Consistency
- eventual
- operationally useful, not authoritative for manufacturing transactions

### Failure behavior
If analytics or AI is unavailable:
- machine-state and alarm data should still be retained in operational context
- analytics lag is acceptable
- AI recommendations remain advisory

---

## 12. GraphQL BFF Integration Responsibilities
The GraphQL BFF is not a transactional system, not a domain authority, and not a replacement for cross-service boundaries.

### Responsibilities
- compose experiences from multiple services
- aggregate read models for user screens
- provide a single experience interface to Angular
- orchestrate read requests where the user experience requires multi-source view composition

### Non-responsibilities
- no domain business logic ownership
- no authoritative manufacturing state
- no persistent business state
- no replacement for the core domains’ own decision flows

### Pattern
GraphQL BFF is a composition and experience layer. It consumes governed facts and read models from multiple sources and shapes them for screens and workflows.

### Important scenario addressed
1. GraphQL BFF composes information from multiple services.
   - Source: GraphQL BFF
   - Target: Manufacturing Core, Inventory Service, Industrial Operations Service, Operational Analytics, AI Decision Support
   - Purpose: assemble plant operations and operational dashboard views
   - Style: synchronous read aggregation
   - Consistency: immediate for read rendering
   - Failure: degrade gracefully; the UI should show partial data and operational error states rather than invent authoritative state

### Interaction / decision matrix
| Scenario | Source | Target | Business purpose | Communication style | Authority | Failure handling |
|---|---|---|---|---|---|---|
| Plant operations page | GraphQL BFF | Manufacturing Core / Inventory / Industrial Ops / Analytics | user-facing operational composition | synchronous reads | read-only composition | partial data, degraded rendering |
| AI-assisted operational view | GraphQL BFF | AI Decision Support / Analytics | display advisory insight | synchronous or async depending on use | advisory, non-authoritative | show unavailable guidance without inventing state |

---

## 13. Business Event Principles
Business events remain an important architectural concept in Version 1 even without Kafka.

Events should represent meaningful business facts, not all technical details. In Version 1, the concept of an event is conceptual and can be implemented through lightweight asynchronous integration patterns when justified.

Examples:
- material reservation created
- material reservation cancelled
- material consumed
- work order started
- work order completed
- quality hold raised
- quality disposition changed
- alarm raised
- alarm cleared
- machine-state change detected
- recommendation published
- anomaly detected

These events should:
- be meaningful to downstream consumers
- be business-oriented
- be governed by ownership rules
- not carry ownership transfer semantics
- not be treated as a replacement for authoritative state

The event describes the result; it does not transfer ownership.

---

## 14. Quality Hold Propagation
Quality within Manufacturing Core remains authoritative for formal Quality Holds and releases.

Within Manufacturing Core, Production Execution must honor the authoritative Quality state through the modular-monolith boundary.

If Inventory activity is affected by a formal Quality Hold, the cross-service interaction must communicate the authoritative hold state explicitly.

For Version 1:
- the Quality decision itself is authoritative inside Manufacturing Core
- cross-service propagation may use a synchronous authoritative interaction where immediate blocking or confirmation is required
- downstream informational consumers may receive the resulting QualityHoldRaised / QualityHoldReleased business facts asynchronously
- an asynchronous copy must not independently release or override the authoritative Quality Hold

### Conceptual pattern
1. Quality inside Manufacturing Core decides the formal hold or release.
2. Production Execution receives the authoritative Quality state within the same service boundary.
3. If a cross-service workflow requires immediate blocking or confirmation, the affected service receives the authoritative hold state explicitly through a synchronous interaction.
4. Downstream informational consumers may then be notified by asynchronous QualityHoldRaised or QualityHoldReleased business facts.

### Failure behavior
If the cross-service propagation is delayed or unavailable:
- the authoritative Quality state remains inside Manufacturing Core
- downstream consumers must not act as if the hold has been released
- any cross-service behavior that depends on the hold should remain blocked or constrained until explicit authoritative confirmation is received

### Interaction / decision matrix
| Scenario | Source | Target | Business purpose | Communication style | Authority | Failure handling |
|---|---|---|---|---|---|---|
| Quality hold decision | Quality module | Production Execution | enforce formal hold within Manufacturing Core | internal modular-monolith interaction | authoritative | no execution continuation without authoritative hold state |
| Cross-service hold propagation | Manufacturing Core / Quality | Inventory Service or dependent service | enforce external blocking when required | synchronous authoritative interaction | authoritative within the business flow | block or constrain the affected action until confirmed |
| QualityHoldRaised fact | Manufacturing Core | downstream consumers | notify downstream info consumers | asynchronous | informational only | no override of the authoritative hold |
| QualityHoldReleased fact | Manufacturing Core | downstream consumers | notify downstream info consumers | asynchronous | informational only | no release without authoritative Quality decision |

---

## 15. Consistency Model
The Version 1 integration model should be pragmatic:

- Strong/immediate consistency is required where a business decision depends on the authoritative state of a domain.
  Examples:
  - material availability before reservation
  - inventory state before production commitment
  - quality release decisions before release to downstream activity
  - authoritative material consumption confirmation

- Eventual consistency is acceptable for downstream operational visibility and analytical consumption.
  Examples:
  - KPI dashboards
  - analytics projections
  - decision-support context updates
  - historical event views

The system should avoid treating eventual consistency as a substitute for authoritative state. Eventual updates cannot override the authoritative owner.

---

## 16. Failure Handling
Failure handling must preserve system safety and ownership integrity.

Conceptual principles:
- if a downstream dependency is unavailable, the workflow must fail safe
- a stale or unavailable read model must not become authoritative
- local error-handling should clearly indicate uncertainty
- the business workflow should not assume optimistic completion when an authoritative dependency cannot confirm it

Examples:
- Inventory unavailable during reservation: no reservation is assumed
- Industrial Operations unavailable during execution-readiness checking: degrade to a constrained operating state
- Analytics unavailable: continue operational flow and log analytical lag
- AI unavailable: continue with human decision authority

### Important scenario addressed
1. One service is temporarily unavailable during a manufacturing workflow.
   - Source: any operational flow
   - Target: dependent service
   - Purpose: maintain business continuity while protecting integrity
   - Style: depends on the workflow; usually synchronous request plus graceful degradation
   - Consistency: immediate for authoritative decisions; eventual for downstream visibility
   - Failure: fail safe, not optimistic completion

---

## 17. Timeout, Retry and Circuit-Breaker Principles
These principles apply conceptually, not to a specific implementation.

- Timeouts should reflect the business criticality of the interaction.
- Synchronous calls to authoritative services should have bounded latency.
- Retries should be safe and idempotent.
- Retry storms must be prevented.
- Circuit breakers should protect downstream services from repeated failures.
- Degraded operation should be preferred to cascading failure.

Important distinction:
- timeouts and retries are technical control patterns
- they do not change the domain ownership rules
- a stale or failed read does not become authoritative

---

## 18. Idempotency
Idempotency is required for authoritative business actions whenever the same operation may be retried.

Examples:
- reservation requests
- material consumption submissions
- production completion signals
- quality hold or release decisions
- AI recommendation acknowledgments

The integration model should ensure that:
- repeated requests do not produce ambiguous or duplicate business effects
- retry behavior does not create duplicated state changes
- receipt of the same business fact can be safely deduplicated at the appropriate boundary

Idempotency is essential because distributed systems commonly experience retries, duplicate messages, and transient network interruption.

---

## 19. Local Read Models and Data Synchronization
Local read models are permitted for:
- operational dashboards
- user experience composition
- local caching
- analytics context enrichment
- AI context preparation

They are not authoritative.

### Rules
- read models do not transfer business ownership
- synchronization may be eventually consistent
- local data must be clearly marked as non-authoritative
- stale local data cannot override authoritative state
- local read models must be treated as informational unless explicitly proven to be synchronized and governed

This rule is especially important for:
- analytics projections
- AI context preparation
- composite user views
- operational summaries and dashboards

---

## 20. Integration Observability
The architecture should provide observability across cross-service flows without exposing low-level implementation detail.

Observability must answer:
- which service initiated the interaction
- which service received it
- what business purpose the integration served
- whether the interaction succeeded, failed, or degraded
- how long the interaction took
- whether stale or non-authoritative data was used in a decision

Observability should focus on:
- business transaction tracing
- service reliability
- failure classification
- state-change tracking
- downstream dependency health

This is necessary to preserve trust in governance, ownership, and operational continuity.

---

## 21. Version 1 Integration Strategy
The recommended Version 1 strategy is intentionally narrow and conservative:

- Manufacturing Core remains a modular monolith with clear internal domain modules
- Inventory Service is an independent and authoritative business service for inventory-level state
- Industrial Operations Service is an independent and authoritative business service for machine-state and event data
- Operational Analytics remains a governed analytical component
- AI Decision Support remains a governed advisory component
- Angular + GraphQL BFF remains the experience layer
- synchronous communication is used only for authoritative, immediate interactions
- asynchronous business facts are used for awareness and derived insight
- no Kafka dependency in Version 1
- no direct cross-service database access
- no reliance on local copies for authoritative decisions

This is the preferred strategy because it:
- preserves data ownership
- avoids premature distributed complexity
- keeps the manufacturing core coherent
- allows operational and AI use cases without turning AI or analytics into transactional authorities
- offers a clear path for future evolution

---

## 22. Future Kafka Adoption Criteria
Kafka is not a Version 1 dependency and should not be adopted simply because the architecture has multiple services.

Kafka may become a later architectural choice only when the following criteria are met:

- sustained event volume exceeds lightweight asynchronous patterns
- multiple independent consumers require the same event stream
- replay requirements become operationally important
- long-lived event history is needed beyond simple operational retention
- decoupling requirements materially exceed current service boundaries
- telemetry and event-stream scale create throughput pressure
- integration with external enterprise systems requires durable event fan-out
- independent consumer scaling is required beyond current patterns

If the platform remains primarily single-site and bounded in event volume, Kafka may never be required. The architecture should not force it for appearance or parity.

---

## 23. Integration Risks
The main integration risks in Version 1 are:

1. Ownership ambiguity
   - risk: a consuming service begins to treat a local copy as authoritative
   - mitigation: enforce data ownership boundaries and non-authoritative read models

2. Over-synchronization
   - risk: too many interactions become synchronous when eventual patterns would be safer
   - mitigation: use synchronous communication only where needed

3. Hidden coupling
   - risk: services become tightly coupled through implicit assumptions or stale local copies
   - mitigation: event and read-model governance

4. Operational drift
   - risk: plant operations depend on stale machine or inventory state
   - mitigation: fail-safe behavior and clear non-authoritative status

5. Analytics or AI becoming pseudo-transactional systems
   - risk: analytics or AI begin to influence authoritative state without explicit governance
   - mitigation: keep them advisory and governed

6. GraphQL BFF overreach
   - risk: BFF begins to own business logic or responsibility for domain state
   - mitigation: keep BFF as composition only

7. Unclear failure semantics
   - risk: dependency unavailability creates inconsistent user experiences
   - mitigation: define fail-safe behavior and degraded-mode rules

---

## 24. ADR Candidates
The following architecture decision records are candidates for later formalization:

1. ADR-001: Version 1 service topology and modular monolith boundary for Manufacturing Core
2. ADR-002: authoritative ownership rules for manufacturing, inventory, operational state, and quality
3. ADR-003: use of synchronous service communication for immediate availability, reservation, and authoritative consumption decisions
4. ADR-004: asynchronous event semantics for operational awareness and analytics
5. ADR-005: GraphQL BFF as experience-layer composition only
6. ADR-006: analytics and AI are advisory, not transactional
7. ADR-007: Kafka is deferred until explicit operational criteria are met
8. ADR-008: read models are non-authoritative and must not override source-of-truth state
9. ADR-009: fail-safe and degraded-mode behavior for dependency outages
10. ADR-010: separation of raw industrial telemetry from authoritative manufacturing transactional state
11. ADR-011: formal quality hold propagation remains authoritative inside Manufacturing Core and only explicit cross-service propagation can block downstream state changes
12. ADR-012: business events describe facts that have already occurred at the authoritative owner and never transfer ownership

This is the proposed architecture for Version 1: governed, bounded, and intentionally constrained to business-meaningful integration patterns without premature implementation detail.

---

## 25. Explicitly Deferred Detailed Design
Explicitly defer:

- individual REST endpoints and HTTP routes
- request/response DTOs
- GraphQL schemas
- business-event payload schemas
- Kafka topics and broker configuration
- database tables and schemas
- EF Core entities
- migration implementation
- Docker container definitions
- Kubernetes topology
- Azure resources
- implementation classes
- detailed API versioning

These decisions belong to later detailed architecture, platform design, ADRs, or implementation phases.

This document intentionally defers those details to later phases and does not define them as part of the approved Version 1 integration design.

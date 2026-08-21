# Development Roadmap

## 1. Purpose

This document defines the approved implementation sequence for the Version 1 Enterprise AI Nexus platform. It translates the approved Phase 0 architecture decisions into a staged development roadmap that preserves the architecture intent, service ownership model, and sequencing discipline for the first release.

This roadmap is intentionally aligned to the approved architecture and does not redefine the service topology, data ownership model, or platform principles. It is a delivery sequence, not a new architecture decision set.

---

## 2. Approved Architecture Basis

The following rules are the architectural basis for the roadmap and remain in force throughout the implementation sequence:

- Manufacturing Core remains the Version 1 modular monolith.
- Inventory Service remains independently owned and independently operated.
- Industrial Operations remains independently owned and independently operated.
- The GraphQL BFF remains a user-experience composition and interaction coordination layer only.
- The GraphQL BFF does not own authoritative business state or business logic.
- No direct cross-service database access is permitted.
- Keycloak is the IAM boundary, not the business authority.
- EF Core Migrations are the Version 1 migration mechanism for .NET-owned transactional schemas.
- Docker Compose is used for local development environment orchestration.
- OpenTelemetry is the instrumentation standard.
- Kafka is explicitly deferred from Version 1.
- AI remains advisory and non-authoritative.
- Contextual dependency criticality is preserved: dependencies are evaluated by business and operational context, not by a blanket assumption that every dependency has equal criticality.
- Asset & Maintenance owns equipment master data, EquipmentId, and the maintenance lifecycle.
- Industrial Operations owns observations, interpreted operational state, alarms, operational events, and equipment-condition observations.
- Industrial Operations may maintain only minimum local equipment identity mappings needed to correlate operational data.
- Ownership does not transfer from Asset & Maintenance to Industrial Operations through local mapping.
- The platform remains focused on a single-facility manufacturing flow in Version 1.

---

## 3. Implementation Sequence Overview

The approved roadmap proceeds in ordered phases from foundation through release readiness. The sequence deliberately distinguishes between architecture proof and end-user readiness:

- Phase 0 establishes the architecture baseline and business ownership model.
- Phase 1 through Phase 4 build the technical platform, security boundaries, and the first architecture-validating transactional vertical slice.
- Phase 5 delivers the first usable MVP, which is intentionally greater than a technical architecture proof: it is operationally usable and meaningful to real users.
- Phase 6 through Phase 9 extend the platform incrementally into industrial operations, analytics, AI, and final V1 release readiness.

The first architecture-validating transactional vertical slice is not the same as the first usable MVP. The first architecture-validating vertical slice proves the service topology, ownership boundaries, and transactional integrity patterns. The first usable MVP adds the completeness and usability required for real operational adoption.

---

## 4. Manufacturing Core Implementation Sequence

Manufacturing Core remains a single deployable modular monolith in Version 1. Its internal implementation sequence follows the approved manufacturing domain coherence and business ownership model.

The sequence is:

1. Product & Manufacturing Definition
2. Manufacturing Engineering
3. Planning & Scheduling
4. Production Execution
5. Quality
6. Genealogy
7. Asset & Maintenance

This ordering represents implementation dependency and delivery priority only. It does not permit modules to bypass explicit ownership, domain terminology, invariants, or internal module boundaries. Manufacturing Core remains one Version 1 deployable modular monolith.

---

## 5. Phase 0 — Architecture Baseline and Ownership Definition

### Goal

Establish the approved architectural baseline and ownership model for the Version 1 platform before implementation begins.

### Scope

- Confirm the single-facility V1 scope.
- Confirm service topology and responsibilities.
- Confirm Manufacturing Core as a V1 modular monolith.
- Confirm Inventory Service independence.
- Confirm Industrial Operations independence.
- Confirm the GraphQL BFF as composition and interaction coordination only.
- Confirm data ownership boundaries and authoritative state ownership.
- Confirm Keycloak as IAM rather than business authority.
- Confirm EF Core Migrations for V1.
- Confirm Docker Compose for local development.
- Confirm OpenTelemetry as instrumentation standard.
- Confirm Kafka deferred.
- Confirm AI as non-authoritative.
- Confirm contextual dependency criticality.

### Dependencies

- Approved business and technical architecture
- Domain ownership model
- Platform guardrails and project constraints

### Deliverables

- Approved architecture intent and boundaries
- Published ownership model
- Service and experience-layer scope for Version 1
- Platform and engineering guardrails for implementation

### Architectural Guardrails

- Maintain the distinction between authoritative business state and supporting derived insight.
- Preserve the no direct cross-service database access rule.
- Preserve graph composition responsibilities within the BFF.
- Keep AI non-authoritative and subordinate to domain authority.
- Maintain manufacturing ownership boundaries inside Manufacturing Core.

### Testing Focus

- Ownership review and boundary validation
- Architectural conformance checks
- Dependency criticality review for service execution and auth flows

### Exit Criteria

- The team agrees on the V1 service topology and ownership model.
- The platform boundaries are stable enough for implementation sequencing.
- The architecture is sufficiently concrete to proceed without changing the core operating model.

### Deferred From This Phase

- Deep implementation of business workflows beyond the baseline architecture
- Execution-heavy analytics features
- AI functionality beyond advisory support models
- Event-driven scale patterns beyond the approved deferred position

---

## 6. Phase 1 — Technical Foundation

### Milestone Checkpoint

Technical Foundation — End of Phase 1

### Goal

Establish the technical runtime foundation required to build the first secure and traceable manufacturing workflows.

### Scope

- Platform baseline for application runtime and local development
- Shared developer environment and service startup model
- Application instrumentation baseline using OpenTelemetry
- Database and migration discipline for the V1 transactional services
- Internal service and module boundaries in Manufacturing Core
- Technical readiness for business service implementation without changing architecture intent

### Dependencies

- Phase 0 architectural baseline
- Approved platform standards
- Local development environment model

### Deliverables

- Working technical baseline for the platform components
- Standardized observability and runtime instrumentation
- Migration discipline for transactional services
- Development environment ready for implementation work

### Architectural Guardrails

- Manufacturing Core remains a modular monolith in V1.
- Inventory Service remains independent from Manufacturing Core ownership.
- Industrial Operations remains independent from Manufacturing Core transactional authority.
- GraphQL BFF remains composition and interaction coordination only.
- No direct cross-service database access is introduced in implementation.

### Testing Focus

- Local environment startup and dependency readiness
- Instrumentation and observability validation
- Migration and schema evolution readiness
- Basic service health and dependency behavior

### Exit Criteria

- The team can run the platform and validate the technical baseline in a local development workflow.
- The platform can support business implementation without violating the architecture model.
- Key technical and operational constraints are understood before business flows are implemented.

### Deferred From This Phase

- End-user operational features beyond the technical foundation
- Domain-scale analytics and AI workflows
- Broad industrial integration beyond controlled operational capability

---

## 7. Phase 2 — Security and Identity Foundation

### Milestone Checkpoint

Security and Identity Foundation — End of Phase 2

### Goal

Establish the security and identity model so that business flows can be implemented under clear trust and authorization boundaries.

### Scope

- Identity and IAM boundary definition using Keycloak
- Application trust model across user and service interactions
- Experience-layer access boundaries for GraphQL BFF composition
- Domain ownership of business authorization and validation
- Policies that prevent the BFF from becoming the sole authoritative decision-maker
- Operational readiness for secure transactional flows in later phases

### Dependencies

- Phase 1 technical platform baseline
- Approved Keycloak role as IAM boundary
- Domain service ownership model

### Deliverables

- Secure identity and trust model for V1
- Role and boundary model for user and service interaction
- Platform security expectations aligned with the domain ownership model

### Architectural Guardrails

- Keycloak is IAM, not business authority.
- Domain services remain responsible for business authorization decisions and validation.
- The GraphQL BFF may handle experience-level controls but not authoritative business authorization.
- Industrial Operations and Manufacturing Core keep clear boundaries even when operational context is shared.

### Testing Focus

- Identity boundary validation
- Trust and access control review
- Authorization drift detection
- Business logic separation checks

### Exit Criteria

- The platform has a secure identity foundation for all planned V1 flows.
- The team can implement transactional business logic without collapsing authorization ownership into the BFF.
- Secure service interaction is established before domain workflows are broadened.

### Deferred From This Phase

- Full operational analytics and AI-driven advisory workflows
- Advanced automation or autonomous decision-making
- Broadening the platform beyond the security-foundation scope

---

## 8. Phase 3 — First Secured Manufacturing Capability

### Milestone Checkpoint

First Secured Manufacturing Capability — End of Phase 3

### Goal

Deliver the first secure manufacturing capability within the modular monolith while preserving the domain ownership and transactional boundaries of the architecture.

### Scope

- Product and manufacturing definition foundation
- Production intent and authoritative manufacturing-state implementation within Manufacturing Core
- Security-enforced access for the first manufacturing workflow
- Internal module sequencing within Manufacturing Core
- Initial operational consistency for manufacturing execution within the modular monolith

### Dependencies

- Completed Phase 1 technical foundation
- Completed Phase 2 identity and trust model
- Approved Manufacturing Core internal module sequence

### Deliverables

- First secure manufacturing workflow implemented within the approved ownership model
- Manufacturing Core internal module boundaries are exercised in production-like flow
- Controlled transactional state behavior without bypassing module ownership

### Architectural Guardrails

- Manufacturing Core remains a V1 modular monolith; no premature internal service split.
- Replaceable local abstractions do not change business ownership.
- Product and manufacturing definition ownership remains internal to Manufacturing Core.
- Business authorization stays at the domain and service boundary, not at the BFF.

### Testing Focus

- Manufacturing workflow security validation
- Transactional consistency checks within Manufacturing Core
- Boundary conformance between modules and business ownership
- Controlled failure and authorization review

### Exit Criteria

- A first secure manufacturing capability has been implemented and validated.
- The architecture is proven sufficient to support additional manufacturing functionality without redesign.
- The domain remains coherent and the modular monolith remains the correct V1 model.

### Deferred From This Phase

- Multi-service transactional orchestration beyond the first capability
- Industrial Operations-rich operational integration
- Analytics and AI use cases outside the core manufacturing workflow

---

## 9. Phase 4 — First Architecture-Validating Transactional Vertical Slice

### Milestone Checkpoint

First Secured Multi-Service Transactional Flow — End of Phase 4

### Goal

Prove the cross-service architecture, transactional ownership model, and integration boundaries by implementing the first architecture-validating transactional vertical slice.

This is the phase that validates the architecture itself. It is distinct from the first usable MVP, which comes later.

### Scope

- Execute a focused end-to-end transactional flow spanning the business domains required for the first architecture proof
- Validate authoritative ownership across Manufacturing Core, Inventory Service, and other required V1 boundaries
- Exercise the rules around no direct cross-service database access
- Confirm the GraphQL BFF acts only as experience composition and interaction coordination
- Confirm operational context and state flow under the approved ownership rules

### Dependencies

- Phase 3 secure manufacturing capability
- Approved service boundaries and ownership model
- Local dependency model and operational readiness

### Deliverables

- First end-to-end architecture-validating transactional slice
- Verified service boundary behavior under real domain flow
- Evidence that the V1 service topology works before broadening toward MVP readiness

### Architectural Guardrails

- No direct cross-service database access.
- Inventory Service remains independent and authoritative for its domain state.
- Manufacturing Core remains the authoritative manufacturing business service.
- Industrial Operations remains responsible for observations and interpreted operational state, not authoritative manufacturing transactions.
- The GraphQL BFF remains an experience composition layer only.
- AI remains advisory and non-authoritative.

### Testing Focus

- Transactional flow validation across service boundaries
- Dependency and ownership conformance
- Authorization and trust validation in multi-service flow
- Failure handling and degraded-state behavior
- Consistency between authoritative state and derived operational state

### Exit Criteria

- The architecture is proven through a complete transactional slice that spans the intended V1 boundaries.
- The team has evidence that the approved architecture supports secure and coherent multi-service flow.
- The platform is ready to evolve into the first usable MVP without major topology change.

### Deferred From This Phase

- Broad user-facing feature completeness beyond the slice
- Full analytics maturity
- Advanced AI capability and autonomous recommendation workflows
- Non-essential capability expansion beyond the architecture proof

---

## 10. Phase 5 — First Usable MVP

### Milestone Checkpoint

First Usable MVP — End of Phase 5

### Goal

Deliver the first usable MVP that is operationally useful to users and demonstrates the approved architecture in a concrete business context.

This is the first genuinely usable product milestone. It is intentionally distinct from the earlier architecture-validating transactional slice.

### Scope

- Expand the first proven transaction and capability set into a functioning, usable operational experience
- Deliver a user-facing operational flow that can support early real use in the target manufacturing setting
- Integrate the business flow, experience composition, and supporting service boundaries in a coherent user journey
- Ensure the MVP is usable without requiring the full future-state platform

### Dependencies

- Phase 4 architecture-validating transactional slice
- Security foundation and identity model
- Approved service boundaries and ownership model

### Deliverables

- Usable MVP workflow for the manufacturing scenario
- Experience integration via the GraphQL BFF without authoritative business ownership
- Operationally coherent user journey covering the primary V1 use case

### Architectural Guardrails

- Maintain the V1 service topology and business ownership model.
- Preserve Manufacturing Core as modular monolith with internal ownership boundaries.
- Use the GraphQL BFF for experience composition and interaction coordination only.
- Keep Inventory Service and Industrial Operations independent and authoritative in their own business domains.
- Maintain the separation between authoritative state and advisory AI output.

### Testing Focus

- End-user workflow validation
- Experience-layer reliability and composition coverage
- Cross-boundary transactional validation under real usage patterns
- Readiness of the operational baseline for limited real-world use

### Exit Criteria

- The platform is useful to real users for the approved single-facility manufacturing scenario.
- The architecture remains consistent with the V1 design even as usability and operational completeness are broadened.
- The system is ready to support the next integration and intelligence milestones.

### Deferred From This Phase

- Full advanced industrial operations integration beyond the core MVP scope
- Advanced analytics maturity beyond operational visibility
- Broad AI-based decision support beyond advisory assistance
- Future-state scale and distribution improvements

---

## 11. Phase 6 — First Industrial Operations Integration

### Milestone Checkpoint

First Industrial Operations Integration — End of Phase 6

### Goal

Integrate the industrial operations domain into the V1 platform in a way that preserves ownership boundaries and keeps machine-state reality distinct from manufacturing transaction authority.

### Scope

- Industrial Operations observations and operational state flow into the broader V1 platform
- Equipment-condition observations and operational events in the approved ownership model
- Correlation between machine reality and Manufacturing Core execution context
- Integration of operational state as a supporting context source rather than as transactional authority

### Dependencies

- Phase 5 first usable MVP
- Approved ownership model for Industrial Operations
- Stable integration patterns across V1 services

### Deliverables

- Industrial Operations integration in the operational platform
- Clear flow of operational context into the broader user experience and business processes
- Preservation of ownership rules for equipment identity and operational interpretation

### Architectural Guardrails

- Industrial Operations owns observations, interpreted operational state, alarms, operational events, and equipment-condition observations.
- Asset & Maintenance owns equipment master data, EquipmentId, and the maintenance lifecycle.
- Industrial Operations may maintain only minimum local equipment identity mappings needed to correlate observations.
- Industrial Operations does not own authoritative manufacturing state or equipment master data.
- No direct cross-service database access is permitted for operational or manufacturing data.

### Testing Focus

- Industrial operational state validation
- Correct interpretation of machine and equipment context
- Cross-boundary ownership discipline
- Operational degraded-state behavior and trust boundaries

### Exit Criteria

- Industrial Operations is integrated without transferring ownership or violating the transactional model.
- The system can carry operational context into the user experience and supporting workflows without authority confusion.
- The platform is ready for analytics and insight generation built on governed operational facts.

### Deferred From This Phase

- Broad event-driven platform scale patterns
- Advanced industrial automation beyond the approved V1 scope
- Full AI operational autonomy

---

## 12. Phase 7 — First Analytics Capability

### Milestone Checkpoint

First Analytics Capability — End of Phase 7

### Goal

Establish the first analytics capability using governed facts from the approved service model without making analytics an authoritative state owner.

### Scope

- KPI and operational insight generation from approved manufacturing, inventory, and industrial-state facts
- Operational visibility into the business flow already proven in earlier phases
- Contextual analytics that support production performance, operational health, and decision support
- Derived insight generation while preserving data ownership boundaries

### Dependencies

- Phase 6 first industrial operations integration
- Approved non-authoritative analytics responsibility
- Stable service data access through governed integration patterns

### Deliverables

- First analytics capability aligned to the platform architecture
- Readiness to provide operational insight without becoming a source of authoritative state
- Derived operational view available for business and experience use

### Architectural Guardrails

- Analytics remains non-authoritative.
- Operational Analytics consumes governed facts and does not own authoritative business state.
- The GraphQL BFF remains a presentation and experience composer rather than a business authority.
- AI remains advisory and subordinate to approved domain decisions.

### Testing Focus

- Fact integrity and data quality checks
- Read-model reliability and freshness
- Cross-service analytics consistency
- Insight justification and correct source attribution

### Exit Criteria

- The platform has an initial analytics capability grounded in governed facts and approved domain ownership.
- The analytics layer does not introduce ownership ambiguity or undermine the transactional model.
- The system is ready to support AI-supported insights built on trusted operational context.

### Deferred From This Phase

- Advanced predictive modeling
- Large-scale historical analytics beyond V1 scope
- Autonomous operational optimization logic

---

## 13. Phase 8 — First AI Capability

### Milestone Checkpoint

First AI Capability — End of Phase 8

### Goal

Introduce the first AI capability as a non-authoritative decision-support function grounded in governed manufacturing, operations, and analytical context.

### Scope

- AI capability operating over approved and governed inputs
- Decision-support patterns that supplement business operations without owning business decisions
- Integration with operational context, manufacturing facts, and analytics evidence
- Structured recommendation workflows aligned with the approved architecture boundary

### Dependencies

- Phase 7 first analytics capability
- Approved AI non-authoritative model
- Security and identity foundation

### Deliverables

- Initial AI decision-support capability
- Advisory pattern grounded in governed context
- AI recommendations that remain subordinate to business authority and approved workflows

### Architectural Guardrails

- AI is non-authoritative.
- AI never directly mutates authoritative manufacturing state.
- AI is advisory only and subordinate to domain rules, ownership, authorization, and approved workflows.
- AI must consume governed data and does not replace the role of manufacturing ownership and business authority.
- The GraphQL BFF remains a composition layer; it does not become an AI authority.

### Testing Focus

- Recommendation correctness under approved inputs
- Trust and governance checks
- Business boundary conformance
- Human review and approval compatibility
- Failure modes and degraded advisory behavior

### Exit Criteria

- The platform has a first AI capability that is demonstrably advisory and non-authoritative.
- AI contributions are aligned to the manufacturing and operational ownership model.
- The team can proceed to final V1 release readiness without redefining the architecture.

### Deferred From This Phase

- Autonomous closed-loop control
- Broad higher-order AI optimization beyond V1 advisory scope
- Deep predictive research beyond the approved V1 model

---

## 14. Phase 9 — V1 Release Readiness

### Milestone Checkpoint

V1 Release Readiness — End of Phase 9

### Goal

Complete the platform readiness work required for the first production-quality V1 release while preserving the approved architecture and operational constraints.

### Scope

- Release-quality validation of the full V1 scope
- Security, identity, ownership, and operational readiness review
- End-to-end validation of the core manufacturing flow and supporting services
- Validation of the industrial operations, analytics, and AI integration boundaries
- Release readiness review across user experience, service boundaries, and operational governance

### Dependencies

- All prior phases and milestone checkpoints
- Full V1 architecture and governance model
- Stable implementation of the approved service and ownership boundaries

### Deliverables

- Release-ready V1 implementation aligned with the approved architecture
- Operational readiness evidence for the V1 scope
- Go-live readiness for the approved single-facility manufacturing platform

### Architectural Guardrails

- Preserve Manufacturing Core as the V1 modular monolith.
- Preserve Inventory Service and Industrial Operations independence.
- Preserve the GraphQL BFF as experience composition and interaction coordination only.
- Preserve no direct cross-service database access.
- Preserve Keycloak as IAM, not business authority.
- Preserve AI as non-authoritative and advisory.
- Preserve contextual dependency criticality.
- Preserve equipment ownership and operational ownership rules.

### Testing Focus

- Full-system validation of the approved V1 behavior
- Security, authorization, trust, and operational consistency checks
- Business flow readiness across manufacturing, inventory, industrial operations, analytics, and AI-support functions
- Release-level readiness review and governance sign-off

### Exit Criteria

- The platform is ready for V1 release under the approved architecture.
- The implementation remains faithful to the Phase 0 architectural decisions.
- The platform is release-ready without introducing architecture drift or unnecessary service decomposition.

### Deferred From This Phase

- Post-V1 evolution and future-state scale work
- Additional industrial automation and deeper AI capability beyond V1 scope
- Broader distributed platform initiatives not required for the approved V1 release

---

## 15. Milestone Summary

1. Technical Foundation — End of Phase 1
2. Security and Identity Foundation — End of Phase 2
3. First Secured Manufacturing Capability — End of Phase 3
4. First Secured Multi-Service Transactional Flow — End of Phase 4
5. First Usable MVP — End of Phase 5
6. First Industrial Operations Integration — End of Phase 6
7. First Analytics Capability — End of Phase 7
8. First AI Capability — End of Phase 8
9. V1 Release Readiness — End of Phase 9

---

## 16. Mermaid Roadmap Diagram

```mermaid
flowchart LR
    P0[Phase 0\nArchitecture Baseline]\n    P1[Phase 1\nTechnical Foundation\nMilestone 1]\n    P2[Phase 2\nSecurity and Identity Found.\nMilestone 2]\n    P3[Phase 3\nFirst Secured Manufacturing Capability\nMilestone 3]\n    P4[Phase 4\nFirst Architecture-Validating\nTransactional Vertical Slice\nMilestone 4]\n    P5[Phase 5\nFirst Usable MVP\nMilestone 5]\n    P6[Phase 6\nFirst Industrial Operations Integration\nMilestone 6]\n    P7[Phase 7\nFirst Analytics Capability\nMilestone 7]\n    P8[Phase 8\nFirst AI Capability\nMilestone 8]\n    P9[Phase 9\nV1 Release Readiness\nMilestone 9]

    P0 --> P1 --> P2 --> P3 --> P4 --> P5 --> P6 --> P7 --> P8 --> P9

    subgraph MCore[Manufacturing Core\nV1 modular monolith]
        M1[Product & Manufacturing Definition]
        M2[Manufacturing Engineering]
        M3[Planning & Scheduling]
        M4[Production Execution]
        M5[Quality]
        M6[Genealogy]
        M7[Asset & Maintenance]
    end

    P0 -. architecture baseline .-> MCore
    P3 --> M1
    P3 --> M2
    P3 --> M3
    P4 --> M4
    P5 --> M5
    P6 --> M6
    P8 --> M7

    subgraph Ops[Independent service boundaries]
        INV[Inventory Service]
        IO[Industrial Operations]
        BFF[GraphQL BFF]
        IAM[Keycloak / IAM]
    end

    P2 --> IAM
    P4 --> INV
    P5 --> BFF
    P6 --> IO
    P7 --> P8
```

---

## 17. Explicitly Deferred Beyond Version 1

The following items remain explicitly outside the Version 1 scope and are not part of the approved implementation sequence for this roadmap:

- Broad event-driven distributed integration patterns beyond the approved Phase 0 model
- Kafka as a Version 1 platform dependency
- Autonomous closed-loop machine control
- Advanced predictive or prescriptive AI beyond advisory use
- Full multi-facility scale and multi-region expansion
- Industry-specific compliance packs and extended regulatory scope
- Broad AI optimization or autonomous operational decisioning
- Additional service decomposition beyond the approved V1 modular monolith model
- Future decomposition of Manufacturing Core by domain or runtime boundary without evidence-driven justification
- Expanded platform patterns that are not required to satisfy the V1 architecture and business goals

This section is intentionally separate from the implementation phases and remains deferred beyond Version 1.

---

## 18. Final Architectural Position

The roadmap stays faithful to the approved architecture:

- Manufacturing Core remains the V1 modular monolith.
- Inventory Service and Industrial Operations remain independent.
- Ownership remains explicit and business-aligned.
- The GraphQL BFF remains experience composition and interaction coordination only.
- Keycloak remains identity infrastructure, not business authority.
- AI remains non-authoritative and advisory.
- Kafka remains deferred.
- Operational analytics remain derived and non-authoritative.
- The sequence deliberately separates architecture validation from usable MVP delivery.

This preserves the approved preconditions for a coherent, secure, and operationally defensible Version 1 platform while keeping future expansion clearly outside the V1 scope.

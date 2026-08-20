# Enterprise AI Nexus — Microservices Design

## 1. Purpose

This document defines the approved Version 1 architecture for Enterprise AI Nexus. It captures the business service boundaries, supporting components, ownership rules, and topology decisions that remain valid after review.

The design intentionally limits Version 1 to the smallest credible topology for a single-facility manufacturing environment. It preserves domain ownership, supports clear operational boundaries, and avoids premature decomposition of manufacturing domains into separate deployable services.

This document does not define concrete API contracts, schema definitions, database tables, message topics, runtime packaging, or implementation details.

## 2. Version 1 Architecture Strategy

The approved Version 1 strategy is intentionally conservative and operationally simple:

- Exactly three independently deployable business services exist in Version 1:
  1. Manufacturing Core
  2. Inventory Service
  3. Industrial Operations Service
- Operational Analytics remains a separately deployable supporting component.
- AI Decision Support remains a separately deployable supporting component.
- The Experience Layer is a composition layer built from Angular and a GraphQL BFF.

Manufacturing Core is a single independently deployable modular monolith. It contains multiple internal modules with explicit ownership, domain invariants, and internal boundaries, but those modules are not separate Version 1 microservices.

This strategy preserves strong manufacturing ownership while avoiding a distributed monolith that would result from splitting every manufacturing domain into its own service too early.

## 3. Service Classification

The approved Version 1 classification is:

- Manufacturing Core: Independent business service (modular monolith)
- Inventory Service: Independent business service
- Industrial Operations Service: Independent business service
- Operational Analytics: Supporting independently deployable component
- AI Decision Support: Supporting independently deployable component
- Experience Layer: Composition layer using Angular and GraphQL BFF

The following are not separate Version 1 business services:
- Product & Manufacturing Definition
- Manufacturing Engineering
- Production Planning & Scheduling
- Production Execution
- Quality
- Manufacturing Genealogy & Traceability
- Asset & Maintenance

Those domains remain internal modules inside Manufacturing Core in Version 1.

## 4. Manufacturing Core

### WHAT

Manufacturing Core is the Version 1 business service responsible for the authoritative manufacturing operating model of the facility. It owns the flow of manufacturing definition, planning, execution, quality, equipment maintenance, and genealogy records across the plant.

It contains the following internal modules:
- Product & Manufacturing Definition
- Manufacturing Engineering
- Production Planning & Scheduling
- Production Execution
- Quality
- Manufacturing Genealogy & Traceability
- Asset & Maintenance

### WHY

These modules are tightly coupled by definition, operational need, and shared ownership. They form one coherent manufacturing decision system and should remain together in Version 1 because they share the same business authority, operational principles, and failure domain.

Keeping these functions inside one deployable service avoids unnecessary service sprawl while still preserving internal module boundaries.

### OWNS

Manufacturing Core owns the authoritative manufacturing decisions and state for the facility, including:
- product and manufacturing definition
- manufacturing engineering rules and revisions
- production plans and schedules
- work-order execution and production state
- quality holds, dispositions, and releases
- genealogy records and trace relationships
- equipment master data and maintenance decisions

### DOES NOT OWN

Manufacturing Core does not own:
- inventory-level quantity, availability, reservation state, and location decisions
- operational equipment state, machine alarms, and event data
- raw industrial observations and condition signals
- enterprise-wide analytical definitions
- AI-generated recommendations as authoritative production state

### COMMUNICATION

Manufacturing Core communicates with other business services and supporting components as follows:
- reads inventory availability and reservation state from Inventory Service
- receives operational state and machine-condition observations from Industrial Operations Service
- consumes analytical and operational context from Operational Analytics
- receives actionable recommendations from AI Decision Support for human review
- publishes manufacturing state and quality events for downstream operational awareness

### V1 OR FUTURE

Manufacturing Core is an independently deployable service in Version 1.

Internal modules may evolve into separate services in future versions only when clear business and operational reasons emerge, such as a materially separate planning domain, a heavily independent traceability capability, or a distinct engineering/maintenance operating model.

### TRADE-OFF

The trade-off is intentional simplicity versus theoretical decomposition. Manufacturing Core is a modular monolith, not a collection of independent V1 microservices. This preserves explicit internal ownership and operational coherence while keeping the Version 1 deployment model manageable and governable.

### Internal Modules and Ownership

#### Product & Manufacturing Definition

Owns product identity, revision, material definitions, BOMs, routing, and manufacturing specifications. It is the authoritative source for what is manufactured and how it should be manufactured.

#### Manufacturing Engineering

Owns engineering changes, process design, product-effectivity rules, and manufacturing standards. This module governs the engineering intent that Manufacturing Core executes.

#### Production Planning & Scheduling

Owns the scheduling model, production demand alignment, work sequencing, and plant-level planning logic. It coordinates production intent within the Manufacturing Core while remaining internal to that service in Version 1.

#### Production Execution

Owns actual execution of work orders, WIP state, completion events, material consumption, and production status transitions. It is the operational execution authority within the Manufacturing Core.

#### Quality

Owns formal quality holds, disposition decisions, inspections, defect handling, non-conformance actions, and release decisions. Quality remains a module inside Manufacturing Core, with formal authority over quality state in Version 1.

#### Manufacturing Genealogy & Traceability

Owns manufacturing trace relationships across lots, components, and product lineage. It maintains the manufacturing history needed to connect material, process, and output lineage within the production model.

#### Asset & Maintenance

Owns equipment master data, maintenance decisions, maintenance plans, service actions, and equipment-related operational readiness within Manufacturing Core. It is the owner of equipment identity and maintenance intent within the manufacturing domain.

## 5. Inventory Service

Inventory Service is the authoritative service for inventory-level state. It owns:
- quantity on hand
- quantity available
- reservation state
- material location and movement state
- lot and serial identity at the inventory level
- availability and constrained state

Inventory owns inventory-level lot/serial identity, quantity, location, availability, and reservation state. It is the system of record for material availability and material movement decisions.

Communication:
- receives production demand and material-consumption intent from Manufacturing Core
- provides availability and reservation decisions to Manufacturing Core
- publishes inventory-state changes for operational and analytical awareness

V1 classification:
- Independent business service

## 6. Industrial Operations Service

Industrial Operations Service owns operational state, alarms, events, and equipment-condition observations for the facility. It is the authoritative service for the plant's operational reality as seen at the equipment and process level.

It owns:
- machine state
- alarm state
- operational event streams
- downtime and condition observations
- equipment-condition signals and interpretation

Industrial Operations may maintain minimal local EquipmentId mappings for operational correlation, but this does not transfer equipment ownership. Equipment master data and equipment identity remain owned by Asset & Maintenance inside Manufacturing Core.

This service does not own manufacturing transactions, inventory reservation state, or formal quality disposition state. It provides interpreted operational facts to other services without becoming the owner of authoritative manufacturing decisions.

Communication:
- receives operational observations from industrial sources
- provides equipment and operational state context to Manufacturing Core, Operational Analytics, and AI Decision Support
- contributes operational facts that support maintenance, analytics, and decision support

V1 classification:
- Independent business service

## 7. Operational Analytics Component

Operational Analytics is a supporting independently deployable component. It owns analytical interpretation of governed operational and manufacturing facts without owning the authoritative business state.

Its role is to provide:
- KPI definitions and aggregations
- production and equipment performance views
- operational dashboards
- business-level analytical context derived from authoritative systems

It is read-oriented and does not hold authoritative control over manufacturing transactions.

Communication:
- consumes facts and events from Manufacturing Core, Inventory Service, and Industrial Operations Service
- provides governed analytical context to the Experience Layer and AI Decision Support

V1 classification:
- Supporting component

## 8. AI Decision Support Component

AI Decision Support remains a separately deployable supporting component in Version 1. It provides advisory recommendations and operational anomaly support derived from governed facts, but it cannot directly mutate authoritative manufacturing transactions.

It may produce:
- anomaly detections
- maintenance guidance
- operational recommendations for human review
- decision-support signals for operational awareness

AI does not own product definitions, inventory state, production execution state, quality disposition state, or equipment master data. Its outputs are advisory and must remain subordinate to human operational decision authority.

Communication:
- consumes governed facts from Operational Analytics, Manufacturing Core, and Industrial Operations Service
- publishes recommendations and support signals to the Experience Layer and operational users
- does not mutate authoritative manufacturing state directly

V1 classification:
- Supporting component

## 9. Experience Layer

The Experience Layer is a composition concern, not a business domain. It is the user-facing interaction layer that presents operational views and workflows across the platform.

The approved implementation model is:
- Angular for the user experience and front-end composition
- GraphQL BFF for request orchestration and aggregation

The GraphQL BFF owns no domain business rules and no transactional state. It composes information from authoritative services and supporting components for user workflows, dashboards, and plant operations screens.

The Experience Layer is responsible for user interaction, not for manufacturing ownership.

## 10. Cross-Service Ownership Rules

The following ownership rules are explicit and mandatory:

1. Authoritative records remain with the owning business domain.
2. No service may silently become the owner of another domain's transactional state.
3. Manufacturing Core owns product definition, engineering intent, production execution state, quality state, manufacturing genealogy, and equipment master data within the manufacturing domain.
4. Inventory Service owns inventory-level lot/serial identity, quantity, location, availability, and reservation state.
5. Industrial Operations Service owns operational state, alarms, events, and equipment-condition observations.
6. Asset & Maintenance inside Manufacturing Core owns equipment master data and EquipmentId.
7. Industrial Operations may maintain minimal local EquipmentId mappings only for operational correlation; this does not transfer equipment ownership.
8. Genealogy inside Manufacturing Core owns manufacturing trace relationships.
9. Quality module inside Manufacturing Core owns formal quality holds, dispositions, and releases.
10. Analytics and AI consume governed data and provide derived insight; they do not own authoritative manufacturing transactions.
11. The GraphQL BFF owns no business rules and no transactional state.

These rules prevent the hidden creation of a shared system of record across services.

## 11. Communication Principles

The architecture relies on clear, bounded, business-driven communication:

- Manufacturing Core depends on Inventory Service for material availability and reservation state.
- Manufacturing Core depends on Industrial Operations Service for operational state and equipment-condition context.
- Manufacturing Core owns the authoritative manufacturing execution and quality state.
- Quality decisions remain inside Manufacturing Core and are not owned by the Experience Layer or by AI.
- Operational Analytics derives insight from authoritative facts without owning them.
- AI Decision Support remains advisory and never mutates authoritative manufacturing state.
- The Experience Layer composes views and workflows without owning domain authority.

The system should favor explicit ownership, controlled data flow, and clear dependency direction over shared-state assumptions.

## 12. Version 1 Topology Diagram

The approved Version 1 topology is:

```text
                        Enterprise AI Nexus

        +--------------------------------------------------------+
        | Experience Layer                                        |
        | - Angular                                               |
        | - GraphQL BFF                                          |
        | - no domain business rules                             |
        | - no transactional state                                |
        +--------------------------+-----------------------------+
                                   |
                                   v
        +--------------------------+-----------------------------+
        | Manufacturing Core (independently deployable)          |
        |                                                        |
        |  +------------------------+                           |
        |  | Product & Manufacturing|                           |
        |  | Definition             |                           |
        |  +------------------------+                           |
        |  +------------------------+                           |
        |  | Manufacturing          |                           |
        |  | Engineering            |                           |
        |  +------------------------+                           |
        |  +------------------------+                           |
        |  | Production Planning & |                           |
        |  | Scheduling             |                           |
        |  +------------------------+                           |
        |  +------------------------+                           |
        |  | Production Execution   |                           |
        |  +------------------------+                           |
        |  +------------------------+                           |
        |  | Quality                |                           |
        |  +------------------------+                           |
        |  +------------------------+                           |
        |  | Manufacturing Genealogy|                           |
        |  | & Traceability         |                           |
        |  +------------------------+                           |
        |  +------------------------+                           |
        |  | Asset & Maintenance    |                           |
        |  +------------------------+                           |
        +--------------------------+-----------------------------+
                                   |
                      +------------+------------+
                      |                         |
                      v                         v
      +------------------------+     +-----------------------------+
      | Inventory Service      |     | Industrial Operations      |
      | - inventory state      |     | Service                    |
      | - availability        |     | - machine state           |
      | - reservations        |     | - alarms                  |
      | - lot/serial identity |     | - events                  |
      | - location/quantity   |     | - condition observations  |
      +------------------------+     +-----------------------------+
                                   |
                                   v
        +--------------------------------------------------------+
        | Operational Analytics (supporting component)           |
        | - KPI and dashboard context                            |
        | - analytical interpretation                            |
        +--------------------------------------------------------+
                                   |
                                   v
        +--------------------------------------------------------+
        | AI Decision Support (supporting component)             |
        | - recommendations and anomaly support                 |
        | - advisory only                                        |
        +--------------------------------------------------------+
```

This is the approved Version 1 topology. Manufacturing Core remains a single deployable service with explicit internal module boundaries. Inventory and Industrial Operations remain separate independently deployable business services. Operational Analytics and AI Decision Support remain supporting components.

## 13. Future Service Extraction Criteria

Future extraction should be considered only when a module demonstrates strong separation in ownership, operational behavior, or scaling needs.

### Manufacturing Core modules that may separate later

- Production Planning & Scheduling may become a distinct service if planning logic grows into a materially different operating model.
- Manufacturing Genealogy & Traceability may become a dedicated capability if traceability or compliance requirements become dominant.
- Asset & Maintenance may separate if equipment maintenance and production engineering diverge significantly.

### Non-criteria for Version 1

The Version 1 model does not require splitting modules merely because they are conceptually related. The key test is whether the domain has distinct ownership, independent operational pressure, and a materially different business or technical behavior.

## 14. Distributed-Monolith Risks

The primary risk in this architecture is an over-splitting of manufacturing domains into separate microservices before the business and operational boundaries are actually independent.

The approved strategy mitigates that risk by:
- keeping manufacturing definition, planning, execution, quality, genealogy, and maintenance inside one deployable business service in Version 1
- preserving separate authority for inventory and industrial operations
- keeping analytics and AI as supporting, non-authoritative components
- preventing the Experience Layer from owning manufacturing decisions

This keeps Version 1 cohesive while still preserving explicit internal boundaries and future extraction options.

## 15. ADR Candidates

The following decisions should be recorded as ADRs:

1. Manufacturing Core remains the single deployable manufacturing service in Version 1.
2. Inventory Service owns inventory-level quantity, location, availability, and reservations.
3. Industrial Operations Service owns operational state, alarms, events, and condition observations.
4. Equipment master data and EquipmentId remain owned by Asset & Maintenance inside Manufacturing Core.
5. Quality state remains internal to Manufacturing Core and is not split into a separate Version 1 service.
6. Genealogy remains internal to Manufacturing Core in Version 1 and is treated as a trace authority for manufacturing relationships.
7. AI Decision Support remains advisory and cannot directly mutate authoritative manufacturing transactions.
8. GraphQL BFF composes views without owning domain rules or transactional state.
9. Operational Analytics remains a supporting component derived from authoritative facts.
10. Future extraction of modules from Manufacturing Core should happen only on clear business and operational evidence.

## 16. Explicitly Deferred Detailed Design

The following design details are explicitly deferred and are not part of this architecture artifact:

- endpoint contracts
- GraphQL schema definitions
- database tables and persistence models
- Kafka topics or other messaging specifics
- Docker container design
- Kubernetes topology
- Azure deployment choices
- implementation classes or code structure

These concerns belong to later design and platform execution phases after the approved ownership and topology decisions are accepted.

This document therefore defines the approved Version 1 business architecture, while intentionally deferring platform and engineering design details to subsequent phases.


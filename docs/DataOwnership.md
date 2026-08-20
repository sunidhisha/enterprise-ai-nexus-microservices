## 1. Purpose and scope
This document defines authoritative ownership boundaries for enterprise manufacturing data in Version 1. The intent is to clarify which domain owns the canonical truth for each business concept and to prevent accidental ownership drift across services.

This proposal is governance-focused and intentionally implementation-neutral. It does not define database tables, schemas, entities, migrations, APIs, messaging topics, or infrastructure.

## 2. Core ownership principles
The following principles govern all ownership decisions:

- The service or domain that owns a concept is responsible for the definition, lifecycle, and authoritative state of that concept.
- A consuming service may retain a local read model or derived view for performance, decision support, or display, but it does not become the owner of the underlying fact.
- A local copy may support decisions or display, but it must never authoritatively override state owned by another service.
- Derived values, projections, and analytical measures are owned by the domain that defines the calculation, not by the source domain that supplies the manufacturing facts.
- Identity and definition ownership are distinct from transactional state ownership:
  - definition ownership: who defines the meaning and lifecycle of a concept
  - state ownership: who maintains the current runtime truth for that concept
- No direct cross-service table access is permitted.
- Local copy does not transfer ownership.
- Stale local data cannot override authoritative state.
- Analytics is not a transactional system of record.
- AI is not a transactional system of record.
- GraphQL BFF owns no persistent domain state.
- Raw telemetry is not authoritative manufacturing transactional data.

## 3. Ownership model summary
Version 1 data ownership is split across the following explicit domains:

- Manufacturing Core / Product & Manufacturing Definition owns product and material definition and manufacturing-definition concepts.
- Manufacturing Core / Asset & Maintenance owns equipment master identity and asset state context.
- Manufacturing Core / Production Execution owns production work order execution state.
- Manufacturing Core / Quality module owns inspections, holds, disposition, and authorized releases.
- Manufacturing Core / Genealogy owns manufacturing trace relationships.
- Inventory Service owns inventory state for the referenced material.
- Industrial Operations owns interpreted machine state, operational events, alarms, and equipment-condition observations.
- Operational Analytics owns KPI definitions, calculations, and analytical projections.
- AI Decision Support owns recommendation records, anomaly findings, and recommendation state.

## 4. Product and material ownership
Manufacturing Core / Product & Manufacturing Definition owns the authoritative definition of product and material concepts, including:

- ProductId definition
- MaterialId definition
- product master definition
- material master definition
- manufacturing definition
- BOM
- routing
- revision
- effectivity

This domain is responsible for the business meaning of a product or material as it is defined and changed over time.

Inventory Service references ProductId and MaterialId when managing inventory. It does not own the definition of the material master or product master.

Inventory owns the operational state associated with the referenced material, including:

- lot/serial inventory identity
- quantity
- location
- availability
- reservation
- material movement history

The important boundary is that Inventory owns inventory state for the referenced material, not the material master definition itself.

## 5. Inventory ownership
Inventory Service owns the operational state of material availability and movement for the referenced product or material. This includes:

- physical inventory balances
- lot and serial state
- quantity on hand
- inventory location and sub-location
- stock status and availability
- reservations and allocations
- material movements and inventory history
- inventory-related hold state

Inventory may consume ProductId and MaterialId as references, but it does not define or modify the manufacturing meaning of those identifiers. It maintains the runtime truth of where material exists and how it is allocated.

## 6. Production Work Order ownership
ProductionWorkOrderId is owned by Production Execution inside Manufacturing Core.

This ownership is distinct and specific. Production Execution owns:

- work order definition as a production execution object
- production sequence and execution state
- work order status
- production quantity commitments
- execution-linked resource and operation state
- production progress

Production Work Order ownership is not interchangeable with maintenance work orders. A production work order is not a generic work order concept; it is a production execution construct with its own identity and lifecycle.

## 7. Maintenance Work Order ownership
MaintenanceWorkOrderId is owned by Asset & Maintenance inside Manufacturing Core.

This domain owns:

- maintenance work order definition
- maintenance task tracking
- asset-linked maintenance states
- planned and unplanned maintenance execution
- maintenance completion and follow-up actions
- equipment condition-related maintenance events

This is a separate ownership line from ProductionWorkOrderId. The two identifiers represent different operational objects under the same Manufacturing Core umbrella and must not be collapsed into a generic WorkOrder ownership model.

## 8. Industrial Operations ownership
Industrial Operations owns interpreted operational states and event streams derived from equipment activity, including:

- interpreted machine state
- operational events
- alarms
- equipment-condition observations

Industrial Operations references EquipmentId owned by Manufacturing Core / Asset & Maintenance.

This means the equipment master identity remains authoritative with Asset & Maintenance. Industrial Operations may interpret machine and operational behavior against that equipment identity, but it does not own the equipment master definition. Raw telemetry and local equipment mappings do not transfer equipment master ownership.

## 9. Quality ownership
Manufacturing Core / Quality module owns:

- inspections and inspection results
- defects/non-conformance records
- formal Quality Holds
- disposition decisions
- authorized releases

Quality ownership remains specific and formal. It does not expand into a generic “Quality and Compliance” domain for this Version 1 artifact. Quality is the authoritative owner of conformance decisions and release state relevant to manufacturing material, lots, or production outcomes.

## 10. Asset and equipment ownership
Manufacturing Core / Asset & Maintenance owns the authoritative equipment master identity and asset context, including:

- EquipmentId
- equipment identity and definition
- equipment class and characteristics
- installation and removal state
- asset condition context
- asset hierarchy and relationships
- maintenance linkage

This domain owns the business definition of the equipment asset. Industrial Operations and production execution may reference EquipmentId and consume equipment-related state, but they do not own the equipment master.

## 11. Genealogy ownership
Manufacturing Core / Genealogy owns manufacturing trace relationships.

Genealogy is responsible for establishing and maintaining relationships between:

- materials
- production execution
- equipment
- finished output
- intermediate states
- lot/serial lineage context

Inventory owns inventory-level lot/serial identity and state. Genealogy references those identifiers to establish relationships between materials, production execution, equipment, and finished output. Genealogy does not own the inventory balance itself; it owns the traceability relationships that connect the operating context across manufacturing events.

## 12. Operational Analytics ownership
Operational Analytics owns derived analytics definitions, including:

- KPI definitions
- calculations
- aggregates
- analytical projections
- scorecards and operational summaries

Operational Analytics is responsible for the logic used to transform manufacturing facts into useful insight. It does not own the underlying manufacturing facts used to calculate them.

This distinction is critical:
- source truth remains with the operational domain that produces the fact
- derived analytical measures remain with Operational Analytics
- dashboards, KPI rollups, and aggregate views are analytic outputs, not authoritative transaction state

## 13. AI Decision Support ownership
AI Decision Support owns:

- recommendation records
- anomaly findings
- recommendation context and status
- recommendation provenance and traceability

This domain is responsible for the decision-support artifacts created from operational and analytical context. It owns the record of what was recommended, why, when, and under what conditions.

For Version 1, AI Decision Support does not own model artifacts, model versions, feature definitions, or training metadata. Those are future technical and ML lifecycle subjects to be designed later. This proposal deliberately separates recommendation records from the model lifecycle and ML engineering layers.

## 14. Shared service and local-copy usage
A service may maintain a local copy or read model to optimize access, user experience, or decision support. Such local models are valid when used for:

- display
- local filtering
- operational user assistance
- derived context enrichment
- non-authoritative decision support

However, a local copy must never become an authoritative source that overrides state owned by another service.

This rule applies even when the local copy appears consistent with the source. A local data replica is not a transfer of ownership, and a stale local record cannot be allowed to override the authoritative state of the owning domain.

## 15. Explicit stale-data rule
A consuming service may use a local copy or read model for decisions or display, but a stale local copy must never authoritatively override state owned by another service.

This is the governing rule for distributed ownership, especially in microservice and event-driven environments. The authoritative data owner remains the source-of-truth domain, and any local read-only model must be treated as non-authoritative unless explicitly designed as a controlled, synchronized, shared representation under a defined governance model.

## 16. Authoritative Ownership Matrix
The following matrix defines the intended authoritative ownership and usage rules for Version 1.

| Subject | Authoritative owner | Allowed consumers / references | Local non-authoritative copy permitted |
|---|---|---|---|
| ProductId / MaterialId | Manufacturing Core / Product & Manufacturing Definition | Inventory, Production Execution, Genealogy, Quality, Analytics, AI | Yes, for read-only reference and display |
| BOM / Routing / Revision | Manufacturing Core / Product & Manufacturing Definition | Production Execution, Inventory reference, Analytics, Quality reference | Yes, for read-only reference and display |
| Inventory lot/serial state | Inventory Service | Production Execution, Genealogy, Quality, Analytics, AI | Yes, for read-only operational view |
| Inventory balance | Inventory Service | Production Execution, Quality, Analytics, AI, Genealogy references | Yes, for read-only operational view |
| Inventory reservation | Inventory Service | Production Execution, Quality, Genealogy references | Yes, for read-only operational view |
| ProductionWorkOrderId | Manufacturing Core / Production Execution | Inventory, Quality, Genealogy, Analytics, AI | Yes, for read-only execution view |
| MaintenanceWorkOrderId | Manufacturing Core / Asset & Maintenance | Asset & Maintenance, Quality, Analytics, AI | Yes, for read-only maintenance view |
| EquipmentId | Manufacturing Core / Asset & Maintenance | Industrial Operations, Production Execution, Maintenance, Genealogy, Analytics | Yes, for reference and display |
| Machine state | Industrial Operations | Production Execution, Analytics, AI, operational dashboards | Yes, as a local derived/observed operational view |
| Operational events and alarms | Industrial Operations | Production Execution, Maintenance, Analytics, AI | Yes, as a local operational stream view |
| Quality Hold | Manufacturing Core / Quality module | Inventory, Production Execution, Quality, Analytics, AI reference only | Yes, for display and downstream decision support |
| Quality disposition | Manufacturing Core / Quality module | Inventory, Production Execution, Genealogy, Analytics, AI reference only | Yes, for display and downstream decision support |
| Genealogy relationships | Manufacturing Core / Genealogy | Inventory, Production Execution, Quality, Analytics, AI | Yes, for read-only traceability view |
| KPI definitions/projections | Operational Analytics | Business users, dashboards, AI contextual enrichment | Yes, by design |
| AI recommendations/anomaly findings | AI Decision Support | Users, operational dashboards, downstream workflow integration | Yes, as a decision-support artifact |

## 17. Governance and decision rights
The ownership model is intended to make accountability explicit:

- The owning domain defines the business meaning and lifecycle of a concept.
- The owning domain maintains the authoritative state for that concept.
- A consuming domain may read and transform it for display, planning, analytics, or recommendation.
- Derived or cached representations may support local workflows but must not override authoritative state.
- When a question arises about who owns a data concept, the answer should be determined by:
  1. business definition ownership,
  2. runtime state responsibility,
  3. the domain that has authoritative control over changes.

This governance model prevents conflicts between domains, keeps state accountability clear, and supports safe downstream use of manufacturing data without creating hidden ownership ambiguity.

## 18. Final constraints and non-goals
This Version 1 proposal intentionally excludes detailed design for broader enterprise capabilities not being designed here. The following rules remain in force:

- no direct cross-service table access
- local copy does not transfer ownership
- stale local data cannot override authoritative state
- Analytics is not a transactional system of record
- AI is not a transactional system of record
- GraphQL BFF owns no persistent domain state
- raw telemetry is not authoritative manufacturing transactional data
- this artifact does not define database tables, schemas, EF Core entities, migrations, Flyway scripts, REST endpoints, GraphQL schemas, Kafka topics, Docker containers, Azure resources, or implementation classes

This proposal preserves the approved Version 1 architecture and the required ownership boundaries for product, material, inventory, work orders, industrial operations, quality, genealogy, analytics, and AI recommendation records.

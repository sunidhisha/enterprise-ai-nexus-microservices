# Enterprise AI Nexus — Platform Architecture

## 1. Purpose

This document defines the proposed Version 1 platform architecture for Enterprise AI Nexus. It preserves the approved business, application, integration, security, and data-ownership decisions while defining the platform model for execution, runtime governance, developer experience, and operational controls.

This proposal is intentionally architecture-level. It does not define implementation details such as Dockerfiles, docker-compose.yml, exact container names, ports, networks, volumes, PostgreSQL database or schema names, EF migration classes, Flyway scripts, Keycloak configuration, secrets implementation, exact health endpoints, OpenTelemetry configuration code, GitHub Actions workflows, Kubernetes manifests, Azure resources, Terraform/Bicep, or production deployment topology.

---

## 2. Platform Architecture Goals

The platform layer must provide runtime consistency, security boundaries, operational visibility, and development productivity without leaking infrastructure concerns into the business domains.

### Recommendation: Platform as runtime and governance boundary, not domain logic

- WHAT: The platform provides the execution model, cross-cutting concerns, service boundaries, health semantics, configuration patterns, observability standards, and operational governance for the approved business services.
- WHY: The approved architecture explicitly separates business ownership from infrastructure concerns. This keeps manufacturing rules and state ownership in the correct domain services.
- TRADE-OFF: This adds platform governance and coordination overhead, but it prevents hidden coupling and preserves domain accountability.
- BOUNDARY: Application and domain architecture own business rules, authoritative state, and domain validation. Platform architecture owns runtime standards, dependency handling, health semantics, configuration patterns, and operational process.
- V1 OR FUTURE: V1.

### Recommendation: Preserve explicit service ownership and non-authoritative supporting components

- WHAT: Manufacturing Core, Inventory Service, and Industrial Operations Service each own authoritative responsibilities. Operational Analytics and AI Decision Support remain supporting components that generate derived insight and recommendations.
- WHY: The approved architecture requires clear separation between authoritative state and derived or advisory capability.
- TRADE-OFF: More explicit integration contracts and governance are required.
- BOUNDARY: Only authoritative domain services may own transactional state. Supporting services consume governed facts and provide derived capability.
- V1 OR FUTURE: V1.

### Recommendation: Keep Version 1 intentionally focused and operationally simple

- WHAT: Version 1 should optimize for correctness, observability, and low operational friction rather than distributed scale.
- WHY: The approved scope is a single-facility manufacturing flow, not a broad event-heavy distributed platform.
- TRADE-OFF: This defers some scale and platform optimization until the platform proves the need.
- BOUNDARY: Platform simplicity remains a deliberate design choice.
- V1 OR FUTURE: V1.

---

## 3. Version 1 Runtime Model

The runtime model should support a structured and governable local environment while keeping service boundaries clear.

### Angular

- WHAT: Angular remains the operational user experience application.
- WHY: It supports a rich operational web experience for plant operations, quality, maintenance, and inventory workflows.
- TRADE-OFF: It adds front-end complexity, but it fits the approved experience layer and operational requirements.
- BOUNDARY: Angular owns experience concerns only; it does not own authoritative state.
- V1 OR FUTURE: V1.

### GraphQL BFF

- WHAT: GraphQL remains the backend-for-frontend composition layer.
- WHY: It aggregates data and shapes experience-oriented responses without becoming a business authority.
- TRADE-OFF: It introduces composition logic and request orchestration, but it remains the right choice for screen-oriented experience aggregation.
- BOUNDARY: GraphQL owns composition, not authoritative business logic or state.
- V1 OR FUTURE: V1.

### Manufacturing Core

- WHAT: Manufacturing Core remains the Version 1 modular monolith service for manufacturing definition, planning, execution, quality, genealogy, and asset and maintenance.
- WHY: The approved microservices design identifies these internal modules as tightly coupled and operationally coherent.
- TRADE-OFF: This intentionally limits decomposition in V1 to avoid unnecessary distributed complexity.
- BOUNDARY: The modular monolith remains a single deployable service with internal module boundaries.
- V1 OR FUTURE: V1.

### Inventory Service

- WHAT: Inventory Service remains an independent business service with authoritative responsibility for inventory state.
- WHY: Inventory has distinct ownership, availability semantics, reservation behavior, and operational accountability.
- TRADE-OFF: It creates a separate runtime dependency, but that aligns with the approved ownership model.
- BOUNDARY: Inventory owns inventory state; Manufacturing Core owns production intent and execution state.
- V1 OR FUTURE: V1.

### Industrial Operations Service

- WHAT: Industrial Operations Service remains an independent service for machine state, alarms, operational events, and equipment-condition observations.
- WHY: Industrial telemetry and manufacturing transactional state are separate concerns and must not be conflated.
- TRADE-OFF: This introduces an operational-state boundary, but it is the structurally correct ownership model.
- BOUNDARY: Industrial Operations owns operational interpretation and observations; it does not own authoritative manufacturing transactions.
- V1 OR FUTURE: V1.

### Operational Analytics

- WHAT: Operational Analytics remains a supporting component for KPI definitions, calculations, and operational projections.
- WHY: Analytics is non-authoritative and must work from governed facts rather than transactional ownership.
- TRADE-OFF: It may require derived read models or materialized views, but they remain non-authoritative.
- BOUNDARY: Analytics consumes governed facts; it does not own source business state.
- V1 OR FUTURE: V1.

### AI Decision Support

- WHAT: AI Decision Support remains a supporting component producing recommendations and anomaly support from governed context.
- WHY: The approved architecture requires AI to be advisory and subordinate to authoritative domain rules, authorization, and approved business workflows.
- TRADE-OFF: It adds inference runtime and governance requirements, but AI must not become an authority in the manufacturing domain.
- BOUNDARY: AI consumes governed context and produces recommendations; it does not mutate authoritative state.
- V1 OR FUTURE: V1 with advisory scope only.

### Keycloak

- WHAT: Keycloak remains a separate identity platform service.
- WHY: Platform identity and business authorization are distinct concerns. Centralized identity reduces duplication and keeps domain logic focused.
- TRADE-OFF: It requires identity lifecycle governance, but this is the right platform responsibility in V1.
- BOUNDARY: Keycloak owns identity, tokens, and session context. Domain services own business authorization and business validation.
- V1 OR FUTURE: V1.

### PostgreSQL

- WHAT: PostgreSQL is the Version 1 persistence platform for authoritative transactional data.
- WHY: It is a strong fit for relational manufacturing data and ASP.NET Core domain services.
- TRADE-OFF: It does not remove the need for domain ownership boundaries; it only provides the database engine.
- BOUNDARY: PostgreSQL is physical infrastructure. Logical and service ownership remain domain concerns.
- V1 OR FUTURE: V1.

---

## 4. Docker Strategy

Docker is appropriate in Version 1, but it should be used for local consistency and dependency isolation rather than as a substitute for production architecture.

### Recommendation: Use Docker for deterministic local environments and dependency bootstrapping

- WHAT: Docker should provide consistent runtime baselines for local developer work and startup of shared dependencies.
- WHY: This reduces environment drift and makes startup of infrastructure dependencies predictable.
- TRADE-OFF: It adds a layer of tooling and platform discipline, but it is a worthwhile trade-off for a multi-service system.
- BOUNDARY: Docker belongs in the platform layer, not in the business domain layer.
- V1 OR FUTURE: V1.

### Recommendation: Use Docker Compose for local development, not production orchestration

- WHAT: Docker Compose should manage shared local infrastructure such as PostgreSQL, Keycloak, and optional lightweight observability dependencies.
- WHY: This is the simplest professional model for local developer setup and working against realistic service dependencies without requiring every application service to run inside containers during normal development.
- TRADE-OFF: Docker Compose is not an orchestration system and should not be treated as one.
- BOUNDARY: Local developer topology is a platform concern. Business services remain domain-owned and independently governable.
- V1 OR FUTURE: V1 for local development; not a production orchestration model.

### Important V1 rule

- Application runtimes may run directly on the developer machine for easy debugging.
- Docker Compose manages shared local infrastructure.
- Containerized application execution may still be supported later for integration validation and deployment packaging.

---

## 5. PostgreSQL Strategy

PostgreSQL is a strong Version 1 persistence platform, provided the ownership model remains explicit and disciplined.

### Recommendation: Use PostgreSQL as the transactional platform while preserving logical ownership boundaries

- WHAT: PostgreSQL is the database engine for authoritative transactional data in Version 1.
- WHY: It aligns well with ASP.NET Core, relational manufacturing data, and explicit data ownership.
- TRADE-OFF: It does not eliminate the need for schema governance or service ownership boundaries.
- BOUNDARY: Database infrastructure is physical and platform-level. Logical ownership remains domain-level and service-level.
- V1 OR FUTURE: V1.

### Recommendation: Preserve the principle that physical database infrastructure and logical ownership are different concerns

- WHAT: A shared PostgreSQL server or instance may be operationally convenient in Version 1.
- WHY: Operational convenience is valid as long as logical integrity and service data ownership remain explicit.
- TRADE-OFF: Shared infrastructure can invite accidental coupling if the team treats it as a shared business schema by default.
- BOUNDARY: Physical database topology does not grant logical ownership. Logical domain ownership remains with the service or business domain that owns the data.
- V1 OR FUTURE: V1.

### Key rule

- A shared PostgreSQL instance does not mean shared domain tables or schemas.
- Ownership remains service-owned and domain-owned even when the server is shared.

---

## 6. Database Migration Strategy

### Decision: EF Core Migrations are the recommended Version 1 migration mechanism for .NET-owned transactional schemas

- WHAT: Use EF Core Migrations for database schema evolution in the primary ASP.NET Core transactional services.
- WHY:
  - Version 1 transactional business services are primarily ASP.NET Core.
  - EF Core provides a single, clear migration mechanism for those services.
  - It keeps migration ownership with the service that owns the schema.
  - It avoids maintaining two competing migration mechanisms in V1.
  - Version 1 should favor operational simplicity unless a second migration technology provides demonstrated value.
- TRADE-OFF: This is simpler and more consistent for .NET-owned services, but it is not the best fit for every future architecture scenario.
- BOUNDARY: The service that owns a schema owns its migration evolution. Another service must not migrate another service’s schema.
- V1 OR FUTURE: V1.

### Migration governance principles

- migrations are owned by the service/domain that owns the schema
- migration artifacts are version controlled
- migrations are code reviewed
- CI/CD validates migrations
- production schema changes are controlled deployment activities
- application startup must not blindly mutate production schemas
- prefer forward-fix strategies for production data/schema changes
- services must not migrate another service’s schema
- migration behavior must be explicit and reviewable

### Flyway as a future option

- WHAT: Flyway may be reconsidered later if future requirements justify:
  - SQL-first database ownership
  - heterogeneous or non-.NET transactional services
  - DBA-controlled migration workflows
  - stronger SQL-first operational governance
  - migration patterns not well served by EF Core
- WHY: Flyway may become valuable in future scenarios, particularly for SQL-first governance or mixed-technology service ownership.
- TRADE-OFF: It adds a second migration mechanism and operational burden, which is not justified in V1 unless needed.
- BOUNDARY: Flyway is a future operational option, not a V1 requirement.
- V1 OR FUTURE: Future.

### Explicit V1 rule

- AI components do not own authoritative transactional schemas.
- Future Python/FastAPI AI components do not by default require Flyway.

---

## 7. Keycloak Runtime Responsibility

Keycloak remains the identity provider and IAM platform, not the manufacturing business authorization engine.

### Recommendation: Keycloak as a separate platform identity service

- WHAT: Keycloak provides identity, token context, federation, and standards-based identity flows.
- WHY: The approved security architecture explicitly separates identity from business authorization and business validation.
- TRADE-OFF: The architecture requires clear trust boundaries and identity governance, but preserves domain correctness.
- BOUNDARY: Keycloak owns identity, session, and token context. Business domains own business authorization and validation rules.
- V1 OR FUTURE: V1.

### Key rule

- AI Decision Support is advisory and subordinate to authoritative domain rules, authorization, and approved business workflows.
- Where a workflow requires human approval, AI cannot bypass it.
- This does not imply that every authoritative manufacturing transaction requires human approval; rather, it means AI does not supersede domain authority, approval rules, or approved workflows.

---

## 8. Configuration Management

Configuration should be externalized, environment-aware, and clearly separated from secrets.

### Recommendation: Standardize configuration by environment and purpose

- WHAT: Configuration is organized by environment, service, and concern.
- WHY: This reduces drift and makes it easy to reason about runtime differences across development, test, and deployment environments.
- TRADE-OFF: It adds disciplined configuration conventions, but this is essential for a multi-service platform.
- BOUNDARY: Platform architecture defines the pattern. Domain teams define the meaning of the values.
- V1 OR FUTURE: V1.

### V1 configuration categories

- environment-specific configuration
- connection information
- service URLs
- feature toggles
- logging configuration
- health and readiness thresholds
- non-secret operational parameters

### Principles

- configuration must be explicit and reviewable
- secrets must never be treated as ordinary configuration
- service hostnames and endpoints must be externalized
- logging and tracing settings should be manageable and environment-aware
- configuration should not be embedded in business logic or source code assumptions

---

## 9. Secrets Management

### Recommendation: Separate configuration from secrets and keep secrets out of Git

- WHAT: Secret values are managed separately from operational configuration.
- WHY: This is a required platform security practice and aligns with the approved security architecture.
- TRADE-OFF: It adds operational process for secret lifecycle and rotation.
- BOUNDARY: Platform architecture defines policy; service owners identify which secrets the service requires.
- V1 OR FUTURE: V1.

### Principles

- no secrets committed to Git
- no application secrets inline in source code or scripts
- separate configuration values from secret material
- development and production secret handling must differ in a clear and disciplined way
- least privilege for all service identities and credentials
- secret leakage is a major platform risk and should be treated seriously

### Future option

- Managed secret infrastructure may be added later when cloud or production operations require it.
- This is not required for V1.

---

## 10. Health and Readiness

Readiness must reflect whether a component can perform its required business responsibility, not simply whether a process is alive.

### Recommendation: Define readiness by required business responsibility and dependency criticality

- WHAT: Health and readiness semantics should distinguish:
  - liveness
  - readiness
  - critical dependency
  - optional dependency
  - degraded capability
- WHY: A service can be running while still unable to serve its required function, or it can be partially degraded without affecting authoritative business continuity.
- TRADE-OFF: This requires disciplined status modeling and monitoring.
- BOUNDARY: Domain services define their required business responsibilities. Platform architecture defines the health semantics and patterns.
- V1 OR FUTURE: V1.

### Critical dependency

A dependency is critical when the component cannot perform its required business responsibility without it.

### Optional dependency

A dependency is optional when the component can continue its core capability without it, while a related capability degrades.

### Degraded capability

A degraded capability means a non-critical function is unavailable, but the authoritative core operation remains available.

### Examples

- AI unavailable
  - AI recommendations unavailable
  - core manufacturing operation continues where AI is optional
- Analytics unavailable
  - reporting and analytical capability degrades
  - authoritative manufacturing transactions continue
- Industrial connectivity degraded
  - Industrial Operations reports degraded capability according to the affected connection
  - the rest of the platform does not automatically become unavailable
- Keycloak dependency unavailable
  - Keycloak unavailability may prevent new authentication, token acquisition, or session refresh.
  - The effect on already authenticated API requests depends on the later token-validation design.
  - Authentication must never be silently bypassed.

### Key principle

Readiness should answer:
- Can this component perform its required business responsibility?
- Is the required dependency available?
- Is the capability degraded or unavailable?

Not:
- Is the process running?

---

## 11. Observability

OpenTelemetry remains the recommended Version 1 telemetry instrumentation standard.

### Recommendation: Use OpenTelemetry as the telemetry standard, not necessarily the final storage or monitoring vendor

- WHAT: OpenTelemetry should be the standard for traces, logs, and metrics instrumentation.
- WHY: It is vendor-neutral, widely adopted, and suitable for distributed services and AI recommendation traceability.
- TRADE-OFF: It requires disciplined instrumentation. However, it avoids premature lock-in to a single production monitoring stack.
- BOUNDARY: Platform architecture owns the instrumentation direction. Service teams instrument the business and technical flows relevant to their domain.
- V1 OR FUTURE: V1.

### V1 observability model

- structured logs
- correlation IDs and trace context
- distributed tracing
- metrics
- business-operation tracing
- auditability for operational decisions
- telemetry-ingestion monitoring
- AI recommendation traceability
- clear distinction between operational telemetry and authoritative business state

### Important principle

OpenTelemetry is not necessarily the final vendor-specific observability storage, alerting, or visualization platform. It is the instrumentation and telemetry standards direction for V1.

---

## 12. Resilience

### Recommendation: Standardize resilience patterns as a platform requirement

- WHAT: The platform should define standard patterns for timeout, retry, idempotency, failure isolation, degraded operation, stale data handling, and dependency outages.
- WHY: This prevents inconsistent and unsafe behavior across services and dependencies.
- TRADE-OFF: It requires team discipline and explicit decision-making.
- BOUNDARY: Platform architecture defines the patterns; service teams implement them according to domain behavior.
- V1 OR FUTURE: V1.

### V1 resilience principles

- explicit timeouts
- safe retries
- idempotent state-changing interactions where feasible
- failure isolation between services and capabilities
- degraded operation when a non-critical dependency fails
- refusal to accept stale non-authoritative data as authoritative
- dependency outages must be treated as dependency events with clear operational impact
- business events remain informational unless they are explicitly defined as authoritative by the owning domain

This preserves the approved integration design decisions while keeping the system resilient and operationally understandable.

---

## 13. Local Development Topology

### Recommendation: Developer-machine runtime plus shared Docker Compose dependencies

- WHAT: Application runtimes may run directly on the developer machine for easy debugging, while Docker Compose manages shared infrastructure.
- WHY: This is the simplest professional local development topology for a multi-service system in Version 1.
- TRADE-OFF: It adds some local process management complexity, but it avoids over-containerizing the developer experience.
- BOUNDARY: Local dev topology is a platform concern, not a domain concern.
- V1 OR FUTURE: V1.

### Recommended conceptual topology

Developer Machine
- Angular
- GraphQL BFF
- Manufacturing Core
- Inventory Service
- Industrial Operations Service
- Operational Analytics
- AI Decision Support

Docker Compose managed local infrastructure
- PostgreSQL
- Keycloak
- optional lightweight observability dependencies

### Important rule

- Do not require every application service to run inside Docker for normal development.
- Containerized application execution may still be supported later for integration validation and deployment packaging.

---

## 14. Developer Experience

### Recommendation: Keep developer workflow practical, reproducible, and observable

- WHAT: V1 should support a straightforward developer experience with clear bootstrap, migration, health verification, and debugging workflow.
- WHY: Senior engineering teams need repeatability and low ambiguity.
- TRADE-OFF: More automation requires upkeep, but it is worth it for a multi-service platform.
- BOUNDARY: Developer experience is a platform concern; business logic remains domain-owned.
- V1 OR FUTURE: V1.

### V1 developer expectations

- clear environment bootstrap
- reproducible local infrastructure startup
- service-level health verification
- consistent structured logging and trace IDs
- straightforward debugging across service boundaries
- explicit migration ownership and review discipline
- seed/reference data that is controlled and domain-owned
- dependency startup without turning local development into a mini-production orchestrator

---

## 15. CI/CD Platform Concerns

### Recommendation: Conceptual pipeline order with distinct migration validation and migration execution

- WHAT: The conceptual pipeline is:
  Build
  → Test
  → Security Checks
  → Migration Validation
  → Package
  → Deployment Preparation
  → Controlled Schema Migration
  → Application Deployment
  → Readiness Verification
- WHY: Migration validation should be separated from migration execution and deployment order should remain explicit and controlled.
- TRADE-OFF: This is more disciplined than a single deployment step, but it is the correct risk control for schema changes.
- BOUNDARY: Platform architecture defines the process semantics; service teams own service validation and migration correctness.
- V1 OR FUTURE: V1.

### Principle

Migration validation is not the same as executing the migration. Production schema changes are controlled deployment activities, not blind startup events.

The exact ordering of schema migration and application deployment may vary for backward-compatible expand/contract changes. The architectural requirement is that schema evolution and application deployment remain controlled, validated, coordinated, and compatible.

---

## 16. Production Evolution

### Recommendation: Keep V1 simple and evolution-friendly

- WHAT: Version 1 should be designed for later evolution toward:
  - container orchestration
  - Azure-hosted topology
  - managed PostgreSQL
  - managed secrets
  - centralized observability
  - Kafka or event streaming if justified
  - horizontal scaling
- WHY: This is a legitimate future platform path, but it should not be forced into V1 prematurely.
- TRADE-OFF: Deferring these decisions avoids complexity and needless cost, but requires the architecture to remain portable and cleanly layered.
- BOUNDARY: Business architecture remains portable; production topology decisions are future concerns.
- V1 OR FUTURE: V1 foundation; future scaling decisions later.

### No mandatory V1 assumptions

- Kubernetes is deferred.
- Kafka is deferred.
- Azure production topology is deferred.
- managed platform services are not required in V1.

---

## 17. Platform Security

The platform security model must preserve the approved decisions around Keycloak, service identities, machine identities, least privilege, trusted boundaries, and auditability.

### Recommendation: Security should remain explicit and domain-aware

- WHAT: Security should be anchored in identity separation, least privilege, and explicit trust boundaries.
- WHY: Manufacturing systems require strong trust and accountability across user, service, and machine flows.
- TRADE-OFF: This demands more governance, but it prevents false trust and lateral movement.
- BOUNDARY: Identity and trust are platform concerns. Business authorization and business validation remain domain concerns.
- V1 OR FUTURE: V1.

### V1 platform security principles

- Keycloak remains the IAM platform.
- service-to-service trust is explicit and scoped.
- machine/device identity is distinct from human identity.
- AI and analytics remain governed consumers, not authoritative state owners.
- least privilege applies to all service and user access.
- auditability remains essential for operational decisions.

---

## 18. Platform Ownership Boundaries

### Application / domain architecture

- owns business capabilities
- owns domain rules and authoritative state
- owns service boundaries and service responsibility

### Platform architecture

- owns runtime model
- owns health, configuration, observability, local topology, migration process, and cross-cutting standards

### Security architecture

- owns IAM model, trust boundaries, identity separation, and least-privilege patterns
- does not replace business authorization

### Data architecture

- owns data ownership model
- preserves authoritative data boundaries
- ensures database infrastructure does not collapse domain ownership

### Integration architecture

- owns synchronous vs asynchronous patterns
- owns event intent and dependency behavior
- preserves explicit service boundaries and non-authoritative downstream consumption

---

## 19. Version 1 Platform Scope

### IN V1

- Angular experience layer
- GraphQL BFF
- Manufacturing Core modular monolith
- Inventory Service
- Industrial Operations Service
- Operational Analytics supporting component
- AI Decision Support supporting component
- Keycloak as IAM platform
- PostgreSQL as transactional persistence platform
- Docker for developer consistency and local dependency management
- Docker Compose for local developer infrastructure topology
- explicit configuration and secret separation
- health/readiness semantics
- OpenTelemetry instrumentation standard
- migration governance with EF Core Migrations for .NET-owned transactional schemas
- clear domain ownership and data isolation rules

### DEFERRED

- Kafka and event-streaming infrastructure
- Kubernetes orchestration
- production Azure topology
- managed cloud database infrastructure
- managed secrets platform
- centralized enterprise observability platform
- advanced AI lifecycle and model governance
- multi-region and multi-facility platform scale
- Terraform/Bicep
- Azure resources
- production deployment topology

---

## 20. Platform Risks

### Major Version 1 risks

- accidental shared-database coupling
- migration ownership conflicts
- Docker complexity without clear purpose
- environment drift
- secret leakage
- weak observability and poor traceability
- over-complex local development topology
- Docker Compose mistaken for production orchestration
- premature Kubernetes adoption
- conflating operational telemetry with transactional truth
- allowing AI or analytics to become hidden authoritative owners
- unclear readiness semantics for optional dependencies

These are real risks and should be governed in the platform design.

---

## 21. ADR Candidates

The following decisions should be captured as ADRs:

- PostgreSQL as the V1 persistence platform
- EF Core Migrations as the recommended V1 migration mechanism
- Flyway as a future option only if justified
- Docker use in V1
- Docker Compose as local development dependency orchestration
- Keycloak platform placement and IAM responsibility
- OpenTelemetry as the telemetry standard
- configuration and secret handling policy
- local development topology
- production orchestration deferral
- service ownership and data-isolation boundaries

Each ADR should capture:
- context
- decision
- rationale
- trade-offs
- consequences
- future evolution path

---

## 22. Final Recommendation Summary

### Recommended Version 1 platform strategy

- Use ASP.NET Core for the primary transactional business services.
- Keep Angular + GraphQL as the experience layer.
- Keep Manufacturing Core as a modular monolith in V1.
- Keep Inventory and Industrial Operations as independent services.
- Keep Operational Analytics and AI Decision Support as advisory/supporting components.
- Use PostgreSQL as the persistence platform while preserving strict logical data ownership.
- Use EF Core Migrations as the Version 1 migration mechanism for .NET-owned transactional schemas.
- Keep Flyway as a future option only if justified.
- Use Docker and Docker Compose for local developer consistency and shared dependency startup.
- Keep local application runtimes on the developer machine for debugging.
- Use OpenTelemetry as the telemetry standard without locking to a final vendor.
- Keep Keycloak as the IAM platform and not as the manufacturing authorization engine.
- Maintain explicit readiness semantics for critical vs optional dependencies.
- Defer Kafka, Kubernetes, cloud production topology, and advanced platform scale until the business and operations justify them.

---

## 23. Explicitly Deferred Detailed Design

The following detailed design topics are explicitly deferred and should not be defined in this architecture proposal:

- Dockerfiles
- docker-compose.yml
- exact containers
- ports
- networks
- volumes
- PostgreSQL database/schema names
- EF migration classes
- Flyway scripts
- Keycloak configuration
- exact secrets implementation
- exact health endpoints
- OpenTelemetry implementation
- GitHub Actions workflows
- Kubernetes manifests
- Azure resources
- Terraform/Bicep
- production topology

This keeps the proposal at the correct architectural level and preserves the approved Version 1 scope, ownership rules, and platform boundaries.

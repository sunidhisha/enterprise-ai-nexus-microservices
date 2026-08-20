# Enterprise AI Nexus — Security Architecture

## 1. Purpose

This document defines the approved Version 1 security architecture for Enterprise AI Nexus. It translates the approved product vision, solution architecture, and microservices design into a security model that is appropriate for a manufacturing platform with distributed business services, an experience layer, industrial telemetry, analytics, and AI decision support.

This document is intentionally architecture-level. It does not define Keycloak realms, clients, exact claims, exact OAuth/OIDC scopes, token payload structure, secret material, ASP.NET Core authorization-policy implementation, Angular guards, GraphQL directives, database security design, Docker configuration, Kubernetes configuration, Azure resources, or detailed implementation classes.

The security model is designed to protect manufacturing operations, data ownership, and human accountability while preserving the approved architecture boundaries.

---

## 2. Architectural Context

The approved architecture establishes the following security-relevant boundaries:

- The Experience Layer is a composition layer built from Angular and a GraphQL Backend-for-Frontend.
- Manufacturing Core is the authoritative manufacturing business service for the facility.
- Inventory Service owns inventory-level state and availability decisions.
- Industrial Operations owns operational state, alarms, events, and interpreted machine context.
- Operational Analytics is a governed analytical supporting component.
- AI Decision Support is advisory and does not own authoritative manufacturing transactions.
- Raw telemetry and industrial observations are not the same as authoritative manufacturing state.
- Manufacturing Core, Inventory Service, and Industrial Operations remain domain owners of their respective authoritative state.

These boundaries are the foundation for the Version 1 security architecture.

---

## 3. Security Principles

### Principle 1: Identity is centralized and enterprise-governed

Keycloak is the proposed Version 1 Identity Provider / IAM platform for the platform.

It is responsible conceptually for:
- human authentication
- identity federation where required
- standards-based OIDC/OAuth 2.0 identity flows
- identity/session/token issuance
- identity-level role/group context

It is not responsible for:
- manufacturing business rules
- production-order state validation
- inventory business decisions
- Quality Hold decisions
- machine-state decisions
- AI recommendation authority

Owning domain services remain responsible for business authorization and business validation.

### Principle 2: Authentication, authorization, and business validation are separate concerns

Authentication answers: Who are you?

Authorization answers: Are you permitted to attempt this action for this resource/scope?

Business validation answers: Is this action valid given current manufacturing state?

Example:
A ProductionManager may be authenticated and authorized to request production-order release for Plant-01. Production Execution may still reject the request because material is unavailable, a formal Quality Hold exists, the manufacturing definition is invalid, or required prerequisites are incomplete. That is business validation, not authentication or authorization failure.

### Principle 3: Domain services own authoritative decisions and authorization

Business authorization and business validation remain within the domain service that owns the authoritative action or state.

The GraphQL BFF may validate experience-level access and presentation, but it must not become the sole authorization authority for authoritative actions.

### Principle 4: AI and analytics are governed consumers, not authoritative owners

AI and analytics support decisions through governed facts and recommendations. They do not own authoritative manufacturing state and cannot bypass domain authorization or directly mutate authoritative manufacturing transactions.

### Principle 5: Security must fail safe

For authoritative actions:
- Cannot authenticate -> reject
- Authenticated but unauthorized -> reject
- Authorized but business rule invalid -> domain rejection
- Identity provider unavailable -> do not silently bypass authentication
- Stale authorization context -> must not grant broader authority than can be confirmed
- Analytics unavailable -> manufacturing operations continue
- AI unavailable -> manufacturing operations continue without AI assistance

### Principle 6: Human accountability is required for high-impact manufacturing decisions

The system must preserve clear accountability for high-impact operational actions. Human or domain accountability is required where defined by workflow or policy. AI cannot bypass required human or domain approval where such approval has been defined.

---

## 4. Recommended Version 1 Security Model

### 4.1 Keycloak as the Version 1 Identity Provider / IAM platform

#### WHAT
Keycloak is the proposed Version 1 Identity Provider / IAM platform.

#### WHY
The platform spans plant operations, quality, manufacturing, inventory, maintenance, and analytics. A centralized identity platform provides a consistent source of authentication and identity context for both human users and service interactions. This keeps identity concerns out of the manufacturing domain logic and preserves separation of concerns.

#### TRADE-OFF
This creates a dependency on centralized identity governance and requires a clear operating model for user and service identity lifecycle management. There is additional coordination required across platform teams and domain teams.

#### BOUNDARY
Keycloak owns identity, session, and token context. Domain services remain responsible for business authorization and business validation.

#### V1 OR FUTURE
V1.

---

### 4.2 Distinct identity types

The architecture explicitly distinguishes the following identity types:

#### Human identity
- employees
- operators
- managers
- planners
- inspectors
- technicians

#### Service identity
- Manufacturing Core
- Inventory Service
- Industrial Operations
- Analytics
- AI Decision Support
- GraphQL BFF where applicable

#### Machine/device identity
- industrial gateways
- machine connectors
- telemetry producers

#### WHAT
Identity is categorized by actor type so the system does not confuse people, services, and devices.

#### WHY
A machine or telemetry producer is not an employee identity, and a service identity is not a human identity. Mixing identity types leads to false trust assumptions and weak authorization boundaries.

#### TRADE-OFF
Additional governance is required to manage multiple identity classes with different lifecycle expectations and trust characteristics.

#### BOUNDARY
Identity classification belongs to the platform identity and trust model; domain services enforce rules appropriate for the identity type and resource context.

#### V1 OR FUTURE
V1.

---

### 4.3 Authorization model: identity + role + facility scope + action + resource context

#### WHAT
The platform should authorize business access based on the model:
- Identity
- Role
- Facility/Plant scope
- Domain action
- Resource context

Example:
ProductionManager + Plant-01 + ReleaseProductionOrder + ProductionOrder-123

#### WHY
Manufacturing authorization is context-sensitive. Plant scope is essential. Authorization must not simply be “this person has a role somewhere,” because role and resource boundaries matter in a manufacturing production environment.

#### TRADE-OFF
This model is more structured and harder to implement casually. It requires governance around role scope and resource context.

#### BOUNDARY
Authorization policy belongs to the identity and access model, while final enforcement belongs to the owning domain service.

#### V1 OR FUTURE
V1.

This authorization model prevents accidental cross-plant authority. Plant-01 authority must not automatically grant Plant-02 authority.

---

### 4.4 Manageable V1 RBAC model

#### WHAT
The platform should use a compact and manageable Version 1 RBAC model. Candidate roles are evaluated and consolidated to avoid unnecessary role explosion.

#### WHY
A manufacturing platform needs enough role differentiation to match real business responsibilities, but role explosion introduces governance overhead, unclear ownership, and inconsistent operational behavior.

#### TRADE-OFF
Some edge-case access needs may require workflow-based exceptions or domain policy rather than a new role for every narrow case.

#### BOUNDARY
Role governance belongs to the identity and access layer. Domain services enforce action-specific authorization under the role model.

#### V1 OR FUTURE
V1.

#### Proposed business roles
- PlantManager
- ProductionManager
- ProductionPlanner
- Operator
- InventoryManager
- WarehouseOperator
- QualityManager
- QualityInspector
- MaintenanceManager
- MaintenanceTechnician
- ManufacturingEngineer
- ExecutiveViewer

#### Proposed administrative/platform roles
- SystemAdministrator

This distinction preserves operational business roles from platform administration responsibilities.

---

### 4.5 GraphQL BFF security boundary

#### WHAT
The GraphQL BFF may:
- validate authenticated identity
- enforce experience-level access
- propagate appropriate identity/security context
- prevent obviously unauthorized UI operations

The GraphQL BFF must not become the sole authorization authority for authoritative actions.

#### WHY
The BFF is an experience composition layer. It is not the correct place to own the authoritative manufacturing decision. Domain services remain accountable for real business authorization and validation.

#### TRADE-OFF
This reduces convenience for a monolithic single-layer design but strengthens separation of concerns and preserves domain integrity.

#### BOUNDARY
The GraphQL BFF is responsible for experience access. Domain services remain responsible for authoritative business authorization.

#### V1 OR FUTURE
V1.

---

### 4.6 Service-to-service security model

#### WHAT
Service-to-service communication should use workload/service identity, authenticated service calls, least privilege, caller identity awareness, and explicit trust boundaries.

#### WHY
Backend services must not implicitly trust each other. In a manufacturing platform, domain boundaries matter. Explicit service identity and caller context prevent over-broad trust and reduce lateral movement risk.

#### TRADE-OFF
The platform requires more governance and explicit integration design. This increases coordination cost but improves trust and controllability.

#### BOUNDARY
Service identity and trust boundaries are platform-level concerns. Domain services enforce the business meaning of the call.

#### V1 OR FUTURE
V1.

---

### 4.7 Analytics and AI security boundary

#### WHAT
Analytics and AI Decision Support should consume governed data under facility scope, data-sensitivity controls, and least privilege. They must not bypass domain authorization or directly mutate authoritative manufacturing transactions.

#### WHY
Analytics and AI are decision-support components. They provide recommendations, operations insights, and anomaly findings, but they are not the owners of production records, quality dispositions, inventory state, or equipment master decisions.

#### TRADE-OFF
This limits AI autonomy and increases the need for human or domain validation in some workflows. However, it preserves trust, accountability, and manufacturing integrity.

#### BOUNDARY
Analytics and AI operate in the supporting component layer, while domain services own authoritative business actions.

#### V1 OR FUTURE
V1 for the boundary model; broader autonomous AI action remains future.

---

## 5. Trust-Boundary Diagram

```text
Human User
    |
    v
Angular
    |
    v
Keycloak
    |
    | authenticated identity / token context
    v
GraphQL BFF
    |
    +-------------------------------+
    |                               |
    v                               v
Manufacturing Core             Inventory Service
    |                               |
    |                               |
    +---------------+---------------+
                    |
                    | governed facts / business events
                    v
         +---------------------------+
         | Industrial Operations     |
         | machine/device identity    |
         | operational facts         |
         +-----------+---------------+
                     |
                     +----------------------+
                                        |
                                        v
                                 Analytics
                                        |
                                        v
                                      AI
```

Additional conceptual note:
- Human identity flows through Angular -> Keycloak -> GraphQL BFF -> domain services.
- Machine/device identity enters through the Industrial Operations boundary rather than through the human authentication path.
- Analytics and AI consume governed facts and recommendations but do not own authoritative manufacturing transactions.

---

## 6. Identity-Type Matrix

| Identity type | Typical examples | Purpose | Security principle |
|---|---|---|---|
| Human identity | employees, operators, planners, managers, inspectors, technicians | user access to workflows | human accountability and role scope |
| Service identity | Manufacturing Core, Inventory Service, Industrial Operations, Analytics, AI Decision Support, GraphQL BFF where applicable | service-to-service interaction | least privilege and explicit trust |
| Machine/device identity | industrial gateways, machine connectors, telemetry producers | device and data source authentication | source provenance and isolation |

---

## 7. V1 Role Proposal

### Business roles
- PlantManager
- ProductionManager
- ProductionPlanner
- Operator
- InventoryManager
- WarehouseOperator
- QualityManager
- QualityInspector
- MaintenanceManager
- MaintenanceTechnician
- ManufacturingEngineer
- ExecutiveViewer

### Administrative/platform roles
- SystemAdministrator

### Design guidance
- Keep the role set manageable and aligned to real plant operations.
- Use facility or plant scope as a first-class authorization dimension.
- Keep administrative roles separate from business roles.
- Avoid role proliferation unless there is clear operational necessity.

---

## 8. Role + Facility-Scope Model

The platform should evaluate authorization as:
- Identity
- Role
- Facility/Plant scope
- Domain action
- Resource context

Conceptually:
- ProductionManager
- Plant-01
- ReleaseProductionOrder
- ProductionOrder-123

This does not imply authority for:
- Plant-02
- other production orders outside the correct scope
- other domains without explicit permission

This model is especially important for manufacturing organizations operating multiple plants or site-specific operational authorities.

---

## 9. Authorization Responsibility Matrix

| Concern | Primary owner | Secondary owner | Notes |
|---|---|---|---|
| Human authentication | Keycloak | platform identity governance | handles user identity, session, and identity context |
| Identity federation | Keycloak | enterprise IAM governance | for external identity sources when needed |
| User role and group context | Keycloak | enterprise identity governance | identity-level context, not business logic |
| Experience-level access checks | GraphQL BFF | domain services | user experience gating only |
| Business authorization | owning domain service | service policy owner | authoritative verification for business actions |
| Business validation | owning domain service | domain workflow owners | validates current state and policy |
| Machine/device authentication | Industrial Operations boundary | industrial integration trust boundary | not equivalent to employee identity |
| Service-to-service trust | platform integration boundary | service owners | least privilege and explicit trust |
| AI recommendation authority | AI Decision Support | domain service approval workflow | advisory only |
| Analytics access | Analytics layer | domain data owners | subject to facility scope and data sensitivity |
| Auditable action tracking | platform governance + domain services | operational controls | for traceability and accountability |

---

## 10. Security Failure Model

For authoritative actions, the platform must follow a predictable failure model:

| Scenario | Required outcome |
|---|---|
| Cannot authenticate | reject |
| Authenticated but unauthorized | reject |
| Authorized but business rule invalid | domain rejection |
| Identity provider unavailable | do not silently bypass authentication |
| Stale authorization context | must not grant broader authority than can be confirmed |
| Analytics unavailable | manufacturing operations continue |
| AI unavailable | manufacturing operations continue without AI assistance |

This protects the platform from over-trusting incomplete or stale context while preserving continuity of operational manufacturing processes.

---

## 11. Major Security Risks

### 1. Treating machine identity as employee identity
- WHAT:
  Machine and device trust is conflated with human identity.
- WHY:
  This weakens source provenance and can create false trust in industrial telemetry and operational state.
- TRADE-OFF:
  Requires more deliberate trust separation and industrial onboarding governance.
- BOUNDARY:
  Industrial Operations boundary and platform trust model.
- V1 OR FUTURE:
  V1.

### 2. Over-centralizing authorization in the GraphQL BFF
- WHAT:
  The experience layer becomes the real business authorization authority.
- WHY:
  This undermines domain ownership and makes business validation inconsistent across services.
- TRADE-OFF:
  Domain services must take more explicit ownership of authorization logic.
- BOUNDARY:
  GraphQL BFF and domain service boundaries.
- V1 OR FUTURE:
  V1.

### 3. Conflating authentication with business validity
- WHAT:
  An authenticated, authorized user is assumed to be permitted to complete the action without business validation.
- WHY:
  Manufacturing decisions depend on state, prerequisites, and formal rules; identity alone is insufficient.
- TRADE-OFF:
  Requires stronger domain validation logic and clearer workflow design.
- BOUNDARY:
  Domain business services.
- V1 OR FUTURE:
  V1.

### 4. Lack of facility scoping
- WHAT:
  A user or service is permitted across plants without plant-specific scope checks.
- WHY:
  Plant boundaries matter in manufacturing and should not be assumed away.
- TRADE-OFF:
  Requires explicit scope handling in authorization and user context.
- BOUNDARY:
  Identity and domain authorization boundaries.
- V1 OR FUTURE:
  V1.

### 5. AI overreach
- WHAT:
  AI recommendations or anomaly findings turn into direct changes or implicit authority.
- WHY:
  AI is advisory and must not bypass the domain decision model.
- TRADE-OFF:
  More human or domain review steps may be needed in some workflows.
- BOUNDARY:
  AI Decision Support and owning domain services.
- V1 OR FUTURE:
  V1 for the boundary; autonomous actions remain future.

### 6. Excessive role sprawl
- WHAT:
  Many narrowly defined roles accumulate.
- WHY:
  Governance becomes harder and permission drift occurs.
- TRADE-OFF:
  More discipline is required in role design and access review.
- BOUNDARY:
  Identity and access governance.
- V1 OR FUTURE:
  V1.

### 7. Weak service identity controls
- WHAT:
  Service-to-service trust is implicit and not scoped.
- WHY:
  This increases the risk of over-privileged or overly broad inter-service communication.
- TRADE-OFF:
  Requires stronger service onboarding and trust governance.
- BOUNDARY:
  Platform integration and service ownership boundaries.
- V1 OR FUTURE:
  V1.

### 8. Weak auditability for high-impact actions
- WHAT:
  Operations, decisions, and approvals are not traceable to accountable actors and scopes.
- WHY:
  Manufacturing decisions require traceability for accountability, investigation, and quality management.
- TRADE-OFF:
  Audit capture adds operational overhead and cost.
- BOUNDARY:
  Domain services and the shared platform governance model.
- V1 OR FUTURE:
  V1.

---

## 12. ADR Candidates

### ADR-001: Adopt Keycloak as the V1 enterprise identity provider
- WHAT:
  Use Keycloak as the platform’s Version 1 IAM platform.
- WHY:
  Centralized identity protects the platform and supports enterprise readiness.
- TRADE-OFF:
  Requires identity governance and central dependency management.
- BOUNDARY:
  Platform identity boundary.
- V1 OR FUTURE:
  V1.

### ADR-002: Separate authentication, authorization, and business validation
- WHAT:
  The platform must keep these concerns distinct.
- WHY:
  Manufacturing correctness depends on state-aware business validation.
- TRADE-OFF:
  More explicit logic and workflow discipline.
- BOUNDARY:
  Identity layer and owning domain services.
- V1 OR FUTURE:
  V1.

### ADR-003: Domain services own authoritative authorization and validation
- WHAT:
  Authorization and validation must remain with the owning domain service.
- WHY:
  This preserves domain accountability and protects business integrity.
- TRADE-OFF:
  More distributed enforcement and more explicit service responsibilities.
- BOUNDARY:
  Domain service boundary.
- V1 OR FUTURE:
  V1.

### ADR-004: Treat device identity separately from human identity
- WHAT:
  Machine/device identity is not human identity.
- WHY:
  Industrial telemetry requires source provenance and device-specific trust rules.
- TRADE-OFF:
  Requires separate onboarding and governance.
- BOUNDARY:
  Industrial Operations and platform trust boundary.
- V1 OR FUTURE:
  V1.

### ADR-005: Use a compact, plant-scoped RBAC model
- WHAT:
  Business roles remain compact and facility-scoped.
- WHY:
  This avoids role explosion and preserves operational clarity.
- TRADE-OFF:
  Some edge-case access may require workflow-based policy.
- BOUNDARY:
  Identity and access governance.
- V1 OR FUTURE:
  V1.

### ADR-006: AI and analytics are advisory only in V1
- WHAT:
  AI and analytics consume governed data and produce recommendations, not authoritative business state changes.
- WHY:
  This preserves manufacturing integrity and prevents bypass of domain authorization.
- TRADE-OFF:
  More human review may be required for certain decisions.
- BOUNDARY:
  AI/analytics helping layer and business domain services.
- V1 OR FUTURE:
  V1 for the model; autonomous action remains future.

---

## 13. Final Security Position

The approved Version 1 security model is intentionally conservative, explicit, and domain-driven.

Keycloak is the proposed identity provider, but it is only the identity authority, not the business authority. Manufacturing Core, Inventory Service, Industrial Operations, and the other domain-owning services remain responsible for the business validity and business authorization associated with their authoritative state and actions.

The GraphQL BFF is an access-aware experience composition layer, not the final business authorization authority. Analytics and AI are governed consumers and advisory decision-support components, not authoritative owners of manufacturing transactions.

This aligns to the approved manufacturing architecture and preserves operational integrity, accountability, auditability, and governance in Version 1.

---

## 14. Explicit Deferred Detailed Design

The following items are explicitly deferred and belong to PlatformArchitecture, ADRs, detailed design, or later implementation phases. They are intentionally not part of this Version 1 security architecture decision set:

- Keycloak realm configuration
- Keycloak client configuration
- exact OAuth/OIDC scopes
- exact JWT claims
- token payload structure
- client IDs
- secrets and credentials
- ASP.NET Core authorization-policy implementation
- Angular guards/interceptors
- GraphQL authorization directives
- machine/device credential implementation
- service-to-service credential implementation
- certificate configuration
- database security implementation
- Docker security configuration
- Kubernetes security configuration
- Azure identity/resources
- detailed audit-storage implementation

These details are not architecture decisions for this document and must be addressed in the relevant detailed design and implementation work rather than in this architecture proposal.

---

## 15. Summary of Approved Security Decisions

The approved Version 1 security architecture establishes the following outcomes:

1. Keycloak is the Version 1 Identity Provider / IAM platform.
2. Authentication, authorization, and business validation are separate and must remain distinct.
3. Domain services own authoritative business authorization and business validation.
4. Human, service, and machine/device identity are explicitly differentiated.
5. Authorization uses identity, role, facility scope, domain action, and resource context.
6. A compact V1 role model is preferred over role explosion.
7. GraphQL BFF is a UI access boundary, not the final business authorization authority.
8. Service-to-service trust must be explicit, least-privilege, and identity-aware.
9. Analytics and AI remain governed, advisory, and non-authoritative in Version 1.
10. Security failure behavior is explicit and fail-safe.
11. Auditability and traceability are part of the security model.
12. Detailed implementation choices remain deferred to later design and implementation phases.

---

## 16. Final SecurityArchitecture.md Structure

1. Purpose and scope
2. Architectural context
3. Security principles
4. Recommended Version 1 security model
   - Keycloak as identity provider
   - identity types
   - authorization model
   - manageable V1 RBAC model
   - GraphQL BFF security boundary
   - service-to-service security
   - analytics and AI security boundary
5. Trust-boundary diagram
6. Identity-type matrix
7. V1 role proposal
8. Role + facility-scope model
9. Authorization responsibility matrix
10. Security failure model
11. Major security risks
12. ADR candidates
13. Final security position
14. Explicit deferred detailed design
15. Summary of approved security decisions
16. Final document structure

This document is the approved final Version 1 Security Architecture proposal.

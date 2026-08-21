# Architecture Decision Records

## Purpose

This directory records the durable architecture decisions for the Enterprise AI Nexus Phase 0 platform. ADRs capture the approved architectural intent, constraints, trade-offs, and rationale without turning implementation details into architecture history.

## Status meanings

- Accepted: the decision is approved and currently active.
- Proposed: the decision is under consideration but not yet approved.
- Superseded: a newer ADR replaces this decision.
- Deprecated: the decision is retained for historical context but is no longer in use.

## ADR index

- ADR-001 — Version 1 Service Topology and Experience Boundary
- ADR-002 — Manufacturing Core as a Modular Monolith
- ADR-003 — Authoritative Data Ownership Boundaries
- ADR-004 — PostgreSQL as Version 1 Persistence Platform
- ADR-005 — EF Core Migrations for Version 1
- ADR-006 — GraphQL as Backend-for-Frontend
- ADR-007 — Keycloak as Version 1 Identity Provider and IAM Boundary
- ADR-008 — Docker Compose for Local Development
- ADR-009 — OpenTelemetry as Observability Standard
- ADR-010 — Kafka Deferred from Version 1
- ADR-011 — AI Decision Support Is Non-Authoritative
- ADR-012 — Dependency Criticality and Health/Readiness Semantics

## Rule for creating future ADRs

Create ADRs only for durable architecture decisions with cross-cutting impact. Do not record implementation detail, tactical configuration churn, or temporary operational choices as ADRs.

## Rule for accepted ADRs

Accepted ADRs are not silently rewritten when architecture changes. If a previously accepted decision is replaced, the change must be recorded explicitly in a new ADR and the older ADR must be marked as Superseded or Deprecated.

## Rule for superseding decisions

A superseding decision requires a new ADR. The newer ADR must explain what changed, why, and what assumptions or constraints are no longer valid.

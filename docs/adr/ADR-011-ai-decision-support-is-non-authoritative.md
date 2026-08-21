# ADR-011: AI Decision Support Is Non-Authoritative

## Status
Accepted

## Context

AI is valuable for recommendations and support, but the manufacturing platform must preserve clear ownership and authority boundaries. AI must not become an alternative source of truth for operational decisions.

## Decision

AI Decision Support will produce recommendations and anomaly findings only, and will never own authoritative manufacturing state or bypass domain authorization, workflow approval, or authoritative business validation.

## Alternatives Considered

- Permit AI to directly mutate authoritative manufacturing state.
- Treat AI outputs as equivalent to business decisions.
- Keep AI as an ungoverned, implicit business authority.

## Rationale

The approved architecture makes AI advisory and non-authoritative. This preserves domain integrity, prevents hidden state authority drift, and keeps AI as a decision-support capability instead of a replacement for business ownership.

## Consequences

### Positive

- Clear separation between recommendation and authoritative state.
- Safer operational model for AI-assisted manufacturing decisions.
- Stronger control over authorization and workflow governance.

### Negative / Trade-offs

- AI recommendations may be constrained by the need to remain subordinate to business authority.
- Some workflows may require explicit human approval or domain validation.

## Constraints / Guardrails

- AI Decision Support may produce recommendations, anomaly findings, predictions, and other decision-support outputs.
- AI does not own authoritative manufacturing state.
- AI must not directly mutate authoritative production, inventory, quality, maintenance, equipment-master, or genealogy state.
- AI must not bypass domain authorization, authoritative business validation, required workflow approvals, or formal Quality authority.
- Where human approval is required by business workflow or policy, AI must not bypass that approval.
- AI output remains advisory regardless of model confidence.
- The authoritative owning business/domain boundary determines whether and how an AI recommendation results in a business action.
- AI unavailability must not prevent core manufacturing operations where AI is classified as an optional dependency.
- AI recommendations and findings must remain conceptually traceable to sufficient input/context and decision provenance for later governance and auditability.
- Future AI capabilities may remain modules within the initially independently deployable AI Decision Support component.
- This decision does not imply that every AI/ML capability automatically becomes a separate microservice.
- Future extraction of an AI capability should require demonstrated evidence such as materially different scaling requirements, ownership, security boundaries, model lifecycle, availability requirements, or runtime requirements.
- This ADR does not define ML algorithms, model architecture, Python/FastAPI implementation, prompts, vector databases, feature stores, model registries, model deployment infrastructure, or confidence thresholds.

## Future Reconsideration Triggers

- A future architecture requires AI to participate in authoritative decision-making under explicit governance.
- The platform introduces a formal controlled AI authority model.
- Regulatory or operational requirements mandate AI as part of a specific approved workflow.
- A specific AI capability demonstrates materially different ownership, scaling, security, model lifecycle, availability, or runtime requirements that warrant explicit architectural reconsideration.

## Related Architecture

- Solution Architecture
- Data Ownership
- Integration Design
- Security Architecture
- Platform Architecture

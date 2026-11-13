# Data Model: Application Fit Assessment

**Feature**: [spec.md](spec.md)
**Design basis**: [research.md](research.md)

## Entities

### Application

An application in the landscape, either the input or a candidate destination.

| Field | Meaning |
|---|---|
| application_id | Stable organization identifier |
| name, description | Human-readable identity and purpose |
| lifecycle_state | Existing, planned, retiring, or retired |
| owner_id | Accountable business or technology owner |
| criticality, classification | Impact and access context |
| source_record_ids | Authoritative landscape records |

An application can supply many capabilities and participate in many landscape
relationships. A planned application cannot be treated as operationally ready.

### Capability

A business outcome and the behavior required to deliver it.

| Field | Meaning |
|---|---|
| capability_id | Stable capability identifier |
| name, business_outcome | What the function delivers |
| required_behaviors | Reviewable behavior statements |
| service_requirements | Required availability, latency, accuracy, and controls |
| owner_id | Business owner who can validate parity |

`ApplicationCapability` links an application to a capability with its implemented
behaviors, known limits, lifecycle state, and evidence IDs. This supports multiple
applications claiming the same capability without assuming parity.

### LandscapeRelationship

An existing or proposed link between applications, capabilities, data, or users.

| Field | Meaning |
|---|---|
| relationship_id | Stable link identifier |
| source_id, target_id | Endpoints |
| kind | Data flow, interface, consumer, dependency, ownership, or control |
| direction, criticality | Impact of change |
| current_or_proposed | Distinguishes observed and target-state links |
| disposition | Retain, move, replace, retire, or unresolved |
| evidence_ids | Records proving the link |

A downstream consumer is not considered resolved merely because a target
capability exists.

### EvidenceRecord

A reference to an authoritative or supplied item, with the minimum permitted
snapshot details required to reproduce a decision.

| Field | Meaning |
|---|---|
| evidence_id | Internal stable reference |
| source_id, source_record_id, source_version | Origin and version or digest |
| source_owner_id | Person or team accountable for the record |
| observed_at, effective_at, retrieved_at | Evidence time context |
| classification, permitted_actions | Source access constraints |
| locator, digest | Restricted-safe reference and integrity marker |
| freshness_policy_id | Source-specific validity policy |

Raw restricted content is held only when explicitly permitted. Source deletion
or later edits do not silently rewrite a completed revision.

### Claim

A material assertion about an application, capability, relationship, or
obligation.

| Field | Meaning |
|---|---|
| claim_id, revision_id | Identity and owning assessment revision |
| subject_id, predicate, value | Asserted fact or relationship |
| status | Verified, inferred, assumed, disputed, or unknown |
| evidence_ids, derived_from_claim_ids | Provenance chain |
| classification | Most restrictive contributing classification |
| validation_owner_id, validation_action | Required follow-up when unresolved |

Contradictory claims remain separate and are linked as disputes. A verified
claim requires current permitted evidence and an explicit validation basis.

### Assessment and AssessmentRevision

`Assessment` identifies the input application, submitted scope, requester,
created time, and current revision. `AssessmentRevision` is an immutable
calculation over a defined snapshot.

| Revision field | Meaning |
|---|---|
| revision_id, assessment_id, prior_revision_id | Identity and history |
| status | Queued, collecting, evaluating, reviewable, or failed |
| snapshot_id, ruleset_version | Reproducibility inputs |
| requested_by, started_at, completed_at | Actor and processing times |
| gap_ids, error_category | Incomplete source or job outcomes |

A correction or new evidence creates a new revision. Prior revisions and their
decision records remain readable to authorized reviewers. A prior revision
is displayed as superseded when `Assessment.current_revision_id` points
elsewhere; its completed record is not mutated.

### FitAssessment

One capability-to-destination evaluation within a revision.

| Field | Meaning |
|---|---|
| fit_id, revision_id, capability_id, destination_id | Evaluation key |
| coverage | Full, partial, none, or unverified |
| covered_behavior_ids, gap_ids | Detailed parity result |
| service_gap_ids, control_gap_ids | Nonfunctional differences |
| evidence_ids | Support for coverage classification |

Full coverage requires every in-scope required behavior and service/control
obligation to be verified. Stale, disputed, or inaccessible material evidence
forces unverified or partial coverage as appropriate.

### Recommendation

A proposed disposition for one capability, with comparison to alternatives.

| Field | Meaning |
|---|---|
| recommendation_id, revision_id, capability_id | Identity |
| disposition, preferred_destination_id | Retain, migrate, consolidate, replace, or investigate |
| alternatives | Other viable destinations or explicit none |
| efficiency_factors | Duplication, change effort, risk, operating impact, business value |
| rationale, assumption_ids, evidence_ids | Explainable basis |
| displayed_status | Proposed, accepted, rejected, or superseded; derived from HumanDecision and current revision |

Numeric cost or savings appears only when sourced cost evidence exists. A
recommendation is not a human-approved decision.

### DecommissionGate

One prerequisite for retirement of the input application.

| Field | Meaning |
|---|---|
| gate_id, revision_id, gate_type | Capability parity, dependencies, data, operations, controls, owner attestation |
| status | Met, blocked, unverified, or not applicable |
| evidence_ids, blocker_ids | Proof or missing proof |
| resolution_owner_id, next_action | Follow-up |

Aggregate readiness is derived as `ready_for_approval` only when every
applicable gate is met. Any blocked or unverified gate yields `conditional`
or `not_supported`, with reasons. The retirement action itself is outside
this model.

### TransitionStage

An ordered current-to-target roadmap step.

| Field | Meaning |
|---|---|
| stage_id, revision_id, order, phase | Position and type |
| capability_ids, relationship_ids | Scope of change |
| prerequisite_stage_ids | Dependency ordering |
| owner_id, expected_outcome | Accountability and result |
| validation_gate, fallback | Go/no-go and recovery |
| displayed_state | Proposed, accepted, or superseded; derived from HumanDecision and current revision |

A stage cannot be proposed before its prerequisites or omit a validation gate
and fallback for a cutover.

### HumanDecision and AuditEvent

`HumanDecision` records assessment approval, rejection, or override with
actor, time, rationale, and revision. It does not execute migration or retirement.
`AuditEvent` records actor, action, outcome, timestamp, affected record IDs,
and a redacted change summary. Both are append-only.

## Relationships

- Application **many-to-many** Capability through ApplicationCapability.
- Application **many-to-many** Application through LandscapeRelationship.
- Assessment **one-to-many** AssessmentRevision; only one revision is current.
- AssessmentRevision **one-to-many** Claim, FitAssessment, Recommendation,
  DecommissionGate, TransitionStage, HumanDecision, and AuditEvent.
- Claim, FitAssessment, Recommendation, and DecommissionGate **many-to-many**
  EvidenceRecord through evidence references.
- TransitionStage **many-to-many** TransitionStage through prerequisites.

## Validation and Access Rules

1. Every material recommendation and retirement status references evidence,
   labelled assumptions, or explicit gaps (FR-003, FR-007, FR-009).
2. No `verified` claim lacks a permitted, current evidence reference. A
   disputed or stale claim remains visible to authorized reviewers (FR-003).
3. A capability with any unverified required behavior cannot have `full`
   coverage (FR-004).
4. A decommission gate is `met` only with recorded validation evidence.
   Readiness never implies automated retirement (FR-009, FR-010).
5. Derived records inherit the strictest source classification; redacted
   views show the existence of a gap without source details (FR-014).
6. Completed revisions, human decisions, and audit events are immutable.
   Reassessment creates a successor revision and a visible change set
   (FR-013, FR-015).
7. Roadmap prerequisites form an acyclic order. Cutover stages require
   owners, validation, and fallback (FR-012).

## State Transitions

```text
AssessmentRevision: queued -> collecting -> evaluating -> reviewable
                                      \-> failed
Assessment.current_revision_id: old revision -> successor revision

Recommendation view: proposed -> accepted | rejected | superseded
TransitionStage view: proposed -> accepted | superseded
```

Only an authorized human action can accept, reject, or override a recommendation.
A failed assessment cannot become reviewable without a new run or explicit
recovery that produces a new immutable revision.

# Contract: Assessment and Review Interface

**Version**: 1 (design contract)

**Applies to**: Authorized reviewer interface and other approved clients
**Related**: [data-model.md](../data-model.md),
[decision-record.md](decision-record.md)

This contract describes behavior and payload semantics. Route names are a
reference design and may be adapted to organization conventions without
changing the required authorization and decision states.

## Common rules

- All operations require an authenticated organization identity.
- Every operation checks actor, action, application scope, record ownership,
  and evidence classification. A view or export is filtered again at delivery.
- IDs are opaque. Timestamps use UTC with an offset. Every response identifies
  the assessment revision and ruleset version when a result exists.
- Repeated submissions with the same idempotency key and identical request
  return the same assessment; a changed payload with the same key is a conflict.
- Assessment creation and reassessment are asynchronous. Clients use the
  returned status link; partial knowledge-source failure appears as a
  reviewable gap when safe, not as fabricated coverage.

## Operations

| Operation | Input | Result | Required behavior |
|---|---|---|---|
| `GET /health` | None | Readiness status | Report ready only when required service and durable job dependencies are available |
| `POST /assessments` | Input application ID or description, scope, supplied evidence references, idempotency key | `202` assessment ID and queued revision | Reject empty identity/scope; authorize source access |
| `GET /assessments/{id}` | Assessment ID | Current status and latest permitted decision record | Hide restricted fields and show non-revealing gaps |
| `GET /assessments/{id}/revisions/{revisionId}` | Revision ID | Immutable permitted revision | Preserve prior result after reassessment |
| `POST /assessments/{id}/evidence` | Evidence reference or correction, reason, expected revision | `202` successor revision | Record actor and change; never mutate the old revision |
| `POST /assessments/{id}/reassess` | Expected revision and reason | `202` successor revision | Freeze a new source snapshot and ruleset version |
| `POST /assessments/{id}/decisions` | Revision, decision type, outcome, rationale | `201` human decision | Require authorized owner/reviewer; never trigger migration or retirement |
| `GET /assessments/{id}/export` | Revision and permitted format | Redacted decision record | Apply export policy to every derived field |

## Input and output semantics

`AssessmentRequest` requires `application` (known ID or supplied
description), `scope` (business capabilities and landscape boundary), and
`requested_by` from authenticated identity. Optional evidence references
must be accessible to the submitting actor. The response reports
`assessment_id`, `revision_id`, `status`, and `status_url`.
The idempotency key is supplied in the `Idempotency-Key` header.

A reference request has this shape:

```json
{
  "application": { "id": "pilot-source" },
  "scope": { "capabilities": "all", "landscape_boundary": "pilot" },
  "evidence_refs": []
}
```

`AssessmentStatus` is `queued`, `collecting`, `evaluating`,
`reviewable`, or `failed`. A failed status includes a safe error category,
retry guidance, and correlation ID, never raw source content.

`EvidenceCorrection` names the affected claim or source record, proposed
value or reference, reason, and expected current revision. It produces a
successor revision and a change summary.

```json
{
  "affected_claim_id": "claim-stale",
  "evidence_ref": {
    "source_id": "pilot-landscape",
    "record_id": "pilot-evidence-updated",
    "version": "2"
  },
  "reason": "Validated source correction",
  "expected_revision_id": "revision-from-current-record"
}
```

`HumanDecision` names a completed revision and records `approve`,
`reject`, or `override` with rationale. Approval of the assessment is
distinct from approval to execute a migration or retire an application.

## Errors

| Status | Meaning |
|---|---|
| `400` or `422` | Invalid scope, missing required fields, or invalid state |
| `401` | Identity missing or expired |
| `403` | Actor lacks permission for the action or export |
| `404` | Assessment or record unavailable to this actor; do not reveal existence |
| `409` | Revision conflict or idempotency key reused with different content |
| `429` | Submission limit reached; include retry guidance |
| `503` | No usable knowledge source or durable job capacity available |

Source-specific failures that still permit a useful assessment are recorded
as gaps in the decision record with a safe source category and resolution
owner. Clients do not reinterpret these as verified facts.

## Acceptance checks

1. A user without source permission cannot retrieve its evidence through an
   assessment, revision, or export (FR-014).
2. A partial target match remains partial and lists the missing behaviors
   (FR-004).
3. Adding evidence creates a new revision and shows changed conclusions
   while preserving the prior revision (FR-013).
4. Recording an approval changes the human decision state only; it does not
   execute migration or retirement (FR-015).

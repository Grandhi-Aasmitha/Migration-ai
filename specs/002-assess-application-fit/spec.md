# Feature Specification: Assess Application Fit and Migration

**Feature Branch**: `main`

**Created**: 2026-11-13

**Status**: Draft

**Input**: User description: "Assess any input application against an existing application
landscape and knowledge layer; identify where its capabilities fit, whether existing systems can
take over those capabilities, why that approach is efficient, whether the input application can
be decommissioned, and provide a roadmap and step-by-step target architecture. Keep the
specification general rather than tied to a named application."

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Understand the Current Application (Priority: P1)

An application owner submits an application for assessment. The assessor builds a reviewable
picture of its business capabilities, users, data, integrations, dependencies, operational
obligations, and known constraints using the existing knowledge layer and supplied evidence.
The owner can see what is verified, inferred, missing, or conflicting.

**Why this priority**: A recommendation cannot be trusted until the source application's
responsibilities and dependencies are understood.

**Independent Test**: Submit a documented sample application and confirm that the assessment
lists its capabilities, dependencies, evidence references, and gaps without requiring a target
recommendation.

**Acceptance Scenarios**:

1. **Given** an application with available landscape records, **When** an owner starts an
   assessment, **Then** the result identifies its capabilities, data, integrations, dependencies,
   obligations, and the source of each material finding.
2. **Given** conflicting or incomplete records, **When** the assessment is viewed, **Then** the
   affected findings are flagged, their sources are visible, and no unsupported fact is presented
   as verified.

---

### User Story 2 - Evaluate Target Fit and Disposition (Priority: P2)

An architect compares each source capability with existing or planned applications in the
landscape. The assessor presents candidate destinations, coverage and gaps, trade-offs, and a
reasoned recommendation to retain, migrate, consolidate, replace, or investigate further.
It also states whether decommissioning is ready, conditional, or unsupported.

**Why this priority**: The core business value is a defensible fit decision that avoids
duplicating capabilities and exposes the work needed before retirement.

**Independent Test**: Supply a landscape with one full-match capability, one partial match, and
one unmatched capability; verify distinct fit results, rationale, and conditional disposition.

**Acceptance Scenarios**:

1. **Given** a candidate target that covers only part of a source capability, **When** fit is
   assessed, **Then** the covered and uncovered behavior is listed separately, with required
   validation before any migration recommendation.
2. **Given** all documented capabilities have candidate destinations but an unresolved
   downstream dependency remains, **When** decommission readiness is assessed, **Then** the
   application is marked conditional and the dependency is named as a blocker.
3. **Given** multiple viable target options, **When** the recommendation is shown, **Then** it
   compares business value, change effort, risk, duplication, and operational impact, and states
   why the preferred option is more efficient under stated assumptions.

---

### User Story 3 - Review a Staged Transition Plan (Priority: P3)

A migration planner receives a proposed current-to-target architecture and an ordered roadmap.
Each stage shows the capabilities and data moving, dependencies to resolve, validation needed,
ownership, decision gates, and the fallback if a stage fails. The planner can review and revise
the proposal before any separate implementation approval.

**Why this priority**: An actionable sequence turns a fit recommendation into a reviewable
program of work while preserving operational continuity.

**Independent Test**: Use an assessment with two target destinations and confirm that the plan
orders prerequisite work before cutover and decommission decisions.

**Acceptance Scenarios**:

1. **Given** an approved assessment for planning, **When** the roadmap is generated, **Then**
   stages identify prerequisites, scope, owners, validation evidence, decision gates, and fallback.
2. **Given** a data or integration dependency shared by two stages, **When** the architecture is
   reviewed, **Then** the dependency is shown in the current and proposed states and sequenced
   before dependent cutovers.

---

### User Story 4 - Challenge and Update the Assessment (Priority: P3)

A reviewer can correct a finding, add evidence, and request reassessment. The assessor retains
the previous result and shows which recommendations, readiness decisions, and roadmap stages
changed as a result.

**Why this priority**: Landscape knowledge changes, and stakeholders need a clear audit trail
for consequential decisions.

**Independent Test**: Change the evidence for one capability and verify that the revised result
identifies changed conclusions while retaining the prior assessment for comparison.

**Acceptance Scenarios**:

1. **Given** an assessment with a recorded evidence gap, **When** a reviewer adds valid evidence,
   **Then** the affected findings and recommendations are recalculated and changes are visible.
2. **Given** a reviewer without permission for a sensitive record, **When** the assessment is
   viewed, **Then** restricted content is withheld while the existence of a decision-relevant
   evidence gap remains visible.

### Edge Cases

- If the input application is absent from the landscape, create a provisional assessment from
  supplied material and mark the inventory gap.
- If two systems claim the same capability, compare their actual coverage and ownership; do not
  equate a shared label with functional parity.
- If evidence is stale, contradictory, or inaccessible, flag the affected conclusion and request
  validation instead of asserting a definitive fit or retirement decision.
- If a capability has no suitable destination, recommend retention or further investigation and
  carry the gap into the roadmap.
- If legal retention, archival access, licensing, operational support, or downstream consumers
  remain unresolved, do not mark decommissioning ready.
- If a target cannot meet required service levels or security constraints, show the shortfall
  even when its business function appears to match.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Authorized users MUST be able to select or describe any input application and
  provide assessment scope, business context, and supporting material.
- **FR-002**: The assessor MUST use available, authorized knowledge-layer and landscape records
  to identify source capabilities, users, data, integrations, dependencies, service obligations,
  and constraints; missing categories MUST be shown as gaps.
- **FR-003**: Every material finding MUST include its evidence reference and status as verified,
  inferred, assumed, disputed, or unknown, with the date of the underlying evidence when known.
- **FR-004**: The assessor MUST compare each source capability with candidate existing or planned
  destinations and classify coverage as full, partial, none, or unverified, including behavior
  and nonfunctional gaps.
- **FR-005**: The assessor MUST identify affected data, interfaces, consumers, controls, and
  operational processes for each proposed capability move.
- **FR-006**: The assessor MUST present at least one viable disposition per source capability:
  retain, migrate, consolidate, replace, or investigate; if no viable destination is evidenced,
  it MUST say so.
- **FR-007**: Each recommended destination MUST include an explanation of expected efficiency
  compared with retaining the current arrangement, covering duplication, change effort, risk,
  operating impact, and business value; uncertain comparisons MUST be labelled.
- **FR-008**: The assessment MUST show alternative viable destinations and the reasons they were
  not preferred, or explicitly state that none were evidenced.
- **FR-009**: Decommission readiness MUST be reported as ready for approval, conditional, or not
  supported; the result MUST check capability parity, dependency disposition, data and retention
  obligations, operational readiness, and accountable owner sign-off.
- **FR-010**: A conditional or unsupported decommission result MUST list blockers and the
  evidence or action required to resolve each blocker. A ready-for-approval result MUST NOT
  trigger retirement automatically.
- **FR-011**: The assessor MUST produce a current-to-target architecture description showing
  capability ownership, data flows, integrations, and transitional states at a level suitable
  for stakeholder review.
- **FR-012**: The roadmap MUST order discovery, gap closure, target preparation, migration,
  parallel validation, cutover, and retirement decision where applicable; each stage MUST include
  prerequisites, owner, expected outcome, validation gate, and fallback.
- **FR-013**: Authorized reviewers MUST be able to add or correct evidence, record a decision,
  and generate a revised assessment with a visible change history.
- **FR-014**: Access to source records and assessment results MUST respect the viewer's
  permissions; restricted evidence MUST not be disclosed in findings or exports.
- **FR-015**: The assessment MUST distinguish generated recommendations from human-approved
  decisions and record the actor and date of each approval or override.
- **FR-016**: Users MUST be able to review and share the assessment, fit matrix, decommission
  status, architecture description, and roadmap as a coherent decision record.

### Key Entities *(include if feature involves data)*

- **Application**: Source or candidate destination, with owner, lifecycle state, and scope.
- **Capability**: Business function and required behavior supplied by an application.
- **Landscape Relationship**: Data flow, integration, consumer, dependency, or ownership link
  between applications and capabilities.
- **Evidence Record**: Source, date, access classification, and finding it supports or disputes.
- **Fit Assessment**: Capability-to-destination coverage, gaps, alternatives, and rationale.
- **Disposition Decision**: Recommended or approved future state, owner, status, and basis.
- **Transition Stage**: Ordered roadmap step, prerequisites, validation gate, and fallback.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In a review set of representative applications with complete source material,
  100% of documented capabilities appear in the fit matrix, including unmatched capabilities.
- **SC-002**: 100% of material recommendations and decommission statuses display supporting
  evidence or a clearly labelled assumption or gap.
- **SC-003**: 100% of decommission results with an unresolved capability, dependency, data, or
  operational obligation are marked conditional or not supported in acceptance review.
- **SC-004**: At least 90% of pilot reviewers can identify the preferred target, its rationale,
  major gaps, and next decision gate from the decision record without additional explanation.
- **SC-005**: For a well-documented sample application, at least 80% of pilot assessors can
  obtain a reviewable fit assessment and staged roadmap within 30 minutes of submission.
- **SC-006**: In pilot review, at least 80% of application owners and architects rate the
  assessment as useful for deciding the next migration step.

## Assumptions

- The feature is advisory. It does not execute migrations, change target systems, or retire an
  application; accountable humans approve those actions in separate processes.
- The existing knowledge layer contains or can be supplied with application inventory,
  capability, dependency, and architecture evidence. Assessment quality depends on its currency.
- The same assessment method applies across application types and industries; examples of named
  applications and target systems do not constrain the feature's scope.
- A destination may be an existing system, a planned system, or continued operation of the input
  application. Planned destinations are labelled as such.
- Efficiency is a reasoned comparison using available evidence, not a guaranteed financial
  saving. Numeric cost estimates require supplied cost data.
- Organization-specific retention, security, service level, and approval policies are supplied
  by the organization and remain authoritative for final decisions.

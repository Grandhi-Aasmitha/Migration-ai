# Tasks: Application Fit Assessment

**Input**: Design documents from `specs/002-assess-application-fit/`

**Prerequisites**: [plan.md](plan.md), [spec.md](spec.md),
[research.md](research.md), [data-model.md](data-model.md),
[contracts/](contracts/), [quickstart.md](quickstart.md)

**Tests**: Automated tests are required by the project constitution.
Write each story's listed tests first and confirm they fail for the intended
missing behavior before implementation.

**Organization**: Tasks follow the four prioritized user stories. Paths
below are repository-relative and refer to files to create or update during
implementation. No application source files are created by this task list.

## Format: `[ID] [P?] [Story] Description`

- **[P]**: Safe to perform in parallel after listed prerequisites are met;
  tasks use different files and do not depend on one another.
- **[Story]**: `[US1]` through `[US4]` map to the specification's user stories.
- Task IDs are in the suggested implementation order. Each task names its
  output file or files.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Establish the planned Python service, React reviewer interface,
and quality commands. No feature behavior is implemented in this phase.

- [ ] T001 Initialize Python 3.13 service metadata, locked dependency workflow, FastAPI and pytest entry points in backend/pyproject.toml.
- [ ] T002 [P] Initialize TypeScript 5 and React scripts for build, lint, and browser tests in frontend/package.json and frontend/tsconfig.json.
- [ ] T003 [P] Define repository-wide text encoding and line-ending rules in .editorconfig.
- [ ] T004 Define validated environment settings for database, identity provider, knowledge adapters, job worker, and policy inputs in backend/app/config.py.
- [ ] T005 Add initial lint, type-check, unit-test, and frontend-build gates in .github/workflows/quality.yml using the commands from backend/pyproject.toml and frontend/package.json.

**Checkpoint**: A clean checkout installs dependencies and runs empty quality
gates without requiring production credentials.

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Establish shared identity, authorization, persistence, and job
contracts used by every story.

**Critical gate**: Complete this phase before starting any user story.

- [ ] T006 Write failing policy tests for actor, action, application scope, ownership, classification, and non-revealing denial in backend/tests/unit/test_access_policy.py.
- [ ] T007 Define opaque IDs, UTC timestamp parsing, classification order, and common result/error types in backend/app/domain/types.py.
- [ ] T008 [P] Define paginated source snapshot operations and explicit not-found, forbidden, stale, unavailable, and version-change outcomes from contracts/knowledge-source.md in backend/app/ports/knowledge.py.
- [ ] T009 [P] Define immutable revision, evidence-reference, decision, and audit persistence operations in backend/app/ports/repository.py.
- [ ] T010 [P] Define durable enqueue, claim, retry, completion, and failure operations for assessment jobs in backend/app/ports/jobs.py.
- [ ] T011 Implement central authorization in backend/app/domain/access.py; derived records MUST inherit the strictest contributing classification and denied records MUST expose only a non-revealing gap.
- [ ] T012 Connect organization identity claims to the policy port and enforce action/object checks at the service boundary in backend/app/api/auth.py.
- [ ] T013 Create PostgreSQL connection and transaction adapter with parameterized reads/writes in backend/app/adapters/persistence.py.
- [ ] T014 Create core tables and constraints for assessment identity, immutable revisions, source references, durable jobs, and audit IDs in backend/migrations/001_core.sql.
- [ ] T015 Implement the durable worker queue adapter with bounded retries, lease recovery, and safe error categories in backend/app/worker/queue.py.
- [ ] T016 Create the FastAPI app, structured redacted error handling, correlation IDs, and authenticated `GET /health` readiness in backend/app/api/app.py.
- [ ] T017 Build non-production landscape fixtures with full, partial, and unmatched capabilities; stale/conflicting/restricted evidence; and an unresolved consumer in backend/tests/fixtures/landscape.py.

**Checkpoint**: Shared contracts, permissions, data transactions, jobs,
health checks, and fixtures work without implementing any story-specific
assessment behavior.

---

## Phase 3: User Story 1 — Understand the Current Application (Priority: P1) 🎯 MVP

**Goal**: Submit a generic input application and receive a reviewable
inventory of capabilities, data, integrations, dependencies, obligations,
evidence provenance, and gaps.

**Independent Test**: With only the test knowledge adapter and a documented
sample application, an authorized user can submit an assessment and see every
source responsibility with verified/inferred/assumed/disputed/unknown status;
conflicting, stale, and inaccessible evidence remains explicit.

### Tests for User Story 1

- [ ] T018 [P] [US1] Write failing adapter contract tests for pagination, snapshot versions, source classification, conflicts, forbidden records, stale records, and unavailable sources in backend/tests/contract/test_knowledge_source.py.
- [ ] T019 [P] [US1] Write failing `POST /assessments` and `GET /assessments/{id}` contract tests for scope validation, idempotency, queued status, and redacted findings in backend/tests/contract/test_assessment_api.py.
- [ ] T020 [P] [US1] Write a failing end-to-end source-investigation test using backend/tests/fixtures/landscape.py in backend/tests/integration/test_current_state.py.
- [ ] T021 [P] [US1] Write a failing browser test for submission, progress, current-state review, and visible evidence gaps in frontend/tests/current-state.spec.ts.

### Implementation for User Story 1

- [ ] T022 [P] [US1] Define Application lifecycle as "Existing, planned, retiring, or retired", Capability behaviors/service requirements, ApplicationCapability, and LandscapeRelationship disposition as "Retain, move, replace, retire, or unresolved" in backend/app/domain/landscape.py.
- [ ] T023 [P] [US1] Define EvidenceRecord provenance and Claim status as "Verified, inferred, assumed, disputed, or unknown"; require current permitted evidence before verified status, use explicit nulls with gap status for unknown dates/owners, retain raw restricted content only when permitted, and preserve contradictory claims in backend/app/domain/evidence.py.
- [ ] T024 [US1] Define Assessment and immutable AssessmentRevision with status "Queued, collecting, evaluating, reviewable, or failed"; keep current_revision_id on Assessment in backend/app/domain/assessment.py.
- [ ] T025 [US1] Implement the knowledge-layer adapter's actor-scoped pagination, stable ID/version mapping, source-specific freshness, and explicit failure outcomes without using broader service permission to disclose records in backend/app/adapters/knowledge_source.py.
- [ ] T026 [US1] Normalize source capabilities, data flows, users, consumers, constraints, and obligations into claims while retaining evidence IDs and missing-category gaps in backend/app/use_cases/collect_evidence.py.
- [ ] T027 [US1] Implement authorized submission, idempotency, snapshot/ruleset freeze, immutable revision persistence, and durable job creation in backend/app/use_cases/start_assessment.py.
- [ ] T028 [US1] Execute collection jobs, record progress and safe failure categories, and make the current-state revision reviewable in backend/app/worker/assess.py.
- [ ] T029 [US1] Expose `POST /assessments` and `GET /assessments/{id}` with the contract's 202/400/401/403/404/409/503 behavior in backend/app/api/assessments.py.
- [ ] T030 [US1] Build submission and current-state screens with evidence status, source date, conflicts, and non-revealing gaps in frontend/src/pages/AssessmentStart.tsx and frontend/src/pages/CurrentState.tsx.

**Checkpoint**: US1 works as a standalone MVP. Verify its tests pass and
that no target-fit recommendation is needed to understand the source.

---

## Phase 4: User Story 2 — Evaluate Target Fit and Disposition (Priority: P2)

**Goal**: Compare each capability to candidate destinations, explain a
preferred disposition and alternatives, and report evidence-based
decommission readiness.

**Independent Test**: Feed a frozen generic source and candidate snapshot
directly into the fit workflow. Full, partial, none, and unverified coverage
are distinct; an unresolved consumer keeps retirement conditional even
when all named capabilities have candidates.

### Tests for User Story 2

- [ ] T031 [P] [US2] Write failing fixture tests for full/partial/none/unverified behavior and service coverage in backend/tests/unit/test_coverage_rules.py.
- [ ] T032 [P] [US2] Write failing tests for all retirement gates, blockers, owner attestation, and no automatic retirement in backend/tests/unit/test_decommission_rules.py.
- [ ] T033 [P] [US2] Write failing decision-record and export contract tests for alternatives, evidence/assumption labels, efficiency factors, and redaction in backend/tests/contract/test_decision_record.py.
- [ ] T034 [P] [US2] Write a failing browser test that distinguishes partial coverage, unmatched capabilities, alternatives, and retirement blockers in frontend/tests/fit-matrix.spec.ts.

### Implementation for User Story 2

- [ ] T035 [US2] Define FitAssessment coverage as "Full, partial, none, or unverified", Recommendation disposition as "Retain, migrate, consolidate, replace, or investigate", DecommissionGate status as "Met, blocked, unverified, or not applicable", and Recommendation displayed status as "Proposed, accepted, rejected, or superseded" derived from HumanDecision/current revision in backend/app/domain/fit.py.
- [ ] T036 [US2] Classify capability fit at behavior, service, and control level; a capability with any unverified required behavior MUST NOT have full coverage in backend/app/domain/coverage_rules.py.
- [ ] T037 [US2] Compare destinations with retain-current baseline across duplication, effort, risk, operating impact, and business value; omit numeric savings without sourced costs in backend/app/domain/recommendation_rules.py.
- [ ] T038 [US2] Derive ready_for_approval only when every applicable parity, consumer/dependency, data, operational, control, and owner-attestation gate is met; otherwise list blockers and next actions in backend/app/domain/decommission_rules.py.
- [ ] T039 [US2] Evaluate all source capabilities, affected data/interfaces/controls/processes, candidate destinations, unmatched gaps, and alternatives from an immutable revision in backend/app/use_cases/evaluate_fit.py.
- [ ] T040 [US2] Assemble a versioned, evidence-linked decision record that distinguishes generated advice from human decisions in backend/app/use_cases/build_decision_record.py.
- [ ] T041 [US2] Add permitted fit and readiness fields to `GET /assessments/{id}` without leaking restricted evidence in backend/app/api/assessments.py.
- [ ] T042 [US2] Implement authorization-filtered `GET /assessments/{id}/export` with partial-view/redaction status in backend/app/api/exports.py.
- [ ] T043 [US2] Render the capability-to-destination matrix, alternatives, and factor-by-factor rationale in frontend/src/pages/FitMatrix.tsx.
- [ ] T044 [US2] Render each decommission gate, blocker, resolution owner, and required evidence without an execute-retirement control in frontend/src/pages/DecommissionReadiness.tsx.

**Checkpoint**: US2 can be tested with fixture snapshots even if the US1
source adapter is unavailable. Once integrated, every documented capability
is represented, including unmatched ones.

---

## Phase 5: User Story 3 — Review a Staged Transition Plan (Priority: P3)

**Goal**: Produce a current-to-target architecture and a roadmap whose
prerequisites, validation gates, owners, and fallbacks are reviewable.

**Independent Test**: Supply a frozen fit assessment with two destinations
and a shared dependency. The roadmap places dependency resolution before
dependent cutovers and shows the current and proposed relationship states.

### Tests for User Story 3

- [ ] T045 [P] [US3] Write failing rule tests for acyclic prerequisites, discovery-to-retirement ordering, cutover validation, and fallback in backend/tests/unit/test_roadmap_rules.py.
- [ ] T046 [P] [US3] Write failing decision-record contract tests for current/target capability ownership, data flows, integrations, and transition states in backend/tests/contract/test_transition_record.py.
- [ ] T047 [P] [US3] Write a failing browser test for architecture and ordered-roadmap review using a fixture decision record in frontend/tests/roadmap.spec.ts.

### Implementation for User Story 3

- [ ] T048 [US3] Define TransitionStage order, phase, capability/relationship scope, prerequisite IDs, owner, expected outcome, validation gate, and fallback; derive displayed state as "Proposed, accepted, or superseded" from HumanDecision/current revision in backend/app/domain/transition.py.
- [ ] T049 [US3] Derive current, target, and coexistence capability ownership plus data and integration relationships from the assessed snapshot in backend/app/use_cases/derive_architecture.py.
- [ ] T050 [US3] Build discovery, gap closure, target preparation, migration, parallel validation, cutover, and retirement-decision stages where applicable; reject cyclic prerequisites and cutovers without fallback in backend/app/use_cases/build_roadmap.py.
- [ ] T051 [US3] Add architecture and roadmap sections to the versioned decision record without treating them as approved changes in backend/app/use_cases/build_decision_record.py.
- [ ] T052 [US3] Display staged roadmap and current-to-target relationships with prerequisites and decision gates in frontend/src/pages/Roadmap.tsx and frontend/src/components/ArchitectureView.tsx.

**Checkpoint**: US3's rule and UI tests pass with a frozen fit fixture. Its
integration into the full decision record can then be verified with US1/US2.

---

## Phase 6: User Story 4 — Challenge and Update the Assessment (Priority: P3)

**Goal**: Let an authorized reviewer correct evidence, reassess, record a
human decision, and compare immutable revisions.

**Independent Test**: Start from a stored reviewable fixture revision, add a
validated evidence reference, and verify a successor revision and diff while
the prior revision remains unchanged. Restricted viewers see only permitted
fields in both revisions and exports.

### Tests for User Story 4

- [ ] T053 [P] [US4] Write failing tests for immutable completed revisions, current pointer movement, changed conclusions, and stale expected revision conflicts in backend/tests/unit/test_revision_diff.py.
- [ ] T054 [P] [US4] Write failing contract tests for evidence correction, reassess, historical revision, and human decision endpoints in backend/tests/contract/test_review_api.py.
- [ ] T055 [P] [US4] Write failing integration tests for restricted evidence redaction on current/historical views and exports, plus append-only audit events in backend/tests/integration/test_review_permissions.py.
- [ ] T056 [P] [US4] Write a failing browser test for adding evidence, comparing revisions, and recording a non-executing decision in frontend/tests/revision-review.spec.ts.

### Implementation for User Story 4

- [ ] T057 [US4] Define append-only HumanDecision and AuditEvent with actor, time, action/outcome, revision, rationale, and redacted change summary in backend/app/domain/review.py.
- [ ] T058 [US4] Add append-only decision/audit tables and constraints to prevent edits to completed revision content in backend/migrations/002_review.sql.
- [ ] T059 [US4] Validate correction permissions and expected revision, preserve the prior revision, freeze a successor snapshot, and enqueue reassessment in backend/app/use_cases/revise_assessment.py.
- [ ] T060 [US4] Calculate added, removed, and changed claims, fit classifications, readiness gates, and roadmap stages between two permitted revisions in backend/app/use_cases/diff_revisions.py.
- [ ] T061 [US4] Record approve/reject/override only for an authorized actor and explicit revision; never invoke migration or retirement in backend/app/use_cases/record_decision.py.
- [ ] T062 [US4] Expose `POST /assessments/{id}/evidence`, `POST /assessments/{id}/reassess`, `GET /assessments/{id}/revisions/{revisionId}`, and `POST /assessments/{id}/decisions` with safe 403/404/409 behavior in backend/app/api/review.py.
- [ ] T063 [US4] Render prior/current findings and changed recommendations with classification-aware redaction in frontend/src/pages/RevisionComparison.tsx.
- [ ] T064 [US4] Render explicit human approval, rejection, or override with rationale and revision identity in frontend/src/pages/DecisionReview.tsx.

**Checkpoint**: US4 passes its tests against a stored fixture without
depending on live knowledge-layer availability.

---

## Phase 7: Polish & Cross-Cutting Concerns

**Purpose**: Integrate the increments, verify the constitution's release
gates, and document the production handoff.

- [ ] T065 [P] Add a full generic journey test covering submission, fit, roadmap, revision, and conditional retirement in backend/tests/integration/test_end_to_end.py.
- [ ] T066 [P] Benchmark 500 applications, up to 100 capabilities and 1,000 evidence references per assessment, and 50 concurrent reviewers against the plan's reviewable-time/read-time goals in backend/tests/performance/test_pilot_envelope.py.
- [ ] T067 Create a reproducible multi-stage OCI build for API and worker code with locked dependencies and a non-root runtime in ops/deployment/Dockerfile.
- [ ] T068 Document health signals, source/worker failure alerts, privacy-safe metrics, and rollback steps in ops/deployment/runbook.md.
- [ ] T069 Require all unit, contract, integration, browser, security, build, and image checks before release in .github/workflows/quality.yml.
- [ ] T070 [P] Add automated checks for missing evidence provenance, restricted export fields, unauthorized object access, and no automatic retirement in backend/tests/security/test_assessment_boundaries.py.
- [ ] T071 Run the scenarios in specs/002-assess-application-fit/quickstart.md and record fixture version, ruleset version, timing, access profiles, SC-001 through SC-006 results, and failures in specs/002-assess-application-fit/validation-results.md.
- [ ] T072 Update README.md with the runnable setup, test, deployment, and rollback commands verified by T071.

**Checkpoint**: All required tests and release gates pass. The system is
still advisory; production integration needs organization-owned source,
security, retention, and platform inputs from research.md.

---

## Dependencies & Execution Order

### Phase Dependencies

- Phase 1 Setup → Phase 2 Foundational → user-story phases.
- Story phases may be implemented in parallel after Phase 2 with fixture
  inputs. Shared aggregate integration follows the priority order
  US1 → US2 → US3; US4 uses a stored revision fixture and may proceed
  alongside US2/US3.
- Phase 7 integration, performance, and release validation follows all
  desired story phases.

### User Story Dependencies

- **US1 (P1)**: Requires Phase 2 only; deployable MVP for source
  investigation.
- **US2 (P2)**: Requires Phase 2; tests use frozen source/candidate
  fixtures. Production integration consumes US1 snapshots.
- **US3 (P3)**: Requires Phase 2; tests use frozen fit fixtures.
  Production decision-record integration follows US2.
- **US4 (P3)**: Requires Phase 2; tests use a stored reviewable revision.
  Production reassessment consumes US1 collection and US2/US3 evaluators.

### Within Each Story

1. Write the story's automated tests and confirm they fail for missing
   behavior.
2. Implement domain models and validation rules.
3. Implement adapters and use cases.
4. Expose API operations and reviewer UI.
5. Run unit, contract, integration, and browser checks before marking the
   story complete.

### Parallel Opportunities

- T001, T002, and T003 use independent setup files and can proceed together.
- After T007, T008, T009, and T010 can proceed in parallel.
- Within each story, the [P] test tasks target different files and may be
  authored in parallel before implementation.
- T022 and T023 are independent domain model files after US1 tests.
- T065, T066, and T070 target separate verification files after story
  completion.

## Parallel Example: User Story 1

```text
After Phase 2:
  T018 contract test -> backend/tests/contract/test_knowledge_source.py
  T019 API test      -> backend/tests/contract/test_assessment_api.py
  T020 journey test  -> backend/tests/integration/test_current_state.py
  T021 browser test  -> frontend/tests/current-state.spec.ts
Then, after those tests fail for the intended missing behavior:
  T022 landscape entities -> backend/app/domain/landscape.py
  T023 evidence entities  -> backend/app/domain/evidence.py
```

## Parallel Example: User Story 2

```text
After Phase 2 with frozen fit fixtures:
  T031 coverage tests     -> backend/tests/unit/test_coverage_rules.py
  T032 retirement tests   -> backend/tests/unit/test_decommission_rules.py
  T033 decision contract  -> backend/tests/contract/test_decision_record.py
  T034 reviewer UI test   -> frontend/tests/fit-matrix.spec.ts
```

## Parallel Example: User Story 3

```text
After Phase 2 with frozen fit fixtures:
  T045 roadmap rules      -> backend/tests/unit/test_roadmap_rules.py
  T046 transition record  -> backend/tests/contract/test_transition_record.py
  T047 roadmap UI         -> frontend/tests/roadmap.spec.ts
```

## Parallel Example: User Story 4

```text
After Phase 2 with a stored reviewable revision fixture:
  T053 revision rules     -> backend/tests/unit/test_revision_diff.py
  T054 review contract    -> backend/tests/contract/test_review_api.py
  T055 permission journey -> backend/tests/integration/test_review_permissions.py
  T056 review UI          -> frontend/tests/revision-review.spec.ts
```

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1 and Phase 2.
2. Complete US1 tests, domain, adapter, workflow, API, and reviewer screens.
3. Validate US1 against its independent test without waiting for fit,
   roadmap, or revision features.
4. Demonstrate the current-state evidence and gaps as the first useful
   increment.

### Incremental Delivery

1. Add US2 to compare targets and assess retirement conditions.
2. Add US3 to produce transition architecture and the staged roadmap.
3. Add US4 to support corrections, human decisions, and revision diffs.
4. Complete Phase 7, validate SC-001 through SC-006, and apply release
   gates before production use.

### Parallel Team Strategy

After the foundation is complete, separate contributors may build US2 fit
rules, US3 roadmap rules, and US4 revision logic against frozen fixtures.
Integrate their shared decision-record behavior in priority order and run
the full journey before release. Do not merge partial work that bypasses
permission or evidence gates.

## Notes

- `[P]` means different files and no unfinished direct dependency within
  the stated phase; it is not permission to skip tests or gates.
- The plan's reference stack and pilot scale are assumptions until
  organization owners confirm platform and source details.
- `specs/001-assess-application-fit/plan.md` is a separate unfilled local
  template and is not part of this feature.

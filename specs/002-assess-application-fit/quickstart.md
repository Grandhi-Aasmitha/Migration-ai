# Quickstart: Validate Application Fit Assessment

**Purpose**: Run the feature's end-to-end acceptance scenarios after the
application is implemented. This repository currently contains design
artifacts only; these commands define the validation target and are not
expected to succeed until the service and test landscape exist.

**References**: [assessment contract](contracts/assessment-api.md),
[knowledge source contract](contracts/knowledge-source.md),
[decision record contract](contracts/decision-record.md),
[data model](data-model.md).

## Prerequisites

1. A non-production instance of the assessment service is running with
   `GET /health` reporting ready, and its configured knowledge adapter
   points to a test landscape.
2. The test landscape contains a generic input application
   `pilot-source`, destinations `pilot-target-a` and
   `pilot-target-b`, and three source capabilities:
   - one with verified full coverage at target A;
   - one with partial coverage at target B;
   - one with no evidenced destination.
3. The landscape includes an unresolved downstream consumer, stale claim
   `claim-stale`, two conflicting claims, one restricted evidence record,
   and corrected evidence `pilot-evidence-updated` at version `2`.
4. Two test identities are available: an authorized assessor and a reviewer
   who cannot access the restricted evidence. Set their tokens in environment
   variables; do not place tokens in this repository or command output.
5. Organization policies and source freshness rules for the fixture are
   configured. The test is isolated from production records.

## Setup and smoke check

In PowerShell, set the instance URL and authorized test token in the
environment, then run:

```powershell
$baseUrl = $env:ASSESSMENT_BASE_URL
$token = $env:ASSESSMENT_TEST_TOKEN
if (-not $baseUrl -or -not $token) { throw "Set ASSESSMENT_BASE_URL and ASSESSMENT_TEST_TOKEN" }
$headers = @{ Authorization = "Bearer $token"; "Idempotency-Key" = "pilot-fit-001" }
Invoke-RestMethod -Method Get -Uri "$baseUrl/health" -Headers $headers
```

**Expected**: The health response reports ready. A failed readiness check
blocks acceptance testing and release.

## Scenario 1: Assess and inspect fit

```powershell
$body = @{
  application = @{ id = "pilot-source" }
  scope = @{ capabilities = "all"; landscape_boundary = "pilot" }
  evidence_refs = @()
} | ConvertTo-Json -Depth 5
$created = Invoke-RestMethod -Method Post -Uri "$baseUrl/assessments" -Headers $headers -ContentType "application/json" -Body $body
$created
$deadline = (Get-Date).AddMinutes(10)
do {
  $current = Invoke-RestMethod -Method Get -Uri "$baseUrl/assessments/$($created.assessment_id)" -Headers $headers
  if ($current.status -in @("reviewable", "failed")) { break }
  Start-Sleep -Seconds 2
} while ((Get-Date) -lt $deadline)
if ($current.status -ne "reviewable") { throw "Assessment did not become reviewable" }
$current
```

**Expected**: Creation returns an assessment and queued revision. The
current result becomes `reviewable` within the test window. All three
capabilities appear; coverage is full,
partial, and none respectively. The partial result lists missing behavior.
The stale and conflicting claims retain separate provenance and do not
become verified by summary text.

## Scenario 2: Check recommendation and decommission gates

Read the decision record from Scenario 1.

**Expected**: It compares the preferred destination with alternatives
across duplication, effort, risk, operating impact, and business value. It
labels unsupported cost estimates. The unresolved consumer and unmatched
capability prevent `ready for approval` decommission readiness. The
record names blockers, owners, and validation actions. Its roadmap places
dependency resolution and parallel validation before cutover, with owners
and fallbacks.

## Scenario 3: Check restricted access and export

Set `ASSESSMENT_RESTRICTED_TEST_TOKEN` to the limited reviewer's token.
Request the same assessment and export with that token:

```powershell
$limitedHeaders = @{ Authorization = "Bearer $env:ASSESSMENT_RESTRICTED_TEST_TOKEN" }
Invoke-RestMethod -Method Get -Uri "$baseUrl/assessments/$($created.assessment_id)" -Headers $limitedHeaders
Invoke-RestMethod -Method Get -Uri "$baseUrl/assessments/$($created.assessment_id)/export" -Headers $limitedHeaders
```

**Expected**: The limited viewer sees a non-revealing restricted-evidence
gap, with no restricted values, source name, or direct link. The export
applies the same or stricter redaction and identifies itself as partial.
Unauthorized mutations return `403`; inaccessible assessment IDs do
not reveal whether a hidden assessment exists.

## Scenario 4: Revise evidence without changing history

Using the authorized assessor, submit a corrected evidence reference through
`POST /assessments/{id}/evidence` with the current revision ID, affected
claim, reason, and source reference. Then retrieve both revision IDs.

```powershell
$current = Invoke-RestMethod -Method Get -Uri "$baseUrl/assessments/$($created.assessment_id)" -Headers $headers
$correction = @{
  affected_claim_id = "claim-stale"
  evidence_ref = @{ source_id = "pilot-landscape"; record_id = "pilot-evidence-updated"; version = "2" }
  reason = "Validated source correction"
  expected_revision_id = $current.revision_id
} | ConvertTo-Json -Depth 5
$revised = Invoke-RestMethod -Method Post -Uri "$baseUrl/assessments/$($created.assessment_id)/evidence" -Headers $headers -ContentType "application/json" -Body $correction
Invoke-RestMethod -Method Get -Uri "$baseUrl/assessments/$($created.assessment_id)/revisions/$($current.revision_id)" -Headers $headers
Invoke-RestMethod -Method Get -Uri "$baseUrl/assessments/$($created.assessment_id)/revisions/$($revised.revision_id)" -Headers $headers
```

**Expected**: A successor revision is created. The original revision
remains unchanged, and the decision record shows the changed claims, fit,
readiness, and roadmap stages. A stale expected revision returns `409`.
Any human approval refers to one explicit revision and does not execute a
migration or retirement.

## Acceptance measures

- Count documented fixture capabilities against fit rows: **100%**,
  including unmatched ones (SC-001).
- Review all material recommendations and readiness statements: **100%**
  show evidence, labelled assumptions, or gaps (SC-002).
- Run blocked-gate fixtures: **100%** remain conditional or not supported
  (SC-003).
- In pilot review, at least **90%** of reviewers identify the target,
  rationale, gaps, and next gate without explanation (SC-004).
- At least **80%** of pilot assessors complete submission through
  reviewable roadmap within **30 minutes** (SC-005).
- At least **80%** of owners and architects rate the decision record
  useful for the next migration decision (SC-006).

Record the fixture version, ruleset version, test identities' access profiles,
timings, reviewer results, and any failed checks in the acceptance report.

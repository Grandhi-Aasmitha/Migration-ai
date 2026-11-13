# Contract: Reviewable Decision Record

**Version**: 1 (output contract)
**Related**: [data-model.md](../data-model.md),
[assessment-api.md](assessment-api.md)

The decision record is a coherent, versioned view of one assessment
revision. It is a proposal until an accountable human records a decision.
It must remain understandable without access to implementation details.

## Required sections

| Section | Required content |
|---|---|
| Context | Input application, scope, owner, revision, evidence snapshot, ruleset version, creation time |
| Current state | Capabilities, users, data, interfaces, consumers, dependencies, obligations, and material gaps |
| Fit matrix | Each source capability, candidate destinations, full/partial/none/unverified coverage, behavior and service gaps |
| Recommendation | Preferred disposition, alternative options or explicit none, efficiency factors, trade-offs, assumptions, and evidence |
| Decommission readiness | Ready for approval, conditional, or not supported; every gate, blocker, resolution owner, and required evidence |
| Transition architecture | Current and target capability ownership, data flows, integrations, and temporary coexistence |
| Roadmap | Ordered stages, prerequisites, owners, expected outcomes, validation gates, and fallbacks |
| Decision history | Human approvals, rejections, overrides, prior revision reference, and changed conclusions |

Every material statement identifies whether it is verified, inferred,
assumed, disputed, or unknown and includes an accessible evidence reference
or explicit gap. A source title is not sufficient evidence by itself; the
record must identify a stable source record and version or digest.

## Comparison rules

- A candidate with a matching capability name but unverified required
  behaviors is `unverified`, not `full`.
- Efficiency is a factor-by-factor comparison against retaining the
  existing arrangement. Unsupported numeric savings are omitted.
- Planned target applications are visibly labelled planned and cannot
  satisfy an operational-readiness gate until validated.
- If no candidate is evidenced, the record says so and recommends retention
  or investigation rather than forcing migration.
- Unresolved dependencies, consumers, data obligations, controls, or owner
  attestation prevent a `ready for approval` decommission result.

## Access and export rules

A record is rendered for the viewer's permissions. Restricted evidence
cannot appear in text, links, diagrams, downloadable exports, or change
summaries. The record may show "restricted evidence gap" without revealing
the source's name or value. Every export includes the revision and redaction
status so recipients do not mistake a partial view for complete evidence.

## Version and decision rules

Assessment revisions are immutable. New evidence produces a new revision
with an explicit diff: added, removed, and changed findings; changed fit
classifications; changed readiness gates; and changed roadmap stages.
Human decisions refer to a specific revision and record actor, time, and
rationale. The record never claims that a generated roadmap has been
approved merely because it exists.

## Acceptance checks

1. Every documented source capability appears in the fit matrix, including
   unmatched ones (SC-001).
2. Every recommendation and decommission status has evidence, labelled
   assumptions, or gaps (SC-002).
3. A blocked retirement gate makes readiness conditional or not supported
   (SC-003).
4. A reviewer can identify preferred target, rationale, major gaps, and next
   decision gate from one record (SC-004).

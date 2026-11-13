# Contract: Existing Knowledge Source

**Version**: 1 (adapter contract)

**Consumer**: Assessment application service
**Purpose**: Read authorized landscape evidence without binding the domain
engine to a particular inventory, knowledge graph, document store, or vendor.

## Required capabilities

| Capability | Request | Result |
|---|---|---|
| Resolve application | Actor context, application ID or search description | Stable application record or explicit not-found/forbidden |
| List capabilities | Actor context, application ID, cursor, snapshot token | Paged capability implementations and behavior statements |
| List relationships | Actor context, application ID, cursor, snapshot token | Paged data flows, integrations, consumers, dependencies, and owners |
| Get evidence | Actor context, source record ID and version | Permitted evidence reference, metadata, and allowed fields |
| Read source policy | Source ID | Freshness, classification, retention, and attribution rules |

Each source adapter must publish an owner, coverage statement, contract
version, and supported operations. If a source cannot provide a stable
snapshot token, it must report that limitation so the assessment marks
cross-page consistency as unverified.

## Record envelope

Each returned record contains:

- `source_id`, `record_id`, and `record_version` or digest;
- `record_type`, normalized subject ID, and source-specific locator;
- `source_owner_id`, `observed_at`, `effective_at`, and `retrieved_at`
  where known;
- `classification`, permitted actions, and any field-level restrictions;
- normalized facts and relationship references;
- `snapshot_token` and next-page cursor where pagination applies.

Unknown dates or owners are explicit nulls with gap status. An absent field
is never interpreted as a negative claim.

## Access and provenance

The adapter receives an actor and authorized assessment scope. It must not
use a broader service permission to disclose a record the actor cannot see.
The application service enforces authorization again before persisting a
derived claim and before returning or exporting it. A derived claim inherits
the strictest contributing classification.

The adapter must preserve source version and provenance, not just a summary.
When two sources disagree, it returns both claims with distinct references.
The domain engine decides status and required validation; adapters do not
silently pick a winner.

## Failure outcomes

| Outcome | Required treatment |
|---|---|
| Not found | Mark inventory or relationship gap; do not invent an application |
| Forbidden | Show a non-revealing restricted-evidence gap |
| Unavailable or timeout | Retry within a bounded policy, then mark affected findings unverified |
| Stale | Retain source record and date, mark validation needed under that source's freshness policy |
| Version changed during paging | Restart snapshot once or mark consistency unverified |
| Invalid mapping | Quarantine affected record, report safe adapter error and owner |

An inaccessible or failed source cannot prove full capability parity or a
decommission gate. Failure details in logs contain IDs and safe error
categories, never raw restricted content.

## Contract tests

Every adapter must pass common fixtures for full and partial pagination,
classification propagation, source version changes, conflicting claims,
forbidden records, stale records, unavailable source, and invalid mapping.
These tests prove behavior against the contract without assuming a single
knowledge-layer product.

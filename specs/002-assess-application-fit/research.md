# Research: Application Fit Assessment

**Feature**: [spec.md](spec.md)
**Date**: 2026-11-13

## Decision 1: Reference architecture

**Decision**: Use a modular monolith with a pure assessment domain, application use cases,
ports for external capabilities, and adapters for the knowledge layer, identity service, and
storage. Keep the reviewer interface separate from domain rules. Run long assessments in a
worker using the same domain package and a durable work queue.

**Rationale**: The feature has one closely related decision workflow and needs consistent
evidence handling. Explicit boundaries support replacement of landscape sources without
rewriting assessment logic. A durable job avoids browser request timeouts and preserves
progress and failures.

**Alternatives considered**: Microservices add cross-service consistency and deployment
overhead before independent scaling is demonstrated. A single request-response process is
simpler but cannot reliably handle slow or unavailable knowledge sources. A graph database
may become useful for complex dependency traversal, but relational relationships are enough
for the initial scope.

**References**: [FastAPI dependency boundaries](https://fastapi.tiangolo.com/tutorial/dependencies/),
[Docker build guidance](https://docs.docker.com/build/building/best-practices/).

## Decision 2: Provisional implementation stack

**Decision**: Plan a Python 3.13 service with FastAPI, a TypeScript/React reviewer interface,
PostgreSQL for operational records, and pytest for domain, adapter, and service tests.
Select organization-supported patch releases during implementation. Treat these as a
reference stack, subject to organization platform standards before work begins.

**Rationale**: Python fits evidence normalization and deterministic decision processing;
FastAPI provides explicit dependencies and contract documentation. React supports fit
matrix review and assessment revision comparison. PostgreSQL stores related records,
immutable revisions, and audit references. This is one service and one database, not a
distributed data platform.

**Alternatives considered**: A server-rendered interface reduces frontend complexity but
limits interactive comparison. Spring Boot is a suitable replacement where it is the
organization's standard. A dedicated vector store or graph store has no demonstrated need
in the specification.

**References**: [FastAPI dependencies](https://fastapi.tiangolo.com/tutorial/dependencies/),
[FastAPI testing](https://fastapi.tiangolo.com/tutorial/testing/),
[PostgreSQL row security](https://www.postgresql.org/docs/17/ddl-rowsecurity.html),
[React UI composition](https://react.dev/learn/describing-the-ui).

## Decision 3: Knowledge-layer integration

**Decision**: Read authorized, paginated records through a versioned
`KnowledgeSource` contract. Each adapter maps source identifiers, versions, timestamps,
owners, classification, capabilities, and relationships to a common snapshot model.
The assessor stores references and the minimum approved derived data needed to reproduce
a decision. Each source declares not-found, forbidden, stale, and unavailable outcomes.

**Rationale**: The existing knowledge layer is unspecified and may contain several systems.
An adapter contract resolves that uncertainty without claiming a vendor or schema. It
prevents full replication and makes access restrictions visible to the assessment.

**Alternatives considered**: Direct queries from domain rules couple decisions to one
source. Copying the entire knowledge layer creates access and freshness drift. Free-text
retrieval alone cannot establish capability parity or permission provenance.

**Reference**: [W3C provenance model](https://www.w3.org/TR/prov-o/).

## Decision 4: Evidence, conflicts, and revisions

**Decision**: Represent each material finding as a claim with status
`verified`, `inferred`, `assumed`, `disputed`, or `unknown`; record source ID,
version, evidence date, assessment revision, derivation, and classification. Keep
contradictory claims side by side. Apply source-specific freshness policies. Freeze
the evidence snapshot and ruleset version for every completed assessment revision.

**Rationale**: A recommendation must be reproducible and reviewable after the landscape
changes. A single global freshness cutoff would misclassify sources with different
update cycles. Material stale, disputed, or inaccessible claims cannot prove full parity.

**Alternatives considered**: Overwriting findings loses disagreement and audit history.
Treating a model-generated summary as verified evidence violates the constitution.
Implementing a full provenance ontology would add complexity without current value.

**Reference**: [W3C PROV-O](https://www.w3.org/TR/prov-o/).

## Decision 5: Recommendation and retirement rules

**Decision**: Use deterministic, versioned rules for coverage classification,
disposition, efficiency comparison, and decommission readiness. A generative model may
summarize or suggest questions, but its output cannot by itself mark a fact verified,
prove parity, or approve retirement. "Ready for approval" requires evidenced capability
coverage, dependency and consumer disposition, data retention and retrieval, operational
handover, required control checks, and accountable owner attestation. Actual retirement
is outside this system.

**Rationale**: The decision record must explain why a target is preferred and identify
every blocking obligation. Human approval is a separate state from generated advice.

**Alternatives considered**: Automatic retirement would exceed the advisory scope.
Numeric savings without supplied cost data would imply unsupported precision.

**References**: [AWS application retirement guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/migration-app-retirement-best-practices/introduction.html),
[Microsoft source workload decommission guidance](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/migrate/decommission-source-workload).

## Decision 6: Authorization and audit

**Decision**: Use the organization's identity provider. Enforce authorization for every
record read, mutation, and export using actor role plus record classification and
ownership attributes. Derived findings inherit the most restrictive relevant source
classification. A denied source appears only as a non-revealing evidence gap. Record
append-only audit events with actor, time, action, outcome, revision, and redacted
change summary; keep raw restricted evidence out of logs.

**Rationale**: Role checks alone do not protect individual applications, findings, or
properties. Reviewing and exporting derived material must not bypass source permissions.

**Alternatives considered**: UI-only filtering and a single shared service identity are
insufficient. PostgreSQL row security may add defense in depth but does not replace
application-level authorization.

**References**: [NIST ABAC guidance](https://csrc.nist.gov/pubs/sp/800/162/upd2/final),
[OWASP object authorization risk](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/),
[NIST security controls](https://csrc.nist.gov/pubs/sp/800/53/r5/upd1/final).

## Decision 7: Validation and deployment

**Decision**: Verify the rule engine with fixture tests, knowledge adapters with
contract tests, persistence and permissions with integration tests, and the reviewer
flow with browser tests. Build a versioned OCI image from locked dependencies,
run quality and security checks before release, and deploy with health checks,
observability, and a documented rollback. Choose the hosting platform and
organization-specific retention periods before implementation deployment.

**Rationale**: This meets the constitution's test and release gates without assuming an
enterprise cloud provider or CI product. Hosting and retention details are configuration
and policy inputs, not unresolved feature behavior.

**Alternatives considered**: A vendor-specific deployment design would invent an
unprovided landscape constraint. Manual-only testing would leave permission and
decision regressions undetected.

**References**: [FastAPI testing](https://fastapi.tiangolo.com/tutorial/testing/),
[Docker build best practices](https://docs.docker.com/build/building/best-practices/).

## Design inputs required before production

- Knowledge-layer owners must supply source contracts, classification mappings, and
  freshness expectations for each connected source.
- Security and records owners must supply identity claims, retention policies,
  export restrictions, and approval roles.
- Platform owners must confirm runtime support, traffic and data volumes, hosting
  environment, monitoring, and rollback procedure.
- These inputs select adapters and configuration; they do not change the generic
assessment method or permit unevidenced decommission claims.

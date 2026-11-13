# AI Application Migration Architect — High Level Architecture

**Purpose**: Discover suitable target applications for an input application,
explain capability fit and gaps, assess decommission readiness, and produce a
reviewable migration blueprint. This is an advisory design, not an automated
migration or retirement tool.

```mermaid
flowchart TB
    Architect["Enterprise architect workspace<br/>Enter source application, scope, and evidence"]
    Knowledge["Existing enterprise knowledge layer<br/>Documents · application inventory · relationships · existing APIs"]
    Identity["Organization identity and access policies"]
    API["Migration architect API<br/>Assessment management and authorized review"]
    Workflow["Migration assessment orchestrator<br/>Coordinates evidence, analysis, revisions, and outputs"]

    Architect --> API
    Identity -. "access rules" .-> API
    API --> Workflow
    Knowledge -->|"permitted landscape evidence"| Workflow

    subgraph Intelligence["Orchestrated migration intelligence — logical assessment stages"]
        direction LR
        S1["1. Source<br/>analysis"] --> S2["2. Target<br/>discovery"]
        S2 --> S3["3. Capability<br/>fit"]
        S3 --> S4["4. Dependency<br/>and gap analysis"]
        S4 --> S5["5. Architecture<br/>comparison"]
        S5 --> S6["6. Solution<br/>architecture"]
        S6 --> S7["7. Migration<br/>roadmap"]
    end

    Workflow --> S1
    S7 --> Gates["Evidence validation gates<br/>Coverage · constraints · cost/risk · decommission blockers"]
    Gates --> Blueprint["Proposed migration blueprint<br/>Target applications · target architecture · gaps · roadmap · validation · fallback"]
    Blueprint --> Review["Human architect review<br/>Approve, reject, or request revision"]
    Review -. "corrections and decisions" .-> API

    classDef knowledge fill:#e6f7e9,stroke:#4c9b70,color:#173d27,stroke-width:2px;
    classDef workspace fill:#d9ecff,stroke:#3377ae,color:#173d61,stroke-width:2px;
    classDef workflow fill:#dceaff,stroke:#5480bd,color:#1d355d,stroke-width:2px;
    classDef stage fill:#f2e9fb,stroke:#9a74bb,color:#40265a,stroke-width:1.5px;
    classDef gate fill:#fff0d2,stroke:#bb8c2b,color:#5c3d0e,stroke-width:2px;
    classDef output fill:#d4ecff,stroke:#2579ae,color:#123e60,stroke-width:2px;

    class Knowledge knowledge;
    class Architect,Identity,API workspace;
    class Workflow workflow;
    class S1,S2,S3,S4,S5,S6,S7 stage;
    class Gates,Review gate;
    class Blueprint output;
```

## How to read this diagram

1. The architect supplies an application and assessment scope. The API
   authorizes the request using organization identity and access policies.
2. The workflow uses permitted facts from the existing enterprise knowledge
   layer. Missing, stale, conflicting, or restricted evidence remains
   visible as a safe gap instead of being treated as verified.
3. The seven stages are **logical steps within one assessment workflow**.
   The plan does not require seven separately deployed AI agents.
4. Rules compare source capabilities with possible target applications,
   explain alternatives and efficiency, check dependency and data
   obligations, and produce a staged target architecture.
5. The blueprint is a **proposal**. A human architect reviews the evidence
   and records approval, rejection, or a request for revision. The system
   does not execute migration or decommissioning.

## Intentional simplifications

- The separate knowledge adapter and its two arrows from the reference
  image are omitted. The implementation plan still requires a permission
  aware source contract behind the logical knowledge input.
- The migration-specific operational store box is omitted. The current
  plan still calls for persistent assessment revisions, jobs, decisions,
  and audit records; this diagram hides that internal detail.
- Product names such as Confluence, Neo4j, Entra ID, and Spring Boot are
  examples from the reference image, not required integrations in the
  generalized specification.

See the [feature specification](../specs/002-assess-application-fit/spec.md),
[implementation plan](../specs/002-assess-application-fit/plan.md), and
[decision record contract](../specs/002-assess-application-fit/contracts/decision-record.md)
for the detailed behavior.

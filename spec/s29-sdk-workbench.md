# §29 SDK & Workbench Architecture

This specification section defines implementation contracts for the Decision Lineage Protocol substrate.

The Decision Lineage Protocol defines nineteen primitives, ten behavioral invariants, seventy-nine SHACL shapes, seventy-two cross-cutting operations, forty-four query patterns, and three substrate profiles. §26 specifies the logical model. §27 specifies the operation catalog. §28 specifies the query architecture. This section specifies how those layers are consumed: the SDK that programmatic clients use to interact with the substrate, and the Workbench that human operators use to govern through it.

The SDK provides typed access to primitives, operations, and queries through three composable layers. The Workbench provides the governance user interface — organized not by screen or workflow, but by governance concern. Together, they constitute the development and governance experience for the DLP substrate.

---

## §29.1 Architecture Overview

### §29.1.1 SDK Architecture

The SDK exposes the substrate through three layers. Each layer builds on the one below it; each layer is independently usable.

#### Table 29.1.1: SDK Layer Architecture

| Layer | Purpose | Consumes | Produces |
|---|---|---|---|
| **Core** | Direct access to the logical model and operations | §26 typed primitives, §27 operation contracts, §5 invariant enforcement | Primitive CRUD with pre-write validation, atomic and composite operations, invariant violation reports |
| **Query** | Access to the query pattern library | §28 named patterns, parameterized templates, return structure types | Typed query results with pagination, graph traversals, aggregate computations |
| **Extension** | Application-specific behavior at documented boundaries | Core and Query layers, extension point contracts | Custom content packages, governance rules, query patterns, composite operations, event subscriptions |

The Core layer is the SDK's foundation. Every operation passes through it. The Query layer adds read-path sophistication — named patterns with typed parameters and return structures that abstract the dual-language query engine (§28). The Extension layer provides the mechanisms through which applications customize the substrate for domain-specific governance.

### §29.1.2 MCP Server Integration

The MCP Server (§28) is the primary AI agent interface. The SDK wraps MCP tools for programmatic access while preserving the same operation semantics and return structures. The relationship is architectural:

- MCP tools expose thirty-five query patterns and operation endpoints to AI agents through natural-language parameter descriptions.
- The SDK Query layer exposes the same thirty-five patterns plus nine SDK-only internal patterns through typed method signatures.
- Both interfaces produce identical return structures (§28). A query executed through MCP and the same query executed through the SDK Query layer return structurally equivalent results.

The MCP Server is not a separate system. It is an interface projection of the SDK — a translation layer that maps MCP tool invocations to SDK method calls.

### §29.1.3 Workbench Architecture

The Workbench is the governance user interface — the human interface to the substrate. It is organized by governance concern, not by application workflow. Three component classes serve three governance scopes:

#### Table 29.1.2: Workbench Component Classes

| Class | Scope | Available To | Components |
|---|---|---|---|
| **Core** | Every substrate interaction | All profiles (EAS, BAS, PAS) | Primitive browser, signal capture, IQ capture, decision surface |
| **Governance** | Governance activation, conformance, and knowledge integration | EAS and BAS profiles | Governance activation dashboard, conformance dashboard, authority browser, knowledge integration dashboard |
| **Enterprise** | KPI management, orchestration, portfolio oversight | EAS profile only | KPI dashboard, orchestration monitor, portfolio browser |

Component availability is profile-gated (§29.13). Components activate when the substrate instance reaches the profile tier and graduation stage that requires them. No component renders data the profile cannot produce.

Each Workbench component consumes query patterns from §28 and renders operation results from §27. The component does not contain governance logic. It renders governance state and captures human actions that the SDK Core layer executes as operations. The Workbench is a rendering and interaction layer over the SDK.

---

## §29.2 SDK Core Layer

The Core layer provides typed access to the nineteen primitives and seventy-two cross-cutting operations with integrated invariant enforcement. Every write operation passes through the two-pass SHACL validation engine (§5.3) before persistence.

### §29.2.1 Typed Primitive Access

The SDK exposes each of the nineteen primitive types as a typed record structure. The nine Tier 1 record types (IntentRecord, CommitmentRecord, CapacityRecord, WorkRecord, EvidenceRecord, DecisionRecord, AuthorityRecord, AccountRecord, ConstraintRecord) carry the common metadata fields defined in §28: `id`, `instance_id`, `truth_type`, `verification_state`, `record_lifecycle_state`, `created_at`, `created_by`, `version`. Each type extends common fields with its MVR fields (§7).

#### Table 29.2.1: Core Layer Primitive Operations

| Operation Class | Behavior | Invariant Enforcement |
|---|---|---|
| **Create** | Instantiate a new primitive record with MVR field validation. Pre-write Pass 1 SHACL validation (always Blocking). | B1–B6 Pass 1 shapes fire on applicable types. B7 schema pass verified at deployment. |
| **Read** | Retrieve a primitive record by identifier. Truth type and lifecycle state metadata included. | None — reads do not modify state. |
| **Update** | Modify mutable fields on an existing record. Progressive immutability enforced: fields immutable from their assignment point cannot be modified. | Pass 1 re-validated on write. Monotonic truth type promotion enforced (Derived → Declared → Authoritative; no demotion). |
| **Supersede** | Create a new version that supersedes the current record. Original preserved with `record_lifecycle_state = superseded`. | Supersession creates Evidence (Authoritative) recording the supersession event. |

### §29.2.2 Operation Contracts

The Core layer exposes the seventy-two cross-cutting operations (§27) as typed method contracts. Each operation specifies:

- **Input parameters** — typed, validated against the operation's preconditions.
- **Primitive composition** — which primitives the operation creates, reads, or modifies, and in what sequence.
- **Authorization** — what authority is required, whether human-only or delegable to AI actors.
- **Invariant implications** — which behavioral invariants (B1–B10) are triggered by this operation.
- **Return structure** — the typed result object produced on success, or the validation error on failure.

Operations compose atomically. A `signal.create` operation creates a Signal record, an Evidence record (truth type Authoritative for the observation act), and routes the signal to the authority chain (B8) — all within a single transactional boundary. If any step fails validation, no records are persisted.

#### Table 29.2.3: Operation Domain Summary

| Domain | Operation Count | Key Operations | Primary Invariants |
|---|---|---|---|
| Signal (§14, §15) | 8 | `signal.create`, `signal.route`, `signal.escalate`, `signal.resolve`, `signal.dismiss` | B7 (all objects flaggable), B8 (signals route to authority) |
| IQ (§16) | 9 | `iq.create`, `iq.assign`, `iq.investigate`, `iq.resolve`, `iq.promote`, `iq.abandon` | B9 (resolution creates Decision), B4 (resolution Decision requires Account) |
| VP (§17) | 8 + 9 dispositions | `vp.materialize`, `vp.mark_stale`, `vp.recompute`, `vp.invalidate` | B3 (VP truth type always Derived) |
| Governance Activation (§12) | 6 | `gov_activation.discover_profile` through `gov_activation.propagate_changes` | B1, B2 (closure Work/Commitment), B6 (activated constraints bind) |
| Orchestration (§A1, §A2) | 11 | `orchestration.classify` through `orchestration.emit`, `aicar.classify_work`, `delegation.compose` | B4 (routing Decisions require Account), B5 (delegation chain valid) |
| KPI & Measurement (§28 Domain J) | 5 | `kpi.introduce`, `kpi.retire`, `kpi.monitor_invariant_signals`, `kpi.review_implications` | B4 (introduction/retirement Decisions require Account) |
| Profile & Portfolio (§21, §23) | 12 | `profile.create_instance`, `profile.trigger_graduation`, `portfolio.spawn_instance`, `portfolio.tighten_constraint` | B5 (authority chain), B6 (constraint binding in cascade) |
| Graduation (§8, §A1) | 3 | `graduation.advance`, `graduation.auto_regress`, `graduation.human_regress` | B4 (advancement/regression Decisions require Account) |
| Agent Behavioral (SM-6) | 5 | `agent.context_load`, `agent.navigate`, `agent.monitor`, `agent.execute`, `agent.alert` | B3 (all AI outputs Derived), B5 (delegation verified), B7/B8 (alerts create signals) |
| Ontology Alignment (§18, §19) | 5 | `ontology.preload`, `ontology.lazy_discover`, `ontology.align`, `ontology.bridge`, `ontology.graduate_concept` | B4 (graduation Decision requires Account) |

### §29.2.3 Invariant Enforcement Integration

The SDK Core layer integrates the two-pass SHACL validation engine as a mandatory pre-write gate.

#### Table 29.2.2: Validation Pass Integration

| Pass | Shapes | Timing | SDK Behavior |
|---|---|---|---|
| **Pass 1: SHACL Core** | B1–B4, B5-immediate, B6-binding, B8-core (7 shapes) | Synchronous, pre-write | Operation rejected with `ValidationError` containing shape URI, focus node, violation detail. Always Blocking across all profiles. |
| **Pass 2: SHACL-SPARQL** | B5-T, B6-U, B8-routing, B9 (4 shapes) + 62 extension shapes | Configurable: per-operation, batch, or periodic | Enforcement mode determined by profile configuration. Violation produces `ConformanceResult` with enforcement action (blocked, warned, or logged). |
| **Schema Pass** | B7 (1 shape) | Deployment-time | SDK initialization verifies signal attachment surfaces exist for all 19 primitive classes. Deployment fails if schema pass does not pass. |

Pass 1 validation is non-negotiable. No profile, no configuration, and no extension can downgrade a Pass 1 shape below Blocking. Pass 2 validation is profile-configurable within defined bounds: enforcement mode may be adjusted between Blocking, Warning, and Logging, but no shape may be downgraded below Logging.

The SDK returns validation results as typed objects. A `ValidationError` on a Pass 1 failure contains: the invariant violated (B1–B10), the shape URI that fired, the focus node (the primitive instance that failed), the severity (`sh:Violation`), and a remediation guidance string describing what the caller must provide to satisfy the shape.

---

## §29.3 SDK Query Layer

The Query layer provides typed access to the forty-four named query patterns defined in §28. Each pattern is a parameterized template with typed inputs and a typed return structure. The Query layer abstracts the dual-language query engine — callers specify what they want; the engine determines whether to use Cypher (graph traversal), SQL (relational aggregation), or both.

### §29.3.1 Named Pattern Access

Each query pattern is exposed as a named method with typed parameters and a typed return structure.

#### Table 29.3.1: Query Layer Pattern Access

| Domain Family | Patterns | MCP-Exposed | SDK-Only | Primary Return Structures |
|---|---|---|---|---|
| A: Authority & Delegation | QP-AUTH-01 through QP-AUTH-05 | 3 | 2 | PathResult, TreeResult |
| B: Decision Lineage | QP-LIN-01 through QP-LIN-04 | 4 | 0 | PathResult, TreeResult, SubgraphResult |
| C: Governance State | QP-GOV-01 through QP-GOV-05 | 5 | 0 | ActivationStatusRecord, GapRegisterEntry, ConformanceResult |
| D: Truth & Verification | QP-TRU-01 through QP-TRU-04 | 4 | 0 | Primitive records, verification records |
| E: Actor & Instance | QP-ACT-01 through QP-ACT-04 | 3 | 1 | Actor context, role assignments |
| F: Orchestration & Work | QP-ORC-01 through QP-ORC-05 | 3 | 2 | WorkSpecificationRecord, RoutingDecisionRecord |
| G: Signal & IQ | QP-SIG-01 through QP-SIG-04 | 3 | 1 | Signal records, IQ records, escalation chains |
| H: Profile & Portfolio | QP-PRO-01 through QP-PRO-04 | 2 | 2 | Profile status, instance trees, constraint cascades |
| I: Conservation & Invariant | QP-INV-01 through QP-INV-03 | 3 | 0 | ConformanceResult, violation history, override audit |
| J: KPI & Measurement | QP-KPI-01 through QP-KPI-03 | 3 | 0 | KPILifecycleRecord, InvariantSignalReading |
| K: Temporal & Cycle | QP-TMP-01 through QP-TMP-03 | 2 | 1 | Cycle records, timeline aggregation, account history |

### §29.3.2 Parameterized Templates

Every query pattern accepts typed parameters. Parameters fall into three categories:

- **Scope parameters** — `instance_id` (required on all patterns), `actor_id`, `authority_id` — identify the query target.
- **Filter parameters** — `status`, `truth_type`, `cognitive_type`, `invariant` — narrow the result set.
- **Traversal parameters** — `max_depth`, `relationship_type`, `time_window` — control graph traversal depth and temporal range.

The SDK validates parameters before query execution. Invalid parameters produce a `ValidationError` with the parameter name, the constraint violated, and a suggested action.

### §29.3.3 Return Structure Types

All query results are typed. The SDK Query layer defines eleven return structure families:

- **Primitive records** (9 types) — one per Tier 1 primitive, with common metadata fields.
- **Graph results** (3 types) — PathResult, TreeResult, SubgraphResult for graph traversal patterns.
- **Aggregate results** (3 types) — CountSummary, DistributionResult, TimelineResult for analytical patterns.
- **Governance state results** (3 types) — GapRegisterEntry, ActivationStatusRecord, ConformanceResult for governance domain patterns.
- **Orchestration results** (3 types) — RoutingDecisionRecord, WorkSpecificationRecord, GraduationAssessment for orchestration patterns.
- **KPI results** (2 types) — InvariantSignalReading, KPILifecycleRecord for KPI domain patterns.

List-returning patterns wrap results in `PaginatedList<T>` with `cursor`, `limit`, `total`, and `has_more` fields. Every query response includes `QueryMetadata` recording query execution identity, timing, engine selection, and profile context.

---

## §29.4 SDK Extension Points

The SDK defines extension points at documented boundaries. Each extension point specifies what is extensible, what constraints govern extension, and what composition rules apply. Extension points enable application-specific governance without modifying the core substrate.

### §29.4.1 Content Package Extensions

Content packages customize the substrate for specific governance domains. A content package bundles:

- **Intent templates** — pre-defined intent structures for domain-specific objectives (e.g., compliance intents, financial intents, operational intents).
- **Evidence types** — domain-specific evidence classifications with validation rules.
- **Authority templates** — pre-configured delegation structures for common organizational patterns.
- **Constraint sets** — domain-specific SHACL shapes that extend the core seventy-nine shapes.

Content packages are scoped to profiles. An EAS content package may activate governance patterns unavailable at BAS or PAS. Content packages compose additively: multiple packages may be active simultaneously, and their constraints compose through intersection (§A2.2).

#### Table 29.4.1: Content Package Constraints

| Constraint | Rule |
|---|---|
| **MUST** compose with core shapes | Package shapes MUST NOT override, relax, or contradict any of the 79 core SHACL shapes. Package shapes add domain-specific validation; they do not modify invariant enforcement. |
| **MUST** declare profile compatibility | Each package declares which profiles (EAS, BAS, PAS) it supports. The SDK rejects activation of a package incompatible with the instance's profile. |
| **MUST** use Namespace scoping | Package-defined concepts enter the Tenant or Domain namespace scope (§4.6.1). Core namespace is reserved for protocol-level definitions. |
| **MUST NOT** introduce new primitive types | Content packages extend the governance model through composition of existing primitives, not through new primitive definitions. |

### §29.4.2 Custom Governance Rules

Applications may define additional SHACL shapes for domain-specific constraints beyond those provided by content packages. Custom shapes follow the same registration and enforcement model as the core shapes.

Custom shapes MUST:
- Target existing primitive classes or content package extensions.
- Declare a severity level (`sh:Violation`, `sh:Warning`, or `sh:Info`).
- Declare an enforcement mode (Blocking, Warning, Logging, or Advisory).
- Register in the shapes graph with a distinct namespace URI.

Custom shapes MUST NOT:
- Override the enforcement mode of any core shape.
- Relax any constraint established by an ancestor instance in the portfolio hierarchy (tighten-only cascade, §23).
- Create circular dependencies with existing shapes.

### §29.4.3 Custom Query Patterns

Applications may register additional query patterns with the SDK Query layer. Custom patterns use the same parameterized template system as core patterns.

Custom patterns MUST:
- Declare typed input parameters and a typed return structure.
- Use the `PaginatedList<T>` wrapper for list results.
- Include `QueryMetadata` in every response.
- Register with a pattern identifier in the application namespace (not the `QP-` prefix reserved for core patterns).

Custom patterns MUST NOT:
- Bypass depth-gated visibility rules (§23). Custom patterns respect the same visibility constraints as core patterns.
- Return data from instances outside the caller's authorized scope.

### §29.4.4 Custom Composite Operations

Applications may define composite operations built from atomic SDK Core operations. A composite operation sequences multiple atomic operations within a transactional boundary.

Custom operations MUST:
- Compose exclusively from documented SDK Core operations.
- Produce the same invariant enforcement behavior as the component operations executed individually.
- Record the composite operation in the audit trail as a single governance act with lineage to its component operations.

Custom operations MUST NOT:
- Suppress or defer invariant validation that would fire on component operations.
- Create primitives without satisfying their behavioral invariant requirements.

### §29.4.5 Event Subscription Extensions

The SDK provides an event subscription mechanism for governance events. Applications subscribe to events at documented lifecycle boundaries.

#### Table 29.4.2: Subscribable Event Categories

| Category | Events | Use Cases |
|---|---|---|
| **Invariant violations** | Shape validation failures (79 shapes × 4 enforcement modes) | External alerting, compliance dashboards, audit integration |
| **State transitions** | Primitive lifecycle transitions, state machine transitions (SM-1 through SM-9) | Workflow automation, notification systems, integration triggers |
| **Signal lifecycle** | Signal creation, routing, escalation, resolution, dismissal | Escalation management, SLA monitoring, authority notification |
| **Governance activation** | Pipeline stage transitions, gap register updates, closure roadmap changes | Governance reporting, compliance tracking, audit trail enrichment |
| **KPI events** | KPI introduction, retirement, threshold breach, trend change | Dashboard updates, executive reporting, early warning systems |

Event subscriptions are passive observers. A subscription receives notification of governance events after they occur; it cannot block, modify, or intercept the event. This preserves the substrate's invariant enforcement integrity — extensions observe governance but do not gate it.

**DESIGN SPACE:** The event delivery mechanism — webhooks, message queues, server-sent events, polling — is an implementation choice. The SDK MUST provide a subscription registration API and guarantee at-least-once delivery semantics.

### §29.4.6 Extension Composition Rules

Extensions compose with the core substrate through defined rules that prevent extension conflicts and maintain governance integrity.

#### Table 29.4.3: Extension Composition

| Composition | Rule | Enforcement |
|---|---|---|
| **Shape composition** | Custom shapes ∩ core shapes = core shapes. Custom shapes add constraints; they never relax core constraints. When a custom shape and a core shape both target the same property, the more restrictive constraint applies. | SDK validates at shape registration time. |
| **Pattern composition** | Custom patterns may reference core patterns as sub-queries. Custom patterns inherit the visibility and authorization constraints of any core patterns they compose. | SDK validates at pattern registration time. |
| **Operation composition** | Custom composite operations sequence core operations. The composite inherits all invariant implications of its component operations. No invariant suppression is possible through composition. | SDK validates at operation execution time. |
| **Content package composition** | Multiple content packages compose through intersection. When two packages define constraints on the same primitive class, the intersection (most restrictive) applies. Package ordering does not affect the effective constraint set. | SDK validates at package activation time. |

### §29.4.7 Interchange Format Support

The SDK exposes query results through the three interchange formats specified in §28: MCP/JSON, JSON Export, and RDF/OWL Export. SDK consumers additionally access results as typed native objects — the programmatic access layer through which the interchange formats are produced.

#### Table 29.4.4: Interchange Format Architecture

| Format | Primary Consumer | Fidelity | Serialization |
|---|---|---|---|
| **MCP/JSON** | AI agents via MCP tool invocation | Simplified — graph structures flattened to arrays, OWL restrictions omitted, temporal semantics simplified to ISO 8601 strings | JSON with natural-language parameter descriptions |
| **JSON Export** | Web applications, REST clients, lightweight integrations | Simplified — same structural fidelity as MCP/JSON, with pagination via `cursor` + `limit` | JSON with pagination metadata |
| **RDF/OWL** | Formal compliance tools, archival systems, OWL reasoners | Full fidelity — all thirteen edge types preserved as distinct RDF properties, PROV-O provenance, W3C SHACL validation report format | JSON-LD (primary), Turtle (alternative) |

All three formats represent the same underlying query results; they differ in serialization and structural fidelity. The SDK typed access layer produces the interchange format appropriate to the consumer: MCP tools return MCP/JSON, programmatic SDK methods return typed objects convertible to JSON Export or RDF/OWL on request.

Key fidelity differences between JSON formats and RDF/OWL (§28): decision lineage (QP-LIN-01) loses causal branching in JSON. Impact radius (QP-LIN-03) collapses thirteen edge types to a generic `edge_type` field. Instance tree (QP-PRO-02) collapses five relationship types to generic "children." Conformance check (QP-GOV-04) simplifies the W3C SHACL validation report format. For governance applications requiring full fidelity, the RDF/OWL format is the canonical export.

---

## §29.5 Workbench Component Taxonomy

The Workbench organizes governance interaction into twelve component types across three classes. Each component has a defined data contract (what query patterns it consumes), interaction contract (what operations it invokes), and profile gate (when it activates).

### §29.5.1 Core Components

Core components are present in every substrate instance regardless of profile. They provide the fundamental governance interaction capabilities that all profiles require.

#### Table 29.5.1: Core Component Inventory

| Component | Data Contract | Interaction Contract | Section |
|---|---|---|---|
| **Primitive Browser** | QP-TRU-01 (truth type filter), QP-LIN-01/02/03 (lineage traversal), QP-AUTH-01 (authority chain) | Primitive CRUD operations, lineage navigation, truth type filtering | §29.6 |
| **Signal Capture** | QP-SIG-01 (signal query), QP-SIG-03 (escalation chain) | `signal.create`, `signal.create_reservation`, `signal.route`, `signal.escalate`, `signal.resolve`, `signal.dismiss` | §29.7 |
| **IQ Capture** | QP-SIG-02 (IQ query) | `iq.create`, `iq.create_from_signal`, `iq.create_from_claim`, `iq.assign`, `iq.resolve`, `iq.promote`, `iq.abandon` | §29.7 |
| **Decision Surface** | QP-LIN-04 (decision reconstruction), QP-GOV-05 (policy surface), VP materialization results | `vp.materialize`, disposition routing operations, Decision creation | §29.8 |

### §29.5.2 Governance Components

Governance components activate at EAS and BAS profiles. They provide governance activation tracking, conformance monitoring, and authority chain management.

#### Table 29.5.2: Governance Component Inventory

| Component | Data Contract | Interaction Contract | Profile Gate |
|---|---|---|---|
| **Governance Activation Dashboard** | QP-GOV-01 (governance registry), QP-GOV-02 (activation pipeline), QP-GOV-03 (gap register) | `gov_activation.*` (six-stage pipeline operations) | EAS, BAS |
| **Conformance Dashboard** | QP-INV-01 (invariant compliance), QP-INV-02 (violation history), QP-INV-03 (override audit), QP-GOV-04 (conformance check) | Conformance check execution, violation drill-down | EAS, BAS |
| **Authority Browser** | QP-AUTH-01 (authority chain), QP-AUTH-02 (delegation tree), QP-AUTH-03 (check authority) | `authority.delegate`, `delegation.compose`, delegation lifecycle operations | EAS, BAS |
| **Knowledge Integration Dashboard** | Alignment pipeline status, review queue depth, graduation progress, namespace coverage, bridge integrity, ontology family health | `ontology.align`, `ontology.bridge`, `ontology.graduate_concept`, alignment claim confirm/correct/reject | EAS, BAS |

The Knowledge Integration Dashboard provides two surfaces for TMI Operate mode (§18, §19) alignment management:

**Contextual inline.** When a practitioner works with a governed object (in the Primitive Browser, Decision Surface, or any component rendering primitive instances), pending alignment claims for that object's primitives appear inline. The user can confirm, correct, or dismiss an alignment proposal without leaving their governance context. This surfaces alignment review as a natural side-effect of governance work — the practitioner encounters alignment proposals where they are relevant, not in a separate queue.

**Dedicated steward view.** An ontology steward — a governance role with authority over the knowledge integration scope — accesses the full dashboard to manage:
- **Pipeline health** — TMI Alignment Pipeline (§18.3) throughput, confidence distribution across the three matchers, Stage 7 Review Queue depth and age
- **Graduation progress** — alignment claims by namespace scope (sandbox → tenant → domain → core), advancement and reversion trends (§18.4)
- **Namespace coverage** — which ontology families have active alignment coverage, gap detection across the ten recognized families (§19.2)
- **Bridge integrity** — cross-ontology bridge relationship health (§19.5), propagation accuracy, stale bridge detection
- **Conservation law tension** — alignment proposals flagged for potential conservation law conflict (§18.6), organized by affected primitive and law

The contextual surface is available to all users who encounter governed objects with pending alignment claims. The dedicated steward view is authority-gated — only actors with integration scope authority see the full dashboard.

### §29.5.3 Enterprise Components

Enterprise components activate at EAS profile only. They provide KPI management, AI orchestration oversight, and recursive portfolio governance.

#### Table 29.5.3: Enterprise Component Inventory

| Component | Data Contract | Interaction Contract | Profile Gate |
|---|---|---|---|
| **KPI Dashboard** | QP-KPI-01 (KPI lifecycle), QP-KPI-02 (invariant signals), QP-KPI-03 (control KPIs) | `kpi.introduce`, `kpi.retire`, `kpi.monitor_invariant_signals`, `kpi.review_implications` | EAS |
| **Orchestration Monitor** | QP-ORC-01 (work specifications), QP-ORC-02 (routing decisions), QP-ORC-05 (delegation scope) | Orchestration resolution audit, routing decision review, AICAR classification review | EAS |
| **Portfolio Browser** | QP-PRO-01 (profile status), QP-PRO-02 (instance tree), QP-ACT-01 (actor context), QP-ACT-02 (actor portfolio) | `portfolio.spawn_instance`, `portfolio.tighten_constraint`, `portfolio.relax_constraint`, `portfolio.depth_gated_read` | EAS |

### §29.5.4 Component Data Flow Architecture

Every Workbench component follows a uniform data flow: query patterns provide read access to governance state; operations provide write access through the SDK Core layer; validation results flow back to the component as rendering updates.

#### Table 29.5.4: Component Data Flow

| Direction | Mechanism | Example |
|---|---|---|
| **Read** (substrate → component) | SDK Query layer pattern invocation. Typed return structures populate component rendering. | Decision Surface invokes `QP-LIN-04` (reconstruct_decision) to populate the investigability chain. |
| **Write** (component → substrate) | SDK Core layer operation invocation. User interaction triggers an operation through the component. | Signal Capture component invokes `signal.create` when the user raises a signal. |
| **Validation feedback** (substrate → component) | SHACL validation results returned from write operations. Violations render inline. | Conformance Dashboard receives `ConformanceResult` from `QP-INV-01` and renders per-invariant status. |
| **Event notification** (substrate → component) | Event subscription mechanism delivers governance events to active components. | KPI Dashboard receives threshold breach events and updates invariant signal indicators. |

Components do not communicate directly with each other. Cross-component navigation is link-based: a violation rendered in the Conformance Dashboard links to the focus node in the Primitive Browser; a signal in the Signal Capture component links to the flagged object's detail view. The Workbench maintains navigational consistency through primitive identity — every governed object has a stable URI that any component can reference.

### §29.5.5 State Machine Rendering

The Workbench renders nine lifecycle state machines (SM-1 through SM-9) and three truth type state machines (§6). Each state machine renders through its owning component.

#### Table 29.5.5: State Machine to Component Mapping

| State Machine | Governs | States | Rendering Component |
|---|---|---|---|
| SM-1: Actor Lifecycle | Actor registration through archival | registered → active → suspended → deactivated → archived | Authority Browser |
| SM-2: Graduation | AI governance maturity (per-scope) | A → A+ → B → C (two-layer: Options + Stages 0–4; regression paths at every stage) | Orchestration Monitor |
| SM-3: Profile Graduation | Instance profile transitions | monitoring → triggered → evaluating → decided → executing → completed | Portfolio Browser |
| SM-4: Work Specification | Work specification lifecycle | draft → confirmed → active → completed / promoted / rejected | Orchestration Monitor |
| SM-5: Delegation | AI delegation lifecycle | proposed → active → modified → suspended → revoked / expired | Orchestration Monitor |
| SM-6: Agent Behavioral | AI actor operating cycle | listening → context_loading → navigating / monitoring / executing → proposing / alerting / completing → listening | Orchestration Monitor |
| SM-7: Signal Lifecycle | Signal from creation to resolution | created → routed → processing → escalated → monitoring → resolved / dismissed | Signal Capture |
| SM-8: IQ Lifecycle | IQ from capture to promotion | open → assigned → investigating → resolved / promoted / abandoned / superseded | IQ Capture |
| SM-9: VP Cache State | VP freshness management | current → stale → recomputing → current / invalidated | Decision Surface |

The §6 truth type system defines three additional orthogonal state machines: Evidence Verification (`VerificationState`), Derived Promotion (`TruthType`), and Claims Graduation (`ClaimsGraduationStage`). These are rendered by the Primitive Browser on every Evidence instance — the verification state indicator, truth type indicator, and claims graduation stage are visible on every record.

Each state machine rendering shows: the current state, the set of valid transitions from the current state, the guard conditions for each transition (what must be true for the transition to fire), and the authority requirement (who can trigger the transition). Transitions that require a Decision primitive (most non-automatic transitions) link to the Decision Surface for resolution.

---

## §29.6 Primitive Browser & Lineage Navigation

The Primitive Browser provides navigation and inspection of primitive instances within a substrate instance. It is the foundational Workbench component — every other component links back to the Primitive Browser when a user needs to inspect a specific governed object.

### §29.6.1 Primitive Instance Rendering

The Primitive Browser renders instances of all nineteen primitive types. Each instance renders with:

- **Identity** — primitive type, instance identifier, human-readable label.
- **Truth type indicator** — visual distinction between Authoritative, Declared, Derived, and Opaque records (§6). Derived records carry visible attribution identifying the generating agent or computation. Opaque records carry visible boundary indicator identifying the inspection boundary source.
- **Verification state** — current verification outcome (unverified, verification_pending, verified, verification_failed, cannot_verify).
- **Record lifecycle state** — whether this record is active or superseded.
- **Invariant status** — which behavioral invariants (B1–B10) apply to this instance and their current validation state.

#### Table 29.6.1: Primitive Browser View Modes

| Mode | Data Source | Rendering |
|---|---|---|
| **Instance list** | QP-TRU-01 with type and truth type filters | Paginated list of primitive instances with truth type, verification state, and lifecycle indicators. Sortable by type, creation date, truth type. Filterable by any combination of type, truth type, verification state, lifecycle state. |
| **Instance detail** | Direct primitive record read + QP-LIN-04 (decision reconstruction) | Full MVR fields, linked primitives, governance metadata, attached signals, attached IQs, authority chain, constraint scope. |
| **Lineage view** | QP-LIN-01 (decision lineage), QP-LIN-02 (decision descendants), QP-LIN-03 (impact radius) | Graph visualization of the primitive's position in the governance graph. Traversable in both directions: upstream (what led to this) and downstream (what depends on this). |

### §29.6.2 Truth Type Visualization

Truth type is a first-class visual element. Every primitive rendering distinguishes the three truth types through consistent visual language:

- **Authoritative** — the canonical record. Human-verified, immutable. Rendered with the highest visual confidence.
- **Declared** — human-asserted, attestation-backed. Rendered with moderate visual confidence and the declaring entity attribution.
- **Derived** — machine-generated, requires promotion. Rendered with explicit Derived marking, generating agent attribution, and a promotion pathway indicator when promotion is applicable.

The visual distinction is mandatory. A governance interface that renders Derived and Authoritative records identically violates the epistemic boundary (§6.2).

### §29.6.3 Lineage Traversal

The Primitive Browser provides graph-based lineage traversal using the thirteen cross-primitive relationship types defined in §4.4. Traversal supports:

- **Upstream navigation** — from a Decision, trace back through the Evidence it evaluated, the Authority that legitimized it, the Intent that framed it, and the Account that contextualized it.
- **Downstream navigation** — from a Decision, trace forward to the Work it authorized, the Commitments it created, and the Constraints it established.
- **Impact radius** — from any primitive, visualize all connected primitives within a configurable depth limit (QP-LIN-03).

Lineage traversal respects depth-gated visibility (§23). A user in a child instance sees full detail for their own instance, full detail for direct children, and aggregate patterns for deeper descendants. The Primitive Browser does not expose data outside the user's authorized visibility scope.

### §29.6.4 Cross-Primitive Relationship Rendering

The Primitive Browser renders the thirteen cross-primitive relationship types (§4.4) as navigable edges in the governance graph. Each relationship type is visually distinct and carries its cardinality and invariant enforcement status.

#### Table 29.6.2: Relationship Type Rendering

| Relationship | From → To | Invariant | Rendering Indicator |
|---|---|---|---|
| motivates | Intent → Commitment | — | Directional link, M:M |
| frames | Intent → Decision | — | Directional link, M:M |
| requires | Commitment → Capacity | B2 | Enforced link (violation = broken) |
| authorizes | Commitment → Work | B1 | Enforced link (violation = broken) |
| enables | Capacity → Work | — | Directional link, M:M |
| produces | Work → Evidence | — | Directional link, 1:M |
| evaluates | Decision → Evidence | — | Directional link, M:M |
| contextualizes | Account → Decision | B4 | Enforced link (violation = broken) |
| records | Account → Evidence | — | Directional link, 1:M |
| legitimizes | Authority → Decision | — | Directional link, M:M |
| delegates | Authority → Authority | B5 | Enforced chain (must reach root) |
| governs | Authority → Primitive | — | Directional link, 1:M |
| constrains | Constraint → Primitive | B6 | Enforced binding (must have targets) |

Relationships with invariant enforcement render with a distinct visual treatment: the link itself indicates whether the invariant is currently satisfied. A broken B1 link (Work without Commitment) renders as a violation indicator inline, navigable to the Conformance Dashboard violation detail.

---

## §29.7 Signal & IQ Capture Components

Signal Capture and IQ Capture are the two human capture mechanisms specified in §14–§16. The Workbench renders them as paired entry points: Signal Capture for object-anchored problems, IQ Capture for context-anchored emergence.

### §29.7.1 Signal Capture Component

The Signal Capture component implements the §15 contract: zero friction, always available, structurally typed, optionally deep, authority-routed, context preserved.

**Data consumed:** QP-SIG-01 (signal list with status and type filters), QP-SIG-03 (escalation chain for routing visualization).

**Rendering contract:**

| Element | Data Source | Interaction |
|---|---|---|
| **Capture entry point** | B7 guarantee — present on every governed object across all 19 primitive types | One-click signal creation. Signal type selection (6 types: reality_mismatch, logical_inconsistency, business_concern, compliance_concern, other, reservation). Optional free text, severity, related signals. |
| **Signal list** | QP-SIG-01 filtered by status, object type, severity | Paginated list with status indicator, signal type, flagged object reference, routing target, escalation history. |
| **Signal detail** | Direct signal record read | Immutable capture text, context snapshot, routing history, resolution or dismissal record, linked IQs. |
| **Routing visualization** | QP-SIG-03 (escalation chain) | Authority chain of the flagged object with current routing position highlighted. Algedonic bypass controls for escalation to any depth. |
| **Reservation capture** | Signal creation with `signal_type = reservation` | Reservation nature selection (7 types), surface trigger configuration (condition, timestamp, event). Non-blocking — the reserved object proceeds. |

The Signal Capture entry point MUST be present on every interface that renders a governed object. This is the B7 architectural guarantee: if an object is rendered, it is flaggable.

### §29.7.2 IQ Capture Component

The IQ Capture component implements the §16 contract: zero friction, always available, structurally typed, optionally deep, context preserved, investigator-routed.

**Data consumed:** QP-SIG-02 (IQ list with status and intent type filters).

**Rendering contract:**

| Element | Data Source | Interaction |
|---|---|---|
| **Global capture entry point** | Always accessible from any Workbench screen | One-field IQ creation: capture text. Optional intent type selection (8 types), priority, assignment, linked objects. |
| **Contextual capture entry point** | Available on every governed object view | Auto-fills origin context (origin URL, object type, object identity). Optional blocking relationship to in-progress Decision (§16.6). |
| **IQ list** | QP-SIG-02 filtered by status, intent type, assignment | Paginated list with status indicator, intent type, origin type, assigned investigator, promotion targets. |
| **IQ detail** | Direct IQ record read | Immutable capture text (with edit history), capture context snapshot, investigation notes, resolution record, promotion targets, linked signals. |
| **Branch inventory** | IQ with `intent_type = branch_inventory` | Thread inventory with disposition per thread (pursued, parked, dropped, merged, promoted). Capture method recorded (explicit, prompted, detected). |
| **Confidence probe** | IQ with `intent_type = confidence_probe`, agent-initiated | Lightweight probe: "settled / parked / provisional / needs revisit." Opt-in — dismissal recorded as disposition. |

### §29.7.3 Cross-Mechanism Integration

The Workbench renders cross-mechanism interactions (§14.5) as bidirectional links. When a signal spawns an IQ, the signal detail view shows the linked IQ with its current lifecycle state. When an IQ spawns a signal, the IQ detail view shows the linked signal. Both link types are navigable — clicking traverses to the linked record's detail view.

The Workbench surfaces reservation signals on reference: when a reserved object is rendered in any context (Primitive Browser, Decision Surface, governance dashboard), active reservations are visible as annotations with their surface trigger status and reservation nature.

---

## §29.8 Decision Surface Components

The Decision Surface component renders the §17 contract: the interactive interface through which humans evaluate governance conditions and resolve them through Decision primitives. Decision Surfaces compose from three Validated Projection types — Plan Baseline, Condition Assessment, and Prediction.

### §29.8.1 Decision Surface Rendering

The Decision Surface presents four compositional elements, each traceable to its source VP.

#### Table 29.8.1: Decision Surface Composition

| Element | Source | Rendering |
|---|---|---|
| **Condition panel** | Condition Assessment VP | Seven-dimensional analysis: condition type, causation, materiality, permanence, controllability, disposition pathway, trajectory. Each dimension rendered with its value and supporting evidence references. |
| **Baseline context** | Plan Baseline VP | Expected-state trajectory with baseline mode indicator (explicit, inferred, normative, comparative). Confidence score with mode-specific computation visible. Source primitive references for investigability. |
| **Disposition options** | Disposition routing (§17.6) + Prediction VPs | Nine disposition options rendered as selectable actions. Each option carries its Prediction VP: projected future states, confidence scores, assumption lists. Options without predictions are rendered with an "insufficient data" indicator. |
| **Investigability chain** | Evidence + source primitives | Every claim traces to Evidence. Every prediction traces to model + assumptions + source data. Every baseline traces to source primitives. All links are navigable. |

### §29.8.2 VP Overlay Rendering

Validated Projections are Derived truth (§6). The Decision Surface renders them with explicit Derived marking:

- **Confidence indicators** — numerical confidence (0.0–1.0) with mode-specific computation visible on hover or drill-down.
- **Staleness indicators** — VP cache state (Current, Stale, Recomputing, Invalidated) visible on each component. Stale components show time since staleness and whether recomputation is in progress.
- **Assumption inventory** — each VP's assumptions are accessible. The user can challenge any assumption and request an alternative projection with modified parameters.
- **Version provenance** — the specific VP version (projection_id + version) that was Current when the surface was composed.

If any component VP transitions to Stale or Invalidated between surface composition and human resolution, the Decision Surface re-renders with updated components before permitting Decision creation.

### §29.8.3 Information Quality Indicators

The Decision Surface integrates information quality indicators from the capture mechanisms:

- **IQ presence/absence** — for each decision option, the surface shows whether investigative queries have been conducted and what they found. Options with unresolved blocking IQs (§16.6) are marked as blocked.
- **Signal inventory** — active signals attached to the condition's subject primitive are visible. Unresolved signals are flagged.
- **Reservation visibility** — active reservations on the condition's subject are surfaced during decision evaluation.
- **Evidence completeness** — for each option, the surface indicates what evidence exists (with truth type) and what evidence gaps remain.

### §29.8.4 Authority Verification

Before a human resolves a Decision Surface, the Workbench verifies:

- The resolving actor holds Authority over the condition's scope (QP-AUTH-03).
- The authority chain is traceable to root (B5).
- The Decision will link to an Account (B4 — the SDK Core layer enforces this).

Authority verification results are rendered inline. If the actor lacks sufficient authority, the Decision Surface displays the required authority scope and the actor's current authority, with an escalation pathway.

### §29.8.5 Reservation Capture at Resolution

When a human resolves a Decision Surface by selecting a disposition, the interface offers reservation capture (§14.6.1). The resolver may record held objections as reservation signals attached to the Decision. Reservations do not block the resolution — they enrich the lineage record. The reservation nature taxonomy (framing, terminology, completeness, edge_case, confidence, timing, other) and surface trigger configuration are accessible at resolution time.

---

## §29.9 Governance Dashboard Components

Governance dashboard components render the governance activation pipeline (§12), conformance state (§5, §13), and authority structures. They activate at EAS and BAS profiles.

### §29.9.1 Governance Activation Dashboard

The Governance Activation Dashboard renders the six-stage activation pipeline (§12) for the substrate instance.

**Data consumed:** QP-GOV-01 (governance registry), QP-GOV-02 (activation pipeline), QP-GOV-03 (gap register).

#### Table 29.9.1: Activation Dashboard Elements

| Element | Data Source | Rendering |
|---|---|---|
| **Pipeline visualization** | QP-GOV-02 (activation pipeline) | Six stages rendered as a sequential pipeline: Discover → Analyze → Implement → Evidence → Verify → Sustain. Each activation source positioned at its current stage. Stage transition requirements visible. |
| **Source activation status** | QP-GOV-01 (governance registry) + QP-GOV-02 | Per-source card showing: activation type (full, partial, reference), trigger conditions matched, gap count, closure roadmap reference. Grouped by governance layer (Layer 1: universal frameworks, Layer 2: regulation, Layer 3: operational). |
| **Gap register** | QP-GOV-03 (gap register) | Gap inventory sorted by criticality (conservation-critical → invariant-enforcing → standard → advisory). Each gap shows: verification types present and missing, closure stage (awareness → enablement → management → governance), materiality level. |
| **Closure roadmap** | QP-GOV-03 + closure Work items | Ordered closure sequence with Work item status, Commitment links (B1), Capacity verification (B2), responsible authority. Progress indicators per gap. |

### §29.9.1.1 Gap Register Detail

The gap register is the operational core of the Governance Activation Dashboard. Each gap represents a governance requirement that is not yet fully evidenced. The rendering provides actionable context for closure.

#### Table 29.9.4: Gap Register Entry Rendering

| Field | Source | Rendering |
|---|---|---|
| **Governance pattern** | `pattern_ref` from GapRegisterEntry | Link to the governance source and pattern signature that established the requirement. |
| **Verification types present** | `verification_types_present` array | Enumeration of which verification types (EXIST through RATIFIED, §6.7) are satisfied. Rendered as a graduated coverage indicator across the seven-type spectrum. |
| **Verification types missing** | `verification_types_missing` array | Enumeration of which verification types are not yet satisfied. Each missing type is an actionable gap. |
| **Criticality** | `criticality_level` | Four levels rendered with visual weight: conservation-critical (highest), invariant-enforcing, standard, advisory (lowest). Criticality determines closure priority. |
| **Closure stage** | `closure_stage` | Four progression stages: awareness → enablement → management → governance. Current stage highlighted with remaining stages visible. |
| **Materiality** | `materiality_level` | Three levels: baseline, high_risk, certification_grade. Determines evidence rigor required for closure. |
| **Closure work items** | Linked Work primitives | List of Work items created for gap closure, each with status, assigned actor, and Commitment reference (B1). |

Gap register entries sort by criticality (conservation-critical first) and within criticality by closure stage (earlier stages first). This ordering is deterministic — the four-level criticality scheme (§12) produces a total order over gaps.

### §29.9.2 Conformance Dashboard

The Conformance Dashboard renders invariant validation results across all seventy-nine SHACL shapes and the ten behavioral invariants.

**Data consumed:** QP-INV-01 (invariant compliance), QP-INV-02 (violation history), QP-INV-03 (override audit), QP-GOV-04 (conformance check).

#### Table 29.9.2: Conformance Dashboard Elements

| Element | Data Source | Rendering |
|---|---|---|
| **Invariant status panel** | QP-INV-01 (invariant compliance) | Ten behavioral invariants (B1–B10) with pass/fail status. Each invariant shows: conservation law it protects, current validation result, violation count, sample violations. Grouped by invariant group (Resource Integrity, Epistemic Integrity, Governance Routing). |
| **Violation timeline** | QP-INV-02 (violation history) | Chronological violation feed with time-window filtering. Each violation shows: invariant, shape URI, focus node, severity, enforcement action taken, override reference (if any). |
| **Override audit** | QP-INV-03 (override audit) | Override inventory showing: Decision that authorized the override, invariant overridden, violation reference, authority chain verification, rationale, scope of exception. |
| **Shape validation results** | QP-GOV-04 (conformance check) | Per-shape validation results across all 79 shapes. Filterable by category (Governance, Actor, Portfolio, Orchestration, KPI), severity, enforcement mode. |
| **Conformance level indicator** | Computed from QP-INV-01 | Aggregate conformance assessment: fully conformant (all Pass 1 passing, all Pass 2 at configured mode), partially conformant (Pass 1 passing, Pass 2 violations at Warning/Logging), non-conformant (Pass 1 violations). |

### §29.9.3 Authority Browser

The Authority Browser provides interactive navigation of authority chains, delegation trees, and authority verification.

**Data consumed:** QP-AUTH-01 (authority chain), QP-AUTH-02 (delegation tree), QP-AUTH-03 (check authority).

#### Table 29.9.3: Authority Browser Elements

| Element | Data Source | Rendering |
|---|---|---|
| **Authority chain** | QP-AUTH-01 | Linear chain from any Authority to root. Each node shows: authority holder, scope, delegation rules, status (active, suspended, revoked, expired). Chain integrity indicator (B5). |
| **Delegation tree** | QP-AUTH-02 | Hierarchical tree from root authority downward. Expandable nodes showing all delegated authorities with scope attenuation visualization. Depth-bounded rendering with configurable max_depth. |
| **Authority check** | QP-AUTH-03 | Interactive authority verification: select an actor and a required scope, see whether authority exists and the path through the delegation chain. Used before governance acts to verify authorization. |
| **Multi-path authority** | QP-AUTH-02 + QP-AUTH-03 | When multiple authority paths exist for the same governance scope, the browser renders all paths with scope intersection analysis. |

---

## §29.10 KPI Dashboard Components

The KPI Dashboard renders the three KPI classes defined in the KPI evolution model (§28 Domain J): Invariant Signals (always-on health monitoring), Control KPIs (intent-driven phase-specific levers), and Intent Progress Indicators (strategic continuity tracking). The KPI Dashboard activates at EAS profile only.

**Data consumed:** QP-KPI-01 (KPI lifecycle), QP-KPI-02 (invariant signals), QP-KPI-03 (control KPIs).

### §29.10.1 Invariant Signal Zone

Invariant Signals are always-on, never time-boxed, and never targets. They protect the enterprise during rapid change. The Invariant Signal zone renders eight dimensions continuously.

#### Table 29.10.1: Invariant Signal Dimensions

| Dimension | Governance Question | Invariant Mapping | Threshold Visualization |
|---|---|---|---|
| Cycle time distribution | How responsive is the system? | B1, B2 (resource integrity) | Normal / Warning / Critical with trend line |
| Quality / rework rate | How sound is execution? | B3 (epistemic integrity) | Normal / Warning / Critical with trend line |
| Variance / predictability | How reliable are commitments? | B1, B2 (resource integrity) | Normal / Warning / Critical with trend line |
| Escalation frequency | How stressed is the system? | B8 (signal routing) | Normal / Warning / Critical with trend line |
| Judgment load | How burdened is human cognition? | B4 (decision context) | Normal / Warning / Critical with trend line |
| Evidence completeness | How auditable is the record? | B3 (epistemic integrity) | Normal / Warning / Critical with trend line |
| Semantic integrity | How stable is meaning? | B6 (constraint binding) | Normal / Warning / Critical with trend line |
| Change failure rate | What is the cost of change? | B6 (constraint binding) | Normal / Warning / Critical with trend line |

Each dimension renders:
- **Current value** — latest reading from QP-KPI-02.
- **Trend indicator** — increasing, stable, or decreasing, computed from historical values.
- **Threshold status** — normal, warning, or critical relative to configured thresholds.
- **Historical series** — time-series visualization with configurable time window.
- **Drill-down** — from any dimension, navigate to the underlying primitives contributing to the reading.

Invariant Signals run in the background of every EAS governance session. They do not require user action to render — they are ambient governance health indicators.

### §29.10.2 Control KPI Zone

Control KPIs are intent-driven, phase-specific, and explicitly temporary. The Control KPI zone renders active controls with their governance context.

**Rendering contract:**

| Element | Data Source | Rendering |
|---|---|---|
| **Active control list** | QP-KPI-03 (control KPIs filtered by active status) | Each control KPI shows: name, current value, target value, phase, linked Intent reference, days remaining in expected lifespan, governance domain. |
| **Introduction history** | QP-KPI-01 (KPI lifecycle with trigger type) | For each control KPI: introduction Decision reference, trigger type (intent-driven, constraint-driven, emergent), trigger reference, authority approval. |
| **Retirement history** | QP-KPI-01 (KPI lifecycle with retirement events) | Retired CKPIs with: retirement reason (intent advanced, phase complete, constraint resolved, replaced), final state, learning summary (mandatory). |
| **Lifecycle timeline** | QP-KPI-01 with time-window filter | Chronological view of KPI introduction and retirement events. Phase boundaries visible. |

The governing rule — no KPI may silently disappear — is enforced structurally. The Workbench does not provide a "delete KPI" action. KPI retirement requires a Decision with an Account (B4), an authority approval, and a mandatory learning summary.

### §29.10.3 Intent Progress Indicator Zone

Intent Progress Indicators are slow-moving, aggregated, and reinterpreted more often than redefined. The IPI zone renders strategic continuity.

**Rendering contract:**

| Element | Data Source | Rendering |
|---|---|---|
| **Active IPI list** | QP-KPI-01 (KPI lifecycle filtered by `kpi_class = intent_progress_indicator`) | Each IPI shows: name, current value, linked Intent reference, confidence trend, measurement method. |
| **Confidence trend** | QP-KPI-02 (signal readings for IPI dimensions) | Confidence trajectory over time. Direction of change visible. Points where confidence shifted with causal context (new evidence, changed assumptions). |
| **Intent linkage** | QP-KPI-01 `trigger_reference` → Intent primitive | Navigation from IPI to the Intent it tracks. Intent lifecycle state (draft, active, achieved, abandoned, superseded) visible alongside the IPI reading. |

---

## §29.11 Orchestration Monitor Components

The Orchestration Monitor renders AI work orchestration state: work specification status, routing decision audit, AICAR cognitive type classification, and delegation scope. The Orchestration Monitor activates at EAS profile only.

**Data consumed:** QP-ORC-01 (work specifications), QP-ORC-02 (routing decisions), QP-ORC-05 (delegation scope), QP-ACT-01 (actor context).

### §29.11.1 Work Specification Status

The Orchestration Monitor renders active and historical work specifications.

#### Table 29.11.1: Work Specification View

| Element | Data Source | Rendering |
|---|---|---|
| **Specification list** | QP-ORC-01 filtered by status, cognitive type | Paginated list showing: work spec identifier, cognitive decomposition summary (A/B/C/D component types), composition strategy (sequential, parallel, iterative, conditional, delegated, federated), status (draft, confirmed, active, completed, promoted, rejected). |
| **Specification detail** | QP-ORC-01 single record | Full cognitive decomposition with per-component detail: component ID, cognitive type, assigned actor, delegation reference, autonomy level. Effective constraints per component. Five-dimensional context. |
| **Lifecycle tracking** | QP-ORC-01 with SM-4 state machine positions | State machine visualization: Draft → Confirmed → Active → Completed/Promoted/Rejected. Transition history with Decision references for each transition. |

### §29.11.2 Routing Decision Audit

Every routing decision in the seven-step resolution engine (§A2) is a governance act with full lineage. The Orchestration Monitor renders routing decisions for audit and review.

#### Table 29.11.2: Routing Decision View

| Element | Data Source | Rendering |
|---|---|---|
| **Decision list** | QP-ORC-02 | Paginated list of routing decisions with: five-dimensional position, selected actor(s), computed HITL level, confidence assessment, authority basis. |
| **Decision detail** | QP-ORC-02 single record | Full routing decision: alternatives considered (all eligible actors evaluated), selected assignments (actor-to-component matches), selection rationale (Account reference), five-dimensional resolution (AICAR type × decision type × consequence level × operating posture × graduation stage), confidence score. |
| **Five-dimensional visualization** | QP-ORC-02 `five_dimensional_position` | Five-axis rendering of the routing space position. Position determines HITL requirement. Threshold boundaries visible. |

### §29.11.3 AICAR Classification Visibility

The Workbench renders the AICAR cognitive type classification for transparency and audit.

#### Table 29.11.3: AICAR Classification View

| Element | Data Source | Rendering |
|---|---|---|
| **Classification summary** | QP-ORC-01 `cognitive_decomposition` | Distribution of cognitive work types across active work specifications: A (Citation), B (Reasoning), C (Context), D (Communication). Composite classification visible (e.g., A+B+C). |
| **Per-type risk profile** | AICAR type properties (§A1.2) | Each cognitive type rendered with: hallucination risk level, verification complexity, required review method. Red-zone indicator for high-stakes + high-novelty combinations. |
| **Type-specific verification** | Shape validation results for AICAR shapes (#39–#42) | Type A citation fidelity verification status. Type B reasoning integrity review status. Red-zone blocking status for rubber-stamp detection. |

### §29.11.4 Delegation Scope Browser

The Delegation Scope Browser renders the delegation landscape for AI actors within the instance.

#### Table 29.11.4: Delegation Scope View

| Element | Data Source | Rendering |
|---|---|---|
| **Active delegations** | QP-ORC-05 per actor | Each delegation shows: cognitive types permitted, namespace boundaries, autonomy level (0–3), confidence floor, active constraints. Delegation lifecycle state (SM-5: proposed, active, modified, suspended, revoked, expired). |
| **Scope attenuation** | QP-AUTH-02 (delegation tree) + QP-ORC-05 | Visualization of scope narrowing from human authority through delegation chain to AI actor. Each delegation level shows scope reduction. |
| **Autonomy level mapping** | QP-ORC-05 `autonomy_level` + graduation stage | Autonomy level rendered against graduation stage ceiling: Stage A → level 0, Stage A+ → level 1, Stage B → level 2, Stage C → level 3. |

### §29.11.5 Portfolio Browser

The Portfolio Browser renders the recursive instance hierarchy (§23): parent-child relationships, constraint cascade visualization, depth-gated views, and cross-instance aggregation.

**Data consumed:** QP-PRO-01 (profile status), QP-PRO-02 (instance tree), QP-ACT-01 (actor context), QP-ACT-02 (actor portfolio).

#### Table 29.11.5: Portfolio Browser Elements

| Element | Data Source | Rendering |
|---|---|---|
| **Instance tree** | QP-PRO-02 (instance tree with relationship type filter) | Recursive tree visualization with five relationship types (§23): parent_child, spawns, graduates_to, feeds, licenses_from. Each node shows: instance identifier, profile type (EAS/BAS/PAS), graduation stage, active tier count. Expandable to configurable depth. |
| **Constraint cascade** | Constraint records at each hierarchy level | Per-level constraint display showing intersection composition: root constraints ∩ level-1 constraints ∩ ... ∩ parent constraints ∩ own constraints. Tighten-only monotonicity visible: child constraints MUST be ≥ parent constraints. Where a constraining authority has approved relaxation through the five-step evidence-driven process (§23), the authorizing Decision reference and the modified constraint boundary are rendered at the affected level. |
| **Depth-gated views** | QP-PRO-02 + depth-gating rules (§23) | Own data: full detail. Direct children: full detail. Deeper descendants: aggregate patterns only (counts, distributions, status summaries). Siblings and cousins: no visibility. Parent operational data: no visibility. Visibility boundaries rendered explicitly — the user sees where their view terminates and why. |
| **Profile graduation status** | QP-PRO-01 (profile status with trigger evaluation) | Per-instance graduation state: current stage, trigger status per graduation trigger, sensitivity configuration. Graduation recommendation (maintain, advance, regress) from GraduationAssessment return structure. |
| **Cross-instance aggregation** | QP-PRO-02 aggregated across visible children | Aggregate governance health across direct children: invariant signal summaries, conformance status distribution, active signal counts, unresolved IQ counts. Aggregates carry truth type Derived — they are computed summaries, not authoritative state. |
| **Spawning interface** | `portfolio.spawn_instance` operation | Instance creation with profile assignment by complexity, constraint inheritance (tighten-only), authority delegation (scope ⊆ parent), graduation stage ceiling (≤ parent stage). |

---

## §29.12 SHACL Violation Rendering Contract

Every SHACL shape validation result maps to a Workbench rendering. The seventy-nine shapes across five categories (Governance, Actor, Portfolio, Orchestration, KPI) plus the seventeen Phase 1 foundation shapes produce validation outcomes that the Workbench must render consistently.

### §29.12.1 Three-Level Rendering Model

The Workbench renders validation outcomes at three severity levels.

#### Table 29.12.1: Violation Rendering Levels

| Validation Outcome | Severity | Rendering | User Action |
|---|---|---|---|
| **sh:Violation (Blocking)** | `sh:Violation` with enforcement mode = Blocking | Error panel: shape URI, focus node (primitive type and ID), violation detail (which property failed, what was expected), remediation guidance (what the user must provide). Operation blocked — the action that triggered the violation did not execute. | User corrects the violation and retries the operation. |
| **sh:Warning** | `sh:Violation` with enforcement mode = Warning, or `sh:Warning` | Warning banner: shape URI, focus node, governance implication (what risk the warning indicates). Operation permitted — the action executed despite the warning. Warning recorded in governance trail. | User acknowledges the warning. May choose to address the underlying concern or accept the governance risk. |
| **sh:Info (Logging)** | `sh:Info` or enforcement mode = Logging | No active rendering in the primary interface. Violation recorded in audit trail. Visible in Conformance Dashboard violation timeline and through QP-INV-02 (violation history) queries. | No immediate user action required. Violations accumulate for periodic governance review. |

### §29.12.2 Shape-to-Component Mapping

Each SHACL shape maps to the Workbench component that renders its violations.

#### Table 29.12.2: Shape Category Rendering

| Shape Category | Shape Count | Blocking | Warning | Logging | Rendering Component |
|---|---|---|---|---|---|
| Phase 1 Foundation (B1–B10 invariants) | 10 | 8 | 2 | 0 | Conformance Dashboard (invariant status panel) |
| Phase 1 Verification Types (§6) | 7 | 4 | 3 | 0 | Conformance Dashboard (shape validation results) |
| Governance (§11–§13) | 17 | 10 | 7 | 0 | Governance Activation Dashboard + Conformance Dashboard |
| Actor (§22) | 15 | 11 | 3 | 1 | Authority Browser + Conformance Dashboard |
| Portfolio (§23) | 6 | 4 | 0 | 2 | Portfolio Browser + Conformance Dashboard |
| Orchestration (§A1/§A2) | 16 | 15 | 1 | 0 | Orchestration Monitor + Conformance Dashboard |
| KPI & Measurement (§28 Domain J) | 8 | 6 | 2 | 0 | KPI Dashboard + Conformance Dashboard |
| **Total** | **79** | **58** | **18** | **3** | |

The Conformance Dashboard is the universal fallback — every shape violation appears there regardless of which component triggered it. Domain-specific components (Governance Activation Dashboard, Authority Browser, Portfolio Browser, Orchestration Monitor, KPI Dashboard) additionally render violations relevant to their governance concern.

### §29.12.3 Remediation Guidance Pattern

Every Blocking violation renders with remediation guidance. The guidance pattern is structured:

1. **What failed** — the shape URI and a human-readable description of the constraint.
2. **Where it failed** — the focus node: primitive type, instance identifier, and the specific property that did not satisfy the shape.
3. **Why it matters** — the invariant or governance rule the shape enforces, and the governance failure that the shape prevents.
4. **How to fix it** — specific, actionable guidance. For B1 violations: "Link this Work item to a Commitment." For B5 violations: "This Authority has no delegation chain to root. Either mark it as a root authority or establish a delegation from an existing authority."

Remediation guidance is generated from the shape metadata — the shape URI, target class, and property constraints — not from hard-coded strings. This enables custom shapes (§29.4.2) to produce remediation guidance using the same pattern.

### §29.12.4 Violation Context Preservation

When a violation occurs, the Workbench preserves the operational context:

- **The operation that triggered the violation** — which operation was attempted, by whom, when.
- **The primitive state at violation time** — the focus node's properties at the moment of validation failure.
- **The shape that fired** — the full shape URI, severity, and enforcement mode.
- **The validation pass** — Pass 1 (Core), Pass 2 (SPARQL), or Schema Pass.

This context is recorded as Evidence with truth type Authoritative (the validation result is a system fact) and is queryable through QP-INV-02 (violation history).

---

## §29.13 Profile-Gated Component Availability

Workbench components activate based on the substrate instance's profile type and graduation stage. No component renders data that the profile cannot produce. No component offers interactions that the profile does not support.

### §29.13.1 Profile-Component Matrix

#### Table 29.13.1: Component Availability by Profile

| Component | PAS | BAS | EAS | Graduation Gate |
|---|---|---|---|---|
| Primitive Browser | Active | Active | Active | None — available at all stages |
| Signal Capture | Active | Active | Active | None — B7 requires flaggability at all profiles |
| IQ Capture | Active | Active | Active | None — B9 requires emergence capture at all profiles |
| Decision Surface | Active | Active | Active | None — decision rendering available at all profiles |
| Governance Activation Dashboard | — | Active | Active | Stage A (activation pipeline requires governance registry) |
| Conformance Dashboard | Limited | Active | Active | PAS: B1–B4 + B5-immediate + B6-binding + B7 + B8-core only. BAS/EAS: full 79 shapes. |
| Authority Browser | Limited | Active | Active | PAS: authority chain only. BAS/EAS: full delegation tree + multi-path. |
| Knowledge Integration Dashboard | — | Active | Active | Stage A (alignment pipeline requires governance activation; contextual inline available at all profiles with pending alignment claims) |
| KPI Dashboard | — | — | Active | Stage B (KPI management requires governed operation tier) |
| Orchestration Monitor | — | — | Active | Stage C (orchestration requires AI-native extensions tier) |
| Portfolio Browser | — | — | Active | Stage A (portfolio management available at EAS from inception) |

### §29.13.2 Graduation-Gated Rendering

Within a profile, graduation stage determines which component features are available.

#### Table 29.13.2: Graduation Stage Feature Activation

| Stage | Tier Requirement | New Component Features |
|---|---|---|
| **A (Capture)** | Tiers 1–2 | Full Primitive Browser. Signal and IQ capture. Basic Decision Surface (no VP-based predictions). Basic Conformance Dashboard (Pass 1 shapes). |
| **A+ (Pattern Recognition)** | Tiers 1–2 | Agent-initiated confidence probes (§14.6.3). Branch inventory prompting. Pattern detection feeds (QP-SIG-04). |
| **B (Advise)** | Tiers 1–3 | Full Decision Surface with VP overlays (Plan Baseline, Condition Assessment, Prediction). KPI Dashboard (EAS). Full Conformance Dashboard (Pass 1 + Pass 2 shapes). |
| **C (Orchestrate)** | Tiers 1–4 | Orchestration Monitor. Full AICAR classification visibility. Autonomous routing audit. Delegation scope browser with autonomy level visualization. |

### §29.13.3 PAS Simplification

PAS (Project Accountability Substrate) instances operate with a simplified Workbench. The simplification follows from PAS's bounded scope, limited governance hierarchy, and reduced RACIVG model (Owner + Collaborator instead of full six-role model).

PAS-specific rendering rules:
- **Conformance Dashboard** renders Pass 1 shapes only. Pass 2 shapes (B5-T, B6-U, B8-routing, B9) render at Logging level — visible in audit views, not in active dashboards.
- **Authority Browser** renders authority chains (QP-AUTH-01) but does not render full delegation trees (QP-AUTH-02) — PAS authority structures are flat by design.
- **Signal routing** renders default routes without algedonic bypass controls — PAS delegation depth is typically insufficient to warrant multi-level escalation.
- Governance Activation Dashboard, KPI Dashboard, Orchestration Monitor, and Portfolio Browser do not render at PAS.

---

## §29.14 SDK Constraints

SDK constraints are specified as MUST, MUST NOT, and DESIGN SPACE statements. MUST and MUST NOT are implementation requirements. DESIGN SPACE identifies decisions left to substrate implementors.

### §29.14.1 Core Layer Constraints

**MUST:**
1. Enforce Pass 1 SHACL validation (B1–B4, B5-immediate, B6-binding, B8-core) as Blocking on every write operation across all profiles.
2. Enforce the Schema Pass (B7) at deployment time. Deployment MUST fail if any of the 19 primitive classes lacks a signal attachment surface.
3. Return typed `ValidationError` on Pass 1 failure containing: invariant violated, shape URI, focus node, severity, and remediation guidance.
4. Enforce progressive immutability: fields immutable from their assignment point MUST reject modification attempts.
5. Enforce monotonic truth type promotion (Derived → Declared → Authoritative). Demotion MUST be rejected.
6. Execute composite operations atomically. If any component operation fails validation, no records from the composite are persisted.
7. Record every operation in the audit trail with actor identity, timestamp, operation type, and affected primitive references.

**MUST NOT:**
8. Allow any profile, configuration, or extension to downgrade a Pass 1 shape below Blocking.
9. Allow any invariant enforcement to be downgraded below Logging for any profile.
10. Persist a primitive record that fails structural validation against its MVR field requirements (§7).
11. Allow AI actors to create records with truth type Authoritative or Declared. All AI-generated records enter as Derived (B3).

**DESIGN SPACE:**
12. Transport protocol for SDK method invocation — REST, gRPC, native library binding, or other protocols.
13. Concurrency model for simultaneous write operations — optimistic locking, pessimistic locking, or serializable transactions.
14. Pass 2 evaluation scheduling — per-operation, batched, periodic, or event-triggered. The evaluation schedule is profile-configurable within documented bounds.

### §29.14.2 Query Layer Constraints

**MUST:**
15. Expose all forty-four query patterns as typed methods with validated parameters and typed return structures.
16. Include `QueryMetadata` (query_id, pattern_id, executed_at, execution_ms, language_used, profile_type) in every query response.
17. Enforce depth-gated visibility (§23) on all query patterns. No query may return data from instances outside the caller's authorized visibility scope.
18. Use `PaginatedList<T>` wrapper for all list-returning patterns with `cursor`, `limit`, `total`, and `has_more` fields.
19. Return `ValidationError` on invalid query parameters before query execution.

**MUST NOT:**
20. Expose SDK-only patterns (QP-AUTH-04, QP-AUTH-05, QP-ACT-04, QP-ORC-03, QP-ORC-04, QP-SIG-04, QP-PRO-03, QP-PRO-04, QP-TMP-02) through the MCP Server. These patterns are internal to SDK consumers.
21. Return stale Validated Projection results without staleness indicators. If a VP is in Stale state, the response MUST include staleness metadata.

**DESIGN SPACE:**
22. Query engine selection strategy — whether the SDK automatically selects Cypher, SQL, or hybrid execution based on the query pattern, or whether the caller can influence engine selection.
23. Result caching strategy for frequently-accessed query patterns.

### §29.14.3 Extension Layer Constraints

**MUST:**
24. Provide extension registration APIs for content packages, custom shapes, custom patterns, custom operations, and event subscriptions.
25. Validate that custom SHACL shapes do not override, relax, or contradict any of the 79 core shapes.
26. Validate that custom query patterns respect depth-gated visibility (§23).
27. Guarantee at-least-once delivery for event subscriptions.
28. Reject content package activation when the package's declared profile compatibility does not match the instance's profile type.

**MUST NOT:**
29. Allow extension-defined shapes to reduce the enforcement mode of core shapes.
30. Allow custom operations to suppress or defer invariant validation that would fire on component operations.
31. Allow event subscriptions to block, modify, or intercept governance events.
32. Allow content packages to introduce new primitive types.

**DESIGN SPACE:**
33. Event delivery mechanism — webhooks, message queues, server-sent events, or polling.
34. Content package distribution mechanism — bundled with SDK, external registry, or file-based loading.
35. Extension namespace management — how application namespaces are allocated, validated, and prevented from colliding.

### §29.14.4 Workbench Component Constraints

**MUST:**
36. Render the Signal Capture entry point on every interface that displays a governed object (B7 enforcement).
37. Render the IQ Capture global entry point on every Workbench screen (§16.7).
38. Render truth type indicators (Authoritative, Declared, Derived, Opaque) on every primitive instance rendering. Derived records MUST carry visible agent attribution. Opaque records MUST carry visible boundary indicator.
39. Render VP-derived content with explicit Derived marking, confidence indicators, and staleness state (§17).
40. Re-render the Decision Surface when any component VP transitions to Stale or Invalidated before human resolution (§17.8).
41. Record the specific VP versions (projection_id + version) in every Decision primitive created from a Decision Surface resolution.
42. Render remediation guidance on every Blocking SHACL violation with: what failed, where it failed, why it matters, and how to fix it.
43. Enforce profile-gated component availability — no component renders data that the profile cannot produce.

**MUST NOT:**
44. Render Derived and Authoritative records identically. The visual distinction is mandatory.
45. Allow Decision creation from a Decision Surface whose component VPs include any in Invalidated state.
46. Auto-select disposition on a Decision Surface without human selection. Disposition is a governance act.
47. Provide a "delete KPI" action. KPI retirement requires a Decision with Account, authority approval, and mandatory learning summary.
48. Use IQ capture patterns by user for performance evaluation, supervisor visibility without consent, or individual-identifying aggregation (§16.8 pattern privacy).

**DESIGN SPACE:**
49. Visual design — colors, typography, layout, spacing, responsive breakpoints, accessibility implementation.
50. Interaction patterns — keyboard shortcuts, drag-and-drop, modal flows, command palette implementation.
51. Component rendering technology — web components, framework-specific components, native platform controls.
52. Dashboard layout — fixed, configurable, or role-based arrangement of governance components.
53. Notification and alerting UX — how warnings, invariant signal threshold breaches, and escalation events are presented to users.

---

## Scope

Scope limited to logical model specification. Transport protocol, serialization format, and implementation languages are DESIGN SPACE.

## Implementation Requirements

SDK implementations MUST respect all invariant shapes, enforce truth type boundaries, and validate all constraints at specified passes. Specification is immutable during lock period. SHACL two-pass validation is mandatory. Truth type boundaries are preserved through all operations. Session isolation is required for all writes.

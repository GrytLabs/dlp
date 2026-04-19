# §13 Policy Projection Surfaces

§11 specifies the governance registry structure — what governance content exists and how it is organized into three layers of pattern signatures. §12 specifies the activation pipeline — which patterns apply to a specific organization, what gaps exist, and how closure proceeds. This section specifies what happens with the resulting governance state: it is rendered as human-readable policy documents, and those documents are verified against enforcement reality.

The core architectural inversion: policy is not the input. Policy is an output. Traditional organizations write policy documents first, then try to enforce them. The substrate inverts this. The substrate enforces governance through primitives, invariants, and constraints. Policy documents are projections — read-only views that render substrate state as human-readable governance artifacts. No projection may add governance. A projection may only reveal what already exists.

## §13.1 The Projection Principle


**Policy is not the input. Policy is an output.**

The substrate is primary. Documents are views over enforced structure. No projection may add governance — it may only reveal what already exists.

This inversion is not a design preference. It is the architectural consequence of the conservation laws (§9). If policy documents were the input — if governance originated in prose — then the binding force of commitments, the scope of authority delegations, and the universality of constraints would depend on the fidelity of human interpretation of written language. The conservation laws would hold only to the extent that readers agreed on meaning. The substrate makes conservation laws structural: obligation invariance is enforced by B1 and B2 (§5), not by a paragraph in a policy manual.

Policy documents remain essential. Organizations need human-readable governance artifacts for communication, training, regulatory submission, and audit. The projection principle ensures these artifacts are derived from enforcement reality rather than aspirational description.

Three protocol layers separate the projection concept from its implementation.

### Table 13.1.1: Protocol Layer Placement

| Protocol Layer | Projection Content | Example |
|---|---|---|
| **DLP-World-Model** | The projection concept — substrate state renders as governance output. Seven domains as protocol requirement. | "The protocol must expose Authority & Decision Rights as a queryable governance surface" |
| **DLP Substrate** | Projection engine, rendering mechanics, document catalog, template marketplace | Authority Matrix renderer, SOP generator, conformance coupling logic |
| **Licensee Configuration** | Organizational vocabulary, specific thresholds, augmentation text | "In our firm, 'Partner' means holders of Authority with scope 'firm-wide'" |

The projection concept belongs to the open protocol — any DLP implementation must project state. The projection engine belongs to the licensed substrate — rendering mechanics, document catalog, and template infrastructure are substrate implementation. The projection content — specific organizational language, thresholds, and augmentation — belongs to the licensee.

## §13.2 Seven Core Policy Domains


Seven substrate-native governance domains structure the projection surface. Each domain satisfies three tests: universality (every governed organization needs it), substrate-native (maps directly to primitives), and existential failure (its absence causes governance breakdown).

These domains replace traditional departmental governance categories (Financial, Operations, Compliance, HR, Information, Risk, Governance). The realignment is structural: projection surfaces mirror the protocol's primitive structure, not the organizational chart. §10 references these seven domains as edge properties in the perimeter observation model.

### Domain 1: Authority & Decision Rights

**Purpose:** Define who is allowed to decide, assert, approve, or override — and under what conditions.

**Projects from:** Authority assignments, Delegation rules, Override semantics, Invariants

**Governs:** Decision initiation, decision finalization, approval thresholds, escalation paths

**Does NOT govern:** Quality of decisions, expertise hierarchies, human judgment replacement

**Canonical documents projected:** Authority Matrix, Delegation Schedule, Decision Rights Charter, Approval Authority Table

### Domain 2: Commitment & Obligation

**Purpose:** Define what counts as a binding commitment, how it is created, and how it is discharged.

**Projects from:** Commitments, Cycles, Authority, Evidence requirements

**Governs:** Financial commitments, service obligations, internal promises, delivery expectations

**Does NOT govern:** Pricing, performance quality, outcome guarantees

**Canonical documents projected:** Commitment Register, Obligation Schedule, Service Level Framework, Promise-to-Delivery Matrix

### Domain 3: Evidence, Record & Truth

**Purpose:** Define what counts as evidence, how truth is established, and what can be relied upon later.

**Projects from:** Evidence semantics, Truth types (Authoritative, Declared, Derived), Event immutability, Confidence thresholds

**Governs:** Record retention, proof of completion, auditability, dispute resolution inputs

**Does NOT govern:** Dispute decisions, intent interpretation, correctness guarantees

**Canonical documents projected:** Evidence Standards Manual, Record Retention Schedule, Truth Classification Guide, Audit Evidence Requirements

### Domain 4: Work Execution & Lifecycle

**Purpose:** Define how work enters, progresses, completes, and closes under governance.

**Projects from:** Work definitions, Cycle models, Authority gates, Completion evidence

**Governs:** Work initiation, state transitions, completion conditions, rework and rollback triggers

**Does NOT govern:** How work is performed, workflow optimization, project management judgment

**Canonical documents projected:** Work Lifecycle Procedures, State Transition Matrix, Completion Criteria Checklist, Cycle Close-Out Protocol

### Domain 5: Access, Identity & Boundary

**Purpose:** Define who can see, touch, or act on what, and where boundaries exist.

**Projects from:** Entity definitions, Authority, Constraints, Identity signals

**Governs:** Access rights, role boundaries, data visibility, separation of concerns

**Does NOT govern:** User authentication (tool concern), credential management, organizational chart definitions

**Canonical documents projected:** Access Control Matrix, Role Boundary Definitions, Data Classification Guide, Separation of Duties Chart

### Domain 6: Exception, Escalation & Override

**Purpose:** Define what happens when the system cannot proceed normally.

**Projects from:** Invariants, Constraint violations, Authority overrides, Event classifications

**Governs:** Exception handling, escalation triggers, override logging, emergency actions

**Does NOT govern:** Exception prevention, judgment automation, conflict resolution

**Canonical documents projected:** Exception Handling Procedures, Escalation Matrix, Override Authorization Protocol, Emergency Action Guide

### Domain 7: Learning, Change & Evolution

**Purpose:** Define how the system is allowed to change based on outcomes.

**Projects from:** Learning primitives, Evidence outcomes, Orientation shifts, Authority over change

**Governs:** What learning is accepted, who can approve changes, when models update, how drift is handled

**Does NOT govern:** Improvement guarantees, retrospective replacement, strategy decisions

**Canonical documents projected:** Change Control Procedures, Learning Acceptance Criteria, Model Update Protocol, Drift Detection & Response Guide

### Domain Sufficiency Argument

Seven domains, not six or eight. The three-part test validates each domain independently.

### Table 13.2.1: Domain Sufficiency Argument

| Domain | Universality | Substrate-Native | Existential Failure Mode |
|---|---|---|---|
| 1. Authority & Decision Rights | Every organization has authority structures | Maps to Authority, Decision, Delegation primitives | Without: no one knows who can decide |
| 2. Commitment & Obligation | Every organization makes promises | Maps to Commitment, Capacity, Evidence primitives | Without: promises are unenforceable |
| 3. Evidence, Record & Truth | Every organization needs proof | Maps to Evidence, Truth Type, Account primitives | Without: nothing can be verified |
| 4. Work Execution & Lifecycle | Every organization does work | Maps to Work, Cycle, Decision Gate primitives | Without: work has no governance |
| 5. Access, Identity & Boundary | Every organization has boundaries | Maps to Entity, Authority, Constraint primitives | Without: no separation of concerns |
| 6. Exception, Escalation & Override | Every organization encounters exceptions | Maps to Constraint violation, Authority override, Event primitives | Without: exceptions are ungoverned |
| 7. Learning, Change & Evolution | Every organization must adapt | Maps to Learning, Evidence outcome, Orientation primitives | Without: the system cannot improve |

The three-part test is empirically grounded. A stronger foundation is available: each domain preserves a specific organizational symmetry from §9. Seven domains are sufficient because they correspond to seven preserved organizational symmetries. Each domain's projection surface is necessary because it makes a specific symmetry visible and testable.

### Table 13.2.2: Domain-to-Symmetry Grounding

| Domain | Organizational Symmetry Preserved | Conservation Law (§9) | Consequence of Domain Removal |
|---|---|---|---|
| 1. Authority & Decision Rights | Delegation invariance: same delegation → same scope regardless of organizational level | Scope preservation | Authority scope becomes invisible — delegations cannot be verified as consistent across recursion levels |
| 2. Commitment & Obligation | Obligation invariance: same agreement → same binding force regardless of context shift | Binding force | Commitment discharge becomes unobservable — promises exist but their fulfillment cannot be tracked |
| 3. Evidence, Record & Truth | Verification invariance: same evidence → same epistemic status regardless of presenter | Epistemic status | Truth type distinctions become invisible — Authoritative, Declared, and Derived evidence are indistinguishable in governance documents |
| 4. Work Execution & Lifecycle | Execution invariance: same transformation → same state change regardless of executor | State transformation fidelity | Work governance becomes opaque — state transitions occur without visible governance controls |
| 5. Access, Identity & Boundary | Boundary invariance: same rule → same prohibition regardless of role or preference | Rule universality | Constraint enforcement becomes invisible — rules exist but their universal application cannot be demonstrated |
| 6. Exception, Escalation & Override | Signal-routing symmetry: exceptions reach appropriate authority | Emergency signaling | Exception handling becomes unobservable — overrides occur without visible accountability |
| 7. Learning, Change & Evolution | Purpose invariance: learning produces decisions, not silent drift | Organizational direction | Adaptation becomes invisible — the system changes but the governance basis for change cannot be traced |

## §13.3 Four Canonical Projection Types


Four structural categories govern all document rendering. Every governance document maps to one or more types. The types determine output format, not just content — they are orthogonal to the seven domains.

### Table 13.3.1: Projection Types

| Type | What It Expresses | Derived From | Rendering Format | Example |
|---|---|---|---|---|
| **Type I: Boundary** | What is allowed or forbidden | Constraints, Authority limits, Invariants | Declarative statements, prohibition lists | "Only designated authorities may commit financial resources above $X." |
| **Type II: Authority** | Who can assert, decide, approve | Authority assignments, Delegation rules, Override paths | Matrices, responsibility charts, RACI variants | "Operational staff may initiate work but cannot finalize commitments without review." |
| **Type III: Evidence & Compliance** | What must be proven, and how | Evidence requirements, Confidence thresholds, Review rules | Checklists, evidence requirements, acceptance criteria | "All completed work cycles must include verifiable evidence before closure." |
| **Type IV: Procedural** | How work flows under policy | Cycle definitions, Decision points, Authority gates | Flowcharts, step sequences, decision trees | "When a request is received, it enters review, requires approval, proceeds to execution, and closes upon evidence acceptance." |

Types are orthogonal to domains. A single document may combine types across domains:

### Table 13.3.2: Type Coverage Examples

| Document | Domain(s) | Type(s) |
|---|---|---|
| Authority Matrix | 1 | II |
| Standard Operating Procedures | 4, 6 | IV, I |
| Policy Manual | 1, 2, 3, 5, 6 | I, II, III |
| Engagement Letter | 1, 2, 5 | I, II |
| Audit Trail Report | 3, 4, 6 | III |

## §13.4 Projection Pipeline


Projections follow a five-step rendering pipeline. No step adds governance — each only extracts, normalizes, or renders what the substrate already enforces.

### Step 1: Scope Selection

Every projection is scoped. There is no "global policy" without context. Scope dimensions: Entity, Role, Cycle, Integration, Time window. The scoping decision is a governance act — a human or organizational unit defines what governance surface to render.

### Step 2: Governing Structure Extraction

The system identifies relevant substrate structures within scope: applicable authority rules, active constraints, required evidence, applicable invariants. No interpretation at this step — extraction only. The output is a structured representation of the governance state that exists within the defined scope.

### Step 3: Semantic Normalization

Internal substrate logic is normalized into human-readable forms: declarative statements, conditional clauses, explicit thresholds. AI assistance is permitted at this step — it may normalize language without inventing rules. All AI-assisted normalization produces Derived truth type content (§6) and is reviewable. A projection containing AI-normalized language MUST NOT be published without human review.

### Step 4: Rendering

Output is rendered in the appropriate format for the projection type: human-readable text, flow diagrams (Type IV), checklists (Type III), matrices (Type II), structured documents. All projections are read-only artifacts. A projection cannot modify substrate state.

### Step 5: Provenance Annotation

Every projected document includes source primitive references, constraint IDs, authority references, version and timestamp, conformance test status, and truth type per content element. Provenance is not optional. A projection without provenance is a document, not a governance artifact.

### Table 13.4.1: Projection Pipeline Stages

| Step | Process | Input | Output | Truth Type Produced |
|---|---|---|---|---|
| 1. Scope Selection | Define governance surface boundaries | Entity, Role, Cycle, Integration, Time window parameters | Validated scope definition | Authoritative — scope is a human governance decision |
| 2. Governing Structure Extraction | Identify relevant substrate structures within scope | Scope definition + substrate state | Structured governance state representation | Authoritative — extraction only, no interpretation |
| 3. Semantic Normalization | Normalize internal logic into human-readable forms | Structured governance state | Human-readable declarative statements | Derived (AI output) → Declared after human review |
| 4. Rendering | Output in format appropriate for projection type | Normalized governance language + projection type | Formatted governance document | Derived — mechanical transformation |
| 5. Provenance Annotation | Attach source references, constraint IDs, authority references, timestamps, conformance status, truth type metadata | Rendered document + substrate metadata | Complete projected document with provenance | Authoritative — metadata from substrate state |

### Table 13.4.2: Actor Participation Per Pipeline Step

| Pipeline Step | Allowed Actor Types | Governance Posture | Truth Type Produced |
|---|---|---|---|
| Step 1: Scope Selection | Human, Organizational Unit | N/A (input decision) | Authoritative |
| Step 2: Governing Structure Extraction | System | Automation | Authoritative |
| Step 3: Semantic Normalization | AI Agent, Human | Assistive (AI proposes, human reviews) | Derived → Declared after human review |
| Step 4: Rendering | System, AI Agent | Automation | Derived |
| Step 5: Provenance Annotation | System | Automation | Authoritative |

AI may normalize language (Step 3) and render documents (Step 4), but AI-generated content enters as Derived truth type. This is consistent with the epistemic boundary maintenance architecture (§6.2): all AI-generated output lands in staging with truth type Derived, and no architectural path exists for AI output to bypass staging.

**SDK Constraints:**
- **MUST:** Enforce actor type constraints at each pipeline step boundary. A rendering step attempted without prior scope selection is rejected.
- **MUST:** Track truth type per content element through the pipeline. Every paragraph in the output document carries its truth type origin.
- **MUST NOT:** Publish a projection containing AI-normalized language (Step 3) without human review. The review promotes Derived content to Declared.
- **DESIGN SPACE:** Whether Steps 2–5 execute synchronously as a single pipeline invocation or asynchronously with intermediate persistence is an implementation choice.

## §13.5 Truth Type Tracking in Provenance


Every section of a projected document carries truth type metadata. This enables document consumers to know the epistemic status of every paragraph.

**Authoritative.** Content extracted directly from substrate state (Step 2) and system-generated provenance (Step 5). This content reflects what the substrate enforces. It is the projection's backbone — the claims that are structurally backed.

**Declared.** Human augmentation (§13.6) — explanatory language, regulatory citations, contextual examples. Attributed to a specific human author. Declared content is canonical (used in decisions) but not substrate-extracted — it is a human assertion added to the projection for communication purposes.

**Derived.** AI-normalized language (Step 3). Derived content MUST be flagged and reviewable. Rendered in a distinct visual style in document output, distinguishing it from Authoritative and Declared content. Derived content that has undergone human review and promotion becomes Declared, following the promotion workflow defined in §6.3.

The per-element truth type tracking closes an architectural loop. A consumer reading a projected authority matrix can distinguish: this delegation scope was extracted from the substrate (Authoritative — trustworthy), this explanatory note was added by a governance officer (Declared — attributed), and this summary paragraph was AI-generated (Derived — needs verification). The epistemic status of governance documentation is no longer implicit.

## §13.6 Human Augmentation Rules


Projections render substrate state. Humans may augment projections with additional context, but the augmentation boundary is enforced. Augmentation is communication — it may explain, illustrate, and reference, but it may not govern.

**Humans MUST be able to:**
- Add explanatory language (commentary, not governance)
- Add regulatory citations (external reference, not new constraint)
- Add contextual examples (illustration, not rule)
- Add non-enforceable guidance (recommendation, not mandate)

**Humans MUST NOT:**
- Alter projected constraints — creates a gap between document and enforcement
- Add authority not present in the model — creates phantom authority
- Remove enforcement requirements — creates false assurance

**DESIGN SPACE:** If a human adds content that contradicts substrate state, the implementation MUST flag it as non-authoritative commentary — visible, attributed, and clearly distinguished from projected content. This maintains the projection principle: the projection can only reveal what exists. Human commentary that contradicts substrate state is not an error — it is a signal that either the commentary is wrong or the substrate needs to be updated. The flag ensures the contradiction is visible rather than silently accepted.

All human augmentation carries truth type Declared. The attribution records which human added the content, when, and under what authority. Augmentation content is versioned — updates to human-added content create new versions, not in-place modifications. The original projected content (Authoritative) is never modified by augmentation.

## §13.7 Conformance Coupling


Projections are claims. Conformance tests are proof obligations. A projection is valid only if the substrate actually enforces what the projection asserts.

Without conformance backing, projections could be performative — documents that assert governance without enforcement. The conformance loop prevents this:

1. A projection expresses how the system claims to operate
2. A conformance test verifies the substrate actually enforces that claim
3. If conformance fails, one of three conditions holds: the projection is invalid (document claims something not enforced), the substrate is misconfigured (enforcement gap), or both

Conformance is structural integrity, not domain opinion. It answers "does the system do what the document says?" — not "should the system do this?" Conformance does not test business quality (whether the authority structure is sound), regulatory sufficiency (whether the evidence requirements satisfy a specific regulation), or moral correctness (whether the policy is right).

Each conformance test maps to a formal behavioral invariant (§5), grounding the test in the conservation law architecture rather than informal governance questions.

### Table 13.7.1: Invariant-Backed Conformance Tests

| Conformance Question | Formal Invariant | Projection Domain(s) | Failure Diagnosis |
|---|---|---|---|
| Does authority enforcement exist for the projected authority structure? | B5: Authority must be traceable | Domain 1 | Authority chains contain untraceable links — projected authority structure is not enforced |
| Are events immutable as the projection claims? | B3: Evidence requires Truth Type | Domain 3 | Events lack truth type classification — projected immutability claims are unverifiable |
| Are evidence requirements enforced as specified? | B3 + B1: Evidence requires Truth Type; Work requires Commitment | Domain 3, 4 | Evidence requirements not tied to work completion — projected evidence standards are decorative |
| Are overrides logged as the projection asserts? | B8: Signals route to authority | Domain 6 | Override signals not routed to authority chain — projected override logging is incomplete |
| Are cycles closed only with evidence as projected? | B1 + B3: Work requires Commitment; Evidence requires Truth Type | Domain 4 | Cycles close without evidence or commitment backing — projected lifecycle governance is hollow |
| Are commitments capacity-backed as projected? | B2: Commitment requires Capacity | Domain 2 | Commitments exceed available capacity — projected obligation governance masks overcommitment |
| Are decisions accountable as projected? | B4: Decision requires Account | Domain 1 | Decisions lack account context — projected decision governance is unauditable |
| Are constraints binding as projected? | B6: Constraint binds primitives | Domain 5 | Constraints not bound to governed objects — projected access governance is aspirational |
| Can all governed objects be flagged as projected? | B7: All objects flaggable | All domains | Some governed objects lack signal surfaces — projected governance has invisible blind spots |
| Do IQ resolutions produce decisions as projected? | B9: IQ resolution creates Decision | Domain 6 | Resolved queries produce no decisions or work — projected exception governance captures but never acts |

The ten conformance tests cover all ten behavioral invariants. Each test targets a specific governance claim that projections make. When a conformance test fails, the violated invariant identifies the governance failure class — the diagnosis is structural, not interpretive.

### Connection to the Governance Pipeline

The conformance coupling completes the three-stage governance pipeline defined in §11.8:

```
Governance Activation (§12) → populates governance state
Policy Projection (§13.1–§13.6) → renders governance state as documents
Conformance Tests (§13.7) → verifies documents match enforcement reality
```

Conformance failures feed back into the activation pipeline (§12) as governance gaps. A policy projection that claims enforcement where none exists produces a conformance failure, which registers as a gap in the closure roadmap. The pipeline is self-correcting.

## §13.8 Graduated Conformance Levels


Binary conformance (pass/fail) is insufficient. The seven verification types from §6.7 enable graduated conformance: a projection can be verified to different depths depending on its governance requirements.

### Table 13.8.1: Graduated Conformance Levels

| Conformance Level | Verification Types Required | Meaning |
|---|---|---|
| **Structural** | EXIST | The substrate has the structures the projection claims exist |
| **Operational** | EXIST, COMPLETE, CURRENT | The structures exist, are fully populated, and reflect current state |
| **Governed** | EXIST, COMPLETE, CURRENT, APPROVED | The structures are operationally complete and authority-approved |
| **Auditable** | EXIST, COMPLETE, CURRENT, APPROVED, CONSISTENT, COMPLIANT | Full audit-grade conformance — structures satisfy internal consistency and external standards |
| **Sovereign** | All 7 types including RATIFIED | Highest assurance — structures have received formal collective endorsement. Reserved for regulatory submissions. |

Each projected document carries its conformance level as metadata. A document at Structural conformance asserts less than one at Auditable conformance — the consumer knows the verification depth behind the projection's claims.

### Table 13.8.2: Profile-Level Minimum Conformance

| Profile | Minimum Conformance Level | Rationale |
|---|---|---|
| PAS | Structural (EXIST) | Minimal governance surface — structures must exist but full population is not required |
| BAS | Operational (EXIST, COMPLETE, CURRENT) | Operational governance — structures must be populated and current for day-to-day governance |
| EAS | Governed minimum; Auditable for regulatory documents | Full governance surface — authority-approved structures; regulatory documents require audit-grade conformance |

**SDK Constraints:**
- **MUST:** Every projected document carry its conformance level as queryable metadata.
- **MUST:** Conformance level computation use only the verification types present in the substrate's verification records for the relevant governance artifacts.
- **MUST NOT:** A document claim a conformance level higher than what its verification records support.
- **DESIGN SPACE:** Whether conformance level is computed eagerly (on every projection) or lazily (on demand) is an implementation choice.

## §13.9 Canonical Document Catalog


Projected documents are organized in three tiers by deployment necessity. The documents listed are illustrative examples — the catalog is extensible. Organizations may define additional projection templates within the structural constraints of the seven domains and four projection types.

### Table 13.9.1: Tier 1 — Foundational Documents (Always Projectable)

| Document | Domain(s) | Type(s) | Substrate Requirements |
|---|---|---|---|
| Organizational Charter | 1, 2, 5 | I, II | Intent, Authority, Commitment |
| Authority Matrix | 1, 6 | II | Authority, Delegation, Override |
| Policy Manual | 1, 2, 3, 5, 6 | I, II, III | All core primitives |
| Standard Operating Procedures | 4, 6 | IV | Work, Cycle, Authority gates |
| Role Descriptions | 1, 5 | II | Authority, Capacity, Boundary |
| Evidence Requirements Guide | 3, 4 | III | Evidence, Truth Type, Confidence |

Tier 1 documents are available from any substrate instance with sufficient governance state populated. They constitute the minimum viable governance projection surface.

### Table 13.9.2: Tier 2 — Operational Documents (Context-Dependent)

| Document | Domain(s) | Type(s) | Substrate Requirements |
|---|---|---|---|
| Service Level Agreement | 2, 3, 4 | I, III | Commitment, Evidence, Cycle |
| Engagement Letter | 1, 2, 5 | I, II | Authority, Commitment, Boundary |
| Work Order Template | 2, 4 | III, IV | Commitment, Work, Cycle |
| Compliance Checklist | 3, 4 | III | Evidence, Cycle, Completion |
| Change Request Form | 7 | III, IV | Learning, Authority over change |
| Incident Report Template | 6 | III, IV | Exception, Override, Event |

Tier 2 documents are available when the substrate instance has operational-level governance state — active work cycles, commitments, evidence production.

### Table 13.9.3: Tier 3 — Governance Documents (Regulated/Complex Entities)

| Document | Domain(s) | Type(s) | Substrate Requirements |
|---|---|---|---|
| Board Resolution Template | 1, 7 | II | Authority (highest level), Change |
| Audit Trail Report | 3, 4, 6 | III | Event, Evidence, Override |
| Risk Register | 5, 6 | I, III | Constraint, Exception, Boundary |
| Delegation of Authority Schedule | 1 | II | Authority, Delegation |
| Conflict of Interest Declaration | 5 | I | Boundary, Constraint |

Tier 3 documents are available for EAS profiles or BAS profiles with activated governance frameworks requiring regulatory-grade documentation.

### Table 13.9.4: Profile-Tier Alignment

| Profile | Default Projection Tier | Rationale |
|---|---|---|
| PAS | Tier 1 (subset) | Minimal governance surface — project charter, evidence log |
| BAS | Tier 1 + Tier 2 | Operational governance — full foundational and operational documents |
| EAS | Tier 1 + Tier 2 + Tier 3 | Complete governance surface including regulatory documents |

## §13.10 Cross-Domain Dependencies


Policy domains are not independent. They interact through shared primitive references. A change in one domain cascades to others through the substrate. These cascades are not incidental — they are consequences of the conservation laws (§9) that bind the primitive grammar.

### Table 13.10.1: Cross-Domain Cascade Mechanisms

| Source Domain | Affects | Cascade Mechanism | Conservation Law Backing |
|---|---|---|---|
| 1. Authority & Decision Rights | → 2, 4, 6, 7 | Authority changes affect who can commit, execute, override, and approve change | B1 (Work requires Commitment) and B2 (Commitment requires Capacity) are Authority-gated — scope preservation (§9.4) ensures delegation changes propagate |
| 2. Commitment & Obligation | → 3, 4 | New commitments create evidence requirements and work obligations | B1 and B3 create evidence obligations — binding force (§9.4) ensures commitment changes produce downstream requirements |
| 3. Evidence, Record & Truth | → 2, 4, 6 | Changed evidence standards affect commitment discharge, work completion, and exception resolution | Epistemic status (§9.4) — changed verification standards cascade because B3 binds evidence quality to all consuming primitives |
| 4. Work Execution & Lifecycle | → 2, 3 | New work creates commitments and evidence obligations | State transformation fidelity (§9.4) — B1 ensures work creates commitment linkage; work execution produces evidence |
| 5. Access, Identity & Boundary | → 1, 6 | Boundary changes affect authority scope and exception handling | Rule universality (§9.4) — B6 ensures constraint changes bind to all affected primitives across authority and exception domains |
| 6. Exception, Escalation & Override | → 1, 7 | Exceptions may trigger authority review and change processes | Emergency signaling (§9.4) — B8 ensures override signals route to authority; B9 ensures resolved exceptions produce decisions |
| 7. Learning, Change & Evolution | → ALL | Learning-driven changes cascade to all domains through modified constraints, authority, and evidence requirements | Organizational direction (§9.4) — adaptation modifies constraints (B6), which bind all primitives; changes require accountable decisions (B4) |

**Cascade handling.** Cross-domain cascades route through the substrate's authority and constraint resolution — the change propagation architecture ensures that ripple effects are properly governed. Projections re-render automatically when source substrate state changes. Stale projections are detectable through provenance timestamps and conformance test re-execution.

**SDK Constraints:**
- **MUST:** The projection engine detect when source substrate state has changed since the last rendering and flag the projection as stale.
- **MUST:** Stale projections be visually distinguished from current projections.
- **DESIGN SPACE:** Whether stale projections trigger automatic re-rendering or require explicit refresh is an implementation choice. The architecture requires that staleness is detectable, not that re-rendering is automatic.

## §13.11 Primitive-to-Domain Mapping


Every domain draws from specific primitives. The full 19×7 matrix grounds projections in substrate structure, ensuring that every governance document traces to specific primitive composition rather than abstract document categories.

### Table 13.11.1: Primitive-to-Domain Mapping

| Primitive | Tier | Domain 1 | Domain 2 | Domain 3 | Domain 4 | Domain 5 | Domain 6 | Domain 7 |
|---|---|---|---|---|---|---|---|---|
| Intent | 1 | ○ | ● | | | | | ● |
| Commitment | 1 | | ● | ● | | | | |
| Capacity | 1 | | ● | | ● | | | |
| Work | 1 | | | ● | ● | | | |
| Evidence | 1 | | ● | ● | ● | | ● | ● |
| Decision | 1 | ● | | | ● | | ● | ● |
| Authority | 1 | ● | ● | | ● | ● | ● | ● |
| Account | 1 | | | ● | ● | | | |
| Constraint | 1 | ● | | | | ● | ● | |
| Identifier | 2 | | | ● | | ● | | |
| Entity | 2 | ● | | | | ● | | |
| Context | 2 | ○ | ○ | ○ | ○ | ○ | ○ | ○ |
| Namespace | 2 | | | | | ● | | ● |
| Orientation | 3 | | ○ | | | | | ● |
| Learning | 3 | | | ● | | | | ● |
| Activation | 3 | | | | ● | | | |
| Interpretation | 4 | | | ● | | | ● | ● |
| Environment Interface | 4 | | | ● | | ● | ● | |
| Cycle | 5 | | ● | | ● | | | |

● = Primary source | ○ = Secondary source

### Design Rationale for Key Mappings

**Entity → Domain 1 + Domain 5.** Entities hold authority (Domain 1) and define boundaries (Domain 5). An Entity is the subject of authority delegation and the unit against which access boundaries are drawn. Every authority assignment targets an Entity; every boundary separates Entities.

**Context → all domains (secondary).** Context frames every governance operation but is never the primary source for any projection. A projected authority matrix operates within a Context (temporal, spatial, relational) but the authority structure itself derives from Authority and Decision primitives, not from Context. Context provides the "when and where" framing; the other primitives provide the "what and who."

**Learning → Domain 7 (primary) + Domain 3 (primary).** Learning is the mechanism of Domain 7 (Change & Evolution) — it is the primitive through which the system adapts. Learning also maps to Domain 3 (Evidence, Record & Truth) because the learning process produces evidence about what was learned, why, and with what confidence. The dual primary mapping reflects that organizational learning is simultaneously a change mechanism and an evidence-producing process.

**Cycle → Domain 2 + Domain 4.** Cycles govern commitment discharge periods (Domain 2) — a quarterly commitment is discharged within a quarterly Cycle. Cycles also govern work lifecycle phases (Domain 4) — work progresses through defined temporal phases. The mapping ensures that temporal governance is projected through both the commitment and work execution surfaces.

**Interpretation → Domain 6 + Domain 7.** Exception handling (Domain 6) requires interpretation of what went wrong — an AI system's meaning resolution under uncertainty is formalized through Interpretation. Learning (Domain 7) requires interpretation of what changed and why. Both domains consume the structured meaning-resolution that Interpretation provides.

**Environment Interface → Domain 5 + Domain 6.** Boundaries (Domain 5) are defined against external environments — the Environment Interface is the sensor surface where the substrate's interior meets external domains. Exceptions (Domain 6) often originate from environment changes — external system failures, regulatory shifts, or environmental disruptions that the Environment Interface detects.


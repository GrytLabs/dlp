# §7 Minimum Viable Record

The Minimum Viable Record (MVR) is the minimal set of fields required for a Tier 1 primitive instance to be compliant with the Decision Lineage Protocol. If a record contains all MVR fields with valid values, it is DLP-compliant. If it is missing any MVR field, it is rejected at write time. This section defines the required fields for each of the nine Tier 1 primitives, specifies how MVR validation operates alongside behavioral invariant validation, and establishes the extension framework for optional fields beyond the MVR base.

### §7.1 MVR Scope

The MVR applies to Tier 1 primitives only. This is a deliberate scope boundary, not an omission.

#### Table 7.1.1: MVR Scope by Tier

| Tier | Primitives | MVR Applies | Specification Mechanism |
|---|---|---|---|
| **1** (Irreducible Core) | Intent, Commitment, Capacity, Work, Evidence, Decision, Authority, Account, Constraint | Yes | This section (§7.2) |
| **2** (Infrastructure) | Identifier, Entity, Context, Namespace | No | Infrastructure specifications (§4.6) |
| **3** (Governed Operation) | Orientation, Learning, Activation | No | Profile-configurable via Namespace extensions (§7.4) |
| **4** (AI-Native) | Interpretation, Environment Interface | No | Profile-configurable via Namespace extensions (§7.4) |
| **5** (Configuration) | Cycle | No | Profile-configurable via Namespace extensions (§7.4) |

#### Rationale

**Tier 1 carries MVR because these nine primitives are the protocol's irreducible governance vocabulary.** Every DLP-compliant substrate must represent them. A universal field standard ensures that records are interoperable across substrates and auditable against a common specification. Without MVR, two substrate implementations could represent the same Commitment with incompatible field sets, making cross-substrate governance impossible.

**Tier 2 does not carry MVR because infrastructure primitives are substrate plumbing.** Identifier, Entity, Context, and Namespace are the mechanism by which Tier 1 instances are addressed, attributed, situated, and scoped. Their structure is defined by infrastructure specifications (§4.6), not by governance field requirements. An Identifier needs to be unique and resolvable; it does not need a governance field set.

**Tiers 3–5 do not carry MVR because their field requirements vary by organizational context.** An Enterprise Accountability Substrate (EAS) may require different Orientation fields than a Project Accountability Substrate (PAS). Fixing MVR fields for these tiers would either over-specify (forcing fields irrelevant to some profiles) or under-specify (defining a lowest common denominator too thin to be useful). Instead, Tiers 3–5 are governed through the Namespace extension framework (§7.4), where domain and tenant extensions define field requirements appropriate to each profile.

### §7.2 MVR Fields by Primitive

Each table below defines the required fields for one Tier 1 primitive. Every field listed is mandatory — a record missing any field is rejected at write time. The field definitions are authoritative; the TypeScript interfaces in `mvr-definitions.ts` and the SHACL shapes in `mvr-validation-shapes.ttl` are derived from this specification.

#### §7.2.1 Intent

| Field | Type | Description |
|---|---|---|
| `id` | Identifier | Unique identifier for this intent |
| `created_by` | Entity reference | Who expressed the intent |
| `created_at` | DateTime | When the intent was expressed |
| `description` | String | What is the objective |
| `priority` | String | Relative importance |
| `authority_scope` | Authority reference | Who has authority to act on this intent |

Intent is the origin point of governance action. Every downstream Commitment traces to an Intent, establishing the organizational purpose that justifies resource allocation.

#### §7.2.2 Commitment

| Field | Type | Description |
|---|---|---|
| `id` | Identifier | Unique identifier for this commitment |
| `committed_by` | Entity reference | Who is responsible |
| `committed_to` | Entity reference | Who they are accountable to |
| `committed_at` | DateTime | When the commitment was made |
| `intent_id` | Intent reference | What intent does this serve |
| `commitment_text` | String | What exactly are they committing to |
| `deadline` | DateTime | When is it due |
| `capacity_allocation` | Capacity reference | Resources promised |
| `authority_required` | Authority reference | What authority is needed |

Commitment links intent to execution. The `capacity_allocation` field connects to the Capacity primitive, enabling B2 enforcement (§5). The `intent_id` field ensures every commitment traces to an organizational objective.

#### §7.2.3 Capacity

| Field | Type | Description |
|---|---|---|
| `id` | Identifier | Unique identifier for this capacity |
| `owner` | Entity reference | Who has this capacity |
| `capacity_type` | String | What kind: time, expertise, budget, authority |
| `measurement_unit` | String | How to measure: hours, dollars, percentage |
| `total_available` | Number (≥ 0) | Total amount available |
| `allocated_to` | Commitment reference[] | Commitments using this capacity (one or more) |
| `reserved_for` | Commitment reference[] | Emergencies or priorities (one or more) |

Capacity is the feasibility check that grounds commitments in reality. Without verified capacity, commitments are promises without substance — the governance failure B2 prevents.

#### §7.2.4 Work

| Field | Type | Description |
|---|---|---|
| `id` | Identifier | Unique identifier for this work item |
| `work_type` | String | What kind of work: task, process, project |
| `assigned_to` | Entity reference | Who is doing it |
| `commitment_id` | Commitment reference | What commitment does it fulfill |
| `start_date` | Date | When it begins |
| `end_date` | Date | When it should complete |
| `status` | Enum | Current state: `planned`, `in_progress`, `blocked`, `completed` |
| `input_requirements` | String | What must be available |
| `output_description` | String | What will be produced |
| `process_steps` | String | How is it done |

Work is the state transformation primitive — the mechanism through which commitments become outcomes. The `commitment_id` field enables B1 enforcement (§5): no work without a commitment.

#### §7.2.5 Evidence

| Field | Type | Description |
|---|---|---|
| `id` | Identifier | Unique identifier for this evidence |
| `source` | String | Where it came from: person, system, document |
| `collected_at` | DateTime | When it was collected |
| `artifact_description` | String | What is it |
| `artifact_reference` | String | Where to find it (URI or path) |
| `truth_type` | Enum | Epistemic origin: `Authoritative`, `Declared`, `Derived`, `Opaque` |
| `relevant_to` | Primitive reference | What decision or claim does it support |
| `confidence_level` | Decimal [0.0, 1.0] | How confident in this evidence |

The `truth_type` field carries exactly four values from the truth type vocabulary (§6.1). This is the field enforced by invariant B3 (§5): every evidence artifact must be epistemically classified. The four truth types — Authoritative (canonical, append-only), Declared (human-asserted, versioned), Derived (AI-generated, ephemeral), Opaque (boundary-enforced, structurally uninspectable per §6.1) — are the complete epistemic vocabulary. Verification state, claims graduation stage, and record lifecycle state are orthogonal vocabularies tracked independently (§6).

#### §7.2.6 Decision

| Field | Type | Description |
|---|---|---|
| `id` | Identifier | Unique identifier for this decision |
| `decision_by` | Entity reference | Who decided |
| `authority_source` | Authority reference | What right do they have to decide |
| `decided_at` | DateTime | When the decision was made |
| `decision_scope` | String | What is being decided: entity, outcome, timeframe |
| `options_considered` | String[] | What choices were available (one or more) |
| `selected_option` | String | Which one was chosen |
| `rationale` | String | Why this option |
| `confidence` | Decimal [0.0, 1.0] | Confidence in the decision |
| `decision_type` | Enum | Classification: `selection`, `approval`, `rejection`, `delegation`, `escalation`, `deferral` |
| `orientation_thread` | Orientation reference | Snapshot reference capturing the active Orientation (§4.6.2) at decision time. Records the organizational direction against which this decision was made. Optional at Minimum depth; required at Complete depth. Enables drift computation: `compute_decision_deviation()` compares the decision outcome against the captured orientation to measure how far the decision deviated from organizational intent. |

Decision is the choice point where governance becomes action. The `authority_source` field links to an Authority instance, enabling verification that the decision-maker had the right to decide. Invariant B4 (§5) requires every Decision to link to an Account — that linkage is an inter-record relationship enforced by B4 shapes, not an MVR field on Decision itself.

#### §7.2.7 Authority

| Field | Type | Description |
|---|---|---|
| `id` | Identifier | Unique identifier for this authority |
| `authority_type` | String | What kind: approval, decision, oversight, veto |
| `holder` | Entity reference | Who has it |
| `delegated_from` | Authority reference | Source authority: person or role |
| `delegated_at` | DateTime | When delegation occurred |
| `scope_of_authority` | String | What can they decide about |
| `decision_limits` | String | How much can they approve |
| `expiration_date` | DateTime | When does it expire |
| `conditions` | String | Any restrictions |

Authority records the right to act. The `delegated_from` field creates the delegation chain that B5 (§5) enforces: every non-root authority must trace through a chain that terminates at a root. Root authorities satisfy B5 through the `isRootAuthority` flag rather than `delegated_from`; the B5 shape handles this disjunction.

#### §7.2.8 Account

| Field | Type | Description |
|---|---|---|
| `id` | Identifier | Unique identifier for this account event |
| `actor_id` | Entity reference | Who performed the action |
| `action_type` | Enum | What kind of action: `create`, `update`, `approve`, `reject`, `delegate`, `escalate`, `complete`, `revoke` |
| `action_reference` | Primitive reference | The primitive instance this action targeted |
| `timestamp` | DateTime | When the action occurred |
| `outcome` | String | Result: what changed, what was produced |
| `context_snapshot` | Key-value pairs | Relevant state at the time of action |
| `authority_chain` | Authority reference[] | Delegation path that legitimized the action (one or more) |

Account is a per-event linkage record. Each Account instance captures one governance action: who did what, when, to which primitive, and with what outcome. This is not a periodic aggregation — Account does not contain period boundaries, rollup counts, or summary statistics. When temporal framing is needed, aggregation queries scope by Cycle (§28).

The `context_snapshot` field captures the conditions under which the action was taken. This enables post-hoc audit to determine reasonableness: not just *what* was decided, but *what was known* at decision time. The `authority_chain` field records the delegation path from the actor's immediate authority to the root, enabling verification that the actor had the right to act.

Invariant B4 (§5) requires every Decision to link to an Account. The Decision references the Account event that captures state context at decision time. This relationship is enforced by B4 shapes as an inter-record constraint, not as an MVR field.

#### §7.2.9 Constraint

| Field | Type | Description |
|---|---|---|
| `id` | Identifier | Unique identifier for this constraint |
| `constraint_type` | String | Policy, compliance, physical, organizational |
| `applies_to` | Primitive reference[] | What primitives this applies to (one or more) |
| `constraint_text` | String | What exactly is required or forbidden |
| `enforcement_mode` | Enum | How enforced: `blocking`, `warning`, `logging`, `advisory` |
| `authority_source` | Authority reference | Who set this rule: policy, regulation, board |
| `effective_date` | Date | When it takes effect |
| `review_schedule` | String | When to revisit |

Constraint records the rules that govern governance itself. The `applies_to` field and `enforcement_mode` field together satisfy B6 (§5): every constraint must bind at least one primitive target and specify how it is enforced. The four enforcement modes correspond to the enforcement mode specification in §5.4.

### §7.3 Validation Architecture

MVR validation ensures that every Tier 1 record contains the fields required for governance. It operates alongside behavioral invariant validation (§5) as two complementary layers of SHACL enforcement.

#### Two Layers of Validation

| Layer | Concern | Shapes Prefix | Scope | Example |
|---|---|---|---|---|
| **MVR validation** | Intra-record integrity | `dlpmvr:` | Does this record have all required fields with correct types? | "Evidence must have a `truth_type` field" |
| **Invariant validation** | Inter-record relationship integrity | `dlps:` | Does this record maintain correct relationships with other records? | "B1: Work must link to a Commitment that exists" |

MVR shapes and invariant shapes target the same primitive classes. A `dlp:Evidence` instance is validated against both `dlpmvr:EvidenceShape` (are all MVR fields present?) and `dlps:B3-EvidenceRequiresTruthType` (is the truth type from the controlled vocabulary?). Both must pass for the record to be valid.

#### Conjunction Semantics

SHACL applies all shapes that target a given class via conjunction — a node must conform to every applicable shape, not just one. When a substrate loads both the MVR shapes graph and the invariant shapes graph, every primitive instance is validated against both layers automatically. No explicit join or ordering is required.

This conjunction produces the complete validation picture:

| Primitive | MVR Shape | Invariant Shapes | Combined Validation |
|---|---|---|---|
| Work | `dlpmvr:WorkShape` | `dlps:B1-WorkRequiresCommitment` | All fields present AND commitment link valid |
| Commitment | `dlpmvr:CommitmentShape` | `dlps:B2-CommitmentRequiresCapacity` | All fields present AND capacity link valid |
| Evidence | `dlpmvr:EvidenceShape` | `dlps:B3-EvidenceRequiresTruthType` | All fields present AND truth type in vocabulary |
| Decision | `dlpmvr:DecisionShape` | `dlps:B4-DecisionRequiresAccount` | All fields present AND account link valid |
| Authority | `dlpmvr:AuthorityShape` | `dlps:B5-AuthorityTraceable` | All fields present AND delegation chain valid |
| Constraint | `dlpmvr:ConstraintShape` | `dlps:B6-ConstraintBindsPrimitives` | All fields present AND target binding valid |

Intent, Capacity, and Account have MVR shapes but no dedicated invariant shapes targeting them directly. They participate in invariant enforcement as link targets — Capacity is referenced by B2 through Commitment, Account is referenced by B4 through Decision.

#### Shape Overlap

Some fields appear in both MVR shapes and invariant shapes. The `truth_type` field on Evidence is constrained by both `dlpmvr:EvidenceShape` (field must be present) and `dlps:B3-EvidenceRequiresTruthType` (field value must be in the three-value vocabulary). Similarly, the `applies_to` and `enforcement_mode` fields on Constraint appear in both `dlpmvr:ConstraintShape` and `dlps:B6-ConstraintBindsPrimitives`.

This overlap is intentional and harmless. SHACL conjunction semantics apply both constraints. If the constraints agree (and they do — the MVR shape checks presence, the invariant shape checks the value), conjunction produces correct combined validation. The overlap serves documentation clarity: each shapes file is self-contained and readable without cross-referencing.

#### Validation Timing

MVR validation runs on every write operation. It is a Pass 1 check (§5.3) — SHACL Core, always Blocking, O(1) per instance. A record missing an MVR field is rejected before invariant shapes are evaluated. This ordering is pragmatic: checking field presence before checking inter-record relationships avoids confusing error messages (a missing `commitment_id` field would trigger both the MVR shape and the B1 shape; running MVR first reports the root cause).

### §7.4 Extension Framework

Extensions add optional fields beyond the MVR for specialized functionality. Extensions do not change MVR compliance — a record with all required MVR fields is compliant regardless of whether extension fields are present or absent.

#### Namespace-Scoped Extensions

Extensions connect to the Namespace four-tier governance model (§4.6):

| Namespace Scope | Governance Authority | Field Examples | Lifetime |
|---|---|---|---|
| **Core** | DLP specification authority | The nine core primitives, behavioral invariants, MVR fields | Immutable after protocol lock |
| **Domain** | Industry standards bodies, sector working groups | GAGAS terminology, COSO controls vocabulary, IFRS concepts | Curated by domain authorities; shared across organizations in a sector |
| **Tenant** | Organizational governance authority (steward) | Organization-specific decision categories, custom risk flags | Managed by steward; private to one organization |
| **Sandbox** | Extension creator, with time limit | New concepts under evaluation, pilot program fields | Time-bounded; must graduate to Tenant or Domain before expiration |

MVR fields are Core namespace by definition. They are the protocol-level field requirements that every substrate must enforce. Domain, Tenant, and Sandbox extensions layer additional fields on top of the MVR base without modifying it.

#### SHACL Extension Layering

Extensions are expressed as SHACL shapes layered on top of MVR validation shapes. Each extension field maps to a `sh:property` constraint within an extension shape that targets the same primitive class as the base MVR shape.

The layering model:

**Layer 1: MVR base shapes** (`dlpmvr:` prefix). Always loaded. Enforce the universal field requirements defined in §7.2. These shapes are the same for every substrate, every profile, every organization.

**Layer 2: Extension shapes** (scoped by Namespace). Loaded per-profile. Add field requirements beyond the MVR. An extension shape for the Evidence primitive adds properties to `dlp:Evidence` — SHACL conjunction semantics apply the extension constraints alongside the MVR constraints automatically.

**Layer 3: Profile-specific extension sets.** Each substrate profile (EAS, BAS, PAS) activates a different combination of extension shapes based on organizational context. An Enterprise Accountability Substrate might activate Domain extensions for GAGAS terminology plus Tenant extensions for organization-specific risk categories. A Project Accountability Substrate might activate all four scopes.

Extension shapes follow the same naming convention as MVR shapes, scoped by Namespace tier:

| Shape Type | Naming Pattern | Example |
|---|---|---|
| MVR base | `dlpmvr:<Primitive>Shape` | `dlpmvr:EvidenceShape` |
| Domain extension | `dlpmvr:<Primitive>-ext-domain-<field>` | `dlpmvr:Evidence-ext-domain-regulatory_reference` |
| Tenant extension | `dlpmvr:<Primitive>-ext-tenant-<field>` | `dlpmvr:Evidence-ext-tenant-risk_category` |
| Sandbox extension | `dlpmvr:<Primitive>-ext-sandbox-<field>` | `dlpmvr:Evidence-ext-sandbox-pilot_flag` |

#### Extension Graduation

Extensions graduate upward through Namespace tiers: Sandbox → Tenant → Domain. Core extensions require protocol lock and are not part of runtime graduation.

Graduation follows the same evidence-accumulation model as claims graduation (§6.5): an extension proves its value at one tier before promotion to the next. The graduation lifecycle:

| Status | Meaning |
|---|---|
| **Active** | Operating at its current scope |
| **Proposed** | Nominated for promotion to next tier |
| **Under Review** | Being evaluated by the governing authority of the target tier |
| **Graduated** | Promoted — now operates at the higher tier |
| **Expired** | Sandbox extension not graduated before deadline |
| **Retired** | Explicitly removed from its scope |

Sandbox extensions carry an expiration date enforced by the substrate. If a Sandbox extension is not graduated to Tenant or Domain before its deadline, it expires — the extension shape is deactivated and the fields are no longer validated. This prevents experimental fields from accumulating indefinitely.

#### Extension Constraints

Extensions are subject to the same validation rules as MVR fields:

1. **Type enforcement.** Extension fields specify a data type. The extension shape enforces it.
2. **Cardinality enforcement.** Extension fields specify whether they are required within their scope. A Domain extension marked as required is enforced for all organizations in that domain.
3. **Scope isolation.** A Tenant extension for organization A does not affect organization B's validation. Extension shapes are activated per-Namespace — they validate only within the Namespace where they are defined.
4. **No MVR interference.** Extensions cannot weaken MVR constraints. An extension shape may add fields to a primitive but cannot remove or relax existing MVR field requirements. SHACL conjunction semantics enforce this structurally: the MVR shape always applies regardless of extension configuration.

## Scope

This section specifies field requirements for Tier 1 primitives only. It does NOT specify: how Tier 2 infrastructure primitives are implemented (§4.6.1), how Tiers 3–5 are extended beyond the MVR (§7.4), the detailed SHACL shape definitions (shapes graph, separate artifact), or operational validation procedures (substrate-specific).

## Locked Design Positions

**Locked.** MVR for nine Tier 1 primitives with defined field sets. Field minimalism principle: no required field without governance semantic necessity, invariant enforcement necessity, or cross-substrate interoperability necessity. Two-layer validation architecture: MVR shapes (intra-record) + invariant shapes (inter-record). Extension framework with Namespace-scoped extensions graduating from Sandbox to Domain.

## Implementation Requirements

SDK implementations MUST enforce all MVR fields as mandatory at write time. SDK implementations MUST reject records missing any MVR field before invariant validation begins. SDK implementations MUST support the extension layering model with SHACL conjunction semantics. SDK implementations MUST enforce Sandbox extension expiration dates.

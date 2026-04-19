# §26 Implementation Schema

This specification section defines implementation contracts for the Decision Lineage Protocol substrate.

The Decision Lineage Protocol substrate operates on a logical data model organized into four schema domains, approximately eighty tables, thirty-two controlled vocabulary types, and seventy-nine SHACL validation shapes. This section specifies the complete logical model — tables, fields, types, relationships, and constraints — that governs how the nineteen primitives, thirteen cross-primitive relationships, and ten state machines are represented in persistent storage.

This section contains no DDL. The logical model specified here is implemented by DDL and ontology artifacts in the SDK. §26 specifies the logical model that both implement.

---

## §26.1 Schema Architecture Overview

### §26.1.1 Four Schema Domains

The substrate logical model partitions into four schema domains. Each domain has a distinct namespace, governance authority, and evolutionary cadence.

### Table 26.1.1: Schema Domain Summary

| Domain | Namespace | Content | Tables |
|---|---|---|---|
| **DLP** | `dlp.*` | Core primitives, cross-primitive relationships, events, actors, delegations, work specifications, validated projections, KPIs | ~30 |
| **EDIM** | `edim.*` | Governance registry, activation rules, pattern signatures, gap register, change events, KPI measurement infrastructure | ~18 |
| **KI** | `ki.*` | Knowledge concepts, claims, evidence bindings, semantic mappings, ontology alignment, chunks | ~17 |
| **Content Packages** | `eas.*`, `bas.*`, `pas.*` | Profile-specific content: intent templates, evidence types, authority templates, graduation triggers, role mappings | ~15 |

**Domain boundaries are governance boundaries.** DLP tables are protocol-level — they exist in every substrate deployment. EDIM tables support the governance registry and are deployment-mandatory. KI tables support the knowledge integration layer and activate with Tier 3+. Content package tables are profile-specific — an EAS deployment loads `eas.*` tables; a PAS deployment loads `pas.*` tables.

### §26.1.2 Relationship to the Ontology

The ontology layer (defined in `protolex-ontology.ttl`) defines the nineteen primitive classes, their properties, and cross-primitive relationships using OWL under the Open World Assumption. The schema layer (this section) represents the same structure in relational form under the Closed World Assumption.

The two layers are complementary, not redundant:

| Layer | Assumption | Purpose | Artifact |
|---|---|---|---|
| Ontology (OWL) | Open World | Define what primitives *are*; enable advisory reasoning | `protolex-ontology.ttl` |
| Schema (SQL) | Closed World | Store primitive instances; enforce structural constraints | DDL schemas |
| Shapes (SHACL) | Closed World | Validate instances against constraints; two-pass enforcement | SHACL shapes |

Every table in the schema corresponds to a class or property in the ontology. Every ENUM in the schema corresponds to a controlled vocabulary in the ontology. The mapping is mechanical — the schema is the relational projection of the ontology.

### §26.1.3 Two-Layer SHACL Architecture

Shapes are organized in two layers that correspond to the two-pass validation strategy (§5.3):

**Ontology layer** (`protolex-ontology.ttl`): Class definitions, property definitions, relationship definitions. Nineteen primitive classes across five tiers. This is the logical model expressed in OWL.

**Shapes layer** (`invariant-shapes.ttl`, `verification-type-shapes.ttl`, and Phase 7 extension shapes): Constraint definitions that validate instances against the ontology. Seventy-nine total shapes across five domains: governance (17), actor (15), portfolio (6), orchestration (16), KPI (8), plus the Phase 1 foundation (10 invariant shapes, 7 verification type shapes).

Pass 1 (SHACL Core) shapes run on every write at O(1) per instance and are always Blocking. Pass 2 (SHACL-SPARQL) shapes require graph traversal and run on a configurable schedule with profile-configurable enforcement modes.

### §26.1.4 Universal Fields

Every table in the DLP domain carries a standard set of fields that enable truth type tracking, versioning, and temporal governance.

### Table 26.1.2: Universal Field Set

| Field | Type | Description | Enforcement |
|---|---|---|---|
| `instance_id` | UUID FK → `dlp.instances` | Substrate instance scope | NOT NULL |
| `truth_type` | ENUM (`authoritative`, `declared`, `derived`, `opaque`) | Epistemic origin per §6.1 | NOT NULL; B3 enforced on Evidence |
| `record_lifecycle_state` | ENUM (`active`, `superseded`) | Operational currency per §6 | NOT NULL DEFAULT `active` |
| `created_at` | TIMESTAMPTZ | Record creation timestamp | NOT NULL |
| `created_by` | UUID FK → `dlp.entities` | Creating actor | NOT NULL |
| `version` | INTEGER | Record version number | NOT NULL DEFAULT 1 |

**Truth type is universal.** Every governed record carries `truth_type` to classify its epistemic origin. On Evidence records, B3 enforces this field. On other records, it tracks provenance — a Decision created by an AI assistant carries `truth_type = derived` until a human promotes it.

**Record lifecycle state is universal.** Records are never deleted. Supersession creates a new record with `record_lifecycle_state = active`; the prior record transitions to `superseded`. The superseded record is retained for audit trail.

---

## §26.2 DLP Core Schema

The DLP domain contains the primitive tables, support tables, and relationship tables that implement the nineteen-primitive governance model.

### §26.2.1 Tier 1: Irreducible Core

Nine tables implement the nine irreducible primitives. Each table carries the universal field set (Table 26.1.2) plus primitive-specific fields.

### Table 26.2.1: Intent Table — `dlp.intents`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `intent_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `statement` | TEXT | NOT NULL | What is the objective |
| `owner_id` | UUID | FK → `dlp.entities`, NOT NULL | Who expressed the intent |
| `context_id` | UUID | FK → `dlp.contexts` | Situational context |
| `status` | ENUM | NOT NULL | `draft`, `active`, `achieved`, `abandoned`, `superseded` |
| `goal_type` | ENUM | NOT NULL | Intent classification |
| `priority` | JSONB | | Relative importance structure |
| `time_bound` | JSONB | | Temporal framing |
| `success_criteria` | JSONB | | Measurable conditions for achievement |
| `parent_intent_id` | UUID | FK self | Hierarchical intent decomposition |
| `derived_from_id` | UUID | FK → `dlp.orientations` | Pre-decisional framing source |
| `authority_scope_id` | UUID | FK → `dlp.authorities` | Governing authority |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `effective_from` | TIMESTAMPTZ | | Temporal validity start |
| `effective_to` | TIMESTAMPTZ | | Temporal validity end |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

### Table 26.2.2: Commitment Table — `dlp.commitments`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `commitment_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `status` | ENUM | NOT NULL | `proposed`, `active`, `fulfilled`, `breached`, `terminated` |
| `context_id` | UUID | FK → `dlp.contexts` | Situational context |
| `governing_authority_id` | UUID | FK → `dlp.authorities` | Governing authority |
| `derived_from_id` | UUID | FK → `dlp.intents` | Primary motivating intent (convenience FK; M:M via join table) |
| `commitment_text` | TEXT | NOT NULL | What is committed to |
| `deadline` | TIMESTAMPTZ | | When due |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `effective_from` | TIMESTAMPTZ | NOT NULL | Universal field |
| `effective_to` | TIMESTAMPTZ | | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

**Companion tables.** `dlp.commitment_parties` tracks the roles of entities in a commitment (obligor, obligee, witness, guarantor, plus RACIVG role per §22). `dlp.commitment_terms` tracks individual terms within a commitment.

### Table 26.2.3: Capacity Table — `dlp.capacities`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `capacity_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `owner_id` | UUID | FK → `dlp.entities`, NOT NULL | Who holds this capacity |
| `capacity_type` | ENUM | NOT NULL | `resource`, `capability`, `availability`, `constraint` |
| `status` | ENUM | NOT NULL | `available`, `allocated`, `exhausted`, `blocked` |
| `type_data` | JSONB | | Capacity-type-specific structure |
| `measurement_unit` | TEXT | | How measured (hours, dollars, percentage) |
| `total_available` | NUMERIC | CHECK (≥ 0) | Total amount |
| `context_id` | UUID | FK → `dlp.contexts` | Situational context |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `effective_from` | TIMESTAMPTZ | NOT NULL | Universal field |
| `effective_to` | TIMESTAMPTZ | | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

### Table 26.2.4: Work Table — `dlp.works`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `work_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `title` | TEXT | NOT NULL | Work item title |
| `work_type` | ENUM | NOT NULL | `task`, `process`, `project` |
| `owner_id` | UUID | FK → `dlp.entities`, NOT NULL | Assigned executor |
| `commitment_id` | UUID | FK → `dlp.commitments`, NOT NULL | Authorizing commitment (B1 enforcement) |
| `status` | ENUM | NOT NULL | `defined`, `ready`, `in_progress`, `blocked`, `review`, `complete`, `cancelled` |
| `context_id` | UUID | FK → `dlp.contexts` | Situational context |
| `inputs` | JSONB | | Required inputs |
| `outputs` | JSONB | | Expected outputs |
| `acceptance_criteria` | JSONB | | Completion criteria |
| `start_date` | DATE | | Planned start |
| `end_date` | DATE | | Planned end |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

The `commitment_id` FK is NOT NULL — this is the structural enforcement of B1 (Work requires Commitment). Every work item traces to exactly one authorizing commitment.

### Table 26.2.5: Evidence Table — `dlp.evidences`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `evidence_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `assertion` | TEXT | NOT NULL | What is this evidence about |
| `evidence_type` | ENUM | NOT NULL | Evidence classification |
| `source` | JSONB | NOT NULL | Provenance (person, system, document) |
| `captured_at` | TIMESTAMPTZ | NOT NULL | When collected |
| `captured_by` | UUID | FK → `dlp.entities`, NOT NULL | Who collected |
| `truth_type` | ENUM | NOT NULL | `authoritative`, `declared`, `derived`, `opaque` — B3 enforced |
| `confidence` | NUMERIC(3,2) | CHECK (0.0–1.0) | Confidence level |
| `verification_state` | ENUM | NOT NULL DEFAULT `unverified` | `unverified`, `verification_pending`, `verified`, `verification_failed`, `cannot_verify` — SM-10 Dimension 1 |
| `claims_graduation_stage` | ENUM | | `asserted`, `investigating`, `corroborated`, `verified`, `graduated` — SM-10 Dimension 3 |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | `active`, `superseded` |
| `promotion_workflow_state` | ENUM | | `staging`, `human_review`, `promoted`, `archived` — SM-10 Dimension 2 (for Derived evidence) |
| `work_id` | UUID | FK → `dlp.works` | Work that produced this evidence (nullable; not all evidence comes from work) |
| `artifact` | JSONB | | Artifact reference or content |
| `context_id` | UUID | FK → `dlp.contexts` | Situational context |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

The `truth_type` field enforces B3 — every evidence artifact carries exactly one epistemic classification. The `verification_state`, `promotion_workflow_state`, and `claims_graduation_stage` fields implement the three orthogonal dimensions of SM-10 (Truth Type Transitions). These dimensions are independent: an evidence record can simultaneously be `declared` (truth type), `verified` (verification state), and `corroborated` (claims graduation).

### Table 26.2.6: Decision Table — `dlp.decisions`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `decision_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `statement` | TEXT | NOT NULL | What was decided |
| `decision_type` | ENUM | NOT NULL | `selection`, `approval`, `rejection`, `delegation`, `escalation`, `deferral` |
| `made_by` | UUID | FK → `dlp.entities`, NOT NULL | Decision maker |
| `made_at` | TIMESTAMPTZ | NOT NULL | When decided |
| `authority_id` | UUID | FK → `dlp.authorities` | Primary legitimizing authority (convenience FK; M:M via join table) |
| `account_id` | UUID | FK → `dlp.accounts`, NOT NULL | State context at decision time (B4 enforcement) |
| `status` | ENUM | NOT NULL | `draft`, `final`, `superseded`, `reversed` |
| `context_id` | UUID | FK → `dlp.contexts` | Situational context |
| `decision_zone` | ENUM | | `zone_1`, `zone_2`, `zone_3` |
| `rationale` | JSONB | | Why this option |
| `rationale_type` | ENUM | | `evidence_based`, `authority_directed`, `policy_required`, `precedent_following`, `emergency` |
| `options_considered` | JSONB | | Options evaluated |
| `selected_option` | TEXT | | Chosen option |
| `state_before` | JSONB | | State snapshot before decision |
| `state_after` | JSONB | | State snapshot after decision |
| `confidence` | NUMERIC(3,2) | CHECK (0.0–1.0) | Decision confidence |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

The `account_id` FK is NOT NULL — this is the structural enforcement of B4 (Decision requires Account). Every decision links to exactly one account providing state context.

### Table 26.2.7: Authority Table — `dlp.authorities`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `authority_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `name` | TEXT | NOT NULL | Authority name |
| `authority_type` | ENUM | NOT NULL | `approval`, `decision`, `oversight`, `veto`, `advisory`, `delegated` |
| `holder_id` | UUID | FK → `dlp.entities`, NOT NULL | Who holds the authority |
| `parent_authority_id` | UUID | FK self | Delegation source (B5 enforcement) |
| `is_root_authority` | BOOLEAN | NOT NULL DEFAULT false | True for root authorities |
| `scope` | JSONB | | Authority scope conditions |
| `scope_type` | ENUM | | `full`, `domain_restricted`, `entity_restricted`, `temporal`, `conditional` |
| `source_type` | ENUM | | `primary`, `delegated`, `statutory`, `contractual`, `organizational` |
| `status` | ENUM | NOT NULL | `active`, `suspended`, `revoked`, `expired` |
| `delegation_rules` | JSONB | | Delegation constraints |
| `autonomy_level` | INTEGER | CHECK (0–3) | AI delegation autonomy per §A1 |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `effective_from` | TIMESTAMPTZ | NOT NULL | Universal field |
| `effective_to` | TIMESTAMPTZ | | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

B5 is enforced structurally: every row must satisfy `(is_root_authority = true) OR (parent_authority_id IS NOT NULL)`. This CHECK constraint replaces the v8 pattern of embedding delegation origin in a JSONB `source` field, which could not enforce B5 transitive chain validation.

### Table 26.2.8: Account Table — `dlp.accounts`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `account_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `entity_id` | UUID | FK → `dlp.entities`, NOT NULL | Owning entity |
| `entity_type` | ENUM | NOT NULL | Uses `dlp.governed_primitive_type` — what is being tracked |
| `account_type` | ENUM | NOT NULL | `balance`, `state`, `register`, `log` |
| `current_value` | JSONB | | Current state representation |
| `as_of` | TIMESTAMPTZ | NOT NULL | State timestamp |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

**Companion table.** `dlp.account_transactions` captures individual state changes within an account. Each transaction is append-only and immutable, carrying `transaction_type` (ENUM), `amount` (JSONB), `from_value`/`to_value` (JSONB), `occurred_at` (TIMESTAMPTZ), and `evidence_id` (FK → `dlp.evidences`).

### Table 26.2.9: Constraint Table — `dlp.constraints`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `constraint_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `name` | TEXT | NOT NULL | Constraint name |
| `constraint_type` | ENUM | NOT NULL | `policy`, `compliance`, `physical`, `organizational` |
| `description` | TEXT | | Detailed constraint description |
| `applies_to_class` | TEXT | NOT NULL | Target primitive class |
| `enforcement_mode` | ENUM | NOT NULL | `blocking`, `warning`, `logging`, `advisory` (B6 enforcement) |
| `shacl_shape_uri` | TEXT | | Reference to shapes graph |
| `authority_source_id` | UUID | FK → `dlp.authorities` | Who established this constraint |
| `scope` | JSONB | | Applicability conditions |
| `effective_from` | TIMESTAMPTZ | NOT NULL | When active |
| `effective_to` | TIMESTAMPTZ | | Expiration |
| `review_schedule` | INTERVAL | | Review cadence |
| `status` | ENUM | NOT NULL | `active`, `suspended`, `deprecated` |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

The Constraint primitive (§4.1) requires first-class persistent representation. B6 is enforced by requiring `enforcement_mode` NOT NULL and at least one row in `dlp.constraint_targets` per constraint.

### §26.2.2 Tier 2: Unconditional Infrastructure

Four tables implement the infrastructure primitives. These provide addressing, ownership, situational context, and scoping for all Tier 1 instances.

### Table 26.2.10: Entity Table — `dlp.entities`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `entity_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `entity_type` | ENUM | NOT NULL | `human`, `organization`, `role`, `system` |
| `name` | TEXT | NOT NULL | Display name |
| `status` | ENUM | NOT NULL | `active`, `inactive`, `suspended` |
| `parent_entity_id` | UUID | FK self | Recursive ownership |
| `capabilities` | TEXT[] | | Actor capabilities |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities` | Universal field |

The `entity_type` ENUM no longer includes a generic `agent` value. AI actors are registered through the Actor Layer (§26.4), which provides the four-type classification (human, ai_automation, ai_agentic, ai_assistive) and governance posture.

### Table 26.2.11: Identifier Table — `dlp.identifiers`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `identifier_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `owning_primitive_type` | ENUM | NOT NULL | Uses `dlp.governed_primitive_type` |
| `owning_primitive_id` | UUID | NOT NULL | Referenced primitive instance |
| `identifier_type` | ENUM | NOT NULL | `uuid`, `uri`, `tax_id`, `registration_number`, `external_ref` |
| `identifier_value` | TEXT | NOT NULL | The identifier value |
| `namespace_id` | UUID | FK → `dlp.namespaces` | Governing namespace |
| `issued_at` | TIMESTAMPTZ | | When issued |
| `issued_by` | UUID | FK → `dlp.entities` | Issuing authority |
| `is_current` | BOOLEAN | NOT NULL DEFAULT true | Version currency |
| `version_number` | INTEGER | NOT NULL DEFAULT 1 | Identifier version |
| `supersedes_id` | UUID | FK self | Prior version |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |

**Unique constraint:** `(instance_id, identifier_type, identifier_value)`.

### Table 26.2.12: Context Table — `dlp.contexts`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `context_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `context_type` | ENUM | NOT NULL | `temporal`, `spatial`, `relational`, `situational`, `interpretive`, `composite` |
| `scope` | JSONB | | Context scope conditions |
| `active` | BOOLEAN | NOT NULL DEFAULT true | Currency flag |
| `parent_context_id` | UUID | FK self | Context nesting |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `effective_from` | TIMESTAMPTZ | | Universal field |
| `effective_to` | TIMESTAMPTZ | | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

### Table 26.2.13: Namespace Table — `dlp.namespaces`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `namespace_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `namespace_uri` | TEXT | UNIQUE, NOT NULL | Canonical URI |
| `prefix` | TEXT | NOT NULL | Short prefix for references |
| `scope` | ENUM | NOT NULL | `core`, `domain`, `tenant`, `sandbox` — four governance scopes per §4.6.1 |
| `steward_id` | UUID | FK → `dlp.entities` | Namespace steward |
| `governance_authority_id` | UUID | FK → `dlp.authorities` | Governing authority |
| `description` | TEXT | | Namespace purpose |
| `parent_namespace_id` | UUID | FK self | Hierarchical nesting |
| `status` | ENUM | NOT NULL | `active`, `proposed`, `under_review`, `graduated`, `expired`, `retired` |
| `graduation_source_id` | UUID | FK self | For sandbox→tenant→domain graduation |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `updated_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

The four governance scopes (core, domain, tenant, sandbox) control who defines concepts and how definitions evolve. Namespace graduation follows the claims graduation model (§6.5) — extensions prove their value at one tier before promotion to the next.

### §26.2.3 Tier 3: Governed Operation

### Table 26.2.14: Orientation Table — `dlp.orientations`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `orientation_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `owner_id` | UUID | FK → `dlp.entities`, NOT NULL | Whose orientation |
| `value_stance` | JSONB | | Value priorities |
| `harm_boundary` | JSONB | | Harm thresholds |
| `time_horizon` | ENUM | NOT NULL | `real_time`, `short_term`, `long_term`, `balanced`, `explicit_mix` |
| `agency_model` | JSONB | | Agency configuration |
| `non_goals` | TEXT[] | | Explicit non-objectives |
| `status` | ENUM | NOT NULL | `active`, `inactive`, `suspended` |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

### Table 26.2.15: Learning Table — `dlp.learnings`

**Design note.** The Learning primitive operates at the intersection of two models. §4.6.2 defines institutional learning as a four-stage knowledge flow (Crossan/Lane/White 4I framework: Intuition → Interpretation → Integration → Institutionalization). §A2.5 defines the substrate's operational mechanics: pattern detection → human validation → threshold adjustment → structural reconfiguration. The schema models the operational mechanics as the primary structure (what the code does at runtime) and carries the 4I stage as an annotation column (where this learning event sits in the organizational knowledge flow). This avoids an abstraction mismatch where queries would need to reason about organizational theory rather than substrate operations.

| Field | Type | Constraints | Description |
|---|---|---|---|
| `learning_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `learning_type` | ENUM | NOT NULL | `pattern_observation`, `pattern_validation`, `threshold_adjustment`, `structural_reconfiguration` — maps to §A2.5 pipeline stages |
| `detection_scope` | ENUM | NOT NULL | `work_type_cluster`, `quality_pattern`, `cognitive_type_signal`, `routing_divergence`, `threshold_miscalibration` — what the substrate observed (Table A2.5.1) |
| `source_type` | ENUM | NOT NULL | Uses `dlp.governed_primitive_type` |
| `source_id` | UUID | NOT NULL | Source primitive instance |
| `target_type` | ENUM | NULL | Uses `dlp.governed_primitive_type` — NULL for observations not yet linked to targets |
| `target_id` | UUID | NULL | Target primitive instance — NULL for observations not yet linked to targets |
| `status` | ENUM | NOT NULL | `detected`, `awaiting_review`, `validated`, `applied`, `rejected`, `monitoring` |
| `governance_level` | ENUM | NOT NULL | `l1_operational`, `l2_tactical`, `l3_strategic`, `l4_orientation` |
| `organizational_learning_stage` | ENUM | NULL | `intuition`, `interpretation`, `integration`, `institutionalization` — 4I annotation (§4.6.2); NULL when stage is not yet classifiable |
| `learning_mode` | ENUM | NULL | `exploration`, `exploitation` — dual-mode governance annotation (§4.6.2); NULL when mode is not applicable |
| `evidence_base` | UUID[] | | References to Evidence records supporting this learning event |
| `promotion_decision_id` | UUID | FK → `dlp.decisions`, NULL | Decision record that validated this learning (NULL until `validated` status) |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

### Table 26.2.16: Activation Table — `dlp.activations`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `activation_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `target_type` | ENUM | NOT NULL | Uses `dlp.governed_primitive_type` |
| `target_id` | UUID | NOT NULL | Target primitive instance |
| `status` | ENUM | NOT NULL | `pending`, `ready`, `active`, `blocked`, `complete` — operational status |
| `governance_stage` | ENUM | NOT NULL | `discover`, `analyze`, `implement`, `evidence`, `verify`, `sustain` — six-stage pipeline per §12 |
| `blockers` | JSONB | | Blocking conditions (structured: blocker_type, description, blocking_primitive_type, blocking_primitive_id) |
| `progress` | JSONB | | Stage progress metrics |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

The `governance_stage` field aligns with the six-stage activation pipeline defined in §12. The `status` field tracks operational state; `governance_stage` tracks governance maturity progression independently.

### §26.2.4 Tier 4: AI-Native Extensions

### Table 26.2.17: Interpretation Table — `dlp.interpretations`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `interpretation_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `interpretation_type` | ENUM | NOT NULL | Fourteen types: `signal`, `term`, `constraint`, `scope`, `intent`, `exception`, `classification`, `extraction`, `recommendation`, `analysis`, `generation`, `translation`, `prediction`, `gap_identification` |
| `subject_type` | ENUM | NOT NULL | Uses `dlp.governed_primitive_type` |
| `subject_id` | UUID | NOT NULL | Subject being interpreted |
| `question` | TEXT | | Interpretation question |
| `status` | ENUM | NOT NULL | `pending`, `agent_proposed`, `human_review`, `resolved`, `disputed`, `withdrawn` |
| `decision_zone` | ENUM | | Routing zone |
| `stakes_assessment` | ENUM | | `low`, `high` |
| `novelty_assessment` | ENUM | | `low`, `high` |
| `routing_matrix_cell` | TEXT | | Stakes×novelty cell identifier |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

The `interpretation_type` ENUM extends the six-type vocabulary with eight AICAR resolution types from §A1: classification, extraction, recommendation, analysis, generation, translation, prediction, gap_identification.

### Table 26.2.18: Environment Interface Table — `dlp.environment_interfaces`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `interface_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `interface_type` | ENUM | NOT NULL | `inbound_signal`, `outbound_action`, `feedback_loop` |
| `direction` | ENUM | NOT NULL | `inward`, `outward` — per §4.6.3 bidirectional specification |
| `external_party` | JSONB | | External system details |
| `party_type` | ENUM | | `client`, `prospect`, `regulator`, `market`, `partner`, `public`, `competitor`, `system` |
| `dbi_perimeter_ref` | TEXT | | DBI perimeter reference for §10 integration |
| `policy_projection_domain` | ENUM | | PPD 5 (Access, Identity & Boundary) or PPD 6 (Exception, Escalation & Override) reference |
| `status` | ENUM | NOT NULL | `active`, `dormant`, `closed` |
| `type_data` | JSONB | | Interface-type-specific data |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

### §26.2.5 Tier 5: Operational Configuration

### Table 26.2.19: Cycle Table — `dlp.cycles`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `cycle_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `cadence_id` | UUID | FK → `dlp.cycle_cadences`, NOT NULL | Cadence definition |
| `cycle_type` | ENUM | NOT NULL | `execution_pulse`, `activation_sweep`, `weekly_review`, `learning_integration`, `orientation_check`, `commitment_reconciliation` |
| `sequence_number` | INTEGER | NOT NULL | Position within cadence |
| `scheduled_at` | TIMESTAMPTZ | NOT NULL | Planned execution |
| `started_at` | TIMESTAMPTZ | | Actual start |
| `completed_at` | TIMESTAMPTZ | | Actual completion |
| `status` | ENUM | NOT NULL | `scheduled`, `running`, `completed`, `skipped`, `failed` |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `version` | INTEGER | NOT NULL DEFAULT 1 | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

**Companion tables.** `dlp.cycle_cadences` defines repeating temporal patterns. `dlp.cycle_dependencies` captures inter-cycle ordering requirements.

### §26.2.6 Support Tables

### Table 26.2.20: Instance Table — `dlp.instances`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `instance_id` | UUID | PK | Unique identifier |
| `profile_type` | ENUM | NOT NULL | `eas`, `bas`, `pas` |
| `graduation_stage` | ENUM | NOT NULL DEFAULT `A` | `A`, `A_plus`, `B`, `C` — SM-2 governance maturity |
| `status` | ENUM | NOT NULL | `active`, `archived` |
| `parent_instance_id` | UUID | FK self | Portfolio hierarchy per §23 |
| `constraint_cascade_parent_id` | UUID | FK self | Tighten-only cascade source |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

### Table 26.2.21: Event Table — `dlp.events`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `event_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `event_type` | ENUM | NOT NULL | `create`, `update`, `delete`, `status_change`, `delegation`, `escalation`, `override`, `violation`, `graduation`, `activation` |
| `event_category` | ENUM | NOT NULL | `primitive_lifecycle`, `governance_activation`, `signal_processing`, `iq_resolution`, `delegation`, `authority_change`, `constraint_violation`, `graduation` |
| `target_type` | ENUM | NOT NULL | Uses `dlp.governed_primitive_type` |
| `target_id` | UUID | NOT NULL | Target primitive instance |
| `action` | ENUM | NOT NULL | `create`, `update`, `approve`, `reject`, `delegate`, `escalate`, `complete`, `revoke`, `archive`, `suspend`, `reinstate` |
| `actor_id` | UUID | FK → `dlp.entities`, NOT NULL | Who performed the action |
| `event_data` | JSONB | | Event-specific payload |
| `occurred_at` | TIMESTAMPTZ | NOT NULL | Event timestamp |

The events table is append-only and immutable. It serves as the audit log for all substrate operations.

### Table 26.2.22: Signal Flags Table — `dlp.signal_flags`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `signal_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `object_type` | ENUM | NOT NULL | Uses `dlp.governed_primitive_type` — all 19 primitive types |
| `object_id` | UUID | NOT NULL | Flagged object |
| `signal_type` | ENUM | NOT NULL | Includes `reservation` per §15 |
| `reservation_nature` | ENUM | | `concern`, `uncertainty`, `disagreement`, `precedent_question`, `scope_question`, `timing_question`, `resource_question` — when signal_type = reservation |
| `signal_text` | TEXT | NOT NULL | Description of the signal |
| `signal_status` | ENUM | NOT NULL DEFAULT `created` | `created`, `routed`, `processing`, `escalated`, `monitoring`, `resolved`, `dismissed` — SM-7 lifecycle |
| `raised_by` | UUID | FK → `dlp.entities`, NOT NULL | Who raised the signal |
| `raised_at` | TIMESTAMPTZ | NOT NULL | When raised |
| `routed_to` | UUID | FK → `dlp.authorities` | B8 routing target |
| `acknowledged_by` | UUID | FK → `dlp.entities` | Who acknowledged |
| `acknowledged_at` | TIMESTAMPTZ | | When acknowledged |
| `resolved_by` | UUID | FK → `dlp.entities` | Who resolved |
| `resolved_at` | TIMESTAMPTZ | | When resolved |
| `resolution_summary` | TEXT | | Resolution description |
| `dismissal_rationale` | TEXT | | Why dismissed |
| `dismissed_by` | UUID | FK → `dlp.entities` | Who dismissed |
| `dismissed_at` | TIMESTAMPTZ | | When dismissed |
| `escalation_history` | JSONB | | Escalation audit trail |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |

The `signal_status` ENUM replaces the v8 `processed` BOOLEAN with the seven-state SM-7 lifecycle. The `object_type` ENUM accepts all nineteen primitive types.

### Table 26.2.23: Investigative Query Table — `dlp.investigative_queries`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `iq_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `iq_number` | TEXT | NOT NULL | Formatted identifier |
| `intent_type` | ENUM | NOT NULL | `explore`, `validate`, `disambiguate`, `fill_gap`, `challenge`, `connect`, `branch_inventory`, `confidence_probe` — eight types per §16 |
| `origin_type` | ENUM | NOT NULL | `claim`, `signal`, `capture`, `workflow` |
| `status` | ENUM | NOT NULL | `open`, `assigned`, `investigating`, `resolved`, `promoted`, `abandoned`, `superseded` — SM-8 lifecycle |
| `source_type` | ENUM | | Uses `dlp.governed_primitive_type` |
| `source_id` | UUID | | Source primitive |
| `question` | TEXT | NOT NULL | The investigative question |
| `assigned_to` | UUID | FK → `dlp.entities` | Assigned investigator |
| `assignment_method` | ENUM | | `manual`, `auto_routed`, `inherited` |
| `promotion_targets` | JSONB | | For Promoted state: [{target_type, target_id}] |
| `abandonment_rationale` | TEXT | | For Abandoned state |
| `reversion_evidence_id` | UUID | FK → `dlp.evidences` | When Resolved → Open on new evidence |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

B9 enforcement: when `status = resolved`, a linked Decision or Work item must exist (enforced by SHACL-SPARQL shape B9-IQResolutionCreatesDecision).

---

## §26.3 Cross-Primitive Relationship Schema

The thirteen cross-primitive relationships defined in §4.4 are implemented through a combination of direct FK columns and join tables. Direct FKs handle 1:M relationships. Join tables handle M:M relationships and provide named relationship metadata.

### §26.3.1 Direct FK Implementations

### Table 26.3.1: Direct FK Relationships

| Relationship | From Table | FK Column | To Table | Cardinality | Invariant | Nullability |
|---|---|---|---|---|---|---|
| **authorizes** | `dlp.works` | `commitment_id` | `dlp.commitments` | 1:M | B1 | NOT NULL |
| **contextualizes** | `dlp.decisions` | `account_id` | `dlp.accounts` | 1:M | B4 | NOT NULL |
| **delegates** | `dlp.authorities` | `parent_authority_id` | `dlp.authorities` | 1:M | B5 | Nullable (root authorities are NULL) |
| **produces** | `dlp.evidences` | `work_id` | `dlp.works` | 1:M | — | Nullable (external evidence has no work source) |

The NOT NULL FKs on `works.commitment_id` and `decisions.account_id` are the structural enforcement of B1 and B4 respectively. These constraints cannot be deferred or profile-configured — they hold universally across all substrate deployments.

### §26.3.2 Join Table Implementations

Join tables implement M:M relationships and invariant-required relationship tables.

### Table 26.3.2: Join Table Inventory

| Relationship | Join Table | Columns | Invariant | Purpose |
|---|---|---|---|---|
| **motivates** (Intent → Commitment) | `dlp.intent_commitment_links` | `intent_id` FK, `commitment_id` FK, `link_type` ENUM, `created_at` | — | M:M — multiple intents motivate one commitment; one intent motivates commitments across boundaries |
| **frames** (Intent → Decision) | `dlp.intent_decision_links` | `intent_id` FK, `decision_id` FK, `framing_role` TEXT, `created_at` | — | M:M — multiple intents frame the evaluative context for a decision |
| **requires** (Commitment → Capacity) | `dlp.capacity_allocations` | `allocation_id` PK, `commitment_id` FK, `capacity_id` FK, `allocated_amount` JSONB, `allocated_at` TIMESTAMPTZ, `status` ENUM, `truth_type` ENUM | B2 | B2 enforcement: every commitment must have ≥1 allocation row |
| **enables** (Capacity → Work) | `dlp.capacity_work_links` | `capacity_id` FK, `work_id` FK, `allocation_amount` JSONB, `linked_at` TIMESTAMPTZ | — | M:M — makes feasibility relationship queryable |
| **evaluates** (Decision → Evidence) | `dlp.decision_evidence_links` | `decision_id` FK, `evidence_id` FK, `evaluation_role` ENUM, `created_at` | — | M:M — which evidence a decision weighed, in what role (supporting, refuting, contextualizing) |
| **records** (Account → Evidence) | `dlp.account_evidence_links` | `account_id` FK, `evidence_id` FK, `recording_role` TEXT, `recorded_at` TIMESTAMPTZ | — | Account-level evidence accumulation beyond transaction-level |
| **legitimizes** (Authority → Decision) | `dlp.decision_authority_links` | `decision_id` FK, `authority_id` FK, `legitimization_type` ENUM, `created_at` | — | M:M — dual authorization (primary, co_authorizer, reviewer) |
| **governs** (Authority → Primitive) | `dlp.authority_governance_links` | `authority_id` FK, `governed_type` ENUM, `governed_id` UUID, `governance_role` TEXT, `effective_from` TIMESTAMPTZ, `effective_to` TIMESTAMPTZ | — | Authority-to-primitive scope mapping |
| **constrains** (Constraint → Primitive) | `dlp.constraint_targets` | `constraint_id` FK, `target_type` ENUM, `target_id` UUID, `applied_at` TIMESTAMPTZ | B6 | B6 enforcement: every constraint must have ≥1 target row |

### §26.3.3 Invariant Enforcement through Schema

Five behavioral invariants are enforced structurally through the schema:

### Table 26.3.3: Schema-Level Invariant Enforcement

| Invariant | Schema Mechanism | Enforcement Level |
|---|---|---|
| **B1** (Work requires Commitment) | `dlp.works.commitment_id` NOT NULL FK → `dlp.commitments` | Write-time rejection |
| **B2** (Commitment requires Capacity) | `dlp.capacity_allocations` with CHECK: every `commitment_id` in `dlp.commitments` has ≥1 row | Deferred constraint or SHACL Pass 1 |
| **B4** (Decision requires Account) | `dlp.decisions.account_id` NOT NULL FK → `dlp.accounts` | Write-time rejection |
| **B5** (Authority traceable) | `dlp.authorities` CHECK: `(is_root_authority = true) OR (parent_authority_id IS NOT NULL)` | Write-time rejection |
| **B6** (Constraint binds primitives) | `dlp.constraints.enforcement_mode` NOT NULL + ≥1 row in `dlp.constraint_targets` | Write-time rejection + SHACL Pass 1 |

B3 (Evidence requires Truth Type) is enforced by the `truth_type` NOT NULL ENUM constraint on `dlp.evidences`. B7 (All objects flaggable) is enforced at the schema level by the `object_type` ENUM accepting all nineteen primitive types. B8 and B9 require graph traversal and are enforced by SHACL-SPARQL shapes in Pass 2.

---

## §26.4 Actor & Delegation Schema

The Actor Layer (§22) introduces dedicated tables for actor registration, role envelopes, and delegation lifecycle — replacing the v8 pattern of embedding actor semantics within the generic Entity table.

### Table 26.4.1: Actor Table — `dlp.actors`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `actor_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `entity_id` | UUID | FK → `dlp.entities`, NOT NULL | Link to base entity |
| `actor_type` | ENUM | NOT NULL | `human`, `ai_automation`, `ai_agentic`, `ai_assistive` — four types per §22 |
| `governance_posture` | ENUM | NOT NULL DEFAULT `standard` | `standard`, `enhanced`, `critical` |
| `parent_entity_id` | UUID | FK → `dlp.entities` | Organizational owner (mandatory for AI actors) |
| `actor_status` | ENUM | NOT NULL DEFAULT `registered` | `registered`, `active`, `suspended`, `deactivated`, `archived` — SM-1 lifecycle |
| `current_behavioral_state` | ENUM | | `listening`, `context_loading`, `navigating`, `proposing`, `monitoring`, `alerting`, `executing`, `completing` — SM-6 runtime state (AI actors only) |
| `activation_decision_id` | UUID | FK → `dlp.decisions` | Decision authorizing activation |
| `suspension_decision_id` | UUID | FK → `dlp.decisions` | Decision authorizing suspension |
| `deactivation_decision_id` | UUID | FK → `dlp.decisions` | Decision authorizing deactivation |
| `archival_decision_id` | UUID | FK → `dlp.decisions` | Decision authorizing archival |
| `registered_at` | TIMESTAMPTZ | NOT NULL | Registration timestamp |
| `activated_at` | TIMESTAMPTZ | | Activation timestamp |
| `suspended_at` | TIMESTAMPTZ | | Suspension timestamp |
| `deactivated_at` | TIMESTAMPTZ | | Deactivation timestamp |
| `archived_at` | TIMESTAMPTZ | | Archival timestamp |
| `registered_by` | UUID | FK → `dlp.entities`, NOT NULL | Who registered this actor |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |

Every SM-1 state transition requires a Decision reference. The `current_behavioral_state` field tracks SM-6 runtime state for AI actors — enabling governance auditing of agent behavioral cycles without requiring a separate audit table for the common query case.

### Table 26.4.2: Role Envelope Table — `dlp.role_envelopes`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `envelope_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `actor_id` | UUID | FK → `dlp.actors`, NOT NULL | Owning actor |
| `racivg_role` | ENUM | NOT NULL | `R`, `A`, `C`, `I`, `V`, `G` — per §22 RACIVG model |
| `scope` | JSONB | NOT NULL | Governance scope conditions |
| `permitted_actions` | JSONB | NOT NULL | Allowed operations within scope |
| `decision_rights` | JSONB | NOT NULL | Decision types and limits |
| `constraints` | JSONB | NOT NULL | Envelope constraints |
| `escalation_thresholds` | JSONB | NOT NULL | When to escalate |
| `execution_modality` | ENUM | NOT NULL | `human_only`, `ai_assisted`, `ai_executed_with_review` |
| `context_modifiers` | JSONB | | Phase, severity, risk_posture, confidence, automation_coverage modifiers (commutative per §22.6) |
| `effective_from` | TIMESTAMPTZ | NOT NULL | Envelope activation |
| `effective_to` | TIMESTAMPTZ | | Envelope expiration |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

Role envelopes define what an actor can do within a governance scope. The RACIVG role determines the envelope's position in the accountability model: EAS deployments use all six roles; BAS deployments restrict to R, A, C, I (V and G inactive); PAS deployments map to simplified Owner (R+A) and Collaborator (C+I) roles.

### Table 26.4.3: Delegation Table — `dlp.delegations`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `delegation_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `holder_id` | UUID | FK → `dlp.actors`, NOT NULL | Delegation holder |
| `source_authority_id` | UUID | FK → `dlp.authorities`, NOT NULL | Source authority being delegated |
| `scope` | JSONB | NOT NULL | Delegated scope (must be ⊆ source authority scope) |
| `autonomy_level` | INTEGER | NOT NULL, CHECK (0–3) | 0=manual, 1=observed, 2=suggested, 3=delegated |
| `delegation_status` | ENUM | NOT NULL DEFAULT `proposed` | `proposed`, `active`, `modified`, `suspended`, `revoked`, `expired` — SM-5 lifecycle |
| `grant_decision_id` | UUID | FK → `dlp.decisions`, NOT NULL | Decision authorizing grant |
| `granted_by` | UUID | FK → `dlp.entities`, NOT NULL | Granting human |
| `granted_at` | TIMESTAMPTZ | NOT NULL | Grant timestamp |
| `modification_decision_id` | UUID | FK → `dlp.decisions` | Most recent modification decision |
| `suspension_decision_id` | UUID | FK → `dlp.decisions` | Suspension decision |
| `revocation_decision_id` | UUID | FK → `dlp.decisions` | Revocation decision |
| `expiration_timestamp` | TIMESTAMPTZ | | Auto-expiration time |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |

SM-5 transition guards: every state transition except Expired requires a Decision reference. Expired fires automatically on timestamp. The `autonomy_level` must not exceed the graduation stage of the instance scope (Stage A → 0, Stage A+ → 1, Stage B → 2, Stage C → 3).

### Table 26.4.4: Agent Behavioral Events — `dlp.agent_behavioral_events`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `event_id` | UUID | PK | Unique identifier |
| `actor_id` | UUID | FK → `dlp.actors`, NOT NULL | Agent actor |
| `delegation_id` | UUID | FK → `dlp.delegations`, NOT NULL | Active delegation |
| `from_state` | ENUM | NOT NULL | SM-6 source state |
| `to_state` | ENUM | NOT NULL | SM-6 target state |
| `transition_trigger` | TEXT | NOT NULL | What caused the transition |
| `track` | ENUM | NOT NULL | `navigation`, `monitoring`, `execution` — three behavioral tracks |
| `occurred_at` | TIMESTAMPTZ | NOT NULL | Transition timestamp |
| `truth_type` | ENUM | NOT NULL DEFAULT `authoritative` | System-generated audit record |

This table provides the full audit trail for SM-6 behavioral cycles. The `current_behavioral_state` on `dlp.actors` provides the current-state query shortcut; this table provides the complete transition history.

---

## §26.5 Orchestration Schema

The orchestration schema implements §A1 (Agent Runtime Infrastructure) and §A2 (Work Orchestration) — the AI-native work specification model with cognitive decomposition and five-dimensional routing.

### Table 26.5.1: Work Specification Table — `dlp.work_specifications`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `work_spec_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `work_id` | UUID | FK → `dlp.works`, NOT NULL | Associated work item |
| `cognitive_decomposition` | JSONB | NOT NULL | Task decomposition structure |
| `actor_assignments` | JSONB | NOT NULL | Actor-to-task mapping |
| `effective_constraints` | JSONB | NOT NULL | Active constraints for this specification |
| `composition_strategy` | ENUM | NOT NULL | `sequential`, `parallel`, `iterative`, `conditional`, `delegated`, `federated` — six composition patterns per §A1 |
| `cognitive_type` | ENUM | NOT NULL | `A_citation`, `B_reasoning`, `C_context`, `D_communication` — AICAR types |
| `consequence_level` | ENUM | NOT NULL | `reversible`, `consequential`, `structural` |
| `operating_posture` | ENUM | NOT NULL | `normal`, `degraded`, `emergency` |
| `graduation_stage` | ENUM | NOT NULL | References instance graduation stage |
| `decision_type` | ENUM | NOT NULL | References Decision classification |
| `work_spec_status` | ENUM | NOT NULL DEFAULT `draft` | `draft`, `confirmed`, `active`, `completed`, `promoted`, `rejected` — SM-4 lifecycle |
| `confirmed_by` | UUID | FK → `dlp.entities` | Confirmation actor |
| `confirmed_at` | TIMESTAMPTZ | | Confirmation timestamp |
| `confirmation_decision_id` | UUID | FK → `dlp.decisions` | Decision authorizing confirmation |
| `promotion_decision_id` | UUID | FK → `dlp.decisions` | Decision authorizing promotion |
| `rejection_decision_id` | UUID | FK → `dlp.decisions` | Decision authorizing rejection |
| `rejection_rationale` | TEXT | | Rejection reason |
| `skip_confirmation_authorized` | BOOLEAN | NOT NULL DEFAULT false | Draft → Active skip (only when graduation_stage = C AND consequence_level = reversible) |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

The five fields — `cognitive_type`, `decision_type`, `consequence_level`, `operating_posture`, `graduation_stage` — constitute the five-dimensional routing context from §A1. Every work specification carries all five dimensions.

SM-4 governs the lifecycle. The Draft → Active skip is only permitted when `graduation_stage = C` AND `consequence_level = reversible`. All other transitions to Active require the Confirmed intermediate state.

### Table 26.5.2: Validated Projection Cache — `dlp.validated_projections`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `vp_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `subject_ref` | UUID | NOT NULL | Subject entity |
| `subject_type` | ENUM | NOT NULL | Uses `dlp.governed_primitive_type` |
| `vp_type` | ENUM | NOT NULL | `plan_baseline`, `condition_assessment`, `prediction` |
| `cache_state` | ENUM | NOT NULL DEFAULT `current` | `current`, `stale`, `recomputing`, `invalidated` — SM-9 |
| `materialized_at` | TIMESTAMPTZ | NOT NULL | When computed |
| `valid_until` | TIMESTAMPTZ | | Validity expiration |
| `version` | INTEGER | NOT NULL DEFAULT 1 | VP version |
| `confidence` | NUMERIC(3,2) | CHECK (0.0–1.0) | Projection confidence |
| `derivation_method` | TEXT | | How computed |
| `source_snapshot` | JSONB | | Input data snapshot |
| `stale_reason` | TEXT | | Why stale |
| `stale_at` | TIMESTAMPTZ | | When marked stale |
| `invalidation_reason` | TEXT | | Why invalidated |
| `triggering_event_ref` | UUID | | Event that triggered invalidation |
| `invalidated_at` | TIMESTAMPTZ | | When invalidated |
| `recomputation_requested_at` | TIMESTAMPTZ | | When recomputation started |
| `recomputed_from_vp_id` | UUID | FK self | Prior VP version |
| `truth_type` | ENUM | NOT NULL DEFAULT `derived` | All VPs are Derived per §17 |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |

All validated projections carry `truth_type = derived`. VPs are never Authoritative or Declared — they are machine-generated projections that require human review before promotion.

### Table 26.5.3: Graduation Records — `dlp.graduation_records`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `graduation_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `scope_descriptor` | TEXT | NOT NULL | AICAR×DecisionType×Namespace scope |
| `current_stage` | ENUM | NOT NULL | `A`, `A_plus`, `B`, `C` — SM-2 |
| `transition_decision_id` | UUID | FK → `dlp.decisions` | Decision authorizing transition |
| `evidence_summary` | JSONB | | Evidence supporting the transition |
| `metrics_at_transition` | JSONB | | Performance metrics at transition time |
| `rollback_condition_expressions` | JSONB | | Automated regression conditions |
| `transitioned_at` | TIMESTAMPTZ | NOT NULL | When transition occurred |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

### Table 26.5.4: Profile Graduation Evaluations — `dlp.profile_graduation_evaluations`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `evaluation_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `graduation_path` | ENUM | NOT NULL | `pas_to_bas`, `bas_to_eas` |
| `evaluation_status` | ENUM | NOT NULL DEFAULT `monitoring` | `monitoring`, `triggered`, `evaluating`, `decided_go`, `decided_nogo`, `executing`, `completed`, `declined` — SM-3 |
| `source_instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Instance graduating from |
| `target_instance_id` | UUID | FK → `dlp.instances` | Instance graduating to (created on Executing) |
| `decision_id` | UUID | FK → `dlp.decisions` | Graduation decision |
| `evidence_carry_forward` | JSONB | | Evidence migrated to new instance |
| `cooldown_until` | TIMESTAMPTZ | | For Declined → Monitoring re-entry |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

This table supports both PAS→BAS and BAS→EAS graduation paths.

---

## §26.6 EDIM Governance Registry Schema

The EDIM domain implements the governance registry infrastructure (§11–§13): pattern signatures, activation rules, governance sources, change propagation, and KPI measurement.

### Table 26.6.1: EDIM Core Tables

| Table | Purpose | Key Fields | Update Notes |
|---|---|---|---|
| `edim.nodes` | Governance domain node registry | `node_id`, `node_type`, `primitive_alignment` (TEXT — maps to one of 19 primitives) | Add `primitive_alignment` for 19-primitive model coverage |
| `edim.edges` | Cross-node relationship registry | `edge_id`, `edge_type`, `source_node_id`, `target_node_id` | Map `edge_type` to 13 cross-primitive relationships |
| `edim.authority_assignments` | RACIVG role assignments | `entity_id`, `node_id`, `responsibility` ENUM → `dlp.racivg_role`, `profile_type` (context for RACIVG enforcement) | Convert `responsibility` TEXT to ENUM; add profile context |
| `edim.live_state` | Runtime governance state | `node_id`, `execution_status`, `governance_activation_stage` ENUM | Add `governance_activation_stage` aligned with §12 six-stage pipeline |
| `edim.subscriptions` | Signal routing subscriptions | `subscription_id`, `subscriber_id`, `event_types` | Aligned |
| `edim.validation_guards` | Constraint enforcement | `guard_id`, `v_type` ENUM (seven verification types), `constraint_id` FK → `dlp.constraints` | Add FK linking guards to Constraint primitive |
| `edim.guard_executions` | Validation execution records | `execution_id`, `guard_id`, `result`, `evidence_id` | Aligned with §5.5 validation reports |

### Table 26.6.2: EDIM Governance Source Tables

| Table | Purpose | Key Fields | Update Notes |
|---|---|---|---|
| `edim.governance_sources` | Framework registry | `source_id`, `layer` ENUM (`layer1`, `layer2`, `layer3`), `authority_type` ENUM, `novelty_status` ENUM | Map layers to seven governance domains |
| `edim.pattern_signatures` | Pattern library | `signature_id`, `pattern_type` ENUM, `primitive_alignment`, `conservation_law_reference` | Verify 19-primitive coverage; add conservation law reference |
| `edim.activation_rules` | Governance activation triggers | `rule_id`, `activation_type` ENUM (`full`, `partial`, `reference_only`) | Verify against §12 six-stage pipeline |
| `edim.organization_profiles` | Organization configuration | `profile_id`, `organization_type` ENUM, `risk_appetite` ENUM | No CAS references |

### Table 26.6.3: EDIM Change Propagation Tables

| Table | Purpose | Key Fields | Update Notes |
|---|---|---|---|
| `edim.change_events` | Change event log | `event_id`, `source_node_id`, `primitive_context` ENUM, `conservation_law_ref` | Add primitive context and conservation law reference |
| `edim.blast_radius_cache` | Impact analysis cache | `cache_id`, `source_node_id`, `affected_nodes` | Verify against SM-9 VP cache state model |
| `edim.change_notifications` | Notification dispatch | `notification_id`, `required_role` ENUM → `dlp.racivg_role`, `action_required` ENUM | Align `required_role` with RACIVG; convert `action_required` to ENUM |

### Table 26.6.4: EDIM Control Tables

| Table | Purpose | Key Updates |
|---|---|---|
| `edim.control_families` | Control family registry | Profile-scoped via `applies_to_profile` using profile_type ENUM (eas, bas, pas) |
| `edim.controls` | Individual controls | No CAS references |
| `edim.compensating_control_types` | Compensating control definitions | Aligned |
| `edim.segregation_gap_rules` | Separation of duties rules | Aligned |

---

## §26.7 KI Knowledge Schema

The KI domain implements the knowledge integration layer (§18–§20): source management, concept governance, claims maturation, semantic mappings, and ontology alignment.

### Table 26.7.1: KI Core Tables

| Table | Purpose | Key Fields | Update Notes |
|---|---|---|---|
| `ki.source_types` | Source type registry | `source_type_id`, `name`, `description` | Aligned — seed data stable |
| `ki.sources` | Knowledge source instances | `source_id`, `source_type_id` FK, `metadata` | Aligned |
| `ki.concept_domains` | Domain classification | `domain_id`, `substrate_code` ENUM → `dlp.profile_type` (core, eas, bas, pas), `namespace_id` FK → `dlp.namespaces` | Add namespace FK |
| `ki.concepts` | Concept definitions | `concept_id`, `semantic_type` ENUM, `approval_status` ENUM | Map `approval_status` to Namespace graduation (proposed → reviewed → confirmed → canonical) |
| `ki.concept_relationships` | Concept-to-concept links | `relationship_id`, `source_concept_id`, `target_concept_id`, `relationship_type` | Aligned |
| `ki.chunks` | Content chunks | `chunk_id`, `source_id` FK, `content_type` ENUM, `content` | Convert `content_type` to ENUM |
| `ki.physical_containers` | Physical content containers | `container_id`, `substrate_code` ENUM → `dlp.profile_type` | Profile-type scoped |
| `ki.semantic_mappings` | Ontology alignment mappings | `mapping_id`, `mapping_type` ENUM, `bridge_type` ENUM | Add `bridge_type` ENUM: `equivalent`, `subsumes`, `specializes`, `overlaps`, `complements` per §19 |
| `ki.confidence_bands` | Confidence level definitions | `band_id`, `lower_bound`, `upper_bound` | Aligned |

### Table 26.7.2: KI Claims Tables

| Table | Purpose | Key Fields | Update Notes |
|---|---|---|---|
| `ki.claim_types` | Claim type registry | `claim_type_id`, `name` | Aligned with §6.5 |
| `ki.epistemic_states` | State reference table | `state_code`, `description` | States aligned with SM-10 Dimension 3 |
| `ki.claims` | Claim instances | `claim_id`, `graduation_status` ENUM, `namespace_scope` ENUM (core, domain, tenant, sandbox) | Convert `graduation_status` to ENUM; add `namespace_scope` for graduation target |
| `ki.claim_evidence` | Evidence-to-claim bindings | `claim_id` FK, `evidence_id` FK, `support_type` ENUM, `support_strength` ENUM, `binding_method` ENUM | Convert all VARCHAR fields to ENUM |
| `ki.claim_dependencies` | Inter-claim dependencies | `claim_id` FK, `depends_on_claim_id` FK | Aligned |
| `ki.claim_transitions` | Claim state transitions | `transition_id`, `claim_id` FK, `transition_type` ENUM | Convert `transition_type` to ENUM |
| `ki.graduation_criteria` | Graduation requirements | `criteria_id`, `claim_type_id` FK, `weights` | Verify weights against graduation model |
| `ki.graduation_reviews` | Review records | `review_id`, `claim_id` FK, `graduation_stage` ENUM | Add `graduation_stage` context (A→A+→B→C) |

### Table 26.7.3: KI Ontology Alignment Tables

| Table | Purpose | Key Fields | Update Notes |
|---|---|---|---|
| `ki.ontology_sources` | External ontology registry | `source_id`, `domain_coverage` | Add 19-primitive alignment |
| `ki.mapping_review_queue` | Pending alignment reviews | `review_id`, `proposed_primitive` ENUM → `dlp.governed_primitive_type`, `bridge_type` ENUM | Verify against 19-primitive vocabulary; add bridge type |
| `ki.alignment_feedback` | Review feedback records | `feedback_id`, `review_id` FK, `error_type` | Aligned |
| `ki.mapping_graduation_weights` | Graduation scoring weights | `weight_id`, `mapping_type`, `weight_value` | Verify against graduation model |

### Table 26.7.4: KI IQ Consolidation

The logical model consolidates investigative queries to a single `dlp.investigative_queries` table (Table 26.2.23). KI-originated IQs use `origin_type = capture` and reference KI-specific source objects through the polymorphic `source_type`/`source_id` fields.

---

## §26.8 Content Package Schema

Content packages provide profile-specific governance templates and configuration. Three packages exist — one per substrate profile.

### §26.8.1 EAS Content Package

The Enterprise Accountability Substrate content package provides templates and configuration for enterprise governance.

### Table 26.8.1: EAS Tables

| Table | Purpose | Key Updates |
|---|---|---|
| `eas.organization_instances` | EAS deployment instances | Add `graduation_decision_id` FK per SM-3. Verify `governance_phase` (G0–G3) against graduation model. |
| `eas.governance_intent_templates` | Intent configuration templates | Aligned |
| `eas.governance_intent_sub_intents` | Intent decomposition | Aligned |
| `eas.governance_evidence_types` | Evidence type registry | Add `truth_type_constraint` to specify valid truth types per evidence type |
| `eas.governance_account_templates` | Account configuration | Aligned |
| `eas.authority_templates` | Authority configuration | Add RACIVG role mapping. EAS uses full six-role RACIVG model. |
| `eas.authority_functional_types` | Authority function classification | Map to four actor types |
| `eas.authority_delegation_rules` | Delegation rule configuration | Add AI-Cannot-Be-Principal constraint on A (Accountable) role |
| `eas.role_templates` | Role configuration | Map to role envelope specification. Add `execution_modality` ENUM. |
| `eas.governance_commitment_templates` | Commitment templates | Aligned |
| `eas.governance_phase_definitions` | Phase gate configuration | Verify against six-stage activation pipeline |
| `eas.domain_extensions` | Governance domain extensions | Aligned — supports seven governance domains |
| `eas.extension_evidence_types` | Domain-specific evidence types | Aligned |

### §26.8.2 BAS Content Package

### Table 26.8.2: BAS Tables

| Table | Purpose | Key Updates |
|---|---|---|
| `bas.business_instances` | BAS deployment instances | Add `graduation_stage` per SM-2/SM-3. Add `constraint_cascade_parent_id` for §23 tighten-only rule. |
| `bas.intent_templates` | Intent templates (18-domain) | Aligned |
| `bas.evidence_types` | Evidence type registry | Add `truth_type_constraint` |
| `bas.account_templates` | Account configuration | Aligned |
| `bas.commitment_templates` | Commitment templates | Aligned |
| `bas.authority_templates` | Authority configuration | BAS uses RACI (four-role subset; V and G inactive). Add `racivg_roles_active` validation rule. |
| `bas.phase_definitions` | Phase configuration | Verify against activation pipeline |
| `bas.business_intents` | Runtime intent instances | Aligned |

### §26.8.3 PAS Content Package

### Table 26.8.3: PAS Tables

| Table | Purpose | Key Updates |
|---|---|---|
| `pas.project_instances` | PAS deployment instances | Add `parent_instance_id` for §23 portfolio hierarchy. Add `constraint_cascade_parent_id`. |
| `pas.project_type_templates` | Project type configuration | Aligned |
| `pas.evidence_types` | Evidence type registry | Add `truth_type_constraint` |
| `pas.authority_templates` | Authority configuration | PAS uses simplified roles (Owner, Collaborator). Add `actor_type_constraint`. |
| `pas.graduation_triggers` | Graduation trigger definitions | Aligned — `destination_substrate` uses `dlp.profile_type` (BAS, EAS only) |
| `pas.graduation_pathways` | Graduation pathway definitions | Aligned |
| `pas.graduation_evaluations` | Graduation evaluation instances | Superseded by `dlp.profile_graduation_evaluations` (Table 26.5.4) for SM-3 compliance. Retained for backward compatibility; new evaluations use DLP-level table. |

### §26.8.4 CAS Reference Remediation

Four CAS references exist in the v8 schema. All are removed in the logical model:

### Table 26.8.4: CAS Removal Points

| Location | CAS Reference | Remediation |
|---|---|---|
| `ki.concept_domains.substrate_code` | Includes `cas` value | Remove `cas`, use `dlp.profile_type` ENUM (core, eas, bas, pas) |
| `ki.physical_containers.substrate_code` | Includes `cas` value | Remove `cas`, use `dlp.profile_type` ENUM |
| `edim.control_families.applies_to_cas` | BOOLEAN column | Remove column. Replace with `applies_to_profile` referencing `dlp.profile_type` |
| `dlp.instances.substrate_tier` | ENUM values | Rename to `profile_type`. Values: `eas`, `bas`, `pas` only. |

---

## §26.9 Controlled Vocabulary Specification

The logical model introduces thirty-two controlled vocabulary types (PostgreSQL ENUMs). All fields that accept a constrained set of values use either ENUMs for fixed vocabularies or reference tables for extensible vocabularies.

### §26.9.1 DLP Domain ENUMs

### Table 26.9.1: DLP Controlled Vocabularies

| # | ENUM Name | Values | Source | Used By |
|---|---|---|---|---|
| 1 | `dlp.governed_primitive_type` | `intent`, `commitment`, `capacity`, `work`, `evidence`, `decision`, `authority`, `account`, `constraint`, `identifier`, `entity`, `context`, `namespace`, `orientation`, `learning`, `activation`, `interpretation`, `environment_interface`, `cycle` | §4.5 | All polymorphic type fields across all tables |
| 2 | `dlp.enforcement_mode` | `blocking`, `warning`, `logging`, `advisory` | §5.4 | `dlp.constraints.enforcement_mode` |
| 3 | `dlp.constraint_type` | `policy`, `compliance`, `physical`, `organizational` | §4.1 | `dlp.constraints.constraint_type` |
| 4 | `dlp.namespace_scope` | `core`, `domain`, `tenant`, `sandbox` | §4.6.1 | `dlp.namespaces.scope` |
| 5 | `dlp.namespace_status` | `active`, `proposed`, `under_review`, `graduated`, `expired`, `retired` | §7.4 | `dlp.namespaces.status` |
| 6 | `dlp.actor_type` | `human`, `ai_automation`, `ai_agentic`, `ai_assistive` | §22 | `dlp.actors.actor_type` |
| 7 | `dlp.racivg_role` | `R`, `A`, `C`, `I`, `V`, `G` | §22 | `dlp.role_envelopes.racivg_role`, EDIM authority assignments |
| 8 | `dlp.governance_posture` | `standard`, `enhanced`, `critical` | §22 | `dlp.actors.governance_posture` |
| 9 | `dlp.execution_modality` | `human_only`, `ai_assisted`, `ai_executed_with_review` | §22 | `dlp.role_envelopes.execution_modality` |
| 10 | `dlp.consequence_level` | `reversible`, `consequential`, `structural` | §A1 | `dlp.work_specifications.consequence_level` |
| 11 | `dlp.operating_posture` | `normal`, `degraded`, `emergency` | §A1 | `dlp.work_specifications.operating_posture` |
| 12 | `dlp.graduation_stage` | `A`, `A_plus`, `B`, `C` | §8, §12 | `dlp.instances.graduation_stage`, SM-2 |
| 13 | `dlp.verification_state` | `unverified`, `verification_pending`, `verified`, `verification_failed`, `cannot_verify` | §6.4 | `dlp.evidences.verification_state` |
| 14 | `dlp.claims_graduation_stage` | `asserted`, `investigating`, `corroborated`, `verified`, `graduated` | §6.5 | Claims tracking |
| 15 | `dlp.record_lifecycle_state` | `active`, `superseded` | §6 | All primitive tables |
| 16 | `dlp.verification_type` | `exist`, `complete`, `current`, `approved`, `consistent`, `compliant`, `ratified` | §6.7 | Verification operations |
| 17 | `dlp.signal_status` | `created`, `routed`, `processing`, `escalated`, `monitoring`, `resolved`, `dismissed` | §15, SM-7 | `dlp.signal_flags.signal_status` |
| 18 | `dlp.iq_intent_type` | `explore`, `validate`, `disambiguate`, `fill_gap`, `challenge`, `connect`, `branch_inventory`, `confidence_probe` | §16 | `dlp.investigative_queries.intent_type` |
| 19 | `dlp.iq_origin_type` | `claim`, `signal`, `capture`, `workflow` | §16 | `dlp.investigative_queries.origin_type` |
| 20 | `dlp.iq_status` | `open`, `assigned`, `investigating`, `resolved`, `promoted`, `abandoned`, `superseded` | §16, SM-8 | `dlp.investigative_queries.status` |
| 21 | `dlp.reservation_nature` | `concern`, `uncertainty`, `disagreement`, `precedent_question`, `scope_question`, `timing_question`, `resource_question` | §15 | `dlp.signal_flags.reservation_nature` |
| 22 | `dlp.governance_activation_stage` | `discover`, `analyze`, `implement`, `evidence`, `verify`, `sustain` | §12 | `dlp.activations.governance_stage` |
| 23 | `dlp.delegation_status` | `proposed`, `active`, `modified`, `suspended`, `revoked`, `expired` | SM-5 | `dlp.delegations.delegation_status` |
| 24 | `dlp.work_spec_status` | `draft`, `confirmed`, `active`, `completed`, `promoted`, `rejected` | SM-4 | `dlp.work_specifications.work_spec_status` |
| 25 | `dlp.profile_type` | `eas`, `bas`, `pas` | §21 | `dlp.instances.profile_type` |
| 26 | `dlp.kpi_class` | `invariant_signal`, `control_kpi`, `intent_progress_indicator` | FD-7.1 | `dlp.kpi_definitions.kpi_class` |
| 27 | `dlp.vp_cache_state` | `current`, `stale`, `recomputing`, `invalidated` | §17, SM-9 | `dlp.validated_projections.cache_state` |
| 28 | `dlp.bridge_type` | `equivalent`, `subsumes`, `specializes`, `overlaps`, `complements` | §19 | `ki.semantic_mappings.bridge_type` |
| 29 | `dlp.cognitive_work_type` | `A_citation`, `B_reasoning`, `C_context`, `D_communication` | §A1 | `dlp.work_specifications.cognitive_type` |
| 30 | `dlp.composition_pattern` | `sequential`, `parallel`, `iterative`, `conditional`, `delegated`, `federated` | §A1 | `dlp.work_specifications.composition_strategy` |
| 31 | `dlp.authority_type` | `approval`, `decision`, `oversight`, `veto`, `advisory`, `delegated` | §7, §22 | `dlp.authorities.authority_type` |
| 32 | `dlp.governance_domain` | `authority_decision_rights`, `commitment_obligation`, `evidence_record_truth`, `work_execution_lifecycle`, `access_identity_boundary`, `exception_escalation_override`, `learning_change_evolution` | §13 | `edim.kpi_definitions.governance_domain` |

### §26.9.2 TEXT-to-ENUM Conversions

Thirty-nine fields identified in the v8 schema as TEXT or VARCHAR with constrained value sets are converted to ENUM types in the logical model:

### Table 26.9.2: TEXT-to-ENUM Conversion Summary

| Domain | Fields Converted | Pattern |
|---|---|---|
| DLP | 12 | Polymorphic type fields (`target_type`, `source_type`, `entity_type`, etc.) → `dlp.governed_primitive_type`; assessment fields (`stakes_assessment`, `novelty_assessment`) → dedicated ENUMs |
| EDIM | 9 | Role fields (`responsibility`) → `dlp.racivg_role`; action fields → `edim.required_action`; status fields → domain-specific ENUMs |
| KI | 16 | Type fields (`semantic_type`, `content_type`, `attribution_type`) → KI-specific ENUMs; status fields (`approval_status`, `graduation_status`, `review_status`) → lifecycle ENUMs |
| Content Packages | 2 | Destination fields (`destination_substrate`) → `dlp.profile_type` |

### §26.9.3 JSONB-to-Structured Conversions

Eight JSONB fields in the v8 schema carry internal structure that benefits from companion ENUM columns:

### Table 26.9.3: JSONB Structuring

| Field Location | Current Type | Added Structure | Rationale |
|---|---|---|---|
| `dlp.authorities.source` | JSONB | `parent_authority_id` FK + `is_root_authority` BOOLEAN + `source_type` ENUM | B5 structural enforcement requires explicit FK, not JSONB |
| `dlp.authorities.scope` | JSONB | `scope_type` ENUM companion column | Enables scope-type filtering without JSONB parsing |
| `dlp.works.assignment` | JSONB | `assigned_to_id` FK + `assignee_type` ENUM + `assignment_method` ENUM | Actor Layer requires typed assignment |
| `dlp.decisions.rationale` | JSONB | `rationale_type` ENUM companion column | Enables decision reasoning classification |
| `dlp.decisions.options` | JSONB | Retained — structured list with typed fields | Internal structure sufficient |
| `dlp.commitments.acceptance_event` | JSONB | `acceptance_type` ENUM companion column | Enables acceptance classification |
| `dlp.activations.blockers` | JSONB | Enforce internal structure (blocker_type ENUM per element) | Structured JSONB validation |
| `dlp.environment_interfaces.external_party` | JSONB | `party_type` ENUM companion column | Enables party-type filtering |

The JSONB fields are retained for their flexible internal structure. The companion ENUM columns provide indexed, queryable classification without requiring JSONB extraction in WHERE clauses.

---

## §26.10 State Machine Alignment

Ten state machines govern lifecycle transitions across the substrate. Every state machine maps to a status ENUM field on the corresponding table. ENUM values match state machine states exactly — no semantic translation layer exists between the state machine definition and the schema.

### Table 26.10.1: State Machine to Schema Mapping

| SM | Name | States | Schema Table | Status Field | ENUM Type |
|---|---|---|---|---|---|
| SM-1 | Actor Lifecycle | 5 | `dlp.actors` | `actor_status` | `registered`, `active`, `suspended`, `deactivated`, `archived` |
| SM-2 | Graduation Model | 4 | `dlp.instances` + `dlp.graduation_records` | `graduation_stage` | `A`, `A_plus`, `B`, `C` |
| SM-3 | Profile Graduation | 8 | `dlp.profile_graduation_evaluations` | `evaluation_status` | `monitoring`, `triggered`, `evaluating`, `decided_go`, `decided_nogo`, `executing`, `completed`, `declined` |
| SM-4 | Work Specification Lifecycle | 6 | `dlp.work_specifications` | `work_spec_status` | `draft`, `confirmed`, `active`, `completed`, `promoted`, `rejected` |
| SM-5 | Delegation Lifecycle | 6 | `dlp.delegations` | `delegation_status` | `proposed`, `active`, `modified`, `suspended`, `revoked`, `expired` |
| SM-6 | Agent Behavioral Cycle | 8 | `dlp.actors` + `dlp.agent_behavioral_events` | `current_behavioral_state` | `listening`, `context_loading`, `navigating`, `proposing`, `monitoring`, `alerting`, `executing`, `completing` |
| SM-7 | Signal Lifecycle | 7 | `dlp.signal_flags` | `signal_status` | `created`, `routed`, `processing`, `escalated`, `monitoring`, `resolved`, `dismissed` |
| SM-8 | IQ Lifecycle | 7 | `dlp.investigative_queries` | `status` | `open`, `assigned`, `investigating`, `resolved`, `promoted`, `abandoned`, `superseded` |
| SM-9 | VP Cache State | 4 | `dlp.validated_projections` | `cache_state` | `current`, `stale`, `recomputing`, `invalidated` |
| SM-10 | Truth Type Transitions | 12 (3 dims) | `dlp.evidences` (+ all truth_type fields) | `truth_type`, `verification_state`, `promotion_workflow_state`, `claims_graduation_stage` | Three orthogonal ENUM fields |

### §26.10.1 Transition Decision Requirements

State machine transitions carry different decision requirements:

### Table 26.10.2: Transition Decision Requirements

| SM | Transition Pattern | Decision Required | Rationale |
|---|---|---|---|
| SM-1 | All transitions | Yes — FK to `dlp.decisions` | Actor lifecycle changes are governance decisions |
| SM-2 | All transitions | Yes | Graduation changes AI autonomy level |
| SM-3 | All except Monitoring→Triggered (automatic) | Yes for evaluated transitions | Automatic trigger on threshold; human decision for go/nogo |
| SM-4 | All except Draft→Confirmed (skip path) | Yes | Confirmation skip requires Decision + conditions |
| SM-5 | All except Expired (automatic) | Yes | Expired fires on timestamp; all others are governance decisions |
| SM-6 | None — runtime transitions | No | SM-6 is a cyclic behavioral machine; transitions are audited via `dlp.agent_behavioral_events` |
| SM-7 | Created→Routed (may be automatic) | Routing may be automatic | Signal routing follows B8 authority chain |
| SM-8 | All except Resolved→Open (reversion) | Reversion requires Evidence reference | New evidence triggers reversion |
| SM-9 | All — cache management | No | VP cache transitions are infrastructure operations |
| SM-10 | Truth type transitions | Yes — promotion requires human action | Derived→Declared requires human review per §6 |

### §26.10.2 SM-10 Orthogonal Dimensions

SM-10 is unique among the ten state machines: it operates across three independent dimensions tracked by three separate ENUM fields on a single record. A governed object simultaneously carries state in all three dimensions:

| Dimension | ENUM Field | States | Independence |
|---|---|---|---|
| Evidence Verification | `verification_state` | 5 states | Transitions independently of truth_type and claims_graduation |
| Derived Promotion | `truth_type` + `promotion_workflow_state` | truth_type: 3 values; workflow: 4 states | Promotion changes epistemic origin, not verification status |
| Claims Graduation | `claims_graduation_stage` | 5 stages | Reads verification state (Investigating→Corroborated requires Verified evidence) but does not transition it |

**Cross-dimension interaction.** Claims graduation reads verification state: a claim cannot advance from Investigating to Corroborated unless supporting Evidence records are in `verification_state = verified`. This is a read dependency, not a transition coupling — the graduation machine reads the verification machine's state but does not cause verification transitions.

**Monotonic promotion.** The `truth_type` field follows monotonic promotion: `derived → declared → authoritative`. Reverse transitions are prohibited. A DDL CHECK constraint enforces that new `truth_type` values are ≥ old values in the ordering. Supersession (creating a new record) is the mechanism for representing a change in epistemic assessment — the original record retains its truth type.

---

## §26.11 SHACL Shape Inventory

Seventy-nine SHACL shapes validate substrate instances across five functional domains. The Phase 1 foundation provides seventeen shapes (ten invariant shapes + seven verification type shapes). Phase 7 adds sixty-two shapes across governance, actor, portfolio, orchestration, and KPI domains.

### Table 26.11.1: Shape Inventory by Domain

| Domain | Shapes | Blocking | Warning | Logging | Advisory | Source |
|---|---|---|---|---|---|---|
| **Phase 1: Invariant Shapes** | 10 | 8 | 2 | 0 | 0 | `invariant-shapes.ttl` |
| **Phase 1: Verification Types** | 7 | 4 | 3 | 0 | 0 | `verification-type-shapes.ttl` |
| **Governance (§11–§13)** | 17 | 10 | 7 | 0 | 0 | Phase 7 extension |
| **Actor (§22)** | 15 | 11 | 3 | 1 | 0 | Phase 7 extension |
| **Portfolio (§23)** | 6 | 4 | 0 | 2 | 0 | Phase 7 extension |
| **Orchestration (§A1/§A2)** | 16 | 15 | 1 | 0 | 0 | Phase 7 extension |
| **KPI (FD-7.1)** | 8 | 6 | 2 | 0 | 0 | Phase 7 extension |
| **Total** | **79** | **58** | **18** | **3** | **0** | |

### §26.11.1 Phase 1 Foundation Shapes

**Invariant Shapes** (`invariant-shapes.ttl`): B1-WorkRequiresCommitment, B2-CommitmentRequiresCapacity, B3-EvidenceRequiresTruthType, B4-DecisionRequiresAccount, B5-AuthorityTraceable (Core), B5-AuthorityChainReachesRoot (SPARQL), B6-ConstraintBindsPrimitives (Core), B6-ConstraintUniversality (SPARQL), B8-SignalsRouteToAuthority (Core+SPARQL), B9-IQResolutionCreatesDecision (SPARQL). B7 is a schema-level conformance guarantee, not a per-instance shape.

**Verification Type Shapes** (`verification-type-shapes.ttl`): ExistVerification, CompleteVerification, CurrentVerification (SPARQL), ApprovedVerification, ConsistentVerification (SPARQL), CompliantVerification (SPARQL+META), RatifiedVerification (META).

### §26.11.2 Phase 7 Governance Shapes (§11–§13)

### Table 26.11.2: Governance Shapes

| # | Shape URI | Target | Enforcement | Description |
|---|---|---|---|---|
| 1 | `dlps:GovernanceDomainCoverage` | `dlp:Instance` | Blocking | Every instance maps to ≥1 of seven governance domains |
| 2 | `dlps:GovernanceDomain-AuthorityDecision` | `dlp:Authority`, `dlp:Decision` | Warning | Domain 1: Authority & Decision Rights |
| 3 | `dlps:GovernanceDomain-CommitmentObligation` | `dlp:Commitment`, `dlp:Capacity` | Warning | Domain 2: Commitment & Obligation |
| 4 | `dlps:GovernanceDomain-EvidenceRecordTruth` | `dlp:Evidence`, `dlp:Account` | Warning | Domain 3: Evidence, Record & Truth |
| 5 | `dlps:GovernanceDomain-WorkExecution` | `dlp:Work`, `dlp:Cycle` | Warning | Domain 4: Work & Execution Lifecycle |
| 6 | `dlps:GovernanceDomain-AccessBoundary` | `dlp:Entity`, `dlp:EnvironmentInterface` | Warning | Domain 5: Access, Identity & Boundary |
| 7 | `dlps:GovernanceDomain-ExceptionEscalation` | `dlp:Constraint` | Warning | Domain 6: Exception, Escalation & Override |
| 8 | `dlps:GovernanceDomain-LearningEvolution` | `dlp:Learning`, `dlp:Orientation` | Warning | Domain 7: Learning, Change & Evolution |
| 9 | `dlps:ActivationStageSequence` | `dlp:Activation` | Blocking | `governance_stage` constrained to six-stage pipeline |
| 10 | `dlps:ActivationStageTransition` | `dlp:Activation` | Blocking | Forward transitions require Decision reference |
| 11 | `dlps:ActivationEvidenceGate` | `dlp:Activation` | Blocking | Implement→Evidence requires linked Evidence |
| 12 | `dlps:ActivationVerifyGate` | `dlp:Activation` | Blocking | Evidence→Verify requires linked assessment |
| 13 | `dlps:Domain1-DelegationChainIntegrity` | `dlp:Authority` | Blocking | Scope attenuation in delegation chains |
| 14 | `dlps:Domain1-OverrideSemantics` | `dlp:Decision` | Blocking | Override decisions require full provenance |
| 15 | `dlps:Domain3-TruthTypePreservation` | `dlp:Evidence` | Blocking | Monotonic promotion only (no demotion) |
| 16 | `dlps:Domain5-SeparationOfDuties` | `dlp:Entity` | Warning | A-role and R-role separation where profile enforces |
| 17 | `dlps:Domain6-ExceptionRequiresEvidence` | `dlp:Decision` | Blocking | Exception overrides require Declared/Authoritative evidence |

### §26.11.3 Phase 7 Actor Shapes (§22)

### Table 26.11.3: Actor Shapes

| # | Shape URI | Target | Enforcement | Description |
|---|---|---|---|---|
| 18 | `dlps:ActorTypeValid` | `dlp:Actor` | Blocking | Four actor types enforced |
| 19 | `dlps:ActorRegistration` | `dlp:Actor` | Blocking | Mandatory registration fields |
| 20 | `dlps:ActorLifecycleState` | `dlp:Actor` | Blocking | SM-1 states enforced |
| 21 | `dlps:AIActorRequiresParent` | `dlp:Actor` | Blocking | AI actors must have organizational owner |
| 22 | `dlps:AICannotBePrincipal` | `dlp:Actor` | Blocking | AI actors cannot hold A (Accountable) role |
| 23 | `dlps:AICannotBeRootAuthority` | `dlp:Authority` | Blocking | AI actors cannot hold root authority |
| 24 | `dlps:AIDecisionRequiresHumanAuthority` | `dlp:Decision` | Blocking | AI decisions must trace to human-held authority |
| 25 | `dlps:RACIVGRoleValid` | `dlp:Actor` | Blocking | RACIVG role vocabulary enforced |
| 26 | `dlps:EASProfileRACIVG` | `dlp:Actor` | Advisory | EAS permits all six roles |
| 27 | `dlps:BASProfileRACI` | `dlp:Actor` | Warning | BAS restricts to R, A, C, I |
| 28 | `dlps:PASProfileSimplified` | `dlp:Actor` | Warning | PAS uses Owner + Collaborator |
| 29 | `dlps:AccountableRoleExactlyOne` | Governance scope | Blocking | Exactly one A-role holder per governed primitive |
| 30 | `dlps:RoleEnvelopeComplete` | `dlp:RoleEnvelope` | Blocking | All envelope fields required |
| 31 | `dlps:RoleEnvelopeModalityValid` | `dlp:RoleEnvelope` | Blocking | Execution modality vocabulary enforced |
| 32 | `dlps:RoleEnvelopeContextModifiers` | `dlp:RoleEnvelope` | Logging | Context modifier commutativity |

### §26.11.4 Phase 7 Portfolio Shapes (§23)

### Table 26.11.4: Portfolio Shapes

| # | Shape URI | Target | Enforcement | Description |
|---|---|---|---|---|
| 33 | `dlps:InstanceHierarchyValid` | `dlp:Instance` | Blocking | No self-parenting; parent profile ≥ child profile |
| 34 | `dlps:InstanceRelationshipType` | Instance relationship | Blocking | Five relationship types enforced |
| 35 | `dlps:GraduationCreatesNewInstance` | `dlp:Instance` | Blocking | Graduation creates new instance; source archives |
| 36 | `dlps:TightenOnlyCascade` | `dlp:Constraint` | Blocking | Child constraints ≥ parent constraints |
| 37 | `dlps:ConstraintRelaxationRequiresDecision` | `dlp:Constraint` | Blocking | Relaxation requires authorized Decision |
| 38 | `dlps:DepthGatingVisibility` | `dlp:Instance` | Logging | Hierarchy visibility rules |

### §26.11.5 Phase 7 Orchestration Shapes (§A1/§A2)

### Table 26.11.5: Orchestration Shapes

| # | Shape URI | Target | Enforcement | Description |
|---|---|---|---|---|
| 39 | `dlps:AICARTypeValid` | `dlp:WorkSpecification` | Blocking | AICAR cognitive type vocabulary |
| 40 | `dlps:AICARTypeARequiresVerification` | `dlp:WorkSpecification` | Warning | Citation fidelity verification for Type A |
| 41 | `dlps:AICARTypeBRequiresReview` | `dlp:WorkSpecification` | Warning | Reasoning integrity review for Type B |
| 42 | `dlps:AICARRedZoneBlocking` | `dlp:Interpretation` | Blocking | Red zone (high stakes + high novelty) requires review evidence |
| 43 | `dlps:DelegationLifecycleState` | `dlp:Delegation` | Blocking | SM-5 states enforced |
| 44 | `dlps:DelegationRequiresAuthority` | `dlp:Delegation` | Blocking | Every delegation authorized by Decision |
| 45 | `dlps:DelegationScopeAttenuation` | `dlp:Delegation` | Blocking | No scope amplification through delegation |
| 46 | `dlps:DelegationAutonomyLevel` | `dlp:Delegation` | Blocking | Autonomy level bounded by graduation stage |
| 47 | `dlps:WorkSpecificationComplete` | `dlp:WorkSpecification` | Blocking | Required fields enforced |
| 48 | `dlps:WorkSpecLifecycleState` | `dlp:WorkSpecification` | Blocking | SM-4 states enforced |
| 49 | `dlps:WorkSpecConfirmationRequired` | `dlp:WorkSpecification` | Blocking | Confirmation required except Stage C + reversible |
| 50 | `dlps:WorkSpecOutputTruthType` | `dlp:WorkSpecification` | Blocking | All outputs enter as Derived |
| 51 | `dlps:FiveDimensionalContextComplete` | `dlp:WorkSpecification` | Blocking | All five dimensions specified |
| 52 | `dlps:ConsequenceLevelValid` | `dlp:WorkSpecification` | Blocking | Three-value vocabulary |
| 53 | `dlps:OperatingPostureValid` | `dlp:WorkSpecification` | Blocking | Three-value vocabulary |
| 54 | `dlps:CompositionPatternValid` | `dlp:WorkSpecification` | Blocking | Six-value vocabulary |

### §26.11.6 Phase 7 KPI Shapes (FD-7.1)

### Table 26.11.6: KPI Shapes

| # | Shape URI | Target | Enforcement | Description |
|---|---|---|---|---|
| 55 | `dlps:KPIClassValid` | `dlp:KPIDefinition` | Blocking | Three KPI classes enforced |
| 56 | `dlps:InvariantSignalKPI` | `dlp:KPIDefinition` | Blocking | Invariant signal KPIs reference B1–B10 |
| 57 | `dlps:ControlKPIRequiresDomain` | `dlp:KPIDefinition` | Blocking | Control KPIs reference governance domain |
| 58 | `dlps:IntentProgressKPIRequiresIntent` | `dlp:KPIDefinition` | Blocking | Intent progress KPIs trace to active Intent |
| 59 | `dlps:KPIIntroductionIsDecision` | `dlp:KPILifecycleEvent` | Blocking | KPI introduction is a Decision |
| 60 | `dlps:KPIRetirementIsDecision` | `dlp:KPILifecycleEvent` | Blocking | KPI retirement is a Decision |
| 61 | `dlps:KPIRetirementPreservesHistory` | `dlp:KPIDefinition` | Warning | Retired KPIs retain measurement history |
| 62 | `dlps:InvariantSignalCompleteness` | `dlp:Instance` | Warning | Instances have invariant signal KPIs for applicable B1–B10 |

### §26.11.7 Two-Pass Mapping

### Table 26.11.7: Shape-to-Pass Assignment

| Pass | SHACL Tier | Shapes | Enforcement | Schedule |
|---|---|---|---|---|
| **Pass 1** | SHACL Core | 52 shapes | Always Blocking | Every write operation, O(1) per instance |
| **Pass 2** | SHACL-SPARQL | 24 shapes | Profile-configurable (Blocking/Warning/Logging) | Configurable: per-operation, batch, or periodic |
| **Schema** | Deployment-time | 3 shapes (B7 + deployment validators) | Blocking | Once at deployment; again on schema changes |

Pass 1 shapes are universal — no profile may downgrade them below Blocking. Pass 2 shapes may be adjusted between Blocking and Logging for organizational transitions, but may never be set to Advisory (which is reserved for non-invariant Constraints).

---

## §26.12 KPI Schema

The KPI schema implements the three-class KPI model from FD-7.1. KPIs are governance instruments — their introduction and retirement are Decision instances, not administrative operations.

### Table 26.12.1: KPI Definition Table — `edim.kpi_definitions`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `kpi_id` | UUID | PK | Unique identifier |
| `instance_id` | UUID | FK → `dlp.instances`, NOT NULL | Substrate instance scope |
| `kpi_class` | ENUM | NOT NULL | `invariant_signal`, `control_kpi`, `intent_progress_indicator` — three classes |
| `name` | TEXT | NOT NULL | KPI name |
| `description` | TEXT | | KPI description |
| `invariant_ref` | TEXT | | B1–B10 reference (required when kpi_class = invariant_signal) |
| `governance_domain` | ENUM | | Uses `dlp.governance_domain` — required when kpi_class = control_kpi |
| `tracked_intent_id` | UUID | FK → `dlp.intents` | Tracked intent (required when kpi_class = intent_progress_indicator) |
| `measurement_config` | JSONB | NOT NULL | Measurement method, frequency, thresholds |
| `status` | ENUM | NOT NULL | `active`, `retired` |
| `introduction_decision_id` | UUID | FK → `dlp.decisions`, NOT NULL | Decision that introduced this KPI |
| `retirement_decision_id` | UUID | FK → `dlp.decisions` | Decision that retired this KPI |
| `measurement_archive_ref` | UUID | FK → `dlp.evidences` | Archived measurement history (required on retirement) |
| `truth_type` | ENUM | NOT NULL | Universal field |
| `record_lifecycle_state` | ENUM | NOT NULL DEFAULT `active` | Universal field |
| `created_at` | TIMESTAMPTZ | NOT NULL | Universal field |
| `created_by` | UUID | FK → `dlp.entities`, NOT NULL | Universal field |

### Table 26.12.2: Three KPI Classes

| Class | Derivation | Scope | Example |
|---|---|---|---|
| **Invariant Signal** | Computed from B1–B10 violation rates | Protocol-level; exists in every deployment | B1 violation rate (unauthorized work percentage), B3 violation rate (unmarked evidence percentage) |
| **Control KPI** | Derived from governance domain controls | Domain-specific; varies by governance domain activation | Domain 1 delegation depth distribution, Domain 3 evidence staleness rate |
| **Intent Progress Indicator** | Tracks progress toward an active Intent | Intent-specific; lifecycle-bound to the tracked Intent | Revenue target achievement rate, compliance milestone completion |

Invariant Signal KPIs form the irreducible measurement set. Every active substrate instance carries invariant signal KPIs for all applicable B1–B10 invariants. These KPIs cannot be retired — they persist as long as the corresponding invariant is enforced.

Control KPIs and Intent Progress Indicators are governance-managed. Their introduction requires a Decision (decision_type = `approval`). Their retirement requires a Decision and preservation of measurement history as Authoritative Evidence.

### Table 26.12.3: KPI Lifecycle Events — `edim.kpi_lifecycle_events`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `event_id` | UUID | PK | Unique identifier |
| `kpi_id` | UUID | FK → `edim.kpi_definitions`, NOT NULL | Target KPI |
| `event_type` | ENUM | NOT NULL | `introduction`, `retirement`, `threshold_change`, `measurement_config_change` |
| `decision_id` | UUID | FK → `dlp.decisions`, NOT NULL | Authorizing Decision |
| `event_data` | JSONB | | Event-specific payload |
| `occurred_at` | TIMESTAMPTZ | NOT NULL | Event timestamp |
| `truth_type` | ENUM | NOT NULL | Universal field |

### Table 26.12.4: KPI Measurements — `edim.kpi_measurements`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `measurement_id` | UUID | PK | Unique identifier |
| `kpi_id` | UUID | FK → `edim.kpi_definitions`, NOT NULL | Measured KPI |
| `measured_at` | TIMESTAMPTZ | NOT NULL | Measurement timestamp |
| `value` | JSONB | NOT NULL | Measurement value (supports various types) |
| `threshold_status` | ENUM | | `within_bounds`, `warning`, `violation` |
| `measurement_method` | TEXT | | How measured |
| `truth_type` | ENUM | NOT NULL | Universal field — tracks whether measurement is Authoritative (system-computed) or Derived (estimated) |

---

## §26.13 SDK Constraints

The following constraints govern SDK implementations against the logical model. Each constraint is classified as MUST (violation is non-compliant), MUST NOT (violation is non-compliant), or DESIGN SPACE (implementation choice within specified bounds).

### §26.13.1 Schema Compliance

| ID | Constraint | Level |
|---|---|---|
| §26-C1 | SDK implementations MUST create all nineteen primitive tables across the four schema domains. | MUST |
| §26-C2 | SDK implementations MUST enforce the universal field set (Table 26.1.2) on every DLP domain table. | MUST |
| §26-C3 | SDK implementations MUST NOT add columns to primitive tables without an architecture-level justification. Profile-specific extensions use the Namespace extension framework (§7.4). | MUST NOT |
| §26-C4 | SDK implementations MUST create all thirteen cross-primitive relationship implementations (Table 26.3.1 direct FKs + Table 26.3.2 join tables). | MUST |
| §26-C5 | SDK implementations MUST enforce NOT NULL constraints on B1 (`works.commitment_id`), B4 (`decisions.account_id`), and B5 (`authorities.is_root_authority OR parent_authority_id IS NOT NULL`) FK columns. | MUST |

### §26.13.2 ENUM Usage

| ID | Constraint | Level |
|---|---|---|
| §26-C6 | SDK implementations MUST use the thirty-two ENUM types defined in Table 26.9.1 for all constrained-value fields. | MUST |
| §26-C7 | SDK implementations MUST NOT extend ENUM values without an architecture-level justification. Adding a value to `dlp.governed_primitive_type` requires adding a new primitive to the architecture. | MUST NOT |
| §26-C8 | SDK implementations MUST convert all thirty-nine TEXT-to-ENUM fields identified in §26.9.2. Legacy TEXT fields are not compliant. | MUST |
| §26-C9 | DESIGN SPACE: SDK implementations may implement controlled vocabularies as PostgreSQL ENUM types or as reference tables with FK constraints. Either mechanism satisfies the logical model if it enforces the value set. | DESIGN SPACE |

### §26.13.3 SHACL Validation

| ID | Constraint | Level |
|---|---|---|
| §26-C10 | SDK implementations MUST load and enforce all Pass 1 (SHACL Core) shapes on every write operation. Pass 1 shapes are always Blocking. | MUST |
| §26-C11 | SDK implementations MUST load Pass 2 (SHACL-SPARQL) shapes and enforce them at the profile-configured schedule and enforcement mode. | MUST |
| §26-C12 | SDK implementations MUST NOT downgrade Pass 1 shapes below Blocking. No profile configuration may weaken Pass 1 enforcement. | MUST NOT |
| §26-C13 | SDK implementations MUST NOT set Pass 2 shapes to Advisory mode. Advisory is reserved for non-invariant Constraints. The minimum enforcement for Pass 2 shapes is Logging. | MUST NOT |
| §26-C14 | SDK implementations MUST run the B7 schema conformance test at deployment time and on every schema change. | MUST |
| §26-C15 | DESIGN SPACE: SDK implementations may use any SHACL-compliant validation engine. The shapes are expressed in standard SHACL syntax and do not depend on engine-specific extensions. | DESIGN SPACE |

### §26.13.4 State Machine Alignment

| ID | Constraint | Level |
|---|---|---|
| §26-C16 | SDK implementations MUST enforce ENUM value constraints matching the state machine states defined in §26.10 for all ten state machines. | MUST |
| §26-C17 | SDK implementations MUST enforce transition guards: state transitions that require Decision references (Table 26.10.2) MUST reject transitions without a valid `decision_id`. | MUST |
| §26-C18 | SDK implementations MUST enforce SM-10 monotonic promotion: `truth_type` transitions MUST follow `derived → declared → authoritative` ordering. Reverse transitions MUST be rejected. | MUST |
| §26-C19 | SDK implementations MUST enforce SM-10 orthogonality: `verification_state`, `truth_type`/`promotion_workflow_state`, and `claims_graduation_stage` MUST transition independently. | MUST |
| §26-C20 | DESIGN SPACE: SDK implementations may enforce state machine transitions at the application layer or through database triggers/constraints. Either mechanism satisfies the logical model if invalid transitions are rejected before commit. | DESIGN SPACE |

### §26.13.5 CAS Removal

| ID | Constraint | Level |
|---|---|---|
| §26-C21 | SDK implementations MUST NOT include CAS (Community Accountability Substrate) as a profile type, substrate tier, or content package. | MUST NOT |
| §26-C22 | SDK implementations MUST remediate the four CAS reference points identified in Table 26.8.4. | MUST |

### §26.13.6 Actor Layer

| ID | Constraint | Level |
|---|---|---|
| §26-C23 | SDK implementations MUST enforce AI-Cannot-Be-Principal: AI actors (actor_type ∈ {ai_automation, ai_agentic, ai_assistive}) MUST NOT hold the A (Accountable) RACIVG role. | MUST |
| §26-C24 | SDK implementations MUST enforce AI-Cannot-Be-Root-Authority: AI actors MUST NOT hold root authority (`is_root_authority = true`). | MUST |
| §26-C25 | SDK implementations MUST enforce RACIVG role constraints per profile: EAS permits all six roles; BAS restricts to R, A, C, I; PAS uses simplified Owner/Collaborator mapping. | MUST |
| §26-C26 | SDK implementations MUST enforce delegation autonomy level bounds: `autonomy_level` MUST NOT exceed the graduation stage mapping (A→0, A+→1, B→2, C→3). | MUST |
| §26-C27 | DESIGN SPACE: SDK implementations may implement the Actor Layer as a separate table (`dlp.actors`) joined to `dlp.entities`, or as extended columns on `dlp.entities`. The logical model specifies separate tables; implementations may optimize if all constraints are preserved. | DESIGN SPACE |

---

## §26.14 Terminology

### Table 26.14.1: §26 Terminology Definitions

| Term | Definition |
|---|---|
| **Companion table** | A secondary table providing detailed records (parties, terms, transactions, dependencies) for a primary primitive table — structurally linked by FK |
| **Controlled vocabulary** | A constrained set of permitted values implemented as a PostgreSQL ENUM type or reference table with FK constraint |
| **Five-dimensional routing context** | The five fields on every work specification — cognitive type, decision type, consequence level, operating posture, graduation stage — that determine routing behavior (§A1) |
| **Governed primitive type** | The polymorphic ENUM (`dlp.governed_primitive_type`) listing all nineteen primitive class names, used wherever a field references "any primitive" |
| **Join table** | A table implementing an M:M cross-primitive relationship, carrying the FK pair plus relationship metadata |
| **Pass 1 (SHACL Core)** | The first validation pass: per-instance shapes running at O(1) on every write, always Blocking |
| **Pass 2 (SHACL-SPARQL)** | The second validation pass: graph-traversal shapes running on a configurable schedule with profile-configurable enforcement modes |
| **Universal field set** | The six fields (instance_id, truth_type, record_lifecycle_state, created_at, created_by, version) present on every DLP domain table (Table 26.1.2) |

---

## §26.15 Cross-References

### Table 26.15.1: §26 Cross-Reference Map

| Section | Relationship to §26 |
|---|---|
| §4 — Irreducible Primitives | §26 implements the nineteen primitives as logical tables; thirteen cross-primitive relationships (§4.4) map to FK and join table implementations; primitive tier activation (§4.5) governs which tables are deployment-mandatory |
| §5 — Behavioral Invariants | B1, B2, B4, B5, B6 enforced structurally through NOT NULL FKs and CHECK constraints; B3 enforced via truth_type NOT NULL on Evidence; B7 enforced via governed_primitive_type ENUM accepting all 19 types; B8, B9 enforced via SHACL-SPARQL Pass 2 shapes |
| §6 — Truth Type System | SM-10 three-dimensional state model implemented across truth_type, verification_state, promotion_workflow_state, and claims_graduation_stage fields on dlp.evidences |
| §7 — Minimum Viable Record | Universal field set (Table 26.1.2) implements MVR requirements |
| §9 — State Transformation Model | Conservation laws enforced through schema constraints; record_lifecycle_state (active/superseded) implements the append-only transformation model |
| §12 — Governance Activation | Six-stage activation pipeline implemented via governance_stage ENUM on dlp.activations; EDIM governance registry tables support activation rule evaluation |
| §15 — Signal Architecture | Signal lifecycle (SM-7) implemented via dlp.signal_flags with seven-state status ENUM; reservation_nature ENUM supports §15 reservation semantics |
| §16 — Investigative Queries | IQ lifecycle (SM-8) implemented via dlp.investigative_queries with seven-state status ENUM; eight intent types from §16 |
| §21 — Substrate Profiles | Content package schemas (§26.8) implement EAS/BAS/PAS profile-specific tables; profile_type ENUM replaces v8 substrate_tier |
| §22 — Actor Layer | dlp.actors, dlp.role_envelopes, dlp.delegations implement the four actor types, RACIVG model, and delegation lifecycle; AI-Cannot-Be-Principal enforced via SHACL shape |
| §23 — Portfolio Patterns | dlp.instances.parent_instance_id and constraint_cascade_parent_id implement portfolio hierarchy and tighten-only cascade; dlp.profile_graduation_evaluations implements SM-3 |
| §24 — Entity & Licensing Structure | License verification fields integrate with §24.4; conformance testing validates against §26 schema |
| §25 — License Terms Architecture | License-as-Constraint representation uses dlp.constraints table; license scope dimensions map to instance_id, profile_type, and actor count fields |
| §A1 — Agent Runtime | Work specification table implements five-dimensional routing context; AICAR cognitive types, composition patterns, and consequence levels as ENUMs |
| §A2 — Work Orchestration | dlp.work_specifications and companion tables implement the orchestration grammar; SM-4 lifecycle governs work specification state |

---

## Scope

Scope limited to logical model specification. Transport protocol, serialization format, and implementation languages are DESIGN SPACE.

## Implementation Requirements

SDK implementations MUST respect all invariant shapes, enforce truth type boundaries, and validate all constraints at specified passes. Specification is immutable during lock period. SHACL two-pass validation is mandatory. Truth type boundaries are preserved through all operations. Session isolation is required for all writes.

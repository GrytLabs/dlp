# §17 Decision Surface Design

§13 specifies Policy Projection Surfaces — read-only governance documents rendered from substrate state. Policy projections reveal what the substrate enforces. This section specifies Decision Surfaces — interactive interfaces that present governance state requiring human action. Decision Surfaces reveal what requires resolution.

The infrastructure layer connecting raw substrate queries to actionable decision interfaces is the Validated Projection. A Validated Projection is a Derived truth object with structural enforcement: schema-validated, provenance-tracked, and automatically recomputed from source primitives. Validated Projections occupy the space between ad-hoc derived queries (ephemeral, no schema) and full primitives (independent lifecycle, state machine, events). They have fixed schemas, structural validation rules, recomputation triggers wired to source primitive event handlers, and provenance tracking. They do not have independent lifecycles, their own events, Authoritative or Declared truth status, or direct mutability.

Three Validated Projection types are registered: Plan Baseline, Condition Assessment, and Prediction. Together they compose the Decision Surface — the interface through which a human evaluates a governance condition, reviews predictions for each proposed action, and resolves the condition through a Decision primitive.

Decision Surfaces compose from Validated Projections to enable informed human action. Where Policy Projection Surfaces (§13) render what the substrate enforces as human-readable documents, Decision Surfaces present what requires human resolution as actionable interfaces. The two are complementary outputs of substrate state — one for comprehension, one for action.

### §17.1 The Decision Surface Principle

#### Table 17.1.1: Decision Surface vs. Policy Projection

| Dimension | Policy Projection (§13) | Decision Surface (§17) |
|---|---|---|
| Purpose | Render governance state as human-readable documents | Present governance conditions requiring human resolution |
| Interaction | Read-only — projections may not add governance | Interactive — human selects disposition, creating primitives |
| Output | Documents (authority matrices, SOPs, compliance checklists) | Decision interfaces (condition + baseline + options + predictions) |
| Trigger | Scope selection (human or organizational unit defines the surface) | Condition Assessment materialization (gap between reality and expectation detected) |
| Consumers | Governance officers, auditors, regulators, organizational members | Decision-makers with authority over the condition's scope |

#### Table 17.1.2: Protocol Layer Placement

| Protocol Layer | Decision Surface Content | Example |
|---|---|---|
| DLP-Core | VP truth type is Derived (§6) — not a fourth truth type | All VP output enters with `TruthType = Derived`; actions taken based on VP output require human authority |
| DLP-World-Model | Conservation law context — VPs preserve organizational symmetries through source primitive references | A Plan Baseline preserves obligation invariance (§9) by tracing expected states to Commitment and Intent source primitives |
| DLP substrate | VP infrastructure: registered types, schemas, materialization engine, cache state model, disposition routing | Plan Baseline materialization, Condition Assessment dimensional analysis, Prediction computation, disposition → primitive routing |
| Licensee Configuration | Visualization, thresholds, staleness windows, display preferences | Materiality threshold mapping (L1–L4), valid_until windows by VP type, prediction confidence display format |

### §17.2 Validated Projection Contract

The VP contract specifies the schema and validation rules that all Validated Projections must satisfy. Every registered VP type extends this base contract with type-specific fields and validation logic.

#### Table 17.2.1: VP Required Schema Fields

| Field | Type | Requirement | Description |
|---|---|---|---|
| `projection_id` | Identifier | MUST be unique | Stable reference for this materialization |
| `projection_type` | Enumeration | MUST be registered type | Which VP schema applies |
| `materialized_at` | Timestamp | MUST record when computed | Cache timestamp |
| `materialized_by` | Reference → Entity | MUST record who/what triggered | System, agent, or human |
| `version` | Integer | MUST increment on recomputation | Cache version (monotonic) |
| `valid_until` | Timestamp | MUST define staleness boundary | When recomputation is required |
| `source_refs` | Array[Reference] | MUST list all source primitives | What data was used |
| `derivation_method` | Value (string) | MUST describe computation logic | How the projection was built |
| `confidence` | Number (0.0–1.0) | MUST assess projection reliability | Computed from source quality |
| `status` | Enumeration | MUST track cache state | `current`, `stale`, `recomputing`, `invalidated` |

#### Table 17.2.2: VP Required Provenance Fields

| Field | Type | Requirement | Description |
|---|---|---|---|
| `source_snapshot` | Object | MUST capture source state at materialization | Hash or version of each source primitive at computation time |
| `assumptions` | Array[String] | MUST list computational assumptions | What was assumed during derivation |
| `limitations` | Array[String] | SHOULD list known gaps | What the projection does not cover |
| `supersedes_ref` | Reference → Projection | If replacing previous materialization | Link to prior version |

#### Structural Validation

All Validated Projections are structurally valid only when:

- `projection_type` is a registered type
- Every reference in `source_refs` resolves to a valid primitive instance
- `confidence` falls within the 0.0–1.0 range
- `valid_until` is after `materialized_at`
- `status` transitions follow the cache state model (§17.3)
- `source_snapshot` contains a hash or version for every entry in `source_refs`

VP materialization is automatic. Actions taken based on a VP's output require human approval. This distinction preserves the epistemic boundary (§6.2): Derived truth does not require a human gate for creation, only for promotion or for governance acts taken on its basis.

**SDK Constraints:**
- **MUST:** Enforce structural validation on every materialization before transitioning to `current` status.
- **MUST:** Record `source_snapshot` capturing the state of every source primitive at materialization time.
- **MUST NOT:** Serve a VP whose `source_refs` contain unresolvable references.
- **DESIGN SPACE:** Whether provenance `limitations` are enforced as required or advisory is an implementation choice. The architecture requires the field to exist.

### §17.3 VP Cache State Model

The cache state model governs materialization freshness — whether a cached VP can be served, needs recomputation, or requires human intervention. This is a cache state model, not a lifecycle state machine. Validated Projections have no independent lifecycle. Their states describe the relationship between materialized output and source data, not an epistemic trajectory.

This section summarizes the cache state model for architectural context.

#### Table 17.3.1: Cache State Summary

| State | Entry Condition | Exit Transitions | Human Intervention Required |
|---|---|---|---|
| `Current` | Materialized; `valid_until` not exceeded; source data unchanged | → Stale (time or event), → Invalidated (source basis broken) | No |
| `Stale` | `valid_until` exceeded or source primitive event detected | → Recomputing (scheduled or demand), → Invalidated (escalation) | No |
| `Recomputing` | Recomputation enqueued or in progress | → Current (success), → Stale (transient failure), → Invalidated (source invalidated during processing) | No |
| `Invalidated` | Source data changed in a way that breaks the derivation basis | → Recomputing (human approves re-materialization) | Yes |

`Current` is the only state from which a VP may be served without qualification. `Stale` indicates the derivation method is still sound but source data may have changed. `Recomputing` is transient. `Invalidated` means the derivation basis is broken — mechanical recomputation would produce formally valid but semantically wrong output.

#### Recomputation Triggers

Three categories, ordered by urgency:

**Time-based.** The `valid_until` window has elapsed. The scheduler transitions the VP to `Stale` and enqueues recomputation at normal priority.

**Event-based.** A source primitive state change is detected that affects the VP's derivation method. The VP transitions to `Stale` and recomputation is enqueued at high priority. Event-based triggers take precedence over time-based because they indicate actual source data change, not temporal drift.

**Demand-based.** A consumer accesses a VP while it is `Stale`. Recomputation is synchronous — the consumer waits. This is acceptable because VP recomputation is a query operation over existing primitives, not a long-running computation. If recomputation exceeds a configurable timeout, the system returns the prior version with a staleness warning and enqueues asynchronous recomputation.

#### Recomputation Invariants

Seven invariants constrain every recomputation operation regardless of trigger type.

| ID | Invariant |
|----|-----------|
| R1 | Recomputation MUST NOT change source data — read-only transaction on source primitives, write-only on the VP record |
| R2 | Recomputation MUST produce identical output given identical inputs — same `source_snapshot` + same `derivation_method` = same output |
| R3 | Recomputation MUST update provenance fields — `materialized_at`, `source_snapshot`, `version`, `confidence`, `supersedes_ref` |
| R4 | Recomputation MUST NOT require human approval — VP materialization is Derived truth (§6); human gates apply to actions based on the VP |
| R5 | Recomputation MUST increment version — `version` increments monotonically; prior version retained for audit |
| R6 | Recomputation MUST validate structural schema — output passes the VP's structural validation rules before transitioning to `Current` |
| R7 | Actions taken BASED ON the projection DO require human approval — disposition routing from Condition Assessment creates Decision primitives requiring human resolution |

R1 preserves epistemic integrity (§5, B3): recomputation cannot promote Derived content to Declared or Authoritative. R4 is a consequence of B3: Derived truth does not require human gates for creation. R7 preserves the §6 epistemic boundary: projections inform decisions but do not make them.

#### Version Retention

Recomputation retains the current and prior versions. The `supersedes_ref` field links each version to its predecessor. Consumers querying "what did this VP say at time T?" can retrieve the version that was `Current` at that timestamp. This creates the provenance chain: Decision → VP (version N) → source primitives (snapshot).

**SDK Constraints:**
- **MUST:** Retain current and prior VP versions for provenance audit.
- **MUST:** Enforce all seven recomputation invariants (R1–R7) on every recomputation.
- **MUST NOT:** Serve a VP in `Invalidated` state to consumers. Invalidated VPs are retained for audit but blocked from active use.
- **DESIGN SPACE:** Archival policy for versions beyond current + prior is an implementation choice. The architecture requires at minimum two versions.

### §17.4 Plan Baseline

Plan Baseline is a Validated Projection that materializes the expected-state trajectory for a subject — Intent, Entity, Commitment, or Account — over time. It answers: "What did we expect to be true at each point, and where did that expectation come from?"

Plan Baseline is the foundation of the Decision Surface stack. Without a baseline, Condition Assessment has no expectation to compare against, and Prediction has no reference trajectory to project from.

#### Table 17.4.1: Plan Baseline Schema (extends VP base)

| Field | Type | Requirement | Description |
|---|---|---|---|
| `subject_ref` | Reference | MUST identify what baseline is for | Intent, Entity, Commitment, or Account |
| `subject_type` | Enumeration | MUST classify the subject | `intent`, `entity`, `commitment`, `account` |
| `baseline_mode` | Enumeration | MUST declare expectation source | How the baseline was established |
| `expected_states` | Array[Expected State] | MUST contain at least one | Time-series of expected values, chronologically ordered |

Each expected state entry contains: `as_of` (timestamp), `expected_value` (structure varies by `subject_type`), `source_ref` (which source record generated this expectation), `confidence` (0.0–1.0 for this specific point), and `notes` (contextual explanation).

#### Table 17.4.2: Baseline Mode Definitions

| Mode | Source | Required Fields | Confidence Base |
|---|---|---|---|
| `explicit` | Declared artifact — someone committed to this plan | `declared_by` (→ Entity), `declared_at` (Timestamp), `declaration_evidence_ref` (→ Evidence) | 0.85 |
| `inferred` | Reconstructed from Account/Evidence patterns | `derived_from` (Array, ≥1), `derivation_method` (algorithm description), `sample_size` (≥1), `sample_period` (Duration) | 0.50 |
| `normative` | Derived from standards or policy | `standard_ref` (standard identifier), `constraint_refs` (Array → Constraint), `interpretation_notes` (SHOULD) | 0.75 |
| `comparative` | Derived from cohort or peer data | `cohort_definition` (selection criteria), `cohort_size` (≥1), `cohort_source_ref` (→ Evidence), `similarity_score` (SHOULD, 0.0–1.0) | 0.45 |

#### Confidence Computation

Base confidence is determined by mode. Adjustments refine it:

**Inferred mode:** `sample_adjustment = min(sample_size / 8, 1.0) × 0.2` — more data increases confidence, capping at eight periods. `recency_adjustment = decay(now() − most_recent_source.timestamp)` — more recent data increases confidence. Final: `base + sample_adjustment − recency_adjustment`.

**Comparative mode:** `cohort_adjustment = min(cohort_size / 10, 1.0) × 0.15` — larger cohorts increase confidence. `similarity_adjustment = (similarity_score − 0.5) × 0.2` — higher similarity increases confidence. Final: `base + cohort_adjustment + similarity_adjustment`.

**Explicit mode:** `staleness_penalty = decay(now() − declared_at, half_life = "P6M")` — explicit plans degrade as time passes without reaffirmation. Final: `base − staleness_penalty`.

**Normative mode:** No adjustment — confidence reflects the standard's authority directly.

All confidence values are clamped to the 0.0–1.0 range.

#### Recomputation Triggers

| Baseline Mode | Staleness Trigger | Invalidation Trigger | Recomputation Notes |
|---|---|---|---|
| `explicit` | Success criteria or commitment terms changed | Declaring entity's authority revoked | Recomputes expected_states from updated criteria; confidence staleness penalty applies |
| `inferred` | New transaction within sample_period | None — inferred baselines degrade gracefully | Batched at period end, not per-transaction; recalculates trailing statistics |
| `normative` | No time-based expiry — event-driven only | Referenced Constraint superseded | Invalidates, not stales — new normative baseline requires human selection of replacement standard |
| `comparative` | Cohort data updated | Cohort definition invalidated (peer reclassified) | Recomputes from updated cohort; invalidates if comparison group is no longer valid |

Normative baseline invalidation on constraint supersession transitions to `Invalidated`, not `Stale`. The standard that defined the baseline no longer applies. A new normative baseline must be established against the replacement standard — this is a human judgment, not a mechanical recomputation.

**SDK Constraints:**
- **MUST:** Enforce mode-specific required fields on materialization. A `normative` baseline without `constraint_refs` fails structural validation.
- **MUST:** Compute confidence using the formula specified for each mode. Confidence values MUST NOT be manually assigned.
- **MUST NOT:** Automatically recompute a normative baseline after constraint supersession. The VP enters `Invalidated` and awaits human review.
- **DESIGN SPACE:** The `valid_until` window per baseline mode (explicit: 30–90 days, inferred: 7–30 days, comparative: 14–30 days) is configurable per substrate deployment.

### §17.5 Condition Assessment

Condition Assessment is a Validated Projection that evaluates current state against a Plan Baseline. It answers: "How does reality compare to expectation, and what does the gap imply?"

Condition Assessment is the analytical core of the Decision Surface. It transforms the comparison between a Plan Baseline and current state into a structured seven-dimensional analysis that routes to specific governance actions through disposition.

#### Table 17.5.1: Condition Assessment Schema (extends VP base)

| Field | Type | Requirement | Description |
|---|---|---|---|
| `baseline_ref` | Reference → Plan Baseline | MUST reference the baseline compared against | Which expectation |
| `baseline_mode` | Enumeration | MUST propagate from baseline | Inherited — so routing knows the confidence context |
| `current_state_ref` | Reference → Account or Evidence | MUST reference actual state | What reality looks like |
| `assessment_dimensions` | Array[Dimension Assessment] | MUST contain at least one | The dimensional analysis |

Each dimension assessment entry contains: `dimension` (enumeration), `value` (enumeration), `evidence_refs` (Array → Evidence), and `notes` (contextual explanation).

#### Table 17.5.2: Seven Assessment Dimensions

| Dimension | Question | Values |
|---|---|---|
| 1. Condition Type (What) | What kind of gap exists? | `gap` — expected state not yet reached; `overrun` — exceeded expected bounds; `overlap` — roles/functions not separated; `sequence_break` — actions out of order; `threshold_breach` — value crossed defined boundary; `staleness` — data not refreshed within cadence; `alignment` — current state matches expectation |
| 2. Causation (Why) | Why does this condition exist? | `structural` — organizational design; `operational` — day-to-day execution; `external` — outside forces; `temporal` — time passage (drift); `capacity` — resource/capability limitation; `planning` — original plan flawed or incomplete; `unknown` — not yet determined |
| 3. Materiality (How Much) | How significant is the condition? | `L1_informational` — log only; `L2_advisory` — acknowledgment expected; `L3_action_required` — decision required; `L4_critical` — immediate response required |
| 4. Permanence (How Fixed) | Can the condition be changed? | `transitional` — will self-correct without intervention; `structural` — requires deliberate intervention; `permanent` — cannot be changed, must be accommodated; `unknown` — not yet assessed |
| 5. Controllability (By Whom) | Who can address it? | `internal_authority` — within org's decision authority; `internal_operational` — within org's operational capacity; `external_dependent` — requires external party action; `inherent` — built into the context, no one controls it; `shared` — requires coordinated action |
| 6. Disposition Pathway (Then What) | What governance action is appropriate? | Nine dispositions — see §17.6 |
| 7. Trajectory (Where Heading) | How is the condition trending? | `converging` — gap closing; `stable` — gap holding; `diverging` — gap widening; `accelerating` — divergence rate increasing; `oscillating` — alternating between converging and diverging; `insufficient_data` — not enough history |

The seven dimensions are ordered from descriptive to actionable. Dimensions 1–5 characterize the condition. Dimension 6 determines the governance response. Dimension 7 provides temporal context for urgency assessment. A Condition Assessment with `condition_type = alignment` and `materiality = L1_informational` typically routes to `disposition = no_action` — the system is operating as expected.

**SDK Constraints:**
- **MUST:** Propagate `baseline_mode` from the referenced Plan Baseline. Routing logic uses baseline mode to weight confidence.
- **MUST:** Require at least one assessment dimension per Condition Assessment.
- **MUST NOT:** Auto-select disposition (Dimension 6) without presenting the full dimensional analysis to a human decision-maker.
- **DESIGN SPACE:** Whether all seven dimensions are required or a subset is sufficient for a given Condition Assessment is an implementation choice. The architecture defines the vocabulary; profiles may define minimum dimension sets.

### §17.6 Disposition Routing

Each disposition pathway creates specific primitive instances. Disposition routing is the bridge from assessment to governance action — the mechanism that converts a Condition Assessment's seventh dimension into concrete substrate operations.

Human authority resolves disposition. Every disposition routing is a governance act requiring human decision. The system presents dispositions as options with predicted outcomes (§17.7); the human selects. No auto-disposition exists in the architecture.

#### Table 17.6.1: Disposition → Primitive Routing

| Disposition | Creates | Primitive Type | Key Fields |
|---|---|---|---|
| `investigate` | Work | `research` type | Title: "Investigate condition: {summary}"; acceptance criteria: root cause identified and documented; verification method: `manual_review` |
| `re_evaluate_plan` | Baseline recomputation + Decision | Baseline recompute trigger + `selection` type Decision | Triggers Plan Baseline recomputation; creates Decision on planning assumptions with options generated from assessment |
| `compensate` | Constraint exception + compensating Constraint | Exception on violated Constraint | Activates compensating controls; creates exception with `compensating_control_required = true` |
| `acknowledge` | Evidence + scheduled Work | `attestation` type Evidence + review Work | Records acknowledgment; schedules review at next review cadence |
| `escalate` | Decision | `escalation` type Decision | Routed via Authority chain; urgency mapped from materiality level |
| `accept_risk` | Learning + Decision | `weight_update` type Learning + authorizing Decision | Documents risk acceptance as Learning; REQUIRES Decision authorizing the acceptance — risk acceptance without explicit authority is not permitted |
| `remediate` | Work | `task` type Work | Corrective action with acceptance criteria: condition resolved, re-assessment shows alignment; verification method: `automated_test` (re-run Condition Assessment) |
| `defer` | Scheduled recomputation + Evidence | Future Condition Assessment schedule + `attestation` type Evidence | Schedules future re-assessment at specified deadline; records deferral attestation |
| `no_action` | Audit log entry | None — no primitive creation | Condition aligned; logged for audit trail |

#### Disposition-Materiality Alignment

Dispositions correlate with materiality levels but are not mechanically determined by them. An `L4_critical` condition with `controllability = inherent` may warrant `accept_risk` rather than `remediate`. An `L1_informational` condition with `trajectory = accelerating` may warrant `investigate` rather than `no_action`. The dimensional analysis provides the reasoning basis; the human provides the judgment.

**SDK Constraints:**
- **MUST:** Create the specified primitive types for each selected disposition. A `remediate` disposition MUST produce a Work primitive with re-assessment criteria.
- **MUST:** Require a Decision primitive for `accept_risk` disposition. Risk acceptance without explicit authority violates the epistemic boundary.
- **MUST NOT:** Execute disposition routing without human selection. Disposition is a governance act.
- **DESIGN SPACE:** Whether multiple dispositions can be selected for a single Condition Assessment (e.g., `investigate` + `escalate` simultaneously) is an implementation choice.

### §17.7 Prediction

Prediction is a Validated Projection that models "what happens if we take this action" — projecting future state given a proposed disposition. Predictions are short-lived VPs with aggressive staleness windows (hours to days). Their confidence degrades rapidly with time.

#### Table 17.7.1: Prediction Schema (extends VP base)

| Field | Type | Requirement | Description |
|---|---|---|---|
| `subject_ref` | Reference → Condition Assessment | MUST reference the condition being acted on | Which condition this prediction addresses |
| `predicted_states` | Array[Predicted State] | MUST contain at least one | Time-series of projected future values |
| `if_action` | Disposition | MUST specify which disposition is assumed | Which action this prediction models |
| `model_ref` | Reference | MUST identify the prediction model used | Provenance for the prediction methodology |

Each predicted state entry contains: `as_of` (future timestamp), `predicted_value` (projected state), `confidence` (0.0–1.0), and `assumptions` (what was assumed for this prediction point).

#### Investigability Chain

Every Prediction exposes a full investigability chain: prediction → model → assumptions → source data. A user seeing "If you remediate, we predict alignment by Q3 with 0.72 confidence" can drill into each link:

- **Prediction:** What specific future states are projected, at what confidence
- **Model:** Which prediction model was used, its track record, its limitations
- **Assumptions:** What was assumed — which assumptions, if changed, would alter the prediction
- **Source data:** The Condition Assessment, Plan Baseline, and underlying primitives that fed the model

The user can challenge any assumption and request an alternative prediction with modified parameters. The challenge produces a new Prediction VP, not a modification of the existing one.

#### Recomputation and Staleness

| Trigger | Response |
|---|---|
| Referenced Condition Assessment recomputed | Mark Stale → enqueue recomputation |
| Referenced Condition Assessment invalidated | Mark Invalidated (cascade) — prediction is baseless without valid condition |
| Prediction model updated | Mark Stale → enqueue recomputation with new model version |
| Prediction model deprecated | Mark Invalidated — derivation method no longer valid; human selects replacement model |
| `valid_until` exceeded | Mark Stale — but consider whether re-prediction is useful; short-lived by design |

**SDK Constraints:**
- **MUST:** Expose the full investigability chain (prediction → model → assumptions → source data) in the Decision Surface rendering.
- **MUST:** Record `model_ref` for every Prediction. Predictions without model provenance fail structural validation.
- **MUST NOT:** Present Predictions without their confidence scores and assumption lists. Predictions displayed without epistemic context violate the Derived truth boundary.
- **DESIGN SPACE:** Whether Predictions are pre-computed for all available dispositions or computed on demand when a user explores a disposition option is an implementation choice.

### §17.8 Decision Surface Composition

The Decision Surface — what the user actually sees and acts on — composes from the three registered VP types into a unified interface. The composition model defines four components, each traceable to its source VP.

#### Table 17.8.1: Decision Surface Components

| Component | Source | What It Shows |
|---|---|---|
| The condition | Condition Assessment | What was found (`condition_type`), why it exists (`causation`), how significant (`materiality`), how fixed (`permanence`), who can address it (`controllability`), where it is heading (`trajectory`) |
| The baseline context | Plan Baseline | What was expected (`expected_states`), how the expectation was established (`baseline_mode`), how confident the expectation is (`confidence`) |
| Proposed actions | Disposition routing (§17.6) + Prediction (§17.7) | Each disposition as a selectable option, each with predicted outcome and confidence from Prediction VP |
| Investigability chain | Evidence + source primitives | Every claim traces to Evidence, every prediction to model + assumptions, every baseline to source data — user can drill into any link |

#### Agent Participation

AI agents participate in Decision Surface composition through the Interpretation primitive (§4.6.3):

- The agent proposes a recommendation with confidence level and reasoning chain
- The agent provides alternatives with trade-off analysis for each disposition option
- Agent recommendations enter the Decision Surface as Derived truth — flagged and reviewable, never presented as authoritative guidance
- The human resolves; the agent assists

Agent participation follows the stakes-by-novelty routing matrix (§6.3). High-stakes, high-novelty conditions route agent recommendations through expert interpretation review. The rubber-stamp detection invariant applies: a Decision that adopts an agent recommendation without evidence of human review is rejected.

#### Composition Integrity

The Decision Surface is a composed view, not a persistent object. It is materialized on demand from the current versions of its component VPs. If any component VP transitions to `Stale` or `Invalidated` between surface composition and human resolution, the surface MUST re-render with updated components before the Decision primitive is created. A Decision primitive records the specific VP versions (projection_id + version) that were `Current` at resolution time, creating the audit provenance: Decision → VP versions → source primitive snapshots.

**SDK Constraints:**
- **MUST:** Record the VP version references (projection_id + version for each component) in the Decision primitive created from a Decision Surface resolution.
- **MUST:** Re-render the Decision Surface if any component VP state changes before human resolution.
- **MUST:** Flag agent recommendations as Derived truth with visible attribution in the Decision Surface.
- **MUST NOT:** Allow Decision creation from a surface whose component VPs include any in `Invalidated` state.
- **DESIGN SPACE:** Visual layout, interaction patterns, and rendering format for Decision Surface components are implementation choices. The architecture specifies the composition model and integrity requirements, not the presentation.

### §17.9 VP Registration and Cascade Rules

#### Registration Criteria

A concept qualifies as a Validated Projection — rather than a full primitive or a raw derived query — when it satisfies five criteria derived from the Concept Stress Testing Protocol (§4.2).

#### Table 17.9.1: VP Registration Criteria

| Criterion | Test | Pass Condition |
|---|---|---|
| CSTP Removal | System degrades but does not break without the VP | Removal loses structured analysis but does not eliminate a governance question |
| CSTP Merger | No clean merger with existing primitives | Composition is possible but loses critical metadata |
| CSTP Sufficiency | Composable from existing primitives | Fully derivable from source primitives but requires schema enforcement for downstream routing or trust |
| Critical Metadata | Lost metadata is critical for routing or trust assessment | Confidence propagation, baseline mode, dimensional analysis, or provenance tracking would be lost without schema enforcement |
| No Independent Lifecycle | The concept does not transition states independently of its sources | VP reflects source primitive state; it does not have its own lifecycle events |

Concepts that fail these criteria in one direction or the other are not VPs. If the concept has an independent lifecycle, generates its own events, or cannot be recomputed from source primitives, it is a primitive candidate. If no schema enforcement is needed, provenance tracking is unnecessary, and confidence labeling adds no value, it remains a raw derived query.

#### Table 17.9.2: Registered VP Types

| Type | Purpose | Source Primitives | First Use |
|---|---|---|---|
| `plan_baseline` | Expected-state trajectory over time | Intent, Account, Evidence, Commitment, Constraint | Condition Assessment baseline |
| `condition_assessment` | Reality vs. expectation dimensional analysis | Plan Baseline, Account, Constraint, Learning | Decision Surface input |
| `prediction` | Projected future state given proposed action | Condition Assessment, Account, Learning (historical patterns) | Decision option evaluation |

#### Invalidation Cascade

The three registered VP types form a directed dependency chain:

```
Plan Baseline → Condition Assessment → Prediction
```

Invalidation cascades through this chain. Staleness does not.

#### Table 17.9.3: VP-Type Invalidation Cascade

| Source VP | Affected VP | Cascade Rule |
|---|---|---|
| Plan Baseline → Invalidated | Condition Assessment referencing this baseline | Invalidated (cascade is immediate and automatic) |
| Condition Assessment → Invalidated | Prediction referencing this assessment | Invalidated (cascade is immediate and automatic) |
| Plan Baseline → Stale | Condition Assessment referencing this baseline | NOT automatically Stale — Condition Assessment evaluates its own staleness independently |
| Condition Assessment → Stale | Prediction referencing this assessment | NOT automatically Stale — Prediction evaluates its own staleness independently |

The asymmetry between invalidation and staleness cascade is architectural, not incidental. Invalidation means the derivation basis is broken — a downstream VP whose upstream source is invalidated is definitionally unsound. Serving it would violate provenance integrity. Staleness means the output may need refresh — the downstream VP's own event handlers evaluate whether the upstream staleness affects its derivation. Cascading staleness would create recomputation storms: a single source primitive change would trigger recomputation of every VP in the chain, most of which may be unaffected.

#### Future Candidates

Four VP candidates are identified but not registered. Each requires CSTP analysis before registration, dependent on schema alignment work confirming source primitive query patterns.

| Candidate | Purpose | CSTP Status |
|---|---|---|
| Compliance Posture | Aggregate view across all active Constraints | Not tested — deferred to Phase 7+ |
| Capacity Forecast | Projected capacity given current allocations and commitments | Not tested — deferred to Phase 7+ |
| Authority Map | Materialized view of who-can-do-what given delegation chains | Not tested — deferred to Phase 7+ |
| Decision Impact | Projected downstream effects of a pending Decision | Not tested — deferred to Phase 7+ |

**SDK Constraints:**
- **MUST:** Enforce invalidation cascade immediately when a VP enters `Invalidated` state. The system does not wait for human review of the upstream VP before invalidating downstream VPs.
- **MUST:** Evaluate staleness independently per VP — no automatic staleness cascade.
- **MUST NOT:** Register new VP types without completing the five-criterion registration analysis.
- **DESIGN SPACE:** The registration process for future VP types — whether it requires an architecture decision record, a CSTP session, or both — is a governance choice.

## Scope

Decision Surfaces are interactive interfaces for condition assessment and disposition selection. They do not create governance records directly; disposition routing creates records. Validated Projections carry Derived truth type and do not require human gates for materialization. Actions based on Decision Surfaces require human authority.

## Locked Design Positions

**Locked positions:**

1. Validated Projections are derived objects with fixed schemas, structural validation, and provenance tracking; they occupy the space between ad-hoc queries and full primitives.
2. Three VP types are registered: Plan Baseline, Condition Assessment, Prediction; they compose into the Decision Surface.
3. VP materialization is automatic and carries Derived truth type; actions based on VPs require human approval.
4. Plan Baselines derive from four modes (explicit, inferred, normative, comparative) with mode-specific confidence formulas.
5. Condition Assessment applies seven dimensions to analyze reality-expectation gaps with actionable routing.
6. Disposition routing supports nine dispositions with primitive creation; risk acceptance requires explicit Decision authority.
7. Prediction exposes full investigability chains and allows assumption-based alternatives.
8. Decision Surfaces materialize on demand from current VP versions; component state changes re-render before primitive creation.
9. Invalidation cascades immediately through VP dependency chain (Plan Baseline → Condition Assessment → Prediction); staleness does not cascade.
10. VP versions are retained for audit provenance; Decisions record specific VP versions (projection_id + version) that were Current at resolution time.

## Implementation Requirements

**SDK MUST constraints:**

- MUST enforce structural validation on every VP materialization before transitioning to `current` status.
- MUST record `source_snapshot` capturing the state of every source primitive at materialization time.
- MUST NOT serve VPs in `Invalidated` state to consumers.
- MUST retain current and prior VP versions for provenance audit.
- MUST enforce all seven recomputation invariants (R1–R7) on every recomputation.
- MUST implement four-state cache state model: Current, Stale, Recomputing, Invalidated.
- MUST support time-based, event-based, and demand-based recomputation triggers.
- MUST implement Plan Baseline with four modes (explicit, inferred, normative, comparative) with mode-specific confidence formulas.
- MUST compute Plan Baseline confidence using mode-specific formulas; confidence values MUST NOT be manually assigned.
- MUST NOT automatically recompute normative baselines after constraint supersession; the VP enters `Invalidated` and awaits human review.
- MUST implement Condition Assessment with seven assessment dimensions covering condition type, causation, materiality, permanence, controllability, disposition, and trajectory.
- MUST propagate `baseline_mode` from referenced Plan Baseline to Condition Assessment.
- MUST NOT auto-select disposition without presenting full dimensional analysis to human decision-maker.
- MUST implement nine dispositions with specified primitive creation rules for each.
- MUST require explicit Decision primitive for `accept_risk` disposition.
- MUST NOT execute disposition routing without human selection.
- MUST implement Prediction VP with `if_action` (disposition), `predicted_states`, and `model_ref`.
- MUST expose full investigability chain (prediction → model → assumptions → source data).
- MUST NOT present Predictions without confidence scores and assumption lists.
- MUST implement Decision Surface composition from three VP component types.
- MUST record VP version references (projection_id + version for each component) in the Decision primitive created from surface resolution.
- MUST re-render Decision Surface if any component VP state changes before human resolution.
- MUST flag agent recommendations as Derived truth with visible attribution.
- MUST NOT allow Decision creation from a surface whose component VPs include any in `Invalidated` state.
- MUST enforce invalidation cascade immediately through VP dependency chain (Plan Baseline → Condition Assessment → Prediction).
- MUST evaluate staleness independently per VP; no automatic staleness cascade.
- MUST NOT register new VP types without completing five-criterion registration analysis.

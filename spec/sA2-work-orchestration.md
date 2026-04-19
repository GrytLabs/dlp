# §A2 Work Orchestration


§A1 specifies the structural grammar for AI governance — the AICAR cognitive type taxonomy, the five-dimensional routing space, the delegation model, and the graduation stages. This section specifies how the DLP substrate implements that grammar: the resolution engine that converts work requests into governed Work Specifications, the constraint evaluation mechanics that determine routing, the execution layer interface, and the pattern detection and learning systems that drive graduation.

The relationship between §A1 and §A2 is the same as the relationship between DLP-World-Model and DLP substrate throughout the architecture. §A1 defines what must be true for any DLP implementation. §A2 defines how the DLP substrate makes it true. A competing DLP implementer could build a different Work Orchestration engine; they could not change the structural grammar of §A1.

Work Orchestration introduces no new primitives. Every operation in this section is a composition of the nineteen primitives (§4) — Decision, Authority, Constraint, Work, Evidence, Account, and their Tier 2–5 infrastructure. The orchestration engine is a governed composition pattern, not an extension to the primitive model.

---


## §A2.1 Orchestration Resolution Engine

When a work request arrives, the substrate resolves routing through a seven-step sequence. Each step is a primitive operation with lineage. The sequence is canonical — all steps execute in order, producing a traceable chain from work request to execution.

### Table A2.1.1: Seven-Step Resolution Sequence

| Step | Input | Action | Output | Primitive Operation |
|---|---|---|---|---|
| **1. CLASSIFY** | Work request | Determine work type and required cognitive work types (§A1) | CognitiveWorkType[] | Decision (classification) |
| **2. ACTIVATE** | Work context + domain + five-dimensional position | Evaluate activation constraints; load applicable constraint sets | EffectiveConstraints | Constraint evaluation |
| **3. ROUTE** | CognitiveWorkType[] + EffectiveConstraints | Evaluate routing constraints to identify eligible AI actors | Eligible AIActor[] per component | Constraint + Authority resolution |
| **4. SELECT** | Eligible actors + delegation scopes + five-dimensional resolution | Match actor capabilities to component requirements within delegation bounds | ActorAssignment[] | Decision (selection) + Account (rationale) |
| **5. COMPOSE** | Components + assignments + constraints | Assemble Work Specification | WorkSpecification | Work creation |
| **6. CONFIRM** | WorkSpecification (if required by autonomy level) | Human reviews and approves routing | Confirmed WorkSpecification | Decision (approval) + Authority (human) |
| **7. EMIT** | Confirmed WorkSpecification | Work Specification available to execution layer | Execution layer processes | Work (status → Active) |

Step 6 (CONFIRM) executes conditionally. At Stage A (Capture) and Stage B (Advise), human confirmation is always required. At Stage C (Orchestrate), confirmation is computed from the five-dimensional resolution — the HITL placement is a function of the routing space (§A1), not a fixed design choice. When the five-dimensional resolution produces a low-risk, high-confidence position (e.g., Citation + Cost + Reversible + Normal + Stage C), the substrate may emit without human confirmation. When any dimension elevates risk, confirmation is required.

### Resolution Decision Record

Each resolution produces a RoutingDecision record — a Decision (§4) with routing-specific structure:

| Component | Contents |
|---|---|
| **Alternatives considered** | All eligible AI actors evaluated during the ROUTE step |
| **Selected assignments** | The actor-to-component assignments produced by SELECT |
| **Selection rationale** | Account recording why these actors were selected for these components |
| **Five-dimensional resolution** | The computed position across all five routing dimensions (§A1) |
| **Computed HITL level** | The human-in-the-loop requirement derived from the five-dimensional resolution |
| **Confidence assessment** | Substrate's confidence in the routing decision |
| **Authority basis** | The delegation references being exercised |
| **Resulting Work Specification** | The Work Specification produced by COMPOSE |

When the substrate cannot resolve routing — no eligible actor for a required cognitive work type, constraint conflicts, insufficient delegation, or confidence below the configured floor — it escalates. Escalation is a state transformation with lineage: the substrate records what it could not resolve, why, and to whom the escalation is routed (per B8, §5).

---


## §A2.2 Constraint Evaluation Mechanics

Three constraint types govern AI routing. All three are specializations of the Constraint primitive (§4). They compose through intersection to produce the effective constraint set for any given work request.

### Table A2.2.1: Constraint Type Definitions

| Type | Governs | Applies To | Key Fields |
|---|---|---|---|
| **Routing** | Which actor handles what | WorkType, CognitiveWorkType | Domain match, work type match, cognitive type match, context conditions, preferred actors, required capabilities, excluded actors |
| **Behavioral** | How the actor performs | AIActor, CognitiveWorkType | Evidence grounding, confidence signaling, source attribution, scope boundary, output format, language constraints, enforcement level |
| **Activation** | When constraint sets load | Context, Domain, Temporal | Context match expression, domain match, temporal condition, five-dimensional triggers (decision type escalation, consequence level change, posture shift) |

**Routing Constraints** are evaluated before delegation lookup. They determine which AI actors are eligible for a work request based on work type, cognitive type, domain, and contextual conditions. A routing constraint may specify preferred actors (ordered preference), required capabilities (cognitive work types the actor must support), and excluded actors (actors prohibited for this work type).

**Behavioral Constraints** bind how an AI actor performs work once routed. They travel with the Work Specification and are consumed by the execution layer. Each behavioral constraint specifies an enforcement level: Blocking (prevents non-compliant output), Warning (flags non-compliance but permits output), Logging (records silently), or Advisory (suggests alternatives). The enforcement levels correspond to the four modes defined for the Constraint primitive (§4).

**Activation Constraints** determine when additional constraint sets load based on context. They are the mechanism by which domain-specific rules — regulatory requirements, organizational policies, temporal conditions — dynamically scope AI behavior. An activation constraint fires when its trigger conditions match the current work context, loading the specified constraint set and optionally deactivating superseded constraints. Five-dimensional triggers enable activation based on routing space position: a decision type escalation, consequence level change, or operating posture shift can load additional constraint sets.

### Constraint Composition

Multiple constraints compose through intersection:

```
EffectiveConstraints(work) =
   RoutingConstraints(work.type, work.domain)
 ∩ BehavioralConstraints(work.cognitiveType, selectedActor)
 ∩ ActivationConstraints(work.context, work.dimensionalPosition)
```

The resulting possibility space is the intersection of all active constraints. When constraints conflict — a routing constraint permits an actor that a behavioral constraint excludes — the conflict is itself a Decision requiring resolution through the authority chain. Constraint conflict resolution produces lineage: who resolved the conflict, on what basis, and what the resulting effective constraint set is.

Constraint cascade in recursive structures follows the tighten-only rule (§23). Child instances inherit parent constraints and may tighten but never loosen. The effective constraint set at any recursion level includes all ancestor constraints.

---


## §A2.3 Work Specification Architecture

The Work Specification is the substrate's governed output — the instruction that the execution layer consumes. It represents the substrate's resolution of what cognitive work is needed, who handles each component, and under what constraints. The Work Specification is a composition of primitives, not a new primitive.

### Table A2.3.1: Work Specification Structure

| Component | Contents |
|---|---|
| **Cognitive decomposition** | Component ID, cognitive work type (§A1), description, execution sequence, dependencies between components, input evidence, required output specification |
| **Actor assignments** | Component ID, delegated AI actor, delegation reference (Authority), autonomy level for this component |
| **Effective constraints** | Component ID, active constraints for this component, constraint source references |
| **Specification metadata** | Specification Decision, specification Account, timestamp |
| **Composition strategy** | Composition type (sequential, parallel, conditional), composition actor (AI or human), composition authority (delegation or direct authority) |
| **Five-dimensional context** | Decision type, consequence level, operating posture, graduation stage, cascade history (if threshold breach cascaded from another scope) |

The cognitive decomposition expresses composite work as ordered components. A compliance determination might decompose into: Citation (retrieve regulatory text), Reasoning (analyze whether facts satisfy criteria), Context (consider organizational history and prior determinations), Communication (draft the finding). Each component may route to a different AI actor based on delegation scope, or to the same actor with different constraint sets active.

### Work Specification as State Transformation

Creating a Work Specification is itself a state transformation:

| Element | Value |
|---|---|
| **Trigger** | Work request |
| **Prior state** | Pending work, available actors, active constraints, dimensional position |
| **Delta** | Added: WorkSpecification, Decision (routing), Account (routing rationale). Modified: Work status (pending → specified) |
| **Resulting state** | Specified work with full routing lineage |

This means the routing decision has lineage. "Why did the substrate route this work to Actor A instead of Actor B?" is answerable from the transformation record.

### Table A2.3.2: Work Specification Lifecycle States

| State | Description | Transition Trigger |
|---|---|---|
| **Draft** | Substrate has composed the specification | Work request received; resolution engine completes COMPOSE step |
| **Confirmed** | Human has reviewed and approved (if required by autonomy level) | Human confirmation at CONFIRM step |
| **Active** | Execution layer is processing the specification | Execution layer accepts the specification at EMIT step |
| **Completed** | All components have produced outputs | All component outputs received from execution layer |
| **Promoted** | Human has promoted outputs to Declared or Authoritative | Human review Decision |
| **Rejected** | Human has rejected outputs | Human review Decision |

Each state transition is a primitive operation with lineage. The transition from Draft to Confirmed is a Decision (§4) with Authority (§4). The transition from Active to Completed is verified by checking that all required outputs specified in the cognitive decomposition have been received.

---


## §A2.4 Execution Layer Interface

The substrate produces Work Specifications. The execution layer consumes them. This boundary is intentional and architecturally significant — the substrate governs what happens; the execution layer determines how it happens.

### Table A2.4.1: Substrate vs. Execution Layer Responsibilities

| Substrate Determines | Execution Layer Determines |
|---|---|
| *What* cognitive work is needed | *How* to configure the model |
| *Who* (which AI actor) handles each component | How to translate delegation scope to prompt instructions |
| *What constraints* apply to each component | How to encode constraints as model configuration |
| *What was produced* — captures outputs as Evidence | How to return outputs to the substrate |
| *Lineage* for the full orchestration chain | Execution metadata (model, tokens, duration, tools) |

The substrate does not encode prompts, model parameters, or tool configurations. These are execution layer concerns. The substrate specifies the governance envelope — cognitive work type, behavioral constraints, evidence requirements, authority scope — and the execution layer translates that envelope into model-native configuration. This separation ensures that model changes, prompt engineering evolution, and tool configuration updates do not require governance architecture changes.

### Execution Layer Contract

The execution layer satisfies a contract that maintains governance integrity:

**Accepts:** A confirmed Work Specification.

**Translates:** Cognitive work type → model configuration. Behavioral constraints → prompt instructions or tool configuration. Input evidence → model context.

**Returns:** Three categories of output:
- **Outputs** — Evidence artifacts produced by the AI actor for each component
- **Execution metadata** — Model used, tokens consumed, execution duration, confidence signals, tools invoked, errors encountered
- **Completion status** — Completed (all components produced outputs), Partial (some components produced outputs), or Failed (execution could not proceed)

### Output Re-Entry

Outputs from the execution layer enter the substrate as Evidence (§4) with truth type Derived (§6). Every AI output carries:

| Field | Value |
|---|---|
| Truth type | Derived — invariant, enforced by B3 (§5) |
| Producer | The AI actor that generated the output |
| Authority reference | The delegation under which the output was produced |
| Work Specification reference | The specification that governed production |
| Component reference | Which cognitive component this output satisfies |
| Confidence signal | The AI actor's self-reported confidence |
| Constraint context | The constraints that were active during production |
| Promotion status | `awaiting_review` at entry |

AI outputs enter with promotion status `awaiting_review`. Promotion to Declared or Authoritative requires a human Decision with Authority and Account (§6). The promotion decision is itself a state transformation with lineage: who reviewed, what they found, whether they accepted or rejected, and on what basis.

---


## §A2.5 Pattern Detection and Learning

### Pattern Detection (Stage A+ Mechanics)

With sufficient interaction history, the substrate's pattern detection engine observes:

- **Work type clusters** — what interactions share characteristics, how frequently each type appears, what groupings emerge
- **Quality patterns by context** — acceptance rates, correction rates, common correction types per work context
- **Cognitive type signals** — retrospective classification of past interactions into AICAR cognitive work types (§A1)

Pattern detection activates at Stage A+ — the point where sufficient evidence exists for pattern recognition. The pattern detection engine is a substrate component, not an AI actor — its outputs are computational observations over accumulated interaction data.

### Table A2.5.1: Pattern Accumulation Minimums

| Metric | Default Minimum | Rationale |
|---|---|---|
| Total interactions | 50+ | Sufficient sample for statistical patterns |
| Unique work contexts | 5+ | Prevents overfitting to a single context |
| Human review decisions | 50+ | Requires human judgment data, not just interaction count |
| Observation period | 30+ days | Captures temporal variation in work patterns |

These are substrate defaults. Each substrate instance may configure higher minimums. Derivatives may set higher thresholds. No instance may set lower minimums than the substrate defaults.

### Patterns as Derived Evidence

Detected patterns are Evidence (§4) with truth type Derived (§6). They are substrate-generated observations, not verified organizational knowledge:

| Field | Value |
|---|---|
| Truth type | Derived |
| Producer | Substrate pattern engine |
| Content | Observed patterns (work type clusters, quality patterns, cognitive type signals) |
| Promotion status | `awaiting_review` |

Patterns enter with promotion status `awaiting_review`. Human validation is required before patterns inform routing decisions. An unvalidated pattern may be surfaced to a human reviewer as a recommendation; it may not alter routing behavior. Once a human validates a pattern (promoting it to Declared), the validated pattern becomes part of the routing evidence base and may influence graduation progression.

### Learning Integration

Pattern detection is the sensor; the learning mechanism (§18, Learn stage) is the adaptive response. Pattern detection feeds the graduation evidence chain — accumulated, validated patterns provide the evidence basis for graduation transitions. The learning mechanism provides double-loop feedback: identifying systematic miscalibration in thresholds, surfacing structural adjustments, and producing reconfiguration directives that flow to the Configure stage.

---


## §A2.6 Routing Proposals and Override Tracking

### Routing Proposals (Stage B Mechanics)

At Stage B (Advise), the substrate generates routing proposals for human confirmation. Each proposal contains:

| Component | Contents |
|---|---|
| **Proposed classification** | Cognitive work types identified, classification confidence, classification basis |
| **Proposed actor** | Selected AI actor, selection basis, alternative actors considered, delegation reference |
| **Proposed constraints** | Routing constraints, behavioral constraints, activated constraints, constraint basis |
| **Proposal status** | Pending, Confirmed, Overridden, or Modified |
| **Human decision** | The Decision (§4) and Account (§4) recording the human's response |

A proposal is a substrate recommendation, not a binding decision. The human may confirm (proposal proceeds as recommended), override (human selects a different actor or classification), or modify (human adjusts specific elements while accepting others). Every human response is a Decision with Account, producing lineage.

### Override Tracking

When a human overrides a substrate proposal, the override is high-value lineage. Override patterns are themselves Evidence — they reveal where the substrate's routing model diverges from human judgment.

Systematic override for a particular work type signals one of two conditions: the routing model needs correction (the substrate is consistently misclassifying or misrouting), or the work type is genuinely unsuitable for substrate routing (the work requires human judgment that the routing model cannot capture). Both conditions are diagnostically valuable. Override patterns feed the learning mechanism and inform graduation decisions — a high override rate for a scope prevents graduation to Stage C for that scope.

### Table A2.6.1: Graduation Transition Criteria

| Transition | Key Evidence Required | Default Threshold |
|---|---|---|
| **A → A+** | Sufficient interactions accumulated; initial patterns observed | 50+ interactions over 30+ days |
| **A+ → B** | Patterns validated by human; multiple AI actors registered; scoped delegations exist | 100+ interactions; validated pattern set; ≥2 registered actors |
| **B → C** | Sustained confirmation rate; execution layer operational; human confidence established | 85%+ confirmation rate over 90+ days |

All transition criteria values are substrate defaults. Derivatives may set higher thresholds — an organization with elevated risk tolerance requirements may require 95%+ confirmation rate for B → C. No derivative may set thresholds lower than the substrate defaults.

Each graduation transition is a Decision (§4) with Authority and Account. The transition records the evidence that justified advancement, the authority that approved it, and a rollback condition. Rollback conditions are expressions evaluated continuously — when a rollback condition is met (e.g., output rejection rate exceeds a configured threshold), the substrate automatically regresses to the prior stage and records the regression as a Decision with Account.

### Graduation Invariants

- **No stage skipping.** Graduation proceeds sequentially: A → A+ → B → C. An organization cannot advance from Stage A to Stage C.
- **Evidence-based advancement.** Each transition requires evidence from the current stage demonstrating readiness for the next.
- **Automatic regression.** When rollback conditions are met, the substrate regresses and records why.
- **Per-scope graduation.** Different scopes — defined by AICAR cognitive type × decision type × domain namespace — graduate independently. An organization may be at Stage C for Citation in Cost decisions and Stage A for Reasoning in Risk decisions.

---


## §A2.7 Behavioral Invariant Compliance

Work Orchestration operates within the ten behavioral invariants (§5). The orchestration engine introduces no new invariants — it composes existing ones. This table demonstrates how each invariant is preserved through orchestration.

### Table A2.7.1: B1–B10 Orchestration Compliance Mapping

| Invariant | Rule | Orchestration Compliance |
|---|---|---|
| **B1** | Work requires Commitment | AI work traces to organizational commitment for the task. The Work Specification links to the parent Work, which links to a Commitment (B1). No AI work executes without an authorizing commitment. |
| **B2** | Commitment requires Capacity | AI actor capacity — delegation scope and model availability — is verified during the ROUTE and SELECT steps. A routing that selects an actor without verified capacity is rejected. |
| **B3** | Evidence requires Truth Type | AI outputs always enter as Derived (§6). The truth type is set at output re-entry and cannot be overridden by the execution layer. Promotion to Declared or Authoritative requires human Decision. |
| **B4** | Decision requires Account | Every routing decision — classification, selection, confirmation — links to an Account recording the decision context. The RoutingDecision record captures rationale, alternatives, and five-dimensional resolution. |
| **B5** | Authority must be traceable | AI delegation traces from the AI actor through the delegation chain to a human authority with root traceability. The SELECT step verifies that every actor assignment has a valid delegation reference. |
| **B6** | Constraint binds primitives | The three constraint types (routing, behavioral, activation) compose and bind AI routing. Constraints travel with the Work Specification to the execution layer. Constraint cascade (§23) operates across instance boundaries. |
| **B7** | All objects flaggable | Work Specifications, AI outputs, routing decisions, and delegation records are all flaggable via the Signal Capture mechanism (§15). The orchestration engine does not create objects outside the signal surface. |
| **B8** | Signals route to authority | Signals on AI outputs route to the authority chain for the delegation under which the output was produced. Escalations from the resolution engine route to the authority chain for the work context. |
| **B9** | IQ resolution creates Decision | Investigative Queries spawned from AI output review — patterns of quality issues, delegation scope questions, routing anomalies — resolve through Decision, producing actionable outcomes. |

The governance floor never drops. From Stage A through Stage C, AI outputs are Derived, delegations trace to human authority, and lineage is maintained. The invariants are protocol requirements from day one, not Stage C requirements relaxed for earlier stages.

---


## §A2.8 SDK Constraints Summary

### Table A2.8.1: §A2 SDK Constraints

| ID | Category | Constraint |
|---|---|---|
| §A2-C1 | **MUST** | Implement the seven-step resolution sequence (CLASSIFY → ACTIVATE → ROUTE → SELECT → COMPOSE → CONFIRM → EMIT). Each step is a primitive operation producing lineage. Steps execute in canonical order. |
| §A2-C2 | **MUST** | Implement the three constraint types (Routing, Behavioral, Activation) as specializations of the Constraint primitive (§4). Constraints compose through intersection to produce the effective constraint set. |
| §A2-C3 | **MUST** | Produce a Work Specification for every orchestrated work request. The specification includes cognitive decomposition, actor assignments, effective constraints, specification metadata, composition strategy, and five-dimensional context. |
| §A2-C4 | **MUST** | Maintain the execution layer boundary: substrate determines WHAT (cognitive work), WHO (actor assignment), and CONSTRAINTS (effective constraint set). The substrate does not encode prompts, model parameters, or tool configurations. |
| §A2-C5 | **MUST** | Enter all AI outputs as Evidence with truth type Derived (§6) and promotion status `awaiting_review`. Promotion to Declared or Authoritative requires a human Decision with Authority and Account. |
| §A2-C6 | **MUST** | Produce a RoutingDecision record for every resolution. The record includes alternatives considered, selected assignments, selection rationale, five-dimensional resolution, computed HITL level, confidence assessment, authority basis, and resulting Work Specification. |
| §A2-C7 | **MUST** | Require human validation before detected patterns inform routing decisions. Patterns are Evidence (Derived) with promotion status `awaiting_review`. Unvalidated patterns may be surfaced as recommendations but may not alter routing behavior. |
| §A2-C8 | **MUST** | Enforce sequential graduation: A → A+ → B → C with no stage skipping. Each transition requires a Decision with Authority, Account, and accumulated Evidence meeting the configured transition criteria. |
| §A2-C9 | **MUST** | Implement automatic regression when rollback conditions are met. Regression produces a Decision with Account recording the trigger. |
| §A2-C10 | **MUST** | Record every constraint conflict resolution as a Decision through the authority chain, producing lineage that captures who resolved the conflict, on what basis, and what the resulting effective constraint set is. |
| §A2-C11 | **MUST** | Implement the Work Specification lifecycle (Draft → Confirmed → Active → Completed → Promoted/Rejected). Each state transition is a primitive operation with lineage. |
| §A2-C12 | **MUST NOT** | Allow AI outputs to enter as Authoritative or Declared without human promotion. This is invariant B3 (§5) applied to orchestration — no architectural bypass exists. |
| §A2-C13 | **MUST NOT** | Allow the substrate to encode prompts, model parameters, or tool configurations. These are execution layer responsibilities. The substrate specifies the governance envelope; the execution layer translates it. |
| §A2-C14 | **MUST NOT** | Allow graduation stage skipping or self-promotion. Graduation transitions require accumulated evidence from the current stage and human authorization for advancement. |
| §A2-C15 | **MUST NOT** | Allow unvalidated patterns to modify routing behavior. Pattern detection outputs are Derived Evidence requiring human promotion before they influence routing decisions. |
| §A2-C16 | **DESIGN SPACE** | Specific graduation transition threshold values. The substrate provides defaults (Table A2.6.1); derivatives may set higher thresholds. No derivative may set thresholds below substrate defaults. |
| §A2-C17 | **DESIGN SPACE** | Pattern accumulation minimums. The substrate provides defaults (Table A2.5.1); instances may configure higher minimums. |
| §A2-C18 | **DESIGN SPACE** | Execution layer implementation. The contract (§A2.4) specifies what the execution layer accepts and returns. How the execution layer translates Work Specifications to model configuration is an implementation choice. |
| §A2-C19 | **DESIGN SPACE** | Model configuration strategy. Which models serve which cognitive work types, how prompts are engineered, and how tools are configured are execution layer decisions outside the substrate's governance boundary. |
| §A2-C20 | **DESIGN SPACE** | Specific routing constraint definitions. The protocol defines the routing constraint structure; which actors are preferred or excluded for which work types is an organizational configuration populated during TMI (§20) and refined operationally. |
| §A2-C21 | **DESIGN SPACE** | Specific behavioral constraint definitions. The protocol defines the behavioral constraint structure; which evidence grounding, attribution, and formatting requirements apply to which contexts is organizational configuration. |
| §A2-C22 | **DESIGN SPACE** | Rollback condition expressions. The protocol requires rollback conditions for every graduation transition; the specific expressions (output rejection rate thresholds, escalation frequency thresholds) are substrate-level configuration. |


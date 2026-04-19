# §A1 Agent Runtime Infrastructure


Agent runtime infrastructure specifies the structural grammar for governing AI actors as participants in organizational work. It introduces no new primitives. Every concept in this section is a composition pattern over the nineteen irreducible primitives defined in §4.

This section belongs to the DLP-World-Model tier (§3). It specifies WHAT a governed AI routing system must be — the cognitive work type taxonomy, the delegation authority model, the five-dimensional routing space, the graduation model, and the conformance requirements. §A2 specifies HOW the DLP substrate implements it.

The architectural parallel: agent runtime infrastructure is to AI actors what container runtimes are to application containers, or what MCP is to tool capabilities. MCP provides a capability runtime — what tools an AI can use. DLP provides a governance runtime — under what authority, within what constraints, with what lineage. The relationship is additive, not competitive.

---


## §A1.1 Motivation: The Agent Governance Gap

When organizations deploy AI systems, routing decisions — which model handles which task — typically live in application code. This creates three structural governance gaps that no amount of operational tooling can close.

**Untraced authority.** No record of who authorized which model to handle which type of work, or why. The delegation from human authority to AI actor is implicit, invisible, and ungoverned.

**Invisible routing logic.** When a model produces an incorrect output, there is no lineage to explain why that model was selected for that task. The routing decision itself has no decision record.

**Ungoverned model transitions.** Swapping models — due to cost, capability, or policy — happens outside any governance framework. The organization cannot answer: "When did we switch? Who decided? What was the impact?"

These are not operational bugs. They are structural gaps — the absence of governance grammar for AI participation in organizational work. The DLP already possesses every primitive needed to close them.

### Table A1.1.1: Agent Governance Gap Resolution

| Governance Question | Primitive | Mechanism |
|---|---|---|
| Which AI actors exist? | Actor (§22) | Actor registry with systemic type |
| What can each AI actor do? | Authority (§4.1) | Scoped delegation to AI actor |
| What limits apply? | Constraint (§4.1) | Constraint sets scoped to work types |
| What work needs to be done? | Work (§4.1) | Work trigger with type classification |
| What did the AI produce? | Evidence (§4.1) | Output tagged with actor, authority, truth type Derived (§6) |
| Why was this model selected? | Decision + Account (§4.1) | Routing decision with rationale |
| What happened as a result? | State Transformation (§4.2) | Full lineage of the orchestration event |

Agent runtime infrastructure does not add new capabilities to the protocol. It defines the composition pattern that makes AI routing a governed, traceable operation using existing primitives.

### §A1.1.1 Collaboration Topology Taxonomy

When multiple agents and humans coordinate work, their interaction follows one of six structural topologies. These topologies are *descriptive* — recognized from emergent patterns in how work is actually organized — not *prescriptive*. The substrate classifies observed collaboration into a topology as an intelligence function; it does not require humans to select a topology upfront.

### Table A1.1.2: Six Collaboration Topologies

| Topology | Structure | Example | Governance Concern |
|---|---|---|---|
| **Winner-Take-All** | Multiple actors produce independent outputs; one is selected | Three models draft a summary; human picks the best | Selection rationale must be captured as Decision with Account |
| **Adversarial** | Actors explicitly challenge each other's outputs | Red-team model critiques blue-team model's analysis | Challenge evidence must be preserved; resolution must be traceable |
| **Cooperative-Asymmetric** | Actors have different roles with unequal authority or capability | Senior model reviews junior model's draft | Authority delegation must reflect the asymmetry; review decisions carry the senior delegation |
| **No-Right-Answer** | Multiple valid perspectives coexist; synthesis required | Domain experts offer conflicting but legitimate interpretations | Reconciliation must preserve all perspectives as Evidence, not discard minority views |
| **Structured Scoring** | Actors score or evaluate against defined criteria | Multiple models rate evidence quality on rubric dimensions | Scoring criteria must be captured as Constraints; aggregate methodology must be traceable |
| **Domain-Owned** | Each actor has exclusive authority within a bounded domain | Legal model handles compliance; finance model handles cost analysis | Domain boundaries map to delegation scopes; cross-domain handoffs are governed transitions |

**Topology as classification tool.** Topologies are a vocabulary for the World Model intelligence layer, not a configuration parameter. The substrate observes how work is actually organized — which actors participate, how outputs relate to each other, who decides — and classifies the observed pattern into a topology. This classification feeds routing confidence (§A2), authority-weighted reconciliation (§A1.8), and QuickFire mode detection (§A1.6, Pattern 7).

---


## §A1.2 Cognitive Work Type Taxonomy (AICAR)

AI cognitive work is not undifferentiated. Different tasks require different capabilities, carry different risks, and warrant different governance. The protocol defines a taxonomy of cognitive work types — the AICAR framework (AI Cognitive Activity Review) — that serves as the primary classification key for routing.

AICAR classifies the cognitive tasks that AI actors perform when generating outputs. The taxonomy is derived from a classification of cognitive tasks that humans perform when reviewing AI outputs, extended to the generation side. This dual origin — generation and review sharing the same taxonomy — connects what the AI does to how humans verify it.

### Table A1.2.1: Cognitive Work Types

| Type | Name | Definition | Characteristic |
|---|---|---|---|
| **A** | Citation | Locating, retrieving, and accurately referencing existing information | Fidelity to source material |
| **B** | Reasoning | Analyzing relationships, applying logic, drawing conclusions from evidence | Chain of inference |
| **C** | Context | Integrating situational awareness, domain knowledge, and environmental factors | Sensitivity to conditions |
| **D** | Communication | Structuring, formatting, and presenting information for an audience | Clarity and appropriateness |

### Table A1.2.2: Type Risk Profiles

| Type | Hallucination Risk | Verification Complexity | Review Method |
|---|---|---|---|
| A (Citation) | High — fabricated references | Mechanical — source exists or does not | Check citations against sources |
| B (Reasoning) | Medium — logical errors | Analytical — follow the chain | Evaluate reasoning steps |
| C (Context) | High — incorrect assumptions | Judgmental — requires domain knowledge | Domain expert validates |
| D (Communication) | Low — style not fact | Mechanical — audience-appropriate check | Review for tone, format, completeness |

### Type Properties

Each cognitive work type carries protocol-level properties that constrain routing and review:

- **Evidence requirements.** Input evidence (what the AI actor receives) and output evidence (what the AI actor must produce). A Citation task receives source material and must produce referenced output. A Reasoning task receives evidence and must produce an inference chain.
- **Fidelity floor.** The minimum fidelity standard for outputs of this type. Citation outputs require exact source correspondence. Communication outputs require audience appropriateness.
- **Risk profile.** Hallucination risk, verification complexity, and appropriate review method — the properties that determine where human review effort concentrates.

### Composite Work Types

Most cognitive work involves multiple types in composition. The protocol expresses this as Work decomposition: a parent Work record with component Work items, each classified by cognitive work type, sequenced, and independently routable.

Each component may route to a different AI actor based on delegation scope, or to the same actor with different constraint sets active. The decomposition itself is a governed operation — a Decision (§4.1) that produces a Work structure with lineage.

### Extension Principle

The four-type taxonomy is a closed base set. Derivatives may specialize these types for their domain — creating sub-types that inherit from and conform to the parent type — but may not modify the base taxonomy. Domain-specific sub-types carry all properties of the parent type plus domain-specific constraints.

---


## §A1.3 AI Actor Delegation Authority Model

### AI Actor as Structural Concept

An AI Actor is an Actor (§22) with systemic type and properties specific to governed AI participation: system classification (model, pipeline, or ensemble), cognitive work type capabilities, registration time, and registering actor. The AI Actor is a structural specification at the DLP-World-Model tier. Any governed AI routing system must register its AI participants as Actors with these properties. Operational metadata — context window, output format, tool access, model identifier and version — is Substrate-level configuration (§A2).

### Delegation Structure

AI actors receive authority through explicit delegation. Delegation-as-composition follows the pattern established in the protocol's composition mechanics (§4.3):

> Delegation = Intent + Authority + Commitment + Constraint + Work + Context

This pattern applies within instances and across instance boundaries in recursive structures (§23). The structural delegation specifies:

- **Holder.** The AI actor receiving the delegation.
- **Source.** The human authority being delegated from.
- **Scope.** Cognitive work types the delegation covers, domain namespace boundaries, work type scope, and constraint scope the actor must observe.
- **Limits.** Confidence floor (below which the actor must escalate), output truth type (always Derived — §6), human promotion requirement, and maximum autonomy level.
- **Governance.** Effective interval, delegating actor, and the Decision record that authorized this delegation.

**Invariant.** AI outputs enter the substrate as Derived evidence (§6). This is a DLP-Core invariant — B3 (§5) enforces epistemic classification on every Evidence instance. Human promotion is required for truth type advancement to Declared or Authoritative. No delegation can override this invariant.

### Table A1.3.1: Autonomy Levels

| Level | Name | AI May | Human Must | Appropriate When |
|---|---|---|---|---|
| 0 | Assist | Draft outputs to staging | Review and promote every output | Initial deployment, high-risk tasks |
| 1 | Recommend | Draft and rank alternatives | Select from alternatives, promote | Established patterns, moderate risk |
| 2 | Act-and-Report | Execute within delegation, report after | Review reports, intervene on exceptions | Low-risk, well-understood tasks |
| 3 | Autonomous-within-Bounds | Execute within delegation, report on exception only | Set bounds, review exceptions | Routine tasks with clear constraints |

**Invariant.** Level 3 does not mean ungoverned. The AI actor still operates within delegated authority, still produces Derived evidence, and still creates lineage. "Autonomous-within-bounds" means human attention is directed to exceptions rather than every output.

**Invariant.** Autonomy level is a property of the delegation, not the actor. The same AI actor may operate at Level 0 for one work type and Level 2 for another, based on the authority delegation for each scope.

### Delegation Lifecycle

Delegations are managed through standard primitive operations, all producing lineage:

| Operation | Primitive Composition | Lineage Captures |
|---|---|---|
| Grant delegation | Authority.delegate() + Decision + Account | Who delegated, why, what scope |
| Modify scope | Authority.modify() + Decision + Account | What changed, why |
| Revoke delegation | Authority.revoke() + Decision + Account | Why revoked, what triggered |
| Escalate autonomy | Authority.modify(autonomyLevel++) + Decision + Account + Evidence | What evidence justified increase |
| Reduce autonomy | Authority.modify(autonomyLevel−−) + Decision + Account + Evidence | What triggered reduction |

Every delegation change is a state transformation (§4.2). The substrate maintains complete lineage of which AI actors could do what, when, and why.

---


## §A1.4 Five-Dimensional Routing Space

### The Resolution Problem

Routing an AI actor to a piece of work is not a one-dimensional lookup. It requires simultaneous resolution across multiple governance dimensions. Each dimension constrains the resolution; the intersection of all dimensions determines the routing outcome — including where the human sits in the loop.

### Table A1.4.1: The Five Dimensions

| Dimension | Source | Values | Governs |
|---|---|---|---|
| **AICAR Cognitive Type** | §A1.2 | Citation, Reasoning, Context, Communication | What KIND of cognitive work is needed |
| **Decision Type** | Decision Type Taxonomy | Cost, Risk, Business (+ Policy/Governance, Data/Reporting cross-cutting) | What DIMENSION of organizational concern is primary |
| **Consequence Level** | §A1.4 below | Reversible, Consequential, Structural | What is at STAKE |
| **Operating Posture** | §A1.4 below | Normal, Degraded, Emergency | What CONDITION the organization is in |
| **Graduation Stage** | §A1.5 | A (Capture), A+ (Observed), B (Advise), C (Orchestrate) | What MATURITY level the organization has demonstrated for this scope |

### HITL Placement as Computed Position

Human-in-the-loop placement is not a fixed design choice. It is a function of the five-dimensional resolution:

> HITL Placement = f(cognitiveType, decisionType, consequenceLevel, operatingPosture, graduationStage)

At one resolution point — Citation + Cost + Reversible + Normal + Stage C — the human may be in exception-only review (Level 3 autonomy). At another — Reasoning + Business + Structural + Degraded + Stage A — the human may be making every routing decision manually (Level 0).

The five dimensions are not independent. They interact: a breach on one dimension changes the resolution on others.

### Consequence Levels

Three levels classify what is at stake in a routing decision:

- **Reversible.** The outcome can be undone without material impact. Routing errors are correctable.
- **Consequential.** The outcome has material impact that is difficult or costly to reverse. Routing errors have organizational cost.
- **Structural.** The outcome changes organizational structure, authority relationships, or governance posture. Routing errors affect the governance system itself.

Emergency is a modifier on Operating Posture (see below), not a fourth consequence level. A Reversible decision under Emergency posture is still reversible — but the emergency context means the decision was made under compressed conditions and requires post-review.

### Table A1.4.2: Operating Posture

| Posture | Characteristics | Routing Impact |
|---|---|---|
| **Normal** | Standard operations; all constraints active; full authority chain | Standard resolution across all dimensions |
| **Degraded** | Reduced capacity — key personnel unavailable, system partial outage | Tightened constraints; lower autonomy ceilings; expanded monitoring |
| **Emergency** | Time-compressed; normal process suspended; expanded temporary authority | Dual behavior: simultaneous relaxation and intensification (§A1.8) |

### Cross-Dimensional Cascade

Breach on one dimension changes routing resolution on all other dimensions. This is the drift detection mechanism at the orchestration level:

- A Cost decision exceeds its threshold → triggers Risk assessment.
- A Risk assessment exceeds tolerance → triggers Business alignment check.
- A Business/purpose conflict emerges → triggers Structural review.
- Any level under time compression → triggers Emergency posture.

The cascade operates at two levels simultaneously. At the **micro level**, an individual decision hits a threshold and escalates through Cost → Risk → Business. Each escalation changes the routing resolution for the in-flight work — different AICAR decomposition may be needed, different autonomy level applies, different human review required. At the **macro level**, aggregate position across decisions reveals deviation from Orientation. A pattern of threshold breaches reveals drift trajectory. The substrate surfaces drift before it becomes crisis.

When a cascade moves work to a new decision type scope, the five-dimensional resolution recomputes at the destination. The new scope brings its own graduation stage, its own AICAR decomposition, its own consequence level assessment. Cascade history travels with the work as lineage — the routing resolution at the destination scope has access to the full chain of escalation decisions.

### Thresholds as Instance-Specific Weights

Thresholds are what differentiate one substrate instance from another. They function as weights that tune the five-dimensional resolution:

- **TMI stage (Configure).** Methodology Translation (§20) extracts organizational thresholds from vintage documents or fresh questions. The human principal reviews and confirms during onboarding.
- **Operational stage (Operate).** Operational experience refines thresholds as the substrate processes real work. The substrate proposes adjustments; the human approves.
- **Learning stage (Learn).** Double-loop feedback identifies systematic threshold miscalibration. The learning mechanism surfaces structural adjustments.

In recursive structures (§23), thresholds cascade through recursion levels following the tighten-only rule: child instances may tighten parent thresholds but never loosen them. A parent EAS instance sets cost thresholds; child PAS instances may only tighten those thresholds, never loosen.

### Constraint Relaxation

The tighten-only rule governs delegation direction (parent → child), not permanence. Loosening follows an evidence-driven process through the authority chain:

1. Evidence accumulates at the constrained instance — operational data surfaces that a constraint is overly tight.
2. Evidence surfaces as Signal (§4.2) to the constraining authority through normal signal routing (B8, §5).
3. The constraining authority makes a Decision with Account based on evidence.
4. Modified delegation flows down — still bounded by the parent's own position.
5. The chain is recursive upward if the parent is itself constrained.

The governance value is not permanent tightening — it is ensuring that every loosening is a governed decision with evidence, authority, and lineage.

---


## §A1.5 Graduation Model

### The Dual-Process Foundation

The graduation model maps to the System 1/System 2 structural parallel established in the control-theoretic foundation (§8):

- **Option A (Capture)** = DLP-Core operating alone — mechanical governance. The substrate watches and records.
- **Option B (Advise)** = DLP-World-Model engaged — environmental awareness surfaces patterns, proposes routing.
- **Option C (Orchestrate)** = Full world model — substrate routes within delegation via prediction.

### Table A1.5.1: Graduation Options

| Option | Name | Substrate Role | Evidence Produced |
|---|---|---|---|
| **A** | Capture | Observer and recorder | Interactions, reviews, human routing decisions as Decision records |
| **A+** | Observed | Observer that learns | Patterns detected from accumulated lineage; nothing changes operationally |
| **B** | Advise | Advisor with lineage | Routing proposals, proposal accuracy, override patterns |
| **C** | Orchestrate | Governor and router | Full orchestration lineage, deviation detection, Work Specifications |

Option A is not a lesser version of Option C. Option A is the foundation that makes Option C possible. Without the lineage accumulated in Option A, Option C has no evidence base for its routing decisions.

### Table A1.5.2: Internal Graduation Stages

| Stage | Name | Routing Logic | Human Involvement | Substrate Captures |
|---|---|---|---|---|
| 0 | Manual | Human selects model for each task | Every routing decision | Human routing decisions as Decision records |
| 1 | Observed | Human routes, substrate records patterns | Every routing decision | Routing patterns, frequency, outcomes |
| 2 | Suggested | Substrate proposes routing, human confirms | Confirmation of each routing | Suggestion accuracy, override patterns |
| 3 | Delegated | Substrate routes within delegation, human reviews outputs | Output review, exception handling | Routing decisions, output quality, escalations |
| 4 | Governed | Substrate routes and monitors, human handles exceptions | Exception-only involvement | Full orchestration lineage, deviation detection |

The mapping between Options and Stages: Option A spans Stages 0–1. Option B spans Stages 1–2. Option C spans Stages 2–4. Per-scope graduation means an organization may be at Stage 3 for citation tasks and Stage 1 for reasoning tasks.

### Graduation Invariants

- **No stage skipping.** Graduation proceeds sequentially. An organization cannot jump from Stage 0 to Stage 4.
- **Evidence-based advancement.** Each transition requires evidence from the current stage demonstrating readiness.
- **Automatic regression.** If rollback conditions are met — output rejection rate exceeds threshold, escalation rate spikes, routing accuracy degrades — the substrate automatically regresses to the prior stage and records why.
- **Per-scope graduation.** Different work types may be at different stages. Graduation is scoped to AICAR type × Decision Type × domain namespace. An organization may be at Stage 3 for Citation + Cost tasks and Stage 1 for Reasoning + Business tasks.
- **Lineage through graduation.** The graduation path itself is captured as a series of state transformations (§4.2). The organization can always answer: "How did we get to this level of AI autonomy for this scope?"
- **Transition is a Decision.** Every stage transition and every Option transition is a Decision (§4.1) with Account and Evidence.

Graduation stage definitions are specified here. Transition mechanics — the evidence thresholds, sample sizes, observation periods, and transition criteria values that determine when a stage transition is warranted — are operational concerns specified in §A2.

### Table A1.5.3: Invariants Across Graduation Stages

| Property | A | A+ | B | C |
|---|---|---|---|---|
| AI outputs are Derived (§6) | Yes | Yes | Yes | Yes |
| Human promotion required for truth type advancement | Yes | Yes | Yes | Yes |
| AI delegation traces to human authority (B5, §5) | Yes | Yes | Yes | Yes |
| Lineage maintained for all AI interactions | Yes | Yes | Yes | Yes |
| AI actor registered with structural properties | Yes | Yes | Yes | Yes |
| Review decisions captured as Decision + Account | Yes | Yes | Yes | Yes |

### Evidence Accumulation Chain

Each stage's evidence is the next stage's foundation:

> Option A lineage (interactions, reviews) → feeds Option A+ pattern detection → validated patterns feed Option B routing proposals → proposal accuracy feeds Option C delegation confidence → orchestration lineage feeds ongoing governance

This chain is itself captured in the substrate. The governance floor never drops — the invariants from Table A1.5.3 apply from Stage 0 through Stage 4.

### Spawning Graduation Stage

When a recursive instance spawns a child (§23), the parent chooses the child's starting graduation stage, up to and including the parent's own stage. The parent's stage is the upper bound. This unifies the delegation model: constraints tighten-only, thresholds tighten-only, graduation stage tighten-only. The parent cannot grant what it has not earned. The spawning decision is a Decision with Account, providing lineage on the choice.

### §A1.5.1 Session-Scoped Micro-Graduation

Within a single work session, the substrate may relax governance constraints progressively as the session accumulates evidence of reliable performance. This within-session trust accumulation operates at a finer grain than the cross-session graduation model above.

Three properties define session-scoped micro-graduation:

**Session-bounded.** Micro-graduation state resets at session end. A session that earned relaxed constraints does not carry those relaxations into the next session. Each session starts at the baseline determined by the cross-session graduation stage.

**Monotonically relaxing within session.** Within a session, constraints only relax — they do not tighten through micro-graduation alone. If the drift circuit breaker fires (§A2, Drift Control), micro-graduation resets to session baseline. This separation is intentional: micro-graduation handles smooth trust accumulation; the circuit breaker handles discontinuous governance events.

**Composable with cross-session graduation.** The starting level for a session comes from the cross-session graduation history. Micro-graduation modulates within the session based on session-specific performance. The effective governance posture at any point in a session is: cross-session baseline + session micro-graduation adjustment − any circuit breaker resets.

Micro-graduation is an optimization, not a requirement. Substrates that do not implement micro-graduation operate at the cross-session graduation level throughout each session. Micro-graduation reduces unnecessary human confirmation friction in sessions where early rounds demonstrate reliable routing, without compromising the cross-session evidence base that drives graduation decisions.

---


## §A1.6 Composition Patterns

Seven patterns express how the agent runtime grammar composes primitives for different routing scenarios. Each pattern is a structural sequence showing the primitive operations that produce governed AI routing.

### Pattern 1: Single-Actor Routing

The simplest pattern — one cognitive work type, one AI actor, one set of constraints.

> Work request arrives → Classify: AICAR Type A (Citation) → Route: AI actor holds delegation for Type A in this domain → Specify: single component, single actor → Execute: actor retrieves and cites → Review: human checks citations against sources → Promotion or rejection (Decision + Account)

### Pattern 2: Multi-Actor Decomposition

Complex work decomposes into components routed to different actors based on cognitive type specialization.

> Work request arrives → Classify: [Type A (Citation), Type B (Reasoning), Type D (Communication)] → Route: Type A to retrieval-optimized actor, Type B to reasoning-optimized actor, Type D to drafting-optimized actor → Specify: three components, sequential (A→B→D), three actors → Execute: each actor handles its component → Compose: outputs assembled into integrated deliverable → Review: human reviews integrated output

### Pattern 3: Constraint-Activated Routing

Context activates additional constraints that change routing resolution.

> Work request arrives with high-risk, regulated-domain context → Classify: [Type A, Type B] → Activate: high-risk constraint set loads — requires Level 0 autonomy → Activate: regulated domain constraint set loads — requires citation grounding → Route: only actors with delegation for regulated + high-risk scope are eligible → Specify: components with elevated constraint sets → Execute: actors operate under tighter constraints → Review: human reviews every output (Level 0)

### Pattern 4: Cross-Dimensional Cascade Routing

Threshold breach mid-flight changes the routing context through cascade.

> Work request classified as Cost decision, Reversible consequence → Route at Stage C, Level 2 autonomy → Threshold breach: amount exceeds cost threshold → CASCADE: Cost → Risk assessment triggered → Reclassify: Cost + Risk, Consequential consequence → Check: Risk routing is at Stage A for this scope → Re-route: Level 0 autonomy (human reviews Risk assessment) → Threshold breach recorded as data point for macro-level drift detection

### Pattern 5: Delegation-as-Composition (Recursive)

Delegation across recursive instance boundaries follows the tighten-only cascade.

> Parent EAS delegates to child EAS → Delegation = Intent (engagement scope) + Authority (scoped to domain) + Commitment (engagement timeline) + Constraint (tightened from parent) + Work (engagement work types) + Context (client, jurisdiction, prior engagements) → Child EAS delegates to AI actor within its scope → AI delegation inherits parent constraints (tighten-only) → Graduation stage is child's own (per-scope) → Threshold values tightened from parent

### Pattern 6: Model Transition

Swapping an AI actor for a work type, with full lineage.

> Current state: AI actor handles Type B for a scope at Stage 3 → Decision: revoke current actor's delegation for Type B in this scope (Account: new actor demonstrates higher accuracy per evaluation; Evidence: benchmark results, output comparison, cost analysis) → Decision: grant new actor's delegation for same scope (Account: delegation scope identical, autonomy level reset to Stage 2; Constraint: observation period before graduation beyond Stage 2) → State transformation captures both decisions with full lineage → Result: new actor handles Type B at Stage 2 — must be re-graduated through observation

### Pattern 7: QuickFire Reconciliation

Rapid, session-scoped collaboration where governance is *different*, not absent. QuickFire mode applies when work requires fast iteration with multiple actors under compressed timelines — brainstorming sessions, rapid prototyping, time-boxed analysis sprints.

QuickFire is distinguished from standard orchestration by three properties:

**Relaxed evidence depth.** Standard orchestration requires full evidence chains (L2-L3 depth per TMI, §20). QuickFire operates at L0 — session-level evidence capture is sufficient. Individual actor outputs within a QuickFire session are captured as Evidence (Derived), but the evidence linkage requirements are coarsened to session granularity rather than per-output granularity.

**Session-scoped authority.** In standard orchestration, delegation chains follow the full hierarchy. In QuickFire, delegation flattens within the session — the session initiator holds authority over all session participants (human and AI) for the duration of the session. This flattening is itself a governed Decision: the session initiator must hold sufficient authority to flatten (B5), and the flattening decision produces lineage.

**Coarse decision capture.** Standard orchestration captures routing decisions at the component level. QuickFire captures at the session level — a single Decision record covers the session's work specification, rather than individual component decisions. This reduces governance overhead during rapid iteration while maintaining the session-level audit trail.

> QuickFire session initiated → Session authority established (Decision + Authority) → Multiple actors produce outputs in rapid succession → Evidence captured at session granularity (L0) → Session concludes → Session output enters standard promotion workflow → Human reviews session-level output (B10, §5) → Promotion or rejection

QuickFire governance is strictly weaker than standard orchestration during the session but converges to standard governance at session boundaries — all QuickFire output enters the standard promotion workflow and requires human review before truth type advancement.

---


## §A1.7 Lineage Requirements

### Table A1.7.1: Minimum Lineage for Orchestrated Work

For any work processed through the agent runtime, the following lineage must be capturable:

| # | Question | Answered By | Primitive |
|---|---|---|---|
| 1 | What work was requested? | Work record | Work (§4.1) |
| 2 | What cognitive tasks were identified? | Classification decision | Decision (§4.1) |
| 3 | What constraints were active? | Constraint evaluation | Constraint (§4.1) |
| 4 | Which AI actors were eligible? | Routing resolution | Decision + Account (§4.1) |
| 5 | Which AI actor was selected, and why? | Selection decision | Decision + Account (§4.1) |
| 6 | Under what authority did the AI act? | Delegation reference | Authority (§4.1) |
| 7 | What did the AI produce? | Output evidence | Evidence, truth type Derived (§6) |
| 8 | What constraints governed the output? | Behavioral constraints | Constraint (§4.1) |
| 9 | Who reviewed the output? | Promotion decision | Decision + Authority (§4.1) |
| 10 | Was the output accepted or rejected? | Promotion status | Decision + Account (§4.1) |

### Complete Lineage Chain

The complete orchestration lineage forms a chain traversable in both directions:

> Work Request → Classification Decision → Constraint Evaluation → Routing Decision → Work Specification → Execution → AI Output (Evidence, Derived) → Human Review (Decision) → Promotion or Rejection (State Transformation)

From output back to request = audit trail. From request forward to output = progress tracking. Every link in this chain is a primitive operation with its own lineage.

### Model Transition Lineage

When an AI actor delegation changes — model swap, capability update, scope change — the lineage captures: the prior delegation, the new delegation, the transition Decision with rationale (Account), the Evidence that prompted the change (performance data, cost analysis, policy change), the impact assessment (affected work types, affected constraints, risk assessment), the transition time, and the authorizing Actor.

The organization can always answer: "When did we switch models for this scope? Who decided? What was the justification? What work was affected?"

---


## §A1.8 Authority-Weighted Reconciliation

When multiple actors (human or AI) produce outputs for the same governed decision, the substrate must reconcile divergent perspectives. The overcompromise problem — where synthesis produces bland consensus that discards valuable dissent — is a known failure mode in multi-agent systems. Authority-weighted reconciliation addresses this by grounding reconciliation weights in the Authority primitive's delegation chain, not in a separate reputation system.

### Algorithm Structure

**Inputs.** Participant set (actors who contributed outputs), domain scope (namespace boundary for the reconciliation), delegation chain per participant (how each participant's authority traces to the domain root through B5).

**Computation.** Weight = f(delegation distance from domain root). An actor with a shorter delegation chain to the domain root carries more weight for decisions within that domain. This is not reputation — it is structural authority proximity. The same actor may carry high weight in their domain of expertise and low weight outside it, because their delegation chains differ.

**Application.** Weighted synthesis, not majority vote. The reconciliation produces a synthesized output that reflects the relative authority weights. Minority perspectives with high domain authority are preserved, not averaged away. Dissenting views from high-authority participants are recorded as Evidence (Declared), not discarded.

### Reconciliation Invariants

- Weight derivation MUST use the Authority primitive's delegation chain (B5). No separate reputation, scoring, or voting system.
- Minority views from participants with domain authority above a configurable threshold MUST be preserved as Evidence in the reconciliation record.
- The reconciliation decision MUST produce a Decision (B4) with Account recording the weighting methodology and individual contributions.
- Reconciliation topology (§A1.1.1) is recorded as metadata — the substrate classifies which collaboration topology the reconciliation followed.

---


## §A1.8.1 Learning Primitive Lifecycle (SCT Formalization)

The Learning primitive (§4.6.2, Tier 3) has a formal lifecycle that governs how organizational insights emerge, are captured, triaged, and either crystallize into other primitives or decay. This lifecycle is the protocol-level specification of what the Cognitive Companion content package implements as Suspended Conceptual Thoughts (SCTs).

### Lifecycle Stages

| Stage | Description | Entry Condition | Exit Transitions |
|---|---|---|---|
| **Emergence** | A cognitive artifact surfaces during work — an insight, pattern, hypothesis, or question that does not yet have a home in the governed record | Human observation during any governed work (signal, IQ, review, or spontaneous capture) | → Capture |
| **Capture** | The emerged artifact is recorded as a Learning primitive instance with initial metadata: source context, semantic fingerprint, and creation timestamp | Actor records the artifact through a capture mechanism | → Triage |
| **Triage** | The captured artifact is assessed for: relevance to existing governed objects, novelty relative to existing knowledge, and actionability | Human or substrate assessment (at sufficient graduation stage) | → Crystallization or → Decay |
| **Crystallization** | The artifact is mature enough to become a governed primitive — it converts to an Intent, Commitment, Decision, or other Tier 1 primitive with full lineage tracing back to the Learning instance | Human Decision (B4) authorizing the conversion | Terminal (the Learning instance is superseded by the crystallized primitive) |
| **Decay** | The artifact has not been acted upon within its configured time-to-live, or has been triaged as not actionable | TTL expiration or explicit triage decision | Terminal (the Learning instance is archived; retained for audit trail) |

### Lifecycle Properties

**Learning is not Evidence.** A Learning primitive captures cognitive overflow — things worth remembering that don't yet fit the Evidence classification. Learning instances may *become* Evidence through crystallization, but they are not Evidence at capture time.

**Decay is not deletion.** Decayed Learning instances are archived, not removed. The full lifecycle is retained for organizational learning analytics: what emerged, what was captured, what crystallized, and what decayed. Decay patterns reveal organizational attention gaps.

**Crystallization preserves lineage.** When a Learning instance crystallizes into a Tier 1 primitive, the resulting primitive carries a `crystallized_from: Reference<Learning>` field linking to the source Learning instance. This is the lineage mechanism that connects organizational insights to the governed actions they produced.

---


## §A1.9 Escalation and Emergency Posture

*Note: This section was §A1.8 in the locked spec. Renumbered to accommodate §A1.8 Authority-Weighted Reconciliation.*

### Escalation Model

When the substrate cannot resolve routing, it escalates. Escalation is a state transformation (§4.2) with lineage — the substrate does not silently fail.

### Table A1.8.1: Escalation Types

| Escalation Type | Trigger | Required Action |
|---|---|---|
| Routing failure | No eligible actor for a required cognitive work type | Human performs the work, or delegation scope is expanded |
| Constraint conflict | Active constraints produce empty possibility space | Human resolves the conflict through the authority chain |
| Delegation gap | No delegation covers the required scope | Human grants delegation, or scope is expanded |
| Confidence below floor | Substrate's routing confidence falls below the delegation threshold | Human confirms the routing decision |

Each escalation record captures: the escalation type, the unresolvable component (cognitive work type), the reason (Account), the authority escalated to (on the governance chain for the work context), and the required action. Escalation routes to authority on the flagged object's governance chain — implementing B8 (§5).

### Emergency Posture Mechanics

Emergency posture is triggered when Operating Posture enters Emergency state — time compression, capacity crisis, or external shock. Emergency exhibits dual behavior.

**Relaxation vector.** Expanded temporary authority for faster routing decisions. Reduced confirmation requirements — human may approve batches rather than individual routing decisions. Simplified constraint evaluation — non-critical constraints may be suspended with lineage. Lower evidence thresholds for routing — act first, validate later.

**Intensification vector.** Heightened signal awareness — signal surfacing timelines compressed. Pattern detection sensitivity increased — lower thresholds for anomaly flagging. Expanded monitoring scope — signals that would normally be routine are flagged during emergency. Mandatory post-review obligation created for every decision made under emergency authority.

**Governance debt.** Every action taken under emergency posture creates mandatory governance debt. This debt is structural — it persists until a post-review Decision resolves it. The debt mechanism is the governance: emergency does not remove governance, it shifts from preventive to detective with mandatory remediation. This is analogous to EDIM deviation measurement (§9): emergency does not disable the conservation laws, it shifts how violations are detected and remediated.

---


## §A1.10 Conformance Requirements

*Note: This section was §A1.9 in the locked spec. Renumbered.*

### Table A1.10.1: Minimum Conformance

An implementation claiming agent runtime conformance MUST satisfy all eight requirements:

| # | Requirement | Rationale |
|---|---|---|
| 1 | Register AI actors as Actor (systemic type) with structural properties | Governance requires knowing which AI participants exist |
| 2 | Express AI routing authority as explicit delegations with scope | B5 (§5) — authority must be traceable |
| 3 | Classify cognitive work using the four-type AICAR taxonomy | Routing requires cognitive type classification |
| 4 | Capture routing decisions with Account (rationale) | B4 (§5) — decisions require verifiable context |
| 5 | Enter all AI outputs as Derived evidence | B3 (§5) — evidence requires truth type; AI outputs are always Derived (§6) |
| 6 | Require human promotion for truth type advancement | Conservation of epistemic status (§9) — proof quality is intrinsic |
| 7 | Maintain orchestration lineage per §A1.7 | Audit trail must be reconstructable |
| 8 | Support escalation per §A1.8 | Routing failures must be visible, not silent |

### Table A1.10.2: Extended Conformance

Implementations DESIGN SPACE — may additionally satisfy these requirements for extended conformance:

| # | Capability | Reference |
|---|---|---|
| 1 | Full graduation path (A→A+→B→C) | §A1.5 |
| 2 | Multi-actor decomposition | §A1.6, Pattern 2 |
| 3 | Constraint-activated routing | §A1.6, Pattern 3 |
| 4 | Model transition lineage | §A1.7 |
| 5 | Automatic stage regression | §A1.5, graduation invariants |
| 6 | Five-dimensional routing resolution | §A1.4 |
| 7 | Cross-instance delegation (recursive) | §A1.6, Pattern 5 |

### Non-Conformance

The following are explicitly non-conformant. An implementation exhibiting any of these behaviors MUST NOT claim agent runtime conformance:

- AI actors operating without registered delegation.
- Routing decisions without lineage.
- AI outputs entering as Authoritative or Declared without human promotion.
- Model transitions without decision records.
- Constraint bypass for AI routing — even in Emergency posture. Suspension with lineage is not bypass; suspension records what was suspended, why, and under what authority. Bypass leaves no trace.

### SDK Constraints Summary

**MUST.** Register AI actors as systemic Actors with structural properties. Express delegation with scope, limits, and authority chain. Classify cognitive work using the AICAR four-type taxonomy. Enter AI outputs as Derived evidence. Require human promotion for truth type advancement. Capture routing decisions with Account. Maintain the ten-question lineage chain (Table A1.7.1). Record escalations as state transformations. Record emergency actions as governance debt.

**MUST NOT.** Create new primitive types for AI governance — all concepts compose over the nineteen primitives (§4). Allow AI outputs to enter as Authoritative or Declared. Allow constrained instances to self-loosen thresholds. Allow graduation stage skipping. Allow constraint bypass without lineage (suspension with lineage is permitted).

**DESIGN SPACE.** Operational metadata for AI actors (model identifier, context window, tool access). Graduation transition mechanics (evidence thresholds, observation periods, sample sizes). Escalation routing specifics beyond authority chain requirement. Constraint evaluation ordering and optimization. Work Specification structure and execution layer interface. Emergency posture trigger criteria and duration. How evidence surfaces for constraint relaxation review. Domain-specific AICAR sub-type definitions.


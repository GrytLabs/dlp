# §14 Human Capture Mechanisms

The substrate provides structured capture mechanisms; humans provide the irreplaceable judgment that those mechanisms capture. Capture is not a feature of the governance substrate — it is the primary interface between human cognition and the governance record. Without capture, human observation remains tacit: influential, unrecorded, and invisible to lineage.

The core architectural claim: **humans are the sensors for tacit knowledge.** The substrate cannot detect that a decision feels wrong, that an edge case was silently accepted, or that a thread of reasoning was abandoned without resolution. Only humans detect these states. Capture mechanisms convert those detections into structured governance data.

Two first-order mechanisms serve two distinct cognitive events. Signal Capture (§15) surfaces problems with governed objects — "this thing is wrong." IQ Capture (§16) surfaces cognitive emergence during governance operation — "something emerged that needs a home." §14 provides the unifying framework: the shared principles, the capture taxonomy, the invariant enforcement, the truth type assignments, and the second-order lineage extensions that span both mechanisms.

### §14.1 The Capture Principle

Two capture mechanisms address two orthogonal cognitive events within a single architectural framework.

**Signal Capture** (§15) is object-anchored, problem-oriented, and reactive. A signal attaches to a specific governed object and asserts: something about this object requires attention. Signals route to the flagged object's authority chain (B8, §5).

**IQ Capture** (§16) is context-anchored or free-floating, emergence-oriented, and generative. An IQ captures cognitive overflow — an idea, question, pattern, or gap that surfaced during governance work but does not anchor to any single object. IQs route to an investigator for resolution.

#### Table 14.1.1: Capture Mechanism Comparison

| Dimension | Signal Capture (§15) | IQ Capture (§16) |
|---|---|---|
| **Anchoring** | Object-attached — every signal targets a specific governed object | Context-attached or free-floating — IQ may reference origin context or stand alone |
| **Orientation** | Problem — "this is wrong" | Emergence — "this surfaced" |
| **Trigger** | Reactive — responding to an observed concern | Generative — capturing a cognitive event during work |
| **Routing** | Authority chain of the flagged object (B8) | Investigator assignment (self, domain expert, or object authority) |
| **Resolution** | Signal lifecycle: open → processing → resolved | IQ lifecycle: open → investigating → resolved, with promotion to Work, Decision, or Intent |
| **Truth type of capture act** | Authoritative — human observation is a fact of record | Authoritative — human articulation of emergence is a fact of record |

Both mechanisms share five design principles:

1. **Zero friction.** Minimal steps to complete capture. A signal requires one selection (signal type); an IQ requires one entry (observation text). Everything else is optional.
2. **Always available.** Signal Capture is present on every governed object (B7, §5). IQ Capture is globally accessible from any point in the substrate.
3. **Structurally typed.** Signal types and IQ intent types enable automated routing. Categories map to primitives and resolution paths.
4. **Optionally deep.** Rich metadata — free text, context snapshots, priority, linked objects — is available but never required. Capture must succeed at minimum effort.
5. **Context preserved.** A snapshot of system state at capture time is recorded with every signal and IQ. This snapshot is immutable — it preserves what the human observed at the moment of capture.

### §14.2 Capture Taxonomy

Capture operates at three architectural layers. The first two are first-order: they capture concerns about governed objects and cognitive emergence. The third is second-order: it captures concerns about the capture process itself.

#### Table 14.2.1: Capture Layer Architecture

| Layer | What It Captures | Mechanism | Section |
|---|---|---|---|
| **First-order (Object)** | Concerns about governed objects — problems, mismatches, inconsistencies | Signal Capture | §15 |
| **First-order (Emergence)** | Cognitive overflow during governance operation — ideas, questions, patterns, gaps | IQ Capture | §16 |
| **Second-order (Meta-capture)** | Concerns about the capture process itself — reservations, branch awareness, confidence gaps | Extensions to Signal Capture and IQ Capture | §14.6 |

First-order capture records what humans observe about the governed world. Second-order capture records what humans observe about their own observation process — the reservations they chose not to raise, the threads they chose not to pursue, and the confidence gaps between what they stated and what they felt.

#### Table 14.2.2: Protocol Layer Placement

| Protocol Layer | Capture Content | Example |
|---|---|---|
| **DLP-World-Model** | The capture requirement — human judgment must enter the substrate as structured data | §8 derives the requirement from requisite variety: the organizational model needs human-sensed input that no machine sensor provides |
| **DLP Substrate** | Capture mechanisms, routing logic, lifecycle management, resolution paths | Signal types, IQ intent types, authority-chain routing, IQ promotion to Work/Decision/Intent |
| **Licensee Configuration** | UI patterns, keyboard shortcuts, notification preferences, taxonomy extensions | Button placement, modal flow, signal type additions beyond the required five, IQ intent type extensions |

### §14.3 Invariant Enforcement

Three behavioral invariants (§5) govern capture. Together, they guarantee that problems can always be raised, always reach authority, and always produce governance action when resolved.

#### Table 14.3.1: Capture Invariant Enforcement

| Invariant | Rule | Capture Mechanism | Enforcement |
|---|---|---|---|
| **B7** | All objects flaggable | Signal Capture (§15) | Every primitive class MUST have a signal attachment surface. The flaggable surface covers all 19 primitives across all 5 tiers. |
| **B8** | Signals route to authority | Signal Capture (§15) | Every signal MUST route to an authority on the governance chain for the flagged object. Algedonic bypass permits routing to any depth in the delegation hierarchy — a signal may reach the root authority directly. |
| **B9** | IQ resolution creates Decision | IQ Capture (§16) | Every resolved IQ MUST produce a resolution Decision. The resolution act is itself a governance decision with a traceable `resolution_decision_id`. |

**B7 primitive enumeration.** B7 applies to all 19 primitives across all 5 tiers (§4.5). The flaggable object type enumeration MUST include: Intent, Commitment, Capacity, Work, Evidence, Decision, Authority, Account, Constraint, Identifier, Entity, Context, Namespace, Orientation, Learning, Activation, Interpretation, Environment Interface, Cycle.

**B8 routing scope.** B8 validates that the signal's routing target is somewhere on the flagged object's authority chain — the immediate governing authority or any ancestor at any depth in the delegation hierarchy. This implements the algedonic channel: emergency signals bypass intermediate hierarchy to reach whoever has the power to act. The violation condition is routing to an authority entirely outside the governance chain, not routing to a higher-level authority on the correct chain.

**B9 resolution constraint.** B9 ensures that emergence converts to organizational action. An IQ that reaches "resolved" status without producing a Decision is a structural violation — the organization captured an insight but failed to act on it. The resolution Decision records what the organization decided to do (or decided not to do) about the captured emergence.

### §14.4 Capture Truth Types

All capture acts produce Authoritative truth (§6). A human observation is a fact about what the human observed — not an interpretation, not a claim, not a derived conclusion. The capture text (signal description, IQ observation) is immutable once committed. Subsequent annotations create new records; they do not modify the original capture.

#### Table 14.4.1: Capture Truth Type Assignment

| Capture Element | Truth Type | Rationale |
|---|---|---|
| Signal creation event | **Authoritative** | The act of raising a signal is a fact: this human observed this concern at this time about this object |
| Signal context snapshot | **Authoritative** | System state at capture time is a fact: these were the object's properties when the signal was raised |
| IQ capture text | **Authoritative** | The human's articulation of cognitive emergence is a fact: this is what surfaced in their thinking |
| IQ resolution | **Declared** | Resolution is a human assertion requiring the authority of the resolver — it represents best current judgment, not immutable canonical fact |
| IQ promotion artifacts | **Depends on target primitive** | When an IQ promotes to Work, Decision, or Intent, the promoted artifact carries the truth type rules of its target primitive class (§6) |

The Authoritative classification of capture acts has a structural consequence: capture records cannot be modified, only superseded. A human who changes their mind about a signal does not edit the original — they raise a new signal or resolve the existing one. The original observation remains in the governance record as a fact about what was observed, even if the concern was later dismissed.

### §14.5 Cross-Mechanism Interactions

Signal Capture and IQ Capture are parallel mechanisms, not subordinate. Neither contains the other. They interact at four defined integration points where the output of one mechanism becomes input to the other.

#### Table 14.5.1: Cross-Mechanism Integration Points

| Trigger | From | To | What Happens |
|---|---|---|---|
| **Signal spawns IQ** | Signal Capture (§15) | IQ Capture (§16) | Signal processing reveals a deeper question requiring investigation. Signal remains open with status "processing" until the IQ resolves. `signal.spawned_iq_id` links to the created IQ. |
| **IQ spawns signal** | IQ Capture (§16) | Signal Capture (§15) | Investigation reveals a problem with a specific governed object. A new signal is created with `origin_type = "iq"` and origin tracing to the parent IQ. |
| **IQ from claim** | Claims Graduation (§6.5) | IQ Capture (§16) | Claim adjudication raises a question requiring investigation. IQ created with `origin_type = "claim"` and link to the originating claim. |
| **IQ from workflow** | Workflow execution | IQ Capture (§16) | A workflow step triggers a question or surfaces a gap. IQ created with `origin_type = "workflow"` and link to the originating workflow context. |

**Bidirectional traceability.** Every cross-mechanism interaction creates a link in both directions. A signal that spawned an IQ carries `spawned_iq_id`; the IQ carries `origin_type = "signal"` and `origin_signal_id`. An IQ that spawned a signal carries the spawned signal reference; the signal carries `origin_type = "iq"` and `origin_iq_id`. The lineage chain is traversable in either direction.

### §14.6 Second-Order Lineage

First-order capture mechanisms capture concerns about governed objects (Signal) and cognitive emergence (IQ). Neither captures concerns about the capture process itself. Second-order lineage closes this gap.

**Capture mechanisms must capture the act of capturing.** The protocol records what was externalized. Second-order lineage extends this to capture what was *almost* externalized — the reservations that were swallowed, the threads that were abandoned, and the confidence gaps between what was stated and what was felt. This is the dark matter of decision-making: present, influential, and invisible to first-order lineage.

Three extensions across two mechanisms:

#### Table 14.6.1: Second-Order Lineage Extensions

| Extension | Mechanism Extended | What It Captures | Trigger | Blocking? |
|---|---|---|---|---|
| Reservation Signal | Signal Capture (§15) | Held objections on accepted decisions | Decision point (human-initiated) | NON-BLOCKING |
| Branch Inventory IQ | IQ Capture (§16) | Thread awareness across parallel thinking | Session boundary or complex input | NON-BLOCKING |
| Confidence Probe IQ | IQ Capture (§16) | Gap between stated and felt confidence | Behavioral detection (agent-initiated, Assistive posture) | NON-BLOCKING |

**All second-order lineage extensions are NON-BLOCKING.** They annotate and enrich the lineage record without impeding governance flow. No second-order capture act prevents, delays, or gates any first-order governance operation.

#### §14.6.1 Reservation Signal

Extends Signal Capture (§15) with signal type `reservation`. A reservation signal captures the state: "I accept this decision but hold objections I am choosing not to raise now." This is an implicit decision to defer refinement — without capture, it becomes invisible to the lineage.

Reservations attach to Decision, Commitment, or Claim objects at the point of acceptance. They do not challenge the accepted outcome. They record that the accepting party holds concerns they are choosing not to pursue at this time.

##### Table 14.6.2: Reservation Nature Taxonomy

| Nature | Description | Example |
|---|---|---|
| `framing` | The way it is expressed does not match the mental model | "This framing is off but good enough for now" |
| `terminology` | Words or names do not fit | "I will refine this terminology later" |
| `completeness` | Something is missing | "Edge cases exist that I am not raising" |
| `edge_case` | Unhandled scenarios exist | "This breaks for multi-entity structures" |
| `confidence` | Stated confidence does not match felt confidence | "I said 90% but feel 70%" |
| `timing` | Acceptable now but requires future revisit | "Right now yes, but this will not hold at scale" |
| `other` | Catch-all for reservations outside the above categories | "Something feels off but I cannot categorize it" |

The `other` nature serves as an escape valve. High frequency of `other` reservations indicates taxonomy gaps.

##### Table 14.6.3: Reservation Surface Triggers

| Trigger Type | Mechanism | Example | Priority |
|---|---|---|---|
| `condition` | Surface when the reserved object reaches a specified state | `object.status = 'final'` | High / Medium / Low |
| `timestamp` | Surface by a specified date | `2026-06-01` | High / Medium / Low |
| `event` | Surface when a specified event occurs | `decision_referenced` | High / Medium / Low |

Each trigger carries a priority level governing urgency when the trigger fires.

**Reservation routing rules.** Reservation signals follow routing rules distinct from standard signals:

- Reservation signals MUST NOT block the decision. The reserved object proceeds as accepted.
- Reservation signals MUST attach to the reserved object as metadata.
- Reservation signals MUST queue for the flagger's future review via the configured surface trigger.
- Reservation signals MAY route to the authority chain if the reservation nature involves compliance or authority concerns.

**Behavioral integration.** When a reserved object is later consumed by another governance operation, the reservation surfaces to the consuming context:

- **Referenced** by another decision — the reservation is surfaced to the decision-maker referencing the reserved object.
- **Marked final** — reservation resolution or explicit carry-forward is required before finalization.
- **Used as evidence** — the reservation flags epistemic uncertainty to the consuming context.

#### §14.6.2 Branch Inventory IQ

Extends IQ Capture (§16) with intent type `branch_inventory`. Human cognition branches constantly. In a single session, multiple threads emerge — some are pursued, others parked, others dropped. Linear conversation interfaces force serialization of parallel thought, creating loss. Branch inventory captures thread awareness at session and decision boundaries.

**Thread disposition taxonomy.** Every detected thread receives a disposition:

| Disposition | Meaning |
|---|---|
| `pursued` | Thread was followed in this session |
| `parked` | Acknowledged, deferred intentionally |
| `dropped` | Not pursued — may contain lost signal |
| `merged` | Combined with another thread |
| `promoted` | Became primary focus |

**Trigger points.** Branch inventories are prompted at three boundary types:

| Trigger | Mechanism |
|---|---|
| Session end | Handoff prompt asks for branch inventory |
| Complex prompt detected | Agent surfaces in Assistive posture: "I detect multiple threads" |
| Decision milestone | Checkpoint asks: "What threads emerged?" |

**Capture method classification.** Branch inventories record how they were generated:

- **Explicit** — Human volunteers the inventory without prompting.
- **Prompted** — System asks at a boundary; human responds.
- **Detected** — Agent identifies threads and proposes inventory for human confirmation.

The capture method is recorded because it carries epistemic weight: an explicit inventory reflects deliberate reflection; a detected inventory reflects agent pattern-matching confirmed by human review.

#### §14.6.3 Confidence Probe IQ

Extends IQ Capture (§16) with intent type `confidence_probe`. Confidence probes are agent-initiated capture acts operating under Assistive posture (§A1–§A2): the agent detects a behavioral signal suggesting low-confidence acceptance and prompts the human. The human decides whether and how to respond.

**Detection signals.** The agent monitors for behavioral patterns that may indicate unexpressed governance-relevant information:

| Signal | Pattern | Interpretation |
|---|---|---|
| Short acknowledgment | "ok," "sure," "that works" | Possible non-engagement with the substance |
| Quick topic change | Less than one response before moving on | Possible parking without resolution |
| Circling without landing | Questions probe around a topic without concluding | Possible uncertainty about the conclusion |
| Compound questions | Multiple questions in one prompt | Layered thinking that may contain unresolved threads |

**Probe flow.** The agent surfaces a lightweight question:

> "You moved past [X] quickly. Is that:"
> - **Settled** — confident, no concerns
> - **Parked** — valid but deferring for now
> - **Provisional** — accepting with known uncertainty
> - **Needs revisit** — flag for future attention

The probe is opt-in. The human may dismiss without responding. Dismissal is itself recorded as a disposition — no response is not no data.

**Confidence adjustment.** When a probe receives a response, the system records:

- **Before** — confidence level at time of original capture or acceptance.
- **After** — confidence level after probe resolution.
- **Reason** — what changed (e.g., "explicit confirmation," "realized concern was addressed elsewhere," "identified a gap I had not articulated").

**AI posture constraint.** Confidence probes operate under Assistive posture exclusively. The agent observes behavioral patterns that may indicate unexpressed governance-relevant information. The agent prompts for clarification. The agent cannot record a disposition on the human's behalf. The human remains the sensor; the agent amplifies the sensor's sensitivity. No autonomy escalation is permitted — the agent detects and prompts; the human decides.

#### §14.6.4 Dissent as Composition Pattern

With second-order lineage extensions, dissent is fully expressible as a composition pattern over existing capture mechanisms. No additional primitive or mechanism type is required.

| Dissent Expression | Capture Mechanism |
|---|---|
| Explicit objection to a governed object | Standard Signal (§15) |
| Held objection on an accepted decision | Reservation Signal (§14.6.1) |
| Unexpressed concern detected behaviorally | Confidence Probe IQ (§14.6.3) |
| Alternative thread that was not pursued | Branch Inventory IQ (§14.6.2) |
| Formal challenge to a claim or decision | IQ Capture with `challenge` intent (§16) |

The organizational silence problem is not that the protocol lacks a dissent primitive. First-order capture mechanisms capture expressed dissent. Second-order lineage closes the gap by capturing the decision not to dissent — which is itself a governable act.

#### §14.6.5 Second-Order Operational Rules

The following operational rules are scoped to second-order lineage behavior within Part V. They supplement — but do not modify — the behavioral invariants defined in §5.

**Reservation Signal rules:**
1. Reservation signals MUST NOT block object progression.
2. Reservation signals MUST attach as metadata to the accepted object.
3. Reserved objects surfacing on reference MUST show the reservation to the referencer.
4. Final status MUST require reservation resolution or explicit carry-forward.

**Branch Inventory rules:**
5. Branch inventories MUST record all detected threads, not only those pursued.
6. Dropped threads MUST have a disposition recorded, even if the disposition is "unknown."
7. Branch inventories MUST NOT block session transitions.

**Confidence Probe rules:**
8. Confidence probes MUST be opt-in — the human may dismiss without response.
9. Probe responses MAY adjust recorded confidence scores on the probed object.
10. Probes MUST NOT create new decisions — they annotate existing governance records only.

## Scope

Signal and IQ Capture scope is limited to the 19 primitives across 5 tiers (§4.5). Capture does not apply to Validated Projections or derived views. Second-order lineage extensions (§14.6) are non-blocking and do not gate first-order operations.

## Locked Design Positions

**Locked positions:**

1. Capture is the primary interface between human cognition and the governance record.
2. All capture acts produce Authoritative truth (signal creation, IQ articulation) or Declared truth (resolution).
3. Both Signal and IQ Capture mechanisms are required; neither is subordinate to the other.
4. All three behavioral invariants (B7, B8, B9) are structural requirements, not recommendations.
5. Second-order lineage extensions are NON-BLOCKING to first-order governance operations.
6. Capture text is immutable once committed; changes create new versions, not overwrites.
7. Every resolved IQ MUST produce a resolution Decision (B9 enforcement).

## Implementation Requirements

**SDK MUST constraints:**

- MUST enforce B7: every primitive class across all 19 types and 5 tiers has a signal attachment surface.
- MUST enforce B8: every signal routes to an authority on the governance chain for the flagged object; algedonic bypass is available to any depth in the delegation hierarchy.
- MUST enforce B9: every resolved IQ produces a Decision primitive with a traceable `resolution_decision_id`.
- MUST preserve capture text immutability: original capture is write-once; modifications create versioned entries.
- MUST record context snapshots with every signal and IQ; snapshots are immutable from capture.
- MUST assign Authoritative truth type to capture acts; resolution uses Declared truth type.
- MUST implement all second-order lineage extensions (Reservation Signal, Branch Inventory IQ, Confidence Probe IQ) as non-blocking annotations.
- MUST support bidirectional traceability for cross-mechanism interactions (signal↔IQ links are navigable in both directions).

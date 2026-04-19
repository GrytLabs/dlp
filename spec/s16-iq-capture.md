# §16 IQ Capture

IQ Capture is the mechanism by which humans capture cognitive emergence during substrate operation. When working in the governance substrate, ideas branch, questions surface, patterns appear, and gaps become visible. These cognitive events do not attach to any single governed object — they are the overflow of active sense-making. IQ Capture provides a structured landing place for this overflow, converting tacit cognitive events into structured, investigable governance data.

IQ Capture is an extension point, not a primitive. The DLP-Core protocol defines the emergence capture requirement through behavioral invariant B9 (§5): every resolved Investigative Query must produce a Decision. The DLP substrate provides the implementation — intent types, origin types, routing logic, lifecycle management, and promotion paths. IQ is not one of the 19 primitives (§4); it is an operational mechanism that operates on primitives through promotion: a resolved IQ can produce Work, Decision, or Intent primitives.

§14 provides the parent capture framework. This section specifies IQ Capture as the second of two concrete capture mechanisms. Signal Capture (§15) specifies the first. IQ Capture serves a single architectural purpose: preventing organizational amnesia. Ideas, questions, patterns, and gaps that surface during governance operation are captured rather than lost. IQ is the exhaust valve for cognitive overflow — not an add-on feature, but part of the operating experience.

### §16.1 IQ Capture Definition

#### Table 16.1.1: IQ Capture Boundary

| IQ Capture IS | IQ Capture IS NOT |
|---|---|
| Emergence-oriented — "something surfaced that needs a home" | A flag that something is wrong — use Signal Capture (§15) for object-level problems |
| Generative — creating new investigative threads from cognitive events | A complaint mechanism or support ticket |
| Context-anchored or free-floating — may reference origin context or stand alone | Object-anchored — signals require a specific flagged object; IQs do not |
| Optionally attached to any governed object | A bug report — defects in governed objects are Signal territory |
| Routed to an investigator (self, domain expert, or object authority) | Routed to the flagged object's authority chain — that is Signal routing (B8) |

**Distinction from Signal Capture.** Signal Capture (§15) flags problems with existing objects — "THIS thing is wrong." IQ Capture surfaces cognitive overflow — "something emerged that needs a home." The mechanisms are parallel, not subordinate (§14.5). A signal is reactive and object-attached. An IQ is generative and context-attached or free-floating.

Six design principles govern IQ Capture:

1. **Zero friction.** One entry (observation text) to complete capture. Intent type selection and context are available but never required for submission.
2. **Always available.** IQ Capture is globally accessible from any point in the substrate. Every screen MUST include a global capture entry point.
3. **Structurally typed.** Eight intent types classify the cognitive event and inform routing. Categories map to resolution paths.
4. **Optionally deep.** Priority, assignment, linked objects, and rich context are available but optional. Capture MUST succeed at minimum effort.
5. **Context preserved.** A snapshot of the user's working context at capture time is recorded with every IQ. This snapshot is immutable — it preserves what the human was working on at the moment of capture.
6. **Investigator-routed.** IQs route to an investigator — self-assignment by default for personal inquiry, domain expert or object authority for validation and challenge types.

### §16.2 Intent Types

Eight intent types classify the cognitive event that triggered the capture. The first six are first-order intent types: they capture cognitive emergence about the governed world. The last two are second-order intent types (§14.6): they capture concerns about the capture process itself. All eight use the same IQ lifecycle.

#### Table 16.2.1: Intent Type Definitions

| Intent Type | Description | Typical Prompt | Eventual Resolution |
|---|---|---|---|
| `explore` | Curiosity, hypothesis generation | "I wonder if..." | Research → Claim or dismissal |
| `validate` | Verification need, doubt about a current state | "Is this actually true?" | Confirm or refute → Claim |
| `disambiguate` | Semantic confusion, unclear meaning | "What does this mean exactly?" | Clarification → Definition |
| `fill_gap` | Missing information detected | "We don't know X yet" | Research → Evidence |
| `challenge` | Active dispute, suspected error | "I suspect this is wrong" | Investigation → Restatement |
| `connect` | Pattern recognition, relationship sensed | "This relates to something else" | Link discovery or dismissal |
| `branch_inventory` | Multiple threads emerged during a session or decision boundary | Session boundary, complex input | Thread triage: pursued, parked, dropped, merged, or promoted |
| `confidence_probe` | Gap between stated and felt confidence | Behavioral trigger detection | Disposition clarity: settled, parked, provisional, or needs revisit |

**Branch inventory specifics.** Branch inventory IQs record all detected threads from a session or decision boundary. Each thread receives a disposition: `pursued`, `parked`, `dropped`, `merged`, or `promoted`. The capture method records how the inventory was generated: `explicit` (human volunteers), `prompted` (system asks at boundary), or `detected` (agent identifies and proposes for human confirmation). Branch inventories MUST record all detected threads, not only those pursued. Dropped threads MUST have a disposition recorded, even if the disposition is "unknown."

**Confidence probe specifics.** Confidence probe IQs are agent-initiated under Assistive posture — the agent detects behavioral signals (short acknowledgments, quick topic changes, circling without landing, compound questions) and prompts the human. The probe is opt-in: the human may dismiss without responding. Dismissal is itself recorded as a disposition — no response is not no data. When a response is received, a confidence adjustment may be recorded (before/after confidence levels with reason). The agent cannot record a disposition on the human's behalf. The human remains the sensor; the agent amplifies the sensor's sensitivity. No autonomy escalation is permitted.

### §16.3 Origin Types

Four origin types classify how an IQ was created. Each origin type carries different context and determines default routing.

#### Table 16.3.1: Origin Type Definitions

| Origin Type | Source | Default Routing | Context Captured |
|---|---|---|---|
| `claim` | Spawned from claim adjudication (§6.5) — a question surfaces during claim review | Route to claim reviewer | Claim identity, claim state at spawn time, evidence references |
| `signal` | Spawned from signal processing (§15) — deeper investigation required | Route to signal's authority holder | Signal identity, signal type, flagged object identity and state |
| `capture` | Direct human capture — global or contextual entry point | Route to capturer (self-assign default) | Current work context: origin URL, object type and identity if contextual, session identity |
| `workflow` | System-generated from workflow trigger — a workflow step surfaces a gap | Route per workflow routing rules | Workflow identity, trigger condition, workflow state at trigger time |

#### Table 16.3.2: Routing Defaults by Intent Type

| Intent Type | Default Assignment | Rationale |
|---|---|---|
| `explore` | Self | Personal curiosity, low urgency |
| `validate` | Object authority | Verification requires authoritative answer |
| `disambiguate` | Domain expert pool | Semantic questions need expertise |
| `fill_gap` | Self or research function | Information gathering |
| `challenge` | Object authority | Disputes need authoritative review |
| `connect` | Self | Pattern connection is personal insight |
| `branch_inventory` | Self | Session-boundary reflection is personal |
| `confidence_probe` | Self | Agent-initiated but human-decided disposition |

Origin type determines the initial routing target. Intent type refines assignment within that routing context. When origin type and intent type routing defaults conflict, origin type takes precedence for the routing target; intent type informs the assignment method within that target's scope.

### §16.4 IQ Schema

The IQ schema defines field requirements at the architectural level. Substrate implementations MUST provide these fields; they MAY extend with additional fields.

#### Table 16.4.1: IQ Schema — Required Fields

| Field | Type | Immutable | Description |
|---|---|---|---|
| `iq_id` | Identifier | From creation | Unique IQ identifier |
| `capture_text` | Text | From creation | The human's articulation of the cognitive event — IMMUTABLE; original preserved exactly as captured |
| `intent_type` | Enumeration (8 types) | From creation | Cognitive event classification — one of the eight intent types (§16.2) |
| `origin_type` | Enumeration (4 types) | From creation | Creation source — one of the four origin types (§16.3) |
| `raised_by` | Reference → Entity | From creation | The human who captured the IQ |
| `raised_at` | Timestamp | From creation | When the IQ was captured |
| `status` | Enumeration (7 states) | No | Current lifecycle state — see §16.5 |
| `capture_context` | Object | From creation | Origin context at capture time — IMMUTABLE (see Table 16.4.3) |

#### Table 16.4.2: IQ Schema — Optional Fields

| Field | Type | Immutable | Description |
|---|---|---|---|
| `assigned_to` | Reference → Entity | No | Current investigator — updated on assignment and reassignment |
| `resolution` | Object | From resolution | Resolution record: summary, resolver identity, timestamp, linked artifacts, `resolution_decision_id` |
| `promoted_to` | Array of References | From promotion | Work, Decision, or Intent primitives created by promotion — each entry records `target_type` and `target_id` |
| `superseded_by` | Reference → IQ | From supersession | The IQ that absorbed or replaced this one |
| `parent_signal_id` | Reference → Signal | From creation | Source signal identity, when `origin_type = "signal"` |
| `parent_claim_id` | Reference → Claim | From creation | Source claim identity, when `origin_type = "claim"` |
| `blocking_decision_id` | Reference → Decision | From creation | Decision this IQ blocks, when created from a decision context (§16.6) |
| `capture_text_versions` | Array of Objects | Append-only | Edit history: each entry records `version_text`, `edited_by`, `edited_at` |
| `related_iqs` | Array of References | No | Links to related IQs for cross-referencing |

#### Table 16.4.3: Capture Context Schema

| Field | Requirement | Description |
|---|---|---|
| `origin_url` | REQUIRED | The URL or screen the human was viewing at capture time |
| `origin_object_type` | OPTIONAL | Primitive type of the object in view, if contextual capture |
| `origin_object_id` | OPTIONAL | Identifier of the object in view, if contextual capture |
| `origin_object_label` | OPTIONAL | Human-readable label of the object in view |
| `session_id` | REQUIRED | Session identifier for continuity tracking |
| `captured_at` | REQUIRED | Timestamp of context capture |

**Immutability constraint.** Capture text is IMMUTABLE. The original human articulation is preserved exactly as captured. Edits create new versions appended to `capture_text_versions` with editor identity and timestamp. The original `capture_text` field is never modified. This ensures the original cognitive event is never retroactively rewritten — the governance record preserves what was articulated at the moment of emergence.

**Progressive immutability.** Fields set at creation (capture text, intent type, origin type, attribution, capture context) are never modified. Fields set during processing (assignment, resolution, promotion) are immutable from their assignment point. This mirrors the progressive immutability model of Signal Capture (§15.4).

### §16.5 IQ Lifecycle

The IQ lifecycle governs instances from capture through resolution, promotion, abandonment, or supersession. This subsection provides the architectural summary.

Seven states. Three terminal, one soft-terminal.

#### Table 16.5.1: IQ Lifecycle Summary

| State | Entry Condition | Exit Transitions | Authority Required |
|---|---|---|---|
| **Open** | IQ created via capture event, signal spawn, claim spawn, or workflow trigger | → Assigned (investigator designated) or → Superseded, → Abandoned | None — any human may capture an IQ |
| **Assigned** | Investigator designated (self, routed, or manual) | → Investigating (work begins) or → Assigned (reassignment) or → Superseded, → Abandoned | System (auto-route) or any authorized actor |
| **Investigating** | Investigator begins active work | → Resolved, → Promoted, → Abandoned or → Assigned (return to pool) or → Superseded | Assigned investigator |
| **Resolved** | Answer found and documented | Soft-terminal: → Open (reversion on new information) | Assigned investigator |
| **Promoted** | IQ converted to one or more primitives (Work, Decision, Intent) | Terminal | Assigned investigator |
| **Abandoned** | IQ determined no longer relevant | Terminal | Assigned investigator or IQ authority |
| **Superseded** | Another IQ absorbs or replaces this one | Terminal | IQ authority or superseding IQ's investigator |

**Soft-terminal: Resolved.** A resolved IQ MAY be re-opened if new information surfaces that contradicts or materially extends the original findings. Re-opening transitions the IQ back to Open and creates a new investigation cycle. The original resolution record is preserved (append-only). If the question resurfaces after a promoted IQ, a new IQ is created — promoted IQs cannot revert.

**Terminal: Promoted, Abandoned, Superseded.** Promoted IQs have been operationalized into primitives; further questions create new IQs. Abandoned IQs carry documented rationale; if the question resurfaces, a new IQ is created with a link to the abandoned predecessor. Superseded IQs link to the superseding IQ that inherited their investigation context.

#### Table 16.5.2: Resolution Paths

| Path | From State | To State | What Happens | Required Artifacts |
|---|---|---|---|---|
| Resolved (answer found) | Investigating | Resolved | Investigation complete; findings documented; may produce Claims entering graduation (§6.5) at stage Asserted | Resolution summary, resolver identity, timestamp, `resolution_decision_id`, linked claims (if any) |
| Promoted to Work | Investigating | Promoted | Investigation reveals effort is required — task decomposition, resource allocation, or execution beyond simple inquiry | `promotion_targets[]` with `target_type = "Work"` and `target_id`, promotion rationale, `resolution_decision_id` |
| Promoted to Decision | Investigating | Promoted | Investigation reveals a choice point — the question cannot be resolved by research alone | `promotion_targets[]` with `target_type = "Decision"` and `target_id`, promotion rationale, `resolution_decision_id` |
| Promoted to Intent | Investigating | Promoted | Investigation reveals a strategic gap — an organizational objective not yet formally captured | `promotion_targets[]` with `target_type = "Intent"` and `target_id`, promotion rationale, `resolution_decision_id` |
| Abandoned | Any active state | Abandoned | IQ determined no longer relevant | Abandonment rationale, resolver identity, timestamp, `resolution_decision_id` |
| Superseded | Any active state | Superseded | Another IQ absorbs this one | Superseding IQ identity, supersession rationale, `resolution_decision_id` |

**B9 enforcement.** Every terminal transition — Resolved, Promoted, Abandoned, Superseded — MUST produce a `resolution_decision_id` pointing to a Decision primitive (§4.1) that records the governance act of resolving the IQ. The resolution act IS a governance decision: it links to an Account (B4), is legitimized by the resolver's Authority (B5), and produces an Evidence record (the resolution summary) with truth type Declared (§6). This ensures no IQ resolution is invisible to the governance record. An IQ that reaches a terminal state without a `resolution_decision_id` is a B9 violation.

**Multiple promotion.** A single IQ MAY produce multiple promotion targets simultaneously when the investigation reveals compound findings. An IQ exploring a compliance gap might simultaneously produce a Work item to remediate the gap, a Decision to choose between remediation approaches, and an Intent to formalize a new compliance objective. Each target is recorded in the `promotion_targets[]` array. The single `resolution_decision_id` covers the governance act of the composite promotion.

**Promotion invariant requirements.** Promoted primitives MUST satisfy their own behavioral invariants independently: Work requires Commitment (B1), Decision requires Account (B4). The IQ's investigation context becomes part of the promoted primitive's evidentiary basis, but the promoted primitive carries its own governance obligations.

### §16.6 Decision-Blocking Interaction

IQs interact with the Decision lifecycle through a blocking mechanism. When a Decision participant identifies an investigation need, the formal mechanism ensures the Decision waits for findings rather than proceeding with incomplete information.

**Blocking flow.** A Decision participant says "I need to investigate first." An IQ is created with `origin_type = "capture"` and `blocking_decision_id` pointing to the Decision. The Decision enters a blocked state — pending, blocked by the open IQ. The IQ proceeds through its lifecycle normally (Open → Assigned → Investigating → terminal). When the IQ reaches any terminal state (Resolved, Promoted, or Abandoned), the Decision is unblocked. IQ findings are available as evidence input to the Decision.

#### Table 16.6.1: Decision-Blocking Constraints

| Constraint | Rule | Rationale |
|---|---|---|
| Blocking is voluntary | The actor chooses to create a blocking IQ; the system does not auto-block | Investigation need is a human judgment, not a system determination |
| One-to-many | A single Decision can be blocked by multiple IQs; all must resolve before the Decision unblocks | Multiple investigation needs may exist for a single Decision |
| Abandonment unblocks | If a blocking IQ is abandoned, the Decision unblocks | The investigation was deemed unnecessary; the Decision can proceed |
| Supersession transfers | If a blocking IQ is superseded, the block transfers to the superseding IQ | The investigation continues under a different IQ; the blocking relationship follows |
| Promotion unblocks | If a blocking IQ is promoted to a Decision, the original blocked Decision unblocks | The promoted Decision is a new governance object; the original Decision is no longer blocked |
| IQ findings as evidence | When a blocking IQ resolves, its findings enter the Decision's evidence basis | The Decision proceeds with the investigation context the participant needed |

**Distinction between resolution_decision_id and promoted Decision.** The `resolution_decision_id` records the governance act of resolving the IQ — it is the Decision that says "we investigated and here is what we found." A promoted Decision is a new governance object representing the choice point the IQ revealed. These are two distinct Decision instances. The first closes the IQ; the second opens a new governance thread.

### §16.7 Capture Entry Points

IQ Capture MUST be accessible from two classes of entry point: global (available from any substrate screen) and contextual (available from specific governed object views with auto-filled origin context).

#### Table 16.7.1: Global Capture Entry Points

| Location | Trigger | Context Captured |
|---|---|---|
| Header or navigation bar | Persistent capture icon | Current URL, timestamp, session identity |
| Command palette | Capture command or keyword | Current URL, timestamp, session identity |
| Mobile floating action | Capture action button | Current screen, timestamp, session identity |

#### Table 16.7.2: Contextual Capture Entry Points

| Context | Trigger | Additional Context |
|---|---|---|
| During Work execution | "Something came up" action on the Work view | Work item identity and state |
| During Decision making | "I need to investigate first" action on the Decision view | Decision in progress, trigger context, blocking relationship auto-created |
| During Evidence review | "This raises a question" action on the Evidence view | Evidence item identity, associated claim if applicable |
| From dashboard or monitoring | Quick capture from monitoring view | Dashboard state, active filters |
| From agent interaction | "Agent noticed → Human interprets" — agent surfaces a pattern; human captures interpretation | Agent observation, pattern detected, agent identity |

**DESIGN SPACE:** UI patterns — icon choice, keyboard shortcuts, button placement, modal flow — are implementation decisions. The architecture requires two capabilities: (a) global access from any substrate screen, and (b) contextual access from any governed object view that auto-fills origin context. Substrate implementations MUST satisfy both. They MAY choose any UI pattern that meets the zero-friction and always-available design principles (§16.1).

### §16.8 Governance Constraints

Five governance constraints ensure that IQ Capture maintains structural integrity across the capture, investigation, and resolution lifecycle.

#### Table 16.8.1: IQ Governance Rules

| Rule | Constraint | Rationale |
|---|---|---|
| Capture text immutability | Original `capture_text` is write-once; edits create new versions in `capture_text_versions[]` with editor identity and timestamp | Protects the original cognitive event from retroactive rewriting — the governance record preserves what was articulated at the moment of emergence |
| Resolution requirements | Every terminal transition MUST record: resolution summary, resolver identity, timestamp, linked artifacts, and `resolution_decision_id` | Ensures every IQ resolution is a traceable governance act (B9); no IQ is informally dismissed |
| Pattern privacy | IQ capture patterns by user MUST NOT be used for performance evaluation, supervisor visibility without explicit consent, or aggregation that identifies individuals | Prevents chilling effect on emergence capture — the mechanism requires that humans feel safe asking questions without fear of judgment |
| Re-opening rules | A Resolved IQ MAY be re-opened with documented rationale and new evidence reference; re-opening creates a new investigation cycle with original resolution preserved | Maintains audit trail while allowing new information to reactivate investigation |
| Promotion traceability | Every promoted primitive MUST link back to its source IQ via `origin_iq_id`; the IQ MUST record the promoted primitive via `promoted_to[]` | Preserves bidirectional lineage from cognitive emergence through investigation to organizational action |

**Priority inference.** Four conditions boost IQ priority:

| Condition | Priority Boost | Rationale |
|---|---|---|
| Origin is Decision in progress | +1 | IQ is blocking a Decision; timely resolution prevents governance delay |
| Origin is compliance domain | +1 | Regulatory implications warrant faster investigation |
| Intent is `challenge` | +1 | Active dispute requires timely resolution to prevent organizational drift |
| Multiple similar IQs exist | +1 | Pattern emerging; organizational attention warranted |

**Cross-mechanism interaction.** IQ Capture interacts with Signal Capture (§15) at two integration points. A signal in Processing state may spawn an IQ (§15.7) when deeper investigation is required; the signal enters Monitoring and tracks the IQ's resolution. An IQ during investigation may spawn a signal when a problem with a specific governed object is discovered. Both interactions create bidirectional links for lineage traversal (§14.5).

**Truth type assignment.** The IQ capture act produces Authoritative truth (§6) — a human's articulation of cognitive emergence is a fact about what surfaced in their thinking. IQ resolution produces Declared truth — it represents the resolver's best current judgment. Promoted artifacts carry the truth type rules of their target primitive class (§14.4).

## Scope

IQ Capture is limited to cognitive emergence and investigation contexts. It does not create governance records directly; promotion creates records. IQ routing targets investigators, not authority chains. IQs may be context-anchored or free-floating; they do not require object attachment.

## Locked Design Positions

**Locked positions:**

1. IQ Capture is generative and emergence-oriented; it captures cognitive events that do not anchor to specific objects.
2. All eight intent types are required; six are first-order (emergence about the governed world), two are second-order (emergence about capture itself).
3. IQ capture text is immutable from creation; edits create versioned entries without modifying the original.
4. Every resolved IQ MUST produce a `resolution_decision_id` recording the governance act of resolution (B9 enforcement).
5. IQs can spawn Signals and vice versa with bidirectional traceability.
6. Resolved IQs are soft-terminal and may revert to Open if new information surfaces; Promoted/Abandoned/Superseded are hard-terminal.
7. Decision-blocking is voluntary, one-to-many, and unblocks automatically when the blocking IQ reaches any terminal state.
8. Promotion targets (Work, Decision, Intent) each create a primitive instance carrying its own behavioral invariants.

## Implementation Requirements

**SDK MUST constraints:**

- MUST enforce B9: every terminal transition (Resolved, Promoted, Abandoned, Superseded) produces a `resolution_decision_id` pointing to a Decision primitive.
- MUST provide global IQ capture entry points on every substrate screen.
- MUST provide contextual IQ capture entry points with auto-filled origin context on all governed object views.
- MUST preserve capture text immutability: original capture is write-once; modifications create versioned entries in `capture_text_versions[]`.
- MUST preserve capture context immutability: origin context snapshots are immutable from creation.
- MUST implement eight intent types with first-order/second-order classification.
- MUST implement four origin types with type-specific routing defaults.
- MUST implement seven-state lifecycle with soft-terminal Resolved allowing reversion and hard-terminal Promoted/Abandoned/Superseded.
- MUST enforce progressive immutability: fields set at creation are never modified; fields set during processing are immutable from assignment.
- MUST support decision-blocking: IQs created with `blocking_decision_id` block the referenced Decision until the IQ reaches a terminal state.
- MUST support multiple promotion: a single IQ may produce multiple promotion targets simultaneously.
- MUST enforce promotion invariants: promoted primitives satisfy their own behavioral invariants independently.
- MUST implement bidirectional signal-IQ traceability with navigable links in both directions.
- MUST preserve pattern privacy: IQ capture patterns by user MUST NOT be used for performance evaluation without explicit consent.
- MUST support IQ re-opening: Resolved IQs MAY revert to Open with documented rationale; original resolution is preserved append-only.
- MUST assign truth types: IQ capture is Authoritative; IQ resolution is Declared; promoted artifacts carry their target primitive's truth type rules.

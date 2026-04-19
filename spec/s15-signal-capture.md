# §15 Signal Capture

Signal Capture is the mechanism by which humans surface problems, concerns, or observations about governed objects within the substrate. Every signal attaches to a specific governed object and asserts: something about this object requires attention. Signals route to the flagged object's authority chain (B8, §5), ensuring that captured concerns reach someone with the power to act.

Signal Capture is an extension point, not a primitive. The DLP-Core protocol defines the capture requirement through two behavioral invariants: B7 (every governed object is flaggable) and B8 (every signal routes to authority). The DLP substrate provides the implementation — signal types, routing logic, lifecycle management, and IQ integration. Signal is not one of the 19 primitives (§4); it is an operational mechanism that operates on all 19.

§14 provides the parent capture framework. This section specifies Signal Capture as the first of two concrete capture mechanisms. IQ Capture (§16) specifies the second. Signal Capture serves a single architectural purpose: converting human observation of object-level problems into structured, routable governance data.

### §15.1 Signal Capture Definition

#### Table 15.1.1: Signal Capture Boundary

| Signal Capture IS | Signal Capture IS NOT |
|---|---|
| Object-anchored — every signal targets a specific governed object | Global ideation capture — use IQ Capture (§16) for free-floating insights |
| Problem-oriented — "this thing is wrong" | Emergence-oriented — "something surfaced" is IQ territory |
| Reactive — responding to an observed concern about an existing object | Generative — capturing new ideas or questions belongs to IQ Capture |
| Routed to the flagged object's authority chain (B8) | Routed to the user's personal queue — signals go to authority, not to self |

**Distinction from IQ Capture.** Signal Capture flags problems with existing objects. IQ Capture (§16) captures cognitive overflow that emerges during work but does not anchor to any single object. A signal says "THIS thing is wrong." An IQ says "something emerged that needs a home." The mechanisms are parallel, not subordinate (§14.5).

Six design principles govern Signal Capture:

1. **Zero friction.** One selection (signal type) to complete capture. Free text description is available but never required.
2. **Always available.** Present on every governed object — all 19 primitives across all 5 tiers (B7, §5). Every interface that renders a governed object MUST include a signal capture entry point.
3. **Structurally typed.** Seven signal types classify the nature of the concern and determine routing. Categories map to primitives for automated routing decisions.
4. **Optionally deep.** Free text, severity, related signals, and IQ creation are available but optional. Capture MUST succeed at minimum effort.
5. **Authority-routed.** Every signal routes to an authority on the governance chain for the flagged object (B8, §5). Algedonic bypass permits routing to any depth in the delegation hierarchy.
6. **Context preserved.** A snapshot of system state at capture time is recorded with every signal. This snapshot is immutable — it preserves what the human observed at the moment of capture.

### §15.2 Signal Types

Seven signal types classify the nature of the concern and determine routing. The first five are standard signal types for problem-oriented concerns. The sixth — `curiosity` — is a positive perturbation signal for opportunities. The seventh — `reservation` — is the second-order lineage extension defined in §14.6.1.

#### Table 15.2.1: Signal Type Definitions

| Type | Label | Description | Primary Primitives |
|---|---|---|---|
| `reality_mismatch` | Reality Mismatch | "The data does not match what I see" — observed state diverges from recorded state | Evidence, Account |
| `logical_inconsistency` | Logical Inconsistency | "These things contradict each other" — governed objects are in mutual conflict | Decision, Commitment, Intent |
| `business_concern` | Business Concern | "We can, but should we?" — strategic or value misalignment | Orientation, Capacity |
| `compliance_concern` | Compliance Concern | "We may not be allowed to do this" — authority or constraint question | Authority, Constraint |
| `other` | Something Else | "Does not fit the categories above" — escape valve for uncategorized concerns | (none — taxonomy gap indicator) |
| `curiosity` | Curiosity | "This could be an opportunity" — positive perturbation where something unexpected, promising, or worth investigating has been observed. Semantic polarity is flipped: curiosity signals surface opportunities, not problems. | Orientation, Learning, Evidence |
| `reservation` | Accepted with Reservation | "I accept but hold objections" — second-order lineage (§14.6.1) | Decision, Commitment |

**Curiosity as positive perturbation.** The first five standard signal types are problem-oriented — they surface concerns, mismatches, inconsistencies, and compliance questions. Curiosity inverts the semantic polarity: it signals that something worth investigating has been observed, even though nothing is wrong. A human notices an unexpected pattern, an unexplored opportunity, or an emergent capability that the governance record does not yet reflect. Curiosity signals route through the same B8 mechanism as problem-oriented signals — they reach authority on the flagged object's governance chain — but the authority's response is exploratory rather than corrective. Curiosity signals frequently spawn IQs (§16) because investigation is the natural response to a positive perturbation.

**Signal type selection guidance.** Signal types are mutually exclusive per signal instance. A single concern selects one type. If the concern spans multiple types (e.g., a reality mismatch that also raises a compliance question), the human selects the primary type and may describe the secondary dimension in the free text field, or raise a second signal.

**Taxonomy evolution.** The `other` type serves as an escape valve. High frequency of `other` signals indicates taxonomy gaps — the existing six standard types do not cover the concern space. Signal distribution analytics (signal type frequency by object type, by account, over time) enable taxonomy refinement. Licensee configurations MAY add signal types beyond the required seven; they MUST NOT remove any of the seven.

### §15.3 Flaggable Object Model

B7 (§5) requires that every primitive class have a signal attachment surface. The flaggable object type enumeration covers all 19 primitives across all 5 tiers (§4.5).

#### Table 15.3.1: Flaggable Object Types

| Primitive | Tier | Default Routing Target | Typical Signal Types |
|---|---|---|---|
| **Intent** | 1 | `owner_id` — intent owner is responsible for purpose alignment | `logical_inconsistency`, `business_concern` |
| **Commitment** | 1 | `owner_id` — commitment owner accepted the responsibility | `logical_inconsistency`, `reality_mismatch`, `reservation` |
| **Capacity** | 1 | `owner_id` — capacity owner controls the resource allocation | `reality_mismatch`, `business_concern` |
| **Work** | 1 | `assigned_to_id` — work assignee is executing the transformation | `reality_mismatch`, `logical_inconsistency` |
| **Evidence** | 1 | `verified_by` (or `author_id` if unverified) — verifier is the last authority; author if no verifier assigned | `reality_mismatch`, `compliance_concern` |
| **Decision** | 1 | `made_by` — decision-maker is accountable for the choice | `logical_inconsistency`, `compliance_concern`, `reservation` |
| **Authority** | 1 | `holder_id` — authority holder governs its own delegation scope | `compliance_concern`, `logical_inconsistency` |
| **Account** | 1 | `steward_id` — account steward maintains the state record | `reality_mismatch`, `logical_inconsistency` |
| **Constraint** | 1 | `owning_authority_id` — constraint owner controls the rule | `compliance_concern`, `logical_inconsistency` |
| **Identifier** | 2 | `owning_entity_id` — entity that owns the addressed object | `reality_mismatch` |
| **Entity** | 2 | `parent_entity_id` (or self if root entity) — parent entity in recursive structure | `compliance_concern`, `reality_mismatch` |
| **Context** | 2 | `owner_id` — context owner established the situational frame | `reality_mismatch`, `business_concern` |
| **Namespace** | 2 | `steward_id` — namespace steward governs concept definitions | `logical_inconsistency`, `compliance_concern` |
| **Orientation** | 3 | `owner_id` — orientation owner holds the cognitive stance | `business_concern`, `logical_inconsistency` |
| **Learning** | 3 | `source_entity_id` — source entity initiated the learning cycle | `reality_mismatch`, `business_concern` |
| **Activation** | 3 | `owner_id` — activation owner controls readiness assessment | `business_concern`, `reality_mismatch` |
| **Interpretation** | 4 | `assigned_to` — assigned reviewer owns the meaning-resolution | `reality_mismatch`, `logical_inconsistency` |
| **Environment Interface** | 4 | `owning_entity_id` — entity that owns the boundary with external systems | `reality_mismatch`, `compliance_concern` |
| **Cycle** | 5 | `executed_by` — cycle executor manages the temporal pattern | `reality_mismatch`, `business_concern` |

**Routing rationale for infrastructure and extension primitives.** The Tier 1 primitives have well-established routing targets (owner, assignee, maker, holder). The following six primitives from Tiers 2–5 require specific routing rationale:

- **Account** routes to the account steward. Typical signals are `reality_mismatch` (state record diverges from observed organizational state) and `logical_inconsistency` (account records conflict with decisions or commitments recorded elsewhere).
- **Constraint** routes to the constraint's owning authority. Typical signals are `compliance_concern` (rule applicability is questioned) and `logical_inconsistency` (constraints conflict with each other or with authority delegations).
- **Identifier** routes to the owning entity. Typical signals are `reality_mismatch` (broken references, duplicate identifiers, addressing failures).
- **Namespace** routes to the namespace steward. Namespace governance varies by scope: core-scope concepts route to the protocol steward, domain-scope to domain authority, tenant-scope to organizational steward, sandbox-scope to sandbox owner. Typical signals are `logical_inconsistency` (concept conflicts across scopes) and `compliance_concern` (unauthorized concept additions).
- **Activation** routes to the activation owner. Typical signals are `business_concern` (readiness assessment disputes — conditions are met but the activation should not proceed, or vice versa).
- **Environment Interface** routes to the owning entity responsible for the boundary with external systems. Typical signals are `reality_mismatch` (external data does not match substrate state) and `compliance_concern` (boundary violations, unauthorized data exchange).

### §15.4 Signal Schema

The signal flag schema defines field requirements at the architectural level. Substrate implementations MUST provide these fields; they MAY extend with additional fields.

#### Table 15.4.1: Signal Flag Schema

| Field | Type | Requirement | Immutable | Description |
|---|---|---|---|---|
| `flag_id` | Identifier | REQUIRED | From creation | Unique signal identifier |
| `object_type` | Enumeration (19 primitive types) | REQUIRED | From creation | Flagged object's primitive type — MUST be one of the 19 primitive types |
| `object_id` | Reference | REQUIRED | From creation | Flagged object's identifier |
| `signal_type` | Enumeration (7 types) | REQUIRED | From creation | Concern classification — one of the seven signal types (§15.2) |
| `severity` | Enumeration: low, medium, high, critical | OPTIONAL | No | Signaler's assessment of urgency; defaults to medium if unspecified |
| `description` | Text | OPTIONAL | From creation | Free text description of the concern — IMMUTABLE after capture |
| `raised_by` | Reference → Entity | REQUIRED | From creation | The human who raised the signal |
| `raised_at` | Timestamp | REQUIRED | From creation | When the signal was raised |
| `context_snapshot` | Object | REQUIRED | From creation | System state at capture time — IMMUTABLE after capture (see Table 15.4.2) |
| `routed_to` | Reference → Entity | REQUIRED | Per routing event | Authority holder the signal is routed to; updated on escalation |
| `routed_at` | Timestamp | REQUIRED | Per routing event | When routing occurred |
| `status` | Enumeration (7 states) | REQUIRED | No | Current lifecycle state — see §15.6 |
| `spawned_iq_id` | Reference → IQ | OPTIONAL | From assignment | If this signal spawned an IQ, the linked IQ identifier |
| `related_signals` | Array of References | OPTIONAL | No | Links to related signals on the same or related objects |
| `resolution` | Object | OPTIONAL | From resolution | Resolution record: summary, resolver identity, timestamp, linked artifacts |
| `dismissal` | Object | OPTIONAL | From dismissal | Dismissal record: rationale, dismisser identity, timestamp |
| `escalation_history` | Array of Objects | OPTIONAL | Append-only | Routing history: each escalation event records previous target, new target, reason, timestamp |
| `reservation_nature` | Enumeration (7 natures) | Conditional | From creation | REQUIRED when `signal_type = 'reservation'`; classification per §14.6.1 |
| `surface_trigger` | Object | Conditional | No | REQUIRED when `signal_type = 'reservation'`; trigger configuration per §14.6.1 |

**Progressive immutability.** Signal fields become immutable at the state where they are set. Fields set at creation (object identity, signal type, description, context snapshot, attribution) are never modified. Fields set during processing (routing, resolution, dismissal) are immutable from their assignment point. This enforces audit integrity: the governance record preserves what was observed, when, by whom, and how it was resolved.

#### Table 15.4.2: Context Snapshot Schema

| Field | Requirement | Description |
|---|---|---|
| `object_state` | REQUIRED | Current state of the flagged object at capture time — property values, status, relationships |
| `object_label` | REQUIRED | Human-readable label of the flagged object |
| `captured_at` | REQUIRED | Timestamp of the snapshot |
| `preceding_action` | OPTIONAL | What the signaler was doing immediately before raising the signal |
| `active_constraints` | OPTIONAL | Constraints currently in scope for the flagged object |
| `view_context` | OPTIONAL | The signaler's interface context — view mode, filters, navigation path |

The context snapshot is the evidentiary basis for the signal. It captures what the signaler observed at the moment of capture. The snapshot is IMMUTABLE — even if the flagged object's state changes during processing, the snapshot preserves the state that prompted the concern.

### §15.5 Signal Routing

Signal routing implements B8 (§5): every signal MUST reach an authority on the governance chain for the flagged object. Routing occurs automatically on signal creation and may be overridden through escalation at any non-terminal state.

#### Table 15.5.1: Default Routing Rules

| Tier | Object Type | Default Route | Escalation Path |
|---|---|---|---|
| **1** | Intent | `owner_id` | Owner → parent entity authority → account authority → root authority |
| **1** | Commitment | `owner_id` | Owner → authorizing authority → account authority → root authority |
| **1** | Capacity | `owner_id` | Owner → commitment authority → account authority → root authority |
| **1** | Work | `assigned_to_id` | Assignee → commitment owner → account authority → root authority |
| **1** | Evidence | `verified_by` / `author_id` | Verifier → account authority → root authority |
| **1** | Decision | `made_by` | Decision-maker → authorizing authority → account authority → root authority |
| **1** | Authority | `holder_id` | Holder → delegating authority → root authority |
| **1** | Account | `steward_id` | Steward → parent account steward → root authority |
| **1** | Constraint | `owning_authority_id` | Owning authority → delegating authority → root authority |
| **2** | Identifier | `owning_entity_id` | Owning entity → namespace steward → root authority |
| **2** | Entity | `parent_entity_id` | Parent entity → grandparent entity → root authority |
| **2** | Context | `owner_id` | Owner → account authority → root authority |
| **2** | Namespace | `steward_id` | Steward → scope authority (per namespace governance scope) → root authority |
| **3** | Orientation | `owner_id` | Owner → account authority → root authority |
| **3** | Learning | `source_entity_id` | Source entity → parent entity → root authority |
| **3** | Activation | `owner_id` | Owner → commitment authority → root authority |
| **4** | Interpretation | `assigned_to` | Assigned reviewer → account authority → root authority |
| **4** | Environment Interface | `owning_entity_id` | Owning entity → integration authority → root authority |
| **5** | Cycle | `executed_by` | Executor → cycle owner → account authority → root authority |

**Algedonic bypass.** Any signal MAY be escalated to any depth in the authority chain at any non-terminal state. The default route targets the nearest authority; algedonic bypass allows direct routing to higher authority when the signaler judges the default route insufficient. This implements the algedonic channel: emergency signals bypass intermediate hierarchy to reach whoever has the power to act. Algedonic bypass prevents organizational silence — signals cannot be trapped at a level that lacks the power or willingness to act.

**Routing fallback.** If the primary routing target is null (e.g., unverified evidence with no `verified_by`), the system falls back to the authority chain for the flagged object's containing account. If no account context exists, the signal routes to the instance-level root authority. A signal MUST always route somewhere — an unroutable signal is a B8 violation.

#### Table 15.5.2: Routing Override Rules

| Condition | Override | Rationale |
|---|---|---|
| `signal_type = 'reservation'` | Route to flagger's own queue (not authority chain) unless `reservation_nature` involves compliance or authority concerns | Reservations are self-directed — the flagger defers their own objection for future review (§14.6.1) |
| `severity = 'critical'` | Auto-escalate one level above default routing target | Critical signals warrant immediate attention from higher authority |
| `signal_type = 'compliance_concern'` | Copy to compliance authority regardless of default routing target | Compliance concerns require visibility to the compliance function independent of the flagged object's authority chain |
| Routing timeout (configurable) | Auto-escalate one level | Unacknowledged signals escalate to prevent silent suppression |

### §15.6 Signal Lifecycle

The signal lifecycle governs flags from creation through resolution. This subsection provides the architectural summary.

Seven states. Two terminal.

#### Table 15.6.1: Signal Lifecycle States

| State | Entry Condition | Exit Transitions | Authority Required |
|---|---|---|---|
| **Created** | Human raises signal on a governed object; context snapshot captured | → Routed (automatic, B8) | None — any human may raise a signal on any governed object |
| **Routed** | System auto-routes to authority holder per Table 15.5.1 | → Processing (authority acknowledges) or → Escalated (algedonic bypass or timeout) | System (automatic routing) |
| **Processing** | Authority holder accepts signal for review | → Resolved, → Dismissed, → Escalated, → Monitoring (IQ spawned) | Routed or escalated-to authority holder |
| **Escalated** | Signal forwarded up the authority chain | → Processing (higher authority accepts) or → Escalated (further escalation) | Any human on authority chain, or system (timeout) |
| **Monitoring** | Authority spawns IQ from signal | → Resolved (IQ resolves substantively) or → Dismissed (IQ dismissed) or → Processing (IQ refused/superseded without replacement) | System (triggered by IQ lifecycle events) |
| **Resolved** | Concern substantively addressed | Terminal | Processing authority holder, or system (from IQ resolution) |
| **Dismissed** | Concern determined not actionable | Terminal | Processing authority holder — MUST NOT be the signal raiser |

#### Table 15.6.2: Terminal State Semantics

| Terminal State | Meaning | Required Artifacts |
|---|---|---|
| **Resolved** | The concern was substantive and corrective action was taken or a substantive determination was made | Resolution summary describing the action taken; resolver identity; timestamp; linked governance artifacts (modified objects, new Work items, new Decisions) |
| **Dismissed** | The concern was determined not actionable — noise, misunderstanding, or already addressed | Dismissal rationale (mandatory — silent dismissal is a B8 violation); dismisser identity; timestamp |

**No reversion from terminal states.** A resolved or dismissed signal cannot be reopened. If new information surfaces that re-raises the same concern, a new signal MUST be created. The new signal MAY reference the prior signal, but the governance record maintains distinct lifecycle instances. This preserves audit integrity — each signal is a complete, self-contained lifecycle.

**Dismissal authority constraint.** Dismissal MUST NOT be performed by the signal raiser. Only an authority on the flagged object's governance chain may dismiss a signal. This prevents self-suppression: the person who raised the concern cannot also decide it was not actionable. The dismisser MUST provide a rationale — the rationale is part of the governance record and is subject to audit.

**Reservation lifecycle variant.** Reservation signals (`signal_type = 'reservation'`) follow the same seven-state machine with modified transition behavior (see Reservation Signal Variant). Key differences: routing targets the flagger's own queue (not the authority chain) unless compliance-related; processing is triggered by surface trigger activation (not authority acknowledgment); the flagged object proceeds without interruption (NON-BLOCKING per §14.6.1).

### §15.7 IQ Integration

Signal Capture integrates with IQ Capture (§16) at two defined interaction points. These interactions create bidirectional traceability between the signal and IQ lifecycles.

**Signal spawns IQ.** During processing, the authority holder determines that the concern requires deeper investigation than a signal resolution can provide. The authority creates an IQ with `origin_type = "signal"` and `source_signal_id` linking to the originating signal. The signal transitions to Monitoring and remains open until the IQ reaches a terminal state.

The signal does not close when an IQ is created. The signal's lifecycle tracks the *concern*; the IQ's lifecycle tracks the *investigation* of that concern. Until the investigation concludes, the concern is unresolved.

#### Table 15.7.1: IQ Resolution to Signal State Mapping

| IQ Resolution Outcome | Signal Terminal State | Rationale |
|---|---|---|
| `adjusted` — object was corrected | Resolved | The concern was substantive; corrective action was taken |
| `spawns_new` — IQ spawned new Work, Decision, or IQ | Resolved | The concern was substantive; organizational action followed |
| `converts` — IQ converted to formal Decision or Work | Resolved | The concern was substantive; promoted to governance action |
| `dismissed` — signal was noise | Dismissed | The concern was investigated and determined not actionable |
| `refused` — investigation refused without successor | Returns to Processing | The signal concern remains unresolved; authority must re-evaluate |
| `superseded` — IQ replaced by another IQ | Stays in Monitoring | The signal tracks the successor IQ until it resolves |

**IQ spawns signal.** During IQ investigation, the investigator discovers a problem with a specific governed object. A new signal is created with `origin_type = "iq"` and `origin_iq_id` linking to the parent IQ. The signal enters the standard lifecycle independently of the parent IQ. The IQ records the spawned signal reference for traceability.

**Bidirectional traceability.** Every signal-IQ interaction creates links in both directions. A signal that spawned an IQ carries `spawned_iq_id`; the IQ carries `origin_type = "signal"` and `source_signal_id`. An IQ that spawned a signal carries the spawned signal reference; the signal carries `origin_type = "iq"` and `origin_iq_id`. The lineage chain is traversable in either direction through the dual-language query layer (§28).

#### Table 15.7.2: Cross-Machine Interactions

| Interaction | Direction | Description |
|---|---|---|
| Signal → IQ Lifecycle | Outbound | Spawning an IQ creates an IQ instance entering the IQ lifecycle at Open. The signal enters Monitoring and reads the IQ's terminal state but does not transition it. |
| Signal → Truth Type System | Constraint | Signals carry truth type Authoritative (§6). The context snapshot is Authoritative evidence. If resolution produces new evidence, that evidence enters the truth type system at its own appropriate type. |
| Signal → Record Lifecycle | Independence | Signals can target objects in any record lifecycle state. A signal on a Superseded object is valid. Supersession of the flagged object does not auto-resolve the signal. |
| Signal ← Claims Graduation | Inbound | A pattern of resolved signals on the same object type or account may be asserted as a Claim and enter the Claims Graduation protocol (§6.5). |

## Scope

Signal Capture is limited to flagging problems with existing governed objects. It does not create new governance records; it does not serve as a general feedback mechanism; it does not route to user personal queues. Signal routing MUST respect the authority chain for the flagged object; it MUST NOT bypass authority entirely.

## Locked Design Positions

**Locked positions:**

1. Signal Capture flags problems with specific governed objects and routes to authority on the flagged object's governance chain (B8).
2. All seven signal types are required and mutually exclusive per signal instance.
3. Signal context snapshots are immutable from capture and provide evidentiary basis for the concern.
4. Signal routing implements algedonic bypass; escalation is available to any depth in the authority chain.
5. Dismissal authority constraint: dismissal MUST NOT be performed by the signal raiser.
6. Signals and IQs are independent lifecycle mechanisms that can spawn each other with bidirectional traceability.
7. Reservation signals are non-blocking and route to flagger's personal queue (unless compliance-related).

## Implementation Requirements

**SDK MUST constraints:**

- MUST enforce B7: every interface that renders a governed object MUST include a signal capture entry point covering all 19 primitive types.
- MUST enforce B8: every signal MUST route to an authority on the governance chain for the flagged object; algedonic bypass MUST be available to any depth.
- MUST enforce progressive immutability: fields set at creation remain immutable; fields set during processing are immutable from assignment.
- MUST preserve context snapshots: snapshots are immutable from capture and accessible for audit.
- MUST implement seven signal types with mutually exclusive selection per signal instance.
- MUST implement seven-state lifecycle with two terminal states; no reversion from terminal states.
- MUST enforce dismissal authority constraint: dismissal authority MUST NOT be the signal raiser; dismissal rationale is mandatory.
- MUST enforce critical severity auto-escalation: critical signals escalate one level above default routing target.
- MUST implement routing timeout: unacknowledged signals auto-escalate one level to prevent silent suppression.
- MUST implement reservation signal variant: reservation signals are non-blocking, route to flagger's queue unless compliance-related, with surface trigger configuration.
- MUST support bidirectional signal-IQ traceability: signal↔IQ links are navigable in both directions with `spawned_iq_id`, `origin_type`, and source_id fields.

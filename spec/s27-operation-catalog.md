# §27 Operation Catalog

This specification section defines implementation contracts for the Decision Lineage Protocol substrate.

The Decision Lineage Protocol substrate exposes two hundred and forty-four named operations across the nineteen primitives and ten cross-cutting operation domains. Each operation is a typed, authorized action with defined preconditions, postconditions, and invariant implications. This section specifies the complete logical contract that governs what the SDK must support; transport protocol, serialization format, and client-library ergonomics are DESIGN SPACE. The operations act on the logical schema defined in §26.

This section defines six operation types (§27.1), specifies per-primitive operation contracts organized by tier (§27.2–§27.6), catalogs cross-cutting composite operations (§27.7), consolidates the authorization matrix (§27.8), maps invariant enforcement points per operation (§27.9), and formalizes SDK constraints (§27.10).

---

## §27.1 Operation Architecture Overview

### §27.1.1 Six Operation Types

Every operation in the catalog belongs to exactly one of six types. The type determines the operation's structural role, its relationship to the primitive lifecycle, and its invariant enforcement profile.

### Table 27.1.1: Operation Type Taxonomy

| Type | Definition | Invariant Profile | Example |
|---|---|---|---|
| **Create** | Instantiate a new primitive record with all minimum viable record (MVR) fields | Pass 1 always fires; truth type assigned at creation | `intent.create`, `evidence.create` |
| **Read** | Retrieve primitive records with truth type filtering and projection control | No invariant enforcement; respects depth-gated visibility (§23) | `intent.read`, `authority.read` |
| **Update** | Modify mutable fields on existing records, respecting truth type immutability rules | Pass 1 fires on affected invariants; Authoritative records reject updates | `commitment.update`, `constraint.update` |
| **Delete** | Soft-delete only — transitions `record_lifecycle_state` to `archived` | No hard deletes in the governance graph; audit trail preserved | `context.deactivate`, `account.close` |
| **Transition** | Advance a primitive through its state machine lifecycle | Pass 1 fires; state machine guards enforce valid transitions | `intent.activate`, `work.complete` |
| **Composite** | Multi-primitive operations that compose atomic operations within a single logical transaction | All constituent invariants fire; transaction is atomic | `signal.create`, `authority.delegate` |

**Create and Read are universal.** Every primitive supports at minimum a Create and a Read operation. Tier 1 primitives support the full six-type range. Tier 2 primitives omit some Transition operations because their lifecycles are simpler.

**Transition is the primary governed operation type.** Most governance-significant actions are state machine transitions, not field updates. A Commitment does not become active by updating a status field — it becomes active through `commitment.activate`, which enforces B2 capacity verification, records the transition event, and triggers downstream invariant checks.

**Composite operations are atomic.** A composite operation either completes entirely or fails entirely. Partial completion is not a valid state. The SDK determines whether atomicity is achieved through database transactions, saga patterns, or another mechanism — this is DESIGN SPACE.

### §27.1.2 Authorization Model

Every write operation — Create, Update, Delete, Transition, and Composite — requires authorization. Read operations respect depth-gated visibility (§23) but do not require explicit authorization checks beyond instance membership.

The authorization check comprises five sequential gates:

1. **Authority scope.** The actor holds an Authority with scope covering the operation target.
2. **Authority traceability.** The Authority is traceable to a root authority via the B5 delegation chain.
3. **RACIVG role permission.** The actor's RACIVG role for the target context permits the operation (§22.3).
4. **Graduation gate.** The actor's graduation stage meets the minimum required for the operation (§27.8).
5. **Profile gate.** The profile tier makes the operation available — some operations exist only at EAS or BAS (§27.8).

All five gates pass or the operation is rejected. There is no partial authorization. The authorization check itself produces an Authoritative audit record regardless of outcome.

### §27.1.3 Truth Type Rules

All operations that produce or modify records are governed by truth type rules (§6):

**Authoritative records are append-only.** No field may be modified after creation. Corrections require supersession — a new record with `supersedes_ref` pointing to the original. The original is preserved unchanged.

**Declared records are versioned.** Modifications create new versions. The fields `id`, `created_at`, `created_by`, and `truth_type` are immutable across versions. Temporal bounds (`effective_from`, `effective_to`) are adjustable.

**Derived records are ephemeral.** They are recomputable from source data. Manual editing is prohibited. When source records change, dependent Derived records are marked stale and queued for recomputation.

**Opaque records are boundary-enforced.** Content behind an inspection boundary that cannot be crossed (§6.1). Opaque is system-assigned at boundary detection — no actor writes Opaque directly. Opaque records cannot be promoted through the standard Derived → Declared → Authoritative path; they transition to another truth type only when the inspection boundary is removed (§6.1, Opaque → Computed transition).

**AI actors produce Derived only.** All AI-generated content enters the substrate as Derived regardless of the AI actor's confidence level. Promotion to Declared requires human review. Promotion to Authoritative requires a separate human Decision with traceable Authority (B5). Truth type promotion is monotonic — no demotion path exists.

### §27.1.4 Invariant Enforcement

Every write operation triggers zero or more invariant checks from the ten behavioral invariants (§5). Enforcement follows the two-pass validation strategy:

**Pass 1 (SHACL Core).** Single-node property checks at O(1). Invariants B1, B2, B3, B4, B5-immediate, B6-binding, and B8-core. Always Blocking — no profile may downgrade below Blocking.

**Pass 2 (SHACL-SPARQL).** Graph traversal and transitive closure checks. Invariants B5-T, B6-U, B8-routing, and B9. Profile-configurable enforcement mode — defaults vary by invariant and profile tier.

**Schema Pass.** Shapes-on-shapes validation. Invariant B7 (all objects flaggable). Runs at deployment time and schema changes, not per-operation.

Operations declare which invariants they touch. The invariant enforcement engine validates after the operation writes but before the transaction commits. If any Blocking invariant fails, the transaction rolls back.

---

## §27.2 Tier 1 Primitive Operations

Tier 1 contains the nine irreducible core primitives (§4.1). Each primitive supports the full operation type range and participates in at least one behavioral invariant.

### §27.2.1 Intent

Intent represents a stated objective — the answer to "what should happen?" An Intent lifecycle progresses through proposal, activation, and terminal resolution.

### Table 27.2.1: Intent Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `intent.create` | Create | Description non-empty; `created_by` resolves to active Entity; `authority_scope` references active Authority | Status = `draft`; MVR fields written; truth type assigned (human: Authoritative/Declared; AI: Derived) | B5 |
| `intent.read` | Read | Target exists | Record returned at requested projection | — |
| `intent.update` | Update | Status not terminal (`achieved`, `abandoned`, `superseded`); truth type permits modification | Version incremented; modified fields updated | B5 |
| `intent.propose` | Transition | Status = `draft`; description non-empty; `created_by` resolves | Status → `proposed` | — |
| `intent.activate` | Transition | Status = `proposed`; activator holds activation authority | Status → `active` | B5 |
| `intent.pause` | Transition | Status = `active`; reason provided | Status → `paused` | — |
| `intent.resume` | Transition | Status = `paused` | Status → `active` | — |
| `intent.block` | Transition | Status = `active`; blocker recorded with escalation time | Status → `blocked` | B7, B8 |
| `intent.unblock` | Transition | Status = `blocked`; no active blockers remain | Status → `active` | — |
| `intent.achieve` | Transition | Status = `active`; all weighted success criteria met | Status → `achieved`; child Intents cascade | B4 |
| `intent.abandon` | Transition | Status ∈ {`active`, `paused`, `blocked`}; Decision recorded | Status → `abandoned`; child Intents cascade | B4 |
| `intent.supersede` | Transition | Status not terminal; replacement Intent linked | Status → `superseded` | B4 |

**Lifecycle.** `draft` → `proposed` → `active` ↔ `paused` | `blocked` → `achieved` | `abandoned` | `superseded`.

*Semantic boundary:* Intent declares what should happen. Commitment (§27.2.2) binds parties to do it. Work (§27.2.4) executes it. Achievement of an Intent is a governance conclusion, not a work-tracking status.

### §27.2.2 Commitment

Commitment binds two or more parties to an obligation with terms and capacity allocation. The lifecycle governs negotiation, acceptance, performance, and breach.

### Table 27.2.2: Commitment Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `commitment.create` | Create | Parties ≥ 2; terms ≥ 1; capacity allocation specified | Status = `proposed`; B2 verified | B2, B5 |
| `commitment.read` | Read | Target exists | Record returned | — |
| `commitment.update` | Update | Status not terminal; truth type permits modification | Version incremented; capacity re-validated | B2 |
| `commitment.negotiate` | Transition | Status = `proposed` | Status → `negotiating` | — |
| `commitment.amend` | Composite | Status = `negotiating`; all parties agree to amendment | Terms updated; parties notified | B2 |
| `commitment.accept` | Transition | Status ∈ {`proposed`, `negotiating`}; all parties have accepted | Status → `accepted` | B4 |
| `commitment.activate` | Transition | Status = `accepted`; all parties confirmed; capacity verified | Status → `active` | B2, B4, B5 |
| `commitment.fulfill_term` | Transition | Term exists; Evidence of fulfillment provided | Term marked fulfilled; if all terms complete, status → `fulfilled` | B3, B4 |
| `commitment.breach` | Transition | Term breach detected; Evidence recorded | Breach recorded; material breach → status = `breached` | B3, B4 |
| `commitment.suspend` | Transition | Status = `active` | Status → `suspended` | B4 |
| `commitment.resume` | Transition | Status = `suspended` | Status → `active` | — |
| `commitment.cure` | Transition | Status = `breached`; within cure period; no active breaches remain | Status → `active` | B4 |
| `commitment.terminate` | Transition | Governing authority exercised | Status → `terminated` | B4, B5 |
| `commitment.expire` | Transition | `effective_until` < now() | Status → `expired` | — |
| `commitment.supersede` | Transition | Replacement Commitment linked | Status → `superseded` | B4 |

**Lifecycle.** `proposed` ↔ `negotiating` → `accepted` → `active` ↔ `suspended` → `fulfilled` | `breached` → `terminated` | `expired` | `superseded`. Cure transitions `breached` → `active`.

### §27.2.3 Capacity

Capacity represents a quantified resource — time, skill, budget, authority scope — that can be allocated to Commitments. The B2 invariant ensures Commitments never exceed available Capacity.

### Table 27.2.3: Capacity Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `capacity.create` | Create | `total_available` ≥ 0; owner resolves; measurement unit specified | Status = `available`; allocated = 0 | B5 |
| `capacity.read` | Read | Target exists | Record returned with current allocation state | — |
| `capacity.update` | Update | Status not terminal | Version incremented; available recomputed | B2 |
| `capacity.allocate` | Transition | Requested amount ≤ (available − reserved) | Allocation recorded; available decremented | B2 |
| `capacity.release` | Transition | Allocation exists | Allocated decremented; available incremented | B2 |
| `capacity.check_sufficient` | Read | Target exists | Returns boolean sufficiency result | — |
| `capacity.exhaust` | Transition | Available = 0 (system-triggered) | Status → `exhausted` | B2 |
| `capacity.block` | Transition | Status ∈ {`available`, `partially_available`} | Status → `blocked` | — |
| `capacity.replenish` | Transition | `total_available` updated | Status recalculated from new totals | B2 |

**Lifecycle.** `available` ↔ `partially_available` ↔ `exhausted` | `blocked` | `reserved` | `replenishing`.

### §27.2.4 Work

Work represents effort expended in service of a Commitment. B1 enforces the structural link — every Work instance must reference exactly one Commitment. Work Specifications (SM-4) govern how AI-composed work is confirmed and promoted.

### Table 27.2.4: Work Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `work.create` | Create | `commitment_id` references active Commitment | Status = `defined`; B1 link established | B1, B5 |
| `work.read` | Read | Target exists | Record returned | — |
| `work.update` | Update | Status not terminal | Version incremented | B1 |
| `work.assign` | Transition | Status = `ready`; assignee within delegation bounds | Status → `assigned` | B5 |
| `work.start` | Transition | Status = `assigned` | Status → `in_progress` | B1 |
| `work.block` | Transition | Status = `in_progress`; blocker recorded | Status → `blocked` | B7, B8 |
| `work.unblock` | Transition | Status = `blocked`; no active blockers | Status → `in_progress` | — |
| `work.submit_for_review` | Transition | Status = `in_progress`; outputs present | Status → `review` | — |
| `work.accept` | Transition | Status = `review`; reviewer holds authority | Status → `accepted` | B4, B5 |
| `work.reject` | Transition | Status = `review`; rationale provided | Status → `rejected` | B4 |
| `work.rework` | Transition | Status = `rejected` | Status → `in_progress` | — |
| `work.complete` | Transition | Status = `accepted`; Evidence created | Status → `completed` | B3, B4 |
| `work.abandon` | Transition | Decision recorded | Status → `abandoned` | B4 |
| `work.cancel` | Transition | Governing authority exercised | Status → `cancelled` | B4, B5 |
| `work.supersede` | Transition | Replacement Work linked | Status → `superseded` | — |

**Lifecycle.** `defined` → `ready` → `assigned` ↔ `in_progress` ↔ `blocked` → `review` → `accepted` | `rejected` → `completed` | `abandoned` | `cancelled` | `superseded`. Rejection loops back to `in_progress` via `work.rework`.

### Table 27.2.5: Work Specification Operations (SM-4)

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `work_spec.compose` | Create | Delegation active; AI actor within bounds | Status = `draft`; truth type = Derived | B1, B3 |
| `work_spec.confirm` | Transition | Human reviewer at Stage A/B, or elevated risk at Stage C | Status → `confirmed`; Decision + Account created | B4 |
| `work_spec.skip_confirm` | Transition | Stage C only; consequence level = reversible | Status → `confirmed` (bypasses human review) | B4 |
| `work_spec.emit` | Transition | Status = `confirmed` | Status → `active`; emitted to execution layer | — |
| `work_spec.complete` | Transition | All outputs present | Status → `completed`; outputs = Derived Evidence | B3 |
| `work_spec.promote` | Transition | Human with traceable Authority; AI-Cannot-Be-Principal | Outputs promoted from Derived | B3, B4, B5 |
| `work_spec.reject` | Transition | Human reviewer; rationale mandatory | Status → `rejected` | B4 |

**Lifecycle (SM-4).** `draft` → `confirmed` → `active` → `completed` → `promoted` | `rejected`. Stage C low-risk work may skip confirmation via `work_spec.skip_confirm`.

### §27.2.5 Evidence

Evidence records factual artifacts — observations, measurements, documents, validation reports. Evidence is immutable after creation. The B3 invariant requires every Evidence instance to carry exactly one truth type.

### Table 27.2.6: Evidence Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `evidence.create` | Create | Source identified; truth type assigned; `confidence_level` ∈ [0,1]; human: Authoritative/Declared; AI: Derived only | Record created; immutable after creation | B3, B5 |
| `evidence.read` | Read | Target exists | Record returned at fidelity level L0–L3 | — |
| `evidence.verify` | Composite | Evidence exists; verification criteria applied | Verification record appended; state transitions through Unverified → VerificationPending → Verified/VerificationFailed/CannotVerify | B3 |
| `evidence.supersede` | Composite | Old Evidence preserved; new record linked via `supersedes_ref` | New record created; old unchanged | B3, B4 |
| `evidence.promote` | Transition | Derived → Declared or Declared → Authoritative (monotonic only); human with traceable Authority; AI-Cannot-Be-Principal | Truth type promoted; Decision + Account created | B3, B4, B5 |

**Immutability rule.** Evidence records cannot be updated. Verification adds records — it does not modify the Evidence itself. Corrections require supersession: a new Evidence record that links to and replaces the original. The original is preserved.

### §27.2.6 Decision

Decision records a governance choice — the answer to "what was chosen and why?" Every Decision creates an Account record (B4). AI actors may produce Decision proposals as Derived content; only humans may finalize governance Decisions.

### Table 27.2.7: Decision Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `decision.create` | Create | `decision_by` resolves; `authority_source` traceable (B5); `options_considered` ≥ 2 for selection-type; `rationale` non-empty; human: Authoritative; AI: Derived only (proposals) | Status = `final`; Account created | B4, B5 |
| `decision.read` | Read | Target exists | Record returned with deviation data | — |
| `decision.reverse` | Transition | Status = `final`; within `reversal_window`; `reversible` = true | Status → `reversed`; new Decision created for the reversal | B4 |
| `decision.supersede` | Transition | Replacement Decision linked | Status → `superseded` | B4 |
| `decision.expire` | Transition | Temporal bound exceeded (system-triggered) | Status → `expired` | — |
| `decision.compute_deviation` | Read | Decision exists; deviation metrics defined | Deviation vector computed: ‖Δ‖ = √(Σ(wᵢ · Δᵢ²)) + penalty(Δ_C); escalation if above threshold | — |

**Lifecycle.** `draft` → `final` → `superseded` | `reversed` | `expired`.

**Decision zones govern AI participation.** Zone 1 (Green): AI may decide autonomously at Stage C, logged, reversible, within delegation. Zone 2 (Yellow): AI prepares, human reviews within window. Zone 3 (Orange): human decides with AI support. Zone 4 (Red): human only — AI may not prepare.

### §27.2.7 Authority

Authority represents the right to act — delegated or root. The B5 invariant requires every Authority to be traceable through a delegation chain terminating at a root authority. Authority is the foundation of the authorization model.

### Table 27.2.8: Authority Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `authority.create` | Create | Root: explicit designation; Delegated: parent scope is superset; B5 chain valid | Status = `active`; chain established | B5 |
| `authority.read` | Read | Target exists | Record returned with chain metadata | — |
| `authority.delegate` | Composite | Delegator holds sufficient scope; scope subset verified; creates Authority + Delegation + Decision + Account | New Authority created; B5 chain extended | B4, B5 |
| `authority.revoke` | Transition | Revoking authority holds governing scope | Status → `revoked`; cascade-revokes all downstream delegations; active signals re-routed | B4, B5, B8 |
| `authority.suspend` | Transition | Suspending authority holds governing scope | Status → `suspended`; in-flight work paused; active signals re-routed | B4, B5, B8 |
| `authority.reinstate` | Transition | Status = `suspended`; reinstating authority verified | Status → `active` | B4, B5 |
| `authority.review` | Composite | Review triggered (scheduled, expiry, audit) | Review record created; may trigger revocation or renewal | B4, B5 |
| `authority.expire` | Transition | Temporal bound exceeded (system-triggered) | Status → `expired` | — |
| `authority.can_exercise` | Read | Actor and target specified | Returns `{authorized, reason, authority_ref}` | — |
| `authority.resolve_chain` | Read | Authority specified | Returns complete delegation chain to root | — |

**Lifecycle.** `pending` → `active` ↔ `suspended` → `expired` | `revoked` | `superseded`.

### Table 27.2.9: Delegation Operations (SM-5)

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `delegation.propose` | Create | Delegator identified; scope specified; AI-Cannot-Be-Principal | Status = `proposed` | B5 |
| `delegation.grant` | Transition | Human approves; scope verified as subset of parent | Status → `active`; Decision + Account created | B4, B5 |
| `delegation.modify` | Transition | Human only; AI cannot modify own delegation; scope change verified | Status → `modified`; tighten-only cascade propagated | B4, B5 |
| `delegation.confirm_modification` | Transition | Affected parties confirm | Modification finalized | B4 |
| `delegation.suspend` | Transition | Suspending authority holds governing scope | Status → `suspended` | B4 |
| `delegation.reinstate` | Transition | Status = `suspended` | Status → `active` | B4 |
| `delegation.revoke` | Transition | Rationale mandatory; governing authority exercised | Status → `revoked` (terminal); signals routed through this delegation re-routed | B4, B5, B8 |
| `delegation.expire` | Transition | Temporal bound exceeded; grace period for in-flight work | Status → `expired` (terminal); signals routed through this delegation re-routed | B8 |

**Lifecycle (SM-5).** `proposed` → `active` ↔ `modified` | `suspended` → `revoked` | `expired`.

**AI-Cannot-Be-Principal applies to all delegation writes.** AI actors cannot propose, grant, modify, or revoke delegations. AI actors may query `authority.can_exercise` and `authority.resolve_chain` to inspect delegation state.

### §27.2.8 Account

Account records per-event linkage between actors and their governance actions. Every Decision creates an Account entry (B4). Account records are always Authoritative and immutable.

### Table 27.2.10: Account Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `account.create` | Create | `actor_id` resolves; `action_type` specified; `authority_chain` captured | Record created; truth type = Authoritative; immutable | B4, B5 |
| `account.read` | Read | Target exists | Record returned as Authoritative | — |
| `account.post_transaction` | Composite | Account exists; `transaction_type` specified; `evidence_ref` linked; `authorized_by` verified | Transaction appended; immutable; truth type = Authoritative | B4 |
| `account.reconstruct_at` | Read | Account exists; target timestamp specified | Transaction log replayed through target time | — |
| `account.reconcile` | Composite | Reconciliation initiated; all transactions reviewed | Reconciliation record created; reconciliation itself is a Decision (B4) | B4 |
| `account.close` | Transition | Closing authority exercised | No new transactions accepted | B4, B5 |

**Account is per-event linkage, not aggregation.** Each Account record links one actor to one governance action. Temporal framing — "what was Actor X accountable for during Period Y?" — uses Cycle queries (§28). Account records are never modified or deleted.

### §27.2.9 Constraint

Constraint represents a binding rule — policy, compliance requirement, physical limitation, or organizational boundary. B6 requires every Constraint to target at least one primitive instance and specify an enforcement mode.

### Table 27.2.11: Constraint Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `constraint.create` | Create | `applies_to` non-empty; `enforcement_mode` specified; `authority_source` traceable; SHACL shape URI linked | Status = `active` | B6, B5 |
| `constraint.read` | Read | Target exists | Record returned with current evaluation state | — |
| `constraint.update` | Update | `applies_to` cannot be emptied; `enforcement_mode` cannot be removed | Version incremented | B6 |
| `constraint.evaluate` | Read | Target Constraint and governed instances specified | Returns: `satisfied` / `violated` / `exception_applied` / `not_applicable` | B6 |
| `constraint.grant_exception` | Composite | Override protocol (§5.5): overriding actor holds Authority governing the violated primitive; creates Decision + Evidence + Account | Exception recorded; override Decision = Declared (not Authoritative) | B4, B5, B6 |
| `constraint.revoke_exception` | Transition | Exception exists; revoking authority verified | Constraint re-enforced on target | B6 |
| `constraint.suspend` | Transition | Status = `active` | Status → `suspended` | B4 |
| `constraint.deprecate` | Transition | Status ∈ {`active`, `suspended`}; optional link to replacement | Status → `deprecated` | B4 |

**Lifecycle.** `active` ↔ `suspended` → `deprecated`.

**Override protocol (§5.5).** Exceptions follow a four-step protocol: (1) the overriding actor must hold Authority governing the violated primitive, (2) the override is itself a Decision requiring an Account (B4), (3) Evidence is produced as Declared truth (not Authoritative), and (4) the original violation record is preserved — overrides are append-only.

---

## §27.3 Tier 2 Infrastructure Operations

Tier 2 contains four unconditional infrastructure primitives (Identifier, Entity, Context, Namespace) that exist in every substrate deployment regardless of profile tier (§4.5), plus Actor lifecycle operations (SM-1). These primitives support the Tier 1 governance model but do not carry MVR fields.

### §27.3.1 Identifier

Identifier provides unique naming and resolution services. System-generated UUIDs carry Authoritative truth type. Human-supplied external references carry Declared truth type.

### Table 27.3.1: Identifier Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `identifier.create` | Create | Unique within (instance, type, value) | Record created; truth type assigned | — |
| `identifier.read` | Read | Target exists | Record returned | — |
| `identifier.resolve` | Read | Identifier value and type specified | Returns bound primitive reference or null | — |
| `identifier.version` | Transition | Identifier exists; new version justified | Version incremented; prior version preserved | — |
| `identifier.bind` | Composite | Target primitive exists; no conflicting binding | Identifier bound to primitive instance | — |

### §27.3.2 Entity and Actor

Entity represents a participant — human, organization, role, or system. Actor is the governance identity that an Entity holds within a substrate instance. Actor lifecycle operations (SM-1) govern registration, activation, suspension, and deactivation.

### Table 27.3.2: Entity Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `entity.create` | Create | `entity_type` ∈ {`human`, `organization`, `role`, `system`} | Record created; `entity_type` immutable after creation | B5 |
| `entity.read` | Read | Target exists | Record returned | — |
| `entity.update` | Update | `entity_type` immutable; other fields modifiable | Version incremented | — |
| `entity.deactivate` | Transition | No active sole-Accountable roles | Status → `inactive` | B4 |
| `entity.archive` | Transition | Status = `inactive` | Status → `archived` (terminal) | — |

### Table 27.3.3: Actor Operations (SM-1)

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `actor.register` | Create | Entity exists; `actor_type` ∈ {`human`, `ai_automation`, `ai_agentic`, `ai_assistive`} | Status = `registered`; Actor Context created | B5 |
| `actor.activate` | Transition | Status = `registered`; activation authority verified | Status → `active`; Decision + Account created | B4, B5 |
| `actor.suspend` | Transition | Status = `active`; suspending authority verified | Status → `suspended` | B4 |
| `actor.reinstate` | Transition | Status = `suspended`; reinstating authority verified | Status → `active` | B4 |
| `actor.deactivate` | Transition | AI: all delegations revoked; Human: all roles transferred | Status → `deactivated` | B4 |
| `actor.archive` | Transition | Status = `deactivated` | Status → `archived` (terminal) | B4 |

**Lifecycle (SM-1).** `registered` → `active` ↔ `suspended` → `deactivated` → `archived`.

**Actor Context operations.** Each actor holds an Actor Context per instance they participate in (§22.1). Actor Context supports `actor_context.create`, `actor_context.update`, and `actor_context.delete`. Deletion is blocked if the actor is sole Accountable for active Decisions in the instance.

### §27.3.3 Context

Context provides situational framing for governance actions. Six context types — temporal, spatial, relational, situational, interpretive, and composite — classify the environment in which primitives operate.

### Table 27.3.4: Context Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `context.create` | Create | `context_type` specified; relevant fields populated | Record created | — |
| `context.read` | Read | Target exists | Record returned | — |
| `context.update` | Update | Status = `active` | Version incremented | — |
| `context.deactivate` | Transition | No dependent active primitives require this Context | Status → `inactive` | — |
| `context.scope` | Read | Context and target primitive specified | Returns scoping applicability | — |

### §27.3.4 Namespace

Namespace organizes primitives into hierarchical scoping domains. Four scopes — core, domain, tenant, and sandbox — govern vocabulary and concept ownership. Sandbox namespaces carry mandatory expiration.

### Table 27.3.5: Namespace Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `namespace.create` | Create | `scope` ∈ {`core`, `domain`, `tenant`, `sandbox`}; sandbox requires `expiration` | Status = `proposed` | B5 |
| `namespace.read` | Read | Target exists | Record returned | — |
| `namespace.update` | Update | `scope` immutable; other fields modifiable | Version incremented | — |
| `namespace.graduate` | Transition | Graduation Decision (B4); `sandbox` → `tenant` → `domain` → `core` (monotonic) | Status → `graduated`; scope upgraded | B4 |
| `namespace.expire` | Transition | Sandbox: `expiration` exceeded | Status → `expired` | — |
| `namespace.retire` | Transition | Retirement Decision recorded | Status → `retired` | B4 |
| `namespace.bind` | Composite | Target primitive exists; namespace active | Primitive bound to namespace | — |

**Lifecycle.** `proposed` → `under_review` → `active` → `graduated` | `expired` | `retired`.

---

## §27.4 Tier 3 Governed Operations

Tier 3 primitives — Orientation, Learning, and Activation — support deliberate governance posture and institutional adaptation. They activate at graduation Stage B (§4.7). All Tier 3 write operations require elevated governance authority.

### §27.4.1 Orientation

Orientation declares the pre-decisional framing — values, boundaries, and harm prohibitions — that governs an entity's participation. Only one Orientation may be active per owner. AI cannot create or modify Orientation (L4 gradient level — highest deliberation).

### Table 27.4.1: Orientation Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `orientation.create` | Create | Only one active per owner; `harm_boundary.absolute_prohibitions` non-empty; human only (AI-Cannot-Be-Principal) | Record created; previous Orientation superseded if exists | B5 |
| `orientation.read` | Read | Target exists | Record returned | — |
| `orientation.update` | Update | Formal governance process only (L4); human only | Version incremented | B4, B5 |
| `orientation.supersede` | Transition | Replacement Orientation linked; governance Decision recorded | Status → `superseded` | B4, B5 |
| `orientation.deactivate` | Transition | Owner deactivated or archived | Status → `inactive` | B4 |

### §27.4.2 Learning

Learning captures institutional adaptation signals — pattern observations, validations, threshold adjustments, and structural reconfigurations. Four governance levels (L1–L4) determine the deliberation required for application.

### Table 27.4.2: Learning Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `learning.create` | Create | `learning_type` specified; `governance_level` ∈ {L1, L2, L3, L4} | Status = `detected` | B3 |
| `learning.read` | Read | Target exists | Record returned | — |
| `learning.analyze` | Transition | Status = `detected`; analysis criteria applied | Status → `analyzed` | — |
| `learning.apply` | Transition | Status = `analyzed`; Decision required (B4); human only for L3–L4 | Status → `applied` | B4 |
| `learning.reject` | Transition | Status = `analyzed`; rationale mandatory; human only | Status → `rejected` | B4 |
| `learning.monitor` | Transition | Status = `applied` | Status → `monitoring` | — |

**Lifecycle.** `detected` → `analyzed` → `applied` → `monitoring` | `rejected`.

### §27.4.3 Activation

Activation governs the six-stage governance pipeline (§12): discover, analyze, implement, evidence, verify, sustain. Each stage produces specific primitive outputs and enforces stage-appropriate invariants.

### Table 27.4.3: Activation Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `activation.create` | Create | `governance_stage` ∈ {`discover`, `analyze`, `implement`, `evidence`, `verify`, `sustain`} | Status = `pending` | B6 |
| `activation.read` | Read | Target exists | Record returned with stage progress | — |
| `activation.ready` | Transition | Status = `pending`; prerequisites met | Status → `ready` | — |
| `activation.activate` | Transition | Status = `ready` | Status → `active` | B6 |
| `activation.block` | Transition | Status = `active`; blocker recorded | Status → `blocked` | B7, B8 |
| `activation.unblock` | Transition | Status = `blocked`; no active blockers | Status → `active` | — |
| `activation.complete` | Transition | Status = `active`; stage outputs produced | Status → `complete` | B4, B6 |
| `activation.advance_stage` | Composite | Current stage complete; A and G roles confirmed; next stage requirements projected | Stage advanced; new Activation created for next stage | B4, B5, B6 |

**Status lifecycle.** `pending` → `ready` → `active` ↔ `blocked` → `complete`.

**Six-stage pipeline (§12).** `discover` → `analyze` → `implement` → `evidence` → `verify` → `sustain`. Stage advancement requires both Accountable (A) and Governor (G) role confirmation at EAS. BAS supports a subset of stages.

---

## §27.5 Tier 4 AI-Native Operations

Tier 4 primitives — Interpretation and Environment Interface — extend the substrate with AI-specific governance capabilities. They activate at graduation Stage B and are available at EAS and BAS only (§4.7).

### §27.5.1 Interpretation

Interpretation captures AI-generated meaning-making — classifications, extractions, recommendations, analyses, predictions, and gap identifications. All Interpretations enter as Derived (B3). Resolution requires human action.

### Table 27.5.1: Interpretation Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `interpretation.create` | Create | `interpretation_type` specified (14 types); stakes × novelty routing evaluated; truth type = Derived | Record created; red-zone guard applied for high-stakes × high-novelty | B3, B5 |
| `interpretation.read` | Read | Target exists | Record returned with provenance and routing metadata | — |
| `interpretation.propose` | Transition | Status = `pending`; AI actor within delegation | Status → `agent_proposed` | B3 |
| `interpretation.submit_for_review` | Transition | Status = `agent_proposed` | Status → `human_review` | — |
| `interpretation.resolve` | Transition | Status = `human_review`; human only (AI-Cannot-Be-Principal) | Status → `resolved`; Decision + Account created | B4, B5 |
| `interpretation.dispute` | Transition | Status = `resolved`; dispute Evidence provided | Status → `disputed` | B3, B4 |
| `interpretation.withdraw` | Transition | Status ∈ {`pending`, `agent_proposed`} | Status → `withdrawn` | — |

**Lifecycle.** `pending` → `agent_proposed` → `human_review` → `resolved` | `disputed` | `withdrawn`.

**Stakes × novelty routing matrix (§4.6.3).** Low × Low: Citation Fidelity (verification). Low × High: Pattern Anomaly Flag (escalation). High × Low: Reasoning Integrity (logic review). High × High: Judgment Alignment (expert interpretation) — red zone. Red-zone guard rejects promotions without evidence of substantive review (rubber-stamp detection).

### §27.5.2 Environment Interface

Environment Interface connects the substrate to external systems — inbound signals, outbound actions, and feedback loops. Inbound signals enter as Derived. Outbound actions require Authority.

### Table 27.5.2: Environment Interface Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `env_interface.create` | Create | `interface_type` ∈ {`inbound_signal`, `outbound_action`, `feedback_loop`}; governance authority verified | Status = `active` | B5 |
| `env_interface.read` | Read | Target exists | Record returned | — |
| `env_interface.update` | Update | Status = `active` | Version incremented | — |
| `env_interface.receive_signal` | Composite | Interface active; inbound signal received | Interpretation created (Derived); routed via Ontology Alignment (§19) | B3, B7, B8 |
| `env_interface.send_action` | Composite | Interface active; Authority verified; outbound action authorized | Action dispatched; Evidence recorded | B4, B5 |
| `env_interface.close_loop` | Composite | Feedback data received; both inbound and outbound components exist | Learning signal generated; loop record updated | B3 |
| `env_interface.dormant` | Transition | Status = `active`; inactivity threshold exceeded | Status → `dormant` | — |
| `env_interface.close` | Transition | Status ∈ {`active`, `dormant`}; closing authority exercised | Status → `closed` (terminal) | B4 |

**Lifecycle.** `active` ↔ `dormant` → `closed`.

---

## §27.6 Tier 5 Configuration Operations

Tier 5 contains a single primitive — Cycle — that provides temporal configuration for governance rhythms. Cycle is available at EAS and BAS only.

### §27.6.1 Cycle

Cycle defines recurring governance cadences — execution pulses, activation sweeps, weekly reviews, learning integrations, orientation checks, and commitment reconciliations. Every cycle's decision step requires `sense_maker = human`.

### Table 27.6.1: Cycle Operations

| Operation | Type | Preconditions | Postconditions | Invariants |
|---|---|---|---|---|
| `cycle.create` | Create | `cycle_type` ∈ 6 types; cadence configured; human only | Record created | B5 |
| `cycle.read` | Read | Target exists | Record returned with cadence and dependency data | — |
| `cycle.configure` | Update | Decision step must retain `sense_maker = human` | Configuration updated; dependent cycles notified | B4 |
| `cycle.start` | Transition | Status = `scheduled`; prerequisites met | Status → `running` | — |
| `cycle.complete` | Transition | Status = `running`; all outputs produced | Status → `completed` | B4 |
| `cycle.skip` | Transition | Status = `scheduled`; skip Decision recorded | Status → `skipped` | B4 |
| `cycle.fail` | Transition | Status = `running`; failure condition detected | Status → `failed`; Signal created | B4, B7 |
| `cycle.query_boundaries` | Read | Cycle exists; temporal query specified | Returns cycle boundaries for Account temporal framing | — |

**Lifecycle.** `scheduled` → `running` → `completed` | `skipped` | `failed`.

---

## §27.7 Cross-Cutting Operations

Ten cross-cutting operation domains define composite operations that span multiple primitives. Six primary domains — signals (§14–§15), IQs (§16), VPs (§17), governance activation (§12), orchestration (§A1–§A2), and KPIs — address governance concerns that no single primitive handles alone. Four additional domains — profile/portfolio management, graduation, agent behavioral cycles, and ontology alignment — provide lifecycle and integration operations.

### §27.7.1 Signal Operations

Signals implement the algedonic channel — the mechanism for surfacing problems at any governance level. B7 guarantees that all nineteen primitive classes have a signal attachment surface. B8 guarantees that every signal routes to an authority on the governance chain.

### Table 27.7.1: Signal Operations

| Operation | Type | Composes | Invariants |
|---|---|---|---|
| `signal.create` | Composite | Signal + Evidence (Authoritative) + Context (situational) | B7 |
| `signal.create_reservation` | Composite | Signal with reservation nature (7 types: concern, uncertainty, disagreement, precedent/scope/timing/resource question); does not block governance action | B7 |
| `signal.route` | Composite | Resolves authority chain; routes to immediate governing authority or any ancestor at any depth | B8 |
| `signal.reroute` | Composite | Must stay on governance chain; routing history preserved | B8 |
| `signal.escalate` | Composite | Next higher authority on chain; emergency signals bypass intermediate hierarchy | B8 |
| `signal.resolve` | Composite | Creates resolution Decision + Account; may produce Work, Decision, Evidence, or IQ | B4, B8 |
| `signal.dismiss` | Composite | Creates dismissal Decision with mandatory rationale | B4 |
| `signal.monitor` | Transition | Acknowledged; ongoing monitoring with periodic Evidence updates | — |

**Lifecycle (SM-7).** `created` → `routed` → `processing` → `escalated` → `monitoring` → `resolved` | `dismissed`.

Any actor may raise a signal. Resolution and dismissal require a Decision (B4) by an actor with authority over the flagged object.

### §27.7.2 IQ Operations

Investigative Queries operationalize emergence — the process by which questions arising from governance activity become institutional knowledge. B9 guarantees that every resolved IQ produces a Decision or Work item.

### Table 27.7.2: IQ Operations

| Operation | Type | Composes | Invariants |
|---|---|---|---|
| `iq.create` | Create | 8 intent types (explore, validate, disambiguate, fill_gap, challenge, connect, branch_inventory, confidence_probe); 4 origin types (claim, signal, capture, workflow) | — |
| `iq.create_from_signal` | Composite | Signal-spawned IQ inherits routing context | B8 |
| `iq.create_from_claim` | Composite | Claims review reveals uncertainty | — |
| `iq.create_branch_inventory` | Composite | Second-order lineage extension: enumerates open decision branches | — |
| `iq.create_confidence_probe` | Composite | Second-order lineage extension: evaluates if accumulated evidence shifts original confidence | — |
| `iq.assign` | Transition | Assignment methods: manual, auto-routed, inherited | — |
| `iq.investigate` | Composite | Creates investigation Evidence records and research Work items | B1, B3 |
| `iq.resolve` | Composite | Must produce Decision or Work item (B9); Decision requires Account (B4) | B4, B9 |
| `iq.promote` | Composite | Resolved findings promoted to permanent knowledge | B9 |
| `iq.abandon` | Composite | Decision to not act is itself a Decision — satisfies B9 | B4, B9 |
| `iq.supersede` | Transition | Replacement link established | — |
| `iq.revert` | Transition | `resolved` → `open`; new evidence contradicts resolution; new resolution eventually required | B9 |

**Lifecycle (SM-8).** `open` → `assigned` → `investigating` → `resolved` → `promoted` | `abandoned` | `superseded`. Reversion: `resolved` → `open`.

**Zero friction, globally accessible.** IQ creation has no authorization gate beyond instance membership. The cost of a question is always lower than the cost of an unasked question.

### §27.7.3 VP Operations

Validated Projections materialize derived views from source primitives — plan baselines, condition assessments, and predictions. All VPs carry truth type Derived (B3). No human approval is required for materialization.

### Table 27.7.3: VP Operations

| Operation | Type | Composes | Invariants |
|---|---|---|---|
| `vp.materialize` | Composite | Source primitives queried; derivation method applied; confidence computed; truth type = Derived | B3 |
| `vp.materialize_plan_baseline` | Composite | 4 modes: explicit, inferred, normative, comparative; TTL varies by mode | B3 |
| `vp.materialize_condition_assessment` | Composite | Requires current Plan Baseline; compares current vs. expected across 7 dimensions | B3 |
| `vp.materialize_prediction` | Composite | Requires current Condition Assessment; `model_ref` for reproducibility | B3 |
| `vp.mark_stale` | Transition | Event-based (source changed) or time-based (`valid_until` exceeded) | — |
| `vp.recompute` | Composite | Must not change source data; must produce identical output given identical inputs; must update provenance | B3 |
| `vp.invalidate` | Transition | Recomputation fails; cascade to dependent VPs | — |
| `vp.access_stale` | Composite | Demand-based synchronous recomputation triggered | B3 |

**Cache states (SM-9).** `current` → `stale` → `recomputing` → `current` | `invalidated`.

**Invalidation cascade.** Plan Baseline invalidation immediately invalidates dependent Condition Assessments, which immediately invalidates dependent Predictions. Staleness propagates differently — stale records may be recomputed; invalidated records cannot.

**VP disposition routing.** When a Condition Assessment reveals deviation, the disposition determines the governance response: investigate (creates Work), re-evaluate plan (creates Decision + recomputation), compensate (creates Constraint exception), acknowledge (creates Evidence + scheduled review), escalate (routes Decision by Authority), accept risk (creates Learning + Decision), remediate (creates corrective Work), defer (schedules future assessment), or no action (audit log entry only).

### §27.7.4 Governance Activation Operations

Governance Activation implements the six-stage pipeline (§12) that discovers, projects, and closes governance gaps. Each stage is a composite operation producing specific primitive outputs.

### Table 27.7.4: Governance Activation Operations

| Operation | Stage | Produces | Invariants |
|---|---|---|---|
| `gov_activation.discover_profile` | 1 | Evaluates ~200 pattern signatures against organizational attributes | — |
| `gov_activation.activate_sources` | 2 | Evaluates trigger conditions (ANY-OF, ALL-OF Boolean); tier-aware gating: Stage A = Tiers 1–2, Stage B adds Tier 3, Stage C adds Tier 4 | B6 |
| `gov_activation.project_requirements` | 3 | Projects verification types (V-EXIST through V-RATIFIED); criticality ordering: conservation-critical > invariant-enforcing > standard > advisory | B6 |
| `gov_activation.project_gaps` | 4 | Compares required vs. existing verification types; gap inventory with criticality ordering | — |
| `gov_activation.create_closure_roadmap` | 5 | Creates Work items, Commitments, verifies Capacity; closure stages: Awareness → Enablement → Management → Governance | B1, B2, B6 |
| `gov_activation.propagate_changes` | 6 | Governance posture updated; Learning signals for institutional adaptation | B6 |

### §27.7.5 Orchestration Operations

The seven-step orchestration engine (§A1, §A2) classifies cognitive work, activates constraints, routes to actors, selects within delegation bounds, composes Work Specifications, confirms with human oversight, and emits to the execution layer.

### Table 27.7.5: Orchestration Operations

| Operation | Step | Action | Invariants |
|---|---|---|---|
| `orchestration.classify` | 1 | AICAR decomposition: A-Citation, B-Reasoning, C-Context, D-Communication | B4 |
| `orchestration.activate_constraints` | 2 | Five-dimensional context: AICAR type × Decision type × Consequence level × Operating posture × Graduation stage | B6 |
| `orchestration.route` | 3 | Filters actor registry; matches capabilities within delegation bounds | B5 |
| `orchestration.select` | 4 | Verifies delegation active (SM-5); autonomy level sufficient | B4, B5 |
| `orchestration.compose` | 5 | Assembles Work Specification with composition strategy (sequential, parallel, iterative, conditional, delegated, federated) | B1 |
| `orchestration.confirm` | 6 | Required at Stage A/B (always), Stage C when elevated risk; AI-Cannot-Be-Principal | B4 |
| `orchestration.emit` | 7 | Emitted to execution layer | — |

**AICAR classification operations.** `aicar.classify_work` classifies cognitive work into A/B/C/D components, supporting composite classifications (A+B+C). `aicar.extend_type` registers new derivative types under human approval; derivatives inherit parent properties and cannot modify them (taxonomy closure).

**Delegation-as-Composition.** `delegation.compose` is a composite operation that assembles Intent + Authority + Commitment + Constraint + Work + Context into a coherent delegation envelope. AI-Cannot-Be-Principal. B1/B2/B5 enforced.

### §27.7.6 KPI Operations

KPI operations govern the introduction, retirement, and monitoring of governance health indicators. Three KPI classes — Invariant Signal (IS), Control KPI (CKPI), and Intent Progress Indicator (IPI) — serve distinct governance rhythms.

### Table 27.7.6: KPI Operations

| Operation | Type | Action | Invariants |
|---|---|---|---|
| `kpi.introduce` | Composite | Creates KPI with class (IS/CKPI/IPI), trigger type (Intent-driven, Constraint-driven, Emergent); Decision required | B4 |
| `kpi.retire` | Composite | 4 reasons: intent advanced, phase complete, constraint resolved, replaced; `learning_summary` mandatory; no KPI may silently disappear | B4 |
| `kpi.monitor_invariant_signals` | Read | 8 monitoring dimensions: cycle time distribution, quality/rework rate, variance/predictability, escalation frequency, judgment load, evidence completeness, semantic integrity, change failure rate | — |
| `kpi.review_implications` | Composite | Mandatory when Intent changes; reviews downstream KPI relevance | B4 |
| `kpi.validate_pattern` | Composite | Pattern must be promoted Derived → Declared before influencing routing | B3 |

### §27.7.7 Profile and Portfolio Operations

Profile operations govern instance creation and graduation across the three profile tiers (§21, §23). Portfolio operations manage recursive parent-child instance relationships with tighten-only constraint monotonicity.

### Table 27.7.7: Profile and Portfolio Operations

| Operation | Type | Action | Invariants |
|---|---|---|---|
| `profile.create_instance` | Composite | Creates substrate instance with profile type (EAS/BAS/PAS); content packages loaded; RACIVG scope set (EAS = 6 roles, BAS = 4, PAS = 2); tier activation determined | B5, B6 |
| `profile.trigger_graduation` | Composite | Triggers: revenue, recurring customers, investment, employment, unbounded drift, type-specific thresholds | — |
| `profile.evaluate_graduation` | Composite | Human-initiated evaluation; evidence gathered | B4 |
| `profile.decide_graduation` | Composite | GO/NO-GO Decision; BAS → EAS requires parent concurrence | B4, B5 |
| `profile.execute_graduation` | Composite | Evidence carries forward with original truth types and lineage preserved; source instance archives; new instance establishes own RACIVG authority structure at genesis — BAS → EAS adds OVERSIGHT (V) and ADVISORY (G) authority scopes | B5, B9 |
| `portfolio.spawn_instance` | Composite | Child constraints = intersection of all ancestors (tighten-only); graduation ceiling = parent's stage; AI-Cannot-Be-Principal | B5, B6 |
| `portfolio.tighten_constraint` | Composite | Child tightens parent position (always permitted) | B6 |
| `portfolio.relax_constraint` | Composite | Five-step evidence-driven process; surfaces as Signal to constraining authority; recursive upward if authority itself constrained | B5, B6, B8 |
| `portfolio.depth_gated_read` | Read | Own data: full; direct children: full; deeper: aggregate only; siblings/cousins: none; parent operational: none | — |

### §27.7.8 Graduation Operations

Graduation operations manage the per-scope AI governance maturity model (§8, §A1, SM-2). Graduation is per-scope — each AICAR type × Decision type × domain namespace has an independent graduation stage.

### Table 27.7.8: Graduation Operations

| Operation | Type | Action | Invariants |
|---|---|---|---|
| `graduation.advance` | Composite | Stage criteria evaluated (interaction counts, pattern validation, confirmation rates, sustained accuracy); no stage skipping; Decision + Account created | B4 |
| `graduation.auto_regress` | Composite | System-triggered on: rejection rate spike, escalation rate spike, routing accuracy degradation, deviation detection failure; drops to immediate predecessor only | B4 |
| `graduation.human_regress` | Composite | Human-initiated; may drop to any lower stage; Decision + Account created | B4 |

**Stage criteria thresholds.** Stage A → A+: 50+ interactions, 30+ days, 5+ contexts, 50+ human reviews. Stage A+ → B: 100+ interactions, validated patterns promoted to Declared, 2+ actors. Stage B → C: 85%+ confirmation rate, 90+ days sustained. Stage C maintenance: sustained accuracy, low rejection rate, stable escalation frequency.

### §27.7.9 Agent Behavioral Cycle Operations

The agent behavioral cycle (SM-6) governs how AI actors operate within the substrate. Three tracks — monitoring (Stage A), navigation (Stage A+/B), and execution (Stage C) — are gated by graduation stage.

### Table 27.7.9: Agent Behavioral Cycle Operations

| Operation | Type | Track | Invariants |
|---|---|---|---|
| `agent.context_load` | Composite | All tracks | Loads Actor Context, delegation, Constraints, Work Specifications; B5/B6 validated | B5, B6 |
| `agent.navigate` | Composite | Navigation (A+/B) | Explores decision space; produces recommendations as Derived | B3 |
| `agent.monitor` | Composite | Monitoring (all stages) | Evaluates Constraints; checks 80% threshold proximity; blocking violation → ALERTING (preempts all other tracks) | B6, B7, B8 |
| `agent.execute` | Composite | Execution (C only) | Requires autonomy ≥ 2 and active Work Specification; all outputs = Derived | B1, B3 |
| `agent.alert` | Composite | Monitoring → Alerting | Surfaces deviation to human; creates Signal; returns to LISTENING | B7, B8 |

**Cycle states (SM-6).** LISTENING → CONTEXT_LOADING → NAVIGATING | MONITORING | EXECUTING → PROPOSING | ALERTING | COMPLETING → LISTENING. The cycle is non-terminating — agents return to LISTENING after completing each pass.

### §27.7.10 Ontology Alignment Operations

Ontology alignment operations (§18, §19) manage the Knowledge Integration pipeline that maps external domain concepts to the substrate's nineteen-primitive model.

### Table 27.7.10: Ontology Alignment Operations

| Operation | Type | Action | Invariants |
|---|---|---|---|
| `ontology.preload` | Composite | Deploys concept signatures, bridges, and source registry entries at deployment time; truth type = Declared; bypasses graduation | — |
| `ontology.lazy_discover` | Composite | Runtime discovery; sandbox namespace entry; graduation path: sandbox → tenant → domain → core | B3 |
| `ontology.align` | Composite | Eight-stage KI pipeline: External Field Scan → Lexical Matcher → Structural Matcher → Semantic Matcher → Confidence Aggregator → Claim Writer → Review Queue → Graduation Engine; three independent matchers for consensus | B3 |
| `ontology.bridge` | Composite | Bridge types: Equivalent, Subsumes, Specializes, Overlaps, Complements; confidence decay per type | B3 |
| `ontology.graduate_concept` | Composite | Two-stage graduation: Stage A (normalization gates) + Stage B (multi-dimensional scoring); tier-weighted thresholds | B4 |

---

## §27.8 Authorization Matrix

The authorization matrix maps every write operation to actor type, minimum RACIVG role, minimum graduation stage, and profile tier availability. This section provides the consolidated summary.

### §27.8.1 Actor Type Permissions

Four actor types (§22.2) participate in operations with distinct permission boundaries. Permission values: P = Permitted, X = Prohibited, C = Conditional (conditions specified per operation).

### Table 27.8.1: Actor Type Permission Summary

| Operation Category | Human | AI-Automation | AI-Agentic | AI-Assistive |
|---|---|---|---|---|
| Create root Authority | P | X | X | X |
| Delegate/revoke Authority | P | X | X | X |
| Create/modify Orientation | P | X | X | X |
| Create/modify Constraints | P | X | X | X |
| Grant Constraint exceptions | P | X | X | X |
| Accept/activate Commitments | P | X | X | X |
| Finalize Decisions (Zone 3–4) | P | X | X | X |
| Accept/complete Work (final) | P | X | X | X |
| Resolve Interpretations | P | X | X | X |
| Promote truth types | P | X | X | X |
| Hold Accountable (A) role | P | X | X | X |
| Create Evidence (Derived) | P | P (system events) | C (within delegation) | C (within delegation) |
| Create Zone 1 Decisions | P | X | C (Stage C, reversible) | X |
| Execute Work | P | C (deterministic) | C (Stage C, autonomy ≥ 2) | C (interactive) |
| Allocate Capacity | P | X | C (Stage C, reversible) | X |
| Read any primitive | P | P | P | P |
| Resolve identifiers/chains | P | P | P | P |
| Evaluate Constraints | P | P | P | P |
| Create Signals | P | P | P | P |
| Create IQs | P | P | P | P |
| Detect Learning signals | P | P | P | X |

### §27.8.2 RACIVG Role Requirements

Operations that modify governance state require specific RACIVG roles. The role requirement varies by profile tier: EAS uses all six roles (RACIVG), BAS uses four (RACI), PAS uses two (Owner/Collaborator mapped to R and A).

### Table 27.8.2: RACIVG Minimum Role by Operation Category

| Operation Category | Minimum Role | EAS | BAS | PAS |
|---|---|---|---|---|
| Create/modify Orientation | A + G | A + G | A | Owner |
| Create root Authority | A | A | A | Owner |
| Delegate Authority | A | A | A | N/A |
| Activate Commitments | A | A | A | Owner |
| Finalize Decisions | A | A | A | Owner |
| Complete Work | R | R | R | Owner/Collaborator |
| Advance Activation stage | A + G | A + G | A | N/A |
| Grant Constraint exceptions | A + G | A + G | A | N/A |
| Resolve Interpretations | A | A | A | N/A |
| Create Signals | Any | Any | Any | Any |
| Create IQs | Any | Any | Any | Any |

**Exactly one A per context.** No governance context may have zero or multiple Accountable parties. The A role is always human (§22.3).

**G role for policy implications.** When an operation implicates Constraints or Orientation, the Governor (G) role must participate. G is EAS-only; BAS and PAS approximate through the A role.

**V role for evidence requirements.** When an operation requires Evidence evaluation, the Verifier (V) role must participate. V is EAS-only.

### §27.8.3 Graduation Gates

Some operations require a minimum graduation stage. Graduation is per-scope: AICAR type × Decision type × domain namespace (§8, §A1).

### Table 27.8.3: Graduation-Gated Operations

| Operation | Minimum Stage | Rationale |
|---|---|---|
| `work_spec.skip_confirm` | C | Only fully graduated AI may bypass human confirmation |
| `decision.create` (AI Zone 1) | C | Autonomous AI decisions require maximum graduation |
| `capacity.allocate` (AI) | C | AI capacity allocation requires demonstrated reliability |
| `agent.navigate` (SM-6) | A+ | Navigation track requires observed pattern reliability |
| `agent.execute` (SM-6) | C | Execution track requires full graduation |
| `interpretation.create` | B | AI interpretation requires Tier 3+ governance capability |
| `activation.advance_stage` | A+ | Stage advancement requires demonstrated governance maturity |
| `work.assign` (to AI) | B | AI work assignment requires Tier 3 governance |

**No stage skipping.** Graduation advances one stage at a time: A → A+ → B → C. Auto-regression may drop to the immediate predecessor stage only. Human-initiated regression may drop to any lower stage.

### §27.8.4 Profile Tier Availability

Some operations are available only at specific profile tiers. PAS is the most restricted; EAS is the most permissive.

### Table 27.8.4: Profile-Restricted Operations

| Operation Category | EAS | BAS | PAS |
|---|---|---|---|
| Tier 1–2 primitives (all operations) | Available | Available | Available |
| Tier 3 primitives (Orientation, Learning, Activation) | Available | Available (subset of stages) | Not available |
| Tier 4 primitives (Interpretation, Environment Interface) | Available | Available | Not available |
| Tier 5 primitives (Cycle) | Available | Available | Not available |
| Work Specifications (SM-4) | Available | Available | Not available |
| Delegation lifecycle (SM-5) | Available | Available | Not available |
| Constraint exception management | Available | Available | Not available |
| Namespace graduation | Available | Available | Not available |
| RACIVG V and G roles | Available | Not available | Not available |

---

## §27.9 Invariant Enforcement Points

Each behavioral invariant (§5) is triggered by a specific set of operations. This section maps invariants to their triggering operations, validation pass, enforcement timing, and override availability.

The trigger tables below list the operations whose primary purpose intersects the invariant — the operations where the invariant check is structurally essential to the operation's postconditions. Any operation that produces a Decision also triggers B4; the B4 table lists the operation categories rather than enumerating every Decision-producing operation individually. The per-primitive operation tables in §27.2–§27.6 are authoritative for the complete invariant profile of each operation.

### §27.9.1 B1 — Work Requires Commitment

Every Work instance must link to exactly one Commitment. Prevents shadow work.

**Shape:** `dlps:B1-WorkRequiresCommitment`
**Pass:** 1 (Core). Always Blocking.
**Override:** None. No override mechanism exists for B1.

### Table 27.9.1: B1 Triggering Operations

| Operation | Trigger Condition |
|---|---|
| `work.create` | Validates `commitment_id` references active Commitment |
| `work.update` | Validates `commitment_id` not removed |
| `work_spec.compose` | Validates parent Work has valid Commitment link |
| `orchestration.compose` | Validates composed Work Specification links to Commitment |
| `gov_activation.create_closure_roadmap` | Validates all generated Work items link to Commitments |
| `iq.resolve` | Validates any generated Work items link to Commitments |

### §27.9.2 B2 — Commitment Requires Capacity

Every Commitment must link to at least one Capacity allocation. Prevents impossible promises.

**Shape:** `dlps:B2-CommitmentRequiresCapacity`
**Pass:** 1 (Core). Always Blocking.
**Override:** None.

### Table 27.9.2: B2 Triggering Operations

| Operation | Trigger Condition |
|---|---|
| `commitment.create` | Validates Capacity allocation exists and is sufficient |
| `commitment.update` | Validates updated terms do not exceed Capacity |
| `commitment.activate` | Re-verifies Capacity sufficiency at activation time |
| `capacity.release` | Validates release does not violate dependent Commitments |
| `capacity.exhaust` | Validates no active Commitments depend on exhausted Capacity |
| `gov_activation.create_closure_roadmap` | Validates generated Commitments have Capacity |

### §27.9.3 B3 — Evidence Requires Truth Type

Every Evidence instance must carry exactly one truth type from the four-value vocabulary (Authoritative, Declared, Derived, Opaque — §6.1). All AI-generated content enters as Derived. Opaque is system-assigned at boundary detection. Prevents epistemic corruption.

**Shape:** `dlps:B3-EvidenceRequiresTruthType`
**Pass:** 1 (Core). Always Blocking.
**Override:** None.

### Table 27.9.3: B3 Triggering Operations

| Operation | Trigger Condition |
|---|---|
| `evidence.create` | Validates truth type assigned; AI actor → Derived only |
| `evidence.promote` | Validates monotonic promotion (Derived → Declared → Authoritative) |
| `work_spec.complete` | Validates all outputs carry truth type = Derived |
| `work_spec.promote` | Validates promotion follows truth type rules |
| `agent.execute` | Validates all outputs carry truth type = Derived |
| `interpretation.create` | Validates truth type = Derived for AI-generated interpretations |
| `vp.materialize` | Validates truth type = Derived for all materialized projections |
| `signal.create` | Validates Signal Evidence carries truth type = Authoritative |
| All AI actor write operations | Enforces `outputTruthType: "derived"` at protocol level |

### §27.9.4 B4 — Decision Requires Account

Every Decision must link to exactly one Account. Prevents context-free decisions.

**Shape:** `dlps:B4-DecisionRequiresAccount`
**Pass:** 1 (Core). Always Blocking.
**Override:** None.

### Table 27.9.4: B4 Triggering Operations

| Operation | Trigger Condition |
|---|---|
| `decision.create` | Account created and linked |
| `decision.reverse` | Reversal Decision creates new Account |
| `work_spec.confirm`, `work_spec.promote`, `work_spec.reject` | Confirmation/promotion/rejection Decision requires Account |
| `delegation.grant`, `delegation.modify`, `delegation.revoke` | Delegation governance actions create Decisions with Accounts |
| `graduation.advance`, `graduation.auto_regress`, `graduation.human_regress` | Graduation changes create Decisions with Accounts |
| `iq.resolve`, `iq.abandon` | Resolution and abandonment are Decisions requiring Accounts |
| `signal.resolve`, `signal.dismiss` | Signal resolution/dismissal creates Decisions with Accounts |
| `constraint.grant_exception` | Override Decision requires Account |
| `orchestration.classify`, `orchestration.select`, `orchestration.confirm` | Classification, selection, and confirmation create Decisions |
| `kpi.introduce`, `kpi.retire` | KPI lifecycle events are Decisions requiring Accounts |
| `actor.activate`, `actor.suspend`, `actor.reinstate`, `actor.deactivate`, `actor.archive` | Actor lifecycle transitions create Decisions with Accounts |

### §27.9.5 B5 — Authority Must Be Traceable

Every Authority must be root or link through a delegation chain terminating at root. Prevents untraceable authority.

**Two shapes with different enforcement profiles:**

**B5-immediate (Pass 1, Always Blocking).** Shape `dlps:B5-AuthorityTraceable`. Checks: `isRootAuthority = true` OR `delegated_from` present. O(1).

**B5-T (Pass 2, Default Blocking).** Shape `dlps:B5-AuthorityChainReachesRoot`. Checks: transitive closure to root; detects circular delegation. O(chain depth). May be downgraded to Warning during bootstrapping only — not below Logging.

### Table 27.9.5: B5 Triggering Operations

| Operation | Trigger Condition |
|---|---|
| `authority.create` | Validates root designation or parent chain |
| `authority.delegate` | Validates scope subset and chain extension |
| `delegation.grant`, `delegation.modify` | Validates delegation does not break chain |
| `authority.revoke` | Validates cascade does not orphan downstream authorities |
| `decision.create` | Validates `authority_source` is traceable |
| `constraint.grant_exception` | Validates overriding authority is traceable |
| `portfolio.spawn_instance` | Validates child instance authority derives from parent |
| `portfolio.relax_constraint` | Validates relaxation authority is traceable |
| `evidence.promote` | Validates promoting authority is traceable |

**Profile variability.** EAS and BAS: B5-T is Blocking (Warning during bootstrap only). PAS: B5-T is Warning.

### §27.9.6 B6 — Constraint Binds Primitives

Every Constraint must target at least one primitive instance and specify an enforcement mode.

**Two shapes:**

**B6-binding (Pass 1, Always Blocking).** Shape `dlps:B6-ConstraintBindsPrimitives`. Checks: `applies_to` non-empty AND `enforcement_mode` in vocabulary. O(1).

**B6-U (Pass 2, Default Warning).** Shape `dlps:B6-ConstraintUniversality`. Checks: universal Constraint targets all instances of governed class. O(instances of target class). Override available via §5.5.

### Table 27.9.6: B6 Triggering Operations

| Operation | Trigger Condition |
|---|---|
| `constraint.create` | Validates binding and enforcement mode |
| `constraint.update` | Validates binding not removed |
| `constraint.grant_exception` | Validates exception does not violate universality requirements |
| `gov_activation.activate_sources` | Validates activated sources bind to primitives |
| `gov_activation.propagate_changes` | Validates propagated changes maintain bindings |
| `portfolio.spawn_instance` | Validates child constraints bind to child primitives |
| `portfolio.tighten_constraint` | Validates tightened constraint maintains binding |
| `orchestration.activate_constraints` | Validates activated constraints bind to work context |

**Profile variability.** B6-binding: Blocking at all tiers. B6-U: EAS = Warning (may upgrade for regulatory domains), BAS = Warning, PAS = Logging.

### §27.9.7 B7 — All Objects Flaggable

Every primitive class (all nineteen) must have a signal attachment surface. Prevents invisible problems.

**Shape:** `dlps:B7-SignalSurfaceCoverage`
**Pass:** Schema Pass (deployment-time). O(19).
**Override:** None. Deployment is halted until corrected.

### Table 27.9.7: B7 Triggering Events

| Event | Trigger Condition |
|---|---|
| Schema deployment | Validates all nineteen primitive classes have signal attachment surface |
| Schema modification | Re-validates signal surface coverage after any class or property change |
| `signal.create` | Validates target primitive class is in the flaggable set |
| Tier activation (§4.7) | Validates newly activated tier classes have signal surfaces |
| `env_interface.receive_signal` | Validates inbound signal targets a flaggable primitive class |
| `intent.block`, `work.block`, `activation.block` | Validates blocking event creates signal on the blocked primitive |

**Profile variability.** Blocking at all profiles. EAS: all nineteen classes. BAS: Tiers 1–3 minimum. PAS: Tiers 1–2 minimum.

### §27.9.8 B8 — Signals Route to Authority

Every Signal must route to an authority on the governance chain for the flagged object.

**Two components:**

**B8-core (Pass 1, Always Blocking).** Shape `dlps:B8-SignalsRouteToAuthority` (core). Checks: `routed_to` is not null and resolves to Authority. O(1).

**B8-routing (Pass 2, Default Warning).** Shape `dlps:B8-SignalsRouteToAuthority` (SPARQL). Checks: target is on the flagged object's authority chain at any depth. O(chain depth). Routing outside the chain is a violation; routing to a higher level on the correct chain is valid.

### Table 27.9.8: B8 Triggering Operations

| Operation | Trigger Condition |
|---|---|
| `signal.route` | Validates target is on governance chain |
| `signal.reroute` | Validates new target stays on governance chain |
| `signal.escalate` | Validates escalation target is next higher on chain |
| `agent.alert` | Validates alert routes to authority |
| `authority.revoke`, `authority.suspend` | Validates active signals are re-routed |
| `delegation.revoke`, `delegation.expire` | Validates signals routed through affected delegation are re-routed |

**Profile variability.** B8-core: Blocking at all tiers. B8-routing: EAS = Warning (may upgrade for regulated domains), BAS = Warning, PAS = Logging.

### §27.9.9 B9 — IQ Resolution Creates Decision

Every resolved IQ must produce a Decision or Work item. Prevents inert emergence.

**Shape:** `dlps:B9-IQResolutionCreatesDecision`
**Pass:** 2 (SPARQL). Default Warning. O(1) per resolved IQ.
**Override:** Available via §5.5, rarely needed since abandonment (a Decision to not act) satisfies B9.

### Table 27.9.9: B9 Triggering Operations

| Operation | Trigger Condition |
|---|---|
| `iq.resolve` | Validates resolution produces Decision or Work |
| `iq.abandon` | Validates abandonment Decision exists (abandonment is a Decision) |
| `iq.promote` | Validates promoted findings link to governance action |
| `iq.revert` | Reverted IQ must eventually re-resolve |

**Profile variability.** EAS = Warning (may upgrade to Blocking for high-governance domains), BAS = Warning, PAS = Logging.

### §27.9.10 Enforcement Summary

### Table 27.9.10: Invariant Enforcement Profile Matrix

| Invariant | Pass | EAS | BAS | PAS | Override |
|---|---|---|---|---|---|
| B1 | 1 | Blocking | Blocking | Blocking | None |
| B2 | 1 | Blocking | Blocking | Blocking | None |
| B3 | 1 | Blocking | Blocking | Blocking | None |
| B4 | 1 | Blocking | Blocking | Blocking | None |
| B5-immediate | 1 | Blocking | Blocking | Blocking | None |
| B5-T | 2 | Blocking | Blocking | Warning | §5.5 (bootstrap only) |
| B6-binding | 1 | Blocking | Blocking | Blocking | None |
| B6-U | 2 | Warning | Warning | Logging | §5.5 |
| B7 | Schema | Blocking | Blocking | Blocking | None |
| B8-core | 1 | Blocking | Blocking | Blocking | None |
| B8-routing | 2 | Warning | Warning | Logging | §5.5 |
| B9 | 2 | Warning | Warning | Logging | §5.5 |

**Reading this table.** Pass 1 shapes are universally Blocking — no profile may downgrade them. Pass 2 shapes decrease in strictness from EAS through PAS. No invariant may be downgraded below Logging for any profile. The Schema Pass (B7) runs at deployment time and is universally Blocking.

---

## §27.10 SDK Constraints

### Table 27.10.1: §27 SDK Constraints

| ID | Constraint | Type | Rationale |
|---|---|---|---|
| §27-C1 | Every write operation MUST check all five authorization gates (§27.1.2) before execution | MUST | Authorization is the foundation of governance integrity; partial checks create exploitable gaps |
| §27-C2 | Every operation that produces a record MUST assign truth type at creation time | MUST | B3 requires truth type on all Evidence; truth type tracking on other primitives enables provenance auditing |
| §27-C3 | All AI actor write operations MUST produce records with truth type = Derived | MUST | AI-Cannot-Be-Principal; truth type boundary prevents epistemic corruption (§6) |
| §27-C4 | Truth type promotion MUST follow monotonic order: Derived → Declared → Authoritative | MUST | Demotion would invalidate governance decisions made on the basis of the higher truth type |
| §27-C5 | Composite operations MUST be atomic — complete entirely or fail entirely | MUST | Partial completion creates structurally invalid governance state |
| §27-C6 | Authoritative records MUST NOT be modified after creation | MUST NOT | Append-only invariant preserves audit trail integrity |
| §27-C7 | Derived records MUST NOT be manually edited | MUST NOT | Derived content must be recomputable from sources; manual edits break reproducibility |
| §27-C8 | Operations MUST NOT hard-delete records from the governance graph | MUST NOT | Soft-delete (status → archived) preserves governance lineage; hard deletes destroy audit trail |
| §27-C9 | AI actors MUST NOT hold the Accountable (A) role, create root Authority, delegate Authority, promote truth types, or resolve Interpretations | MUST NOT | AI-Cannot-Be-Principal (§22.3); these operations require human judgment and accountability |
| §27-C10 | AI actors MUST NOT modify their own delegation scope | MUST NOT | Self-modification of delegation boundaries violates principal-agent separation (SM-5) |
| §27-C11 | Pass 1 invariant shapes MUST NOT be downgraded below Blocking for any profile | MUST NOT | Pass 1 shapes enforce structural properties that cannot be deferred without governance failure |
| §27-C12 | Invariant enforcement MUST NOT be downgraded below Logging for any profile | MUST NOT | Even the lightest enforcement mode must preserve audit visibility |
| §27-C13 | B7 schema validation MUST pass before any instance data is written | MUST | Schema shapes validate the signal surface; instance data without signal surfaces violates B7 |
| §27-C14 | The two-pass validation strategy MUST execute after write but before transaction commit | MUST | Pre-commit validation prevents structurally invalid state from becoming persistent |
| §27-C15 | Specific transport protocol (REST, gRPC, GraphQL, message queue) for operation invocation | DESIGN SPACE | Protocol choice depends on deployment topology, latency requirements, and client ecosystem |
| §27-C16 | Client SDK language bindings and ergonomic API surface | DESIGN SPACE | Language choice and API style depend on implementation team and deployment environment |
| §27-C17 | Atomicity mechanism for composite operations (database transactions, saga pattern, event sourcing) | DESIGN SPACE | Multiple valid approaches exist; choice depends on storage layer and consistency requirements |
| §27-C18 | Concurrency control strategy for conflicting write operations | DESIGN SPACE | Optimistic vs. pessimistic locking, conflict resolution policy, and retry strategy are implementation decisions |
| §27-C19 | Authorization cache strategy for the five-gate check | DESIGN SPACE | Caching delegation chains and role mappings improves performance but introduces staleness risk |
| §27-C20 | Pass 2 invariant evaluation scheduling (synchronous, asynchronous, batch) | DESIGN SPACE | Schedule depends on profile tier, operation volume, and acceptable latency for governance feedback |
| §27-C21 | Event notification mechanism for operation completion and invariant violations | DESIGN SPACE | Webhooks, message queues, server-sent events, and polling are all valid approaches |
| §27-C22 | Audit log storage format and retention policy | DESIGN SPACE | Storage format must preserve all fields from Account and invariant violation records; retention policy is deployment-specific |

---

## §27.11 Terminology

| Term | Definition |
|---|---|
| **MVR (Minimum Viable Record)** | The minimal set of fields required to create a valid primitive instance; defined per-primitive in §26 |
| **Operation Type** | One of six structural categories — Create, Read, Update, Delete, Transition, Composite — that classifies every operation in the catalog (§27.1.1) |
| **Truth Type** | Classification of epistemic status — Authoritative, Declared, Derived, or Opaque — assigned at record creation (or system-assigned for Opaque) and governed by monotonic promotion rules (§6). Opaque is boundary-enforced and does not participate in the standard promotion path. |
| **Authorization Gate** | One of five sequential checks — Authority scope, Authority traceability, RACIVG role, Graduation gate, Profile gate — that every write operation must pass (§27.1.2) |
| **Composite Operation** | A multi-primitive operation that executes atomically within a single logical transaction; partial completion is not a valid state |
| **Pass 1 / Pass 2** | The two-pass SHACL validation strategy: Pass 1 (Core, single-node, O(1), always Blocking) and Pass 2 (SPARQL, graph traversal, profile-configurable) |
| **Schema Pass** | Shapes-on-shapes validation that runs at deployment time; enforces B7 signal surface coverage across all nineteen primitive classes |
| **Graduation Stage** | Per-scope AI maturity level — A, A+, B, C — that gates operation availability; advances monotonically one stage at a time (§8, §A1) |
| **Tighten-Only Cascade** | The constraint monotonicity rule: child instances may only tighten inherited parent constraints, never relax them |
| **AI-Cannot-Be-Principal** | The governance boundary (§22.3) prohibiting AI actors from holding Accountable (A) role, creating root Authority, delegating Authority, promoting truth types, or resolving Interpretations |
| **Soft-Delete** | Transition of `record_lifecycle_state` to `archived`; no records are hard-deleted from the governance graph |
| **Supersession** | The mechanism for correcting immutable records: a new record with `supersedes_ref` pointing to the original; the original is preserved unchanged |

---

## §27.12 Cross-References

| Section | Relationship |
|---|---|
| §4 Primitive Taxonomy | Defines the nineteen primitives and five-tier architecture that §27 operations act upon |
| §5 Behavioral Invariants | Defines B1–B10 invariants enforced by §27.9 |
| §6 Truth Type Model | Governs truth type assignment, immutability, and promotion rules applied across all operations |
| §8 Graduation Model | Defines per-scope graduation stages that gate operation availability (§27.8.3) |
| §12 Governance Activation Pipeline | Defines the six-stage pipeline operationalized by §27.7.4 |
| §14–§15 Signal Architecture | Defines signal semantics operationalized by §27.7.1 |
| §16 IQ Architecture | Defines investigative query semantics operationalized by §27.7.2 |
| §17 VP Architecture | Defines validated projection semantics operationalized by §27.7.3 |
| §18–§19 Knowledge Integration | Defines ontology alignment pipeline operationalized by §27.7.10 |
| §21 Substrate Profiles | Defines EAS/BAS/PAS profile tiers referenced in §27.8.4 |
| §22 Actor Model | Defines actor types, RACIVG roles, and AI-Cannot-Be-Principal boundary |
| §23 Portfolio Patterns | Defines depth-gated visibility and tighten-only constraint cascade |
| §24 Entity Licensing | Defines license structure governing SDK implementation |
| §25 License Terms | Defines license-as-constraint architecture and compliance verification |
| §26 Implementation Schema | Defines the logical data model that §27 operations act upon |
| §A1–§A2 Orchestration Appendix | Defines the seven-step orchestration engine operationalized by §27.7.5 |

---

## Scope

Scope limited to logical operation contracts. Transport protocol, serialization format, and implementation languages are DESIGN SPACE.

## Implementation Requirements

SDK implementations MUST respect all invariant shapes, enforce truth type boundaries, and validate all constraints at specified passes. Specification is immutable during lock period. SHACL two-pass validation is mandatory. Truth type boundaries are preserved through all operations. Session isolation is required for all writes.

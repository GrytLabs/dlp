# §25 License Terms Architecture


The Distributed Ledger Protocol establishes a four-layer licensing model (reference §24) that defines what is licensed at each architectural layer: protocol specification, reference substrate, development tooling, and application layer. This section specifies the architectural principles governing how license terms are structurally represented within the governance substrate, how license scope binds to instances, how license constraints cascade through federation boundaries, and how the ecosystem supports multiple independent paths for adoption.

License terms in DLP are not ancillary legal documents attached to an engineering system. They are **first-class governance objects** — Constraint primitive instances in the governance graph, subject to the same invariant enforcement and governance lineage guarantees that govern all other operational aspects of the substrate. A license agreement is a Constraint instance. License compliance is queryable through the dual-language query layer. License lifecycle transitions produce Decision primitives with full governance lineage. This structural integration is foundational.


## License-as-Constraint Architecture

### Structural Representation

The licensing agreement between the protocol implementer and a licensee is represented as a Constraint primitive with the following properties:

| Constraint Property | License Mapping |
|---|---|
| **Target** | The substrate instance(s) governed by the license |
| **Enforcement mode** | Varies by term — Blocking for hard limits, Warning for approaching limits, Logging for usage tracking |
| **Scope dimensions** | Instance count, profile eligibility, user/actor capacity, verification obligations |
| **Source authority** | The protocol maintainer as licensor |
| **Lifecycle** | Proposed → Active → Modified → Suspended → Revoked → Expired |

The Constraint primitive's enforcement modes apply directly to license terms:

- **Blocking**: Hard license limits that prevent operation beyond scope (e.g., instance count exceeded)
- **Warning**: Alerts for approaching license limits or expiration
- **Logging**: Usage tracking for compliance visibility
- **Advisory**: Optimization suggestions

### License Lifecycle

Licenses transition through defined lifecycle states. Every transition produces a Decision primitive with full governance lineage:

| State | Meaning | Status |
|---|---|---|
| **Proposed** | License terms under negotiation | Constraint not yet active |
| **Active** | License in force; terms enforceable | All enforcement modes operational |
| **Modified** | License terms changed (expansion, tier change) | New terms replace prior with lineage |
| **Suspended** | License temporarily non-operational | Logging mode only |
| **Revoked** | License terminated for cause | All operations blocked; data preserved |
| **Expired** | License term completed without renewal | Read-only mode; data preserved |

Every state transition records the transition authority, rationale, and effective date through the governance record. License revocation triggers read-only mode — governance data is preserved and remains exportable through the interchange layer, but no new governance operations proceed. This provides enforcement without data destruction.

### Queryability

Because license terms are Constraint primitive instances, they participate in the governance graph and are queryable through the dual-language query layer (reference §28):

**Graph traversal** (via Cypher): Traverse from a substrate instance to its governing license constraints, from constraints to their source authority, and from constraints to cascade targets across federation boundaries. Graph traversal answers structural questions: Which constraints govern this instance? What is the federation constraint chain? Which instances share constraint sources?

**Relational aggregation** (via SQL): Aggregate license scope utilization — active instance count versus licensed count, actor capacity versus licensed capacity, profile deployment patterns. Relational queries answer operational questions: What percentage of licensed capacity is consumed? When does the license term expire?

**Cross-instance federation queries**: In federated deployments, license constraints are visible at federation boundaries. A parent organization can traverse to federation licensing constraints governing a federated practitioner — the constraint structure is visible even when the practitioner's governance data is depth-gated (reference §23.4). This visibility distinction is precise: the parent sees the constraint terms, not the constrained practitioner data.

### Constraint Layer Precedence

License constraints compose with other constraints in the governance graph. A substrate instance is simultaneously governed by:

- **Protocol constraints** — the behavioral invariants (B1–B10, §5), universal and non-negotiable
- **License constraints** — the licensing agreement terms, varying by license type and scope
- **Profile constraints** — the profile configuration (§21), activating specific governance features
- **Federation constraints** — if federated, the federation licensing agreement terms
- **Organizational constraints** — the licensee's own governance policies

When constraints from different sources apply to the same primitive, enforcement mode hierarchy resolves conflicts: **Blocking > Warning > Logging > Advisory**. A license constraint cannot downgrade a protocol invariant. An organizational policy cannot downgrade a license constraint. This monotonically tightening composition prevents constraints from being circumvented through policy layering.


## License Scope and Instance Binding

### Per-Instance Licensing Model

Every active substrate instance consumes exactly one license allocation. Instance binding is non-transferable. The license allocation bound to Instance A cannot be transferred to Instance B — Instance A must be archived (releasing the allocation) and Instance B must consume a new allocation. This prevents license accounting ambiguity.

| Instance Event | License Effect |
|---|---|
| **Genesis** (instance creation) | New allocation consumed; active instance count increments |
| **Spawning** (child instance creation) | New allocation consumed; child inherits parent license relationship through constraints |
| **Archival** | Allocation released; instance count decrements; data preserved |
| **Graduation** (profile elevation) | New instance created for elevated profile; new allocation required |

### Profile Scope Licensing

The profile scope dimension of the license governs which profile types (EAS, BAS, PAS) the licensee may deploy. Profile scope recognizes that profiles differ in governance feature depth, content scope, and accountability complexity:

- **PAS** (Project Accountability Substrate): Simplified governance, delegated authority from parent instance. PAS instances inherit licensing from their parent instance.
- **BAS** (Business Accountability Substrate): Operational governance with full role definitions and lifecycle.
- **EAS** (Enterprise Accountability Substrate): Complete governance stack with oversight and advisory authorities. EAS profile scope is a distinct licensing dimension.

PAS instances are licensed as children of their parent instance and do not require separate profile scope authorization. EAS profile scope is an explicit license dimension due to the extended governance features and content it enables.

### Tighten-Only Cascade

License constraints cascade through instance topologies and federation boundaries. When a licensed instance spawns child instances, the parent's license constraints cascade to children. Children may add constraints but cannot relax any constraint imposed by the parent's license.

This cascade operates identically across internal (parent spawning child) and external (federation) boundaries. The tighten-only rule is the architectural guarantee that license terms cannot be circumvented through instance topology — spawning a child instance does not create governance space outside the license constraint envelope.

Examples of constraint cascade:

| Parent Constraint | Child Instance | Cascade Behavior |
|---|---|---|
| Maximum actor capacity limit | Spawned child | Child bound to same or lower limit; may set stricter limit |
| Profile type required for operations | Spawned child | Child operates within its profile; more advanced operations require parent context |
| Verification requirement | Spawned child | Child must independently maintain verification |

The cascade is computable: any instance's effective license constraint set is deterministically derived from the ancestor chain — the intersection of all ancestor constraints plus its own.


## Federation License Architecture

### Federation as License Boundary

Federation is the recursive viable system pattern (§23.1) operating across legal entity boundaries. The federation licensing agreement is a Constraint primitive that spans the boundary, structurally identical to any other constraint in the governance graph but carrying the legal semantics of a relationship between separate entities.

Federation licensing agreements specify:

- **Methodology constraints**: The federated practitioner follows the parent organization's licensed methodology. Constraints cascade from parent to practitioner and onward to practitioner's child instances.
- **Quality constraints**: The practitioner meets the parent organization's quality standards, verifiable through conformance testing machinery (reference §24.5).
- **Reporting obligations**: Aggregate metrics flow upward; depth-gating ensures individual engagement detail remains private to the practitioner.
- **Depth-gating rules**: Parent organization sees aggregate performance across practitioners, not individual engagement data. Enforced architecturally through depth-gated visibility (§23.4), not procedurally.

The federation agreement is a composed structure of primitives:

```
Federation(parent → practitioner) =
    Intent     (relationship purpose)
  + Authority  (delegated methodology authority, scoped)
  + Constraint (federation licensing constraints)
  + Commitment (mutual obligations)
  + Work       (operational scope)
  + Account    (relationship context and history)
```

Every lifecycle event in the federation relationship — creation, modification, termination — produces a Decision primitive with full governance lineage.

### Federation Tiers

The three federation licensing tiers represent progressively deeper constraint integration. Each tier adds constraint categories to the federation licensing agreement:

| Constraint Category | Tier 1: Methodology Only | Tier 2: Methodology + Corpus | Tier 3: Full Federation |
|---|---|---|---|
| **Methodology adherence** | Blocking | Blocking | Blocking |
| **Quality standards** | Not applicable | Warning | Blocking |
| **Infrastructure standards** | Not applicable | Not applicable | Blocking |
| **Reporting obligations** | Basic attestation | Compliance + usage | Full aggregate metrics |

At Tier 1, the practitioner follows the parent's methodology (Blocking) while other integrations are not applicable. At Tier 3, most categories operate as Blocking — reflecting deep operational coupling between parent and practitioner.

### Federation License Independence

Federation licensing does not create dependency between the parent organization's license and the practitioner's license. Each entity holds an independent license relationship with the protocol maintainer.

- If parent organization's license is revoked: Federation constraints lose authority; the agreement terminates. Practitioner's license remains unaffected.
- If practitioner's license is revoked: Practitioner loses substrate access. Federation relationship terminates. Parent's license is unaffected.

This independence model prevents cascading license failures. Each entity's license stands on its own credentials.

### Recursive Federation

Federation mirrors the recursive viable system pattern. A parent organization can federate with a practitioner who federates with a sub-practitioner. Each federation boundary is a licensing agreement operating as a Constraint primitive:

```
Parent Org (EAS) ──license constraint──→ Practitioner (EAS) ──license constraint──→ Sub-Practitioner (EAS)
```

The sub-practitioner's effective license constraint set is the intersection of:
- Protocol maintainer's base license terms
- Parent organization's federation constraints
- Practitioner's federation constraints
- Sub-practitioner's own constraints

No federation level can relax any constraint imposed at a higher level. Depth emerges from organizational viability, not from protocol limit.

### Federation Verification

Each entity in the federation chain independently maintains its own verification status. The parent organization can verify that practitioners maintain their verification status without accessing practitioner governance data.

Verification mechanisms at the federation boundary:

- Parent organization queries the public registry (if practitioners have published their attestations)
- Practitioners share compliance certificates with parent organization
- Parent organization verifies practitioner verification status, not governance data
- The federation licensing tier determines reporting obligation: at Tier 1, compliance attestation is sufficient; at Tier 3, practitioners must maintain verification as a Blocking constraint


## Ecosystem Growth Model

### Three Entry Paths

Three paths lead into the DLP ecosystem. Each path has distinct requirements and value propositions:

| Path | Entry Action | License Required | Value Proposition |
|---|---|---|---|
| **Build from specification** | Implement DLP from open protocol specification | None | Full architectural freedom; independent implementation grows ecosystem |
| **Deploy reference substrate** | License and deploy reference substrate implementation | Substrate license | Production-ready runtime with governance content packages and operational machinery |
| **Build with tooling** | Subscribe to development tooling; deploy on licensed substrate | Substrate + tooling | Full platform experience with development tools, governance interface, education, federation support |

### Path Independence and Transitions

The three paths are independent. An organization building from the open specification can later acquire a substrate license without discarding existing work. An organization deploying the substrate can add tooling support independently. Each path can transition to others without architectural barriers:

- **Build from spec → Deploy reference substrate**: License relationship established, UUID watermarking activated, certificate chain issued. Existing specification-based implementation may be migrated or run alongside; governance data remains portable through interchange layer.
- **Deploy substrate → Add tooling**: Tooling subscription adds development and operational surface. Substrate license and runtime unchanged.
- **Any path → Federation**: Federation agreement layers on top. Independent license unchanged.

### Interoperability Across Implementations

The open protocol specification enables interoperability between licensed substrate instances and independent DLP implementations. The interchange layer (§19) provides data portability surface. Governance data produced by any conformant implementation — whether licensed or independent — is structurally compatible at the protocol level because all conformant implementations share the same nineteen primitives, ten behavioral invariants, and state transformation model.

Interoperability does not require license alignment. A licensed instance can exchange governance data with an independent implementation through the interchange layer. Attestation metadata embedded in licensed exports provides provenance information that independent implementations can use for trust evaluation — but attestation is informational, not prerequisite for data exchange.


## License Compliance Architecture

### Compliance as Integrated Operation

License compliance is not a separate system layered on top of the substrate. It is a governance operation embedded in the substrate lifecycle:

- License constraints are Constraint primitives
- Compliance verification produces Evidence records
- Compliance decisions are Decision primitives with Account context
- Compliance integrates with instance initialization, operation, and lifecycle transitions

The compliance architecture has three components: conformance testing (architectural soundness), invariant compliance (behavioral correctness), and license scope verification (scope adherence).

### Conformance Testing

Conformance testing verifies that an implementation correctly realizes the DLP architecture. The conformance test suite operates across three levels:

| Level | What Is Tested |
|---|---|
| **Structural** | Primitive completeness — all nineteen primitives present and correctly typed |
| **Behavioral** | Invariant compliance — all ten behavioral invariants enforced |
| **Compositional** | Primitive composition — cross-primitive relationships compose correctly |

All three levels must pass for verification designation. Conformance testing is both a verification mechanism and a license obligation for licensed implementations.

### Scope Verification

License scope verification is the operational compliance component. The substrate validates its deployment against the contracted scope dimensions through local verification:

**Scope dimensions verified:**
- Instance count (active instances versus licensed maximum)
- Profile scope (profile type deployment within license)
- Actor capacity (active actor contexts within licensed limit)
- Verification obligation (certificate chain validity and status)

Verification occurs at instance creation, profile graduation, actor context creation, and at configurable intervals during operation. Violation responses follow constraint enforcement modes: Blocking prevents the operation, Warning alerts, Logging records silently.

License scope verification operates locally. The substrate tracks its scope dimensions against license Constraint primitives instantiated in the governance graph. No external call to the protocol maintainer is required — the license terms are self-contained within the governance graph.

### Compliance Lifecycle Integration

License compliance integrates throughout the substrate's operational lifecycle:

- **At instance creation (Genesis)**: License credentials validated, certificate chain established, UUID watermarking activated, license constraints instantiated in governance graph.
- **During operation**: License scope dimensions continuously validated. Constraint violations produce Evidence records. Warning violations generate alerts.
- **At data export (interchange)**: Every export includes attestation metadata from the certificate chain. Export is the moment license provenance becomes externally visible.
- **At profile graduation**: License scope verification triggered. New instance requires allocation. Target profile must be within licensed scope. Graduation decision is a governed event with full lineage.
- **At renewal or expiration**: License term expiration triggers lifecycle transition. Substrate transitions to read-only if not renewed. Governance data preserved. Compliance record persists for audit regardless of license status.


## Scope

The license terms architecture constrains the following:

1. **Structural representation**: All license terms must be represented as Constraint primitive instances in the governance graph, not external legal documents.

2. **Non-transferable binding**: Each active instance consumes exactly one license allocation. Allocations cannot be transferred between instances.

3. **Tighten-only cascade**: License constraints must cascade through federation boundaries and spawning relationships following the tighten-only rule. No child instance may relax any parent constraint.

4. **Enforcement universality**: The same license constraint enforces identically for all licensees holding that constraint. Conservation law prevents selective application.

5. **Data preservation**: License revocation or expiration must not destroy governance data. The substrate transitions to read-only state but preserves all records for export and audit.

6. **Independent license paths**: The three ecosystem entry paths must remain independent and non-coercive. Organizations must be able to enter and transition freely.

7. **Integrated compliance**: License compliance is a governance operation embedded in the substrate lifecycle, not a separate audit system.


## Locked Design Positions

The following design decisions are fixed:

1. **License-as-Constraint is structural, not metaphorical.** License terms are Constraint primitive instances in the governance graph, enforceable through the same mechanisms as all other governance constraints.

2. **Instance binding is non-transferable.** Each active instance consumes exactly one license allocation.

3. **Tighten-only cascade is mandatory.** License constraints cascade to all descendant instances following the tighten-only rule. No exception for federation boundaries.

4. **Three ecosystem paths are permanent.** Open specification, reference runtime, and tooling paths must remain independent and non-coercive.

5. **Compliance is integrated, not layered.** License compliance operates through the governance machinery, not as a separate system.


## Related Sections

- **§24 Entity Licensing Structure**: Four-layer licensing model, entity structure, license types, verification architecture, profile graduation
- **§4 Constraint Primitive**: License terms operate through Constraint (§4.1)
- **§5 Behavioral Invariants**: B1–B10 apply to license constraints; shapes graph implementation
- **§23 Portfolio Patterns**: Spawning, graduation, federation relationships that trigger licensing events
- **§28 Dual-Language Query Layer**: License constraints queryable through Cypher and SQL
- **§19 Interchange Layer**: Data portability across implementations and license boundaries


*End of §25 License Terms Architecture (L0 Public Specification)*

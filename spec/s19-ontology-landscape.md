# §19 Ontology Landscape & Pre-Loading

The substrate does not operate in an ontological vacuum. Organizations work within established knowledge frameworks — legal ontologies that formalize obligation and permission, financial ontologies that model agreements and instruments, organizational ontologies that structure authority and membership, provenance models that track attribution and derivation. These frameworks represent decades of domain formalization. The substrate must align with them, not replace them.

§18 specifies the integration engine — the pipeline mechanics by which external concepts are matched, proposed, reviewed, and graduated into the substrate's governance model. This section specifies what that engine processes: which ontology families the substrate recognizes, how their concepts align to the nineteen-primitive model (§4), what gets pre-loaded versus discovered at runtime, and how concepts from different ontology families relate to each other through explicit bridge semantics.

An SDK team reads §18 to understand the alignment pipeline and §19 to understand the content that pipeline processes.

Pre-loading serves three architectural purposes.

**Alignment bootstrapping.** A substrate instance deployed without ontology content cannot align incoming organizational data to primitives. Pre-loaded concept signatures provide the initial alignment surface — the reference vocabulary against which the integration engine's matchers (§18) compare incoming field descriptors. Without pre-loaded signatures, every alignment proposal starts from zero context.

**Cross-ontology coherence.** Organizations operate across ontology boundaries. A financial services firm's data simultaneously touches FIBO (financial instruments), LKIF (regulatory obligations), and W3C ORG (organizational structure). Pre-loaded bridge definitions establish the cross-ontology relationships that enable coherent alignment when incoming data spans multiple knowledge frameworks.

**Authority grounding.** Each recognized ontology family has a standards authority, a maintenance community, and a scope boundary. The source registry records this metadata, enabling the integration engine to distinguish authoritative ontology sources from informal vocabularies and to route alignment proposals through appropriate review channels.

### §19.1 Ontology Pre-Loading Purpose

Three protocol layers separate the ontology alignment concept from its implementation.

#### Table 19.1.1: Protocol Layer Placement

| Protocol Layer | Ontology Content | Governance |
|---|---|---|
| **DLP-Core** | The nineteen primitive definitions that alignment targets. Primitive semantics are the alignment destination — external ontology concepts map to primitives, not the reverse. | Open specification — immutable after lock |
| **DLP-World-Model** | The alignment architecture: ontology landscape recognition (§19), integration pipeline mechanics (§18), bridge semantics, pre-loading strategy. | Open specification — evolution governed |
| **DLP Substrate** | Specific alignment implementations: matcher configurations, pre-load inventory per deployment profile, concept registry seed data, bridge map population. | Licensed — profile-specific and extensible within Constraint bounds |

### §19.2 Ontology Family Catalog

The substrate recognizes ten ontology families at launch. This catalog is not exhaustive — additional families enter through the Namespace governance model (§4.6), beginning at sandbox scope and graduating upward through evidence accumulation and authority approval.

#### Table 19.2.1: Recognized Ontology Families

| Family | Authority | Scope | Primary Primitive Alignment | Pre-Loaded |
|---|---|---|---|---|
| **BFO** | ISO/IEC 21838-2 | Upper ontology — meta-level categories for any domain | Work (Process), Capacity (Disposition/Role), Evidence (Generically Dependent Continuant) | Yes |
| **DOLCE** | ISO/IEC 21838-3 | Foundational — cognitive and linguistic view of reality | Commitment (Social Object), Intent (Non-Physical Endurant), Work (Accomplishment), Decision (Achievement) | Yes |
| **SUMO** | IEEE | Upper — comprehensive categories with full lexical mapping | Work (IntentionalProcess), Commitment (Obligation), Authority (Permission) | Yes |
| **FIBO** | OMG / EDM Council | Financial instruments, entities, agreements, processes | Commitment (Agreement), Constraint (ContractualObligation), Authority (Authorization), Entity (LegalEntity) | Yes |
| **LKIF** | ESTRELLA (EU) | Legal rules, norms, deontic concepts | Constraint (Norm), Commitment (Obligation/Duty), Authority (Permission/Right) | Yes |
| **W3C ORG** | W3C Recommendation | Organizational structures, membership, roles | Entity (Organization), Authority (Post/Role), Commitment (Membership) | Yes |
| **PROV-O** | W3C Recommendation | Provenance of any resource | Work (Activity), Authority (Agent), Evidence (Entity), Account (wasAttributedTo) | Yes |
| **OWL-Time** | W3C / OGC | Temporal entities and relationships | Context (TemporalEntity), Cycle (Interval/ProperInterval) | Yes |
| **Schema.org** | W3C Community Group | Web-wide structured data | Work (Action), Evidence (CreativeWork), Entity (Organization/Person) | Yes |
| **Dublin Core** | ISO 15836 / DCMI | Resource description metadata | Evidence (identifier, source), Authority (creator/contributor), Constraint (rights) | Yes |

**Reading this table.** The Primary Primitive Alignment column identifies the strongest semantic correspondences between each ontology family's core concepts and the substrate's primitive model. These are not exhaustive mappings — each family contains concepts that align to multiple primitives across tiers. The full tier-aware alignment model (§19.3) specifies the complete mapping structure.

### §19.3 Primitive Alignment Model

Earlier ontology alignment efforts mapped external concepts to the nine Tier 1 primitives only. The substrate's nineteen-primitive model (§4) requires alignment across all five tiers. Tier 2–5 primitives participate in ontology alignment wherever external knowledge frameworks formalize concepts that correspond to infrastructure, operational, AI-native, or temporal governance concerns.

#### Table 19.3.1: Tier-Aware Alignment

| Tier | Primitives | Representative Ontology Alignments |
|---|---|---|
| **1 — Irreducible Core** | Intent, Commitment, Capacity, Work, Evidence, Decision, Authority, Account, Constraint | BFO Process → Work. DOLCE Social Object → Commitment. LKIF Norm → Constraint. FIBO Agreement → Commitment. SUMO Permission → Authority. PROV Activity → Work. DOLCE Achievement → Decision. BFO Disposition → Capacity. PROV Entity → Evidence. |
| **2 — Unconditional Infrastructure** | Identifier, Entity, Context, Namespace | W3C ORG Organization → Entity. FIBO LegalEntity → Entity. OWL-Time TemporalEntity → Context. PROV (temporal/situational framing) → Context. SKOS ConceptScheme → Namespace. Dublin Core identifier → Identifier. |
| **3 — Governed Operation** | Orientation, Learning, Activation | Goal-modeling frameworks (GRL, i*) → Orientation. Organizational learning ontologies (Argyris/Schön formalization) → Learning. Workflow readiness ontologies → Activation. |
| **4 — AI-Native Extensions** | Interpretation, Environment Interface | Semantic annotation ontologies → Interpretation. NLP interpretation frameworks → Interpretation. DCAT (Data Catalog Vocabulary) → Environment Interface. Schema.org (web endpoint descriptions) → Environment Interface. |
| **5 — Operational Configuration** | Cycle | OWL-Time Interval/ProperInterval → Cycle. Calendaring ontologies (periodic temporal structures) → Cycle. |

**Tier 2 alignment rationale.** Entity aligns to W3C ORG and FIBO organizational models because these ontologies formalize the ownership and membership structures that the Entity primitive governs. Context aligns to OWL-Time and PROV-O because temporal and situational framing is the Context primitive's semantic territory. Namespace aligns to SKOS concept schemes because SKOS provides the most mature model for hierarchical concept organization — the same structural concern that Namespace governs within the substrate.

**Tier 3–5 alignment rationale.** Higher-tier alignments are sparser because fewer external ontologies formalize the governance concerns these primitives address. Orientation, Learning, and Activation capture cognitive and operational governance processes that most knowledge frameworks leave implicit. Interpretation and Environment Interface formalize AI-specific concerns that predate most established ontologies. Cycle aligns to temporal ontologies that model periodicity, but most temporal ontologies focus on instants and intervals rather than repeating governance structures.

**MUST:** Every alignment proposal maps to exactly one primitive. A single external concept aligns to one primitive; if the concept spans multiple governance concerns, it is decomposed before alignment.

**MUST NOT:** Alignment proposals that span multiple primitives without decomposition. An LKIF Legal Action that maps simultaneously to Decision and Work requires decomposition into two alignment entries — one per primitive.

**DESIGN SPACE:** Specific alignment heuristics per ontology family, confidence thresholds per tier, and the weighting between lexical, structural, and semantic matchers are implementation choices within the integration engine (§18).

#### Alignment Provenance

Every alignment entry carries provenance metadata tracking its epistemic origin through the truth type system (§6). The four truth types — Authoritative, Declared, Derived, Opaque (§6.1) — apply to ontology alignment content as follows.

The primitive definitions that alignments target are Authoritative. The nineteen primitives (§4) and their semantic boundaries are protocol-level, append-only, immutable after lock. An alignment maps an external concept to an Authoritative target; the alignment itself is not Authoritative because it represents an assertion about correspondence, not a canonical governance record.

Pre-loaded alignments — those shipped with the substrate — carry truth type Declared. They are human-curated during substrate development, attributed to the curation authority, and versioned. Pre-loaded alignments are canonical: they are used in alignment decisions and inform the integration engine's matcher behavior.

Runtime-discovered alignments — those proposed by the integration engine during operational use — enter as Derived. They land in staging, carry AI attribution and a time-to-live, and require human review before promotion to Declared. The promotion workflow follows the path defined in §6.3: Derived → staging → human review → Declared.

The nine conservation laws (§9) constrain which alignments are valid. An alignment that would produce obligation invariance violations — where the same agreement carries different binding force depending on ontology source — is structurally invalid regardless of matcher confidence. An alignment that would break delegation invariance — where authority scope varies by ontology provenance — violates a conservation law. The conservation laws are not alignment heuristics; they are hard constraints on alignment validity.

### §19.4 Pre-Loading Strategy

The substrate ships with a pre-loaded alignment surface: concept signatures, cross-ontology bridges, and ontology source records. This content provides the integration engine's initial operating context. Everything beyond the pre-loaded set is discovered at runtime through the lazy-load pattern.

#### Table 19.4.1: Pre-Load Inventory

| Component | Approximate Count | Source | Namespace Scope |
|---|---|---|---|
| **Concept signatures** | ~200 | Core concepts from all ten recognized ontology families: upper ontology anchors (~50 from BFO, DOLCE, SUMO), domain pattern signatures (~100 from FIBO, LKIF, W3C ORG), cross-cutting signatures (~50 from PROV-O, OWL-Time, Schema.org, Dublin Core) | Core and Domain |
| **Bridge definitions** | ~50 | Cross-ontology relationship mappings (§19.5) establishing semantic connections between equivalent, subsumptive, and complementary concepts across families | Domain |
| **Source registry entries** | ~15 | Ontology authority records: name, URI, scope, maintaining authority, license, format, and version for each recognized family plus selected extensions | Core |

**Pre-loaded vs. runtime-discovered: the epistemic distinction.** Pre-loaded content is Declared truth type — human-curated, attributed, versioned, and canonical from first deployment. Runtime-discovered content enters as Derived truth type — machine-proposed, staged, ephemeral until promoted. This distinction is not procedural convenience; it reflects the truth type system's (§6) architectural commitment that canonical governance content requires human judgment.

#### Lazy-Load Discovery Pattern

When operational data contains concepts not in the pre-loaded set, the integration engine executes a five-step discovery pattern:

1. **Identify.** The integration engine's matchers (§18) encounter a field descriptor or concept reference that does not match any pre-loaded signature above the confidence threshold.
2. **Fetch.** The source registry identifies which ontology family likely contains the concept. The relevant ontology subset is retrieved on demand — not the full ontology, but the structural neighborhood around the candidate concept.
3. **Propose.** The integration engine generates an alignment proposal: candidate concept → target primitive, with confidence score, matcher evidence, and tier assignment. The proposal enters as Derived truth type.
4. **Confirm.** A human with appropriate authority reviews the proposal. Confirmation promotes the alignment from Derived to Declared. Rejection archives the proposal with a rejection reason, providing feedback signal to the matchers.
5. **Graduate.** The confirmed alignment graduates through Namespace scopes following the claims graduation protocol (§6.5). New concepts enter at sandbox scope, demonstrating stability and utility before advancing to tenant, domain, or core scope.

#### Table 19.4.2: Namespace Governance for Ontology Concepts

The Namespace primitive (§4.6) serves a dual architectural role in ontology alignment. As a Tier 2 primitive, it provides the scoping infrastructure that every substrate deployment requires. As the governance mechanism for ontology concept lifecycle, it controls how alignment content matures from provisional to canonical.

| Namespace Scope | Ontology Concept Governance | Entry Condition | Graduation Condition |
|---|---|---|---|
| **Sandbox** | Time-bounded evaluation. Runtime-discovered concepts enter here. Alignment proposals are usable within the sandbox but do not propagate to broader substrate operations. | Integration engine proposes alignment; human acknowledges for evaluation | Evidence of alignment stability — concept used successfully in multiple alignment operations without correction |
| **Tenant** | Organization-specific adoption. Concepts that have demonstrated value within a specific organizational context. Managed by the organizational steward. | Sandbox graduation — evidence of stability plus organizational steward endorsement | Cross-organizational validation — concept proves useful beyond a single tenant context |
| **Domain** | Curated by domain authorities. Concepts that represent stable knowledge within a recognized domain (financial, legal, organizational). | Tenant graduation — evidence of cross-organizational utility plus domain authority endorsement | Protocol-level recognition — concept represents a fundamental governance alignment |
| **Core** | Protocol-level, immutable after lock. Reserved for alignments that are architecturally necessary — the pre-loaded concept signatures that the substrate ships with. | Architecture decision during substrate development | N/A — core scope content does not graduate further |

Pre-loaded concept signatures enter at core or domain scope — they bypass the graduation pathway because they are curated during substrate development, not discovered at runtime. This is architecturally deliberate: the pre-loaded surface must be available from first deployment without requiring operational evidence accumulation.

**MUST:** Pre-loaded concepts carry truth type Declared with curation attribution. Runtime-discovered concepts enter as Derived and follow the promotion workflow (§6.3) before advancing beyond sandbox scope.

**MUST NOT:** Runtime-discovered concepts enter directly at domain or core scope. The graduation pathway is the only mechanism for scope advancement beyond sandbox.

**DESIGN SPACE:** Specific pre-load inventory per deployment profile (EAS, BAS, PAS). An EAS deployment serving financial services may pre-load a deeper FIBO concept set than a PAS deployment. The pre-load inventory in Table 19.4.1 represents the union across profiles; profile-specific subsets are implementation choices.

### §19.5 Cross-Ontology Bridge Semantics

Organizations operate across ontology boundaries. A regulatory compliance workflow simultaneously touches LKIF (legal obligations), FIBO (financial commitments), and W3C ORG (organizational authority). When the integration engine aligns incoming data to a concept in one ontology family, bridge definitions enable the substrate to recognize equivalent, subsumptive, or complementary concepts in other families — propagating alignment decisions across ontology boundaries without redundant matching.

#### Table 19.5.1: Bridge Relationship Types

| Bridge Type | Semantic | Example | Symmetry |
|---|---|---|---|
| **Equivalent** | Same concept expressed in different ontology vocabularies. Alignment to one implies alignment to the other. | FIBO Agreement ↔ LKIF Obligation — both align to Commitment | Bidirectional |
| **Subsumes** | Source concept is broader than target. Alignment to the source does not imply alignment to the target, but alignment to the target implies alignment to the source. | W3C ORG Organization subsumes FIBO LegalEntity — every legal entity is an organization, but not every organization is a legal entity | Directional |
| **Specializes** | Source concept is narrower than target. The inverse of subsumes. | PROV Entity specializes BFO Generically Dependent Continuant — PROV entities are a specific kind of information artifact | Directional |
| **Overlaps** | Partial semantic overlap. Neither concept fully contains the other, but they share a defined intersection. | PROV Agent overlaps W3C ORG Post — both model actors with responsibilities, but Agent is broader (includes non-organizational actors) and Post is more specific (positional authority) | Bidirectional |
| **Complements** | Different aspect of the same domain. The concepts are not equivalent or subsumptive but together provide a more complete model than either alone. | BFO (structural categories) complements DOLCE (cognitive categories) — together they cover both the structural and cognitive dimensions of organizational reality | Bidirectional |

#### Bridge Propagation

Bridges enable cross-ontology alignment propagation. When the integration engine (§18) confirms an alignment between an incoming field descriptor and a FIBO concept, the bridge map surfaces all bridged concepts in other families:

- An **equivalent** bridge propagates the alignment at equal confidence. If a field aligns to FIBO Agreement with confidence 0.85, the equivalent bridge to LKIF Obligation carries the same confidence.
- A **subsumes** bridge propagates alignment upward at reduced confidence. Alignment to FIBO LegalEntity propagates to W3C ORG Organization at lower confidence because the broader concept is a weaker match.
- A **specializes** bridge propagates alignment downward with a specificity flag. The narrower concept may be a better match, but requires additional evidence.
- An **overlaps** bridge surfaces the related concept as a candidate for review, not as a propagated alignment. Overlap is informational — it alerts the reviewer to related concepts without asserting alignment.
- A **complements** bridge surfaces the complementary concept for enrichment, not alignment. Complementary concepts add context but do not imply primitive mapping equivalence.

The graduation engine (§18) uses bridge propagation semantics when evaluating alignment proposals. A proposal that is independently confirmed through multiple ontology families via equivalent bridges carries stronger evidence than a proposal confirmed through a single family.

#### Bridge Governance

**MUST:** Bridges are explicit. Every bridge is a named, stored relationship with a defined bridge type from the five-type vocabulary.

**MUST:** Every bridge carries provenance: who asserted the bridge, when, the evidence basis for the assertion, and the truth type of the bridge record itself. Pre-loaded bridges are Declared (human-curated). Runtime-proposed bridges enter as Derived and require human confirmation.

**MUST NOT:** Implicit bridge creation through matcher heuristics alone. The integration engine may propose bridges when it detects semantic similarity between concepts in different ontology families, but the proposal follows the same Derived → review → Declared promotion path as any other alignment content.

**MUST NOT:** Bridge propagation override human review. A propagated alignment via an equivalent bridge is a proposal, not a confirmed alignment. Propagation reduces reviewer effort by surfacing high-confidence candidates; it does not bypass the review step.

**DESIGN SPACE:** Bridge confidence decay functions (how much confidence is lost through subsumes/specializes propagation), maximum propagation chain length (whether bridges compose transitively), and bridge-specific reviewer routing are implementation choices.

### §19.6 SDK Constraints Summary

#### Table 19.6.1: §19 SDK Constraints

| Constraint | Type | Rationale |
|---|---|---|
| Every alignment proposal maps to exactly one primitive | MUST | Prevents ambiguous alignments that span multiple governance concerns. Decomposition before alignment ensures clean primitive composition. |
| Alignment proposals that span multiple primitives without decomposition are rejected | MUST NOT | Multi-primitive alignment proposals create unresolvable ambiguity in the governance graph. The primitive model (§4) requires each node to occupy exactly one semantic position. |
| Pre-loaded concepts carry truth type Declared with curation attribution | MUST | Pre-loaded content is human-curated during substrate development. Declared truth type (§6) reflects this epistemic origin and makes the curation authority traceable. |
| Runtime-discovered concepts enter as Derived truth type | MUST | Machine-proposed alignments are provisional. Derived truth type (§6) enforces staging and human review before operational use. |
| Runtime-discovered concepts enter at sandbox Namespace scope | MUST | The Namespace governance model (§4.6) requires evidence accumulation and authority approval for scope advancement. Sandbox is the only valid entry point for unvalidated content. |
| Runtime-discovered concepts bypass the graduation pathway to enter at domain or core scope | MUST NOT | Scope advancement requires operational evidence. Direct entry at elevated scope would circumvent the evidence-based governance model that Namespace enforces. |
| Bridges are explicit, named relationships with defined bridge type and provenance | MUST | Implicit bridges undermine alignment traceability. Every cross-ontology relationship must be auditable — who asserted it, when, and on what evidence basis. |
| Implicit bridge creation through matcher heuristics alone | MUST NOT | Matcher heuristics may propose bridges, but the proposal must follow the Derived → review → Declared promotion path. Heuristic-only bridges lack the human judgment required for canonical cross-ontology relationships. |
| Bridge propagation override human review of propagated alignments | MUST NOT | Propagation is a proposal mechanism, not a confirmation mechanism. Cross-ontology alignment decisions require human judgment regardless of bridge confidence. |
| Alignment provenance tracks truth type through the full lifecycle (Derived → Declared) | MUST | The truth type system (§6) requires every governed object to carry epistemic origin metadata. Alignment entries are governed objects. |
| Conservation laws (§9) constrain alignment validity | MUST | An alignment that would violate obligation invariance, delegation invariance, or any other conservation law is structurally invalid regardless of matcher confidence. |
| Specific alignment heuristics, confidence thresholds per tier, and matcher weighting | DESIGN SPACE | These parameters are integration engine (§18) configuration, not architectural commitments. Different deployment contexts and ontology families may require different tuning. |
| Pre-load inventory per deployment profile (EAS, BAS, PAS) | DESIGN SPACE | Profile-specific pre-loading enables deployment optimization. The union inventory (~200 concepts, ~50 bridges, ~15 registry entries) is the architectural commitment; profile subsets are implementation choices. |
| Bridge confidence decay functions and maximum propagation chain length | DESIGN SPACE | Propagation tuning depends on the specific ontology families bridged and the organizational context. No single decay function or chain limit is architecturally necessary. |
| Whether alignment proposals are batched or processed individually | DESIGN SPACE | Pipeline scheduling is an integration engine (§18) implementation choice. The architectural requirement is that every proposal follows the truth type lifecycle, not that proposals arrive in a specific cadence. |

### §19.7 Ontology Landscape and Domain Boundaries

Ontology concepts do not respect governance domain boundaries. A single external concept — FIBO's ContractualObligation — may touch Domain 2 (Commitment & Obligation) through its binding-force semantics, Domain 3 (Evidence, Record & Truth) through its documentation requirements, and Domain 5 (Access, Identity & Boundary) through its scope restrictions. The seven governance domains (§13) are projection surfaces, not ontology partitions.

The alignment model handles domain-crossing concepts through decomposition. When an external concept maps to a primitive that participates in multiple governance domains (Table 13.11.1), the alignment entry targets the primitive, not the domain. Domain participation follows from the primitive-to-domain mapping already established in §13 — the alignment model does not duplicate or override that mapping.

This separation ensures that ontology alignment decisions do not inadvertently create governance domain commitments. An alignment of FIBO ContractualObligation to the Constraint primitive automatically inherits Constraint's domain participation (Domain 1, Domain 5, Domain 6 per Table 13.11.1) without requiring the alignment to specify domain applicability independently.


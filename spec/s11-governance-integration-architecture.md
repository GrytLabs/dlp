# §11 Governance Integration Architecture

§8 established that governance must be a world model with specific structural subsystems. §9 established that organizational symmetries must be explicitly preserved. §10 mapped the perimeter of where primitive composition has extended. This section asks: given an organization at a specific point in governed space, what external and internal governance frameworks apply, and how does the substrate represent them?

The substrate is a composition layer, not an originator of governance. It does not create governance standards — it maps, registers, and composes them. The Governance Integration Architecture is the structural specification for this composition.

## §11.1 Protocol Placement


The Governance Integration Architecture is a DLP substrate specification. The governance registry, activation engine, and pattern signature database are substrate implementation concerns. The protocol layers (DLP-Core, DLP-World-Model, Substrate) describe where the specification lives. The governance layers (§11.3) describe what kinds of governance the specification composes.

### Table 11.1.1: Protocol Layer Placement

| Protocol Layer | Governance Integration Content | Example |
|---|---|---|
| **DLP-Core** | Primitives that governance patterns compose from | Authority, Evidence, Constraint, Decision — the grammar that governance sources map to |
| **DLP-World-Model** | The structural model specifying how governance frameworks map to primitive-level patterns | §11 defines the composition architecture; §12 operationalizes activation |
| **DLP Substrate** | The governance registry, activation engine, pattern signature database | Source registry, activation rules, gap projection, closure workflow |
| **Derivatives** | Compliance dashboards, gap reports, standards mapping visualizations | Governance heat maps, closure roadmaps, cross-framework coverage reports |

This distinction is critical. The three-layer governance composition model (§11.3) and the three-layer protocol stack are independent structures. The governance layers describe three categories of governance source: external standards, AI-native risk constructs, and operational authorities. The protocol layers describe where the DLP specification lives: primitive grammar, world model concepts, and substrate implementation. A COSO principle (Layer 1 governance) maps to DLP-Core primitives. An ERKIA risk type (Layer 2 governance) operates at DLP-World-Model level. A FinOps capability mapping (Layer 3 governance) is a Substrate-specific reference. Conflating the two layering systems produces architectural confusion — a governance source's layer and its protocol placement are orthogonal classifications.

## §11.2 The Composition Principle


The substrate does not originate governance. It composes governance from three sources: external standards bodies, AI-native risk constructs, and operational authorities. All governance content is represented as pattern signatures — distillations of 5–15 core concepts per source, not imported whole.

Three architectural constraints govern this composition.

**No whole-standard import.** The substrate MUST NOT import entire standards. A governance framework with 1,000 controls becomes 8–15 pattern signatures capturing its architectural concepts. The full control library remains the standard body's artifact. The substrate maps to it; the substrate does not replicate it.

**Pre-load all, activate selectively.** All approximately 200 pattern signatures across all three governance layers are registered unconditionally in the governance registry. Registration is structural — the pattern signature exists in the system and can be referenced. Activation (§12) is profile-assessed — the pattern signature applies to a specific organization based on its governance profile. Registration is always complete. Activation is always partial.

**Composition, not origination.** For Layer 1 and Layer 3 governance, the substrate is the composition layer — it brings disparate frameworks into a unified governance registry. For Layer 2, the protocol contributes AI-native constructs that address gaps in existing frameworks. Even these contributions are compositional: they compose from the same Tier 1 primitives (§4) and enforce the same conservation laws (§9) as all other governance content.

### Table 11.2.1: Registration vs. Activation

| Operation | Scope | Condition | Lifecycle |
|---|---|---|---|
| **Registration** | All ~200 pattern signatures across all three layers | Unconditional — every pattern is registered regardless of organizational context | At substrate deployment; updated when new governance sources are published |
| **Activation** | The applicable subset of registered patterns for a specific organization | Profile-assessed — organizational characteristics determine which patterns apply | At profile assessment (§12); re-evaluated when the profile changes |

## §11.3 Three-Layer Governance Composition


Governance content composes from three layers into the **Governance Integration Domain**. Each layer represents a structurally distinct category of governance source.

### Table 11.3.1: Three-Layer Governance Composition

| Layer | Canonical Name | Content | Pattern Count (approx.) | Example |
|---|---|---|---|---|
| **1** | **External Standards** | Established governance frameworks from standards bodies — adopted and mapped to primitive-level patterns | ~150 | COSO IC 17 principles, NIST CSF 6 functions, ISO 27001 4 control themes |
| **2** | **AI-Native Risk & Oversight** | Governance constructs that exist because AI systems require them — no adequate pre-AI equivalent | ~25 | ERKIA epistemic risk categories, AICAR review task types |
| **3** | **Operational Authorities** | Operational patterns mapped to external regulatory and operational authority frameworks | ~25 | FinOps maturity model, SRE golden signals, SR 11-7 validation principles |

Layer 1 provides the foundational governance architecture — the control frameworks, risk management models, and security standards that apply to any governed organization. Layer 2 extends this foundation with constructs that address AI-specific governance gaps — risks and oversight mechanisms that pre-AI frameworks do not adequately cover. Layer 3 maps operational practices to their external authorities — positioning specific operational patterns within established research and regulatory guidance.

The three layers are additive, not hierarchical. An organization may activate patterns from all three layers simultaneously. Cross-layer relationships exist: ERKIA (Layer 2) extends COSO ERM (Layer 1) by elevating epistemic risks to primary status. AICAR (Layer 2) operationalizes EU AI Act Article 14 human oversight requirements (referenced in Layer 3). FinOps cost governance (Layer 3) maps to COSO IC internal control principles (Layer 1).

## §11.4 Layer 1: External Standards


Layer 1 contains established governance frameworks organized into a three-tier subdivision. The tiers reflect governance dependency: foundational frameworks apply to all governed organizations, risk and security frameworks apply when specific risk domains are active, and specialized frameworks apply to certification or regulatory requirements.

### Table 11.4.1: Layer 1 Tier Structure

| Tier | Name | Content | Examples |
|---|---|---|---|
| **Foundational** | Universal control frameworks | Frameworks applicable to all governed organizations regardless of industry, scale, or regulatory context | COSO IC (17 principles across 5 components), ISO 9001 (7 management system clauses) |
| **Risk & Security** | Risk management and cybersecurity frameworks | Frameworks activated when formal risk management or cybersecurity programs are required | COSO ERM (20 principles across 5 components), NIST CSF 2.0 (6 functions, 106 subcategories), NIST 800-53 Rev 5 (20 control families, ~1,000 controls), ISO 27001 (93 controls across 4 themes) |
| **Specialized** | Domain-specific and certification frameworks | Frameworks activated by specific regulatory requirements, certification targets, or market positioning | ISO 42001 (AI management system), NIST AI RMF (4 functions, 7 trustworthiness characteristics), SOC 2 (5 trust service categories, 33 criteria), FedRAMP (NIST 800-53 baselines for federal authorization) |

Pattern signature counts by tier: Foundational produces approximately 25 signatures, Risk & Security approximately 80, and Specialized approximately 45. Each signature captures 5–15 core concepts from its source framework.

### Table 11.4.2: Layer 1 Cross-Framework Dependencies

The frameworks in Layer 1 compose hierarchically. Higher-level frameworks build on lower-level foundations. The dependency model specifies which frameworks presuppose which.

| Source Framework | Depends On | Relationship |
|---|---|---|
| COSO IC | — | Foundation — no Layer 1 dependency |
| ISO 9001 | — | Quality management foundation — independent entry point |
| COSO ERM | COSO IC | Enterprise risk management presupposes foundational internal controls |
| NIST CSF 2.0 | — | Cybersecurity governance — independent entry point |
| NIST 800-53 | NIST CSF 2.0 | Prescriptive controls implement CSF outcomes (subcategory-to-control mapping) |
| ISO 27001 | — | Information security management — independent entry point; cross-mapping to NIST 800-53 exists |
| ISO 42001 | ISO 27001 | AI management system integrates with information security management via shared Annex SL structure |
| NIST AI RMF | — | AI risk governance — independent entry point at top of AI governance stack |
| SOC 2 | COSO IC | Trust service common criteria derive from COSO internal control components |
| FedRAMP | NIST 800-53 | Federal authorization selects and configures 800-53 control baselines |

Dependency means that activating the source framework's patterns is architecturally more effective when the dependency framework's patterns are already active. It does not mean the source framework cannot be activated independently — it means governance coverage is richer when dependencies are satisfied. The activation engine (§12) uses these dependencies for closure roadmap prioritization: foundational gaps are addressed before dependent framework gaps.

## §11.5 Layer 2: AI-Native Risk & Oversight


Layer 2 contains two constructs that address governance gaps created by AI systems. These gaps are structural — pre-AI governance frameworks were not designed for organizations where AI systems produce, interpret, and act on knowledge. Both constructs are validated as substantively novel: no existing framework provides equivalent coverage.

### ERKIA: Epistemic Risk Kinds for AI

ERKIA positions epistemic risks — risks to the integrity of knowledge produced, interpreted, and acted upon by AI systems — as primary enterprise risks. Traditional enterprise risk frameworks treat information quality as a supporting concern subordinated within broader operational or compliance categories. ERKIA inverts this hierarchy: epistemic failures cause operational, legal, reputational, and strategic failures. The causal direction runs from knowledge integrity to organizational outcomes, not the reverse.

Five primary risk categories capture the distinct modes of epistemic failure in AI-mediated knowledge work. Four derivative categories capture the organizational consequences that cascade from primary epistemic failures.

### Table 11.5.1: ERKIA Risk Categories

| Category | Type | Description | Tier |
|---|---|---|---|
| **Knowledge Accuracy** | Primary | Risk that knowledge produced or acted upon by AI systems is factually incorrect | 1 |
| **Source Reliability** | Primary | Risk that knowledge sources used by AI systems are not authoritative, current, or trustworthy | 1 |
| **Inference Validity** | Primary | Risk that reasoning and conclusions drawn by AI systems are logically unsound or biased | 1 |
| **Information Currency** | Primary | Risk that knowledge used for decisions is outdated relative to the decision context | 1 |
| **Contextual Applicability** | Primary | Risk that knowledge valid in one context is incorrectly applied to another | 1 |
| **Operational Failure** | Derivative | Operational failures cascading from acting on epistemically compromised knowledge | 2 |
| **Legal Exposure** | Derivative | Legal liability cascading from decisions based on faulty information | 2 |
| **Reputational Damage** | Derivative | Reputation harm cascading from publishing or acting on incorrect claims | 2 |
| **Strategic Misdirection** | Derivative | Strategic failures cascading from flawed intelligence or analysis | 2 |


### AICAR: AI Cognitive Activity Review

AICAR provides a task-type taxonomy for human review of AI outputs. It specifies what kind of cognitive work the reviewer is performing — not when or how oversight occurs, but what the reviewer is cognitively doing during the review.

Four review types, each with three verification activities, produce twelve verification activities total.

### Table 11.5.2: AICAR Review Types

| Type | Name | Description | Verification Activities |
|---|---|---|---|
| **A** | Citation Fidelity | Verify sources, accuracy of references, and attribution correctness | 3: source existence check, quote accuracy, attribution correctness |
| **B** | Reasoning & Judgment | Evaluate logical validity, premise soundness, and conclusion appropriateness | 3: logical validity, premise soundness, conclusion appropriateness |
| **C** | Context & Applicability | Assess relevance to context, applicability limits, and edge case consideration | 3: relevance assessment, applicability limits, edge case consideration |
| **D** | Communication & Risk Posture | Review tone, audience appropriateness, and stakeholder impact assessment | 3: tone calibration, audience appropriateness, risk exposure assessment |

**Differentiation from EU AI Act Article 14.** Article 14 specifies temporal modes of human oversight: human-in-the-loop (human approves all decisions), human-on-the-loop (AI operates, human monitors), and human-in-command (human retains ultimate authority). These modes describe when and how humans engage with AI systems — the oversight mechanism. AICAR specifies what the reviewer is cognitively doing during oversight — the task type. An Article 14 human-in-the-loop reviewer performing Type A work (citation fidelity) is doing fundamentally different cognitive work than one performing Type B work (reasoning and judgment), even though both operate under the same oversight mechanism. The two frameworks are complementary: Article 14 provides the oversight mechanism; AICAR provides the task-type specificity that makes oversight effective. No existing framework — Article 14, NIST AI RMF, Crootof et al.'s nine human roles taxonomy — organizes human review around the cognitive nature of the verification task itself.

## §11.6 Layer 3: Operational Authorities


Layer 3 maps operational patterns to external regulatory and operational authorities. These are not substrate-originated constructs. They are established operational disciplines with authoritative external sources — The substrate maps to them, positioning specific operational patterns within the established research and practice frameworks that govern them.

### Table 11.6.1: Layer 3 Architecture

| Component | Description | Example |
|---|---|---|
| **Authority source** | The external body or research tradition that governs the operational pattern | FinOps Foundation, Google SRE discipline, Federal Reserve/OCC |
| **Pattern signatures** | 5–15 core concepts distilled from the authority source, mapped to substrate primitives | FinOps maturity stages (Crawl → Walk → Run), SRE golden signals (latency, traffic, errors, saturation), SR 11-7 four pillars (development, validation, governance, effective challenge) |
| **Mapping direction** | Operational pattern → external authority, not the reverse | AI cost governance patterns map to FinOps capabilities; capacity planning patterns map to SRE practices; model risk patterns map to SR 11-7 framework |

The Layer 3 set is extensible. Three illustrative mappings establish the structural pattern.

**AI cost governance → FinOps Foundation.** AI compute cost management maps to the FinOps maturity model and its AI-specific working group outputs. Pattern signatures capture maturity stages, token-based cost attribution, and cost optimization capabilities.

**Capacity planning and reliability → Google SRE.** AI infrastructure reliability patterns map to SRE's service level framework (SLIs, SLOs, error budgets), four golden signals, and capacity planning methodology. Pattern signatures capture reliability measurement, overload handling, and graceful degradation practices.

**Model risk management → SR 11-7.** AI model governance maps to the Federal Reserve/OCC supervisory guidance on model risk management and its evolving extensions to AI/ML systems. Pattern signatures capture the four pillars: model development and documentation, independent validation, governance and controls, and effective challenge.

Additional operational authority mappings — privacy controls, inference optimization, human oversight mechanisms — follow the same structure. The set grows as new operational domains are brought under governance without requiring architectural changes to the composition model.

## §11.7 Pattern Signatures


The pattern signature is the representation unit for all governance content across all three layers. Each pattern signature distills a governance source into 5–15 core concepts that map to substrate primitives. It is the architectural bridge between an external governance framework and the DLP primitive grammar (§4).

### Table 11.7.1: Pattern Signature Structure

| Field | Description | Example |
|---|---|---|
| **Source reference** | The governance source this pattern is extracted from | COSO IC 2013, ERKIA v1, FinOps for AI |
| **Pattern identifier** | Unique identifier within the source | Principle 7 (COSO IC), Knowledge Accuracy Risk (ERKIA), Crawl Maturity (FinOps) |
| **Pattern type** | Classification of the governance concept | Principle, control, function, risk type, review type, capability, practice |
| **Core concepts** | The key terms defining this pattern (5–15 per source) | Risk identification, risk analysis, risk response |
| **Primitive alignment** | Which Tier 1 primitives (§4) this pattern maps to | Constraint, Decision (for a risk assessment principle) |
| **Conservation law reference** | Which of the nine conservation laws (§9) this pattern enforces, if any | Boundary invariance — same rule produces same prohibition |
| **Governance criticality** | Four-level classification determining gap prioritization | conservation_critical, invariant_enforcing, standard, advisory |
| **Hierarchy position** | Parent-child relationship to other patterns from the same source | ERKIA Knowledge Accuracy → ERKIA (parent); AICAR Type A Source Check → AICAR Type A (parent) |
| **Activation context** | Conditions under which this pattern becomes applicable to an organization | AI-native organization (ERKIA patterns); federal contractor (NIST 800-53 patterns) |

**Primitive alignment** connects each governance pattern to the Tier 1 primitives it exercises. A COSO IC principle on board oversight maps to Authority and Decision. An ERKIA epistemic risk type maps to Constraint, Evidence, and Decision. An SRE golden signal maps to Evidence and Work. The alignment makes governance content structurally legible within the primitive grammar — every governance concept is ultimately about Authority, Commitment, Evidence, or other Tier 1 primitives and their relationships.

**Conservation law reference** connects governance patterns to the nine conservation laws (§9). A pattern that enforces delegation invariance (same delegation → same scope at every organizational level) is architecturally different from a pattern that captures operational best practice without conservation law connection. This field is null for patterns that do not directly enforce a conservation law — most Layer 3 patterns, for instance, are operationally valuable but do not map to specific conservation laws.

### Table 11.7.2: Governance Criticality Classification

| Level | Definition | Gap Impact | Example |
|---|---|---|---|
| **Conservation-critical** | Pattern directly enforces one of the nine conservation laws (§9) | Gap threatens structural governance integrity — highest closure priority | Authority traceability patterns (delegation invariance), evidence classification patterns (verification invariance) |
| **Invariant-enforcing** | Pattern supports behavioral invariant enforcement (§5) without direct conservation law correspondence | Gap weakens invariant enforcement — high closure priority | Control activity patterns supporting B1 (work requires commitment), monitoring patterns supporting B7/B8 (signal capture and routing) |
| **Standard** | Pattern implements recognized governance best practice | Gap represents governance immaturity — normal closure priority | COSO IC principles, NIST CSF functions, ISO 27001 controls |
| **Advisory** | Pattern captures operational guidance or reference material | Gap represents opportunity, not structural risk — lowest closure priority | Layer 3 operational authority mappings, maturity model progression stages |

The criticality classification transforms gap assessment from flat prioritization to structural prioritization. Conservation-critical gaps are architecturally dangerous — they threaten the invariances that §9 proves must be preserved. Advisory gaps are operational opportunities — valuable but not structurally threatening. The activation engine (§12) uses this classification to order closure roadmaps: conservation-critical gaps before invariant-enforcing, before standard, before advisory.

## §11.8 Architectural Context: Complementary Engines


The Governance Integration Architecture is one of two complementary composition engines in the DLP substrate.

**Governance Integration (§11/§12)** maps governance frameworks — standards, risk constructs, operational authorities — to primitive-level patterns. It answers: what governance applies to this organization, and where are the gaps?

**Ontology Alignment (§19)** maps external domain ontologies — industry vocabularies, methodology structures, domain-specific data models — to primitive-level structures. It answers: what do external domain concepts mean in primitive terms?

Both engines compose external content into the same Tier 1 primitive grammar (§4). Both produce pattern-level structures that the substrate can activate and project. They differ in source material (governance frameworks vs. domain ontologies) and in output (governance gaps vs. alignment mappings). Both are substrate-tier specifications that consume DLP-Core primitives and DLP-World-Model concepts.

Both feed a three-stage pipeline that begins with governance activation and ends with conformance verification.

### Table 11.8.1: Three-Stage Governance Pipeline

| Stage | Section | Input | Output | Feedback Loop |
|---|---|---|---|---|
| **1. Governance Activation** | §12 | Organizational profile + registered pattern signatures (§11) | Activated governance patterns — the applicable subset for this organization | Profile changes trigger re-activation |
| **2. Policy Projection** | §13 | Activated governance patterns + current primitive state | Projected policy documents across seven governance domains | Pattern activation changes trigger re-projection |
| **3. Conformance Tests** | §13.6 | Projected policies + enforcement reality | Conformance results — does enforcement match projection? | Conformance failures register as governance gaps, feeding back into Stage 1 |

The feedback loop is structural. Conformance failures detected at Stage 3 propagate back as governance gaps into Stage 1's gap register. A policy projection that claims enforcement where none exists produces a conformance failure, which produces a gap, which enters the closure roadmap. The pipeline is self-correcting: the organization cannot project governance it does not enforce without the system detecting and registering the discrepancy.


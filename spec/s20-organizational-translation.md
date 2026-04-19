# §20 Organizational Translation (TMI Configure Mode)

Translation & Meaning Integration (TMI) is the organizational world model construction pipeline — a unified capability that bridges human-familiar organizational artifacts and AI-native substrate structure across the full substrate lifecycle. This section specifies TMI's Configure mode — the substrate's implementation of the DLP Configure stage (§18.2): where the DLP lifecycle specifies that Configure must populate the substrate with organizational context and methodology sufficient for governance activation, TMI Configure is the mechanism that satisfies that requirement by translating organizational knowledge into the initial world model. A competing DLP implementer could build a different Configure implementation. This section specifies the Configure mode architecture; §18.2 specifies Configure's output requirements.

TMI Configure addresses all nineteen primitives across all five tiers (§4). The Organization Layer populates structural primitives — who the organization is, what it has, what constrains it. The Methodology Layer configures operational primitives — how work flows, what triggers it, what it produces, who decides. Together, the two layers answer the two foundational questions every substrate instance must resolve after genesis: "Who are you?" and "How does your work work?"

### §20.1 TMI Purpose and Lifecycle Position

TMI resolves the translation problem. Organizations possess institutional knowledge, existing documents, practitioner expertise, mental models, and domain vocabulary. They do not possess governance primitive instances, module registries, routing matrices, or controlled vocabularies mapped to formal ontologies. TMI bridges this gap by asking what organizations already know how to answer and translating responses into substrate-executable structure.

The substrate lifecycle positions TMI between genesis and governance activation:

Genesis (§18.2)
→ Substrate structurally valid but contextually empty
→ TMI (this section)
→ Organization Layer: populate structural primitives (Tiers 1–2)
→ Methodology Layer: configure operational primitives (Tiers 1, 3, 5)
→ Governance Activation (§12)
→ Activation pipeline, gap projection, closure roadmap
→ Operational
→ Substrate receives work, governs according to methodology

Post-genesis, the substrate has structural validity — seven genesis objects, the primitive ontology, behavioral invariants. It does not have organizational context. TMI produces the substrate state that §12's activation pipeline requires as input: an organizational profile with populated primitives sufficient for source activation, requirements projection, and gap assessment.

The translation operates like an auditor arriving at a new engagement. The auditor needs to understand what kind of entity this is, what transactions flow through, what controls should exist, and what evidence should be produced. Then as transactions flow, the audit framework knows what to do with them. The substrate operates identically — TMI is the structured interview that gives it what it needs.

### §20.2 Two-Layer Architecture

TMI is one architectural component, not two independent systems. The two layers share design principles, path architecture, progressive depth model, and substrate destination. They are separated because they address different concerns — organizational structure versus operational methodology — and may involve different participants. A founder answers Organization Layer questions; a domain practitioner answers Methodology Layer questions. Both populate the same substrate.

#### Table 20.2.1: TMI Layer Architecture

| Layer | Guiding Question | Input Source | Primitive Output by Tier |
|---|---|---|---|
| **Organization** | "Who are you and what do you have?" | Vintage business documents or form answers | **Tier 1:** Intent, Commitment, Capacity, Authority, Constraint, Account. **Tier 2:** Entity, Identifier, Context, Namespace. |
| **Methodology** | "How does your work work?" | Practitioner domain knowledge | **Tier 1:** Work patterns. **Tier 3:** Orientation (pre-decisional framing patterns), Learning (adaptation triggers), Activation (readiness conditions). **Tier 5:** Cycle (temporal governance patterns). |

**Tier coverage.** The two layers together address all five primitive tiers. Organization Layer populates the six Tier 1 structural primitives and all four Tier 2 infrastructure primitives. Methodology Layer configures Work (Tier 1) and the operational primitives that govern how work executes: Orientation establishes the interpretive frames through which the methodology processes information, Learning defines when and how the methodology adapts, Activation specifies the readiness conditions that gate work execution, and Cycle encodes the temporal patterns (fiscal periods, review cadences, audit windows) that structure governance over time.

**Decision and Evidence** are primarily populated during Operate, not Configure. Decisions are choice points that occur when work executes. Evidence is produced by work execution. TMI configures the substrate to *recognize* and *classify* decisions and evidence when they occur — it does not pre-populate them.

**Tier 4 (Interpretation, Environment Interface)** activates when AI systems join governance chains (§4.7). TMI does not configure Tier 4 directly — these primitives activate through the integration layer (§18) as external systems connect and AI agents begin participating in governance.

**Sequencing.** Organization Layer executes first. It populates the structural primitives that Methodology Layer references — Work patterns cannot be configured without knowing who has Authority, what Capacity exists, and what Constraints apply. Partial overlap is possible: an organization may begin Methodology Layer extraction while completing Organization Layer depth tiers. The dependency direction is unambiguous: methodology references organization, not the reverse.

### §20.3 Three Onboarding Paths

Each TMI layer supports three paths to substrate population. All three paths produce the same substrate state — the destination is identical regardless of route.

#### Table 20.3.1: Onboarding Paths

| Path | Input | Process | Advantage | Challenge |
|---|---|---|---|---|
| **A: Parse Vintage Documents** | Existing organizational documents (business plans, org charts, capability statements, methodology documentation) | AI extracts primitive-mapped content from uploaded documents; proposes mappings; human confirms or corrects each mapping | Leverages existing documentation investment; familiar starting point for organizations with mature artifacts | Vintage documents are often inconsistent, incomplete, or outdated; narrative format embeds primitives in prose requiring interpretation |
| **B: Fresh Onboarding Questions** | Human responses to guided questions (Organization: ~47 questions; Methodology: ~55 questions) | System structures responses into primitive instances; human reviews structured output | Clean, minimal, current; produces internally consistent substrate state from the start | Requires articulating answers without an existing document to reference |
| **C: Hybrid (recommended)** | Whatever documentation exists + guided questions for gaps | AI parses available documents, identifies which primitives are populated and which are missing, then generates targeted questions for gaps only | Combines the leverage of existing documentation with the precision of guided extraction; respects existing investment while ensuring completeness | Requires gap detection accuracy; AI must correctly identify what is present and what is absent |

**Path C is the recommended default.** Most organizations have partial documentation — a business plan but no capability statement, an org chart but no formal authority matrix. The hybrid path uses what exists and fills what is missing through targeted questions.

**Truth type integration.** Path A AI-parsed mappings enter the substrate as Derived (§6.1) — they are machine-generated proposals in staging, not canonical content. Human confirmation promotes each mapping to Declared. Path B human responses enter as Declared directly — they are human assertions representing best current understanding. Path C combines both patterns: parsed content enters as Derived, human-answered gap-fill enters as Declared. In all paths, TMI-populated primitives satisfy behavioral invariants B1–B10 (§5) at write time.

### §20.4 Organization Layer

The Organization Layer populates organizational structure from information every organization already possesses — whether encoded in existing documents or held as founder knowledge. It addresses six Tier 1 primitives and all four Tier 2 primitives.

#### Table 20.4.1: Organization Layer — Primitive Population

| Primitive | Tier | What Organization Layer Extracts | Illustrative Questions |
|---|---|---|---|
| **Intent** | 1 | Core purpose, value proposition, aspirational state, who is served | "In one sentence, what are you trying to achieve and for whom?" / "What would success look like in three years?" |
| **Commitment** | 1 | Promises made, to whom, by when, terms and conditions | "What have you formally committed to deliver? To whom? By when?" / "What contractual obligations are active?" |
| **Capacity** | 1 | What the entity can deliver, resources, credentials, key personnel capabilities | "What can you actually deliver? What resources make that possible?" / "What certifications or credentials does the organization hold?" |
| **Authority** | 1 | Decision rights, reporting structure, delegation chains, root authority | "Who makes binding decisions? What can each role decide?" / "Who can override whom, and under what conditions?" |
| **Constraint** | 1 | Universal rules regardless of authority — policies, regulations, values | "What is absolutely not permitted, regardless of who requests it?" / "What regulatory requirements apply to your operations?" |
| **Account** | 1 | Current state of key resources, historical position, chart of accounts | "What is the current state of your key resources?" / "What financial accounts track organizational activity?" |
| **Entity** | 2 | Legal name, structure, jurisdiction, organizational units, recursive subdivision | "What is your legal name and structure?" / "How is the organization subdivided into units or departments?" |
| **Identifier** | 2 | Registration numbers, tax identifiers, addressing schemes, URI patterns | "What formal identifiers does the organization carry?" / "How are internal units and roles addressed?" |
| **Context** | 2 | Operating environment, temporal context, jurisdictional context, market position | "What industry and regulatory environment do you operate in?" / "What jurisdictions govern your operations?" |
| **Namespace** | 2 | Organization-specific concept definitions, domain vocabulary, classification schemes | "What terms does your organization define differently from common usage?" / "What classification systems do you use internally?" |

#### Table 20.4.2: Vintage Document → Primitive Decomposition (Representative)

| Document | Primary Primitives Extracted | What the Decomposition Reveals |
|---|---|---|
| **Business Plan** | Intent (purpose, value proposition), Entity (legal structure), Capacity (what can be delivered), Commitment (milestones, funding terms), Account (financial projections) | The business plan is a composite artifact — its narrative form embeds at least five distinct primitives. Extraction decomposes the narrative into addressable governance objects. |
| **Organizational Chart** | Authority (reporting lines, decision rights, delegation structure), Entity (departments, units, roles) | The org chart is primarily an Authority artifact with Entity subdivision. TMI extracts both the explicit structure (boxes and lines) and the implicit Authority delegation chain. |
| **Capability Statement** | Capacity (core competencies, credentials), Evidence (past performance), Authority (key personnel expertise) | The capability statement proves Capacity through Evidence. TMI distinguishes what the organization claims it can do (Capacity) from the proof that it can (Evidence). |
| **Operating Agreement** | Authority (decision rights, voting, management structure), Constraint (transfer restrictions, governance rules), Commitment (member obligations, capital contributions) | The operating agreement is the root authority document — it establishes the Authority delegation chain that B5 (§5) requires to terminate at a traceable root. |

The full decomposition for all vintage document types is reference material. The pattern is consistent: every traditional business document decomposes into a subset of the nineteen primitives. The decomposition is lossy in one direction — the narrative, formatting, and rhetorical structure of the vintage document are not preserved — and lossless in the governance direction — every governance-relevant assertion in the document maps to a primitive instance.

### §20.5 Methodology Layer

The Methodology Layer extracts the structured patterns of how work operates — from practitioners who carry this knowledge intuitively but have never formalized it. The extraction framework is domain-agnostic by design: it targets structural patterns that exist in every organized practice. The same questions extract methodology from a law firm, a hospital, a manufacturing operation, a software team, or an architecture practice.

#### Table 20.5.1: Methodology Extraction Framework

| Part | Questions | Guiding Question | Extracts To |
|---|---|---|---|
| **1: Work Types** | Q1–Q8 | "What are the distinct things you do?" | Work type inventory, naming, objectives, phases, lifecycle sequencing, effort bands |
| **2: Triggers** | Q9–Q16 | "What makes you start each type of work?" | Activation signals, trigger sources (external/internal), prerequisites, decision branch points, stop conditions, next steps |
| **3: Outputs** | Q17–Q25 | "What do you produce?" | Evidence artifacts, components, audiences, quality criteria, review requirements, standards |
| **4: Domain Language** | Q26–Q33 | "What words mean specific things in your work?" | Controlled vocabulary terms, definitions, sources, disambiguation rules, classification categories |
| **5: Authority** | Q34–Q41 | "Who decides what?" | Roles, decision scopes, limits, approval matrices, thresholds, delegation rules |
| **6: Exceptions** | Q42–Q48 | "What happens when things go wrong?" | Exception types, handling procedures, escalation triggers, escalation paths, blocking conditions |
| **7: Connections** | Q49–Q55 | "How does work relate to other work?" | Dependencies, parallel work, data flow, interfaces, handoffs, connecting systems |

#### Table 20.5.2: Illustrative Methodology Questions

| Part | Question | What It Extracts |
|---|---|---|
| 1: Work Types | "What are the main types of work you do? List each as a distinct activity." | Work type inventory — the atomic units of methodology |
| 1: Work Types | "For each work type: what does 'done' look like? What gets produced?" | Evidence output specification per work type |
| 2: Triggers | "For each work type: what triggers it? What makes you start?" | Activation signals — the events that initiate governance-tracked work |
| 2: Triggers | "When this work is done, what typically happens next?" | Routing logic — the post-completion decision that connects work types |
| 3: Outputs | "How do you know the output is good enough? What do you check?" | Quality constraint criteria — the standards that Evidence must meet |
| 4: Domain Language | "What terms do you use that have specific meanings in your domain?" | Controlled vocabulary entries — terms requiring formal definition |
| 4: Domain Language | "Are there terms that sound similar but mean different things?" | Disambiguation rules — preventing semantic confusion in governance records |
| 5: Authority | "What roles are involved in your work? Not people — roles." | Authority role definitions — the positions that hold decision rights |
| 5: Authority | "Are there thresholds? For example, decisions above a certain value that need senior approval." | Authority threshold constraints — limits that trigger escalation |
| 6: Exceptions | "What are the most common problems or exceptions you encounter?" | Exception taxonomy — the known failure modes of the methodology |
| 7: Connections | "Which work types depend on others being done first?" | Work dependency graph — the structural ordering of methodology |
| 7: Connections | "Does your work connect to others' work? How?" | Interface specification — the boundary between this methodology and adjacent ones |

The full question inventory is reference material. The framework's architecture is visible in the structure above: seven parts moving from what the work *is* (Parts 1–3), through how it is *named and authorized* (Parts 4–5), to how it *fails and connects* (Parts 6–7). Each part targets a distinct structural concern. Together, they produce a complete methodology description from any domain.

**Four translation outputs.** After extraction, the Methodology Layer produces four structured artifacts:

- **Module Registry.** Each work type becomes a registered module with lifecycle phase, trigger domains, effort band, evidence pack components, and interface specifications. The module registry is the substrate's executable representation of what work types exist and what each requires.

- **Routing Logic.** Decision branching from trigger and next-step questions becomes formal routing: WHEN [trigger] AND [prerequisites] THEN activate [work type]; AFTER completion, IF [condition] THEN [next step] ELSE [alternative]. Routing logic is the substrate's executable representation of how work flows.

- **Controlled Vocabulary.** Domain-specific terms from language questions become controlled vocabulary entries with definitions, sources, disambiguation rules, and common error corrections. The controlled vocabulary maps to Namespace tenant-scope concepts (§4.6.1), enabling cross-domain alignment through the namespace governance model.

- **Authority Matrix.** Roles, scopes, limits, approval requirements, thresholds, and delegation rules from authority questions become formal authority specifications. The authority matrix extends the Organization Layer's Authority primitive instances with methodology-specific decision rights.

**Tier 3 and Tier 5 configuration.** The Methodology Layer configures higher-tier primitives through the extraction patterns, not through direct Tier 3/5 questions:

- **Orientation** (Tier 3): Work type triggers and decision branch points define the pre-decisional frames through which the methodology interprets incoming signals. When the substrate receives an event, the trigger extraction determines which interpretive frame applies.

- **Learning** (Tier 3): Exception handling patterns and escalation paths define when the methodology recognizes dissonance between expected and actual outcomes — the trigger for institutional adaptation.

- **Activation** (Tier 3): Prerequisites and stop conditions define the readiness conditions that must be satisfied before work can begin — the friction metrics that Activation tracks.

- **Cycle** (Tier 5): Work phases, review cadences, and temporal references embedded in trigger and output questions define the periodic structures that Cycle makes queryable as first-class governance objects.

### §20.6 Progressive Depth

Not all questions are required upfront. Each TMI layer supports three depth tiers, allowing organizations to start minimal and add detail as operational needs require.

#### Table 20.6.1: Organization Layer Depth Tiers

| Depth Tier | Questions (approx.) | What the Organization Gets | Sufficient For |
|---|---|---|---|
| **Minimum** | ~4 | Legal name, structure, core purpose, who is served | Identity established; genesis objects enriched; Tier 1 Intent and Tier 2 Entity populated at minimum viable depth |
| **Operational** | ~25 | Above + team structure, authority matrix, key commitments, resource inventory, operating constraints | Work assignment, basic governance routing, and authority verification; sufficient for governance activation (§12) at Stage A |
| **Complete** | ~47 | Full organizational substrate across all Organization Layer primitives | Complete governance coverage, policy projection (§13), audit readiness; all Organization Layer primitives populated at full depth |

#### Table 20.6.2: Methodology Layer Depth Tiers

| Depth Tier | Questions (approx.) | What the Organization Gets | Sufficient For |
|---|---|---|---|
| **Minimum** | ~12 | Work type inventory, activation triggers, primary outputs, post-completion routing | Basic work capture — the substrate knows what types of work exist, what starts them, what they produce, and what happens next |
| **Operational** | ~25 | Above + lifecycle phases, decision branch points, quality standards, role definitions and authority scopes | Governed work patterns with routing logic and approval gates; methodology is operational with authority enforcement |
| **Complete** | ~55 | Full methodology package: controlled vocabulary, exception handling, escalation paths, cross-work interfaces, dependency mapping | Complete methodology specification; all four translation outputs (module registry, routing logic, controlled vocabulary, authority matrix) fully populated |

**Depth tier design principles:**

1. **Each tier is independently operational.** An organization at Minimum tier has a functioning substrate, not a broken one. Minimum produces a substrate that can receive and classify work. It does not produce a substrate that is waiting for more configuration before it can function.

2. **Each tier adds governance fidelity.** Operational adds routing logic and authority enforcement — the substrate not only captures work but governs its flow. Complete adds vocabulary precision, exception handling, and cross-methodology interfaces — the substrate governs with the full precision of the practitioner's domain knowledge.

3. **Tier progression is non-disruptive.** Adding depth enriches existing primitive instances; it never requires reworking or invalidating them. An Intent populated at Minimum depth gains additional context at Operational depth — the Minimum-depth Intent is not replaced, it is elaborated.

4. **Tier visibility is explicit.** The substrate knows its own depth tier per layer and can project what governance capabilities are currently available versus what would unlock with deeper configuration. This self-awareness enables the substrate to recommend depth progression when operational needs exceed current configuration.

### §20.7 Bidirectional Bridge

TMI establishes a bidirectional bridge between vintage documents and substrate state. The parse direction (vintage → primitives) decomposes familiar artifacts into governance structure. The projection direction (primitives → vintage formats) renders substrate state back into familiar document formats. Post-TMI, vintage formats become projections — views rendered from substrate state rather than primary artifacts.

#### Table 20.7.1: Substrate → Vintage Projection

| Primitive State Combination | Can Project To |
|---|---|
| Intent + Capacity + Commitment | Business Plan |
| Intent + Constraint + Work | Strategic Plan |
| Intent + Constraint | Vision/Mission Statement |
| Capacity + Evidence + Authority | Capability Statement |
| Authority + Entity (structural) | Organizational Chart |
| Authority + Constraint + Commitment | Operating Agreement |
| Account + Commitment + Capacity | Budget / Financial Plan |
| Constraint + Authority + Work | Policies and Procedures |
| Authority + Capacity + Work | Job Descriptions |

**Projection properties:**

1. **Consistency guarantee.** Because projections render from a single substrate state, all vintage format outputs are internally consistent. The organizational chart reflects the same Authority structure as the policies and procedures, which reflect the same Constraints as the operating agreement. Inconsistency between documents — a pervasive problem with manually maintained artifacts — is structurally eliminated.

2. **Currency guarantee.** Projections render from current substrate state, not from a document last updated at an unknown prior date. When the substrate's Authority structure changes, every projected document that includes Authority content reflects the change on next rendering.

3. **Bidirectional traceability.** Every element in a projected document traces to a specific primitive instance. Every primitive instance can identify which projected documents it contributes to. The traceability operates in both directions without reconstruction — it is a structural property of the rendering, not a post-hoc analysis.

**Protocol/methodology boundary.** The TMI architecture embodies a structural separation between framework and content. The question frameworks, extraction-to-primitive mappings, translation output structures, and progressive depth model belong to protocol — they define the grammar of organizational description. The answers to those questions — encoded domain expertise, populated module registries, domain-specific routing logic, and practitioner methodology — belong to methodology. The framework is the grammar; the answers are the sentences. A protocol implementer could build the TMI extraction engine and question interfaces without any domain knowledge. A domain practitioner could answer the questions without understanding governance primitives. TMI connects them.

This boundary maps to the DLP-Core / DLP substrate separation. DLP-Core defines the primitive set that TMI populates, the behavioral invariants that TMI-populated primitives must satisfy, and the truth type system that classifies TMI-produced content. DLP substrate implements TMI as operational onboarding — question interfaces, parsing engine, gap detection, progressive depth management. Domain-specific derivatives provide the methodology content that practitioners encode through TMI's extraction framework.

## Governance

*No Governance section for this document.*

### §20.8 SDK Constraints Summary

#### Table 20.8.1: §20 SDK Constraints

| Constraint | Type | Rationale |
|---|---|---|
| TMI-populated primitives MUST satisfy all ten behavioral invariants (B1–B10, §5) at write time | MUST | TMI produces governance state. Governance state that violates invariants is structurally unsound regardless of when it was produced. A Commitment written during TMI without Capacity backing (B2) is no less invalid than one written during Operate. |
| Human-answered TMI content MUST enter the substrate with truth type Declared (§6.1) | MUST | Human responses are assertions representing best current understanding. They are canonical (usable in governance) but versioned (revisable as understanding evolves). |
| AI-parsed vintage document content MUST enter the substrate with truth type Derived (§6.1) and remain in staging until human confirmation promotes it to Declared | MUST | AI extraction is machine-generated interpretation. The staging requirement (§6.2) applies to all AI-generated content without exception. Promotion requires explicit human action. |
| Organization Layer MUST complete before Methodology Layer begins, or overlap MUST use explicit dependency tracking to ensure Methodology Layer references only already-populated Organization Layer primitives | MUST | Methodology references organization: Work patterns reference Authority, Capacity, and Constraint. Configuring Work without populated Authority produces a methodology that cannot be governed. |
| Each progressive depth tier MUST be independently operational — producing a functioning substrate, not a partially configured one | MUST | An organization at Minimum depth must be able to receive work and activate governance. A depth tier that produces an inoperable substrate forces organizations through the full question set before any value is delivered. |
| The substrate MUST NOT enter governance activation (§12) without Configure stage completion — at minimum, both TMI layers at Minimum depth tier | MUST NOT | §12's activation pipeline requires an organizational profile with populated primitives as input. A substrate without TMI configuration has structural validity but no organizational context — source activation, requirements projection, and gap assessment have nothing to operate on. |
| TMI MUST NOT bypass the truth type system for any content, regardless of source | MUST NOT | The epistemic boundary (§6.2) is architectural, not procedural. TMI is not a privileged writer — it populates the substrate through the same truth type enforcement as any other content source. |
| AI agents MUST NOT perform TMI confirmation acts — promoting Derived content to Declared, or confirming that AI-parsed mappings are correct | MUST NOT | TMI confirmation is a governance act requiring human authority. AI agents may propose mappings, identify gaps, and draft content to staging — but confirmation requires a human with traceable authority (B5). |
| Question phrasing, presentation order, and user interface patterns for TMI onboarding | DESIGN SPACE | The architecture specifies what TMI must extract (primitive instances satisfying invariants at required depth tiers) and the truth type of the output. How questions are phrased, in what order they are presented, and through what interface — conversational, form-based, wizard — is implementation choice. |
| Vintage document parsing implementation, extraction algorithms, and confidence scoring | DESIGN SPACE | The architecture specifies that AI-parsed content enters as Derived and requires human confirmation. The specific parsing approach — LLM-based extraction, rule-based parsing, hybrid, or future methods — is implementation choice, as is the confidence model used to prioritize human review. |
| Projection rendering format, template design, and output medium | DESIGN SPACE | The architecture specifies the primitive-to-vintage-format mapping and the three projection properties (consistency, currency, traceability). How projections are rendered — PDF, web view, editable document — and what templates are used is implementation choice. |
| Depth tier boundary placement — which specific questions constitute Minimum versus Operational versus Complete | DESIGN SPACE | The architecture specifies the properties each depth tier must satisfy (independently operational, non-disruptive progression, explicit visibility). The exact question-to-tier assignment may vary across implementations and may be refined through operational experience. |

## Boundaries

*No Boundaries section for this document.*

## Positions

*No Positions section for this document.*


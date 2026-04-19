# §9 Symmetry & Conservation Laws

§8 established what the organizational world model must be: a governance model with requisite variety, instantiated in specific structural subsystems, operating as a state-prediction engine. This section establishes why the organizational world model must be explicitly constructed rather than learned from data — providing convergence guarantees that ensure the model maintains fidelity to organizational governance principles. The argument is mathematical: log-loss optimization — the dominant training paradigm for modern AI systems — systematically breaks the symmetries that governance must preserve, and this preference strengthens with data scale.

Each of the nine Tier 1 primitives preserves a specific organizational symmetry whose violation produces a diagnosable governance failure. Noether's theorem (1918) provides the formal connection: every preserved symmetry corresponds to a conservation law — a governance quantity that must be conserved across every state transformation. The behavioral invariants (§5) are the enforcement mechanisms for these conservation laws. The primitives are the architectural symmetries that make enforcement possible.

### §9.1 The Symmetry-Breaking Proof

A natural objection to the §8 derivation chain: why not learn the organizational world model from data? If modern AI systems can model language, vision, and physical dynamics, why can't they model organizational decision-making?

The answer is mathematical, not empirical. Chlon (2026, pre-print) uses Minimum Description Length (MDL) theory to prove that log-loss optimization does not merely fail to learn invariances — it actively breaks them. The symmetry-preserving representation is penalized under standard training because the symmetry-breaking representation achieves shorter description length.

| Component | Formal Statement | Implication |
|---|---|---|
| **Setup** | A model class partitions into an invariant subclass (preserves symmetry group G) and an unrestricted class (exploits symmetry-breaking coordinate G). | Two strategies exist: learn the genuine invariance, or learn the cheaper symmetry-broken shortcut. |
| **Orbit-label information** | The gap between invariant and unrestricted optimal codelength is quantified by I_p(Y; G \| Z), where Z = φ(X) is an invariant sufficient statistic and G is the symmetry-breaking coordinate. | The symmetry-broken representation captures information the invariant representation discards — and this information improves log-loss, even though it destroys the invariance. |
| **Scaling behavior** | Invariance is MDL-preferred only if its model complexity reduction exceeds n · I_p(Y; G \| Z), where n is dataset size. The threshold grows linearly with n. | **Scaling makes symmetry breaking worse, not better.** More data strengthens the optimizer's preference for symmetry-breaking representations. |
| **Conclusion** | Under log-loss, the optimizer systematically prefers representations that break symmetries over representations that preserve them, and this preference strengthens with data scale. | Invariances required for governance cannot be learned from data under standard training. They must be architecturally imposed. |

**The 6/9 Intuition.** The formal result has an accessible illustration. Write "6" on the floor and stand on opposite sides. From the south, a well-trained model confidently reports "6." From the north, the same model confidently reports "9." Given position encoding, the model learns: "if standing here → 6; if standing there → 9."

The symmetry-breaking representation wins on log-loss because encoding the position-dependent answer is cheaper than representing the genuine rotational invariance. The optimizer is doing exactly what the objective asks. The problem is not model quality — it is that the objective does not value invariance.

**Organizational Translation.** Replace "6/9" with any organizational governance question. "Is this expenditure authorized?" — a log-loss-trained model learns context-dependent shortcuts: authorized-when-asked-from-this-system, unauthorized-when-asked-from-that-system. "Does this commitment bind the organization?" — the model learns: binding-in-contract-context, non-binding-in-email-context.

An invariant governance infrastructure enforces: same authorization context → same authorization answer, regardless of access path. Same agreement → same binding force, regardless of capture format. The former is cheap and wrong for governance. The latter is expensive and necessary.

Chlon's Mezzanine toolkit provides a mechanism for recovering specific invariances through orbit-averaged teacher distillation. But the critical limitation: you must know the symmetry to patch it. No general fix exists. The nine primitives are the known symmetries for organizational governance.

### §9.2 Convergent Failure Analysis

Chlon's proof is the mathematical layer of a convergent critique from four independent research traditions, each addressing a deeper layer of the same structural problem.

| Layer | Researcher | What's Missing | DLP Response |
|---|---|---|---|
| **Architecture** | LeCun | Pixel-level prediction wastes representational capacity on irrelevant detail. Predict in latent space, not raw data. | DLP primitives ARE the latent space for organizational decisions. Governance operates on abstract state representations (Intent, Commitment, Authority), not raw operational data (emails, invoices, spreadsheets). |
| **Structure** | Bengio | Associative learning lacks causal and compositional structure. Explicit compositional and causal priors must be architecturally provided. | Nine primitives encode compositional relationships (Intent → Commitment → Work → Evidence → Decision). Causal structure is given, not learned. |
| **Formalism** | Yang | MDP formalism does not model action cost, execution time, or temporal abstraction. | Work + Capacity + Constraint encode execution cost. Evidence provides feedback on predicted vs. actual. Intent → Commitment spans timescales. |
| **Objective** | Chlon | Log-loss optimization breaks symmetries that intelligence must preserve. Scaling makes it worse. | Primitives explicitly encode organizational invariances that cannot be learned. Each primitive preserves a specific organizational symmetry (§9.3). |

These are not four separate critiques. They are four layers of the same structural problem, each deeper than the last. Solving architecture without structure fails. Solving structure without formalism fails. Solving formalism without addressing the objective fails. Each layer is necessary but not sufficient. The DLP addresses all four layers simultaneously for organizational domains: primitives are the latent space (LeCun), with compositional structure (Bengio), encoding action costs and temporal spans (Yang), and explicitly imposing invariances the training objective would otherwise break (Chlon).

**Anticipated Objection: Residual Symmetries.** A potential counter-argument: symmetry-breaking under log-loss may itself be structured. The breaking process may preserve residual symmetries or encode a characterizable antisymmetry — analogous to spontaneous symmetry breaking in physics, where residual subgroup symmetries survive.

This does not rescue learned governance. Even if the symmetry-breaking process has exploitable structure, the residual symmetries are properties of the optimization landscape, not of the organizational domain. The specific invariances governance must preserve — obligation invariance, delegation invariance, verification invariance (§9.3) — are organizational properties with no training signal. Characterizing the shape of the breaking does not produce the governance-specific invariances that were broken.

### §9.3 Nine Primitives as Nine Organizational Invariances

§8 derived that the organizational world model requires nine irreducible primitives (§4), mapped to the five VSM subsystems (§8). §9.1 proved that these invariances cannot be learned. This section specifies which invariance each primitive preserves.

The mapping applies Noether's insight to organizational systems: each primitive preserves a specific organizational symmetry whose violation produces a diagnosable governance failure. The symmetry-breaking examples show what a log-loss-trained model would learn in place of the invariance — context-dependent shortcuts that are cheaper to encode but destructive to governance.

| Primitive | Organizational Symmetry | Conservation Law | Violation Consequence | Symmetry-Breaking Example |
|---|---|---|---|---|
| **Intent** | Purpose invariance | Same purpose → same organizational direction regardless of who states it | Purpose drift without detection; strategy means different things to different actors | Model learns "intent from executive context" ≠ "intent from operations context" — breaks invariance that purpose binds the organization uniformly |
| **Commitment** | Obligation invariance | Same agreement → same binding force regardless of context shift | Obligations evaporate under pressure; commitments become context-dependent | Model learns "commitment in contract" has different weight than "commitment in email" — breaks invariance that agreement = agreement |
| **Capacity** | Resource invariance | Same resources → same feasibility regardless of ambition | Overcommitment without detection; feasibility decoupled from resource reality | Model learns optimistic capacity framing in sales context vs. realistic framing in operations — breaks invariance that available = available |
| **Work** | Execution invariance | Same transformation → same state change regardless of executor | Different actors produce different state changes from identical actions | Model learns executor-dependent representations: "when engineering does X" ≠ "when contractor does X" — breaks invariance that action = action |
| **Evidence** | Verification invariance | Same evidence → same epistemic status regardless of presenter | Evidence quality depends on who presents it; credentialism replaces epistemics | Model learns source-dependent evidence weighting — breaks invariance that proof = proof regardless of provenance |
| **Decision** | State-change invariance | Same decision context → same available options regardless of who decides | Decision paths vary by decider identity; options invisible to some actors | Model learns role-dependent option spaces — breaks invariance that context determines options, not identity |
| **Authority** | Delegation invariance | Same delegation → same scope regardless of organizational level | Authority scope varies by relationship; delegation means different things at different levels | Model learns level-dependent authority semantics — breaks invariance that delegation = delegation across recursive structure |
| **Account** | Accountability invariance | Same state → same accountability exposure regardless of narrative | Accountability depends on framing; narratives shield state from scrutiny | Model learns narrative-dependent accountability — breaks invariance that state = exposure regardless of description |
| **Constraint** | Boundary invariance | Same rule → same prohibition regardless of role or preference | Rules apply differently to different actors; constraints become negotiable | Model learns role-dependent constraint application — breaks invariance that boundary = boundary for all |

**The Invariance Test.** Each row in the table above specifies a testable claim. The test: does the governance system produce the same answer regardless of the symmetry-breaking coordinate (who asks, from where, in what context)?

If yes — the invariance is preserved — governance functions correctly. If no — the invariance is broken — governance produces context-dependent answers that undermine its purpose.

This is the formal connection between Chlon's MDL proof and the DLP's architectural design. The nine primitives are not arbitrary categories. They are the nine organizational symmetries whose preservation is necessary for governance to function and whose violation produces specific, diagnosable pathologies.

### §9.4 Conservation Laws

Noether's theorem (1918) proves that every continuous symmetry of a physical system corresponds to a conservation law. The organizational analogue: every preserved organizational symmetry corresponds to a governance quantity that must be conserved across state transformations. If the architecture preserves the symmetry, the conservation law holds. If the architecture lacks the symmetry, the conservation law fails — and the failure is the organizational pathology described in §9.3.

| Primitive | Symmetry | Conserved Quantity | Conservation Mechanism | DLP Enforcement |
|---|---|---|---|---|
| **Intent** | Purpose invariance | Organizational direction | Purpose persists through personnel changes, reorganizations, and strategy refreshes | B9: resolved emergence produces a Decision, closing the loop to organizational purpose |
| **Commitment** | Obligation invariance | Binding force | Agreements persist until explicitly fulfilled, modified, or released | B1: Work requires Commitment. B2: Commitment requires Capacity. Together they prevent responsibility from vanishing. |
| **Capacity** | Resource invariance | Feasibility truth | Available resources are what they are, independent of what the organization wishes they were | B2: Commitments are backed by verified capacity — feasibility cannot be wished away |
| **Work** | Execution invariance | State transformation fidelity | Same action produces same state change through the state transformation model | B1: Work traces to a commitment — no unauthorized state changes occur |
| **Evidence** | Verification invariance | Epistemic status | Proof quality is intrinsic to the artifact, not dependent on who presents it | B3: every Evidence instance carries exactly one truth type from the controlled vocabulary |
| **Decision** | State-change invariance | Option space completeness | Full state is visible at decision time regardless of who queries | B4: every Decision links to exactly one Account, providing the full state context |
| **Authority** | Delegation invariance | Scope preservation | Delegated authority carries the same scope at every organizational level | B5: every Authority instance traces through a delegation chain that terminates at a root |
| **Account** | Accountability invariance | State fidelity | Actual state is what it is, independent of preferred narrative | B4: decision context is a recorded Account, not a post-hoc reconstruction |
| **Constraint** | Boundary invariance | Rule universality | Same rule produces same prohibition for all actors within scope | B6: every Constraint binds target primitives with a specified enforcement mode |

**The algedonic channel.** §5 lists a tenth row: emergency signaling, enforced by B7 and B8. The algedonic channel is not a tenth conservation law parallel to the nine above. It is the emergency enforcement channel — the mechanism by which violations of the nine conservation laws surface even when standard governance routing is too slow or has been compromised. B7 guarantees that any governed object can be flagged. B8 guarantees that signals route to the authority chain. Together they ensure that conservation law violations are observable, not that they constitute an independent conserved quantity.

**Connection to §8.** Table 9.4.1 completes the argument begun in §8. The control-theoretic derivation chain establishes what the organizational world model must contain (§8). The symmetry-breaking proof establishes why these contents must be explicit (§9.1–§9.2). The conservation laws establish what is preserved when the primitives function correctly — and, by contrapositive, what is lost when they are absent or learned rather than given.

The relationship between layers:

| Layer | What It Specifies | Section |
|---|---|---|
| Organizational symmetry | Why the conservation exists | §9.3 |
| Conservation law | What must be preserved | §9.4 (this subsection) |
| Behavioral invariant | How the preservation is enforced | §5 |
| SHACL shape | The declarative constraint definition | Shapes graph (§5.3) |
| Substrate enforcement | How the constraint is implemented | Implementation (§26) |

### §9.5 Three Symmetry-Breaking Gaps

The conservation laws (§9.4) describe equilibrium properties — what holds before and after state transformations. Three gaps in the symmetry-preservation argument require explicit resolution.

| Gap | Question | Resolution | Architectural Mechanism |
|---|---|---|---|
| **Kinetic symmetry breaking** | What holds *during* a state transformation? Are invariances preserved mid-transition? | Transitions are atomic from a governance perspective. There is no observable mid-transition state where core invariances are broken. | Two-pass validation (§5.3). Pass 1 blocking shapes execute synchronously before the write commits. Pass 2 shapes (scheduled SPARQL) may observe transient states during batch windows, but these are monitoring states, not governance commitments. |
| **Recursion-level invariance** | Do conservation laws enforce identically across recursion levels, or can profile configuration modulate enforcement? | Conservation laws hold at every recursion level — they are universal. Enforcement *mode* may vary by profile and level. | The law is universal; the instrumentation is configurable. §5.3 Pass 1 shapes (Blocking) are invariant across profiles. Pass 2 shapes are profile-configurable in enforcement mode (Blocking, Warning, Logging) but not in the conservation law they enforce. |
| **Detection vs. prevention** | Which conservation laws are structurally *prevented* from breaking vs. *detected* after the fact? | Some conservation laws are exact — structurally prevented by Pass 1 blocking shapes. Others are monitored — detected and flagged by Pass 2 scheduled shapes. | **Exact** (Pass 1, always Blocking): B1, B2, B3, B4, B5 (immediate scope), B6 (binding mode). **Monitored** (Pass 2, profile-configurable): B5-T (transitive chain integrity), B6-U (universal application), B8 (routing availability), B9 (purpose alignment). |

**Kinetic symmetry breaking — resolved architecturally.** The two-pass validation architecture (§5.3) eliminates the mid-transition problem by making governance state transitions atomic. A write either passes all Pass 1 shapes and commits, or fails and does not commit. No governance consumer observes a state where Work exists without a Commitment link or Evidence exists without a truth type.

**Recursion-level invariance — resolved by design.** Conservation laws are universal across recursion levels in the same way that physical conservation laws are universal across scales. The instrument used to measure energy depends on the scale (calorimeter vs. particle detector), but the law of conservation of energy holds everywhere. Similarly, the enforcement mode for B6-U may vary between an EAS deployment and a PAS deployment (§5.4), but the conservation law — same rule produces same prohibition — holds at both levels.

**Detection vs. prevention — resolved by classification.** The distinction between exact and monitored conservation laws is a property of the enforcement architecture, not of the conservation laws themselves. All nine conservation laws are universal. The difference is operational: exact laws are enforced synchronously on every write (zero latency to detection), while monitored laws are enforced through scheduled graph traversal (bounded latency to detection). No profile may downgrade an exact law below Blocking. Profiles may configure monitored laws between Blocking, Warning, and Logging, but may not disable detection entirely (§5.4).

### §9.6 Institutional Isomorphism and Substrate Isomorphism

DiMaggio & Powell (1983) identified three mechanisms by which organizations within the same institutional field converge on similar structures — institutional isomorphism. This convergence is not efficiency-driven but legitimacy-driven: organizations become structurally similar because institutional environments reward similarity.

| Mechanism | Source | Driver | Example | Invariance Character |
|---|---|---|---|---|
| **Coercive** | DiMaggio & Powell (1983) | Regulatory and political pressure | SOX compliance requirements force similar internal control structures | Externally imposed by authority |
| **Mimetic** | DiMaggio & Powell (1983) | Uncertainty-driven imitation | Organizations adopt ERP systems because competitors did | Voluntarily adopted under uncertainty |
| **Normative** | DiMaggio & Powell (1983) | Professional standards and training | Accounting firms converge on similar audit methodologies through professional certification | Absorbed through professional socialization |
| **Substrate** | DLP (original contribution) | Shared compositional infrastructure | Organizations composing decisions from the same primitive grammar converge on the same governance structure | Architecturally imposed by infrastructure |

**Original contribution.** The DLP identifies a fourth isomorphism mechanism — substrate isomorphism — not previously described in institutional theory. Organizations converge structurally because they compose decisions from the same primitive grammar. This is not pressure (coercive), imitation (mimetic), or professional norms (normative). It is structural convergence through shared compositional infrastructure — the organizational equivalent of what group-equivariant architectures do for visual invariances. The invariances are properties of the system, not requirements from external sources.

Conservation-law thinking extends beyond physics to engineered systems. Lehman's Laws of Software Evolution (1980) include empirically discovered conservation properties for complex technical systems — the global activity rate in an evolving software system is statistically invariant over its lifetime. Structural symmetries in designed systems produce measurable conservation properties, whether the designer intends them or not. The DLP makes the intention explicit.

## Scope

This section specifies the symmetry-breaking proof, organizational invariances, conservation laws, and three gap resolutions. It does NOT specify: the primitives themselves (§4), the behavioral invariants that enforce conservation laws (§5), specific SHACL shape implementations (shapes graph, separate artifact), or substrate enforcement mechanisms (§26).

## Locked Design Positions

**Locked.** Log-loss optimization breaks organizational symmetries; preference strengthens with data scale. Nine primitives as nine organizational invariances (purpose, obligation, resource, execution, verification, state-change, delegation, accountability, boundary). Nine conservation laws corresponding to symmetries. Noether's theorem applied to organizational systems. Three symmetry-breaking gaps resolved (kinetic, recursion-level, detection vs. prevention). Original contribution: substrate isomorphism as fourth mechanism of institutional convergence.

## Implementation Requirements

SDK implementations MUST preserve all nine conservation laws across all profiles and levels. SDK implementations MUST enforce Pass 1 exact laws (B1–B6 binding) as architecturally synchronous Blocking shapes. SDK implementations MUST ensure that conservation law violations are detectable even when enforcement mode is lowered to Warning or Logging.

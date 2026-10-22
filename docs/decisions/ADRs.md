# Architecture Decision Record Log

## ADR-0001 — Standalone Project Structure

**Status:** Accepted

**Context:** TArchI is a new standalone project at `/home/fola/tarchi` (its own git repo), not a rewrite of the existing Python prototype oracule-core.

**Decision:** Establish TArchI as an independent project with separate codebase, documentation, and contribution guidelines from oracule-core.

**Consequences:** Clean separation between prototype and production; allows independent evolution; increases maintenance burden; enables focused benchmarking.

**Alternatives considered:** Merge into oracule-core, continue as separate prototype, maintain both under monorepo.

---

## ADR-0002 — Trinary Value Layering

**Status:** Accepted

**Context:** The trinary value is layered across three levels at once: truth at proposition level, node triune state at node level, edge polarity at edge level.

**Decision:** Implement trinary logic with three simultaneous layers: proposition-level truth, node-level triune state, and edge-level polarity, enabling comprehensive representation of knowledge and relationships.

**Consequences:** Richer knowledge representation; increased complexity; 27 distinct node states; enables direct modeling of multi-dimensional information.

**Alternatives considered:** Single-layer trinary system; hierarchical decomposition; hybrid binary/trinary approach.

---

## ADR-0003 — Four Orthogonal Trit Roles

**Status:** Accepted

**Context:** Four ORTHOGONAL trit roles in four layers (not one overloaded axis): Truth (-1 false/0 unknown/+1 true), Polarity on edges (-1 opposes/0 undetermined/+1 supports), Epistemic in the ledger (-1 rejected/0 candidate/+1 held), Balance as charge (-1 deficit/0 equilibrium/+1 surplus). Node = (truth, epistemic, balance) = 27 states mapped deterministically onto FOUNDATIONAL/ACTIVE/RESOLVED/EXHAUSTED/QUARANTINED.

**Decision:** Implement four distinct trinary roles across different layers: Truth (proposition evaluation), Polarity (edge relationships), Epistemic (ledger confidence), Balance (node charge). Combine epistemic and balance dimensions into 27 node states with deterministic lifecycle mapping.

**Consequences:** Comprehensive knowledge representation; 27 distinct node states; deterministic progression; enables nuanced reasoning about certainty and stability.

**Alternatives considered:** Single overloaded truth axis; weighted combination approach; probabilistic interpretation.

---

## ADR-0004 — Complete Intelligence Stack

**Status:** Accepted

**Context:** TArchI is a complete self-contained intelligence stack (perceive, reason, generate, act, remember, learn, self-correct) — not a router to other models.

**Decision:** Build end-to-end intelligence stack from ground up: perception (classical NLP), reasoning (derivation), generation (constructors), action (modulations), memory (persistent storage), learning (continuous), self-correction (Rubik layer verification).

**Consequences:** Full autonomy; no external dependencies; larger codebase; comprehensive functionality; higher development complexity.

**Alternatives considered:** Router to existing models (LLMs, symbolic AI); modular plugin architecture; minimal core with external components.

---

## ADR-0005 — Archaion Principle

**Status:** Accepted

**Context:** "Archaion" = the principle: Intelligence From The Beginning — derivation from foundational axioms rather than interpolation from a corpus.

**Decision:** Design system that derives all knowledge from first principles and axioms, building upward through logical derivation rather than learning patterns from data.

**Consequences:** Traceable reasoning; verifiable conclusions; no hallucination; requires initial axiom corpus; harder to adopt for new domains.

**Alternatives considered:** Data-driven interpolation; hybrid approach; purely empirical learning.

---

## ADR-0006 — Staged Development

**Status:** Accepted

**Context:** Scope is staged into milestones M0-M7 rather than fixed up front; the layers interconnect, so each stage is separately verifiable.

**Decision:** Break down development into 8 sequential milestones (M0-M7) with incremental feature delivery. Each milestone builds on previous layers while maintaining independent verifiability of completed functionality.

**Consequences:** Manageable development; regular validation points; flexible scope adjustment; interdependency management complexity.

**Alternatives considered:** Waterfall with full specification; agile sprints; feature-flagged incremental delivery.

---

## ADR-0007 — Rust Core + Python Bindings

**Status:** Accepted

**Context:** Rust core + Python research bindings. Python is never the runtime (per-object overhead would break the 2 GB RAM budget). Rust gives no_std, SIMD, Android NDK and WASM targets.

**Decision:** Implement core system in Rust for performance and portability, with Python bindings for research and prototyping. Runtime will be Rust-only to maintain memory constraints and performance requirements.

**Consequences:** High performance; multiple platform targets; Python accessibility; compilation complexity; learning curve for Rust.

**Alternatives considered:** Python-first implementation; Go runtime; C++ core with Python bindings.

---

## ADR-0008 — Balanced Numeral Family

**Status:** Accepted

**Context:** Numeral family named Balradix: Tridak = base-3 balanced digit {-1,0,+1}; Baldec = base-10 balanced digit {-5..+5}. Rationale: 0-9 came together and negatives are the opposite representation of the same magnitudes; balanced radix needs no sign bit, negates for free and has a single canonical zero.

**Decision:** Adopt balanced numeral system: Tridak (base-3 balanced digits {-1,0,+1}) for atomic operations, Baldec (base-10 balanced digits {-5..+5}) for higher-level encoding. This eliminates sign bits and enables efficient negation.

**Consequences:** Efficient arithmetic; canonical zero representation; no separate sign; intuitive to humans familiar with decimal; specialized implementation required.

**Alternatives considered:** Binary representation; standard base-10; pure trinary without balancing; custom encoding schemes.

---

## ADR-0009 — Binary Control Group

**Status:** Accepted

**Context:** oracule-core is frozen as the BINARY/FLOAT CONTROL GROUP for the trit-vs-bit A/B, with its bug at learning.py:211 (ACTIVE used but never imported) fixed first; later it will be upgraded to consume .tarchi packages.

**Decision:** Use fixed oracule-core as control group for A/B testing against TArchI implementation. Fix the known bug (learning.py:211) first, then extend it to support .tarchi packages for comparative analysis.

**Consequences:** Direct performance comparison; bug fixation opportunity; backward compatibility; enables validation of improvements.

**Alternatives considered:** New reference implementation; independent comparison framework; differential testing.

---

## ADR-0010 — Constructive Generation

**Status:** Accepted

**Context:** Generation is CONSTRUCTIVE via a Realizer trait adapter (answers assembled from the resolved substrate plus derivation path). NO LLMs anywhere in the system, ever. Consequence: language is precise, factual and cited, but not charming; open-ended chat and creative prose are out of scope for v0.1.

**Decision:** Implement generation using constructive reasoning: Realizer trait adapter assembles answers from substrate facts plus derivation paths. Use purely symbolic reasoning with no machine learning components.

**Consequences:** Precise, verifiable outputs; no hallucination; limited expressiveness; not suitable for creative writing; requires explicit knowledge base.

**Alternatives considered:** Neural generation; symbolic generation only; hybrid approaches; template-based generation.

---

## ADR-0011 — Curriculum-Based Learning

**Status:** Accepted

**Context:** The CURRICULUM replaces gradient descent: corpus -> deterministic template extraction (patterns + classical classifier; precedent TextRunner 2007: 11M triples from 500M sentences at ~91% precision with no neural net) -> canonicalization (entity resolution, alias collapse, Balradix IDs) -> verification against axioms (Rubik layer) -> induction of rules/schemas (ILP precedent Aleph/Progol) -> promotion with Baldec strengths + graduation on stability/redundancy/load-bearing -> per-session decay. Low-confidence or contradicting facts are QUARANTINED and never enter the active substrate. Consequence: there is no training step and no model in the LLM sense; the substrate IS the knowledge; learning is continuous, GPU-free, no catastrophic forgetting; a wrong fact is fixed by editing one node.

**Decision:** Replace model training with curriculum-driven knowledge acquisition pipeline. Process: corpus extraction → template mining → canonicalization → axiom verification → rule induction → promotion with quality metrics → ongoing decay and quarantine management.

**Consequences:** No neural network training; continuous learning; verifiable facts only; GPU-free operation; single-fact correction; complex pipeline with multiple stages.

**Alternatives considered:** Traditional ML training; active learning; reinforcement learning; rule extraction from trained models.

---

## ADR-0012 — Closed-Domain v0.1

**Status:** Accepted

**Context:** v0.1 scope: closed-domain RESOLVE + EXPLAIN + CITE over ported domains, deliberately comparable to the binary control on identical queries.

**Decision:** Initial version (v0.1) will focus on closed-domain question answering with RESOLVE (find answers), EXPLAIN (provide reasoning), and CITE (reference sources) capabilities over a curated set of domains. This enables direct comparison with binary control systems.

**Consequences:** Limited domain coverage; high precision requirement; comparable baseline; smaller initial implementation; clear evaluation metrics.

**Alternatives considered:** Open-domain questions; broader feature set; conversational mode; incremental capability expansion.

---

## ADR-0013 — Minimal Schema Mining

**Status:** Accepted

**Context:** v0.1 ships the Curriculum WITHOUT full ILP induction, but WITH minimal schema mining, because schema compression is what makes the 10x size target reachable.

**Decision:** For v0.1, implement only minimal schema mining instead of full ILP induction. Focus on schema compression to achieve the 10x size reduction target while maintaining core functionality.

**Consequences:** Smaller initial package; easier implementation; less comprehensive knowledge extraction; size target achievable; partial rule induction.

**Alternatives considered:** Full ILP induction; no schema mining at all; incremental schema development.

---

## ADR-0014 — Curated Knowledge Corpus

**Status:** Accepted

**Context:** v0.1 knowledge source: a curated corpus plus ground truth (the 4 prototype domains as prose plus real Wikipedia/technical pages on logic, arithmetic and mechanics), so extraction recall/precision and citation coverage are measured rather than guessed.

**Decision:** Initial knowledge base will be curated: prototype domain prose + Wikipedia + technical documentation on logic, arithmetic, and mechanics. This allows measurement of extraction quality and citation coverage.

**Consequences:** High-quality initial facts; measurable performance; domain-specific focus; curation overhead; limited initial coverage.

**Alternatives considered:** Raw web scraping; user-contributed content; semi-automated curation; academic databases.

---

## ADR-0015 — Portable Knowledge Packaging

**Status:** Accepted

**Context:** Knowledge IS packaged: the .tarchi package is TArchI's weights — axioms, rules, facts, index, quarantine, integrity; 4 KiB mmap-able pages; Balradix-coded page table for sub-byte addressing; Tridak as the atomic load/offload unit; two-tier integrity (per-page hash plus semantic re-verification of every fact against the axiom set on load); composable (base + domain + personal) and updatable by delta.

**Decision:** Knowledge packaging format (.tarchi) uses memory-mapped 4 KiB pages with Balradix-coded page tables for sub-byte addressing. Atomic operations use Tridak encoding. Two-tier integrity: page hashes + semantic verification against axioms. Packages are composable and delta-updatable.

**Consequences:** Efficient memory usage; sub-byte addressing; integrity guarantees; composable knowledge; update efficiency; complex format specification.

**Alternatives considered:** Binary format; text-based format; database format; compressed archive format.

---

## ADR-0016 — Astra Unit Definition

**Status:** Accepted

**Context:** "1 Astra" = the verifiable CAPABILITY of a 27B-param reference LLM, measured on BOTH a knowledge-recall suite (verifiable questions with distractors and injected contradictions, scored with citations) AND a derivation suite (multi-step questions scored on the derivation path). Storage is a separate hard constraint in bytes and INT4-param-equivalents. A package that only memorises answers scores zero on derivation, so a database cannot pass as an intelligence.

**Decision:** Define Astra unit as measurable intelligence capability: capability to perform as well as 27B-param LLM on both knowledge recall and derivation tasks. Requires performance on both dimensions, not just memorization.

**Consequences:** Strict intelligence definition; prevents database-only solutions; enables comparison with LLMs; complex evaluation framework; two-dimensional measurement.

**Alternatives considered:** Simple parameter count comparison; knowledge-only measurement; reasoning-only measurement; composite scoring.

---

## ADR-0017 — Size and Memory Targets

**Status:** Accepted

**Context:** Size and RAM targets (INT4 yardstick, 0.5 bytes/param): 1 Astra = 13.5 GB INT4-equivalent reference vs 1.35 GB TArchI target; 3.7 Astra (=100B-param-equivalent) = 50 GB vs ~5 GB package; must operate within a 2 GB resident working set with the package on flash and demand-paged; bytes-per-fact target <=1.5 (acceptable <=2.5); hot pages pinned by the reasoning frontier, not LRU. Tridak density: log2(3)=1.585 bits/trit, integer packing realizes up to 1.667 bits/trit (3->5, 5->8, 8->13, 12->20 bits); 8 trits address 6561 states per byte.

**Decision:** Target TArchI efficiency: 1.35 GB for 1 Astra (vs 13.5 GB reference), ~5 GB for 3.7 Astra (vs 50 GB reference). Must operate in 2 GB RAM with package on flash, demand-paged. Target: <=1.5 bytes/fact, hot pages pinned by reasoning frontier.

**Consequences:** Significant storage reduction; memory-efficient operation; complex addressing; performance optimizations required; aggressive compression.

**Alternatives considered:** More relaxed targets; tiered efficiency goals; optional compression; hardware-specific optimizations.

---

## ADR-0018 — Validation Metrics

**Status:** Accepted

**Context:** Validation is TArchI-native metrics plus a bit-vs-trit A/B on identical code paths: state density, knowledge density, latency, correctness, contradiction detection and self-correction, citation coverage and unsupported-claim rate, device footprint, and the Astra rating.

**Decision:** Validate using TArchI-native metrics: state density, knowledge density, latency, correctness, contradiction detection/self-correction, citation coverage, unsupported-claim rate, device footprint, and Astra rating. Also run A/B tests against binary implementations on identical code paths.

**Consequences:** Comprehensive validation; direct comparison capability; multiple quality dimensions; complex testing infrastructure; performance benchmarking.

**Alternatives considered:** Industry-standard benchmarks; academic evaluation; user testing; manual verification.

---

## ADR-0019 — Rubik and Chess Layers

**Status:** Accepted

**Context:** Rubik's Cube and Chess map onto two layers: the Rubik layer is the deterministic VERIFIER (closed finite group action, illegal states unrepresentable, zero hallucination by construction); the Chess layer is the adversarial PLANNER (branching, value-estimating). Nubits are the generalization: project ALL legal successor states of a state at once, each with polarity and confidence, instead of sampling one next token.

**Decision:** Implement two reasoning layers: Rubik layer for verification (deterministic, finite-state, zero hallucination), Chess layer for planning (adversarial, value estimation). Generalize to Nubits: project all legal successor states with polarity and confidence.

**Consequences:** Deterministic verification; zero hallucination; adversarial planning; comprehensive state exploration; complex state management; strategic reasoning capabilities.

**Alternatives considered:** Single verification layer; pure search-based planning; Monte Carlo tree search; heuristic evaluation.

---

## ADR-0020 — Classical Perception

**Status:** Accepted

**Context:** Perception without LLMs uses classical NLP (lexer -> morphology -> lexicon -> constraint parser -> semantic extraction) and reads ground truth from language servers for code (rust-analyzer, pyright) instead of guessing from raw text. Image/audio perception is out of scope for v0.1.

**Decision:** Implement perception using classical NLP pipeline: lexer → morphology → lexicon → constraint parser → semantic extraction. Use language servers (rust-analyzer, pyright) for code ground truth. Exclude vision/audio perception from v0.1.

**Consequences:** Precise extraction; verifiable results; no hallucination; dependency on language servers; limited input modalities.

**Alternatives considered:** Neural perception; rule-based NLP; hybrid approaches; multimodal input.

---

## ADR-0021 — Truth Guarantee Scope

**Status:** Accepted

**Context:** The original charter claimed "hallucination is structurally impossible" and "all knowledge from axioms". The M0.6 review demonstrated that axiom consistency cannot establish empirical truth — a false but consistent premise survives all checks. The defensible guarantee is checked entailment conditional on explicit trusted premises.

**Decision:** Replace the universal truth guarantee with: outputs are checked entailments conditional on identified trusted premises, plus source-qualified observations with explicit abstention. Empirical claims are rendered as attributed assertions, not established facts.

**Consequences:** Premise/trust/context metadata required; narrower public claims; no LLM or network needed. The unrestricted 'every claim true' intent is withdrawn.

**Alternatives considered:** Keep universal guarantee (rejected as unsatisfiable); add LLM fallback (rejected by founder constraint).

**Supersedes:** ADR-0005 (Archaion Principle) in part; ADR-0019 (Rubik/Chess) absolute truth claim.

---

## ADR-0022 — Causal A/B Control

**Status:** Accepted

**Context:** ADR-0009 designated the Python/float oracule-core as the sole trit-vs-bit control. The M0.6 review showed this conflates language, allocator, object model, search logic, paging, compression, and numeric semantics. Any result could be dominated by Python object overhead rather than trits.

**Decision:** Keep oracule-core as a historical regression oracle. For causal representation comparison, use the same Rust engine and identical workloads with four storage codecs: i8/enums, packed 2-bit ternary, dense base-3 blocks, and conventional dictionary/delta integers. Report decoding CPU, total bytes, cold/warm latency, and energy where measurable. Freeze formulas before running.

**Consequences:** Additional codec/scoring variants and benchmark bookkeeping; no duplicate full application needed. Isolates trinary representation effect.

**Supersedes:** ADR-0009 as the sole control; retains ADR-0018 identical-code-path intent.

---

## ADR-0023 — Self-Correction Mechanism

**Status:** Accepted

**Context:** ADR-0011 claimed "a wrong fact is fixed by editing one node" with "no catastrophic forgetting". The M0.6 review showed one premise can feed 100,000 derived conclusions; replacing its truth does not update dependents automatically. Per-session decay can discard valid knowledge simply from app opens.

**Decision:** Implement versioned snapshots with reverse dependencies or recomputation. Maintain reverse dependency index or invalidate caches by snapshot and recompute. Keep rejected content excluded as proof premise but retain conflict metadata in a separate ledger. Decay applies only to search/cache priority, never to logical truth. Define scoped authority and effective dates for mutable observations.

**Consequences:** Reverse dependency references cost at least 4 bytes/link if dense; 13.2 GB for 1 per 3.3B assertions before compression; recomputation avoids some storage at latency cost. Updates require O(affected closure), not O(1).

**Supersedes:** ADR-0011 one-node-cost and no-forgetting guarantees.

---

## ADR-0024 — Schema Mining Guarantees

**Status:** Accepted

**Context:** ADR-0013 stated minimal schema mining makes the 10x size target achievable. The M0.6 review showed: (a) lossless factoring ≠ inductive generalization; (b) three examples cannot prove a universal rule; (c) induced rules as hard truth create false conclusions; (d) 10x claim depends on unmeasured corpus structure.

**Decision:** v0.1 allows only supplied/reviewed schemas and lossless factoring of stored instances. Mined generalizations are hypotheses with versioned evidence until independently accepted on held-out compositions. Rule section is usable from first rule-bearing package with explicit v0.1 semantics. Full ILP deferred to M7. The 10x target is a falsifiable hypothesis, not an established property.

**Consequences:** Defer full induction; bounded factoring experiment with exception bytes and expanded-query cost measured. Reduces first-build scope.

**Supersedes:** ADR-0013 guaranteed size consequence.

---

## ADR-0025 — Package Storage Contract

**Status:** Accepted

**Context:** ADR-0015 specified trit-granular residency, unconditional eager semantic verification on load, stronger-wins merge, and global alias uniqueness. The M0.6 review found: (a) logical page ≠ OS page (Android 16 KiB); (b) entropy coding destroys independent page access; (c) 3.3B fact verification at 1M/s = 55 minutes; (d) merge order changes answers; (e) 8 trits ≠ 1 byte.

**Decision:** Replace with: block codec with restart points/offset indexes; separate encoded/decoded cache caps; logical Tridak ≠ physical page. Lazy structural checks on load; authenticated manifest/root excluding self-reference; proof checks keyed to immutable theory/dep hashes. Namespaced immutable assertions with deterministic resolution function per canonical signed proposition. Evidence sets preserved. Atomic manifest publication; failed updates retain prior snapshot.

**Consequences:** Block decoding and coarser faults; ~0.8% digest overhead; evidence-set storage; 1-2 weeks for format/failure-model design. Format keeps 4 KiB logical blocks; physical mapping accounts for larger pages.

**Supersedes:** ADR-0015 trit-granular residency, eager full verify, merge policy, sub-byte header claims.

---

## ADR-0026 — Astra Capability Vector

**Status:** Accepted

**Context:** ADR-0016 defined Astra as "verifiable 27B LLM capability on both recall AND derivation" but ADR-0017 treated it as additive (3.7 Astra = 100B equivalence). The M0.6 review showed: (a) parameter count ≠ capability; (b) duplicate packages add zero coverage; (c) 90% ceiling cannot multiply to 333%; (c) database + recursive query can genuinely derive, so "database scores zero" is a grading policy, not architecture property.

**Decision:** Replace scalar additive Astra with a versioned capability report vector: recall accuracy, multi-step proof correctness, coverage/abstention, contradiction handling, total artifact bytes, device latency. If retaining the brand, "Astra-v0 suite parity" means non-inferiority on both locked suites against a frozen reference configuration. Remove fractional-Astra extrapolation and linear parameter equivalence.

**Consequences:** Additional evaluation if 100B parity claimed; no runtime dependency or reference-model download authorized. Protocol, reference checkpoint, prompts, evidence access, scoring, thresholds fixed before testing.

**Supersedes:** ADR-0016 additive unit; ADR-0017 100B equivalence.

---

## ADR-0027 — Size and Memory Targets (Revised)

**Status:** Accepted

**Context:** ADR-0017 claimed 1.35 GB per Astra, 5 GB for 3.7 Astra, 2 GB resident with 1.6 GB hot set + 200 MB engine. The M0.6 review showed: sample is 7.6 B/fact total → ~25 GB at 3.3B facts; even optimistic raw triples cost 30.5 GB; 1.8 GB app allocation leaves ~200 MB for OS on a literal 2 GB phone; 390K pinned pages already exceed 1.6 GB.

**Decision:** Retain 2 GB device as acceptance constraint. Replace claimed derivation with: measured whole-artifact bytes and application budget. Provisional app budget: 512 MiB (256 MiB cache, 64 MiB query/frontier, 64 MiB proof/workspace, 64 MiB engine, 64 MiB headroom). 5 GB artifact target retained as falsifiable aspiration pending compression evidence. Move device feasibility gate before M5.

**Consequences:** Smaller cache can increase page faults; early device stress gate. No assumption of new hardware or funding.

**Supersedes:** ADR-0017 allocation model and 10x derivation.

---

## ADR-0028 — Rubik/Chess Layer Guarantees

**Status:** Accepted

**Context:** ADR-0019 claimed "zero hallucination by construction" (Rubik), "all legal successors at once", and "Chess = adversarial planner with shortest-path guarantee". The M0.6 review showed: (a) De Morgan example admits ¬(P∧Q) from P,Q; (b) fan-out can be 10M+; depth-6 = 10¹² states; (c) no opponent model, Q is uncalibrated heuristic; (d) proof DAG needed, not linear path.

**Decision:** Keep verifier/search separation. Replace with: independent proof checker + bounded heuristic search with explicit budgets (100K applications, 64 MiB frontier, 2s). Return `ResourceLimit` — never false. Proof DAG with premise IDs, substitution, snapshot hash. No universal truth, adversarial, or shortest-path guarantee by analogy.

**Consequences:** Loses unconditional completeness under resource limits; must be explicit to callers. Keeps sound verification kernel.

**Supersedes:** ADR-0019 absolute/eager claims; retains split architecture.

---

## Open questions (resolved or superseded)

### (Q1) which reference LLM defines 1 Astra — **RESOLVED**: Ada replaced by capability vector (ADR-026). Frozen 27B reference checkpoint still needed for dev-time comparison; candidate Gemma-2-27B (exact 27B, license-gated), Qwen2.5-32B, Mistral-Small-24B. Pick one documented checkpoint before testing; access/license/quantization/prompts frozen in protocol.

### (Q2) which realization registers ship in v0.1 — **RESOLVED**: answer, explain-with-proof, conflict/unknown notice only. Source qualification required; compare/summary deferred. ADR-023/ADR-028 enforce sentence-level provenance.

### (Q3) confirm browser/PWA as the first device target — **RESOLVED**: native Rust arm64 is the resource acceptance target; PWA optional demo only. Minimal CLI/harness on actual 2 GB phone; bounded file reads, 512 MiB budget. Move feasibility spike ahead of M5 (ADR-027). If tooling/access forces PWA, timebox 2-day spike on real device; do not claim mmap equivalence.


# TArchI Charter

**Status:** Draft, 2026-10-22 (revised post M0.6 review)

## Author
Fola Adam (creator)

## Core Thesis

**Archaion = "Intelligence From The Beginning"**
- TArchI starts from foundational axioms and DERIVES conclusions, instead of starting from a corpus and interpolating
- TArchI is NON-PARAMETRIC intelligence. It never uses LLMs — at any layer, ever
- Reasoning is search + verification over an explicit, inspectable state space (classical "knowledge-based + search" school), not gradient descent
- **Outputs are checked entailments conditional on explicit trusted premises, plus source-qualified observations with abstention.** Axiom consistency alone does not establish empirical truth.

**Technical Substrates**
- Numeric substrate: Trinary (-1, 0, +1) Tridak digits, packaged in "Balradix" balanced-numeral family
- Every factual claim is citable and machine-verifiable; hallucination is prevented by construction (checked derivation + source qualification), not by a universal truth guarantee
- Target: verifiable knowledge operating on 2 GB of RAM (device), package on flash, demand-paged

## Four Orthogonal Trit Roles

One mechanism, four readings, each in its own layer:

| Role | Layer | Values | Logic |
|------|-------|--------|-------|
| **Truth** | Proposition level | -1 false, 0 unknown, +1 true | Kleene strong three-valued logic (NOT/AND/OR/IMPLIES) |
| **Polarity** | Edge level | -1 opposes, 0 undetermined, +1 supports | Weighted relationship direction |
| **Epistemic** | Ledger level | -1 rejected/quarantined, 0 candidate, +1 held | Trust verification status |
| **Balance** | Charge level | -1 deficit, 0 equilibrium, +1 surplus | Decay, stall detection, frontier selection |

**Node States**: (truth, epistemic, balance) = 3³ = 27 states

| Lifecycle Label | Truth | Epistemic | Balance | Note |
|----------------|-------|----------|--------|------|
| **FOUNDATIONAL** | +1 | +1 | 0 | Axiom set entry, equilibrium |
| **ACTIVE** | Any | +1 | ≠0 | Held, non-zero charge |
| **RESOLVED** | +1 | +1 | 0 | Grounded in FOUNDATIONAL (requires proof) |
| **EXHAUSTED** | 0 | +1 | -1 | Held, zero truth, deficit |
| **QUARANTINED** | Any | -1 | Any | Rejected, in quarantine ledger |

*Lifecycle is a derived classifier using (truth, epistemic, balance) plus axiom-set membership and proof-status flags; it is not a bijection over the 27 triples alone.*

## Intelligence Capability Model

Seven first-class components:

1. **Perceive** - Classical NLP pipeline: lexer → morphology → lexicon → constraint parser → semantic extraction (controlled language; unsupported input rejected)
2. **Reason** - Chess-layer search (Nubit fan-out) + Rubik-layer deterministic verification
3. **Generate** - Constructive Realizer from resolved derivation traces, precise and cited
4. **Act** - Tool/MCP-compatible adapter (deferred beyond v0.1)
5. **Remember** - Paged substrate + per-session decay (decay applies to search priority, not logical truth)
6. **Learn** - Curriculum: extraction → canonicalization → verification → promotion → decay → graduation → schema mining (induced rules are hypotheses until held-out acceptance)
7. **Self-correct** - Contradiction injection + upstream propagation with dependency tracking, plus human oversight (attributable evidence, not infallible override)

## Curriculum (Replaces Gradient Descent)

Continuous knowledge growth through:
1. **Corpus extraction** - Deterministic template extraction (patterns + classical classifier, if explicitly approved)
2. **Canonicalization** - Entity resolution, alias collapse, Balradix-coded IDs (scoped aliases with unknown/ambiguous outcomes)
3. **Verification** - Against axiom set (Rubik layer); empirical claims become source-qualified observations
4. **Schema induction** - Lossless factoring of stored instances; mined generalizations held as hypotheses with versioned evidence
5. **Promotion** - Baldec strengths, graduation on stability + redundancy + load-bearing
6. **Decay** - Per-session degradation of *search priority only*; logical truth never decays
7. **Quarantine** - Low-confidence or contradicting facts never enter active substrate as proof premises

**Key property:** Knowledge grows by adding pages, not re-running gradient descent. Wrong facts fixed by dependency-aware invalidation and recomputation, not by editing one node in isolation.

## Intelligence Capability: Astra (Revised)

**1 Astra** = a portable, versioned knowledge-and-rules artifact whose capability is measured by a **frozen two-suite capability vector** against a documented 27B-param reference checkpoint:
- Knowledge-recall suite (verifiable questions, distractors, injected contradictions) — scored on correctness with citations
- Derivation suite (multi-step questions requiring proof from axioms + rules) — scored on the derivation path

**No additive parameter equivalence.** 3.7 Astra does not mean 3.7× capability of a 27B model. Capability is reported as a vector:
- Recall accuracy (conditional on answering)
- Derivation correctness
- Coverage / abstention rate
- Contradiction handling (detection + correction)
- Total artifact bytes
- Device latency (cold/warm p50/p95)

**Size metrics (falsifiable targets, not established properties):**
- 1 Astra artifact target: ~1.35 GB (vs 13.5 GB INT4-equivalent reference)
- 3.7 Astra artifact target: ~5 GB (vs 50 GB INT4-equivalent reference)
- **Must operate within a 2 GB device RAM**, with an **application budget of ≤ 512 MiB** (provisional; adjusted from device measurement)
- Storage efficiency target: **1.5–2.5 bytes/fact** for curated schema-heavy knowledge (measured whole-artifact, not Fact-section marginal)

**Technical specs (revised):**
- log2(3) = 1.585 bits per trit (theoretical)
- Integer packing: ceil(k×1.585) bits; 8 trits require 13 bits (not 1 byte)
- 5 trits fit in 1 byte (243 states); 8 trits in 13 bits (6561 states)
- Rubik's cube: 4.3252e19 states = 66 bits vs 42 trits
- Chess: ~4.8e44 positions = 149 bits vs 94 trits
- LLM storage: ~10–15 bytes/fact (unverified baseline) vs TArchI target 1.5–2.5 bytes/fact (schema-heavy curated)

## Honest Limits (Updated)

- **Trade-offs:** Open-domain recall and fuzzy similarity for verifiability, density, updateability, footprint
- **Generalization gap:** LLM weights carry implicit generalities (grammar, style) that no fact table expresses
- **Storage claim holds** for schema-heavy curated verified knowledge, not open-domain trivia; 10× is a falsifiable target pending matched-capability measurement
- **Language limitation:** Precise and cited, but not charming. No open-ended chat or creative prose in v0.1
- **Weakest link:** Extraction recall on prose
- **Truth boundary:** Axiom consistency cannot establish empirical truth; a false but consistent premise survives all checks. Outputs are conditional on premises.
- **Fallback:** If 27B reference parity not met, report absolute scores; do not rename baseline performance into a radix breakthrough

## Non-Goals (v0.1)

- LLMs of any kind
- Open-domain QA
- Conversational/charming prose
- Image/audio perception
- Code perception via LSP
- Multi-user/cloud deployment
- Full ILP induction
- Unbounded search / exhaustive state projection

## Glossary

- **Tridak**: Trinary digit system (-1, 0, +1)
- **Balradix**: Balanced-numeral family packaging Tridak digits
- **Baldec**: Strength metric for promoted facts (11-level ordinal score, stored in 4 bits)
- **Nubit**: Chess-layer search fan-out structure
- **Astra**: Portable knowledge artifact; capability measured by frozen two-suite vector
- **Rubik layer**: Verification against axiom set
- **Chess layer**: Search over Nubit fan-out
- **Curriculum**: Knowledge growth pipeline (extraction → canonicalization → verification → promotion → decay → graduation → schema mining)
- **Substrate**: Underlying data structure + storage management
- **Realizer**: Constructive generation from derivation traces
- **.tarchi package**: TArchI distribution format

## Cross-References

- `tridak.md` - Tridak digit specification (revised algebra)
- `nubit.md` - Nubit search structure (revised inference, Q, proof DAG)
- `package-format.md` - Distribution format (revised header, provenance, merge, integrity)
- `decisions/ADRs.md` - Architecture decisions (including ADR-021..028)
- `../discussion/discussion.json` - Full decision record (corrected)
- `PLAN.md` - Implementation plan (reordered, M0.5 gaps added)

## Implementation Context

**Control group**: oracule-core (Python/networkx knowledge-graph engine) is frozen as the binary/float control group for A/B benchmarking after its bug is fixed (`learning.py:211` ACTIVE NameError). It will later be upgraded to consume .tarchi packages.

**Causal A/B**: Same-Rust codec ablation (i8/enum, 2-bit packed, dense base-3, dictionary/delta) against identical workloads; Python prototype retained as regression oracle only.

## See Also

- oracule-core prototype (control group)
- TArchI design documents in sibling `/tarchi/` directory

(End of file)
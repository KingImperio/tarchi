# TArchI — True Archaion Intelligence (revised, 2026-10-22 post M0.6 review)

A trinary (-1, 0, +1) intelligence: engine + portable knowledge package ("Astra"), designed to carry **verifiable knowledge operable on 2 GB RAM device**.

## Context

`oracule-core` (Python/networkx) already prototypes a triune tension tensor `T = [info, contra, reframe]` as continuous floats, plus Matter persistence, graduation rules and `Q(t)` scoring (`engine/dynamics.py:22`, `learning.py:173`, `main.py:137`). It is a float/binary-era prototype.
TArchI is the successor: a **non-parametric, trinary, self-verifying** intelligence that reasons by **derivation** (search + verification over explicit state) rather than by gradient descent, and that **never uses LLMs** — at any layer, ever.

Key insight driving the architecture: an LLM's bytes are mostly redundancy, syntax and unverified trivia, not facts. An explicit, deduped, schema-compressed, verified substrate is **denser per unit of knowledge** than parametric storage, and every claim it makes is citable and checkable.

## Locked decisions (updated post M0.6)

| Decision | Value |
|---|---|
| Location | New standalone project `/home/fola/tarchi`, its own git repo (home dir is itself a repo — do not commit tarchi into it) |
| Numeric substrate | Trinary: Tridak `-1/0/+1`; Balradix family (Tridak = base-3 digit, Baldec = 11-level ordinal score `-5..+5`) |
| Trit roles | 4 orthogonal, layered: **Truth** (proposition), **Polarity** (edge), **Epistemic** (ledger), **Balance** (charge) |
| Runtime | Standalone & offline-first. No LLM in any layer. Rust core + Python research bindings |
| Knowledge unit | **1 Astra** = portable artifact measured by frozen two-suite capability vector (recall + derivation) |
| Package | `.tarchi` package = axioms + rules + facts + index + quarantine + integrity, paged, composable, delta-able |
| Size target | **Falsifiable targets:** 1 Astra ~1.35 GB, 3.7 Astra ~5 GB; **app budget ≤ 512 MiB** on 2 GB device |
| Generation | Constructive Realizer + `Realizer` trait adapter. No LLM renderer |
| Learning | Curriculum: extraction → canonicalization → verification → promotion → decay (priority only) → graduation → schema mining (lossless factoring in v0.1) |
| v0.1 scope | Closed-domain resolve + explain + cite, curated corpus with ground truth |
| Validation | TArchI-native metrics + bit-vs-trit A/B (same-Rust codec ablation; oracule-core = regression oracle) |
| `oracule-core` | Frozen as binary control group; fix `learning.py:211` `ACTIVE` NameError first; later upgraded to consume `.tarchi` packages |

## Non-goals (v0.1)

LLMs of any kind · open-domain QA · conversational/charming prose · image/audio perception · code perception via LSP · multi-user/cloud · ILP induction beyond lossless schema factoring · unbounded search / exhaustive state projection.

## Architecture

```
 Perception ──────────► Substrate (trinary, paged) ──────────► Realizer ──► Output
 lexicon/parser        axioms · rules · facts · index          discourse plans
 extraction            polarity/strength/truth/epistemic/balance
        │                       │
        └─ Curriculum ──────────┘        Nubit projection (Rubik verify + bounded Chess search)
        (extract→canon→verify→promote)   legality · fan-out · value · Proof DAG
```

**Role mapping** (one mechanism, four readings — the "all four" resolution):

- Truth trit: `-1` false / `0` unknown / `+1` true → Kleene strong 3-valued logic.
- Polarity trit (edge): `-1` opposes / `0` undetermined / `+1` supports.
- Epistemic trit (ledger): `-1` rejected (quarantined) / `0` candidate / `+1` held.
- Balance trit (charge): `-1` deficit / `0` equilibrium / `+1` surplus → drives decay (priority), stall, frontier.

Node = `(truth, epistemic, balance)` = **27 states**. Lifecycle is a derived classifier using truth/epistemic/balance plus axiom-set membership and proof-status flags; it is not a bijection over the 27 triples alone.

**Balance aggregation** (revised): Logical truth from proof status under explicit premises; balance = outstanding work / unresolved constraints. Evidence voting is not entailment; charge uses separate ledger, not the same difference as truth.

**Q (quality)** — three trit components mapped to 7 distinct Baldec values via specified quantization thresholds. Q is a search priority heuristic; no opponent model, no shortest-path guarantee.

**Nubits** (Rubik-inspired projection): a node's fan-out of **legal next states** via indexed, resumable iterators with explicit budgets, each Nubit = `{target, polarity, confidence: Baldec[0..5], legality, rule_id, premise_ids, substitution}`. Rubik layer = deterministic verifier (illegal states are unrepresentable → type error). Chess layer = best-first search over the fan-out with structural value heuristic.

**Curriculum** (the training analogue): corpus → lexical/syntactic parse → template extraction → confidence model → canonicalization (scoped entity resolution, alias collapse, Balradix IDs) → verification against axioms → promotion with Baldec strength → decay (search priority only) → graduation (stability + redundancy + load-bearing) → **lossless schema factoring** (Rule section usable, mined rules are hypotheses). Low-confidence or contradicting facts go to **quarantine** (audit only, never proof premises).

**Numeral system (Balradix, revised):** Odd bases only. Base-3 carry: `total = s + c_in; c_out = sign(total) if |total|>1 else 0; d = total - 3*c_out`. Tridak dense packing: `k` trits → `ceil(k·log₂3)` bits (3→5, 5→8, 8→13, 12→20 bits). Runtime 2-bit encoding for hot path; dense base-3 for storage. Baldec = 11-level ordinal score in 4 bits, not positional radix-10.

## Package format `.tarchi` (Astra, revised)

4 KiB pages; `mmap`-able; page table offsets/lengths as fixed little-endian `u64`. Tridak = atomic *logical* load/offload unit; block codec with restart points for entropy-coded pages.

| Section | Contents | v0.1 |
|---|---|---|
| Header | magic, format version, tridak base, axiom-set hash, package id, provenance, license, section table (u64 offsets/lengths) | ✅ |
| Axiom | foundational axioms + rule forms (trinary-encoded) | ✅ |
| Entity | canonical entity table, type, aliases (compressed, scoped) | ✅ |
| Fact | triples: Balradix (s,p,o) + polarity trit + unsigned Baldec strength + provenance id | ✅ |
| Rule | supplied/reviewed schemas + lossless factoring; mined rules as hypotheses | ✅ (usable) |
| Index | entity→facts, subject→facts, predicate→facts, trinary offsets | ✅ |
| Quarantine | rejected/contradicted claims — audit only, never proof premises | ✅ |
| Integrity | per-page Blake3, authenticated manifest/root (excludes self-reference) | ✅ |

Density budget: **≤1.5 bytes/fact target, ≤2.5 acceptable** — **whole-artifact density**. Schema compression via lossless factoring measured; 10× is falsifiable target.

**RAM operating model (revised):** package on flash; resident app budget **≤ 512 MiB** (provisional: 256 MiB cache, 64 MiB query/frontier, 64 MiB proof/workspace, 64 MiB engine, 64 MiB headroom). Hot pages pinned by reasoning frontier with pin caps and spill/recompute. Engine overhead ≤ 200 MB within app budget.

## Module layout

```
/home/fola/tarchi/
├── Cargo.toml                       # workspace
├── crates/
│   ├── tridak/      # Trit, BalWord<B>, Baldec, packing, no_std
│   ├── substrate/   # Node/Edge/Triple, graph, canonicalizer, paged store, quantizer
│   ├── axioms/      # axiom set, rule forms, semantics, 27-state mapping
│   ├── nubit/       # projection, legality (Rubik), planner (bounded Chess), value heuristic
│   ├── curriculum/  # extraction templates, confidence model, verifier, promotion, decay, graduation, schema factoring
│   ├── lexicon/     # lexicon, morphology, parser (perception)
│   ├── realizer/    # discourse plans, templated surface form, Realizer trait
│   ├── package/     # .tarchi read/write, pages, index, integrity, compose, deltas
│   ├── engine/      # ingest/query/explain orchestration, hot-set paging, frontier pinning
│   ├── tarchi-cli/  # ingest | ask | explain | inspect | verify | bench | pack
│   └── tarchi-py/   # pyo3 bindings (cargo feature, never the runtime)
├── docs/            # charter.md, tridak.md, nubit.md, package-format.md
├── domains/         # ported from oracule-core/data/domains
├── corpus/          # curated corpus + ground-truth claims
├── bench/           # A/B harness: same-Rust codec ablation, Astra measurement
└── tests/           # property tests, contract fixtures
```

## Milestones (reordered, M0.5 gaps explicit)

- [ ] **M0 — Charter & specs (REOPEN).** `docs/charter.md`, `tridak.md`, `nubit.md`, `package-format.md`, `PLAN.md`, `discussion/discussion.json`. **Property tests for trit algebra** (Kleene truth tables, negation involution, Balradix carry round-trip, dense pack/unpack injectivity). **Exit:** specs reviewed; algebra tests pass; one normative contract with supersession table.

- [ ] **M0.5 — Semantic decisions closed (EXPLICIT GAPS).** 14 gaps from M0.6 triage must be resolved before M1:

| Gap | Description | Owner | Exit Criterion |
|---|---|---|---|
| G01 | **Trit algebra**: B=3 carry with incoming carry; zero encoding; reject invalid 2-bit codes; dense base-3 pack/unpack; Baldec as 11-level score separate from radix-10 | creator | 27 full-adder vectors pass; 531,441 twelve-trit words round-trip injectively |
| G02 | **Inference core**: typed Horn rules, signed atoms, range restriction, tabled demand-driven queries, indexed joins, bounded built-ins, no unrestricted NAF; Proof DAGs with premise IDs, substitution, snapshot hash; independent checker rejects invalid De Morgan example | creator | Conjunction, recursion, explicit negation, conflict, resource-limit fixtures pass; invalid proof rejected |
| G03 | **Evidence & negation**: canonical atom = predicate + typed args + context; signed literal for logical ¬; edge polarity for influence only; unsigned strength/confidence; conflict ledger with precedence | creator | 'not-prime(4)' and 'prime(4)' distinct canonical IDs; double-sign ambiguity eliminated |
| G04 | **Lifecycle & state**: truth/ledger/charge independent; axiom membership separate; total classifier with explicit priority; proof-exhausted tracked apart from truth=0 | creator | All 27 triples classified; FOUNDATIONAL ≠ RESOLVED; ACTIVE ≠ EXHAUSTED; candidate states handled |
| G05 | **Charge & truth update**: logical truth from proof status; balance = outstanding work; deduplicated evidence by provenance lineage; no signed-magnitude double negation | creator | Grounded +5 support can reach RESOLVED; 100 copies don't outweigh one contrary source |
| G06 | **Size & residency**: per-structure budgets at 1M/100M/3.3B; sparse overlays for mutable fact state; block codec with restart points; encoded/decoded cache caps; OS page size independence | creator | Total app ≤ 512 MiB; hot pages ≤ 256 MiB; decoded block cap ≤ 64 MiB |
| G07 | **Load verification**: offline full audit, install-time structural verify, open-time authenticated metadata, lazy page/proof checks keyed to immutable theory/dep hashes; cache verdicts; reject queries with unchecked deps | creator | Cold first supported answer ≤ 5s; full audit time reported separately; no eager 3.3B scan |
| G08 | **Merge & deltas**: namespaced immutable assertions; deterministic resolution per canonical signed prop; evidence sets preserved; tombstones; atomic manifest; content-addressed blocks or logical deltas | creator | All 6 merge orders of 3 packages yield same resolution; interrupted update recovers prior snapshot |
| G09 | **Provenance & IDs**: stable namespaced IDs + page-local ordinals + dictionaries; source content hash/version/span/extraction version; scoped alias resolution with unknown outcome | creator | 1M source spans individually attributable; global dictionary + local codes measured |
| G10 | **Schema compression**: supplied/reviewed schemas + lossless factoring only in v0.1; mined rules as hypotheses with versioned evidence; exception sets counted | creator | 1M-record sweep with exceptions/expansion cost measured; no universal rule promoted from frequency |
| G11 | **Extraction & perception**: controlled-language grammar + deterministic templates; source-qualified empirical assertions; exact spans; held-out negation/quantifier/unit/alias cases; classical classifier only if explicitly approved | creator | ≥500 labeled sentences; ambiguous parses reject; parser coverage vs general prose reported |
| G12 | **Validation & Astra**: frozen benchmark manifest (suites, reference checkpoint/quant/prompts, source access rules, scoring, thresholds, coverage/abstention/contradiction/bytes/latency, fresh compositions, held-out sources/entities/rule compositions, independent proof replay) | creator | Protocol registered before test; database + recursive query does not pass; zero-error bound reported with CI |
| G13 | **Device feasibility**: native arm64 as resource acceptance target; PWA optional demo; minimal CLI/harness on actual 2 GB phone; bounded file reads, 512 MiB budget | creator | Sustained 30 min offline run without process loss; peak process RSS measured |
| G14 | **Document reconciliation**: one normative contract; supersession table (D02→D03); Q1/Q2/Q3 resolved early; issue-contract-acceptance matrix | creator | All drafts agree on v0.1 deliverable; algebra tests pass; prototype bug status verified |

- [ ] **M1 — Core crates.** `tridak`, `substrate`, `axioms`, `nubit`, `engine` skeleton, CLI skeleton. **Exit:** build a domain graph, project Nubits, verify legality, resolve a query and print a ProofDAG, all in RAM.

- [ ] **M2 — Package format.** `.tarchi` read/write, all sections, integrity, compose/merge, deltas. **Exit:** round-trip tests; **tamper test** (corrupt one page → detected + fact re-verified); merge determinism; per-section byte-density report.

- [ ] **M3 — Curriculum.** `lexicon` (lexicon/morphology/parser), extraction templates, classical confidence model (if approved), canonicalizer, verifier, promotion, quarantine, decay (priority), graduation, **lossless schema factoring** (Rule section usable). **Exit:** ingest curated corpus → facts with provenance; measured precision/recall vs ground truth; quarantine rejects known-bad claims.

- [ ] **M4 — Closed-domain loop.** Port the 4 oracule-core domains; curated corpus + ground truth; CLI `ingest/ask/explain/inspect`; minimal Realizer (answer, explain-with-proof, conflict/unknown notice). **Exit:** end-to-end resolve + explain + cite; **unsupported-claim rate = 0** on the evaluated set.

- [ ] **M5 — A/B + Astra measurement.** Freeze oracule-core as regression oracle (fix `learning.py:211` ACTIVE NameError); port same domains/queries; implement reference-LLM harness (dev-time) for the knowledge and derivation suites; compute capability vector of a sample package. **Exit:** metrics table with trit-vs-float deltas on identical formulas (same-Rust codec ablation); ≥1 Astra-rated package (capability vector).

- [ ] **M6 — Device (moved up).** **Early spike:** minimal native CLI/harness on actual 2 GB Android phone (arm64). PWA optional demo only. **Exit:** measured working set ≤512 MiB app budget, cold start, page-fault count; largest package built operates on the device. Full PWA/NDK integration deferred.

- [ ] **M7 — Induction & generality.** Full ILP, rule induction from instances, more domains, LSP code perception adapter, MCP action adapter. **Exit:** bytes/fact trending to ≤1.5; derivation suite improves.

## Validation (updated)

Metrics (all reported for both trit engine and float control where applicable):

1. **State density** — bits/trits per distinguishable state (cube 66 b vs 42 trits; chess 149 b vs 94 trits).
2. **Knowledge density** — verified facts per KB resident; **whole-artifact bytes/fact** (target ≤1.5).
3. **Latency** — ingest ms/page, resolve ms/query, inferences/sec.
4. **Correctness** — resolution accuracy on the ground-truth claim set.
5. **Contradiction handling** — detection recall/precision on injected contradictions; self-correction rate (dependency-aware); oversight-injection scenario ported from `oracule-core/main.py:137`.
6. **Citation coverage** — answers carrying a derivation path; unsupported-claim rate (target 0).
7. **Footprint** — peak process RSS, cold start, working-set size, page-fault count (device).
8. **A/B deltas** — **same-Rust codec ablation** (i8/enum, 2-bit packed, dense base-3, dictionary/delta); report per-metric trit-vs-float.
9. **Astra rating** — capability vector (recall, derivation, abstention, contradiction, bytes, latency) vs frozen reference on both suites.

Astra measurement needs a reference checkpoint (27B): candidate **Gemma-2-27B** (exact 27B; license-gated), alternatives Qwen2.5-32B / Mistral-Small-24B. Reference runs dev-time only, never on device. Protocol, prompts, evidence access, scoring frozen before test.

## Risks (updated)

- **Bytes/fact > 2.5** (schema compression insufficient) ⇒ 10× target unreachable; mitigation: M3 measures lossless factoring + exceptions; if M5 confirms miss, report absolute scores and abandon parameter-equivalence claims.
- **Derivation suite may not match the reference** ⇒ report absolute scores; do not rename baseline into radix breakthrough.
- **Extraction recall** on prose is the weakest link ⇒ curated corpus with ground truth; domain templates; quarantine absorbs the long tail.
- **No implicit generality** ⇒ TArchI cannot match an LLM on open-domain fluency; stated as a limit in the charter, not hidden.
- **Device access** ⇒ M6 requires actual 2 GB phone and toolchain; if unavailable, report feasibility pending and do not make phone-performance claims.
- **Reference-LLM license/download budget** for dev-time Astra measurement.
- **Python overhead** ⇒ runtime is Rust only; Python is bindings/benchmarks.
- **Home dir is a git repo** ⇒ `/home/fola/tarchi` gets its own repo; never commit it into the parent.

## Open questions (resolved in ADRs)

1. Which reference LLM defines 1 Astra → **ADR-026**: capability vector + frozen checkpoint.
2. Which realization registers ship in v0.1 → **ADR-023/028**: answer, explain-with-proof, conflict/unknown notice.
3. Confirm browser/PWA as first device target → **ADR-027**: native arm64 is acceptance target; PWA optional.

## Future integration

`oracule-core` later becomes a consumer of `.tarchi` packages (same domain schema, shared corpus), so its float engine and TArchI's trinary engine can be A/B'd live, and its domains stay authoritative for both.
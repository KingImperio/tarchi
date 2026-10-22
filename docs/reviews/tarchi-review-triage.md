# TArchI M0.6 Review Triage — Passes A, B, C

Generated: 2026-10-22
Source: three GPT-6.1 Sol review passes (blind / with docs / gap hunt)
Rules: M0.6 reconciliation from plan — creator decides every semantic change; nothing lands until sign-off.

---

## Quick Summary of Reviewer Position

**All three passes agree on the same core failures:**

1. **F01 / F01 / F01** — Eight trits cannot fit in one byte (6,561 states need 13 bits). The 10× density claim has no physical basis on binary hardware.
2. **F05 / F05 / F07** — "Hallucination structurally impossible" is unsatisfiable: axiom consistency ≠ empirical truth; a false but consistent premise survives all checks.
3. **F10 / F11 / F11** — The 5 GB / 3.3B facts budget contradicts the supplied sample (7.6 B/fact → ~25 GB at that density).
4. **F07 / F04 / F08** — "All successors at once" is not a bound; fan-out can be 10M+; search can reach 10¹² states at depth 6.
5. **F12 / F26 / F22** — "Astra = 27B params" is not a capability law; additive equivalence and linear scaling are false.
6. **F03 / F03 / F03** — The worked inference example is unsound: it derives ¬(P∧Q) from P,Q via De Morgan without a negation premise.
6. **F07 / F05 / F05** — Lifecycle mapping is not bijective: 11 unmapped states (not 22), overlaps, external data needed.
6. **F06 / F06 / F06** — Three negation encodings remain unresolved; signed confidence doubles polarity.
6. **F15 / F15 / F15** — 2 GB phone ≠ 2 GB app budget; proposed 1.8 GB hot set leaves ~200 MB for OS.
6. **F14 / F17 / F14** — Full semantic load verification is 50s–9h; not compatible with demand paging.

**All three passes keep the core direction** — a bounded, offline, no-LLM, proof-carrying domain reasoner with trinary semantics — but reject the universal guarantees, density claims, and capability equivalence.

---

## Triage Matrix

| Finding | Severity | Type | Disposition | Owner | Exit / Note |
|---|---|---|---|---|---|
| **A-F01 / B-F01 / C-F01** (trit density impossible) | fatal | size/metric claim | **Revise** `charter.md` §technical specs, `package-format.md` §C, `discussion.json` technical_facts. Remove "eight trits = 6561 states in one byte" and "10× from radix". Keep: trinary semantics valid; measure actual codec bytes. | creator | Gate 0 |
| **A-F02 / B-F05 / C-F07** (truth guarantee impossible) | fatal | locked decision | **Overturn** D05 "all knowledge from axioms" and D19 "zero hallucination". Replace with: checked entailment conditional on explicit trusted premises; empirical claims as source-qualified assertions with abstention. **New ADR** required. | creator | Gate 0 |
| **A-F07 / B-F22 / C-F08** (exhaustive projection infeasible) | fatal | locked decision | **Overturn** D19 "all successors at once". Replace with: bounded iterators, query budgets, resource-limit answers. **New ADR** required. | creator | Gate 1 |
| **A-F12 / B-F26 / C-F22** (Astra parameter equivalence false) | fatal | locked decision | **Overturn** D16 additive Astra and D17 100B equivalence. Replace with: frozen two-suite capability vector (recall, derivation, abstention, contradiction, bytes, latency). **New ADR** required. | creator | Gate 3 |
| **A-F03 / B-F03 / C-F02** (trit algebra bugs) | fatal | semantic gap | **M0.5**: fix carry, zero encoding, base-10 cardinality, pack/unpack, dense codec. Spec reject-invalid-code and bounded word lengths. | creator | Gate 0 |
| **A-F04 / B-F09 / C-F06** (evidence semantics conflated) | major | semantic gap | **M0.5**: separate signed literal (logical negation), edge polarity (influence), unsigned strength/confidence, conflict ledger. Canonical atom identity. | creator | Gate 1 |
| **A-F06 / B-F07 / C-F05** (lifecycle non-bijective) | major | semantic gap | **M0.5**: truth/ledger/charge independent; axiom membership separate; total classifier with priority; track proof-exhausted separately. | creator | Gate 1 |
| **A-F10 / B-F20 / C-F20** (self-correction ≠ one-node edit) | major | locked decision | **Overturn** D11 "one-node total cost" and "no forgetting". Require dependency DAG or recomputation; separate decay from logical truth. **New ADR** required. | creator | Gate 2 |
| **A-F08 / B-F08 / C-F08** (charge/truth coupling) | major | locked decision | **Overturn** D03's vote-derived truth/charge. Use proof status for truth; balance = outstanding work. **New ADR** required. | creator | Gate 1 |
| **A-F15 / B-F15 / C-F15** (2 GB app budget unrealistic) | major | size/metric claim | **Revise** `charter.md`, `package-format.md` §J. Propose 512 MiB app budget; measure on device. Remove "1.6 GB hot set + 200 MB engine" allocation. | creator | Gate 2 |
| **A-F11 / B-F04 / C-F03** (inference kernel unsound) | fatal | semantic gap | **M0.5**: specify finite typed Horn rules, signed atoms, range restriction, tabled queries, proof DAGs, independent checker. Reject De Morgan example. | creator | Gate 1 |
| **B-F10 / C-F10** (header wire format broken) | major | semantic gap | **M0.5**: explicit little-endian u64 offsets/lengths, checked enum, alignment, unknown-feature behavior. No repr(C) casts. | creator | Gate 2 |
| **B-F12 / C-F12** (provenance 8 bits insufficient) | major | semantic gap | **M0.5**: namespaced source IDs, page-local dictionaries, escape widths, exact spans, extraction version. | creator | Gate 2 |
| **B-F13 / C-F13** (state storage for 3.3B facts) | major | size/metric claim | **M0.5**: sparse mutable overlays for active facts; immutable default on flash. Publish per-structure sizes at 1M/100M/3.3B. | creator | Gate 2 |
| **B-F14 / C-F17** (mmap + entropy coding conflict) | major | semantic gap | **M0.5**: block codec with restart points; separate encoded/decoded cache caps; logical Tridak ≠ physical page. | creator | Gate 2 |
| **B-F16 / C-F19** (merge determinism insufficient) | major | semantic gap | **M0.5**: immutable namespaced assertions + deterministic resolution function; evidence sets; tombstones; atomic manifest. | creator | Gate 3 |
| **B-F17 / C-F20** (decay ≠ no forgetting) | major | locked decision | **Overturn** D11 "no catastrophic forgetting" via decay. Versioned snapshots; decay search priority only. **New ADR** required. | creator | Gate 2 |
| **B-F21 / C-F21** (extraction precision ≠ pipeline guarantee) | major | semantic gap | **M0.5**: controlled-language grammar, ambiguous parse rejection, exact source spans, held-out negation/quantifier/alias cases. | creator | Gate 2 |
| **B-F28 / C-F28** (documents contradict each other) | major | process | **Reopen M0**. One normative contract; supersession table; resolve Q1/Q2/Q3 early. | creator | Gate 0 |
| **B-F29 / C-F29** (six missing contracts) | fatal | gap hunt | **M0.5**: six contracts (algebra, query semantics, authority/proof, storage+trust, snapshots, budgets) must exist before implementation. | creator | Gate 0 |
| **B-F30 / C-F30** (refusal conditions) | major | gap hunt | **M0.5**: tripwires at F01–F03, soundness fixtures, 1M compression, device gate, utility gate. | creator | Gate 0 |
| **B-F31 / C-F31** (smallest viable product) | major | gap hunt | **M0.5**: 35–45 engineer-day capped MVP (Gate 0–3 plan). Pre-register utility vs baseline. | creator | Gate 0–3 |
| **B-F32 / C-F24** (A/B control not causal) | major | locked decision | **Overturn** D09 as sole control. Same-Rust codec ablation needed. **New ADR** required. | creator | M5 |
| **B-F35 / C-F35** (v0.1 success ≠ 100B equivalence) | major | locked decision | **Overturn** D16/D17 implication. Keep D12 closed-domain scope. | creator | Gate 3 |

---

## Decisions Requiring Superseding ADRs (Creator Must Approve)

| Old Decision | New ADR | Change Summary |
|---|---|---|
| D05 (all knowledge from axioms) | ADR-021 | Replace with: outputs are checked entailments conditional on explicit trusted premises + source-qualified observations. |
| D09 (Python/float sole control) | ADR-022 | Replace with: same-Rust codec ablation (i8/enum, 2-bit packed, dense base-3, dictionary/delta). Keep prototype as regression oracle. |
| D11 (one-node O(1) correction, no forgetting) | ADR-023 | Replace with: versioned snapshots, reverse dependencies or recomputation, decay only search priority. |
| D13 (minimal mining guarantees 10×) | ADR-024 | Replace with: lossless factoring only in v0.1; induced rules are hypotheses until held-out acceptance; measure exceptions. |
| D15 (trit-granular residency, eager load verify, strong-wins merge) | ADR-025 | Replace with: block codec, lazy structural checks, authenticated manifest, namespaced immutable merge, atomic updates. |
| D16 (additive Astra equivalence) | ADR-026 | Replace with: capability vector (recall, derivation, abstention, contradiction, bytes, latency) against frozen reference. |
| D17 (5 GB / 10× / 100B as established) | ADR-027 | Replace with: 2 GB device constraint retained; 5 GB as falsifiable target pending whole-artifact measurement; no parameter equivalence. |
| D19 (all-at-once, universal truth, adversarial/shortest) | ADR-028 | Replace with: independent proof checker + bounded heuristic search; resource-limit answers; no world-truth guarantee. |

---

## M0.5 Semantic Gaps (from triage, added to plan)

| Gap | Description | Owner | Exit Criterion |
|---|---|---|---|
| G01 | **Trit algebra**: carry for B=3 with incoming carry; zero encoding; reject invalid 2-bit codes; bounded dense base-3 pack/unpack; eleven-level score separate from radix-10 digit. | creator | 27 full-adder vectors pass; 531,441 twelve-trit words round-trip injectively. |
| G02 | **Inference core**: finite typed Horn rules, signed atoms, range restriction, tabled demand-driven queries, indexed joins, bounded built-ins, no unrestricted NAF; proof DAGs with premise IDs, substitution, snapshot hash; independent checker rejects invalid De Morgan example. | creator | Conjunction, recursion, explicit negation, conflict, resource-limit fixtures all pass; invalid proof rejected. |
| G03 | **Evidence & negation**: canonical atom = predicate + typed args + context; signed literal for logical ¬; edge polarity for influence only; unsigned strength/confidence; conflict ledger with precedence. | creator | 'not-prime(4)' and 'prime(4)' have distinct canonical IDs; double-sign ambiguity eliminated. |
| G04 | **Lifecycle & state**: truth/ledger/charge independent; axiom membership separate; total classifier with explicit priority; proof-exhausted tracked apart from truth=0. | creator | All 27 triples classified; FOUNDATIONAL ≠ RESOLVED; ACTIVE ≠ EXHAUSTED; candidate states handled. |
| G05 | **Charge & truth update**: logical truth from proof status; balance = outstanding work; deduplicated evidence by provenance lineage; no signed-magnitude double negation. | creator | Grounded +5 support can reach RESOLVED; 100 copies of one source do not outweigh one contrary source. |
| G06 | **Size & residency**: per-structure budgets at 1M/100M/3.3B; sparse overlays for mutable fact state; block codec with restart points; encoded/decoded cache caps; OS page size independence. | creator | Total app ≤ 512 MiB (provisional); hot pages ≤ 256 MiB; decoded block cap ≤ 64 MiB. |
| G07 | **Load verification**: offline full audit, install-time structural verify, open-time authenticated metadata, lazy page/proof checks keyed to immutable theory/dep hashes; cache verdicts; reject queries with unchecked deps. | creator | Cold first supported answer ≤ 5 s; full audit time reported separately; no eager 3.3B scan. |
| G08 | **Merge & deltas**: namespaced immutable assertions; deterministic resolution per canonical signed prop; evidence sets preserved; tombstones; atomic manifest; content-addressed blocks or logical deltas. | creator | All 6 merge orders of 3 packages yield same resolution; interrupted update recovers prior snapshot. |
| G09 | **Provenance & IDs**: stable namespaced IDs + page-local ordinals + dictionaries; source content hash/version/span/extraction version in counted tables; scoped alias resolution with unknown outcome. | creator | 1M source spans individually attributable; global dictionary + local codes measured. |
| G10 | **Schema compression**: supplied/reviewed schemas + lossless factoring only in v0.1; mined generalizations as hypotheses with versioned evidence; exception sets counted; Rule section usable with explicit v0.1 semantics. | creator | 1M-record sweep with exceptions/expansion cost measured; no universal rule promoted from frequency alone. |
| G11 | **Extraction & perception**: controlled-language grammar + deterministic templates; source-qualified empirical assertions; exact spans; held-out negation/quantifier/unit/alias cases; classical classifier only if explicitly approved. | creator | ≥500 labeled sentences; ambiguous parses reject; parser coverage vs general prose reported. |
| G12 | **Validation & Astra**: frozen benchmark manifest (suites, reference checkpoint/quant/prompts, source access rules, scoring, thresholds, coverage/abstention/contradiction/bytes/latency, fresh compositions, held-out sources/entities/rule compositions, independent proof replay). | creator | Protocol registered before test; database + recursive query does not pass; zero-error bound reported with CI. |
| G13 | **Device feasibility**: native arm64 as resource acceptance target; PWA optional demo; minimal CLI/harness on actual 2 GB phone; bounded file reads; 512 MiB budget. | creator | Sustained 30 min offline run without process loss; peak process RSS measured. |
| G14 | **Document reconciliation**: one normative contract; supersession table (D02→D03); Q1/Q2/Q3 resolved early; issue-contract-acceptance matrix. | creator | All drafts agree on v0.1 deliverable; algebra tests pass; prototype bug status verified in its repo. |

---

## Out of Scope (Reviewer proposed but plan excludes)

| Finding | Reason |
|---|---|
| Recommending an LLM, hosted API, or network dependency | Founder-level no-LLM constraint; reviewer challenged it but creator keeps it. |
| Proposing features the intent does not need (open-domain chat, speech, vision) | Charter non-goals list these explicitly. |
| Assuming hardware/funding/team not stated | No such resources documented. |
| General unrestricted first-order reasoning / theorem proving | Closed-domain v0.1 explicitly restricts this. |
| Neural training or gradient descent | No-LLM constraint. |
| Replacing the founder's trinary premise with binary | Creator's chosen substrate; reviewer says measure codecs, not abandon. |

---

## Creator Actions Required

For each **New ADR** above (ADR-021 through ADR-028), the creator must:
1. Accept / reject / modify the proposed replacement
2. If accepted, I will draft the ADR and append to `docs/decisions/ADRs.md`
3. Update any affected spec documents (`charter.md`, `tridak.md`, `nubit.md`, `package-format.md`, `PLAN.md`, `discussion/discussion.json`)

For each **M0.5 gap** (G01–G14), the creator must:
1. Confirm the gap belongs in M0.5
2. Assign an owner (default: creator)
3. Confirm exit criterion is measurable

Once the creator signs off on the ADRs and M0.5 gaps, M0.6 is complete and we can proceed to M0.7 (AGENTS.md) then M1.

---

## Files to Update After Sign-Off

- `docs/decisions/ADRs.md` — append ADR-021..ADR-028
- `docs/charter.md` — revise technical specs, honest limits, remove impossible claims
- `docs/tridak.md` — fix algebra, carry, base-10, pack/unpack, property checklist
- `docs/nubit.md` — fix inference example, Q_baldec, proof DAG spec
- `docs/package-format.md` — fix header, provenance, merge, integrity, density accounting
- `docs/PLAN.md` — update M0.5 gaps, M0.6 exit, reorder device gate, cap MVP
- `discussion/discussion.json` — correct technical_facts, decision statuses, non-goals
- `/home/fola/tarchi-review/out/` — already contains the three review reports (Pass A/B/C)

---

**Next step:** Creator reviews this triage, approves/rejects/modifies each ADR and M0.5 gap. Then I execute the document updates.
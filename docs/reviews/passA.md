# TArchI — Pass A blind design review

Printed in chat because the requested WSL output path remains inaccessible. No report file was written.

Only the supplied [REVIEW.md](https://raw.githubusercontent.com/KingImperio/tarchi-review/master/passA/REVIEW.md) was retrieved. No repository browsing, clone, implementation inspection, or additional research was performed.

## Scope, assumptions, and what I was not allowed to see

I had no access to existing architectural decisions, D-numbers, code, benchmarks, datasets, or device measurements. Consequently, this report challenges statements in the intent; it does not claim that an unseen implementation has particular defects.

Assumptions:

- "2 GB" means total device RAM, not an application allocation. Calculations distinguish decimal GB from binary MiB.
- The target is an ordinary binary Android device without native ternary instructions.
- The system must operate locally; connectivity is optional for importing evidence.
- No LLM is used for ingestion, training-data creation, inference, generation, or fallback.
- Text and structured observations are acceptable initial perception channels. Unrestricted speech and vision are not presumed requirements for the first version.
- All budgets below are proposed engineering limits, not measured results.
- "Fatal" means fatal to the stated universal claim, not to a narrower useful product.

The single most important change I would make is to replace "hallucination structurally impossible" with "every factual assertion must pass a checked evidence or derivation gate; otherwise abstain." That creates an implementable contract while acknowledging that evidence itself can be wrong.

## Findings and independently derived architecture

### F01  [fatal]

**Claim being tested:**  
Every claim can be verifiable and citable, making hallucination structurally impossible.

**What breaks (be concrete — a scenario, numbers, a code path):**  
A trusted source records "the bridge is open." The bridge closes afterward. A perfectly implemented system retrieves the record, verifies its signature, and cites it. The answer is still false now.

Likewise, a valid proof from one false premise can produce a false real-world conclusion. One incorrect universal rule can contaminate thousands of otherwise valid derivations.

**Why our reasoning fails (or where it is merely unproven):**  
Citation establishes attribution. Proof establishes consequence relative to premises and a formal system. Neither establishes unrestricted correspondence with reality. Perception, source honesty, temporal validity, and semantic interpretation remain fallible.

**Recommended change (concrete enough to implement):**  
Permit factual output only as:

1. A source-attributed observation with timestamp and scope.
2. A derived proposition with checked premises and rule identifiers.
3. An explicit uncertainty, conflict, or abstention.

Restrict the structural guarantee to: "No factual assertion is emitted without an accepted evidence record or checked derivation." Keep source reliability and real-world truth separate.

**Does this overturn a locked decision?**  
No identifiable D-number was supplied. It overturns the absolute truth guarantee in the intent.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Provenance storage, proof checking, and abstentions. A derivation with 100 steps and four 8-byte references per step needs at least 3,200 bytes of references alone. General answers become less fluent and less comprehensive.

### F02  [fatal]

**Claim being tested:**  
Changing the numeric substrate to -1, 0, +1 enables the equivalent of a 100B model on a 2 GB phone.

**What breaks (be concrete — a scenario, numbers, a code path):**  
For 100 billion independently stored ternary values:

- Ideal fixed-length information requirement:  
  \(100×10^9×\log_2(3)/8 ≈ 19.81\) GB.
- Five trits per byte: 20 GB.
- Two bits per trit: 25 GB.
- One signed byte per trit: 100 GB.

Even the ideal representation exceeds the entire device RAM by roughly ten times, before Android, activations, indexes, or runtime memory.

At 4 bits per value, 100B values occupy 50 GB. Ideal ternary storage improves that by about 2.52×, not 10×. Against 16-bit storage, the theoretical improvement is about 10.09×, but the result is still nearly 20 GB.

**Why our reasoning fails (or where it is merely unproven):**  
Radix changes encoding density; it does not establish equivalent capability, eliminate parameters, or make required information disappear. Correlation and sparsity may enable further compression, but they require demonstrated structure.

**Recommended change (concrete enough to implement):**  
Build a compact symbolic engine with flash-resident knowledge and bounded working memory. Define "100B equivalent" only through named tasks and measured outcomes, never through a conversion from trits to parameters.

**Does this overturn a locked decision?**  
No identifiable D-number was supplied. It overturns the proposed causal connection between ternary encoding and the 100B target.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Abandons a general model-size equivalence claim. Gains a feasible memory model, but knowledge coverage and language flexibility must be built and measured separately.

### F03  [major]

**Claim being tested:**  
A trinary substrate will improve speed on the target phone.

**What breaks (be concrete — a scenario, numbers, a code path):**  
Five trits fit in one byte because \(3^5=243\). Extracting a particular trit then requires decoding, arithmetic, or a lookup. A packed value is not directly an ordinary machine arithmetic operand.

A 256-entry lookup returning five signed bytes occupies 1,280 bytes, but decoding 1 MiB of packed data produces approximately 5 MiB of byte-valued intermediates if fully expanded.

**Why our reasoning fails (or where it is merely unproven):**  
Storage density and execution speed are different properties. Binary CPUs can spend more instructions decoding compact ternary data than they save in memory traffic.

**Recommended change (concrete enough to implement):**  
Use balanced ternary for serialized domain values and optional compact scoring components. Decode only bounded blocks into native integers. Represent identifiers, addresses, counters, and lengths using native machine types.

Compare five-trits-per-byte storage with two-bit encoding and signed-byte execution on the actual target. Treat wider balanced radix as a versioned experiment.

**Does this overturn a locked decision?**  
No identifiable D-number was supplied. It rejects a literal requirement that all machine computation be physically ternary.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Two representations and conversion logic. A 64 KiB packed block can expand to approximately 320 KiB of signed-byte values. Speed improvement remains unproven until measured.

### F04  [major]

**Claim being tested:**  
The values -1, 0, +1 can also serve as a complete truth and contradiction model.

**What breaks (be concrete — a scenario, numbers, a code path):**  
For proposition P, the system can have:

- No evidence for either side.
- Evidence for P only.
- Evidence against P only.
- Evidence for both.

That is four epistemic states. Three symbols cannot distinguish them all without additional structure.

**Why our reasoning fails (or where it is merely unproven):**  
Numeric representation, truth status, and confidence are separate concepts. Making zero mean both "unknown" and "conflicted" destroys information needed for correction.

**Recommended change (concrete enough to implement):**  
Store positive and negative support independently, each referencing its evidence. Derive four epistemic states from support presence. Encode those states with multiple trits if required; two trits provide nine combinations.

Use explicit negation and a paraconsistent rule discipline so a contradiction does not authorize arbitrary conclusions.

**Does this overturn a locked decision?**  
No identifiable D-number was supplied. It rejects any one-trit truth model, if intended.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Two support channels, conflict handling, and more complex inference. Evidence references dominate the few bits needed for status.

### F05  [major]

**Claim being tested:**  
The architecture can fit and remain usable on a 2 GB Android device.

**What breaks (be concrete — a scenario, numbers, a code path):**  
A design budgeting 1.8 GB to its own heap fails when Android, graphics, other applications, and mapped file pages also need RAM. Memory mapping a large artifact does not make touched pages free.

**Why our reasoning fails (or where it is merely unproven):**  
Total physical RAM is not the process budget. Heap size alone misses native allocations and resident file-backed pages.

**Recommended change (concrete enough to implement):**  
I would build a Kotlin Android shell around a Rust engine, with these provisional resident-memory ceilings:

| Component | Ceiling |
|---|---:|
| UI and managed runtime | 64 MiB |
| Native engine and database overhead | 48 MiB |
| Resident knowledge pages and indexes | 96 MiB |
| Facts, bindings, and inference workspace | 48 MiB |
| Search frontier | 24 MiB |
| Parsing and output buffers | 16 MiB |
| Update workspace | 24 MiB |
| Margin | 64 MiB |
| **Total target** | **384 MiB** |

Apply backpressure around 320 MiB and stop expanding work before exceeding 384 MiB. These are design targets; device measurements may force smaller limits.

Use immutable indexed knowledge on flash, an append-only local update journal, bounded caches, and resumable jobs. Begin with text and structured inputs, avoiding a mandatory resident perception model.

**Does this overturn a locked decision?**  
No. It retains the phone target while imposing an application budget.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Cache misses and bounded searches reduce coverage. Initial latency targets: 1 second for indexed retrieval and 3 seconds for bounded derivation; these are acceptance targets, not performance claims.

### F06  [major]

**Claim being tested:**  
A knowledge graph is sufficient as the knowledge representation.

**What breaks (be concrete — a scenario, numbers, a code path):**  
"Alice lives in Lagos" and "Alice lived in Lagos in 2020" become indistinguishable if represented as an unqualified edge. Two people sharing a name can also be merged incorrectly, spreading false relationships.

**Why our reasoning fails (or where it is merely unproven):**  
A graph supplies connectivity, not semantics, identity resolution, temporal scope, or provenance.

**Recommended change (concrete enough to implement):**  
Use typed propositions with:

- Stable entity identifiers and explicit predicates.
- Typed arguments, units, and quantifier scope.
- Valid-time intervals and observation times.
- Source and exact evidence-location references.
- Independent positive and negative support.
- Derivation references and schema version.

Use indexed tables for propositions, arguments, sources, rules, and dependencies. Keep fuzzy association edges outside the trusted inference path.

For illustration, a 64-byte fixed proposition record makes one million propositions occupy 64 MB before variable arguments, evidence text, indexes, and dependencies. A provisional artifact cap of 512 MiB therefore requires measured coverage tradeoffs.

**Does this overturn a locked decision?**  
No identifiable D-number was supplied. It would overturn unqualified graph edges as the authoritative factual representation.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
More bytes per proposition and substantial schema work. In exchange, claims become inspectable and retractable.

### F07  [fatal]

**Claim being tested:**  
Tridaks and Nubits can project all possible states, predictions, deductions, or probabilities.

**What breaks (be concrete — a scenario, numbers, a code path):**  
Twenty independent ternary variables already have \(3^{20}=3,486,784,401\) assignments. At 16 bytes per explicit state, that is about 55.8 GB.

A search branching ten ways reaches one million leaves at depth six and 100 million at depth eight. These counts exclude internal nodes and duplicate detection.

**Why our reasoning fails (or where it is merely unproven):**  
Compactly naming a state space does not make enumerating or searching it cheap. A distribution over N unconstrained outcomes generally needs N-1 independent probabilities; three numeric symbols do not remove that requirement.

**Recommended change (concrete enough to implement):**  
Define Tridaks and Nubits operationally before treating them as architecture: state fields, legal transitions, invariants, and computational cost.

Use factorized representations and lazy exploration. Set explicit node, depth, time, and memory limits. Report incompleteness when limits are reached. Use fixed-point probabilities encoded in multiple trits only where a calibrated probabilistic model exists.

**Does this overturn a locked decision?**  
No identifiable D-number was supplied. It overturns exhaustive projection as a general operating principle.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Search becomes incomplete outside bounded domains. Factorization introduces independence assumptions that must be tested.

### F08  [major]

**Claim being tested:**  
Chess-like macro strategy plus Rubik-like micro execution supplies a general reasoning procedure.

**What breaks (be concrete — a scenario, numbers, a code path):**  
A deterministic local plan can be invalidated by an external event. A shortest sequence of interface actions may optimize tap count while increasing the risk of an irreversible mistake.

A heuristic search can fill its entire 24 MiB frontier. At an assumed 96 bytes per node, that permits at most 262,144 nodes before allocator and lookup overhead.

**Why our reasoning fails (or where it is merely unproven):**  
The analogy does not define an admissible heuristic, transition model, objective, or response to uncertain observations. Deterministic execution does not imply a deterministic environment.

**Recommended change (concrete enough to implement):**  
My reasoning procedure would be:

1. Parse input into a typed query; reject ambiguous interpretations.
2. Retrieve directly relevant evidence and rules.
3. Run indexed Horn-rule inference with explicit negation.
4. Track every derived proposition's dependencies.
5. For actions, search typed operators with preconditions and effects.
6. Check the resulting proof or plan independently.
7. Reobserve after each external action and replan when preconditions change.

Initially exclude unrestricted function symbols, unrestricted negation, and arbitrary theorem proving. Use a 20,000-expansion default planning limit plus the memory and time caps.

**Does this overturn a locked decision?**  
No identifiable D-number was supplied. It keeps the analogy as inspiration, not as an algorithm.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Explicit domain modeling, incomplete searches, and limited expressiveness. "Shortest" is claimed only when the selected algorithm and completed search justify it.

### F09  [major]

**Claim being tested:**  
Continuous tension tensors and Q(t) provide learning, knowledge quality, or self-correction.

**What breaks (be concrete — a scenario, numbers, a code path):**  
A false proposition can have high "info," low "contra," and a high Q score because contrary evidence has not been ingested. Injecting a contradiction may change scores without retracting any downstream answers.

Three float32 components alone occupy 12 bytes per node, or 12 MB for one million nodes, before graph storage.

**Why our reasoning fails (or where it is merely unproven):**  
The brief gives no units, update equations, calibration, or causal relationship between those scores and correctness. Three continuous components are also not the same thing as three discrete values.

**Recommended change (concrete enough to implement):**  
Keep tension scores as optional scheduling hints. They must never establish factual validity.

Learn through a staged curriculum:

1. Exact lookup and source attribution.
2. Typed arithmetic and unit conversion.
3. Bounded multi-step deductions.
4. Contradictions, source retractions, and temporal changes.
5. Simulated plans with execution failures.
6. Narrow real-world action domains.

Generate candidate rules through bounded symbolic induction or enumerated templates. Quarantine them until checked against counterexamples and held-out cases. Generalized rules remain hypotheses unless justified within a specified formal domain.

**Does this overturn a locked decision?**  
No identifiable D-number was supplied. It overturns scores or graduation states as sufficient evidence of knowledge.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Curated exercises, counterexamples, and slower knowledge promotion. Most learning produces provisional hypotheses rather than immediate authoritative facts.

### F10  [major]

**Claim being tested:**  
Self-correction follows from detecting contradictions.

**What breaks (be concrete — a scenario, numbers, a code path):**  
Premise A supports B, which supports C. Retracting A while leaving cached B or C active produces stale answers.

An attacker could submit a contradiction to a correct premise and cause the system to "correct" itself into error.

**Why our reasoning fails (or where it is merely unproven):**  
Conflict detection does not decide which source to trust. Correction requires dependency tracking, source assessment, invalidation, and consistent publication.

**Recommended change (concrete enough to implement):**  
Use dependency-aware truth maintenance. Mark affected conclusions unavailable immediately, then recompute within bounded jobs. Publish updates as versioned transactions so readers see a consistent snapshot.

Keep original evidence and competing assertions. Human correction is another attributable input, not an infallible override.

At four 8-byte dependency references per derived fact, one million derived facts require 32 MB of raw references before reverse indexes and database overhead.

**Does this overturn a locked decision?**  
No identifiable D-number was supplied. It retains self-correction but replaces a score-driven interpretation.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Dependency storage and potentially large invalidation cascades. Answers may temporarily abstain while a knowledge version is repaired.

### F11  [major]

**Claim being tested:**  
A no-LLM system can perceive and generate across the intended scope without an explicit language architecture.

**What breaks (be concrete — a scenario, numbers, a code path):**  
"Can I take this with that?" lacks resolved entities. "Every student read a book" has multiple quantifier interpretations. A grammar that silently selects one meaning can generate a valid proof of the wrong question.

A fixed template can also omit a condition and turn "P under condition C" into unconditional P.

**Why our reasoning fails (or where it is merely unproven):**  
A graph and planner do not supply open-ended language understanding. Removing LLMs removes one possible language component; it does not eliminate the work that component performed.

**Recommended change (concrete enough to implement):**  
Retain no-LLM operation. Start with controlled language, deterministic grammars, a lexicon, typed forms, and explicit entity selection.

Generate answers from checked proposition structures using templates that preserve negation, time, units, conditions, and evidence references. Unsupported input receives an explicit rejection.

Treat speech, OCR, and vision as optional non-LLM adapters whose outputs are uncertain observations. Define the prohibited model class and audit dependencies to prevent accidental LLM ingestion or fallback.

**Does this overturn a locked decision?**  
No. I would keep no-LLM as a project constraint, while rejecting unproven general language capability.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Lower language coverage and more structured interaction. Optional perception models require separate flash, RAM, and latency budgets.

### F12  [major]

**Claim being tested:**  
One Astra equals 27B parameters' worth of capability.

**What breaks (be concrete — a scenario, numbers, a code path):**  
A small exact calculator can outperform a large language model on arithmetic while failing entirely at paraphrase interpretation. Neither result defines a parameter equivalence.

Dividing 100B by 27B gives approximately 3.70 Astras, but nothing specifies how combining those artifacts combines competence.

**Why our reasoning fails (or where it is merely unproven):**  
Parameter count is an implementation property, not a universal capability unit. The proposed metric has no task distribution, reference system, accuracy threshold, or resource constraints.

**Recommended change (concrete enough to implement):**  
Use "Astra" as the name of a portable knowledge-and-rules artifact. Publish a capability profile containing:

- Task categories and held-out test sets.
- Accuracy and abstention coverage separately.
- Evidence correctness and proof-checking results.
- Peak resident memory and artifact size.
- Cold and warm p50/p95 latency.
- Sustained performance and energy measurements.

If a 27B reference is eventually used, compare named outputs on identical tasks; report task-specific results rather than converting them into Astras.

**Does this overturn a locked decision?**  
No identifiable D-number was supplied. It overturns the parameter-equivalence definition.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Benchmark construction and repeatable measurement. The portable-artifact concept survives.

### F13  [major]

**Claim being tested:**  
Flash-resident knowledge solves the memory problem without unacceptable latency or update cost.

**What breaks (be concrete — a scenario, numbers, a code path):**  
If a query performs 2,000 uncached dependent reads and each averages an assumed 1 ms, storage alone contributes roughly two seconds. At 5 ms, it contributes ten seconds.

Replacing a 512 MiB artifact atomically can require another 512 MiB while the old copy remains active, plus a journal and temporary metadata.

**Why our reasoning fails (or where it is merely unproven):**  
Flash capacity, random-access latency, resident cache requirements, and update safety are different constraints.

**Recommended change (concrete enough to implement):**  
Package an Astra as one versioned SQLite artifact containing propositions, indexes, rule bytecode, evidence excerpts, schema identifiers, and content digests. Keep executable engine code in the application.

Use a separate bounded local delta journal. Compact during maintenance windows. Reserve space for old and new artifacts, validate the replacement, then switch versions atomically.

Test process death and power interruption during import, compaction, and activation.

**Does this overturn a locked decision?**  
No. It retains flash knowledge and a portable artifact.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Approximately twice the artifact size during full replacement, plus local changes. Index locality becomes a core design requirement.

### F14  [major]

**Claim being tested:**  
A self-sufficient system can safely turn perceived content into actions.

**What breaks (be concrete — a scenario, numbers, a code path):**  
An imported document contains an imperative. If ingestion promotes that text into a goal or executable rule, outside content acquires authority.

A plan can also be logically consistent while acting on stale state, targeting the wrong entity, or repeating a completed external operation after a crash.

**Why our reasoning fails (or where it is merely unproven):**  
Evidence, instructions, hypotheses, and executable capabilities require different authority. Proof of a plan's internal consistency does not grant permission to execute it.

**Recommended change (concrete enough to implement):**  
Separate imported content from user goals and executable operators. Use an explicit capability allowlist, typed action arguments, action identifiers for duplicate suppression, and a journal of attempted and confirmed effects.

Start with simulated actions. Add real operators individually, with current-state checks and recoverable failure behavior.

**Does this overturn a locked decision?**  
No. It defines the missing boundary around "act."

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Operator-specific integration and recovery logic. General autonomous action is deferred until its domains are modeled.

### F15  [major]

**Claim being tested:**  
A convincing demonstration establishes intelligence, reliability, or phone feasibility.

**What breaks (be concrete — a scenario, numbers, a code path):**  
A system can pass one scripted contradiction scenario by memorizing that pattern. It can also achieve zero false assertions by abstaining on every question.

With zero observed failures in 1,000 independent representative trials, a rough 95% upper bound on failure probability is still approximately 3/1,000, or 0.3%; this is not a universal guarantee.

**Why our reasoning fails (or where it is merely unproven):**  
Demonstrations do not establish generalization. Accuracy without coverage rewards silence. Short benchmarks miss thermal throttling and growing journals.

**Recommended change (concrete enough to implement):**  
Use held-out and adversarial tests covering:

- Answer correctness and answer coverage.
- Incorrect premises and conflicting sources.
- Ambiguous language and entity collisions.
- Temporal expiry and transitive retraction.
- Search-budget exhaustion.
- Corrupt artifacts and interrupted updates.
- Backgrounding and process death.
- A 30-minute sustained workload on the actual 2 GB device.

For the trusted core, separately verify parser boundaries, inference rules, proof checking, and output rendering. Measure peak total process memory rather than heap alone.

**Does this overturn a locked decision?**  
No. It rejects demonstration success as sufficient validation.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Device access, a reproducible harness, and curated test data. Acceptance may expose much narrower capability than the original ambition.

### F16  [minor]

**Claim being tested:**  
The Rubik, chess, wider-radix, and qubit comparisons establish a computational advantage.

**What breaks (be concrete — a scenario, numbers, a code path):**  
A -9...+9 digit has 19 values and carries at most \(\log_2(19)≈4.248\) bits of information. One independent digit requires five binary bits in a simple encoding. Renaming it does not create additional capacity.

Likewise, encoding twenty ternary variables compactly does not evaluate their approximately 3.49 billion assignments simultaneously.

**Why our reasoning fails (or where it is merely unproven):**  
State count alone does not determine problem difficulty or intelligence. Quantum state and measurement behavior are not reproduced by adding a third classical symbol. The brief's dismissal of qubits supplies no technical argument relevant to this design.

**Recommended change (concrete enough to implement):**  
Keep the names if useful, but define each construct mathematically. Evaluate compression, operation count, search complexity, and measured task performance independently.

Make wider radix an optional serialization experiment after the base architecture works.

**Does this overturn a locked decision?**  
No identifiable D-number was supplied. It overturns the analogies as evidence.

**Cost of the change (memory, time-to-first-answer, complexity, or scope):**  
Minimal runtime cost. Removes speculative claims and postpones a second numeric format.

## Top 5 findings

1. **F01 — Absolute real-world truth cannot be guaranteed by citations or proofs.** Guarantee evidence-gated assertions and checked derivations, with explicit abstention.
2. **F02 — Ternary encoding does not deliver the 100B-on-2GB target.** Even ideal storage for 100B ternary values needs approximately 19.81 GB.
3. **F07 — Representing possible states does not make exhaustive reasoning feasible.** Twenty ternary variables already yield approximately 3.49 billion assignments.
4. **F12 — "27B parameters per Astra" has no defined technical meaning.** Replace it with a portable artifact and a measured capability profile.
5. **F11 — No-LLM operation requires a deliberate language and perception boundary.** A controlled-language symbolic system is concrete; unrestricted understanding remains unproven.

## Decisions I would overturn

No D-numbers were available in this blind brief. Assigning them would fabricate knowledge of locked decisions. The following are intent-level decisions I would overturn:

- **D-number unavailable:** Absolute hallucination impossibility as a claim about the external world.
- **D-number unavailable:** Ternary radix as sufficient justification for 100B-equivalent capability on 2 GB.
- **D-number unavailable:** Astra as a fixed parameter-equivalence unit.
- **D-number unavailable:** Exhaustive state projection as the general reasoning strategy.
- **D-number unavailable:** Any use of tension scores or graduation states as proof of factual correctness.
- **D-number unavailable:** Literal ternary execution throughout an ordinary binary Android runtime, if intended.

## Decisions I would keep and why

- **No LLMs:** A legitimate research boundary. It forces explicit semantics and a narrower, measurable scope.
- **2 GB Android target:** A useful constraint when paired with a substantially smaller process budget and sustained device testing.
- **Flash-resident knowledge:** Appropriate for a larger evidence collection with bounded working memory.
- **Astra as a portable artifact:** Useful for versioning, reproducibility, offline distribution, and inspection.
- **Balanced ternary as an experiment:** Worth testing for compact domain representations; its advantages must be measured.
- **Explicit contradictions and self-correction:** Valuable when implemented through competing evidence, dependency tracking, and versioned retraction.
- **Macro planning plus deterministic local execution:** Useful in modeled domains with bounded search and reobservation.
- **Human oversight:** Useful as attributable evidence and goal authority, while remaining subject to error.

## Verdict

I would build TArchI as a bounded, evidence-carrying symbolic assistant: a native Android engine, typed flash-resident propositions, explicit positive and negative support, restricted rule inference, bounded planning, controlled-language input, checked template output, and transactional learning with dependency-aware retraction. I would target 384 MiB resident memory and initially cap the knowledge artifact at 512 MiB, then revise those limits from device measurements. This can become a useful and inspectable no-LLM system, but the brief does not establish general intelligence, 100B-equivalent capability, a valid Astra conversion, or absolute truth. The decisive advance would be a demonstrated ability to answer, abstain, and correct itself reliably within a published scope—not a new digit vocabulary.
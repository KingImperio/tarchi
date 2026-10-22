# Nubit.md - Core Inference Primitive for TArchI
# revised, 2026-10-22 post M0.6 review

## 1. Nubit Definition

```rust
// Nubit: unit of PROJECTION — one slot in a state's fan-out of possible legal NEXT states
struct Nubit {
    target: NodeId,              // The resulting state node ID if this Nubit is selected
    polarity: Trit,              // Supports (+1), opposes (-1), or undetermined (0)
    confidence: Baldec,          // Unsigned strength [0, +5] with separate polarity
    legality: Legality,          // First-class verdict: Legal or Illegal with rule reference
    rule_id: RuleId,             // The inference rule licensing this transition
    premise_ids: Vec<NodeId>,    // Explicit premise node IDs (for proof DAG)
    substitution: Substitution,  // Variable bindings for this inference step
}

enum Legality {
    Legal,
    Illegal { rule: &'static str }, // Any violation is unrepresentable at runtime
}

// A set of projected successors from one origin node
struct NubitSet {
    origin: NodeId,
    slots: Vec<Nubit>,  // All members checked before use; only Legal survive to planner
}
```

**Field semantics:**
- `target`: Hash of the successor state (deterministic mapping from substrate transition)
- `polarity`: Trit representing the Nubit's stance toward the anchor (-1 oppose, 0 undetermined, +1 support)
- `confidence`: Unsigned Baldec [0, +5] — strength of evidence; opposition is carried by `polarity`
- `legality`: First-class field preventing unrepresentable states; illegal Nubits are excluded from planner consideration entirely
- `rule_id`: Stable identifier of the inference rule (e.g., `axiom:and_intro`)
- `premise_ids`: Explicit list of premise nodes this inference depends on
- `substitution`: Variable bindings used in this inference step

**Why first-class legality?** Violation becomes a type error rather than a probabilistic hallucination. Illegal states are pruned from the universe before they can be reasoned about, preserving semantic integrity.

## 2. Why Project Everything

**Contrast with LLM autoregression:** LLMs predict ONE next token sequentially from a probability distribution, sampling a single path through time. They cannot parallelize over the full successor set because each prediction conditions on the previous sample.

**Contrast with masked diffusion:** Diffusion models predict ALL masked positions in parallel over a token distribution, but lack explicit legality constraints. They can generate tokens that violate substrate invariants, with no mechanism to rule them out without sampling and rejection.

**TArchI projection is third kind:** DETERMINISTIC enumeration of REACHABLE states with polarity and confidence, computed directly from the substrate. Every successor is checkable against hard constraints.

**Cost/benefit:**
- **Bounded fan-out:** Limited to indexed, query-conditioned joins (not "all at once" — see §6)
- **Parallel computation:** Full set computed simultaneously via indexed iteration
- **Checkable members:** Each Nubit's legality is verifiable before use
- **Analogy:** Rubik's position's legal-turn set = NubitSet; Chess position's legal-move set = branching strategy

## 3. The Rubik Layer — Deterministic Verifier

The Rubik Layer validates EVERY projected Nubit against hard substrate constraints before any reasoning occurs.

**Constraint sources:**
- Axiom set (semantic integrity)
- Type constraints (node/edge topology)
- Uniqueness (no duplicate NodeIds)
- Structural invariants (e.g., no cycles in DAG for acyclic relations)

**Verification procedure:**
1. For each Nubit in a NubitSet, apply all constraint checks
2. If any constraint fails, mark Nubit as Illegal with rule ID
3. Only Legal Nubits survive to Chess planning layer

**Violation reporting:** Illegal Nubits carry rule ID allowing precise citation. Example: `Illegal { rule: "axiom:symmetry" }`

**Re-verification on load:** Every fact in a .tarchi package undergoes full re-verification against current axiom set. Package integrity is hash-verifiable, semantic integrity is constraint-verifiable (weights can never have semantic weight — only hash weight).

## 4. Inference Core (NEW — replaces "Chess Layer" section)

The inference kernel is a **finite, typed, function-free Horn rule system** with explicit signed atoms.

### Rule Form
```
Head :- Body1, Body2, ..., BodyN.
```
- `Head` and `Body` are signed literals: `+P(args)` or `-P(args)` (explicit negation)
- Variables are range-restricted (appear in a positive body literal)
- Built-ins: bounded arithmetic (`+`, `-`, `*`, `/`, comparison), typed equality
- No unrestricted negation-as-failure; negation is explicit signed literal
- Recursion permitted with depth bound

### Query Evaluation
- **Demand-driven tabled evaluation** (SLG-style with subgoal memoization)
- Indexed joins on predicate + leading arguments
- Memoization keyed by normalized goal + bindings + axiom/rules snapshot hash
- Returns: `Proved(ref)`, `Refuted(ref)`, `Unknown`, `Conflicted`, `ResourceLimit`

### Proof Objects
Every successful derivation produces a **Proof DAG** (not a linear path):

```rust
struct ProofNode {
    id: ProofNodeId,                    // Sequential namespaced ID
    rule_id: RuleId,                    // e.g., "axiom:and_intro"
    premise_ids: Vec<ProofNodeId>,      // DAG edges to premises
    substitution: Substitution,         // Variable bindings
    conclusion: SignedLiteral,          // +P(args) or -P(args)
    snapshot_hash: [u8; 32],            // Immutable axiom/rules snapshot
}

struct ProofDag {
    nodes: Vec<ProofNode>,
    root: ProofNodeId,                  // The query conclusion
}
```

**Trace properties:**
- Claims are citable: "See ProofNode #47, rule `axiom:and_intro`, premises #45,#46"
- Independent checker replays applications without trusting planner legality labels
- Conjunction requires two premises; De Morgan cannot introduce negation from positive premises alone

**Example (corrected):**
```
Premise 1: +P(a) (axiom:premise_001)
Premise 2: +Q(a) (axiom:premise_002)
Step 3: +R(a,b) := and_intro(+P(a), +Q(a))      // rule:and_intro
Step 4: +S(a,b) := external_assertion(-R(a,b))   // rule:external_obs (source: human_001)
Step 5: CONFLICT(+R(a,b), -R(a,b))               // rule:conflict_detect
```
No step derives ¬(P∧Q) from P,Q alone.

## 5. Value Function Q_baldec

```rust
// Three trit components, each -1/0/+1
struct Coverage { fraction_of_anchor_neighbors_grounded }  // trit: +1 if >0.5, 0 if ≈0.5, -1 if <0.5
struct ChainCompleteness { exists_grounded_path_from_foundational_node } // trit
struct ContradictionAbsence { no_active_conflict_on_path } // trit

// Convert to Baldec [-5,+5] — rounding specified
fn compute_q(coverage: Trit, completeness: Trit, contradiction: Trit) -> Baldec {
    let sum = coverage as i8 + completeness as i8 + contradiction as i8; // range -3..+3
    // Quantization thresholds (7 distinct outputs):
    // sum=-3 → -5, -2 → -3, -1 → -1, 0 → 0, +1 → +1, +2 → +3, +3 → +5
    match sum {
        -3 => -5,
        -2 => -3,
        -1 => -1,
        0  => 0,
        1  => 1,
        2  => 3,
        3  => 5,
    }
}
```
**Notes:**
- Q is a **search priority heuristic**, not a correctness signal
- No opponent model, no shortest-path guarantee — "Chess" is an analogy, not an adversarial objective
- Tie-break: deterministic (smallest ProofNodeId)

## 6. Active Front — The Frontier (Revised)

**Fan-out bound:** Legal successors are produced by **indexed, resumable iterators** with explicit budgets:
- Max 100,000 proof applications per query
- Max 64 MiB frontier/tables
- Max 2 seconds wall time
- Return `ResourceLimit` when exceeded — never false

**Node selection:** Frontier is the node with highest charge among unexhausted, non-conflicted nodes connected to the anchor, bounded by the above.

**Hot-page residency:** Frontier pins its pages (axioms plus active chain) in resident working set. This is a reasoning decision, not LRU cache eviction. **Pin caps and spill/recompute policies are enforced by the pager.**

**Process:**
1. Evaluate charge of all frontier candidates within budget
2. Select highest (fixed tie-break: smallest NodeId)
3. Materialize relevant axioms and path (bounded)
4. Push onto working set
5. Begin derivation steps

## 7. Stall, Exhaustion and Contradiction

**EXHAUSTED nodes:** Stop moving when all outgoing Nubits have been explored or have confidence ≤ -5.

**CONFLICTED nodes:** Active conflict detected via Rubik layer (opposing evidence on same canonical atom). Cannot resolve until conflict is discharged via dependency-aware invalidation and recomputation.

**CONTRADICTION injection:** From human oversight channel reopens RESOLVED nodes when new contradiction evidence appears, triggering recomputation of affected subgraphs (reverse dependencies or bounded recomputation).

**STALL detection:** Nodes that have no Legal outgoing Nubits or have confidence ≤ -5 enter STALL state.

## 8. Worked Example — Propositional Logic Domain (Corrected)

**Substrate state:**
```rust
// Anchor node A
struct NodeA {
    facts: ["P(a)", "Q(a)"],  // Premises
    inferred: [],              // Empty
    contradictions: [],        // Empty
}
```

**Nubit projection (fan-out):**
```
Nubit_017: target=N_B (+R(a,b) = P(a) ∧ Q(a)), polarity:+1, confidence:+4, legality:Legal, rule:and_intro, premises:[P(a), Q(a)]
Nubit_023: target=N_C (external assertion -R(a,b)), polarity:-1, confidence:+5, legality:Legal, rule:external_obs, premises:[source:human_001]
Nubit_042: target=N_D (conflict +R ∧ -R), polarity:-1, confidence:+5, legality:Legal, rule:conflict_detect, premises:[N_B, N_C]
```

**Rubik verification:** All three Nubits legal — no constraint violations. Note: Nubit_023 is an *external observation* (source-qualified), not a theorem from P,Q.

**Chess search path:**
1. Evaluate Nubit_017: Q = compute_q(+1, +1, +1) = +5 Baldec
2. Evaluate Nubit_023: Q = compute_q(-1, 0, +1) = -1 Baldec
3. Evaluate Nubit_042: Q = compute_q(-1, -1, -1) = -5 Baldec
4. Select Nubit_017 (highest Q)

**Resulting Proof DAG (not linear trace):**
```
ProofNode#1:  +P(a)              [axiom:premise_001]
ProofNode#2:  +Q(a)              [axiom:premise_002]
ProofNode#3:  +R(a,b) = P∧Q      [rule:and_intro, premises:#1,#2]
ProofNode#4:  -R(a,b)            [rule:external_obs, source:human_001]
ProofNode#5:  CONFLICT           [rule:conflict_detect, premises:#3,#4]
```

## 9. Invariants and Property Tests

### Checklist for Rust crate `nubit`:

1. **Legality coverage:** Every projected Nubit is either `Legal` or carries a `rule` in `Illegal` variant.

2. **Deterministic fan-out:** Given a substrate state and axiom set, the NubitSet is identical across runs.

3. **Re-verification consistency:** Re-loading a package and re-verifying against axioms reproduces identical legality verdicts.

4. **Proof DAG reproducibility:** Multiple runs with identical inputs produce identical ProofDag sequences (up to namespaced ID allocation order).

5. **Deterministic frontier selection:** Under fixed tie-break rule, frontier node selection is deterministic across runs.

6. **Illegal exclusion:** No Illegal Nubit appears in any successful ProofDag.

7. **Confidence bounds:** Baldec values always within [0, +5] (unsigned); polarity carries sign.

8. **Polarity integrity:** Trit values restricted to {-1, 0, +1}.

9. **Proof checker independence:** An independent checker replays ProofDag applications and verifies each step's premises, substitution, and conclusion without trusting planner labels.

10. **Resource limit distinctness:** `ResourceLimit` outcome is distinguishable from `Unknown` and `Refuted`.

**Test scenarios:**
- Property-based testing: Generate random substrate states, verify all projections are deterministic
- Exhaustive verification: Small domain where all possible NubitSets can be enumerated
- Proof replay: Execute ProofDag independently, verify outcome matches
- Adversarial: Inject false premise, verify conflict detection and invalidation
- Resource exhaustion: Verify ResourceLimit returned at budget boundaries

---

See also: `charter.md`, `tridak.md`, `package-format.md`, `../discussion/discussion.json`

(End of file)
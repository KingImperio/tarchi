# TArchI Specification — Tridak (revised, 2026-10-22 post M0.6 review)

## 1. THE TRIDAK

The **Tridak** is the atomic digit of TArchI, representing the base‑3 balanced radix digit in `{-1, 0, +1}`.

| State | Symbol | Role | Name | Physical storage (i8) |
|-------|--------|------|------|----------------------|
| -1    | `0xFF` | falsum | **NEG** | bit‑pattern `11111111` (two's complement -1) |
| 0     | `0x00` | nullum | **ZERO** | all bits zero |
| +1    | `0x01` | verum | **POS** | bit‑pattern `00000001` |

*Invariant*: A Tridak value is exactly one of the three states; out‑of‑range writes are undefined behavior and must be rejected.

*Negation*: `¬x = -x`. Negation is a single arithmetic negation on the i8 representation (or bitwise NOT on a sign-magnitude encoding, but the runtime uses i8 arithmetic).

## 2. THE BALRADIX FAMILY

For odd base `B ≥ 3`, the **balanced radix** digit range is `[-(B-1)/2, +(B-1)/2]` — exactly `B` canonical digits.

*Why balanced beats signed‑magnitude & two's complement for intelligence*:
- No dedicated sign bit – digit values alone encode sign
- Negation is free (`-d` is arithmetic negation on i8). No two's complement conversion cost
- Single canonical zero (`0`)
- Symmetric arithmetic – `d1 + d2 = -(d1 + d2)'`

**Named members**:
- **Tridak** (base 3): `{-1, 0, +1}` (1 trit, 1.585 bits theoretical)
- **Baldec** (base 10): an **11-level ordinal score** `{-5, …, +5}` stored in 4 bits — *not a canonical radix-10 digit*. Baldec is a scalar confidence/strength metric, independent of positional arithmetic.
- **Other odd bases**:
  - B=5 → `{-2, -1, 0, +1, +2}` (log₂5 ≈ 2.322 bits/digit)
  - B=7 → `{-3, -2, -1, 0, +1, +2, +3}` (log₂7 ≈ 2.807 bits/digit)

Even bases require an asymmetric canonical alphabet or a normalization rule; they are not used in v0.1.

## 3. BALANCED ADDITION AND CARRY (B=3, fixed)

For base 3, given digit sum `s ∈ [-2, +2]` and incoming carry `c_in ∈ {-1, 0, +1}`:

```
total = s + c_in
c_out = sign(total)   if |total| > 1 else 0    // sign(±2)=±1, sign(±1,0)=0
d     = total - 3 * c_out
```

**Base‑3 full-adder table (3×3×3 = 27 combinations)**:

| a | b | c_in | total | c_out | d |
|---|---|------|-------|-------|---|
| +1| +1|  0   |   2   |  +1   | -1 |
| +1| +1| -1   |   1   |   0   | +1 |
| +1| +1| +1   |   3   |  +1   |  0 |
| +1|  0|  0   |   1   |   0   | +1 |
| +1|  0| -1   |   0   |   0   |  0 |
| +1|  0| +1   |   2   |  +1   | -1 |
| +1| -1|  0   |   0   |   0   |  0 |
| +1| -1| -1   |  -1   |   0   | -1 |
| +1| -1| +1   |   1   |   0   | +1 |
|  0| +1|  0   |   1   |   0   | +1 |
|  0| +1| -1   |   0   |   0   |  0 |
|  0| +1| +1   |   2   |  +1   | -1 |
|  0|  0|  0   |   0   |   0   |  0 |
|  0|  0| -1   |  -1   |   0   | -1 |
|  0|  0| +1   |   1   |   0   | +1 |
|  0| -1|  0   |  -1   |   0   | -1 |
|  0| -1| -1   |  -2   |  -1   | +1 |
|  0| -1| +1   |   0   |   0   |  0 |
| -1| +1|  0   |   0   |   0   |  0 |
| -1| +1| -1   |  -1   |   0   | -1 |
| -1| +1| +1   |   1   |   0   | +1 |
| -1|  0|  0   |  -1   |   0   | -1 |
| -1|  0| -1   |  -2   |  -1   | +1 |
| -1|  0| +1   |   0   |   0   |  0 |
| -1| -1|  0   |  -2   |  -1   | +1 |
| -1| -1| -1   |  -3   |  -1   |  0 |
| -1| -1| +1   |  -1   |   0   | -1 |

*Example*: `+1 + +1 = -1` with carry `+1` (no incoming carry).

**Base‑10 balanced example (informal — Baldec is a score, not positional radix-10):**
```
6 + 7 = 13
c = floor(13/10) = 1
d = 13 - 1*10 = 3
```
Result: score `3` with carry `1`. *This is decimal arithmetic for illustration; Baldec does not use positional carry.*

*Why carries are smaller*: Max carry magnitude is 1 vs `B-1` in unbalanced; this enables a packed substrate with minimal overflow.

## 4. PACKING AND DENSITY

Theoretical bits per trit: `log₂(3) = 1.58496`.

| k (trits) | States (3ᵏ) | Bits (ceil(log₂ N)) | Realized bits/trit | Fits in |
|-----------|-------------|---------------------|-------------------|---------|
| 3         | 27          | 5                   | 1.666…            | 1 byte  |
| 5         | 243         | 8                   | 1.6               | 1 byte  |
| 8         | 6,561       | 13                  | 1.625             | 2 bytes |
| 12        | 531,441     | 20                  | 1.6667            | 3 bytes |

*Sub‑byte addressing*: 5 trits map directly to a single byte (0‑255) with 243 possible states. 8 trits require 13 bits (6561 states) — **not one byte**. The claim "8 trits = 6561 states in one byte" is mathematically false and has been removed.

**State‑space comparisons**:
- Rubik's cube: `4.3252×10¹⁹` states → `66` bits vs `42` trits
- Chess: `4.8×10⁴⁴` positions → `149` bits vs `94` trits
- Generic: `bits = ceil(log₂ N)`, `trits = ceil(log₃ N)`

## 5. BALDEC AS A SCALAR

Baldec values `{-5,…,+5}` replace floating‑point weights for edge strength and confidence. It is an **11-level ordinal score**, not a positional digit.

**Float → Baldec canonical mapping** (for promotion/graduation):
```
s = round(10 * w) - 5   // w in [0,1] maps to [-5, +5]
```
| w      | s (Baldec) | Interpretation |
|--------|-----------|----------------|
| 1.0    | +5        | definite support |
| 0.5     | 0         | neutral / undecided |
| 0.0     | -5        | definite opposition |

*Zero semantics*: `0` = neutral/undetermined — no separate contradiction flag needed.
*Positive*: supporting/strengthening.
*Negative*: opposing/weakening.

**Sign convention**: Edge polarity (`-1/0/+1`) and Baldec strength (unsigned magnitude `0..5` with separate polarity) are kept distinct. The earlier mapping `w=0 → -5` (meaning opposition) is deprecated; opposition is represented by polarity `-1` with positive strength.

## 6. TRIT LOGIC (KLEENE STRONG THREE‑VALUED)

Truth tables for the four orthogonal roles (same digit values, different semantic overlays):

**Truth (proposition)**: `-1` = false, `0` = unknown, `+1` = true.
- NOT: `¬x = -x`
- AND: `x ∧ y` = `min(x, y)` (idempotent, commutative)
- OR: `x ∨ y` = `max(x, y)`
- IMPLIES (material, strong Kleene): `x → y` = `max(-x, y)`
  - `+1 → +1 = +1`, `+1 → 0 = 0`, `+1 → -1 = -1`
  - `0  → +1 = +1`, `0  → 0  = 0`,  `0  → -1 = 0`
  - `-1 → +1 = +1`, `-1 → 0  = +1`, `-1 → -1 = +1`

**Polarity (edge)**: `-1` = opposes, `0` = undetermined, `+1` = supports.
**Epistemic (ledger)**: `-1` = rejected, `0` = candidate, `+1` = held.
**Balance (charge)**: `-1` = deficit, `0` = equilibrium, `+1` = surplus.

All four layers share the same trinary digit set — no axis overloading.

## 7. CONVERSIONS AND CANONICALIZATION

**Trit → bits** (dense base-3 accumulation, not 2-bit runtime encoding):
```rust
// Dense encoding: value = Σ (tᵢ + 1) * 3ⁱ  for tᵢ ∈ {-1,0,+1}
// Rejects inputs with |tᵢ| > 1 or length > max_k
fn pack_trits_dense(trits: &[i8]) -> Result<u64, Error> {
    let mut word = 0u64;
    let mut base = 1u64;
    for &t in trits {
        if !(-1..=1).contains(&t) { return Err(Error::InvalidTrit); }
        word += ((t + 1) as u64) * base;
        base *= 3;
    }
    Ok(word)
}
```

**Bits → trits** (inverse of dense encoding):
```rust
fn unpack_trits_dense(word: u64, k: usize) -> Result<Vec<i8>, Error> {
    if k > 40 { return Err(Error::TooManyTricks); } // 3⁴⁰ < 2⁶⁴
    let mut trits = Vec::with_capacity(k);
    let mut w = word;
    for _ in 0..k {
        let idx = (w % 3) as i8; // 0,1,2
        trits.push(idx - 1);     // -1,0,+1
        w /= 3;
    }
    if w != 0 { return Err(Error::OutOfRange); }
    Ok(trits)
}
```

**Runtime 2-bit encoding** (for hot-path computation, not storage):
```rust
// Pack: -1→0, 0→1, +1→2 (2 bits per trit, code 3 is INVALID)
fn pack_trits_runtime(trits: &[i8]) -> Result<u64, Error> {
    let mut word = 0u64;
    for (i, &t) in trits.iter().enumerate() {
        if !(-1..=1).contains(&t) { return Err(Error::InvalidTrit); }
        let idx = (t + 1) as u64; // 0,1,2
        word |= idx << (i * 2);
    }
    Ok(word)
}
fn unpack_trits_runtime(word: u64, k: usize) -> Result<Vec<i8>, Error> {
    let mut trits = Vec::with_capacity(k);
    for i in 0..k {
        let idx = (word >> (i * 2)) & 0x03;
        if idx == 3 { return Err(Error::InvalidCode); }
        trits.push((idx as i8) - 1);
    }
    Ok(trits)
}
```

**Float → Baldec** (see §5).
**Baldec → Float**: `w = (s + 5) / 10.0`.

**Integer → balanced radix** (standard algorithm, odd B only):
```rust
fn to_balanced(mut x: i64, B: i64) -> Result<Vec<i64>, Error> {
    if B % 2 == 0 { return Err(Error::EvenBase); }
    let half = (B - 1) / 2;
    let mut res = Vec::new();
    while x != 0 {
        let mut r = x % B;
        if r > half  { r -= B; x += B; }
        else if r < -half { r += B; x -= B; }
        res.push(r);
        x /= B;
    }
    if res.is_empty() { res.push(0); }
    res.reverse();
    Ok(res)
}
```

## 8. INVARIANTS AND PROPERTY TESTS

Checklist for Rust crate `tridak`:

- [ ] Negation involution: `¬(¬x) == x` for all Tridak values
- [ ] Canonical zero uniqueness: `to_balanced(0, B) == [0]` for all odd B
- [ ] Carry round-trip: `pack_trits_dense(unpack_trits_dense(word, k)) == word` for valid `word, k`
- [ ] Full adder exhaustive: all 27 (a,b,c_in) combinations produce valid (d, c_out) in `{-1,0,+1}×{-1,0,+1}`
- [ ] Distributivity: `x ∧ (y ∨ z) == (x ∧ y) ∨ (x ∧ z)` for truth values
- [ ] Packing bijection for `k ∈ {3,5,8,12}` — verify injectivity on valid code space, reject invalid codes
- [ ] Baldec↔float monotonicity & round-trip error ≤ `1/20`
- [ ] No operation ever writes outside `[-ceil(B/2), floor(B/2)]` for given B
- [ ] Element‑wise operations preserve trinary digit set `{-1,0,+1}`
- [ ] Runtime 2-bit encoding rejects code 3; max k for u64 is 32

## 9. PERFORMANCE NOTES

- **SIMD‑friendly**: 4 trits fit in an i16 lane; 8 trits in an i32 lane — pure integer addition and range checks, no multiply‑heavy operations
- **Inner‑loop**: Trit arithmetic uses only signed adds and saturating comparisons; weights are trinary (`-1,0,+1`) so `weight * value` is a single conditional move
- **Packing cost model**: Pack dense once at load, operate in 2-bit runtime trit space, unpack only at substrate edges (e.g., JSON export)

## 10. WHY NOT QUANTUM

Objection: "Qubits exist in 0 and 1 simultaneously → richer state space."

TArchI position:
- Superposition is unnecessary for deterministic intelligence; probabilistic collapse adds latency and verification cost
- Balanced radix `-1,0,+1` gives three orthogonal axes (truth, polarity, epistemic) without physical indeterminacy
- Deterministic operations are verifiable, debuggable, and can be formally proven — essential for safety‑critical inference
- A phone can already execute trinary arithmetic in sub‑nanosecond cycles; quantum control requires cryogenic infrastructure

Trade: sacrifice quantum parallelism for simplicity, verifiability, and deterministic correctness.

---

**See also**: `charter.md`, `nubit.md`, `package-format.md`, `../discussion/discussion.json`

(End of file)
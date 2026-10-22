# `.tarchi` — TArchI Package Format (revised, 2026-10-22 post M0.6 review)

Status: Draft, 2026-10-22
Author: Fola Adam
Project: TArchI — True Archaion Intelligence

> **Note for whoever wires this up:** this document was authored as `docs/package-format.md` in the TArchI repo. It is staged here so it can be copied into place: `cp tarchi-package-format-spec.md <repo>/docs/package-format.md`

---

## A. The package *is* the knowledge

TArchI has no trained weights and no gradient descent. Its knowledge lives in a single artifact: the `.tarchi` package.

```
package ::= header, axioms, rules, facts, entities, index, quarantine, integrity
```

The package is therefore the exact counterpart of an LLM's weight file, with three properties weights can never have:

1. **Verifiable** — hash integrity *and* semantic integrity (lazy per-page proof checks keyed to immutable axiom/rule/snapshot hashes; full eager re-verification is a separate offline audit).
2. **Composable** — base + domain + personal packages merge deterministically via a namespaced resolution function.
3. **Updatable by delta** — correcting one fact ships a patch with atomic manifest, not a new release.

## B. The Astra unit (Revised)

**1 Astra** = a portable, versioned knowledge-and-rules artifact whose capability is measured by a **frozen two-suite capability vector** against a documented 27B-param reference checkpoint:

| Suite | What it measures | Scoring |
|---|---|---|
| **Knowledge recall** | verifiable factual questions, with distractors and injected contradictions | correctness **with citations** |
| **Derivation** | multi-step questions that must be *derived* from axioms and rules | the **derivation path** (ProofDAG), not just the final string |

A package that merely memorises answers scores zero on derivation, so a database cannot pass as an intelligence. **No additive parameter equivalence.** 3.7 Astra does not mean 3.7× capability.

Storage is a **separate, hard constraint**, measured in bytes and in INT4-param-equivalents (INT4 = 0.5 B/param, the deployment-realistic phone baseline):

| Unit | Reference (INT4-equivalent) | TArchI target (falsifiable) | Note |
|---|---|---|---|
| 1 Astra (27B-equiv.) | 13.5 GB | **~1.35 GB** | target; not established |
| 3.7 Astra (100B-equiv.) | 50 GB | **~5 GB** | target; not established |

And the binding operational constraint: **the whole thing must operate within a 2 GB device RAM**, with an **application budget of ≤ 512 MiB** (provisional; adjusted from device measurement), package on flash, demand-paged.

## C. Page layout and residency (Revised)

- **Page size 4 KiB**, `mmap`-able, read-only on the device.
- **Page table offsets/lengths** — stored as fixed little-endian `u64` (not Balradix sub-byte). Sub-byte addressing removed; 5 trits per byte is the dense storage unit, not the page table.
- **The Tridak is the atomic *logical* update unit** — residency is managed per page/block. Logical trit updates may require block recompression.
- **Hot pages are pinned by the reasoning frontier**: the axiom pages plus the pages of the active derivation chain. Cold pages are evicted. Residency is a *reasoning* decision, not LRU. **Pin caps and spill/recompute policies enforced by pager.**
- Engine overhead target **≤ 200 MB**; resident application budget **≤ 512 MiB** (provisional; includes cache, query tables, proof workspace, decoded blocks, runtime headroom).
- **Python is bindings-only and never the runtime.** The core is Rust (arena-allocated, no GC); Python's per-object overhead would blow the budget at this substrate size.
- **Block codec**: Entropy-coded pages with restart points and offset indexes. Decoded block cache capped at 64 MiB. Expansion ratio measured and bounded.

## D. Section table

All eight sections exist in v0.1 **even when empty** — format stability, so v0.2 can ship induced rules without breaking existing readers.

| # | Section | Contents | v0.1 |
|---|---|---|---|
| 0 | **Header** | magic, format version, flags, tridak base, axiom-set hash, package id, creation provenance, license/attribution, section offset table | ✅ |
| 1 | **Axiom** | foundational axioms and rule forms, trinary-encoded | ✅ |
| 2 | **Entity** | canonical entity table: id, type, compressed alias list | ✅ |
| 3 | **Fact** | triples as Balradix-coded (subject, predicate, object) ids + polarity trit + Baldec strength + provenance id | ✅ |
| 4 | **Rule** | supplied/reviewed schemas + lossless factoring of stored instances; mined generalizations as hypotheses with versioned evidence | ✅ (usable) |
| 5 | **Index** | entity→facts, subject→facts, predicate→facts, trinary-encoded offsets | ✅ |
| 6 | **Quarantine** | rejected and contradicted claims — **audit only, never used as proof premises** | ✅ |
| 7 | **Integrity** | per-page Blake3 hash, authenticated manifest/root (excludes self-reference) | ✅ |

**Why Quarantine is a first-class section:** it makes the anti-hallucination mechanism *visible*. You can read what TArchI refused to believe and why. An LLM cannot tell you what it discarded, because it has no discard step.

## E. Two-tier integrity (Revised)

| Tier | Guarantees | Failure behaviour |
|---|---|---|
| **Hash** (per-page Blake3) | the bits did not change | page rejected, load aborts for that page |
| **Semantic** (Rubik lazy verification) | every fact's proof dependencies checked against immutable axiom/rule/snapshot hashes | facts with unchecked deps marked **unavailable**; query rejected if required deps cannot be checked |

Weights can only ever be hash-checked. Semantic re-verification is what a `.tarchi` adds — but it is **lazy and cacheable**, not eager on every page load.

## F. Composition and deltas (Revised)

Packages compose: `base` + `domain` + `personal`.

Merge is **deterministic**: compose a union of **namespaced, immutable assertions** and run **one deterministic resolution function per canonical signed proposition/context**.

- Evidence sets preserved; losing evidence retained in conflict ledger
- Canonical ID namespaces and axiom compatibility pinned per package
- Equal-strength disagreement → retained in conflict ledger, not silent overwrite
- Pairwise merge order independence: resolution function is associative and commutative by design

Deltas are small patch packages declaring `base_package_id` + `base_version`, plus per-section **logical record deltas** (not byte patches into compressed sections). Atomic manifest publication; failed updates retain previous snapshot. Content-addressed blocks or tombstones for removals.

## G. Density budget (Revised)

**Target ≤ 1.5 bytes/fact; ≤ 2.5 acceptable** — **whole-artifact density including every section**, not Fact-section marginal. Sources of density:

| Source | Saving |
|---|---|
| Balradix-coded compact IDs | 12 trits ≈ 20 bits vs a 64-bit hash (dense encoding) |
| Polarity (1 trit) + unsigned strength (3 bits for 0..5) | 4 bits total |
| Provenance | page-local ordinals + shared source dictionaries; escape widths defined |
| Dense base-3 packing | 3→5, 5→8, 8→13, 12→20 bits (up to 1.667 bits/trit realized vs 1.585 theoretical) |
| Per-block entropy coding | over Balradix digit streams with restart points |
| Entity dedup / alias collapse | scoped aliases; unknown/ambiguous outcomes preserved |
| **Schema compression (lossless factoring)** | facts stored as *applications of reviewed rules*, exceptions explicit |

**Schema compression is measured, not assumed.** Rule section in v0.1 contains supplied/reviewed schemas and lossless factoring. Induced rules are hypotheses until independently accepted; never promoted from frequency alone.

## H. `.tarchi` vs an LLM weight file

| Dimension | `.tarchi` package | LLM weight file |
|---|---|---|
| Knowledge form | explicit, cited facts + derivation traces (ProofDAG) | implicit, diffuse weights |
| Storage (100B-equiv.) | ~5 GB target (falsifiable) | ~50 GB (INT4-equiv.) |
| Retrieval | exact lookup + derivation path | probabilistic, no citation |
| Correction | small logical delta → atomic manifest | full retrain → new blob |
| Verification | hash + semantic lazy proof checks | impossible |
| Coverage | high precision, curated domain | excellent recall, noisy |
| False claims | quarantined, never asserted as proof premises | fluent and confident |
| Distribution | composable domain packages | one giant blob |

## I. Header layout (Fixed)

Explicit little-endian serialization; **no `repr(C)` casts on untrusted bytes**.

```rust
pub const MAGIC: [u8; 8] = *b"TARCHI01";

#[derive(Clone, Copy, Debug, PartialEq, Eq)]
#[repr(u8)]
pub enum SectionKind {
    Header      = 0,
    Axiom       = 1,
    Entity      = 2,
    Fact        = 3,
    Rule        = 4,
    Index       = 5,
    Quarantine  = 6,
    Integrity   = 7,
}

#[derive(Clone, Copy, Debug)]
pub struct SectionDesc {
    pub kind: SectionKind,      // 1 byte, checked integer
    pub offset: u64,            // little-endian absolute byte offset from file start
    pub length: u64,            // little-endian byte length
}

#[derive(Clone, Debug)]
pub struct HeaderV01 {
    pub magic: [u8; 8],             // b"TARCHI01"
    pub version: u16,               // 0x0001 (LE)
    pub flags: u16,                 // bit0: signature present, bit1: entropy-coded pages
    pub tridak_base: u8,            // 3
    pub reserved1: u8,              // must be zero
    pub header_len: u16,            // total header bytes including section table (LE)
    pub reserved2: u32,             // must be zero
    pub axiom_set_hash: [u8; 32],   // Blake3 of the Axiom section
    pub package_id: [u8; 20],       // canonical identifier
    pub provenance_len: u16,        // UTF-8 provenance string length (LE)
    pub license_id: u16,            // 0=unspecified, 1=MIT, 2=CC0, 3=proprietary
    pub section_count: u16,         // 8 in v0.1
    pub sections: [SectionDesc; 8], // 8 × 17 bytes = 136 bytes
    // Total header before provenance string: 8+2+2+1+1+2+4+32+20+2+2+2+2+136 = 218 bytes
}
```

Byte offsets (explicit, little-endian):

| Offset | Size | Type | Meaning |
|---|---|---|---|
| 0 | 8 | `[u8; 8]` | magic `TARCHI01` |
| 8 | 2 | `u16` | format version |
| 10 | 2 | `u16` | flags |
| 12 | 1 | `u8` | tridak base |
| 13 | 1 | `u8` | reserved |
| 14 | 2 | `u16` | header length |
| 16 | 4 | `u32` | reserved |
| 20 | 32 | `[u8; 32]` | axiom-set hash (Blake3) |
| 52 | 20 | `[u8; 20]` | package id |
| 72 | 2 | `u16` | provenance length |
| 74 | 2 | `u16` | license id |
| 76 | 2 | `u16` | section count |
| 78 | 2 | `u16` | reserved (alignment) |
| 80 | 136 | `[SectionDesc; 8]` | section descriptor table (8 × 17) |
| 216 | var | `[u8]` | provenance UTF-8 string |

**Parsing rules:** Unknown `SectionKind` values are skipped (treated as length 0). Reserved fields must be zero or rejected. Section offsets/lengths must be within file bounds and non-overlapping.

## J. Worked numbers (Revised)

### J.1 Sample package (a sample, not the full Astra)

1,000,000 facts · 50,000 entities · 12,000 aliases · 300 axioms · 5,000 rules · 40,000 quarantined claims.

| Section | Count | Unit cost | Size |
|---|---|---|---|
| Axiom | 300 | ~1 KB | 300 KB |
| Entity | 50,000 | ~40 B | 2.0 MB |
| Alias | 12,000 | ~20 B | 240 KB |
| Fact | 1,000,000 | ~9 B (measured) | 9 MB |
| Rule | 5,000 | ~30 B | 150 KB |
| Index | ~1.1 M entries | ~3 B | 3.3 MB |
| Quarantine | 40,000 | 2 B | 80 KB |
| Integrity | ~2,500 pages | 32 B/page | 80 KB |
| **Total** | | | **≈ 15.2 MB** |

This is a **sample** package — measured total density is ~15.2 B/fact, not 1.5 B/fact. The earlier 7.6 MB / 1.5 B/fact figure excluded entities, aliases, indexes, and integrity overhead.

### J.2 Full 3.7-Astra package (projection)

~3.3 billion facts at the same measured per-fact density ⇒ **~50 GB**, not 5 GB.

**This is the honest projection.** The 5 GB target remains a falsifiable aspiration pending:
- Measured schema-heavy corpus compression ratio
- Exception set sizes for lossless factoring
- Held-out validation on unseen entities/compositions

Working-set story on a 2 GB RAM device (revised):

| Component | Resident (provisional budget) |
|---|---|
| Package | mapped read-only from flash — **not** resident |
| Hot set (axioms + frontier-pinned derivation pages) | ≤ 256 MiB (cap enforced by pager) |
| Decoded block cache | ≤ 64 MiB |
| Query tables / frontier | ≤ 64 MiB |
| Proof / update workspace | ≤ 64 MiB |
| Engine (Rust, arena, no GC) | ≤ 200 MB |
| Runtime headroom / margin | ≤ 64 MiB |
| **Total app budget** | **≤ 512 MiB** |

## K. Versioning and migration

- `version` in the header: readers **reject an unknown major version**, accept a newer minor.
- Section kind ids are extensible; unknown kinds are skipped by readers.
- Adding a new section kind is **non-breaking**. Changing the layout of an existing kind requires a new major version.
- Delta packages declare `base_package_id` (canonical) and `base_version` (u16); a reader must refuse a delta whose base does not match, and must never apply a delta to a different base silently.
- Version split: major (u8), minor (u8) in header `version` field (was u16 monolithic).

## L. Tooling

```bash
tarchi pack --input corpus/ --base base.tarchi --output pkg.tarchi
tarchi inspect pkg.tarchi                 # summary + per-section sizes
tarchi show pkg.tarchi axiom              # dump a section
tarchi verify pkg.tarchi                  # hashes + Rubik lazy re-verification
tarchi compose base.tarchi domain.tarchi personal.tarchi -o merged.tarchi
tarchi diff old.tarchi new.tarchi -o patch.tarchi
tarchi stats pkg.tarchi                   # density, bytes/fact, working-set projection
tarchi bench pkg.tarchi                   # Astra measurement (reference-LLM harness)
```

All operations are **deterministic**; every output carries hashes and provenance for auditability.

## M. Validity checklist — what `tarchi verify` checks

1. **Per-page Blake3** matches the Integrity section.
2. **Authenticity** — root manifest hash matches expected package ID (or optional trusted signature).
3. **Lazy proof verification** — every queried fact's proof dependencies checked against immutable axiom/rule/snapshot hashes; facts with unchecked deps marked unavailable.
4. **Index consistency** — every referenced entity id exists; every referenced fact id is present.
5. **Alias canonicalization** — each alias resolves to exactly one canonical id within its scope; ambiguous aliases flagged.
6. **Quarantine separation** — no quarantined claim appears in the active Fact section as a proof premise.
7. **Orphan references** — no fact references a non-existent entity.
8. **Header invariants** — magic, version, reserved fields, and the section table's offsets/lengths are within the file and non-overlapping.

Verification failure ⇒ the package loads **degraded**; affected facts are unavailable for inference. It never loads silently.

---

## See also

`charter.md` · `tridak.md` · `nubit.md` · `decisions/ADRs.md` · `PLAN.md` · `../discussion/discussion.json`
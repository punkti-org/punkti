# Punkti 0.6 — normative static publication contract

**Status:** **FROZEN — Punkti v0.6 Layer 3, Founder-approved 2026-10-27.** This freezes the publication contract together with the accepted v0.6 carrier (C/S/MID/R) and XY_Z spatial contract. It does not deploy a product or freeze any Layer-4 application semantics.

**Source:** CEO-OS Issue #21. Supersedes the *experiment* profile in
`experimental/punkti-v0.6-static/docs/SPEC.md` as a normative source. The experiment
remains the implementation/measurement evidence for this document.

**Baseline branch:** `draft/punkti-0.6-static-first` (contains the accepted #19 XY_Z
corrective and the accepted #20 static-first architecture).

**Related:** [PUNKTI_0_6_LAYER3_RESET.md](PUNKTI_0_6_LAYER3_RESET.md) (C/S/MID/R and
validation states — unchanged here), [PUNKTI_0_6_SPATIAL_XY_Z.md](PUNKTI_0_6_SPATIAL_XY_Z.md)
(`h = <xy>_<z>` — unchanged here), [PUNKTI_0_6_STATIC_PUBLICATION.md](PUNKTI_0_6_STATIC_PUBLICATION.md)
(accepted architecture decision), [PUNKTI_0_6_MESH.md](PUNKTI_0_6_MESH.md) (dynamic mesh
spec — optional acceleration only, see §11).

**Revision history**

| Rev | Change |
|---|---|
| 1 | Initial contract. |
| 2 | Fixed two internal inconsistencies found by the implementing agent before any code was written: (a) the XY prefix-interval endpoints were degenerate (§8 step 3) — replaced by the explicit prefix-interval definition in §4.2; (b) the pack-membership rule contradicted the pack grouping rule for leaves with more than 256 records (§6) — replaced by a collective-union rule. No normative rule was weakened; §4.2 and §6 are now implementable as written. |
| 3 | Clarifications after the second review pass: pack magic length corrected from 10 to **11** ASCII bytes (`PUNKTIPACK1`); pack entry order fixed to ascending `sha256`; shard `packs` list order fixed to ascending content address; index tree shape constrained (no empty subtree, no dangling shard, manifest index lists exactly the depth-2 prefixes that occur). |
| 4 | Founder freeze decision: packs confirmed as an **optional standardized accelerator**; `~` confirmed as the reserved unanchored index key; publication frozen together with the v0.6 carrier and XY_Z contracts. |

---

## 0.1 Normative constants

Every number used by this contract appears here exactly once. If a number
appears elsewhere in this document, it is a restatement of a row below and MUST
agree with it.

| Name | Value | Meaning |
|---|---|---|
| `profile` | `punkti/pub/0.6` | the `v` member of root, manifest and shard |
| `key_len` | 12 | length of the comparison key `K(xy)` |
| `index_depth` | 2 | length of the manifest index keys (XY prefixes) |
| `leaf_depth` (`D`) | 10 | length of a leaf prefix; depth of the interior tree |
| `min_symbol` / `max_symbol` | `0` / `z` | smallest / largest geohash symbol used for interval padding |
| `geohash_alphabet` | `0123456789bcdefghjkmnpqrstuvwxyz` | lowercase geohash alphabet, 32 symbols |
| `max_index_entries` | 1024 | 32² distinct depth-2 XY prefixes |
| `unanchored_key` | `~` | reserved manifest index key for `h = ""` |
| `pack_magic` | `PUNKTIPACK1` | pack framing magic, **11 ASCII bytes**, no terminator |
| `pack_max_entries` | 256 | maximum records in one pack |
| `digest_len` | 64 | hex characters of a SHA-256 content address |
| `content_address` | SHA-256 | digest used for every immutable object |
| `manifest_members` | `index`, `pack_count`, `prev`, `record_count`, `seq`, `v` | manifest member names |
| `root_members` | `head`, `seq`, `v` | root pointer member names |
| `shard_members` | `children`, `packs`, `prefix`, `records`, `v` | shard member names |
| `record_entry_members` | `h`, `mid`, `sha256` | index record entry member names |
| `unanchored_address` | `""` | the `h` value of an unanchored atom |
| `full_address_prefix` | `punkti/0.6:` | namespace of a full MID address |

---

## 0. Principle

> Nodes validate and write Punkti. Ordinary HTTP serves a static Punkti publication.
> Other nodes replicate by reading, independently verifying, deduplicating and
> republishing it.

A conforming implementation MUST be able to publish Punkti such that a second
implementation can replicate already-published data over ordinary static HTTP with
**no live publisher daemon**. Correctness MUST NOT depend on any dynamic mesh
mechanism.

---

## 1. Objects

A *publication* is a set of immutable objects plus exactly one mutable pointer.

| Logical object | Mutable | Addressed by |
|---|---|---|
| **root pointer** (`root`) | yes — the only mutable object | position |
| **manifest** | no | SHA-256 of its exact bytes |
| **shard** | no | SHA-256 of its exact bytes |
| **record** | no | SHA-256 of its exact bytes (`R`) |
| **pack** | no | SHA-256 of its exact bytes; **optional standardized accelerator** |

Content address = lowercase hex, 64 characters, SHA-256 over the object's **exact
bytes** (`SHA256(R)`, `SHA256(pack bytes)`, `SHA256(canonical JSON bytes)`).

The **filesystem path layout is not normative**. A conforming implementation MAY
store the same logical objects under any paths it can serve over static HTTP. The
reference implementation in `experimental/punkti-v0.6-publication/` uses one
particular layout; conformance vectors are defined over *logical objects and bytes*,
never over paths.

### 1.1 Canonical JSON

Every JSON-shaped object in this contract is encoded as:

* UTF-8, no BOM;
* object members sorted ascending by Unicode code point of the member name;
* no insignificant whitespace (separators `,` and `:`);
* no trailing newline;
* integers only — no fractional, exponent, `NaN` or `Infinity` numbers;
* a parser MUST reject duplicate member names.

This encoding is what is hashed.

---

## 2. Root pointer (mutable)

Exact bytes of the canonical JSON object:

```json
{"head":"<64 lowercase hex>","seq":<int>,"v":"punkti/pub/0.6"}
```

* `v` MUST equal `punkti/pub/0.6`.
* `seq` is an integer ≥ 1. It is a **publication-local monotone counter** of that
  publication — not an atom ordering, not a clock, not globally comparable.
* `head` is the content address of the current manifest.
* The root pointer MUST contain **no other members**. In particular it carries **no
  timestamp**: the profile MUST be byte-deterministic for a given published state.

The root pointer is the only object a publisher may rewrite in place. It MUST be
replaced atomically (write a temporary file in the same directory, `fsync`, rename
over the target, `fsync` the directory).

---

## 3. Manifest

Content address = SHA-256 of exact canonical-JSON bytes.

```json
{"index":{"<prefix>":"<64 hex>"},"pack_count":<int>,"prev":"<64 hex> | null","record_count":<int>,"seq":<int>,"v":"punkti/pub/0.6"}
```

* `v` MUST equal `punkti/pub/0.6`; `seq` MUST equal the `seq` of the root pointer
  that names this manifest.
* `index` maps a **depth-2 XY prefix** to the content address of a shard. Maximum
  1024 XY entries (geohash alphabet has 32 symbols) plus at most one reserved
  entry `"~"` (§3.1).
* `record_count` is the number of record objects referenced by the publication.
* `pack_count` is the number of pack objects referenced by the publication.
* `prev` is the content address of the previous manifest of this publication, or
  `null` for the first manifest.
* The **first** manifest of a publication has `seq = 1` and `prev = null`;
  every later manifest has `seq = previous seq + 1` and `prev` = the previous
  manifest's content address. A reader MUST reject a manifest in which `seq` and
  the nullness of `prev` disagree.
* No other members.

### 3.1 Unanchored records

Records with `h = ""` are unanchored and MUST remain representable and
replicable. They MUST NOT be mixed into an XY prefix subtree, because the padded
key of an unanchored record would collide with a real XY cell.

* The reserved index key is `"~"`. `"~"` is not a geohash character, so it can never
  collide with an XY prefix.
* The shard named by `"~"` is a **leaf** shard whose `prefix` is `"~"` and which
  lists only records with `h = ""`. It has no children.
* No other index key may be `"~"`, and no XY shard may contain an unanchored
  record.

---

## 4. Shard

Content address = SHA-256 of exact canonical-JSON bytes.

```json
{"children":{"<prefix>":"<64 hex>"},"packs":["<64 hex>"],"prefix":"<prefix>","records":[{"h":"<h>","mid":"<full address>","sha256":"<64 hex>"}],"v":"punkti/pub/0.6"}
```

* Members are exactly `v`, `prefix`, `children`, `records`, `packs`.
* `prefix` is either a geohash prefix (lowercase, characters from
  `0123456789bcdefghjkmnpqrstuvwxyz`) or `"~"`.
* A shard is **interior** when `children` is non-empty; then `records` and `packs`
  MUST be empty. A shard is a **leaf** when `records` is non-empty; then `children`
  MUST be empty.
* Children keys are one character longer than the shard's own prefix and MUST
  start with it. A child's prefix length is at most the leaf depth `D` (10).
* Record entries are exactly `{h, mid, sha256}`:
  * `mid` is the full namespace-qualified address (`punkti/0.6:<64 hex>`);
  * `sha256` is the content address of the record object (SHA-256 of `R`);
  * `h` is the canonical spatial address of that record.
* Record entries MUST be sorted ascending by `sha256` (hex string order);
  `packs` MUST be sorted ascending by content address (hex string order — **not**
  by the group index of §6); `children` member order is fixed by canonical JSON.
* A reader MUST reject a shard whose bytes do not hash to the address that named
  it.

### 4.1 Index geometry (normative)

The XY index is a prefix tree over the **12-character key space**.

* `K(xy) = xy.ljust(12, "0")` — the accepted lower-edge comparison key, defined for
  a geohash `xy` of 1–12 characters. `xy` itself is 4–12 characters in a canonical
  address; a shorter `xy` denotes the cell that contains it, exactly as in the
  accepted spatial profile.
* A record with anchored address `h = <xy>_<z>` is filed under the **leaf prefix**
  `L = K(xy)[:10]` (leaf depth `D = 10`).
* The publication tree is: root → manifest → depth-2 shard → … → depth-10 leaf.
  Interior shard prefixes are the successive prefixes of `L`.
* The manifest index MUST list **exactly** the depth-2 prefixes that occur among
  the publication's leaves (at most 1024) plus at most one `"~"` entry. A depth-2
  prefix that occurs MUST be present; a depth-2 prefix that does not occur MUST
  NOT be present.
* Every interior shard MUST have at least one child; every child key MUST occur
  as the prefix of at least one leaf below it. The tree therefore contains no
  empty subtree and no dangling shard.
* A shard at prefix `p` is only reachable through its parent prefix `p[:-1]`; a
  reader MUST verify the prefix chain while walking.
* The `"~"` leaf is not part of the XY tree and has no parent.

The canonical address `h` never depends on this geometry; the geometry is an
indexing layout only.

### 4.2 Prefix intervals and scope intersection (normative)

Prefix selection is defined over the 12-character key space, not over filesystem
paths and not over decimal numbers.

* The **key interval** of a XY prefix `p` of length `d` (2 ≤ d ≤ 10) is the
  inclusive interval of 12-character keys

  ```text
  I(p) = [ p + "0" * (12 - d) ,  p + "z" * (12 - d) ]
  ```

  i.e. the smallest key that starts with `p` (with the minimum symbol `'0'`) up to
  the largest key that starts with `p` (with the maximum symbol `'z'`). Both
  endpoints are inclusive. `I(p)` is an over-approximation of the keys that can
  actually occur under `p` (the geohash alphabet omits some letters); the
  over-approximation is intentional and safe, because selection is refined later on
  the record's exact `h`.
* A **scope** is the accepted lower-edge scope of the spatial profile:
  `xy_lo` / `xy_hi` (inclusive lower, exclusive upper, over `K`), `z_min` /
  `z_max` (inclusive lower, exclusive upper, over exact decimals), each possibly
  absent. A scope is *unrestricted* when all four members are absent.
* The **XY interval** of a scope is
  `[K(xy_lo), K(xy_hi))` when both bounds are present, `[K(xy_lo), +∞)` or
  `(-∞, K(xy_hi))` when one is absent, and `(-∞, +∞)` when neither is.
* A prefix `p` **intersects** a scope iff `I(p)` overlaps the scope's XY interval.
  With an inclusive prefix interval and an exclusive scope upper bound, overlap is
  exactly:

  ```text
  I(p).low  <  scope.XY.high   (or true when the scope has no upper bound)
  I(p).high >= scope.XY.low    (or true when the scope has no lower bound)
  ```

* The `"~"` prefix is selected **only** by an unrestricted scope: any XY or Z
  constraint excludes unanchored records.
* Selection happens on prefixes only; the Z constraint cannot be applied to a
  prefix, because Z is independent of XY and is not part of any index key. A
  reader therefore applies X/Y filtering before fetching record content, and Z
  filtering after the record's `h` has been read (§8 step 6–7).

---

## 5. Record object

* A record object's bytes are the **exact canonical `R`** of one Punkti 0.6 atom
  as defined by the Layer-3 reset (`R = LP(C) || LP(sig)`), unmodified.
* `R` MUST NOT be re-encoded, re-indented, re-canonicalised or rewritten by a
  publisher or reader. The exact bytes are what is stored and what is hashed.
* The content address of a record object is `SHA256(R)`. Canonical individual
  records therefore remain addressable by `SHA256(R)`.
* The Layer-4 payload `x` MUST NOT be reinterpreted, and the spatial address `h`
  MUST NOT be altered to suit the publication layout.

---

## 6. Pack

A pack is an immutable byte string bundling record objects. Framing is
deterministic:

```text
pack := "PUNKTIPACK1"            (11 ASCII bytes, no terminator)
      || u32be(entry_count)
      || entry_count × ( u32be(len(R)) || R )
```

* `entry_count` ≥ 1; `len(R)` ≥ 1.
* Entries are the exact `R` bytes of the records, in the order defined by the
  membership rule below (ascending `sha256` hex within the pack). A pack carries
  **no other metadata and no per-entry digest**. A reader hashes each extracted
  `R` and compares it with the shard index entry.
* **Membership is normative and deterministic**: take the leaf shard's record
  entries, sort ascending by `sha256` hex, and split them into consecutive
  groups of at most 256 entries. Group *k* is one pack of that leaf. A leaf with
  *n* records therefore has `ceil(n / 256)` packs. The set of packs of a leaf is
  thus determined by the leaf's record set; the shard lists them sorted by
  content address (§4), so the group index is recoverable in any case.
* **Per-pack rule**: every record extracted from a pack MUST be one of the
  records listed by the leaf shard that the pack belongs to (matched by
  `sha256`). A pack MUST NOT contain a foreign record, and a pack MUST NOT
  contain the same record twice.
* **Collective rule**: the union of the records extracted from all of a leaf's
  packs MUST equal exactly the leaf's record set — no record of the leaf may be
  missing from the packs, and no record outside the leaf may appear in them.
* A reader MUST reject a pack whose trailing bytes are not consumed, whose count
  does not match, or that violates the per-pack or collective rule.
* Packs are an **optional accelerator** (`--packs` profile). The canonical record
  objects are published in both profiles and remain the canonical addressable
  form of the data.

---

## 7. Publish ordering

A publisher MUST make every object durable **before** the root pointer can name
it:

1. write + `fsync` new/changed **record objects**;
2. if the publisher emits the optional pack profile, write + `fsync` new/changed **packs**;
3. write + `fsync` new/changed **shards** (leaves and interiors);
4. write + `fsync` the new **manifest**;
5. atomically replace the **root pointer**.

A reader MUST therefore never observe a head that references an object which is
not present.

* **State identity.** The *published state* of a publication is the triple
  `(index, record_count, pack_count)` — i.e. everything a manifest says except
  `seq` and `prev`. A publisher MUST NOT advance `seq` (and therefore MUST NOT
  publish a new manifest, and MUST NOT rewrite the root pointer) when the
  published state is unchanged. Otherwise `seq` advances by exactly 1 and `prev`
  becomes the content address of the previous manifest.
* **Idempotence / interruption.** An interrupted publication leaves the previous
  root pointer in place, naming the previous, fully valid manifest. The previous
  manifest, its shards and its records stay published, so the previous state
  remains fully readable. A subsequent publish recomputes the desired state and
  rewrites whatever is missing; content addressing makes this idempotent.
* **Determinism.** Publishing the same logical state twice produces byte-identical
  objects and therefore byte-identical root pointers. Publication uses no wall
  clock: the manifest carries no creation timestamp, and `seq` advances only when
  the published state changes.

---

## 8. Replication (read side)

A replicator uses **static GET semantics only**. It MUST NOT require any endpoint
other than plain object retrieval. (`HEAD` MAY be used as an optimisation.)

Reader algorithm:

1. Fetch the root pointer. Validate §2 exactly. On failure the publication is
   unavailable for this attempt; the reader MUST NOT silently fall back to
   stale cached state.
2. Fetch the manifest named by `head`. **Verify `SHA256(bytes) == head`** — the
   content address is the only integrity guarantee of the pointer. Verify its
   shape per §3, and that `manifest.seq == root.seq`.
3. For each manifest index key, decide whether it can intersect the requested
   scope, using §4.2:
   * an XY prefix key is selected iff its key interval `I(p)` overlaps the scope's
     XY interval;
   * the `"~"` key is selected only by an unrestricted scope.
4. Fetch the selected shard, verify its content address, validate its shape per
   §4, and descend into `children` only while the child prefix still intersects
   the scope (§4.2). Verify the prefix chain at every level.
5. For every selected leaf, obtain the records:
   * **pack profile** — fetch the leaf's packs (verifying each content address),
     decode them per §6, and enforce the per-pack and collective rules;
   * **index-only profile** — fetch each record object by content address.
6. For every fetched `R`: verify `SHA256(R)` against the index entry, decode and
   validate it with the accepted carrier validator, recompute the MID from `C`,
   and require the recomputed MID and `h` to equal the index entry's `mid` and
   `h`.
   * A record that fails structural decoding yields **no** recomputable MID. Such
     a record is a **per-record rejection** (§9): the reader MUST NOT admit it,
     MUST NOT treat the missing MID as a metadata mismatch of the read, and MUST
     continue with the remaining records of the read.
   * A record that decodes but fails provenance verification DOES have a
     recomputable MID (the signature is excluded from the MID). Its metadata is
     therefore checked as above, and it is then a **per-record rejection**: it MUST
     NOT be admitted, MUST NOT displace or poison the valid record with the same
     MID, and MUST NOT abort the rest of the read.
7. Admission, per record:
   * apply the scope to the record's exact `h` (XY **and** Z — Z is checked here,
     not on the prefix);
   * `VALID` → the reader's valid store, keyed by MID (a duplicate MID is an
     idempotent no-op);
   * `UNVERIFIABLE-alg` (or the reserved `UNVERIFIABLE-key`) → a **separate**
     candidate store keyed by `SHA256(R)`; it MUST NOT occupy, displace or poison
     a valid MID;
   * `INVALID-structural` / `INVALID-provenance` → rejected; never stored.
8. **Staging.** A read MUST verify every in-scope referenced object before
   admitting anything from that read. A read that fails part-way MUST admit
   nothing from it.
9. A replicator MAY filter by Z locally after fetching (Z is not part of any
   index key) but MUST be able to filter by XY prefix **before** fetching record
   content.

Anti-poisoning consequence: two distinct `R` may share one MID (the signature is
excluded from the MID). A forged variant is rejected on its own merits and MUST
NOT invalidate a valid record with the same MID.

---

## 9. Failure behaviour

| Condition | Required reader behaviour | Class |
|---|---|---|
| root pointer missing / unparsable / wrong shape | fail this attempt; admit nothing | retryable |
| **stale root** (`seq` below the last accepted `seq` for that base URL) | fail this attempt; admit nothing | retryable |
| **equivocal root** (`seq` equal to the last accepted `seq` but a different `head`) | fail this attempt; admit nothing | retryable |
| **missing object** (any referenced object not retrievable) | fail this attempt; admit nothing | retryable |
| **corrupt object** (bytes do not hash to the address that named them) | fail this attempt; admit nothing | retryable |
| **publication race** (manifest `seq` ≠ root `seq`) | re-read the root pointer and restart the attempt (bounded retries) | retryable |
| pack violates the per-pack or collective rule of §6 | fail this attempt; admit nothing | retryable |
| record fails carrier validation (structural / provenance) | that record is rejected; the rest of the read continues | per-record |
| unsupported algorithm | that record goes to the candidate store | per-record |

* No failure may produce a partial admission.
* Re-running a failed read is safe and idempotent.
* A failure MUST NOT be reported as a completeness or freshness statement in
  either direction.

---

## 10. No completeness or freshness claim

A successful read proves only that the objects it referenced were present and
verified **at the time of that read**. Punkti 0.6 publications make **no** claim
of global completeness, global freshness, authority or ordering across
publications. Withholding, retention and partitions can hide records
indefinitely; that is undetectable from a single publication.

---

## 11. Peer model: a publication URL is a peer

**A publication URL is sufficient as a replication peer.** The replication peer
identity is the base URL of the static object tree. A replicator needs nothing
else: no handshake, no session, no token, no callback registration, no peer
announcement, no coordinator.

Everything in `PUNKTI_0_6_MESH.md` is **optional acceleration** and is explicitly
**not** part of the normative publication surface:

| Dynamic mesh feature | Status |
|---|---|
| `GET /hello` handshake, capabilities, limits, epoch | optional acceleration |
| `POST /push` | optional acceleration |
| `GET /pull` (Range PULL, spatial or MID ranges) | optional acceleration |
| `POST /sync` reciprocal callbacks | optional acceleration |
| peer sessions / peer-to-peer socket state | optional acceleration |
| notifications / change events | optional acceleration |
| cursors, checkpoints, high-water catch-up | optional acceleration |
| discovery hints (`/hello` peers), peer crawling | optional acceleration |
| coverage advertisements | optional acceleration |
| HTTP byte-range (206) partial fetching | optional acceleration |
| dynamic query / filter APIs beyond a scope filter | optional acceleration |
| CDN-specific layout tuning | optional acceleration |
| transport authentication, TLS, signing of publications | optional accelerator / local policy — MUST NOT be required for reading or verifying atoms |

The normative surface is: **root pointer + manifest + shards + records (+ optional
packs)**, fetched with plain GETs.

---

## 12. What this document does not change

* Atom `C`, `S`, MID and `R` — unchanged (`PUNKTI_0_6_LAYER3_RESET.md`).
* The `h = <xy>_<z>` spatial profile, the XY_Z codec limits, the lower-edge
  comparison model and validation states — unchanged
  (`PUNKTI_0_6_SPATIAL_XY_Z.md`).
* Frozen Punkti 0.5.1 — unchanged.
* Layer-4 payload semantics, identity and reputation — out of scope.

This document authorizes no deployment, no release tag, no protocol freeze and no
merge to `main`.

---

## 13. Conformance vectors

The normative vectors for this contract live in
`experimental/punkti-v0.6-publication/` (`docs/NORMATIVE_VECTORS.md` and
`vectors.json`). They cover, at minimum:

| ID | Area |
|---|---|
| V1 | root → manifest discovery |
| V2 | immutable object digest verification (tamper detection) |
| V3 | pack decoding (framing, membership, rejection) |
| V4 | exact `R` / MID verification |
| V5 | same MID, distinct `R` digests; invalid variant anti-poisoning |
| V6 | XY prefix selection |
| V7 | negative and decimal Z preservation |
| V8 | stale / missing / corrupt object behaviour |
| V9 | two independent implementations converging from static-only origins |

A vector is normative over *logical objects and bytes*. Path layout, HTTP status
codes, port numbers, timestamps and wording are not normative.

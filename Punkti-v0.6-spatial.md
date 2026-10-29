# Punkti 0.6 — restored XY_Z spatial address profile

Status: **FROZEN — Founder-accepted spatial-only Punkti v0.6, 2026-10-29**. The 2026-10-27 freeze was reopened in place for the mandatory `h=XY_Z` correction; following independently passing Python/JS conformance and Founder acceptance, v0.6 was re-frozen in place without a version bump. The `punkti/0.6` namespace and frozen v0.5.1 history remain unchanged. Implementation verification does not claim live network availability, PQ signature support or guaranteed long-term preservation.

This document defines the human-readable spatial model for every Punkti 0.6 record.

Normative inputs for Punkti v0.6 Layer 3 are this document, [Punkti-v0.6.md](Punkti-v0.6.md), and [Punkti-v0.6-publication.md](Punkti-v0.6-publication.md). Dynamic Mesh is optional acceleration, not part of normative conformance.

---

## 1. Scope of the correction

The provisional 0.6 profile interleaved latitude, longitude and altitude into
one 12-character Base32 value. That profile is removed as the normative
spatial profile for 0.6.

The restored model is:

```
h = <xy>_<z>
```

* **XY** — a standard lowercase **2D geohash** over latitude/longitude only.
* **Z** — a separate signed decimal **WGS84 ellipsoidal height** in metres.

XY and Z are independently understandable and independently selectable. A
reader looking at two addresses can see at once when XY is the same and only Z
differs:

```
u3butz7k9q_23
u3butz7k9q_-18
u3butz7k9q_23.025
```

The horizontal and vertical dimensions are never entangled. Choosing a
different horizontal precision does not alter the vertical value; choosing a
different vertical value does not alter the horizontal region.

`h` remains a **signed claim of an address**, not proof of physical presence.
Layer 3 carries and spatially addresses; it does not certify that anything is
actually at that location. `x` remains opaque Layer-4/application payload. No
Layer-4 meaning is introduced by this profile.

### 1.1 What this profile does not change

The frozen **v0.5.1** namespace is untouched. The v0.5.1 atom's `h` remains
the canonical 12-character base32 3D geohash produced by `core/geohash3d.py`,
and the v0.5.1 `p:<spatial>` address form is unchanged. The existing
`core/geohash3d.py` and `pwa/geohash3d.js` therefore keep serving the frozen
v0.5.1 contract and are **not** rewritten by this correction. This profile
governs the 0.6 `punkti/0.6` carrier and any 0.6-draft client encoding.

---

## 2. Canonical grammar of `h`

```
h        := XY "_" Z                ; required canonical spatial address
XY       := <4..12 characters>       ; geohash alphabet only
Z        := <canonical decimal>      ; see 2.3
```

* The separator is exactly one ASCII underscore `_` (U+005F).
* `_` is not a member of the geohash alphabet, so the split is unambiguous.
* The canonical `h` length is **6..32 characters**.
* A canonical `h` contains exactly one `_`.

### 2.1 XY component

* Standard lowercase 2D geohash over latitude and longitude only.
* Alphabet (32 characters, ASCII):

  ```
  0123456789bcdefghjkmnpqrstuvwxyz
  ```

* Length in an address: **4 to 12 characters inclusive**.
* Recommended / default publication precision: **10 characters**.
* Prefix semantics are retained: a shorter XY prefix denotes a coarser
  horizontal region. `u3b` is the coarser region that contains `u3butz7k9q`.

**Canonical encoding.** For latitude `lat ∈ [-90, 90]` and longitude
`lon ∈ [-180, 180]` and precision `p ∈ [4, 12]`, the canonical XY is produced
by the standard geohash bisection, and by nothing else:

```
lat_range = [-90, 90]
lon_range = [-180, 180]
lat_bits = []
lon_bits = []
repeat p * 5 times:
    # longitude bit first, then latitude bit; 5 bits per character; MSB first
    mid = (lon_range[0] + lon_range[1]) / 2
    if lon >= mid: lon_bits.append(1); lon_range[0] = mid
    else:           lon_bits.append(0); lon_range[1] = mid
    mid = (lat_range[0] + lat_range[1]) / 2
    if lat >= mid: lat_bits.append(1); lat_range[0] = mid
    else:           lat_bits.append(0); lat_range[1] = mid
bits = []
for i in 0 .. p*5-1:
    bits.append(lon_bits[i // 2] if i % 2 == 0 else lat_bits[i // 2])
XY = "".join(ALPHABET[value_of(bits[5i : 5i+5])] for i in 0 .. p-1)
```

The midpoint belongs to the **upper** half (`>=`). This tie-break rule is
normative; every implementation must agree on it.

**Canonical decoding.** XY denotes the geohash cell that the standard
bisection selects. Decoding returns the cell bounds (`lat_min`, `lat_max`,
`lon_min`, `lon_max`) and the cell centre. The centre is a derived convenience
value, not a stored fact.

**Canonicalization rule.** The canonical XY for a given `(lat, lon, p)` is the
single string produced by the encoding above. There is no other spelling of
that cell at that precision: the alphabet and the bisection make the mapping
one-to-one.

### 2.2 XY bounds are prefixes

A **subscription XY bound** is a prefix of the geohash alphabet of **1 to 12**
characters. A bound may therefore be shorter than any address XY (for example
`u3b`), which is what makes a prefix meaningful as a coarse region selector.

### 2.3 Z component

Z is a signed decimal number of metres, WGS84 ellipsoidal height. There is
**no artificial altitude range**: no `-500..8500` clamp and no lower or upper
bound on the value itself. The only size bound is the 32-character `h` limit.

**Canonical decimal grammar.** Exactly one spelling exists per value:

```
Z := "-"? ( "0" | [1-9][0-9]* ) ( "." [0-9]+ )?
```

with these additional constraints:

1. **No `+` sign.** A leading `+` is never canonical.
2. **No exponent.** `2.3e1` is never canonical.
3. **No redundant leading zeros in the integer part.** `023` is not canonical;
   `0` and `10` are. `0.05` is canonical — the leading `0` before the point
   is required, not redundant.
4. **No redundant trailing zeros in the fraction.** `23.0` and `23.50` are not
   canonical; `23.025` and `23.5` are.
5. **One canonical zero.** The value zero is spelled exactly `0`. `-0`, `0.0`
   and `0.00` are not canonical.
6. **A fraction, if present, has at least one digit.** `23.` is not canonical.
7. **ASCII digits only.** `٢٣` (Arabic-Indic digits) is not canonical. A
   validator MUST use an explicit ASCII digit class and MUST NOT rely on a
   Unicode-aware "is a digit" predicate.
8. **Maximum length 27 characters** (`32 - 4 - 1`).

Integer metres are the normal and default representation. Decimal precision is
permitted where genuinely required; the historical examples use 1 to 3
decimals.

**Exact numeric comparison.** Z values are compared as exact decimals, never
as binary floating point. A conforming implementation MUST NOT derive a Z
comparison from IEEE-754 doubles. Comparison is of the numeric value, so
`23.0` and `23` would compare equal if both ever appeared — but only the
canonical spelling is ever accepted.

**Canonicalization rule.** The canonical Z for a given value is the unique
string matching the grammar above. An implementation MUST reject a
non-canonical spelling rather than silently rewriting it, so that `h` is
byte-stable and therefore MID-stable.

**Client encoder policy.** When a client encodes a measured height given as a
binary floating-point number of metres, it MUST first round to millimetre
resolution (3 decimal places) and then strip trailing zeros, producing the
canonical spelling. This is a *client policy*, not a grammar restriction: the
grammar itself accepts any number of decimal places.

### 2.4 Spatial-only records

Every v0.6 record MUST have `h = <xy>_<z>`. Empty `h`, missing `h`,
`h: null`, arbitrary or malformed `h` is **INVALID-structural**. Check this
before and independently of algorithm dispatch; an unsupported `alg` MUST NOT
rescue an invalid address. Never assign a default `h`, synthesise `_0`, or
derive a location from the client, network, IP address or device.

There is no unanchored record class in revised v0.6. Every eligible record
participates in XY routing.

### 2.5 Length and validity summary

| Property | Rule |
|---|---|
| `h` length | 6..32 characters |
| separator | exactly one `_` |
| XY length in an address | 4..12 characters |
| XY length in a scope bound | 1..12 characters |
| XY charset | `0123456789bcdefghjkmnpqrstuvwxyz` |
| Z length | 1..27 characters |
| Z charset | ASCII digits, optional leading `-`, optional single `.` |

---

## 3. Canonicalization from coordinates

Given a latitude, a longitude and a height in metres:

```
XY = canonical_geohash(lat, lon, p)      ; p default 10
Z  = canonical_decimal(height_in_metres)  ; see 2.3
h  = XY + "_" + Z
```

The decimal spelling of the height is the shortest exact decimal form that
satisfies the grammar in 2.3. Implementations MUST NOT derive the Z spelling
from a binary float's repr without first establishing the exact decimal value
(see the encoder policy in 2.3).

---

## 4. Mesh subscription semantics

A scope is now:

```json
{"xy_lo": null, "xy_hi": null, "z_min": null, "z_max": null}
```

`h_lo`/`h_hi` are removed and replaced by `xy_lo`/`xy_hi`. The renamed members
make the two independent axes explicit and are a visible change to the
0.6-draft Mesh surface.

* `xy_lo` / `xy_hi` — null, or a geohash-alphabet prefix of **1 to 12**
  characters. They bound the **XY portion only**. `xy_lo` is inclusive,
  `xy_hi` is exclusive.
* `z_min` / `z_max` — null, or a **canonical decimal Z string** (section 2.3).
  They bound the **Z value only**. `z_min` is inclusive, `z_max` is exclusive.

`z_min`/`z_max` are canonical decimal strings rather than JSON numbers so that
half-open Z boundaries are exact. A JSON number cannot represent `23.025`
exactly in binary floating point, so a numeric bound would make the boundary
test inexact. The wire representation of a Z bound therefore uses the same
canonical grammar as the Z inside an address — one decimal grammar, everywhere
Z appears.

### 4.1 XY range comparison

Define the **comparison key** of any XY or XY-bound string:

```
K(xy) = xy padded on the right with the alphabet's minimum character '0'
        to exactly 12 characters
```

For example `K("u3b") = "u3b000000000"` and
`K("u3butz7k9q") = "u3butz7k9q00"`.

An address whose XY is `xy` is inside the XY range iff

```
xy_lo is null  or  K(xy_lo) <= K(xy)
xy_hi is null  or  K(xy) <  K(xy_hi)
```

The range is half-open. Comparing keys rather than raw strings is what makes a
prefix such as `u3b` meaningful: `K` maps every prefix onto the lower edge of
the region it names, so `[u3b, u3c)` selects the whole `u3b` region, and a
coarser address inside `u3b` is selected with the finer addresses it contains.

### 4.1.1 Exact statement of the selection rule

The rule is **lower-edge key containment**, and it is stated here precisely so
that no implementation has to infer it:

> An address is selected when its **lower-edge key** `K(xy)` lies within the
> half-open key interval `[K(xy_lo), K(xy_hi))`. Nothing else is consulted:
> not the size of the region the address names, and not whether that region
> overlaps the requested region.

Two consequences follow, and both are intentional:

* A **coarse** address (a short XY) is selected by a range whenever its
  lower-edge key falls inside the range's key interval — even when the
  address names a region *larger* than the requested range, extending past the
  range's upper edge.
* A coarse address whose lower-edge key falls **outside** the range's key
  interval is **not** selected, even when the region it names *overlaps* the
  requested range.

Worked example, which pins the behaviour:

| Range | Address XY | `K(address)` | Selected | Why |
|---|---|---|---|---|
| `[u3bu, u3bu000001)` | `u3bu` | `u3bu00000000` | **yes** | lower-edge key equals the range's lower edge; the region is larger than the range, which does not matter |
| `[u3butz7k9q, u3butz7k9r)` | `u3bu` | `u3bu00000000` | **no** | the lower-edge key precedes the range, even though the `u3bu` region overlaps the requested region |

This is the same convention as the classic fixed-width geohash range idiom:
every address and every bound is compared as its 12-character lower-edge
expansion, so a prefix bound selects everything under that prefix.

**Bound comparison uses `K` as well.** Because `K` is what the selection rule
compares, the equal-or-reversed-bound rejection in §4.4 must also use `K` and
not raw string comparison. Otherwise a scope such as `[u3b, u3b0)` — whose two
bounds name the same lower edge — would be accepted as a non-empty range while
in fact selecting nothing. Under `K` the two bounds compare equal and the
scope is rejected with HTTP 400, consistent with §4.4.

### 4.2 Z range comparison

```
z_min is null  or  Z >= z_min
z_max is null  or  Z <  z_max
```

Comparisons are exact decimal comparisons (section 2.3).

### 4.3 Unrestricted scope

A scope with all four bounds null selects **all anchored records**. Every
eligible record is in the XY tree; no hidden global bucket exists. Invalid
addresses fail structural validation even when the scope is unrestricted.

### 4.4 Bound validation

* `xy_lo`/`xy_hi` MUST be null or 1..12 geohash-alphabet characters.
* `z_min`/`z_max` MUST be null or a canonical Z string of 1..27 characters.
* Equal or reversed bounds are rejected with **HTTP 400**, never an empty
  page. For XY the comparison uses `K`; for Z it is an exact decimal
  comparison. This preserves the #17 baseline rule.
* Invalid bounds fail at node construction as a structural configuration
  error, and on the wire as HTTP 400.

### 4.5 Multiple scopes

Multiple scopes form a union, as in the #17 baseline. An empty scope (all
four bounds null) selects all anchored records. An empty subscription list
selects nothing.

---

## 5. Static-publication compatibility

The [normative static publication contract](Punkti-v0.6-publication.md) routes
every eligible record through the XY tree. Publication index keys are only
geographical prefixes. The removed key `~` MUST NOT be emitted; a manifest,
index or shard containing it is non-conforming and MUST be rejected clearly.
An illustrative directory projection is:

```
/h/u3b/...        # XY prefix hierarchy
             /z/...  # optional Z bucket/index beneath XY
```

This is a *projection* of the spatial address onto a tree, not a property of
the address. The spatial address itself does not depend on any particular
filesystem layout. Concretely:

* The XY prefix hierarchy is available because XY is a prefix code: the first
  `k` characters of a canonical XY are the canonical XY of the coarser region
  at precision `k`.
* The optional Z bucket is available because Z is a separate decimal value
  that is meaningful independently of the horizontal precision.
* Canonical `R` bytes and `SHA256(R)` are independent of this projection.
* There is no leaf outside the XY tree and no special leaf budget.

---

## 6. Layer-3 boundary statements (unchanged in substance)

* `h` is a spatial **address / claim**. Layer 3 carries, spatially addresses,
  structurally validates and provenance-verifies. It does not assert that an
  atom is physically present at its address.
* Spatial selection needs no payload interpretation.
* `x` stays opaque: a human post, AI assertion, sensor measurement, news
  event or unknown future protocol requires no Layer-3 interpretation.
* A Punkto L4 reply MUST use the same exact `h` as its parent. Punkti L3
  transports opaque refs and does not interpret "reply". No compulsory semantic
  identity channel, names, moderation, communities or object-location claims
  are introduced by this correction.

---

## 7. Interoperability consequences

`h` participates in canonical bytes `C`, signing input `S` and content digest
`MID`. This correction leaves the exact C/S/MID/R framing and all existing
canonical spatial addresses unchanged. MID still identifies immutable content;
`SHA256(R)` still addresses the static raw record. Invalid addresses are rejected,
never repaired or assigned a location. Frozen v0.5.1 history is not mutated.

---

## 8. Required deterministic vectors

Implementations and vectors MUST cover, at minimum:

1. **Same XY, different Z** — two addresses that differ only in Z have
   different `h`, different `C`, different MID and different address.
2. **Negative Z** — `-18` is canonical; `-0` is not.
3. **Decimal Z** — `23.025` is canonical; `23.0` and `23.50` are not.
4. **XY prefix range** — `xy_lo = "u3b"`, `xy_hi = "u3c"` selects `u3bu` and
   `u3butz7k9q` and excludes `u3c0`.
5. **Invalid canonical forms** — `+23`, `2.3e1`, `023`, `23.`, `23.0`, `-0`,
   `0.0`, non-ASCII digits, an XY of 3 or 13 characters in an address, a
   missing separator, a doubled separator, an empty Z, and an `h` longer than
   32 characters.
6. **Exact Z half-open boundaries** — `z_min = "23"`, `z_max = "23.025"`
   includes `23` and `23.02` and excludes `23.025` and `22.999`.
7. **Geohash ground truth** — the encode/decode vectors in section 9.
8. **Z ordering** — a total order over canonical Z values, including across
   zero and across differing fraction lengths.
9. **Lower-edge containment counterexample** — range `[u3bu, u3bu000001)`
   with address XY `u3bu` selects the address, even though the address names
   a region larger than the range. The converse also holds: range
   `[u3butz7k9q, u3butz7k9r)` with address XY `u3bu` does **not** select it.
10. **Bound comparison uses `K`** — a scope with `xy_lo = "u3b"` and
    `xy_hi = "u3b0"` is rejected as equal bounds (HTTP 400), because both
    bounds name the same lower edge.
11. **Spatial-only structure** — empty, missing, null and malformed `h` fail
    before algorithm dispatch, including unsupported algorithms.
12. **Geographical publication only** — reject `~` index keys and `~` shards;
    an unrestricted scope selects all anchored records, including `0000_0`.

---

## 9. Ground truth and worked examples

### 9.1 Geohash vectors

| Latitude | Longitude | p=4 | p=10 | p=12 |
|---|---|---|---|---|
| 55.675398 | 12.561895 | `u3bu` | `u3butz7k9q` | `u3butz7k9q7v` |
| 51.5074 | -0.1278 | `gcpv` | `gcpvj0duq5` | `gcpvj0duq533` |
| 35.6762 | 139.6503 | `xn76` | `xn76cydhzv` | `xn76cydhzven` |
| 40.7128 | -74.0060 | `dr5r` | `dr5regw3pp` | `dr5regw3ppyz` |
| 59.9139 | 10.7522 | `u4xs` | `u4xsudvxb2` | `u4xsudvxb2mq` |
| 0.0 | 0.0 | `s000` | `s000000000` | `s00000000000` |
| -90.0 | -180.0 | `0000` | `0000000000` | `000000000000` |
| 90.0 | 180.0 | `zzzz` | `zzzzzzzzzz` | `zzzzzzzzzzzz` |

The first row is the historical Issue #19 example: `u3butz7k9q` is the
precision-10 geohash of Copenhagen.

### 9.2 Historical Issue #19 examples

These are preserved and must remain valid canonical addresses:

```
u3butz7k9q_23
u3butz7k9q_-18
u3butz7k9q_23.025
```

### 9.3 Canonical and non-canonical Z

| Spelling | Canonical | Reason |
|---|---|---|
| `0` | yes | the one canonical zero |
| `23` | yes | integer metres |
| `-18` | yes | negative integer |
| `23.025` | yes | decimal, no trailing zero |
| `0.5` | yes | leading zero required before the point |
| `-0.5` | yes | negative decimal |
| `0.05` | yes | significant leading zero in the fraction |
| `-0` | no | not the canonical zero |
| `0.0` | no | redundant trailing zero |
| `0.00` | no | redundant trailing zeros |
| `23.0` | no | redundant trailing zero |
| `23.50` | no | redundant trailing zero |
| `023` | no | redundant leading zeros |
| `+23` | no | plus sign |
| `2.3e1` | no | exponent |
| `23.` | no | empty fraction |
| `.5` | no | missing integer part |
| `23 ` | no | whitespace |
| `٢٣` | no | non-ASCII digits |

### 9.4 XY range examples

`K` pads on the right with `0` to 12 characters. Selection is lower-edge key
containment (§4.1.1).

| Range | XY | `K(xy)` | Selected |
|---|---|---|---|
| `[u3b, u3c)` | `u3bu` | `u3bu00000000` | yes |
| `[u3b, u3c)` | `u3b0` | `u3b000000000` | yes |
| `[u3b, u3c)` | `u3butz7k9q` | `u3butz7k9q00` | yes |
| `[u3b, u3c)` | `u3c0` | `u3c000000000` | no |
| `[u3butz7k9q, u3butz7k9r)` | `u3bu` | `u3bu00000000` | no |
| `[u3butz7k9q, u3butz7k9r)` | `u3butz7k9q` | `u3butz7k9q00` | yes |
| `[u3butz7k9q, u3butz7k9r)` | `u3butz7k9r` | `u3butz7k9r00` | no |
| `[u3bu, u3bu000001)` | `u3bu` | `u3bu00000000` | yes |

The last row is the counterexample that pins the rule: the `u3bu` address
names a region far larger than the requested range, yet it is selected
because its lower-edge key is inside the range's key interval.

### 9.5 Exact Z half-open boundary

Scope `z_min = "23"`, `z_max = "23.025"`:

| Z | Selected | Reason |
|---|---|---|
| `23` | yes | equal to the inclusive lower bound |
| `23.02` | yes | strictly inside |
| `23.025` | no | equal to the exclusive upper bound |
| `22.999` | no | below the lower bound |
| `-18` | no | below the lower bound |

---

## 10. Remaining spatial ambiguities

Recorded for Founder resolution. Each is a deliberate, documented choice in
this candidate, not an oversight.

1. **Coarse address versus fine range.** The selection rule is lower-edge key
   containment (§4.1.1): an address is selected when its `K` key falls in the
   range's key interval. A coarse address is therefore selected when its
   lower edge falls inside the interval, even if the region it names is larger
   than the range; and it is not selected when its lower edge falls outside the
   interval, even if its region overlaps the requested region. The
   alternative — region-**overlap** semantics, where a coarse address matches
   any range it overlaps — was rejected because it would make one coarse
   address match two adjacent fine ranges and would make the half-open
   boundary inexact. This is a deliberate, documented choice.
2. **Maximum decimal places for Z.** This candidate bounds Z by length only
   (27 characters), not by a fixed number of decimal places, and therefore
   permits more than the 3 decimals seen in the historical examples. A fixed
   cap (for example 3) would be an alternative; it is deliberately not
   imposed, because the issue text asks for one canonical decimal grammar and
   lists only the spelling rules.
3. **Z bounds are strings, not JSON numbers.** Required for exact half-open
   boundaries (section 4). The cost is a visible change to the 0.6-draft Mesh
   scope surface and to the scope binding hash.
4. **Scope member names.** `h_lo`/`h_hi` became `xy_lo`/`xy_hi` so that the
   two axes are independently visible. Keeping the old names would have been
   a smaller diff but would have kept implying that the bound applies to the
   whole of `h`.
5. **XY precision of a published address.** 4..12 is permitted with 10
   recommended. All canonical 4..12-character XY addresses participate in
   publication routing; leaf precision does not restrict address precision.
6. **`h` length limit.** The 32-character maximum applies to the whole
   address, which transitively limits Z to 27 characters and XY to 12.
7. **Client-side integration.** The #17 baseline has no 0.6-draft atom
   creation path in the PWA, so the restored client-side spatial codec is
   delivered as a standalone, tested module and the frozen v0.5.1 create flow
   is left byte-identical. Rewiring the shipped create flow is deferred to a
   separate, explicitly approved task.

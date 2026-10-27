# Punkti v0.6 — Layer 3 Reset

Status: **FROZEN — Punkti v0.6 Layer 3, Founder-approved 2026-10-27**
Scope: Minimal virtual Layer-3 carrier and provenance boundary.
Compatibility: Frozen v0.5.1 atoms coexist unchanged with the explicitly branded v0.6 namespace.

This document is frozen together with the accepted XY_Z spatial profile and static publication contract. Layer-4 semantics, Punkto UI/product behavior, BRTH, messaging, trust/reputation and optional dynamic acceleration remain outside this freeze.

**Founder decisions incorporated:** (1) Geo is native Layer 3: `h` is spatial addressing, not decorative/application metadata; XY denotes horizontal position, Z physical altitude, and ranges select spatial addresses. Spatial range subscriptions and range-scoped reconciliation are first-class requirements for the subsequent Mesh. (2) `x` remains opaque Layer-4/application payload. (3) BRTH is persistent actor identity at Layer 4, usable by AI instances as well as humans and other actors. Frozen v0.5.1 remains unchanged; PR #111 must not be modified during its frozen review state. This proposal does not start Mesh.

Source basis: `CODEX_L3_BRIEF.md` supplies the authoritative v0.5.1 contract and PR #111 concept inventory. The referenced `punkto_atoms/atoms_core.py`, `punkto_atoms/make_atom.py`, and PR #111 identity-root document are absent from this checkout. Direct source comparison and exact header-style comparison remain review prerequisites; this document does not claim to have performed them.

## Objective & Core question

What is the smallest protocol a node must implement to carry, validate, address, link and synchronize atoms for applications it does not understand?

Punkti is a virtual Layer 3: a carrier for immutable atoms with a native spatial address space. A node MUST be able to operate on spatial addresses and ranges and carry an atom without understanding its application, reference vocabulary, or Layer-4 identity. Unknown algorithms do not license interpreting or discarding the payload. Resource admission limits remain possible, but must be explicit and independent of application meaning.

Dynamic Mesh work is no longer normative for Punkti 0.6 and is retained only as optional acceleration evidence. The successor product tasks include **Punkti Identity — BRTH Layer 4 specification** (identity semantics), **NairobiDot — independent Punkti compatibility proof** (a non-Punkto implementation), and **Punkti Internet Red Team** (hostile network and scale review). Their contracts should follow this boundary rather than silently extending the core.

## Layer 3 vs Layer 4 boundary table

| Concern | Layer 3 (carrier core) | Layer 4 (identity/app semantics) | Notes |
|---|---|---|---|
| Atom envelope/canonical bytes | Parse bounded structure; preserve canonical content | Interpret payload and extensions | Unknown application types remain admissible. |
| Immutable content addressing (MID) | Compute immutable content digest | Assign domain meaning to addressed objects | An address is not a person. |
| Timestamp | Carry a signed integer timestamp | Interpret event time, freshness and chronology | No trusted clock or global ordering is implied. |
| Spatial address (`h`, XY_Z) | Carry, structurally check and operate on native spatial addresses/ranges | Interpret application meaning and physical claims | XY is horizontal position; Z is physical altitude, not application meaning. No proof of presence; legacy anchors stay unchanged. |
| Signature/provenance envelope | Verify supported signature schemes over defined bytes | Decide trust and authority | Verification proves control of a key only. |
| Self-certifying public key | Embed key; bind it into signed content and MID | Associate keys with identities | No external registry prerequisite for v0.6. |
| Algorithm/version identifiers | Dispatch explicitly; preserve unsupported values | Choose application assurance policy | Never guess or downgrade an algorithm. |
| References/linking | Carry content-addressed targets and opaque relations | Interpret reply, parent, identity or authorization relations | Missing targets do not invalidate the carrier. |
| Validation state | Report structure and provenance separately | Evaluate semantic validity | Unknown is not verified. |
| Synchronization semantics | Transfer bytes, deduplicate addresses, reconcile spatially scoped sets | Select application views | Spatial selection needs no payload interpretation; no semantic filtering in the core contract. |
| Recovery | Carry opaque data if supplied | Recovery words, derivation and restoration | Secrets should not be published as atoms. |
| Identity root / Birthstone | Carry and address its bytes | BRTH persistent actor identity anchor and root rules | No native identity object in L3. |
| Authorization events | Carry signed objects and links | Grants, revocation and key succession | L3 does not resolve identity authority. |
| Names | Carry bytes | Naming and conflict resolution | No global namespace authority. |
| Reputation/attestations | Carry signed statements | Assess credibility and reputation | Signature validity is not truth. |

Boundary rule: L3 carries, spatially addresses, structurally validates and provenance-verifies; it NEVER interprets application or identity meaning. `h` is native spatial addressing, not decorative metadata. Spatial ranges select horizontal regions and altitude intervals under the spatial profile; this says where an atom is addressed, not what it means or whether its location claim is true. `x` stays opaque: a human post, AI assertion, sensor measurement, news event, robot state or unknown future protocol requires no L3 interpretation.

## Disposition of PR #111 concepts

| PR #111 concept | Classification | Rationale |
|---|---|---|
| 24-word/256-bit recovery root | MOVE TO L4 IDENTITY | Recovery is a BRTH identity lifecycle convention, not a carrier prerequisite. Preserve the design work there. |
| Deterministic offline reconstruction | MOVE TO L4 IDENTITY | Reconstructing an identity belongs with recovery and derivation rules. L3 only needs the resulting public provenance material. |
| Identity separated from operational signing keys | MOVE TO L4 IDENTITY | This is an identity model whose authorization relationships L3 cannot interpret. |
| Immutable Birthstone identity anchor | MOVE TO L4 IDENTITY | L3 can address the immutable atom; BRTH defines why it anchors an identity. |
| Permanent 256-bit content-derived Identity ID | MOVE TO L4 IDENTITY | An Identity ID is an L4 interpretation of content addressing, not a second native L3 identity. |
| Algorithm-agile key/signature objects | KEEP IN PUNKTI (L3) | Explicit algorithms and embedded public material support self-certifying provenance and opaque carriage of unsupported algorithms. |
| Dual post-quantum root families ML-DSA + SLH-DSA | MOVE TO L4 IDENTITY | The dual-root policy belongs to BRTH. Individual algorithm encodings can later receive independent L3 verifier profiles. |
| Offline root / disposable operational keys | MOVE TO L4 IDENTITY | Key custody and delegation lifecycles are optional identity policy. |
| Forward-only identity authorization/revocation events | MOVE TO L4 IDENTITY | L3 preserves events but does not decide their authority, sequence or effect. |
| Extensible child atoms instead of profile fields | MOVE TO L4 IDENTITY | Profile extension through child atoms is an L4 schema convention; generic opaque links remain available in L3. |
| Explicit canonical-serialization review before 0.6 freeze | KEEP IN PUNKTI (L3) | Independent implementations need identical signing bytes and addresses. A separate freeze review is mandatory before claiming interoperability. |
| No requirement to preserve the 0.5 identity construction | MOVE TO L4 IDENTITY | BRTH need not perpetuate the previous identity design. This does not permit rewriting legacy carrier bytes or MIDs. |
| 2-of-2 root-management policy | MOVE TO L4 IDENTITY | Joint root authorization is identity policy, not universal atom admission policy. |

Proposed future reframing of PR #111, after its frozen review state ends and separate authorization is given: split its concepts into (a) an L3 carrier/provenance delta and (b) the BRTH Layer-4 identity specification. Do not modify or supersede PR #111 during frozen review. Preserve the 24-word, one-QR Birthstone, universal identity/KERI-lite and First Real Identity work in the L4 specification. No listed concept needs removal merely because its correct home is L4.

## Minimal v0.6 Layer-3 core (proposal)

The following is a concrete candidate encoding for review, not a frozen wire format. It deliberately uses opaque application bytes and a small fixed envelope. No field denotes a persistent actor identity.

### Atom envelope

Proposed transport is a UTF-8 JSON object with exactly `v`, `t`, `h`, `x`, `alg`, `pk`, `refs`, `ext`, `sig`. Duplicate keys and extra top-level fields are structural errors; extension data goes in `ext`. JSON whitespace and member order do not determine the MID. Strings contain Unicode scalar values; reject lone surrogates and do not normalize text.

| Field | Proposed representation |
|---|---|
| `v` | Literal string `punkti/0.6`. |
| `t` | JSON integer, Unix milliseconds; exact accepted spelling `0\|[1-9][0-9]*`, maximum 9007199254740991. Reject `-0`, negatives, leading zeros, plus signs, decimal points and exponents. |
| `h` | Empty string for no spatial anchor, otherwise `<xy>_<z>` under [the normative spatial profile candidate](PUNKTI_0_6_SPATIAL_XY_Z.md): 4–12 lowercase 2D geohash characters, canonical decimal WGS84 ellipsoidal metres, at most 32 characters total. |
| `x` | Application payload as a UTF-8 string; no required type prefix or schema. |
| `alg` | Nonempty ASCII algorithm identifier, maximum 128 bytes; proposed known profile `ed25519`. Case-sensitive, no aliases. |
| `pk` | Canonical padded RFC 4648 base64 public-key bytes; nonempty. |
| `refs` | Ordered array of objects with exactly `target` and `rel`, both strings. |
| `ext` | Canonical padded base64 opaque extension bytes, possibly empty. |
| `sig` | Canonical padded base64 signature bytes; nonempty. |

A reference target is `namespace:lowercase-hex-digest`, with proposed namespaces `punkti/0.5.1` and `punkti/0.6`, each using a 64-character SHA-256 digest. Future namespace tokens and nonempty even-length lowercase hex digests are structurally carried without attempting resolution. Namespace tokens use ASCII letters, digits, `.`, `/`, `_`, `-`, with a 128-byte limit. `rel` is an opaque UTF-8 string, including unknown vocabulary; empty is allowed. Reference array order and duplicates are preserved and signed, not sorted or interpreted.

Proposed admission limits are a 1 MiB canonical body, a 2 MiB transport record, at most 256 references, and at most 1024 UTF-8 bytes per relation. Nodes should advertise any lower local storage/transfer limits. Limit rejection is a resource result, not a declaration that an application is invalid; these candidate limits need Mesh/red-team review.

### Canonical bytes and immutable addressing

Define `U32(n)` as four-byte unsigned big-endian length/count, and `LP(b) = U32(length(b)) || b`. Every length is in bytes. `UTF8(s)` encodes the unnormalized scalar string. Base64 fields are decoded before framing; reject malformed padding, nonzero unused bits and noncanonical encodings by decode/re-encode comparison.

JSON timestamp import MUST accept exactly `0|[1-9][0-9]*`, with value <=9007199254740991, and explicitly reject `-0`. Canonical framing uses minimal unsigned decimal with the same bound; this import rule does not change carrier bytes.

The canonical body `C` is the concatenation, in this exact order:

1. `LP(UTF8(v))`.
2. `LP(ASCII(t_decimal))`, minimal unsigned decimal, with zero represented as `0`.
3. `LP(UTF8(h))`, then `LP(UTF8(x))`.
4. `LP(ASCII(alg))`, then `LP(base64_decode(pk))`.
5. `U32(number_of_refs)`; for each reference in array order, `LP(UTF8(target)) || LP(UTF8(rel))`.
6. `LP(base64_decode(ext))`.

Proposed signing input `S = ASCII("PUNKTI-L3-SIGN") || 0x00 || C`.
Proposed `mid = lowercase_hex(SHA256(ASCII("PUNKTI-L3-MID") || 0x00 || C))`.
The full address is `punkti/0.6:` followed by `mid`. The signature is excluded from the MID; the algorithm, public key, payload, references and extensions are included. This explicitly separates signature and address domains and prevents substituting a key without changing the address.

Canonical transfer record `R = LP(C) || LP(base64_decode(sig))`. A receiver retains exact `R` bytes and verifies complete decoding with no trailing data. JSON is an import/export representation; a receiver also retains original imported bytes when offering byte-exact JSON export. It never silently rewrites payloads, extensions or legacy records. Metadata such as validation observations lives outside `R`.

### Signature/provenance envelope and identifiers

For the proposed `ed25519` profile, `pk` is the raw 32-byte public key and `sig` is the raw 64-byte Ed25519 signature over `S`, using standard Ed25519 rather than prehash mode. The verifier profile needs reviewed malformed-key and signature-rejection vectors before freeze. Embedded public material supplies self-certifying provenance: a verified signature proves control of the corresponding signing key, not a name, person, authorization or truth claim.

Nodes MUST NOT substitute an algorithm, infer it from key length, or fall back after verification fails. Unsupported `alg` values remain opaque and replicable with an explicit unverified mark. Future algorithm profiles, including PQ, specify their own key/signature encodings and review requirements. Legacy v0.5.1 remains a separate classic Ed25519 path with its original signing input; it does not gain an embedded key retroactively.

### Validation state

Three tiers remain separate: structural validity is an L3 MUST; provenance verification is performed by L3 when the algorithm is supported; semantic validity never belongs in core.

| State | Meaning and handling |
|---|---|
| `VALID` | Structure and supported provenance verification pass; no semantic endorsement. |
| `UNVERIFIABLE-alg` | Structure passes, algorithm unsupported; preserve and replicate as unverified. |
| `INVALID-structural` | Envelope, framing, canonical encoding or required fields fail; reject from the valid atom store. |
| `INVALID-provenance` | Structure passes but a supported signature fails; reject/quarantine that record, never label it valid. |
| `UNVERIFIABLE-key` | Legacy-only observation: external key material is unavailable; preserve the original legacy record without asserting verified provenance. |
| `OPAQUE-version` | Future carrier version unsupported; preserve transfer bytes without asserting inner structural or provenance validity. |

The three requested primary states cannot honestly represent a forged supported signature; the additional failure state avoids conflating provenance with structure. Missing/empty signatures are structural failures, not unknown algorithms. Validation observations are local and recomputable when verifier support changes; peer claims are not trusted as verification evidence.

### Synchronization semantics

Proposed operations are **PUSH** of immutable records and **Range PULL** over lexicographically sorted full addresses. Range bounds are inclusive start/exclusive end; when both endpoints are present, equal or reversed bounds MUST be rejected (HTTP 400 in Mesh), while either null endpoint is unbounded; pages return records, the next address cursor, and an end marker. No timestamps define reconciliation order. A page size bounds work, and a receiver verifies the supplied address against the record whenever it understands that version.

Mesh will define network framing, discovery, capability negotiation, snapshots and range summaries. The core contract is already precise: nodes MUST preserve transferred record bytes, deduplicate logical content by full namespace-qualified MID, preserve unknown application/reference/algorithm data, and avoid semantic interpretation. Transport record hashes may identify byte variants; they do not replace native MIDs.

Spatial range subscriptions and range-scoped reconciliation are first-class requirements for that subsequent Mesh. Spatial XY and Z ranges follow the restored profile: reject `K(xy_lo) >= K(xy_hi)` with K padded right to 12 characters using `0`, and `z_min >= z_max` by exact decimal comparison of canonical strings when both endpoints are present; null is unbounded. Selection uses lower-edge key containment, not region overlap. Subscription construction MUST fail fast for invalid bounds. A spatial range selects atoms by `h`, independently of their opaque `x`; it is distinct from the lexicographic MID ranges used by Range PULL above. A node can subscribe to such a spatial range and carry unknown Layer-4 atoms within it. Mesh must define subscription and reconciliation mechanics separately; no new wire encoding or Mesh implementation is authorized here.

Because signatures are excluded from MID, a bad signature variant must not poison the address permanently. Check a newly received variant before treating it as an already verified duplicate. Store a bounded set of variants or quarantine candidates separately; retain a verified representative when available. Unknown-alg variants cannot safely be assumed equivalent in provenance.

Set-agnostic convergence means that peers exchanging the same admitted set eventually share that set, independent of arrival order or application meaning. It is conditional on eventual delivery, compatible resource policies and retention. No global completeness, deletion agreement, or convergence under permanent partitions is promised. Repeat range sweeps or use Mesh snapshots so concurrent insertions behind a cursor are not lost.

Minimal illustrative JSON (angle-bracket values are placeholders, not a valid signed test vector):

```json
{
  "v": "punkti/0.6",
  "t": 0,
  "h": "",
  "x": "opaque application payload",
  "alg": "ed25519",
  "pk": "<base64 raw public key>",
  "refs": [],
  "ext": "",
  "sig": "<base64 signature over S>"
}
```

## Native identity question

Punkti L3 needs NO native identity concept beyond self-certifying provenance (embedded key + signature); persistent actor identity is entirely optional Layer 4. The same BRTH/IdentityID/authority mechanism may identify a human, a persistent AI instance, an organization, a sensor, a robot, a physical object or another persistent actor. Punkto may interpret one producer as human and another as an AI instance without Punkti needing to know the difference.

NairobiDot can create keys, sign opaque payloads, compute addresses, carry links and synchronize records without BRTH, Punkto, a human account, a global registry, blockchain or central identity authority. It can introduce its own optional L4 conventions. BRTH verification may enrich an application view but never becomes a prerequisite for generic L3 carriage.

## Compatibility & versioning strategy

The frozen v0.5.1 atom has exactly `t, h, x, fp, sig`. Its signing input is `UTF8(t_decimal + "|" + h + "|" + x + "|" + fp)`. Its MID is `lowercase_hex(SHA256(signing_input))`; `sig` is not part of MID. `t` is Unix milliseconds as an integer, `h` is the canonical 12-character base32 3D geohash from `core/geohash3d`, `x` is a UTF-8 payload, `fp` is the author fingerprint/identity string, and `sig` is a base64 Ed25519 signature over that input. This restates the brief's frozen contract; source-level edge cases require the pending direct source check.

Legacy signature verification needs a key-map outside the carrier contract. The atom does not carry a verifiable public key, so a generic node cannot locally establish provenance from the atom alone. A missing map yields `UNVERIFIABLE-key`, not a claim of valid identity. Do not reinterpret legacy Ed25519 as PQ or retrofit BRTH authority.

Proposed v0.6 uses the explicit `v` brand/version field and the separate qualified-address namespace. One store can index `(namespace, mid)` alongside original record bytes and local validation state. A legacy adapter recognizes the frozen five-field shape without inserting a version into the old atom. Existing legacy MIDs, signatures and bytes remain untouched; qualified names are external metadata. Cross-version references use the qualified target address.

Unsupported future versions travel in an outer opaque transport wrapper carrying declared namespace/address, original bytes and a separately checked SHA-256 transport digest. These are candidate records in an opaque store: the node can verify transfer integrity, but cannot authenticate the declared future MID without knowing that version's addressing rules. Do not let an unverified future address overwrite a verified object. Mesh negotiates this wrapper and limits; byte preservation and replication do not imply unsupported inner validation. Old unversioned formats cannot be guessed from arbitrary malformed objects.

## Clean-room implementability checklist + normative-vector requirements

1. Implement the proposed envelope parser, strict field checks, scalar UTF-8 handling, duplicate-key rejection, base64 validation and explicit size limits. Verify malformed inputs fail predictably without interpreting payloads.
2. Implement `U32`, `LP`, `C`, `S`, SHA-256 MID construction and `R` decoding exactly as specified above. Verify full consumption and round-trip canonical body equality.
3. Use standard-library hashing and base64 only for the baseline carrier; use a standard-library Ed25519 verifier where the chosen runtime supplies one. Where it does not, report `UNVERIFIABLE-alg` rather than write custom cryptography or claim verification. PQ libraries are optional and unsupported PQ stays unverified.
4. Generate/import an ordinary supported signing key and demonstrate embedded-key provenance, then mutate every signed field and demonstrate that the old signature fails. Verify key substitution changes MID.
5. Preserve unknown payloads, relation vocabulary, extension bytes and algorithm identifiers through storage and transfer. Show identical canonical records at both ends.
6. Import frozen v0.5.1 fixtures unchanged, calculate legacy MIDs with the legacy procedure, and demonstrate coexistence with v0.6 qualified addresses. Test missing external keys explicitly.
7. Demonstrate PUSH and paginated Range PULL with duplicates, reordered arrivals, missing link targets, interrupted transfers and inserts behind a cursor. Compare the final admitted address sets.
8. Test a forged signature arriving before a valid variant at the same MID; confirm it cannot suppress the valid record. Test unknown algorithms and future versions without falsely reporting `VALID`.
9. Build NairobiDot using this carrier contract without importing Punkto or BRTH schemas. Demonstrate application-specific interpretation exists only above the carrier.
10. Complete independent vector review and the freeze task before advertising interoperable v0.6 support. Document resource-policy differences and unsupported algorithms explicitly.

The exact candidate byte-level procedure is `JSON parse → strict structural checks → decode binary fields → construct C → construct S → verify signature → compute MID → construct/preserve R`. Hashing may precede signature verification for indexing, but an index entry is not a verified result. Legacy processing follows its unchanged signing-input/MID procedure instead.

A follow-up freeze task must settle canonical encoding and framing, domain-separation literals, version and algorithm spellings, timestamp bounds, geohash profile, reference syntax/order, extension rules, key/signature formats, admission limits, variant handling and future-version transport. It must publish example key/payload fixtures with exact expected UTF-8 bytes, `C`, `S`, `R`, signature, digest and qualified address, plus legacy fixtures. Numeric cryptographic vectors and expected digest/signature values are deliberately left to that task; none are invented here.

Negative vectors must cover Unicode edge cases, duplicate keys, integer spellings, invalid base64, framing overflow/truncation/trailing bytes, invalid keys/signatures, unknown algorithms/versions, reordered references and altered public keys. Two independent implementations should agree on every accepted byte sequence and every rejection before freeze. The candidate procedure is implementable for a prototype now; frozen cross-implementation conformance is still pending.

## Threat analysis

| Threat | L3 guarantee | Accepted residual risk | Mitigation |
|---|---|---|---|
| Unknown atom types | Opaque payload carriage does not require type recognition. | Content may be harmful or meaningless to an application. | Bounded storage; application opt-in parsers and safe rendering outside core. |
| Malformed extensions | Outer base64/framing is checked and signed bytes preserved. | Inner extension syntax may be invalid or malicious. | Never execute or parse opaque extensions in core; L4 validates its own schema. |
| Algorithm downgrade | Algorithm and version are bound to signed content and address; no fallback. | An attacker can publish a new weak/unknown-alg record. | Report exact verification state; L4 chooses assurance thresholds without suppressing generic opaque carriage. |
| Application-semantic leakage into core | Carrier validity does not depend on BRTH, names or payload vocabulary. | Implementations may accidentally introduce application gates. | Clean-room NairobiDot tests and boundary review of admission code. |
| Signature withheld/forged | Missing signatures fail structure; known-alg forgeries fail provenance. | Unsupported algorithms cannot be checked; attackers can flood candidates. | Separate unverified storage/state, bound variants and work; never cache a failed variant as the final MID outcome. |
| Key substitution without registry | Embedded key is signed and content-addressed, preventing same-address substitution absent hash/crypto failure. | Anyone can publish a different key and claim another person's name. | L4 trust/identity checks; no L3 personhood claim. |
| Replay/duplicate floods | MID dedup avoids duplicate logical content. | Bandwidth, verification costs and endless new signed content remain possible. | Size/work budgets, peer rate limits, backpressure and bounded negative caches; no semantic admission requirement. |
| Fork/ordering | Immutable references and signed bytes remain inspectable. | No global order, unique successor or identity revocation consensus. | L4 fork policy or explicitly separate consensus; timestamps are untrusted claims. |
| Stale view | Reconciliation can compare admitted address sets. | Withholding, partitions, retention and selective peers prevent global completeness. | Repeated range sweeps, snapshots, diverse peers and local freshness observations; never equate a complete page with global truth. |
| Opaque future-address spoofing | Transport hash checks exact received bytes. | Unknown-version native MID claims cannot be independently verified. | Separate candidate namespace storage, bounded variants and deferred verification; no overwrite of verified content. |

L3 cannot establish real-world identity, truthful location or time, trustworthiness, globally effective revocation, conflict-free identity history, fair retention, Sybil resistance, or universal availability. Those require explicit L4 policy, additional trust assumptions and sometimes consensus. Self-certifying provenance does not solve them. Mesh and Punkti Internet Red Team should test these limits without expanding the core into an identity authority.

## Guiding principle line

A Punkti L3 node carries atoms it does not understand; provenance is self-certifying, identity is an application.


## Founder-accepted replication architecture (2026-10-27)

Punkti 0.6 adopts **static-first publication as the normative correctness and replication path**.

> Nodes validate/write Punkti. Ordinary HTTP serves a static Punkti publication. Other nodes replicate by reading, independently verifying, deduplicating and republishing it.

A serving peer MAY be only a publication base URL. Replication correctness MUST NOT require a live Punkti daemon, peer session, reciprocal callback, dynamic range-query API, PUSH endpoint or notification channel.

Dynamic PUSH / Range PULL / notifications / sessions MAY remain available as optional performance accelerators, but are not required for conformance.

The accepted publication model is a tiny mutable discovery root pointing to immutable content-addressed manifests/indexes/replication packs and canonical individual R objects. Exact record variants remain distinguishable by SHA256(R); logical atom identity remains MID. Static spatial discovery uses the accepted human-readable `h=<xy>_<z>` model, with XY prefix locality and independent exact-decimal Z filtering.

The first experiment is preserved under `experimental/punkti-v0.6-static/` and demonstrated daemon-free replication through a plain static HTTP server, two-/three-node convergence, hostile/failure handling, and a strong request-count advantage for immutable replication packs.

This section records the accepted architecture direction only. Punkti 0.6 is **not frozen** until the final normative publication specification and deterministic vectors are Founder-approved.

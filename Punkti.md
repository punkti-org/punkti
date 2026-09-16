# Punkti Protocol

**Version:** v0.5.1 (release candidate)  
**Canonical source:** https://github.com/punkti-org/punkti  
**License:** CC0 1.0 (public domain dedication) — implementations free and unrestricted

Punkti is the open protocol. Punkto is one reference implementation. Other implementations are welcome and expected.

---

## Product

> Punkti is a decentralized network of small signed messages anchored to readable 3D spatial addresses, replicated by append-only log gossip.

This document defines three things only: the **atom**, the **storage model**, and the **sync contract**. Everything else is out of scope.

---

## 0. Conventions

- **MUST**, **MUST NOT**, **SHOULD**, **MAY** are used per RFC 2119.
- Strings are Unicode scalar-value sequences encoded as UTF-8 on the wire. Unpaired surrogate code points are invalid.
- Punkti performs **no Unicode normalization**. Different code-point sequences are different values.
- JSON input MUST be valid UTF-8. Duplicate JSON object keys are invalid.
- Core string values are compared case-sensitively by their UTF-8 bytes after JSON decoding. Locale-dependent comparison MUST NOT be used.
- Binary text encoding is canonical Base64url without padding unless stated otherwise.
- NDJSON uses one JSON object per line terminated by LF (`\n`, 0x0A). CR and BOM are not permitted.
- Unless explicitly stated otherwise, protocol validity is independent of the receiver's wall clock.

---

## 1. The Atom

A Punkti atom is a signed text event at a point in time and 3D space.

The core atom has exactly five required fields:

```json
{ "t": <number>, "h": <string>, "x": <string>, "fp": <string>, "sig": <string> }
```

Additional JSON fields are extensions, not part of the Punkti core atom.

### 1.1 Core fields

| Field | Type | Size | Meaning |
|---|---|---:|---|
| `t` | canonical int64 token | — | Unix millisecond timestamp |
| `h` | string | ≤ 32 UTF-8 bytes | Punkti spatial address |
| `x` | string | ≤ 200 Unicode code points | Typed payload |
| `fp` | string | 4–64 ASCII chars | Author/key fingerprint reference |
| `sig` | string | 86 ASCII chars | Canonical unpadded Base64url Ed25519 signature |

Core field values MUST be preserved when an atom is relayed.

A relay:

- MUST preserve the five core values exactly.
- MUST NOT let extension fields affect core validation, `mid`, deduplication, signature verification, spatial matching, or core semantics.
- MAY preserve, drop, transform, or add extension fields according to local policy before appending the atom to its own log.
- MUST NOT imply that extension fields are authenticated by the Punkti core signature.
- MUST ignore any extension field named `mid` for protocol decisions and recompute `mid` from the four signed core values.

A relay MAY re-serialize the JSON object. JSON member order, insignificant JSON whitespace, and JSON string escape choice are not part of atom identity.

### 1.2 Timestamp (`t`)

`t` is a signed author claim of Unix milliseconds. It is not proof that the atom was created or received at that time.

The JSON number token for `t` MUST match:

```text
^(?:0|-?[1-9][0-9]*)$
```

and its mathematical value MUST be in the signed int64 range:

```text
-9223372036854775808 .. 9223372036854775807
```

Therefore:

- no leading `+`,
- no leading zero except the value `0`,
- no `-0`,
- no decimal fraction,
- no exponent notation.

Implementations MUST parse and preserve `t` losslessly. Converting the token through binary floating point before range checking or constructing `t_decimal` is non-conformant.

Structural validity of `t` does not depend on the receiver's current time.

### 1.3 Payload (`x`)

`x` MUST begin with a 4-character uppercase ASCII type code and MAY be followed by one ASCII space (0x20) and a body:

```text
x = TYPE [ " " BODY ]
```

- `TYPE` matches `^[A-Z]{4}$`.
- `BODY` is one or more application-defined Unicode scalar values when present. A trailing separator with no body is invalid; use bare `TYPE` for an empty body.
- `x` MUST contain at most 200 Unicode code points.
- Punkti does not normalize Unicode before signing or hashing.

Examples:

```text
TEXT Hello world
BOOT https://node.example/data
```

### 1.4 Spatial address (`h`)

A Punkti spatial address separates horizontal location from vertical height:

```text
h = xy "_" z
```

Examples:

```text
u3butz7k9q_23
u3butz7k9q_-18
u3butz7k9q_0.5
u3butz7k9q_23.025
```

The underscore is the one and only XY/Z delimiter. It does not occur in the geohash alphabet or in `z`.

#### Horizontal component (`xy`)

`xy` MUST match:

```text
^[0123456789bcdefghjkmnpqrstuvwxyz]{4,12}$
```

This is the standard lowercase 2D geohash alphabet and ordinary latitude/longitude geohash semantics.

The recommended Punkti point profile uses 10 geohash characters, giving approximately metre-scale horizontal addressing.

#### Vertical component (`z`)

`z` is WGS84 **ellipsoidal height** in metres, not mean sea level, geoid height, or a national vertical datum. Implementations receiving another height reference MUST convert it before constructing `h`.

`z` MUST match:

```text
^(?:0|-?(?:0\.[0-9]{0,2}[1-9]|[1-9][0-9]*(?:\.[0-9]{0,2}[1-9])?))$
```

This gives one canonical textual representation:

- integer metres are written without `.0`,
- fractional heights use 1–3 decimal places,
- trailing fractional zeroes are omitted,
- values between `-1` and `1` include the leading zero,
- `+` and negative zero are forbidden.

Examples:

```text
0
23
-18
0.5
-0.5
23.5
23.025
```

Invalid examples include:

```text
023
23.0
23.50
-0
.5
23.
+23
1e3
```

Punkti imposes no physical plausibility range on `z`; local admission policy MAY do so. The complete UTF-8 encoding of `h` MUST NOT exceed 32 bytes.

The recommended Punkti point profile is:

```text
10-character xy + "_" + integer-metre z
```

Implementations MAY derive `xy` and numeric `z` into separate local indexes. Those derived indexes are not part of the atom. Conformance does not require converting `z` to binary floating point; the canonical text is authoritative.

### 1.5 Message ID (`mid`)

Define:

```text
t_decimal = the exact canonical token defined by §1.2

signing_input = UTF8(
    t_decimal + "|" + h + "|" + x + "|" + fp
)
```

The message ID is:

```text
mid = lowercase_hex( SHA256(signing_input) )
```

`mid` is exactly 64 lowercase hexadecimal characters.

Properties:

- `mid` covers all four signed statement values: `t`, `h`, `x`, and `fp`.
- `mid` deliberately does **not** include `sig`; proof is not statement identity.
- `mid` is derived and is not a required atom field.
- `mid` is the only core deduplication key.

The concatenation is unambiguous for construction: `t`, `h`, and `fp` cannot contain `|`; `x` may contain any application-defined Unicode because `signing_input` is constructed, not parsed.

If a node already stores an atom with the same `mid`, a later delivery is a duplicate even if its `sig` differs. The first accepted core representation MUST be retained unchanged and another log entry MUST NOT be created.

For an incoming delivery, structural validation and recomputation of `mid` occur before duplicate handling. If that `mid` is already known, duplicate handling terminates admission for that delivery; the incoming `sig` cannot replace the stored proof and need not be cryptographically evaluated.

See §9.1 for the security consequence of accepting an UNVERIFIED proof before a key binding is known.

### 1.6 Signature

Punkti v0.5.1 uses ordinary Ed25519 signing over `signing_input`:

```text
raw_sig = Ed25519_sign(private_key, signing_input)
sig = Base64url_no_padding(raw_sig)
```

- `raw_sig` is exactly 64 bytes.
- `sig` is exactly 86 ASCII characters.
- `sig` itself is not included in `signing_input` or `mid`.
- `sig` MUST be canonical Base64url without padding: decoding it and re-encoding those 64 bytes as unpadded Base64url MUST reproduce the exact input string.

When a 32-byte Ed25519 public key `A` is bound to `fp`, verification MUST use the following strict Punkti profile.

Let the 64 decoded signature bytes be `R_bytes || S_bytes`, each 32 bytes.

1. `A` and `R_bytes` MUST decode as canonical compressed Edwards25519 points. Re-encoding each decoded point MUST reproduce the original 32 bytes.
2. The decoded points `A` and `R` MUST be non-identity points in the prime-order subgroup.
3. `S` is the little-endian integer represented by `S_bytes` and MUST satisfy `0 <= S < L`, where:

```text
L = 2^252 + 27742317777372353535851937790883648493
```

4. Hash `R_bytes || A_bytes || signing_input` with SHA-512, interpret the 64-byte digest as a little-endian integer, and reduce it modulo `L`:

```text
k = little_endian_integer(SHA512(R_bytes || A_bytes || signing_input)) mod L
```

5. Verification succeeds if and only if the cofactorless equation holds:

```text
[S]B = R + [k]A
```

where `B` is the Ed25519 base point.

An implementation MAY use any library or internal representation, but its accept/reject result MUST match this profile and the normative vectors.

### 1.7 Structural validation

An atom is structurally valid if and only if:

1. The JSON root is an object.
2. JSON object keys are unique.
3. The five core fields `t`, `h`, `x`, `fp`, and `sig` are present with the required JSON types.
4. `t` satisfies §1.2.
5. `h` satisfies §1.4.
6. `x` is a JSON string whose decoded value contains at most 200 Unicode code points.
7. `x[0:4]` matches `^[A-Z]{4}$`.
8. If `x` contains more than 4 code points, code point 5 MUST be ASCII space (0x20) and at least one body code point MUST follow it.
9. `fp` is an ASCII string matching `^[A-Za-z0-9_-]{4,64}$`.
10. `sig` satisfies the canonical encoding requirements in §1.6.

Structural validity does not depend on the receiver's current time or on whether a public key binding is available.

### 1.8 Verification status

A structurally valid atom has one of three local cryptographic states:

- **VERIFIED** — a usable public key binding for `fp` is known and `sig` verifies under §1.6.
- **UNVERIFIED** — no usable public key binding for `fp` is currently known.
- **INVALID** — a usable binding is known and verification under §1.6 fails.

On new admission:

- VERIFIED atoms MAY still be refused by local admission policy.
- General-purpose relays SHOULD accept, store, and relay UNVERIFIED atoms to support offline/local-first operation, but MUST NOT present them as authenticated.
- A relay MAY require VERIFIED admission as local policy.
- INVALID atoms MUST be rejected on every transport. They MUST NOT be appended as new atoms or actively relayed.

If new key-binding information becomes available, a node MAY re-evaluate previously stored UNVERIFIED atoms and update local verification metadata.

If a stored UNVERIFIED atom becomes VERIFIED, its core values, `mid`, log bytes, and log position remain unchanged.

If a stored UNVERIFIED atom becomes INVALID:

- the node MUST stop actively PUSHing it,
- newly generated derived resources SHOULD exclude it,
- already exposed bytes in the current `/data/log.ndjson` generation MUST remain byte-identical,
- the node MAY rebase its logical log to exclude it; a rebase MUST issue a new `log_id` under §2.4.

A later delivery with the same `mid` cannot replace an existing line inside the same log generation. A node that wants a later proof for that statement to become canonical MUST first rebase away the INVALID line.

How a public key becomes bound to `fp`, including authority changes and revocation, is out of scope for v0.5.1.

### 1.9 Admission policy

A relay MAY apply local admission policy to newly submitted atoms, including rate limits, allowed namespaces, required verification, and timestamp or height plausibility checks.

Such policy:

- is not part of intrinsic atom validity,
- MUST NOT redefine `mid`,
- MUST NOT cause an intrinsically valid historical atom to become intrinsically invalid,
- SHOULD NOT reject historical PULL data solely because an atom is old.

### 1.10 Transport independence

The atom is transport-agnostic. HTTPS (§4) is the canonical transport binding.

Any medium capable of carrying the five core values — including WebSocket, WebRTC, BLE, QR, NFC, LoRa, paper, or future media — MAY be used.

---

## 2. Storage

Every node that stores Punkti atoms MUST maintain an append-oriented, content-addressed store keyed by `mid`.

Internal persistence is implementation-defined. A node MAY use flat files, a database, object storage, or another storage engine, provided the externally observable Punkti contract is preserved.

### 2.1 Invariants

1. **Core immutability.** The five core values of an accepted atom MUST NOT be modified in place.
2. **Idempotent by `mid`.** Repeated delivery of the same `mid` MUST NOT create another logical atom or append-log entry.
3. **Durable before ack.** A newly accepted atom MUST be durably handled before a successful PUSH acknowledgement or sync-cursor advance.
4. **Extension independence.** Extension fields are not part of atom identity and MAY vary between relays.
5. **Fixed local line bytes.** Once a serving node appends a serialized atom line to a logical-log generation, that line's bytes are fixed for that generation.
6. **Bounded line size.** The JSON bytes before the terminating LF MUST NOT exceed 4096 bytes. Including the LF, one NDJSON record is at most 4097 bytes.
7. **No coordination.** Applying the same retained valid atom set is idempotent and independent of delivery order.

A relay that preserves or adds extensions MUST keep the serialized record within the line-size bound. It MAY drop extensions to do so, but MUST NOT truncate or alter the five core values.

### 2.2 Serving-node resources

A serving node MUST expose:

```text
POST /data/inbox
GET  /data/log.ndjson
GET  /data/manifest.json
```

A serving node SHOULD additionally expose:

```text
GET /data/geo/<prefix>.ndjson
GET /data/coverage.json
```

These are protocol resources, not mandatory physical file paths.

`/data/log.ndjson` is the canonical logical append-only byte stream for that node.

`manifest.json`, `geo/<prefix>.ndjson`, and `coverage.json` are derived and MUST NOT be treated as more authoritative than the canonical atom store.

### 2.3 Spatial queries

Spatial prefix filtering operates on the horizontal `xy` component of `h`.

Given:

```text
h = u3butz7k9q_23
xy = u3butz7k9q
```

the prefix `u3but` matches because:

```text
xy startsWith("u3but")
```

Comparison is bytewise on the lowercase ASCII geohash string.

In `/data/geo/<prefix>.ndjson`, `<prefix>` MUST match:

```text
^[0123456789bcdefghjkmnpqrstuvwxyz]{1,12}$
```

Every atom returned from a shard MUST have an `xy` beginning with `<prefix>`. Prefix matching never extends into the `_z` component.

Implementations MAY additionally index or filter numeric `z`, but no specific database or spatial index is required by the core protocol.

### 2.4 Logical-log continuity and retention

For one `log_id`, `/data/log.ndjson` is a strictly append-only byte sequence:

- any byte already exposed at an offset MUST remain byte-identical for the lifetime of that `log_id`,
- new bytes MAY only be appended after the previous final byte,
- the exposed length MUST always end immediately after an LF,
- an incomplete internal trailing record MUST NOT be exposed,
- a line's serialization is fixed at append time.

A serving node MUST persist its current `log_id` with the logical log and preserve it across restarts while the exposed byte sequence is unchanged.

If the logical log is reset, replaced, truncated, pruned, rebased, compacted into different bytes, restored from unrelated history, or otherwise changed such that any existing offset could refer to different bytes, the node MUST issue a new `log_id`.

A `log_id` MUST NOT be reused. Once retired by that node, the same value MUST NOT identify any later logical-log generation.

`log_id` MUST match:

```text
^[0-9a-f]{32}$
```

Generating it from 128 random bits is RECOMMENDED.

Serving nodes SHOULD retain the full logical log indefinitely. If retention removes previously exposed records, that operation is a rebase and therefore requires a new `log_id` and client restart from byte 0.

A crash-recovery process MAY discard an incomplete internal tail without changing `log_id` only if those bytes were never exposed as part of `/data/log.ndjson`.

---

## 3. Sync

Replication is gossip over append-only logical logs with two primitive operations:

- **PUSH** — send one atom to a peer.
- **PULL** — read bytes from a peer's canonical logical log.

There is no leader, quorum, consensus, or global ordering. Every node is an equal peer.

### 3.1 Cursor

A per-peer pull cursor is:

```text
(log_id, byte_offset)
```

`log_id` identifies the peer's logical-log generation.

`byte_offset` is the next byte position to request and MUST point to the start of a record or to EOF.

For an existing cursor, the `Punkti-Log-Id` header on the actual `/data/log.ndjson` response is authoritative. A separately fetched manifest is useful for discovery but cannot bind a later Range response.

If a log response reports a different `log_id`, the client MUST discard that response body without applying atoms or advancing the old cursor, replace the cursor with `(new_log_id, 0)`, and restart from byte 0.

If a stored cursor exists but the log response has no `Punkti-Log-Id`, the client MUST NOT apply the response using that offset or advance the cursor.

### 3.2 Invariants

1. **Configured-peer delivery.** A node that accepts a new atom MUST enqueue it for PUSH to each enabled peer and retry according to §6.
2. **Idempotent application.** Receiving the same `mid` multiple times has the same logical result as receiving it once.
3. **Cursor monotonicity within one generation.** For a fixed `log_id`, a cursor MUST only advance.
4. **Durability before cursor advance.** The cursor advances only after every complete record through the new offset has been durably accepted, rejected, or deduplicated.
5. **Record-boundary safety.** A cursor MUST NOT advance past an incomplete record.
6. **Source suppression.** A relay SHOULD NOT immediately PUSH an atom back to the peer from which that delivery was received.

Punkti does not guarantee universal network-wide delivery. Delivery depends on peer configuration, connectivity, retention, verification policy, and local admission policy.

---

## 4. HTTP transport (canonical)

### 4.1 Common HTTP requirements

Media types:

- `/data/inbox` request and successful JSON responses: `application/json`
- `/data/manifest.json` and `/data/coverage.json`: `application/json`
- `/data/log.ndjson` and `/data/geo/<prefix>.ndjson`: `application/x-ndjson`

All Punkti GET resources MUST include:

```text
Access-Control-Allow-Origin: *
```

`/data/log.ndjson` MUST support single open-ended byte ranges of the form `bytes=<offset>-` and SHOULD advertise `Accept-Ranges: bytes`.

Responses from `/data/log.ndjson` MUST additionally include:

```text
Access-Control-Expose-Headers: Punkti-Log-Id, Content-Range
```

Cross-origin POST access is local policy.

### 4.2 POST `/data/inbox`

Request body: one JSON object containing a Punkti atom and optional extension fields.

Maximum HTTP request-body size is 4096 bytes, measured before parsing.

Responses:

- `201 Created` — a new atom was accepted and durably appended.
- `200 OK` — the `mid` was already known; no append occurred.
- `400 Bad Request` — structural validation failed.
- `401 Unauthorized` — the atom is INVALID under an available `fp` binding.
- `403 Forbidden` — structurally valid but refused by local admission policy.
- `413 Payload Too Large` — request exceeded 4096 bytes.
- `429 Too Many Requests` — local rate limit; `Retry-After` REQUIRED.

Successful response:

```json
{"ok":true,"mid":"<64-lowercase-hex>","t":<t>}
```

On `201`, the new atom MUST be durably appended to the logical log before the response is returned.

On `200`, the logical log MUST NOT be modified. If the incoming atom has the same `mid` but a different `sig`, the first accepted core representation remains unchanged.

### 4.3 GET `/data/manifest.json`

Example:

```json
{
  "node":"n1",
  "version":"0.5.1",
  "log_id":"6ca1f58e872f4cb88680cf9b782a0906",
  "count":1234,
  "bytes":456789,
  "last_t":1711961189000,
  "updated":1711961200000
}
```

Requirements:

- `version` is the Punkti protocol version string.
- `log_id` identifies the current logical-log generation and satisfies §2.4.
- `bytes` is the current exposed byte length of `/data/log.ndjson` at the moment the manifest is generated.
- For a fixed `log_id`, `bytes` MUST NOT decrease.
- `count`, `last_t`, and `updated` are advisory.
- `manifest.json` is advisory for synchronization timing; the `Punkti-Log-Id` attached to a log response is authoritative for the bytes in that response.

### 4.4 GET `/data/log.ndjson`

Clients with a cursor request:

```text
Range: bytes=<byte_offset>-
```

Every `200`, `206`, and `416` response MUST include:

```text
Punkti-Log-Id: <log_id>
```

`<log_id>` MUST equal the generation from which that response was produced.

#### Generation check

Before parsing or applying any body bytes, the client MUST compare `Punkti-Log-Id` with its stored cursor generation.

If they differ:

1. discard the body,
2. do not advance the old cursor,
3. set the peer cursor to `(<new_log_id>, 0)`,
4. retry from byte 0.

This check closes the manifest/Range race; a pre-request manifest check is not sufficient.

#### `206 Partial Content`

For a Range request, a conforming server SHOULD return `206 Partial Content`.

`Content-Range` MUST start at the requested `byte_offset`. A mismatching start offset is a peer inconsistency and the client MUST NOT advance the cursor.

The client processes only complete LF-terminated records. If more than 4096 bytes are encountered without an LF, the peer violates §2.1 and the client MUST stop processing that response without advancing past the previous record boundary.

#### `200 OK` in response to Range

HTTP servers may ignore `Range` and return the full resource with `200 OK`.

If the `log_id` matches, the client MUST treat the body as starting at byte 0, apply it idempotently, and set its new offset from the complete bytes processed. For an unchanged `log_id`, the returned complete length MUST NOT be less than the previously stored offset.

#### `416 Range Not Satisfiable`

A `416` response MUST include:

```text
Content-Range: bytes */<complete-length>
```

After the generation check:

- if `<byte_offset> == <complete-length>`, the client is up to date and leaves the cursor unchanged;
- if `<byte_offset> > <complete-length>` under the same `log_id`, the peer violates §2.4 and MUST be treated as inconsistent;
- any other impossible combination MUST NOT cause silent cursor reuse or advancement.

#### Record processing

For each complete NDJSON record:

1. parse JSON,
2. perform structural validation,
3. compute `mid`,
4. evaluate verification status if a binding is available,
5. apply local admission policy only where permitted by §1.9,
6. durably accept, reject, or deduplicate the record,
7. advance the cursor only after that durable decision.

A structurally invalid or locally INVALID record is skipped locally, but a client MAY advance past its complete bytes after the rejection decision is durable; otherwise one bad record could stall the peer forever.

### 4.5 GET `/data/geo/<prefix>.ndjson`

If implemented, returns a derived NDJSON shard satisfying §2.3.

A node MAY return `404` for prefixes it does not serve.

### 4.6 GET `/data/coverage.json`

If implemented, an example is:

```json
{
  "node":"n1",
  "version":"0.5.1",
  "prefix_len":4,
  "updated":1711961189000,
  "prefixes":{
    "u3bu":{"count":30,"last_t":1711961189000},
    "f044":{"count":4,"last_t":1711960000000}
  }
}
```

- `prefix_len` MUST be in `1..12`.
- Every key in `prefixes` MUST contain exactly `prefix_len` characters from the standard geohash alphabet.
- `count` and `last_t` are advisory.
- Coverage metadata is derived and advisory.

---

## 5. Signature verification and key binding

Punkti v0.5.1 defines how to verify Ed25519 once a usable 32-byte public key is bound to `fp`; it deliberately does not define how that binding is established.

For every transport, including PULL:

- no usable binding → UNVERIFIED,
- binding known and strict §1.6 verification succeeds → VERIFIED,
- binding known and strict §1.6 verification fails → INVALID.

Newly received INVALID atoms MUST NOT be appended or actively relayed. Bytes already present in an immutable older log generation MAY remain observable until that generation is rebased; serving those fixed historical bytes is not re-admission of the atom.

When a binding later appears, retroactive status changes follow §1.8 and MUST NOT mutate bytes inside an existing log generation.

Identity derivation, key registration, key revocation, authority succession, recovery, and algorithm agility are deferred to future specifications.

---

## 6. Peer topology

- Each node maintains a peers list.
- A node MUST NOT list itself.
- Full mesh among a small set of serving nodes is recommended but not required.
- Clients MAY use serving nodes as sync peers.
- Client-to-client sync is outside the v0.5.1 core.

On local atom creation, a node MUST enqueue one PUSH per enabled peer.

Push and pull are independent and SHOULD be retried with bounded exponential backoff.

Retry intervals, batch sizes, worker scheduling, and peer discovery are implementation-defined.

---

## 7. Conformance — Punkti v0.5.1-MINIMAL

An implementation claims Punkti v0.5.1-MINIMAL conformance if it:

1. Parses and structurally validates the five core fields per §1.
2. Computes `signing_input` and `mid` exactly as §1.5.
3. Implements the strict Ed25519 profile in §1.6 when a test binding is supplied.
4. Preserves accepted core values.
5. Deduplicates only by recomputed `mid`.
6. Exposes `/data/inbox`, `/data/log.ndjson`, and `/data/manifest.json` per §4.
7. Implements immutable logical-log generations and non-reused `log_id` values.
8. Implements `(log_id, byte_offset)` synchronization and checks `Punkti-Log-Id` on every log response.
9. Implements the `xy_z` spatial grammar in §1.4.
10. Passes the normative vectors in [`VECTORS.md`](./VECTORS.md).

Derived spatial resources, alternative transports, storage engines, indexes, and application semantics are optional.

---

## 8. Scope boundary and compatibility

Out of scope for Punkti v0.5.1:

- identity derivation,
- key registration and revocation,
- authority succession and recovery,
- post-quantum signatures and algorithm agility,
- BRTH semantics,
- encryption,
- trust and reputation,
- AI identity,
- non-HTTP transport specifications,
- UI,
- moderation,
- payments,
- application-specific payload grammars.

Future specifications MUST NOT silently redefine the five-field v0.5.1 atom.

The v0.5.1 atom intentionally has no in-band version or algorithm field. Any future incompatible atom/signature scheme MUST be explicitly versioned rather than reinterpreting v0.5.1 bytes.

### Compatibility with v0.5

v0.5.1 is intentionally **wire-breaking** relative to v0.5.

Notable differences include:

- a full 64-hex content-complete `mid`,
- canonical unpadded Base64url signatures,
- a new spatial grammar using `xy_z`,
- receiver-clock-independent structural validity,
- explicit log generations and response-bound `log_id`.

Because `h` and other statement values are signed, a relay cannot silently migrate a v0.5 atom into v0.5.1. The author must create/sign the v0.5.1 representation.

---

## 9. Security considerations

### 9.1 UNVERIFIED proof race

`mid` identifies the signed statement and deliberately excludes `sig`. This is required so proof encoding, re-signing, and future algorithm changes do not create new statement identities.

The consequence is explicit: if a relay accepts an UNVERIFIED atom before it knows the public key for `fp`, a malicious party may race a different 64-byte `sig` for the same `(t,h,x,fp)` and occupy that `mid` on that relay. The later authentic proof is then a duplicate until the relay rebases away an atom it later determines is INVALID.

Punkti v0.5.1 does **not** claim anti-censorship for unknown `fp` bindings. This is a consequence of deferring durable identity/key binding to a later specification.

Mitigations available in v0.5.1:

- security-sensitive relays MAY require VERIFIED admission,
- general-purpose relays MUST label UNVERIFIED data as unauthenticated,
- an UNVERIFIED atom later proven INVALID may be removed only through a rebase with a new `log_id`, after which a later delivery with the same `mid` may be admitted again.

Do not include `sig` in `mid` to address this: arbitrary signatures would then create unbounded distinct message IDs for the same statement.

### 9.2 Extensions

Extension fields are unauthenticated by the Punkti core signature. Relays SHOULD drop unknown extension fields they do not use, especially when doing so reduces storage or amplification risk.

### 9.3 Time and height claims

`t` and `z` are signed author claims, not trusted measurements. A valid signature proves who signed those values under the available binding, not that the clock, position, or datum conversion was truthful.

### 9.4 Availability

A peer may churn `log_id` values or serve inconsistent HTTP responses to force retry or full resynchronization. The generation rules prevent those behaviours from silently corrupting a correct client's cursor, but Punkti v0.5.1 does not attempt to prevent bandwidth denial of service by a malicious peer.

### 9.5 Confidentiality

Punkti v0.5.1 provides signatures, not encryption. Core atoms are public by design.

---

## 10. Normative interoperability vectors

The byte-exact conformance vectors in [`VECTORS.md`](./VECTORS.md) are normative for v0.5.1.

They cover:

- canonical `t`, `h`, `x`, `fp`, and `sig`,
- exact `signing_input`, signature, and `mid`,
- Unicode/no-normalization behaviour,
- duplicate-key rejection,
- duplicate POST and same-`mid`/different-`sig`,
- strict Ed25519 edge cases,
- extensions,
- log-line bounds,
- `log_id` lifecycle,
- Range response races and resets,
- spatial shards.

Two independent implementations SHOULD produce identical outcomes for every vector before v0.5.1 is frozen.

---

## 11. Change log

| Version | Date | Change |
|---|---|---|
| pre-v0.4 | early iterations | Superseded. Five-field atom shape, outbox concept, and spatial addressing introduced. |
| v0.4 | 2026-04-19 | Core RFC — narrowed to protocol + storage + sync; RFC 2119 language. |
| v0.4.1 | 2026-04-21 | Clarified HTTP readability of serving resources. |
| v0.5 | 2026-04-23 | Core-only protocol draft. |
| v0.5.1 | 2026-09-16 | Breaking hardening release candidate: content-complete `mid`, canonical `xy_z` spatial address, strict timestamp/signature grammar, receiver-clock-independent validity, duplicate-safe append, immutable log generations, response-bound `log_id`, implementation-defined storage, and normative interop vectors. |

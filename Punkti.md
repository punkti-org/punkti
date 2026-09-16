# Punkti Protocol

**Version:** v0.5.1 (draft hardening)  
**Canonical source:** https://github.com/punkti-org/punkti  
**License:** CC0 1.0 (public domain dedication) — implementations free and unrestricted

Punkti is the open protocol. Punkto is one reference implementation. Other implementations are welcome and expected.

---

## Product

> Punkti is a decentralized network of small signed messages anchored to readable 3D spatial addresses, replicated by append-only log gossip.

This document defines three things only: the **atom**, the **storage model**, and the **sync contract**. Everything else is out of scope.

---

## 0. Conventions

- **MUST**, **MUST NOT**, **SHOULD**, **MAY** — per RFC 2119.
- Timestamps are Unix milliseconds, UTC, int64.
- Strings are UTF-8.
- Punkti performs **no Unicode normalization**. Different Unicode code-point sequences are different values.
- Binary in transport is Base64 URL-safe, without padding, unless stated otherwise.
- Line separator for NDJSON is `\n` (0x0A). No CR. No BOM.
- Wire format is NDJSON: one JSON object per line.
- Duplicate JSON object keys are invalid.
- Unless explicitly stated otherwise, protocol validity is independent of the receiver's wall clock.

---

## 1. The Atom

A Punkti atom is a signed text event at a point in time and 3D space.

The core atom has exactly five required fields:

```json
{ "t": <int>, "h": <string>, "x": <string>, "fp": <string>, "sig": <string> }
```

Additional JSON fields are extensions, not part of the Punkti core atom.

### 1.1 Core fields

| Field | Type | Size | Meaning |
|---|---|---:|---|
| `t` | int64 | — | Unix millisecond timestamp |
| `h` | string | ≤ 32 chars | Punkti spatial address |
| `x` | string | ≤ 200 Unicode code points | Typed payload |
| `fp` | string | 4–64 chars | Author/key fingerprint reference |
| `sig` | string | 86 Base64url chars | Ed25519 signature of 64 raw bytes |

Core field values MUST be preserved when an atom is relayed.

A relay:

- MUST understand and relay the five core fields when it accepts the atom.
- MUST NOT let extension fields affect core validation, `mid`, deduplication, signature verification, spatial matching, or core semantics.
- MAY preserve, drop, transform, or add extension fields according to local policy.
- MUST NOT imply that extension fields are authenticated by the Punkti core signature.

A relay MAY re-serialize the JSON object. JSON member order and insignificant JSON whitespace are not part of atom identity.

### 1.2 Payload (`x`)

`x` MUST begin with a 4-character uppercase ASCII type code and MAY be followed by one ASCII space (0x20) and a body:

```text
x = TYPE [ " " BODY ]
```

- `TYPE` matches `^[A-Z]{4}$`.
- `BODY` is application-defined UTF-8.
- `x` MUST contain at most 200 Unicode code points.
- Punkti does not normalize Unicode before signing or hashing.

Examples:

```text
TEXT Hello world
BOOT https://node.example/data
```

### 1.3 Spatial address (`h`)

A Punkti spatial address separates horizontal location from vertical height:

```text
h = xy "." z
```

Example:

```text
u3butz7k9q.23
```

where:

- `xy` is a standard lowercase 2D geohash.
- `z` is signed decimal height in metres relative to the **WGS84 ellipsoid**.

#### Horizontal component (`xy`)

`xy`:

- MUST use the standard geohash alphabet:

```text
0123456789bcdefghjkmnpqrstuvwxyz
```

- MUST contain 4–12 characters.
- SHOULD contain 10 characters for the default Punkti point profile.
- Uses ordinary 2D geohash latitude/longitude semantics.

The recommended 10-character profile gives approximately metre-scale horizontal addressing.

#### Vertical component (`z`)

`z`:

- is WGS84 ellipsoidal height in metres.
- MAY be negative.
- MUST NOT use a leading `+`.
- MUST use canonical decimal form with no unnecessary leading zeros.
- MAY contain 1–3 decimal places.
- MUST NOT encode negative zero.

Examples:

```text
u3butz7k9q.23
u3butz7k9q.-18
u3butz7k9q.23.5
u3butz7k9q.23.025
```

The recommended Punkti point profile is:

```text
10-character xy + integer-metre z
```

This gives roughly metre-scale resolution on all three axes while allowing finer horizontal or vertical precision when required.

The complete `h` string MUST NOT exceed 32 characters.

Implementations MAY derive `xy` and numeric `z` into separate local indexes. Those derived indexes are not part of the atom.

### 1.4 Message ID (`mid`)

Define the exact signing input:

```text
signing_input = UTF8(
    t_decimal + "|" + h + "|" + x + "|" + fp
)
```

where `t_decimal` is the canonical decimal representation of `t` with no leading zeros.

The message ID is:

```text
mid = lowercase_hex( SHA256(signing_input) )
```

`mid` is therefore exactly 64 lowercase hexadecimal characters.

Properties:

- `mid` covers all four signed core values.
- `mid` does not include `sig`.
- `mid` is not transmitted as a required atom field; it is derived.
- `mid` is the only core deduplication key.

If a node already stores an atom with the same `mid`, the new delivery is a duplicate and MUST NOT create another log entry.

### 1.5 Signature

```text
sig = Base64url_no_padding(
    Ed25519_sign(
        private_key,
        signing_input
    )
)
```

- The separator is a single `|` (0x7C).
- No whitespace or newline is inserted into `signing_input`.
- `sig` is the URL-safe, unpadded Base64 representation of exactly 64 signature bytes.
- `sig` itself is not included in `signing_input` or `mid`.

### 1.6 Structural validation

An atom is structurally valid if and only if:

1. The JSON root is an object.
2. JSON object keys are unique.
3. All five core fields are present and non-empty.
4. `t` is a JSON integer representable as int64.
5. `h` satisfies §1.3.
6. `x` is valid UTF-8 and contains at most 200 Unicode code points.
7. `x[0:4]` matches `^[A-Z]{4}$`.
8. If `x` contains more than 4 code points, code point 5 MUST be ASCII space (0x20).
9. `fp` matches `^[A-Za-z0-9_]{4,64}$`.
10. `sig` is valid unpadded Base64url and decodes to exactly 64 bytes.

Structural validity does not depend on the receiver's current time.

A timestamp is a signed statement made by the author. It is not proof that the atom was created or received at that time.

### 1.7 Verification status

A structurally valid atom has one of three local cryptographic states:

- **VERIFIED** — a public key binding for `fp` is known and `sig` verifies.
- **UNVERIFIED** — no usable public key binding for `fp` is currently known.
- **INVALID** — a public key binding is known and `sig` does not verify.

On new admission:

- VERIFIED atoms MAY be accepted.
- UNVERIFIED atoms MAY be accepted, stored, and relayed.
- INVALID atoms MUST be rejected as valid Punkti atoms.

How a public key becomes bound to `fp` is out of scope for v0.5.1.

If new key-binding information later becomes available, a node MAY re-evaluate previously stored UNVERIFIED atoms and update local verification metadata.

Such re-evaluation MUST NOT:

- change the atom's five core field values,
- change its `mid`,
- change its existing append-log position, or
- rewrite previous synchronization history.

Authority changes and revocation semantics are forward-looking concerns and are deferred to a future RFC.

### 1.8 Admission policy

A relay MAY apply local admission policy to newly submitted atoms, including rate limits and timestamp plausibility checks.

Such policy:

- is not part of intrinsic atom validity,
- MUST NOT redefine the atom's `mid`,
- SHOULD NOT be applied to historical PULL data solely because an atom is old.

A serving node MAY, for example, reject a newly POSTed atom whose timestamp is implausibly far in the future.

### 1.9 Transport independence

The atom is transport-agnostic. HTTPS (§4) is the canonical transport.

Any medium capable of carrying the five core field values — including WebSocket, WebRTC, BLE, QR, NFC, LoRa, paper, or future media — MAY be used.

---

## 2. Storage

Every node that stores Punkti atoms MUST maintain an append-oriented, content-addressed store keyed by `mid`.

Internal persistence is implementation-defined.

A node MAY use flat files, a database, object storage, memory, or another storage system, provided the externally observable Punkti contract is preserved.

### 2.1 Invariants

1. **Core immutability.** The five core values of an accepted atom MUST NOT be modified.
2. **Idempotent by `mid`.** Repeated delivery of the same `mid` MUST NOT create another logical atom or append-log entry.
3. **Durable before ack.** A new accepted atom MUST be durably stored before a successful PUSH acknowledgement or sync-cursor advance.
4. **Extension independence.** Extension fields are not part of atom identity and MAY vary between relays.
5. **No coordination.** Applying the same retained valid atom set is idempotent and converges independent of delivery order.

### 2.2 Serving-node resources

A serving node MUST expose these logical resources:

```text
/data/log.ndjson
/data/manifest.json
/data/geo/<prefix>.ndjson
/data/coverage.json
```

These are protocol resources, not mandatory physical file paths.

`/data/log.ndjson` is the canonical logical append-only stream.

`manifest.json`, `geo/<prefix>.ndjson`, and `coverage.json` are derived and MUST NOT be treated as more authoritative than the canonical atom store.

### 2.3 Spatial queries

Spatial prefix filtering operates on the horizontal `xy` component of `h`.

Given:

```text
h = u3butz7k9q.23
xy = u3butz7k9q
```

a prefix query for:

```text
u3but
```

matches the atom because:

```text
xy startsWith("u3but")
```

Comparison is bytewise on the lowercase ASCII geohash string.

Implementations MAY additionally index or filter numeric `z`, but no specific database or spatial index is required by the core protocol.

Serving nodes SHOULD expose derived spatial shard resources at:

```text
/data/geo/<prefix>.ndjson
```

Every atom returned from a shard MUST have an `xy` component beginning with `<prefix>`.

### 2.4 Retention and logical-log continuity

Serving nodes SHOULD retain the full logical log indefinitely.

Physical segmentation, rotation, database compaction, and storage layout are implementation details.

A serving node MUST maintain a stable `log_id` for as long as previously issued byte offsets into `/data/log.ndjson` remain meaningful.

If the logical log is reset, replaced, truncated, rebased, restored from unrelated history, or otherwise changed such that existing byte offsets may refer to different bytes, the node MUST change `log_id`.

A node MUST NOT shrink or rewrite the exposed logical log while retaining the same `log_id`.

---

## 3. Sync

Replication is gossip over append-only logical logs with two primitive operations:

- **PUSH** — send one atom to a peer.
- **PULL** — read new bytes from a peer's append-only logical log.

There is no leader, quorum, or consensus. Every node is an equal peer.

### 3.1 Cursor

A per-peer pull cursor is:

```text
(log_id, byte_offset)
```

`log_id` identifies the current logical log generation.

`byte_offset` is the next byte position to request.

Before using a stored byte offset, the client MUST know the peer's current `log_id`.

If the peer's `log_id` differs from the stored cursor's `log_id`, the old byte offset is meaningless and MUST be discarded.

The client then begins from byte offset 0 of the new logical log.

### 3.2 Invariants

1. **Configured-peer delivery.** A node that accepts a new atom MUST enqueue it for PUSH to each enabled peer and retry according to §6.
2. **Idempotent application.** Receiving the same atom multiple times has the same logical result as receiving it once.
3. **Cursor monotonicity within one log generation.** For a fixed `log_id`, a cursor MUST only advance.
4. **Durability before cursor advance.** A cursor advances only after all accepted complete atoms through that offset are durably handled.
5. **Partial-line safety.** A pull MUST NOT advance past an incomplete NDJSON line. Only complete LF-terminated lines are committed.
6. **Source suppression.** A relay SHOULD NOT immediately PUSH an atom back to the peer from which that delivery was received.

Punkti does not guarantee universal network-wide delivery. Delivery depends on peer configuration, connectivity, retention, and local admission policy.

---

## 4. HTTP transport (canonical)

Every serving node MUST expose:

| Method | Path | Purpose |
|---|---|---|
| POST | `/data/inbox` | Push one atom |
| GET | `/data/log.ndjson` | Pull canonical logical log |
| GET | `/data/manifest.json` | Log generation and advisory status |
| GET | `/data/geo/<prefix>.ndjson` | Derived horizontal spatial shard |
| GET | `/data/coverage.json` | Derived shard coverage metadata |

GET resources SHOULD serve with:

```text
Access-Control-Allow-Origin: *
```

A node MAY expose cross-origin POST access according to local policy.

### 4.1 POST `/data/inbox`

Request body: one JSON object containing a Punkti atom and optional extension fields.

Maximum HTTP request-body size: 4 KiB, measured before parsing.

Responses:

- `201 Created` — a new atom was accepted and durably stored.
- `200 OK` — the same `mid` was already known.
- `400 Bad Request` — structurally invalid atom.
- `401 Unauthorized` — signature is known to be invalid under an available `fp` binding.
- `413 Payload Too Large` — request exceeded 4 KiB.
- `429 Too Many Requests` — local rate limit; `Retry-After` REQUIRED.

Successful response:

```json
{"ok":true,"mid":"<64-lowercase-hex>","t":<t>}
```

On `201`, the new atom MUST be durably appended to the logical log before the response is returned.

On `200`, the logical log MUST NOT be modified.

### 4.2 GET `/data/manifest.json`

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

`log_id`:

- is an opaque identifier for one logical-log generation,
- MUST remain stable while existing byte offsets remain meaningful,
- MUST change when the logical log is reset or rebased.

A 128-bit random value encoded as 32 lowercase hexadecimal characters is RECOMMENDED.

`bytes` is the current byte length of `/data/log.ndjson`.

For a fixed `log_id`, `bytes` MUST NOT decrease.

`manifest.json` is advisory for counts, liveness, and timestamps, but `log_id` is authoritative for cursor continuity.

### 4.3 GET `/data/log.ndjson` with `Range`

Clients use:

```text
Range: bytes=<byte_offset>-
```

Responses:

- `206 Partial Content` — tail returned.
- `200 OK` — full log returned when no Range was requested.
- `416 Range Not Satisfiable` — no byte exists at the requested offset.

Before interpreting a `416` as "up to date", the client MUST confirm that the current `manifest.json` has the same `log_id` as its stored cursor.

If `log_id` changed, the client MUST discard the old offset and restart from byte 0.

If `log_id` is unchanged and `manifest.bytes` is less than the stored cursor offset, the serving node is violating §2.4 and the client SHOULD treat the peer as inconsistent rather than silently reusing the cursor.

Client processing:

```text
buf = http.body
last_newline = buf.rfind("\n")
if last_newline < 0:
    return

complete = buf[:last_newline + 1]

for line in complete.split("\n"):
    if line is empty:
        continue

    atom = json.parse(line)

    if atom is structurally invalid:
        skip

    handle atom idempotently by mid

advance byte_offset by len(complete) only after durable handling
```

### 4.4 GET `/data/geo/<prefix>.ndjson`

Returns a derived NDJSON shard.

Each returned atom MUST have:

```text
xy startsWith(prefix)
```

A node MAY return `404` for prefixes it does not serve.

### 4.5 GET `/data/coverage.json`

Example:

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

- `prefix_len` declares the advertised horizontal geohash prefix length.
- `count` and `last_t` describe each shard.
- Coverage metadata is derived and advisory.

---

## 5. Signature verification

Punkti v0.5.1 does not define identity derivation or a key-registration system.

For a structurally valid atom:

- if no usable public key binding for `fp` is known, the atom is UNVERIFIED;
- if a binding is known and the Ed25519 signature verifies, the atom is VERIFIED;
- if a binding is known and verification fails, the atom is INVALID.

Serving nodes MAY accept, store, and relay UNVERIFIED atoms.

Serving nodes MUST reject newly received INVALID atoms as valid Punkti atoms.

When new key-binding information becomes available, nodes MAY re-evaluate stored UNVERIFIED atoms and update local verification metadata.

Retroactive re-evaluation does not rewrite the append-only log or alter previously synchronized bytes.

Identity derivation, key registration, key revocation, authority succession, and algorithm agility are deferred to future specifications.

---

## 6. Peer topology

- Each node maintains a peers list.
- A node MUST NOT list itself.
- Full mesh among a small set of serving nodes is recommended but not required.
- Clients MAY use serving nodes as sync peers.
- Client-to-client sync is outside the v0.5.1 core.

On local atom creation, a node MUST enqueue one PUSH per enabled peer.

Push and pull are independent and SHOULD be retried with bounded exponential backoff.

Retry intervals, batch sizes, worker scheduling, and peer-discovery mechanisms are implementation-defined.

---

## 7. Conformance — Punkti v0.5.1-MINIMAL

An implementation claims Punkti v0.5.1-MINIMAL conformance if it:

1. Parses and structurally validates the five core atom fields per §1.
2. Computes `mid` exactly as §1.4.
3. Preserves the five core field values during relay.
4. Deduplicates by `mid`.
5. Exposes `/data/inbox`, `/data/log.ndjson`, and `/data/manifest.json` per §4.
6. Implements `(log_id, byte_offset)` cursor continuity.
7. Supports HTTP Range pulls with partial-line safety.
8. Does not append a duplicate atom on `200 OK`.
9. Implements the Punkti spatial-address grammar in §1.3.

Implementations MAY support additional transports, extension fields, indexes, storage engines, or application semantics.

---

## 8. Scope boundary

Out of scope for Punkti v0.5.1:

- identity derivation,
- key registration and revocation,
- authority succession,
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

---

## 9. Normative interoperability vectors

Before v0.5.1 is frozen, the project MUST publish byte-exact vectors covering at least:

1. `signing_input` construction.
2. Ed25519 signature encoding.
3. Full 64-hex `mid`.
4. Same `(t,h,fp)` with different `x` producing different `mid`s.
5. Valid and invalid Punkti spatial addresses.
6. Negative, integer, and fractional WGS84 heights.
7. Unicode payloads with no normalization.
8. Duplicate JSON-key rejection.
9. Duplicate POST: `201` then `200` with one log entry.
10. Partial-line Range pull.
11. `log_id` change causing cursor reset.
12. Unknown-extension fields being ignored by core semantics.

The exact test vectors are a release requirement, not an application feature.

---

## 10. Change log

| Version | Date | Change |
|---|---|---|
| pre-v0.4 | early iterations | Superseded. 5-field atom shape, outbox concept, and spatial addressing introduced. |
| v0.4 | 2026-04-19 | Core RFC — narrowed to protocol + storage + sync; RFC 2119 language. |
| v0.4.1 | 2026-04-21 | Clarified HTTP readability of serving resources. |
| v0.5 | 2026-04-23 | Core-only protocol draft. |
| v0.5.1 | 2026-09-16 | Hardening draft: content-complete `mid`, persistent validity, split XY/Z spatial address, duplicate-safe append, log generations, storage/wire separation, parser clarifications. |

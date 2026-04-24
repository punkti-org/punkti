# Punkti Protocol

**Version:** v0.5 (core-only)  
**Canonical source:** https://github.com/Fisker1111/Punkti  
**License:** CC0 1.0 (public domain dedication) — implementations free and unrestricted

Punkti is the open protocol. Punkto is one reference implementation. Other implementations are welcome and expected.

---

## Product

> Punkti is a decentralized network of 200-character signed messages anchored to 3D geohashes, replicated by append-only log gossip.

This document defines three things only: the **atom**, the **storage model**, and the **sync contract**. Everything else is out of scope.

---

## 0. Conventions

- **MUST**, **MUST NOT**, **SHOULD**, **MAY** — per RFC 2119.
- Timestamps are unix milliseconds, UTC, int64.
- Strings are UTF-8.
- Binary in transport is Base64 (URL-safe, no padding) unless stated.
- Line separator is `\n` (0x0A). No CR. No BOM.
- Wire format is **NDJSON**: one compact JSON object per line.

---

## 1. The Atom

An atom is a signed text event at a point in time and 3D space. It is exactly one NDJSON line with five required fields:

```
{ "t": <int>, "h": <string>, "x": <string>, "fp": <string>, "sig": <string> }
```

### 1.1 Fields

| Field | Type   | Size       | Meaning |
|-------|--------|------------|---------|
| `t`   | int64  | 13 digits  | Unix ms timestamp (when) |
| `h`   | string | 4–12 chars | 3D geohash, Base32, interleaved lat+lon+alt (where) |
| `x`   | string | ≤ 200 chars | Payload: 4-char type code, optional space, body (what) |
| `fp`  | string | 4–64 chars | Author fingerprint (who) |
| `sig` | string | Base64     | Ed25519 signature of 64 raw bytes (proof) |

Any other field is reserved. v1 implementations MUST ignore unknown fields for transport decisions but MUST preserve them byte-for-byte when relaying or storing.

### 1.2 Payload (`x`)

Payload MUST begin with a 4-character uppercase ASCII type code, optionally followed by a single space and a body:

```
x = TYPE [" "] BODY
```

- `TYPE` matches `^[A-Z]{4}$`.
- `BODY` is application-defined UTF-8.
- The optional space is part of `x` and therefore covered by the signature.

Example payloads:

```
TEXT Hello world
BOOT https://node.example/data
```

### 1.3 Message ID (`mid`)

```
mid = SHA256( t_decimal + "|" + h + "|" + fp )[0:16]   (hex)
```

- `t_decimal` is `t` as a decimal string with no leading zeros.
- `mid` is the first 16 hex characters of the SHA-256 digest.
- `mid` is the **only** dedup key used across all transports and storage.

If two atoms present the same `mid` but different `sig`, nodes MUST keep the first-seen atom and drop the rest.

### 1.4 Signature

```
sig = Base64( Ed25519_sign( private_key, UTF8( t_decimal + "|" + h + "|" + x + "|" + fp ) ) )
```

- Separator is a single `|` (0x7C). No whitespace, no newlines.
- The signing input MUST NOT include `sig`.

### 1.5 Validation

An atom is valid if and only if:

1. All five fields are present and non-empty.
2. `t` is an integer in the window `[now − 365 days, now + 300 seconds]` at receive time.
3. `h` matches `^[0-9a-z]{4,12}$`.
4. `x` is UTF-8 and ≤ 200 Unicode code points.
5. `x[0:4]` matches `^[A-Z]{4}$`.
6. If `x` has more than 4 characters, the 5th character, if a separator, MUST be a single ASCII space (0x20).
7. `fp` matches `^[A-Za-z0-9_]{4,64}$`.
8. `sig` is valid Base64 of 64 bytes and verifies against the Ed25519 public key bound to `fp` (see §5).

If the public key for `fp` is unknown, the atom is still accepted and relayed but MUST be marked locally as unverified.

### 1.6 Transport independence

The atom is transport-agnostic. HTTPS (§4) is the canonical transport. Any medium that preserves bytes — WebSocket, WebRTC, BLE, QR, NFC, LoRa, paper — MAY be used.

---

## 2. Storage

Every node (server or client) MUST maintain an append-only, content-addressed store of atoms.

### 2.1 Invariants

1. **Append-only.** Stored atoms MUST NOT be modified or deleted except under the retention policy in §2.4.
2. **Idempotent by `mid`.** Inserts MUST be idempotent. Duplicates are silently dropped.
3. **Durable before ack.** A write MUST be durable before any push is acknowledged or any sync cursor advances.
4. **Byte-stable relay.** Relayed atoms MUST carry their original JSON bytes unchanged. No re-serialization. No field reordering.
5. **No coordination.** Two honest nodes converge to the same set of atoms given the same input, without any central clock or consensus.

### 2.2 Serving-node files

A serving node (a node that exposes the HTTP endpoints in §4) MUST represent the canonical log on disk as flat files under a `/data/` path:

```
/data/log.ndjson           # canonical append-only log, one atom per line
/data/manifest.json        # advisory status for the log
/data/geo/<prefix>.ndjson  # derived spatial shard
/data/coverage.json        # derived shard metadata for clients
```

Rules:

- `log.ndjson` is canonical. It MUST NOT be rewritten or compacted except via segment rotation (§2.4).
- `manifest.json`, `geo/<prefix>.ndjson`, and `coverage.json` are **derived**. They MUST NOT be treated as more authoritative than `log.ndjson`.
- Any process that writes these files MUST leave them readable to the HTTP server at all times. A file that exists on disk but returns `403` violates this contract.

### 2.3 Spatial queries

Spatial filtering is **geohash prefix match**:

```
SELECT * FROM atoms WHERE h LIKE 'u3b%' ORDER BY t DESC
```

No spatial indexes, no PostGIS, no R-trees. Prefix-on-string is the contract.

Serving nodes SHOULD expose spatial reads as derived static shard files at `/data/geo/<prefix>.ndjson`, where every returned atom satisfies `atom.h startsWith(prefix)`. v1 nodes SHOULD pick a single fixed shard depth for coverage advertisement (4 characters is recommended).

### 2.4 Retention

- Serving nodes SHOULD retain the full log indefinitely in v1.
- If disk pressure forces rotation, the log MUST be split into immutable segments named `log.NNNN.ndjson` with monotonically increasing numbers. Clients use byte offsets — never line numbers.
- Only the oldest segment MAY be deleted. Never the middle.

---

## 3. Sync

Replication is **gossip over append-only logs** with two primitive operations:

- **PUSH** — a node sends one atom to a peer.
- **PULL** — a node reads new bytes from a peer's append-only log and appends to its own store.

There is no leader, no quorum, no consensus. Every node is an equal peer.

### 3.1 Invariants

1. **At-least-once delivery.** Any atom accepted by one honest node MUST eventually reach every connected honest node.
2. **Idempotent application.** Receiving the same atom multiple times is indistinguishable from receiving it once.
3. **Cursor monotonicity.** A per-peer pull cursor MUST only advance, never retreat, and MUST advance only after durable write of all atoms up to that byte offset.
4. **Partial-line safety.** A pull MUST NOT advance the cursor past an incomplete NDJSON line. Only complete `\n`-terminated lines are committed.
5. **Loop-free.** Relayed atoms MUST NOT be pushed back to their source peer within the same cycle.

---

## 4. HTTP transport (canonical)

Every serving node MUST expose:

| Method | Path                         | Purpose |
|--------|------------------------------|---------|
| POST   | `/data/inbox`                | Push one atom (JSON body) |
| GET    | `/data/log.ndjson`           | Pull the canonical log (supports HTTP `Range`) |
| GET    | `/data/manifest.json`        | Advisory status |
| GET    | `/data/geo/<prefix>.ndjson`  | Derived spatial shard |
| GET    | `/data/coverage.json`        | Derived shard coverage metadata |

All endpoints MUST serve with `Access-Control-Allow-Origin: *`.

### 4.1 POST `/data/inbox`

Request body: a single atom JSON (see §1). Max 4 KiB.

Response:

- `201 Created` — atom accepted. Body: `{"ok":true,"mid":"<mid>","t":<t>}`
- `200 OK` — atom accepted and was already known (dedup hit). Same body.
- `400 Bad Request` — schema invalid. `{"ok":false,"error":"..."}`
- `401 Unauthorized` — signature verification failed (if the node enforces §5).
- `413 Payload Too Large` — body exceeded 4 KiB.
- `429 Too Many Requests` — rate-limited. `Retry-After` REQUIRED.

On any 2xx, the atom MUST be appended to `log.ndjson` as a single line **before** the response is returned.

### 4.2 GET `/data/log.ndjson` with `Range`

Clients use `Range: bytes=<cursor>-` to fetch only the tail beyond their local cursor.

Response:

- `206 Partial Content` — tail returned.
- `200 OK` — full log returned (client had no cursor).
- `416 Range Not Satisfiable` — cursor ≥ current size. The client is up-to-date. The cursor MUST NOT be reset in this case unless the client independently detects that the remote log has shrunk (segment rotation or wipe).

**Client processing algorithm:**

```
buf = http.body
last_newline = buf.rfind("\n")
if last_newline < 0: return 0
complete = buf[:last_newline + 1]
for line in complete.split("\n"):
    if line is empty: continue
    atom = json.parse(line)
    if not valid(atom): skip         # do NOT abort the pull
    INSERT atom ON CONFLICT(mid) DO NOTHING
local_cursor += len(complete)        # advance by complete bytes only
```

### 4.3 GET `/data/manifest.json`

Returns advisory status:

```json
{"node":"n1","version":1,"count":1234,"bytes":456789,"last_t":1711961189000,"updated":1711961200000}
```

- `bytes` MUST equal `stat(log.ndjson).size` at the moment of update.
- `manifest.json` is a hint for dashboards and liveness checks. It is NOT a sync authority. Clients MUST NOT drive cursor decisions from it.

### 4.4 GET `/data/geo/<prefix>.ndjson`

Returns a derived NDJSON shard. Every atom in the response MUST satisfy `atom.h startsWith(prefix)`. A node MAY return `404` for prefixes it does not serve. A node MAY be partial and expose only a subset of prefixes.

### 4.5 GET `/data/coverage.json`

Returns derived metadata describing which geohash shard prefixes the node serves:

```json
{
  "node": "n1",
  "version": 1,
  "prefix_len": 4,
  "updated": 1711961189000,
  "prefixes": {
    "t069": { "count": 30, "last_t": 1711961189000 },
    "f044": { "count": 4,  "last_t": 1711960000000 }
  }
}
```

- `prefix_len` declares the fixed geohash prefix length.
- `count` and `last_t` describe each shard.
- Clients SHOULD use `coverage.json` to decide which shard files to request from partial nodes.

---

## 5. Signature verification

- v1 nodes MUST accept, store, and relay atoms whose signature they cannot verify (unknown `fp`). Such atoms SHOULD be marked unverified locally.
- When a future key-binding atom for `fp` arrives, nodes SHOULD retroactively verify pending atoms.
- v1 nodes MAY reject atoms with verifiable-but-invalid signatures. If rejected, the `mid` SHOULD be blacklisted for 24 hours to prevent replay spam.

Stronger policy (mandatory verification) is deferred to a future RFC.

---

## 6. Peer topology

- Each node maintains a peers list. A node MUST NOT list itself.
- The recommended v1 topology is **full mesh** among serving nodes.
- Clients use serving nodes as their sync peers. Client-to-client sync is out of scope for v1.

On local atom creation, a node MUST enqueue one push per enabled peer. Pushes and pulls are independent; both MUST be retried with bounded exponential backoff. Specific retry intervals, batch sizes, and worker scheduling are implementation-defined.

Clock skew is not a sync problem. It is a UI ordering problem. Atoms within the window defined in §1.5 MUST NOT be rejected on timestamp grounds.

---

## 7. Conformance — Punkti v1-MINIMAL

An implementation claims conformance to Punkti v1-MINIMAL if it:

1. Serves `/data/inbox`, `/data/log.ndjson`, `/data/manifest.json` per §4.
2. Preserves all atom bytes on relay (§2.1.4).
3. Deduplicates by `mid` (§1.3).
4. Supports HTTP `Range` pulls with partial-line safety (§4.2, §3.1.4).
5. Implements cursor monotonicity (§3.1.3).
6. Validates atoms per §1.5 at least at the syntactic level.

Implementing these six rules is sufficient to interoperate.

---

## 8. Scope boundary

Out of scope for v1 (each gets its own RFC and MUST NOT violate this core): identity derivation, key registration/revocation, encryption, trust/reputation, AI identity, non-HTTP transports (WS/WebRTC/BLE), UI, moderation, payments.

---

## 9. Change log

| Version | Date       | Change |
|---------|------------|--------|
| pre-v0.4 | early iterations | Superseded. 5-field atom shape, outbox concept, and 3D geohash introduced. |
| v0.4    | 2026-04-19 | Core RFC — narrowed to protocol + storage + sync; RFC 2119 language |
| v0.4.1  | 2026-04-21 | Clarified: `/data/*` files MUST remain HTTP-readable after every rewrite |
| v0.5    | 2026-04-23 | Core-only. Removed AI-review framing, consensus checklist, open questions, credits. Neutralized implementation-specific paths. Renamed `StarNode` → serving node throughout. |

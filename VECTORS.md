# Punkti v0.5.1 Normative Interoperability Vectors

These vectors are normative for Punkti v0.5.1.

An implementation claiming `v0.5.1-MINIMAL` conformance MUST produce the stated accept/reject, hash, signature, deduplication, and cursor outcomes.

All text is UTF-8. Hex is lowercase. Base64url is unpadded and canonical.

---

## V1 — End-to-end canonical atom

Ed25519 private-key seed:

```text
0101010101010101010101010101010101010101010101010101010101010101
```

Derived public key:

```text
hex  = 8a88e3dd7409f195fd52db2d3cba5d72ca6709bf1d94121bf3748801b40f6f5c
b64u = iojj3XQJ8ZX9UtstPLpdcspnCb8dlBIb83SIAbQPb1w
```

Core values:

```text
t  = 1711961189000
h  = u3butz7k9q_23
x  = TEXT Hello world
fp = testfp01
```

Exact signing input:

```text
1711961189000|u3butz7k9q_23|TEXT Hello world|testfp01
```

Length:

```text
53 bytes
```

Signing-input UTF-8 hex:

```text
313731313936313138393030307c75336275747a376b39715f32337c544558542048656c6c6f20776f726c647c7465737466703031
```

Expected `mid`:

```text
1e4c8e42ac2a206ece143e19c9228df6bf0000198315263c4f96ee3c5d638682
```

Expected raw Ed25519 signature hex:

```text
6ff8c0ed7d72c4495cd06fd1bbd5487008fc3cbdfffd28ccb07ef7894b6b5ceb02d19f2de60e83a12f3625f62ab616634d9f63840a003ad87ec1614ea21fed03
```

Expected canonical `sig`:

```text
b_jA7X1yxElc0G_Ru9VIcAj8PL3__SjMsH73iUtrXOsC0Z8t5g6DoS82JfYqthZjTZ9jhAoAOth-wWFOoh_tAw
```

Expected: structural validation succeeds; with the public key above bound to `testfp01`, verification is **VERIFIED**.

---

## V2 — Payload participates in `mid`

Change only:

```text
x = TEXT Hello worlds
```

Expected `mid`:

```text
81842a1a2c3dd96b3344d092ccc19b1a1395dd7ba055678b199f13cfee66b4fa
```

This MUST differ from V1.

---

## V3 — `|` inside payload is data, not a parser ambiguity

Use:

```text
x = TEXT a|b
```

Expected `mid`:

```text
b2df7296f34d6199e96150cabd106b94b654650c2ccd961edee5e25d849924fa
```

The atom is structurally valid.

---

## V4 — `sig` is proof, not statement identity

Sign V1's exact `signing_input` with this second seed:

```text
0202020202020202020202020202020202020202020202020202020202020202
```

Second public key:

```text
hex  = 8139770ea87d175f56a35466c34c7ecccb8d8a91b4ee37a25df60f5b8fc9b394
b64u = gTl3Dqh9F19Wo1Rmw0x-zMuNipG07jeiXfYPW4_Js5Q
```

Second signature:

```text
JNtySEhu6Sg7ganx0FDu7ITuwGzQdgawG0hL6S07mF_b2XneuCye1dfxerPeteSYkAXGcrUtkhL05v2VSTwCBA
```

This vector tests statement identity and duplicate handling only; it does not assert a binding for the second key.

Expected:

- `mid` remains exactly `1e4c8e42ac2a206ece143e19c9228df6bf0000198315263c4f96ee3c5d638682`.
- If V1 is already stored, delivery of this variant is a duplicate.
- No second log line is appended.
- The first accepted core representation remains unchanged.

---

## V5 — Spatial-address grammar

The delimiter is `_`. The decimal point is used only inside `z`.

### MUST accept

```text
u3butz7k9q_23
u3butz7k9q_0
u3butz7k9q_-18
u3butz7k9q_0.5
u3butz7k9q_-0.5
u3butz7k9q_23.5
u3butz7k9q_23.025
u3butz7k9q_0.001
u3bu_23
u3butz7k9qbc_23
u3bu_12345678901234567890123456
```

The last case is intentionally physically implausible but syntactically valid: Punkti does not impose a physical `z` range; the complete `h` length bound is the resource bound.

### MUST reject

```text
u3butz7k9q.23
u3butz7k9q_23.0
u3butz7k9q_23.50
u3butz7k9q_0.0
u3butz7k9q_-0
u3butz7k9q_-0.0
u3butz7k9q_23.
u3butz7k9q_.5
u3butz7k9q_023
u3butz7k9q_+23
u3butz7k9q_1e3
u3butz7k9q_23.4567
u3butz7k9q
U3BUTZ7K9Q_23
u3ailz7k9q_23
u3b_23
u3butz7k9qbcd_23
```

Any `h` whose UTF-8 encoding exceeds 32 bytes MUST be rejected.

---

## V6 — Canonical timestamp token

### MUST accept

```text
0
1
-1
1711961189000
-1711961189000
9007199254740993
-9223372036854775808
9223372036854775807
```

`9007199254740993` is included to catch implementations that incorrectly route `t` through IEEE-754 binary64.

### MUST reject

```text
-0
01
+1
1711961189000.0
1.711961189e12
1711961189e3
-9223372036854775809
9223372036854775808
"1711961189000"
```

---

## V7 — Payload code points

MUST accept:

```text
TEXT
TEXT body
```

An `x` containing exactly 200 Unicode code points MUST be accepted if all other rules pass.

An `x` containing 201 Unicode code points MUST be rejected.

Counting is by Unicode code points, not UTF-8 bytes and not UTF-16 code units.

The following MUST be rejected:

```text
TEXTbody
TEXT 
text body
```

A JSON string containing an unpaired surrogate escape MUST be rejected as invalid Unicode input.

---

## V8 — No Unicode normalization

Use:

```text
x1 = TEXT café
x2 = TEXT café
```

They are canonically equivalent Unicode text but distinct Punkti values.

Expected:

```text
mid(x1) = d5268bce5ceee4d7f629739d0c7578aeb50165ca863330d9ffb69363074ef5e9
mid(x2) = e212018083d77ba0b52e4174a7e301ac06a825a18a097c1583f1224377b05a35
```

Both are valid and the two `mid`s MUST differ.

---

## V9 — Duplicate JSON keys

This input is invalid:

```json
{"t":1711961189000,"h":"u3butz7k9q_23","x":"TEXT Hello world","fp":"testfp01","sig":"b_jA7X1yxElc0G_Ru9VIcAj8PL3__SjMsH73iUtrXOsC0Z8t5g6DoS82JfYqthZjTZ9jhAoAOth-wWFOoh_tAw","x":"TEXT other"}
```

A parser's first-wins or last-wins behaviour MUST NOT affect the outcome: the object is rejected before atom validation.

---

## V10 — JSON reserialization invariance

These two objects contain the same core values and MUST produce the same `signing_input`, `mid`, and signature-verification outcome:

```json
{"t":1711961189000,"h":"u3butz7k9q_23","x":"TEXT Hello world","fp":"testfp01","sig":"b_jA7X1yxElc0G_Ru9VIcAj8PL3__SjMsH73iUtrXOsC0Z8t5g6DoS82JfYqthZjTZ9jhAoAOth-wWFOoh_tAw"}
```

```json
{ "sig":"b_jA7X1yxElc0G_Ru9VIcAj8PL3__SjMsH73iUtrXOsC0Z8t5g6DoS82JfYqthZjTZ9jhAoAOth-wWFOoh_tAw", "fp":"testfp01", "x":"TEXT Hello world", "h":"u3butz7k9q_23", "t":1711961189000 }
```

Member order and insignificant JSON whitespace are not identity.

---

## V11 — Extensions are non-core

Adding:

```json
{"note":"local metadata","mid":"0000000000000000000000000000000000000000000000000000000000000000"}
```

to V1 MUST NOT change the recomputed `mid`.

The extension field named `mid` MUST be ignored for protocol identity.

A relay may drop these extension fields and still refer to the same atom.

---

## V12 — POST and dedup lifecycle

Starting with an empty node and no public-key binding for `testfp01`:

1. POST V1 → `201 Created`; exactly one logical-log record is appended.
2. POST V1 again → `200 OK`; logical-log byte length is unchanged.
3. POST V4 (same `mid`, different `sig`) → `200 OK`; logical-log byte length remains unchanged and the stored V1 `sig` is unchanged.

---

## V13 — Canonical Base64url

Canonical V1 signature:

```text
b_jA7X1yxElc0G_Ru9VIcAj8PL3__SjMsH73iUtrXOsC0Z8t5g6DoS82JfYqthZjTZ9jhAoAOth-wWFOoh_tAw
```

This 86-character string decodes to the same 64 bytes in permissive Base64url decoders but is non-canonical and MUST be rejected:

```text
b_jA7X1yxElc0G_Ru9VIcAj8PL3__SjMsH73iUtrXOsC0Z8t5g6DoS82JfYqthZjTZ9jhAoAOth-wWFOoh_tAx
```

Also reject:

- padded 88-character form,
- standard-Base64 `+` or `/`,
- any decoded length other than 64 bytes.

A conforming validator can test canonicality by decode → unpadded-Base64url re-encode → byte-for-byte string equality.

---

## V14 — Strict Ed25519 verification

V1's canonical signature is VERIFIED under the V1 public key.

The following signature keeps `R` unchanged but replaces scalar `S` with `S + L`:

```text
b_jA7X1yxElc0G_Ru9VIcAj8PL3__SjMsH73iUtrXOvvpJWKAHKV-QXTHJkJsPV3TZ9jhAoAOth-wWFOoh_tEw
```

Raw hex:

```text
6ff8c0ed7d72c4495cd06fd1bbd5487008fc3cbdfffd28ccb07ef7894b6b5cebefa4958a007295f905d31c9909b0f5774d9f63840a003ad87ec1614ea21fed13
```

It is 64 bytes and structurally decodable, but MUST be **INVALID** because `S >= L`.

Implementations MUST also reject non-canonical point encodings and points outside the non-identity prime-order subgroup as required by Punkti §1.6.

---

## V15 — Logical-log line bound

A serialized JSON record containing exactly 4096 bytes before its terminating LF is within the protocol limit.

A record requiring 4097 bytes before LF violates the protocol.

A client encountering more than 4096 bytes from a record start without LF MUST stop processing that response without advancing past the previous complete record.

---

## V16 — `log_id` and Range generation race

Let the stored cursor be:

```text
(aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa, 100)
```

The client requests:

```text
Range: bytes=100-
```

### Same generation

Response:

```text
206 Partial Content
Punkti-Log-Id: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Content-Range: bytes 100-199/200
```

Expected: process complete records and advance from offset 100.

### Generation changed between manifest and Range

Response:

```text
206 Partial Content
Punkti-Log-Id: bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb
Content-Range: bytes 100-199/200
```

Expected:

- apply **zero** body bytes,
- do not advance the old cursor,
- replace cursor with `(bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb, 0)`,
- retry from byte 0.

This outcome is required even if a previously fetched manifest reported the old generation.

---

## V17 — `416`, `200`-for-Range, and generation lifecycle

### EOF

Cursor:

```text
(aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa, 200)
```

Response:

```text
416 Range Not Satisfiable
Punkti-Log-Id: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Content-Range: bytes */200
```

Expected: up to date; cursor unchanged.

### Impossible shrink

Cursor offset `250`, same `log_id`, `Content-Range: bytes */200`.

Expected: peer inconsistency; MUST NOT silently reset or advance.

### Server ignores Range

A Range request receives:

```text
200 OK
Punkti-Log-Id: aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
```

with the full log from byte 0.

Expected: process idempotently from byte 0 and set the new cursor from the complete bytes processed. Under the same generation, the complete returned length MUST NOT be below the previously stored offset.

### Restart and rebase

- process restart with byte-identical logical log → same `log_id`;
- any rebase/prune/rewrite affecting exposed bytes → new `log_id`;
- a retired `log_id` MUST never be reused.

---

## V18 — Verification-state lifecycle and proof-race boundary

With no binding for `testfp01`, V1 is UNVERIFIED.

With the V1 public key bound to `testfp01`, V1 is VERIFIED.

With that same binding and a modified signature, the incoming atom is INVALID and MUST NOT be appended.

If an already stored UNVERIFIED record later becomes INVALID:

- its bytes in the current log generation are not rewritten,
- it is no longer actively PUSHed,
- removing it from the canonical log requires a rebase and new `log_id`.

A later same-`mid` proof cannot replace the existing line inside that generation.

This vector intentionally records the v0.5.1 security boundary: anti-censorship before durable `fp` binding is not provided by the core protocol.

---

## V19 — Spatial shard matching

Given atoms at:

```text
u3butz7k9q_23
u3butz7k9q_-18
u3bvaaaaaa_10
```

prefix:

```text
u3but
```

matches exactly the first two.

Prefix matching is applied only to `xy`, never to `_z`.

---

## Release gate

Before freezing v0.5.1, at least two independent implementations SHOULD be run against these vectors.

Any disagreement in structural validity, `mid`, signature verification, duplicate handling, log-generation handling, or cursor movement is a protocol defect to resolve before freeze.

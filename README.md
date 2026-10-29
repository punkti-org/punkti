# Punkti

> Punkti is an open protocol for placing small, cryptographically signed messages at readable addresses in 3D space.

A Punkti atom is a tiny signed event anchored to:

- a horizontal 2D geohash,
- a WGS84 ellipsoidal height,
- a timestamp,
- an author/key reference,
- a short message.

---

## The goal

> Give every cubic meter of the world a readable and writable address.

The recommended Punkti point profile uses:

```text
10-character 2D geohash + "_" + integer WGS84 metres
```

Example:

```text
u3butz7k9q_23
```

Horizontal and vertical components remain separately searchable and can use finer precision when required.

---

## What this repo is

This repository contains the **Punkti protocol specification**.

It defines:

1. **The atom** — a signed message connected to a point in 3D space.
2. **The storage model** — append-oriented, content-addressed atom storage.
3. **The sync model** — v0.5.1 uses immutable HTTP byte-range log generations; the frozen spatial-only v0.6 protocol specifies static publication over ordinary HTTP, with dynamic sync APIs optional accelerators.

Punkti is not a hosted product. It is the open foundation that different implementations can build on.

---

## Punkti vs Punkto

| Name | Meaning |
|---|---|
| **Punkti** | The open protocol and shared specification |
| **Punkto** | One hosted/product implementation built on Punkti |

Think:

- HTTP → many web servers
- Git → many hosts
- Matrix → many clients
- Punkti → many spatial nodes

---

## Spatial addressing

Punkti keeps horizontal and vertical addressing separate:

```text
h = <2D-geohash>_<WGS84-ellipsoidal-height-metres>
```

Examples:

```text
u3butz7k9q_23
u3butz7k9q_-18
u3butz7k9q_0.5
u3butz7k9q_23.025
```

`_` is the XY/Z delimiter. `.` is used only as the decimal point inside the height.

This keeps addresses readable and makes ordinary database indexing cheap:

- geohash prefix matching for XY,
- numeric range filtering for Z,
- no required GIS database,
- no 3D bit-interleaving decoder.

---

## Example

A drone records a measurement:

- location → `u3butz7k9q`
- WGS84 ellipsoidal height → `42 m`
- Punkti spatial address → `u3butz7k9q_42`
- message → `"wind: 12m/s"`

That becomes a signed Punkti atom at that spatial address.

Any compatible node can later retrieve or replicate it.

---

## Status

Status: **FROZEN — Founder-accepted spatial-only Punkti v0.6, 2026-10-29**. The 2026-10-27 freeze was reopened in place for the mandatory `h=XY_Z` correction; following independently passing Python/JS conformance and Founder acceptance, v0.6 was re-frozen in place without a version bump. The `punkti/0.6` namespace and frozen v0.5.1 history remain unchanged. Implementation verification does not claim live network availability, PQ signature support or guaranteed long-term preservation.

Punkti v0.6 defines the complete Layer-3 contract: canonical carrier/signing/MID/R identity, human-readable `h=<xy>_<z>` spatial addressing, and static publication/replication over ordinary HTTP. Dynamic PUSH/Range-PULL/session mechanisms are optional accelerators, not conformance requirements. Replication packs are a standardized optional accelerator; canonical individual R objects remain sufficient. Every v0.6 record requires canonical `h=<xy>_<z>`: 4–12 lowercase geohash characters, an underscore and canonical exact signed decimal WGS84 ellipsoidal metres, 6..32 characters total. Empty, missing, null or malformed addresses fail structural validation before algorithm dispatch. Publications contain only geographical index keys; all eligible records participate in XY routing. An unrestricted scope selects all anchored records. Punkto L4 replies use their parent's same exact `h`; Punkti L3 keeps references opaque.

Frozen v0.5.1 remains available unchanged as the predecessor protocol version. Future incompatible Layer-3 changes require a new protocol version.

---

## Specification and vectors

- [Punkti v0.6 carrier / Layer-3 contract](./Punkti-v0.6.md)
- [Punkti v0.6 XY_Z spatial contract](./Punkti-v0.6-spatial.md)
- [Punkti v0.6 static publication contract](./Punkti-v0.6-publication.md)
- [Punkti v0.6 normative publication vectors](./VECTORS-v0.6.md)
- [Frozen Punkti v0.5.1 specification](./Punkti.md)
- [Normative v0.5.1 vectors](./VECTORS.md)

---

## Design principles

- Small beats large
- Plain text beats magic
- Append-only beats mutable state
- Interop beats platform lock-in
- Local-first beats cloud-first
- Subtraction beats addition

---

## Conformance

A minimal Punkti v0.5.1 implementation can:

1. parse and validate the five core atom fields,
2. compute and deduplicate by the content-complete `mid`,
3. verify the strict Ed25519 profile when a key binding is available,
4. preserve core values while treating extensions as non-core,
5. expose immutable logical-log generations,
6. sync with `(log_id, byte_offset)` and response-bound `Punkti-Log-Id`,
7. implement the canonical `xy_z` spatial-address grammar,
8. pass the normative interoperability vectors.

---

## License

CC0 1.0 Universal.

The protocol is public domain.
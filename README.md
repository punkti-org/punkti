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
3. **The sync model** — peer replication over immutable HTTP byte-range log generations.

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

**v0.5.1 release candidate**

v0.5.1 is the protocol-hardening release: deterministic statement identity, canonical parsing, immutable log generations, and race-safe HTTP Range synchronization.

It is intentionally wire-breaking relative to v0.5.

Identity/key succession, post-quantum authority, and algorithm agility are reserved for Punkti v0.6.

---

## Specification and vectors

- [Punkti Specification](./Punkti.md)
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

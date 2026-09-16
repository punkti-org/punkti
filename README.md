# Punkti

> Punkti is an open protocol for placing small, cryptographically signed messages at readable addresses in 3D space.

A Punkti atom is a tiny signed event anchored to:

- a horizontal 2D geohash,
- a WGS84 vertical coordinate,
- a timestamp,
- an author/key reference,
- a short message.

---

## The goal

> Give every cubic meter of the world a readable and writable address.

The recommended Punkti point profile uses:

```text
10-character 2D geohash + integer WGS84 metres
```

Example:

```text
u3butz7k9q.23
```

The horizontal and vertical components remain separately searchable and can use finer precision when required.

---

## What this repo is

This repository contains the **Punkti protocol specification**.

It defines:

1. **The atom** — a signed message connected to a point in 3D space.
2. **The storage model** — append-oriented, content-addressed atom storage.
3. **The sync model** — simple peer replication over HTTP byte-range pulls.

Punkti is not a hosted product.

It is the open foundation that different implementations can build on.

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
h = <2D-geohash>.<WGS84-height-metres>
```

Examples:

```text
u3butz7k9q.23
u3butz7k9q.-18
u3butz7k9q.23.5
```

This keeps addresses readable and makes ordinary database indexing cheap:

- geohash prefix matching for XY,
- numeric range filtering for Z,
- no required GIS database,
- no 3D bit-interleaving decoder.

---

## Example

A drone records a measurement:

- location → `u3butz7k9q`
- WGS84 height → `42 m`
- Punkti spatial address → `u3butz7k9q.42`
- message → `"wind: 12m/s"`

That becomes a signed Punkti atom stored at that spatial address.

Any compatible node can later retrieve or replicate it.

---

## Status

**Draft v0.5.1**

v0.5.1 is a protocol-hardening release focused on deterministic object identity, persistent validity, spatial-address clarity, and robust sync.

Post-quantum authority and algorithm agility are intentionally reserved for Punkti v0.6.

---

## Specification

Read the protocol:

[Punkti Specification](./Punkti.md)

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
2. compute and deduplicate by `mid`,
3. preserve the five core values,
4. expose the canonical HTTP resources,
5. sync append-only logical logs using `(log_id, byte_offset)`,
6. implement the Punkti XY.Z spatial-address grammar.

---

## License

CC0 1.0 Universal.

The protocol is public domain.

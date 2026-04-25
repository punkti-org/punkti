# Punkti

> Punkti is an open protocol for placing small, cryptographically signed messages at precise addresses in 3D space.

A Punkti atom is a tiny signed event anchored to:

* a 3D geohash
* a timestamp
* an author key
* a short message

---

## The goal

> Give every cubic meter of the world a readable and writable address.

---

## What this repo is

This repository contains the **Punkti protocol specification**.

It defines:

1. **The atom** — a signed message connected to a point in 3D space
2. **The storage format** — append-only NDJSON logs
3. **The sync model** — simple peer replication over HTTP range pulls

Punkti is not a hosted product.

It is the open foundation that different implementations can build on.

---

## Punkti vs Punkto

| Name       | Meaning                                             |
| ---------- | --------------------------------------------------- |
| **Punkti** | The open protocol and shared specification          |
| **Punkto** | One hosted / product implementation built on Punkti |

Think:

* HTTP → many web servers
* Git → many hosts
* Matrix → many clients
* Punkti → many spatial nodes

---

## Why this matters

Most digital systems describe the world using:

* files
* URLs
* database rows

Punkti describes the world using:

> addressed space

This enables:

* spatial computing
* AR anchors
* drones
* sensors
* geology and terrain data
* underground infrastructure
* decentralized mapping
* local-first field notes

---

## Example

A drone records a measurement at a specific location:

* location → 3D geohash: `u4pruydqqvj`
* altitude → `42m`
* message → `"wind: 12m/s"`

This becomes a signed Punkti atom stored at that address.

Any compatible node can later retrieve or replicate it.

---

## Conceptual model

```
        Z (altitude)
        |
        ●  ← Punkti (signed message)
       / \
      /   \
     X     Y

A point in 3D space becomes an addressable location.
```

---

## Status

**Early draft (v0.5)**

The protocol is intentionally small and evolving.

Feedback, criticism, and alternative implementations are welcome.

---

## Specification

Read the full protocol here:

👉 [Punkti Specification](./Punkti.md)

---

## Design principles

* Small beats large
* Plain text beats magic
* Append-only beats mutable state
* Interop beats platform lock-in
* Local-first beats cloud-first
* Subtraction beats addition

---

## Conformance

An implementation is Punkti-compatible if it can:

1. create valid atoms
2. sign them
3. store them without modification
4. expose append-only logs
5. sync logs with peers
6. reject invalid signatures



---

## License

CC0 1.0 Universal.

The protocol is public domain.

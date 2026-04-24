# Punkti

> Punkti is a decentralized network of 200-character signed messages anchored to 3D geohashes, replicated by append-only log gossip.

**Status:** DRAFT v0.5 — open for review and implementation.

**Specification:** [Punkti.md](./Punkti.md)

---

## What this repo is

This repo holds the **Punkti protocol specification**. Nothing else.

Punkti is the open protocol. It defines three things only: the **atom** (a signed text event at a point in time and 3D space), the **storage model** (append-only NDJSON), and the **sync contract** (gossip over HTTP with `Range` pulls). If an atom can move over any medium that preserves bytes, Punkti can work over that medium.

**Punkti is not a product.** There are multiple implementations, and more are expected. Think Matrix / Element, or HTTP / Apache.

- **Punkto** — one reference implementation (PWA + serving-node mesh)
- *your-kti-here* — write your own; interop is the only requirement

## Conformance

An implementation interoperates with the network if it satisfies the six rules in [Punkti.md §7](./Punkti.md#7-conformance--punkti-v1-minimal). That is sufficient — no registration, no central authority.

## License

[CC0 1.0 Universal](./LICENSE) — public domain dedication. Implementations are free and unrestricted.

## Contributing

Protocol changes are proposed as diffs against [Punkti.md](./Punkti.md). Open a pull request with the change, the reason, and what gets removed. Subtraction beats addition. If a change does not fit the one-sentence product above, it does not belong here.

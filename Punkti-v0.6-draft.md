# Punkti v0.6 — Draft Architecture

**Status:** Founder-accepted architecture direction; **not frozen**  
**Frozen predecessor:** Punkti v0.5.1 in [Punkti.md](./Punkti.md) remains unchanged.  
**Purpose:** define the accepted Layer-3 direction before the final normative publication/vector pass.

## 1. First principle

> **Nodes validate and publish Punkti. Ordinary HTTP serves Punkti. Other nodes replicate Punkti by reading, independently verifying, deduplicating and republishing what a publication serves.**

Static publication is the normative replication/correctness path for the Punkti v0.6 design.

A serving side MUST NOT require a live Punkti daemon for another implementation to read and replicate already-published data. A replication peer MAY therefore be represented by a Punkti publication base URL.

Dynamic services such as PUSH, Range PULL, notifications, callbacks, peer sessions, WebSockets or live query APIs MAY be provided as performance accelerators. Punkti correctness and interoperability MUST NOT depend on them.

## 2. Layer boundary

Punkti Layer 3 owns only generic infrastructure semantics:

- canonical immutable atoms/records;
- MID and exact record identity;
- signatures/provenance;
- human-readable spatial addressing;
- static publication;
- generic replication and independent verification.

Punkti does not define application meaning. Messaging, BBS behavior, BRTH identity UX, trust/reputation, moderation, AI behavior and other application semantics belong above Layer 3.

## 3. Spatial address

The accepted v0.6 spatial direction is:

```text
h = <xy>_<z>
```

### XY

- standard lowercase 2D geohash;
- 4–12 characters;
- precision 10 is recommended/default only, not mandatory;
- prefix semantics describe progressively coarser horizontal regions.

### Z

- separate signed exact-decimal WGS84 ellipsoidal height in metres;
- negative values are valid;
- no artificial altitude minimum/maximum;
- no fixed decimal-place cap beyond the canonical spelling and total-address size constraint;
- exact decimal comparison, not IEEE-754 float semantics.

Examples:

```text
u3butz7k9q_23
u3butz7k9q_-18
u3butz7k9q_23.025
```

## 4. Atom and record identity

The v0.6 carrier work distinguishes:

- **MID** — logical atom identity derived from the canonical body;
- **SHA256(R)** — exact immutable transported-record identity.

The signature is not part of MID. Multiple exact record variants can therefore share a MID while remaining distinguishable by SHA256(R). Invalid variants MUST NOT poison or replace a valid record for the same MID.

The final v0.6 normative bytes remain subject to freeze review. This draft does not rewrite v0.5.1.

## 5. Static publication model

The accepted logical model is intentionally small:

```text
tiny mutable root
    ↓
immutable content-addressed manifest
    ↓
immutable spatial indexes / replication packs
    ↓
immutable individual R objects
```

The exact filesystem path names used by the first experiment are not themselves frozen protocol semantics.

Required properties:

- only the minimum discovery pointer should be mutable;
- referenced immutable objects are content-addressed by the digest of their exact bytes;
- publication order must make referenced immutable objects durable/available before a mutable root points to them;
- a reader verifies object digests and every imported Punkti record independently;
- canonical individual R objects remain retrievable/addressable by SHA256(R);
- XY prefix locality may be used for static spatial discovery without changing canonical `h`;
- Z remains independently filterable;
- unanchored atoms remain representable;
- stale roots, missing/corrupt objects and publication races must fail safely and be retryable;
- a source publication never proves global completeness or freshness.

## 6. Replication packs

The accepted experiment compared:

1. an index-only layout, where readers fetch many individual record objects;
2. immutable replication packs containing complete carrier records.

The pack profile is preferred for bootstrap/read efficiency while keeping individual R objects available.

Measured experiment results:

- 5,000 fresh atoms: 5,005 index-only requests vs 26 pack requests (~192x fewer);
- 100,000 atoms: 100,005 index-only requests vs 398 pack requests (~251x fewer);
- transferred bytes increased by only about 1.2% in the measured runs.

These measurements support the architecture choice; they are not normative performance guarantees.

## 7. Daemon-free acid test

The accepted experiment demonstrated that a publication created by a Punkti implementation could be served afterward by a plain static HTTP server with the Punkti application stopped.

An independent reader could:

1. discover the root;
2. fetch immutable publication objects;
3. verify object hashes;
4. verify and admit Punkti records;
5. deduplicate;
6. republish them;
7. converge with other nodes under compatible coverage/retention policies.

This property is a core design requirement for the v0.6 publication profile.

## 8. Node trust

Cryptographic validity and node-service trust are separate.

A client may prefer some nodes for availability, freshness, retention, privacy, geographic coverage or performance. That preference is client/operator policy, not Layer-3 consensus.

A node cannot make an invalid atom valid. It can omit, delay, filter or forget data. Comparison across independent publications may expose such differences, but Punkti does not invent a global-completeness guarantee.

## 9. Optional acceleration

A high-performance node may offer dynamic capabilities such as:

- PUSH;
- range PULL;
- notifications;
- session-oriented replication;
- richer query APIs.

These are optional accelerators. A conforming v0.6 static publication must remain sufficient for correctness and interoperability.

## 10. What remains before freeze

Before Punkti v0.6 can be frozen, the accepted architecture must be reduced to the smallest normative publication contract with deterministic vectors, including:

- root → manifest discovery;
- immutable-object digest verification;
- deterministic replication-pack framing;
- exact R/MID verification;
- same-MID/distinct-R anti-poisoning behavior;
- XY prefix selection;
- negative/decimal Z preservation;
- stale/missing/corrupt publication behavior;
- convergence using independent implementations and static-only origins.

The final normative pass must separate logical protocol objects/bytes from incidental filesystem aesthetics.

No production deployment or v0.6 freeze is implied by this document.

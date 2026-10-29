# Punkti v0.6 — Draft Architecture (superseded)

Status: **FROZEN — Founder-accepted spatial-only Punkti v0.6, 2026-10-29**. The 2026-10-27 freeze was reopened in place for the mandatory `h=XY_Z` correction; following independently passing Python/JS conformance and Founder acceptance, v0.6 was re-frozen in place without a version bump. The `punkti/0.6` namespace and frozen v0.5.1 history remain unchanged. Implementation verification does not claim live network availability, PQ signature support or guaranteed long-term preservation.

This draft remains superseded by the linked v0.6 Layer-3 documents.

The accepted normative documents are:

- [Punkti v0.6 carrier / Layer-3 contract](./Punkti-v0.6.md)
- [Punkti v0.6 XY_Z spatial contract](./Punkti-v0.6-spatial.md)
- [Punkti v0.6 static publication contract](./Punkti-v0.6-publication.md)
- [Punkti v0.6 normative publication vectors](./VECTORS-v0.6.md)

Frozen v0.5.1 remains unchanged in [Punkti.md](./Punkti.md).

The applicable Founder decisions are:

1. static publication is normative replication/correctness;
2. dynamic sync mechanisms are optional accelerators;
3. replication packs are a standardized optional accelerator, not required for correctness;
4. every v0.6 record requires canonical `h=<xy>_<z>` and participates in XY routing; publication index keys are geographical only;
5. carrier C/S/MID/R framing stays unchanged; the spatial-only correction was re-frozen following independent conformance tests and Founder acceptance.

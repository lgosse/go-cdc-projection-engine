---
type: Architecture Decision Record
title: "ADR-0055: Derived Elasticsearch fence metadata"
description: Defines the private, rebuildable Elasticsearch metadata used for atomic local projection fencing.
tags: [architecture, adr, elasticsearch, metadata, fencing, contracts]
status: accepted
decision_id: ADR-0055
accepted_on: 2026-09-25
owner: Project maintainer
conditions:
  - Reserve a private root `engine_meta` namespace with a metadata version and per-contributing-entity fence entries.
  - Each fence entry contains a scoped entity key, source.ts_ms, source.ord, deleted marker, and mutation fingerprint.
  - MongoDB remains authoritative; the Elasticsearch copy is rebuildable, used only for atomic local fencing, and omitted from public responses.
  - Encoding and privacy protection for entity keys and mutation fingerprints remain open under Q-124.
---

# ADR-0055: Derived Elasticsearch fence metadata

## Status

Accepted on 2026-09-25. Owner: Project maintainer.

## Context

MongoDB is authoritative for source fences and engine control state. Elasticsearch
scripts still need enough local state to atomically reject a stale mutation
against the target document. A projection can combine a root and independently
updated related entities, so one root-level timestamp cannot protect every
contributor. Equal-revision handling also needs to distinguish an identical
redelivery from a different mutation at the same revision ([ADR-0054](0054-equal-source-revision-conflicts.md)).

## Decision

- Reserve a root `engine_meta` field for engine-owned metadata. It is not part
  of the consumer-visible projection and must be excluded from public API
  responses.
- Use this derived document shape:

  ```json
  {
    "engine_meta": {
      "version": 1,
      "fences": [
        {
          "entity_key": "<scoped canonical entity key>",
          "source_ts_ms": 1700000000000,
          "source_ord": 7,
          "deleted": false,
          "mutation_fingerprint": "<fingerprint>"
        }
      ]
    }
  }
  ```

- Store one fence entry for each source entity that can independently
  contribute to or delete content in that Elasticsearch document. The
  `entity_key` distinguishes the source name, replica set, database,
  collection, and typed canonical entity ID. The revision fields use the
  source tuple from ADR-0052. `deleted` preserves a derived tombstone marker.
  `mutation_fingerprint` lets local fencing distinguish identical redelivery
  from conflicting content at an equal revision under ADR-0054.
- Treat this metadata as a rebuildable compare guard, not an authority. MongoDB
  retains the authoritative fence and control state. Do not copy checkpoints,
  leases, retry state, DLQ payloads, or unrelated engine metadata into
  Elasticsearch.
- Use strict explicit mappings for `engine_meta`; projection manifests cannot
  declare consumer fields under this reserved namespace. Encoding and privacy
  protection for `entity_key` and `mutation_fingerprint` are not selected here
  and remain tracked by Q-124.

## Alternatives considered

1. **Store only one revision at the document root.** This is compact, but an
   event for one child could incorrectly fence another contributor, or an old
   root event could overwrite a newer child change.
2. **Store only the revision tuple in each fence.** This is smaller, but cannot
   tell identical redelivery from conflicting content at the same revision as
   required by ADR-0054.
3. **Copy full MongoDB engine metadata into Elasticsearch.** This simplifies
   some lookups but duplicates authoritative state and increases data exposure
   and document size without helping the local conditional write.
4. **Keep a minimal versioned fence copy per contributor.** This supports local
   atomic comparisons and duplicate/conflict detection while leaving MongoDB
   authoritative. It increases document size with contributor count and
   requires key/fingerprint privacy rules. This is accepted.

## Consequences

- Each local scripted write can compare the incoming event to the fence for
  exactly the contributing entity it changes.
- Deletion markers and equal-revision mutation fingerprints travel with the
  derived target state and can be rebuilt from authoritative MongoDB metadata.
- A document with many independently fenced contributors carries more metadata;
  cardinality limits and nested-document capacity remain governed by Q-012 and
  capacity evidence.
- Anyone with direct Elasticsearch access may be able to inspect private fence
  metadata unless access controls and the Q-124 encoding decision prevent it.
- Deleting `engine_meta` does not erase the authoritative MongoDB fence, but
  the target must be rebuilt or have the derived metadata restored before safe
  writes resume.

## Validation

- A root update compares against the root entity fence; a child update compares
  against that child's fence and does not use another contributor's revision.
- A lower `(source_ts_ms, source_ord)` cannot overwrite a newer fence for the
  same `entity_key`.
- At an equal revision, the fingerprint supports the duplicate/conflict
  outcomes in ADR-0054; no Kafka offset or arrival-order tie-break is used.
- A deletion sets the derived `deleted` marker, preventing an older event from
  restoring the entity through the local write path.
- Removing the derived metadata does not remove MongoDB's authority, and public
  responses omit the whole `engine_meta` field.

## Review triggers

Revisit if target scripts cannot safely update the fence array atomically,
contributor cardinality makes the metadata too large, or a target backend offers
a different safe conditional-write mechanism.

## Related decisions and concepts

- [ADR-0052: MongoDB source-ordering scope](0052-mongodb-source-ordering-scope.md)
- [ADR-0054: Equal source revision conflicts](0054-equal-source-revision-conflicts.md)
- [Projection schema](../02-contracts/projection-schema.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Follow-up register](../follow-ups.md) (Q-011, Q-012, Q-124)

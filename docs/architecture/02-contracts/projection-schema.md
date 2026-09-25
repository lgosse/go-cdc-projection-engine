---
type: Architecture Review Topic
title: Projection schema
description: Defines ownership and limits of the Elasticsearch document shape.
tags: [contracts, elasticsearch, schema, mappings]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0009
---

# Projection schema

## Decision

The manifest owns the logical projection schema and explicit Elasticsearch
mapping/settings for each schema version. Physical indices are versioned and
read/write aliases control which version is active.

Production mappings default to rejecting undeclared fields. Dynamic mapping
requires an explicit manifest opt-in.

Root fields and projected relationship fields form the consumer-visible
projection. Engine metadata is never part of the public API contract. MongoDB
metadata remains authoritative for source fences, reconciliation state, and
other engine control data. For atomic local fencing, Elasticsearch stores a
rebuildable root `engine_meta` object with a version and per-contributing-entity
fences: scoped entity key, `source_ts_ms`, `source_ord`, `deleted`, and
`mutation_fingerprint` ([ADR-0055](../decisions/0055-derived-elasticsearch-fence-metadata.md)).
The entity key and mutation fingerprint are domain-separated HMAC-SHA-256
tokens, encoded as unpadded Base64URL. Each physical target pins a token-format
version and nonsecret key ID in `engine_meta`; raw source identifiers and HMAC
keys are never stored in Elasticsearch
([ADR-0064](../decisions/0064-fence-token-encoding-and-protection.md)). These
deterministic tokens remain pseudonymous and linkable within their key scope.
Public APIs omit this reserved field.

Lifecycle semantics are independent of physical storage. A separately
identified owned child or independent entity may be represented as a bounded
Elasticsearch `nested` member or in a separate index according to the relation
and capacity contracts. Snapshot data may be an ordinary embedded object;
query requirements can still justify nested mapping semantics. Every nested
relation declares a maximum expected cardinality. High-cardinality relations
require a separate follow-up design rather than silently becoming unbounded
nested arrays. Lifecycle roles are defined by
[ADR-0059](../decisions/0059-relation-lifecycle-classification.md); physical
capacity choices follow [ADR-0056](../decisions/0056-nested-relation-capacity-boundaries.md).

## Pros

- Prevents accidental field-type drift.
- Engine metadata supports deterministic writes and audits.
- Bounded nested data makes capacity risks reviewable.

## Cons and risks

- Strict mappings increase the operational cost of new fields.
- Hidden metadata increases document and update size.
- Elasticsearch nested documents can be expensive even below hard limits.

## Conditions and boundaries

- `engine_meta` is reserved for engine-owned fields; manifests may not declare
  consumer fields in this namespace. Entity-key and mutation-fingerprint token
  rules follow [ADR-0064](../decisions/0064-fence-token-encoding-and-protection.md);
  legal retention treatment remains deferred under Q-076.
- Projection authors declare expected child cardinality and choose nested or
  separately indexed storage. Expected-bound overruns alert but do not reject
  valid data inside the engine's benchmarked hard envelope. Bootstrap blocks
  cutover and live writes block the smallest safe scope only at that hard
  envelope ([ADR-0056](../decisions/0056-nested-relation-capacity-boundaries.md));
  numeric limits remain evidence-gated by Q-070.
- The public API must omit engine metadata even when an Elasticsearch document
  stores a derived copy.
- Cardinality and document-size thresholds for separate child indices require
  measured capacity evidence.
- Compatibility rules for manifest versions and consumer-visible field changes
  remain part of manifest and migration decisions.

## Validation

- The same manifest produces compatible mappings in live, bootstrap, and
  migration modes.
- Undeclared fields fail validation or ingestion unless dynamic behavior was
  explicitly opted into.
- Nested child queries preserve per-member field correlation.
- Elasticsearch metadata loss does not remove authoritative MongoDB fencing or
  reconciliation state.
- Local writes compare the event against the fence entry for the same scoped
  entity; an equal revision follows ADR-0054's duplicate/conflict rule.
- Public responses and documented consumer fields exclude engine metadata.
- An expected-cardinality overrun alerts while safe processing continues; a
  hard-envelope overrun blocks cutover or live mutation without dropping valid
  source data.

## Review trigger

Revisit if consumer compatibility requires relaxed mappings, if Elasticsearch
document or nested-query limits are approached, or if local fencing cannot be
implemented safely with a derived metadata copy.

## Related concepts

- [Manifest contract](manifest-contract.md)
- [Relationship model](relationship-model.md)
- [Identity, time, and ordering](identity-time-ordering.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Transformation contract](transformation-contract.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Blue/green migration](../04-data-lifecycle/blue-green-migration.md)

## Follow-up questions

- **Resolved by [ADR-0055](../decisions/0055-derived-elasticsearch-fence-metadata.md):** What exact metadata namespace and derived-copy fields are required for local fencing? Reserve root `engine_meta` with a version and per-contributor entries for scoped entity key, source revision, deletion marker, and mutation fingerprint; MongoDB remains authoritative.
- **Resolved by [ADR-0056](../decisions/0056-nested-relation-capacity-boundaries.md):** When should large child sets become separate indices instead of nested arrays? The author chooses the model; move beyond the tested hard envelope for cardinality, document size, or required performance. Numeric limits remain evidence-gated by Q-070.
- **Resolved by [ADR-0064](../decisions/0064-fence-token-encoding-and-protection.md):** How should the entity key and mutation fingerprint be encoded and protected in derived Elasticsearch fence metadata?
  Use domain-separated HMAC-SHA-256 with dedicated managed key material and
  fixed Base64URL encoding; each target pins its key ID and format version.
  HMAC tokens remain pseudonymous, and Q-076 governs legal retention.

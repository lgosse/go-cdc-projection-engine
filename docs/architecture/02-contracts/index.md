# Contracts and model

Stamp every contract that makes configuration and event processing deterministic.

- [Manifest contract](manifest-contract.md) - Versioning, validation, and
  expressiveness of projection declarations; versioned semantic manifests,
  layered preflight validation, and multi-projection event fan-out are accepted
  in [ADR-0040](../decisions/0040-manifest-contract.md).
- [CDC event envelope](cdc-event-envelope.md) - Required upstream event data;
  v1 applies only `c`/`u`/`d`, DLQs `r` and unknown operations, and uses engine
  bootstrap rather than Debezium startup data snapshots
  ([ADR-0047](../decisions/0047-debezium-operation-and-snapshot-scope.md));
  identity uses document-image-first precedence with Kafka-key fallback
  ([ADR-0005](../decisions/0005-identity-and-ordering-scope.md)); oversized
  and unbufferable events have bounded diagnostics and durable custody rules
  ([ADR-0048](../decisions/0048-event-size-and-buffer-diagnostics.md)).
  Transaction metadata is optional diagnostic context in v1 and does not define
  freshness or transaction grouping ([ADR-0049](../decisions/0049-transaction-metadata-treatment.md)).
- [Identity, time, and ordering](identity-time-ordering.md) - Canonical IDs and
  type-tagged, length-framed serialization ([ADR-0053](../decisions/0053-canonical-source-id-serialization.md)); MongoDB freshness uses `source.ts_ms` + `source.ord` within
  one source/replica-set/entity scope ([ADR-0052](../decisions/0052-mongodb-source-ordering-scope.md)); equal-revision conflicts go to entity-local repair ([ADR-0054](../decisions/0054-equal-source-revision-conflicts.md)); production verification remains a gate (Q-123).
- [Relationship model](relationship-model.md) - Root, child, reference, and
  multi-hop dependency semantics; v1 accepts rooted acyclic expansion. Each
  relation has an explicit snapshot, owned-child, or independent-entity
  lifecycle role separate from storage shape
  ([ADR-0059](../decisions/0059-relation-lifecycle-classification.md)).
  Referenced-entity changes trigger reverse-indexed root recomputation
  ([ADR-0057](../decisions/0057-reference-change-propagation.md)); high fan-out
  uses benchmark-derived per-relation deferred-recomputation thresholds
  ([ADR-0058](../decisions/0058-reference-fanout-execution-threshold.md)). V1
  invalidates at entity/relation granularity, including unused-field changes
  ([ADR-0061](../decisions/0061-projection-dependency-invalidation.md));
  independent-reference deletes recompute dependent roots and remove optional
  copied fields or apply explicit safe missing-value rules without cascading
  ([ADR-0063](../decisions/0063-independent-reference-delete-effects.md)).
- [Transformation contract](transformation-contract.md) - Where and how derived
  values are evaluated; Bloblang-in-Go uses a deterministic, versioned
  data-only allowlist with measured resource budgets
  ([ADR-0060](../decisions/0060-bloblang-subset-and-resource-budgets.md)).
  Dependency changes recompute roots through that declared materialized graph
  ([ADR-0061](../decisions/0061-projection-dependency-invalidation.md)); each
  physical target pins its manifest hash, transformation version, and digest,
  with semantic changes handled through blue-green migration
  ([ADR-0062](../decisions/0062-transformation-version-target-association.md)).
- [Deletion and replay](deletion-and-replay.md) - Tombstones, removals, and
  resurrection prevention; durable root and child fencing is accepted for v1,
  with a 30-day retention boundary and explicit expiry risk
  ([ADR-0050](../decisions/0050-deletion-fence-retention-policy.md)); rebuilds
  require complete source-bounded child enumeration and a safe CDC handoff
  ([ADR-0051](../decisions/0051-child-rebuild-from-source-snapshots.md)); a
  deleted independent reference recomputes dependent roots, removes optional
  copied fields or applies explicit safe missing-value behavior, and does not
  cascade ([ADR-0063](../decisions/0063-independent-reference-delete-effects.md)).
- [Projection schema](projection-schema.md) - Elasticsearch document and mapping
  ownership; explicit versioned mappings and Mongo-authoritative metadata are
  accepted for v1, with private per-entity derived fences under `engine_meta`
  ([ADR-0055](../decisions/0055-derived-elasticsearch-fence-metadata.md));
  entity keys and mutation fingerprints use domain-separated HMAC-SHA-256
  tokens with target-pinned key IDs and fixed encoding
  ([ADR-0064](../decisions/0064-fence-token-encoding-and-protection.md));
  author-declared expected cardinality warns while benchmarked hard caps block
  unsafe work ([ADR-0056](../decisions/0056-nested-relation-capacity-boundaries.md)).

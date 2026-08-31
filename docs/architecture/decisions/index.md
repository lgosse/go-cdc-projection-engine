# Decision register

Accepted decisions:

- [ADR-0001: Live ingestion source](0001-live-ingestion-source.md) - Kafka is
  the engine's sole live CDC input.
- [ADR-0002: Redis authority boundary](0002-redis-authority-boundary.md) - Redis
  is non-authoritative for projector progress and control-plane state.
- [ADR-0003: Durable state ownership](0003-durable-state-ownership.md) - Kafka
  owns cursors and engine-owned MongoDB owns control-plane and DLQ state.
- [ADR-0004: Debezium envelope boundary](0004-debezium-envelope-boundary.md) -
  Existing Debezium records are consumed as-is and unprocessable records go to
  the durable DLQ.
- [ADR-0005: Identity and ordering scope](0005-identity-and-ordering-scope.md) -
  Kafka tracks delivery while source-scoped metadata fences entity freshness.
- [ADR-0006: Relationship expansion scope](0006-relationship-expansion-scope.md) -
  V1 uses rooted acyclic expansion graphs while allowing scalar cross-references.
- [ADR-0007: Transformation execution boundary](0007-transformation-execution-boundary.md) -
  Bloblang-in-Go is authoritative and Painless is limited to fenced mutations.
- [ADR-0008: Deletion and replay semantics](0008-deletion-and-replay-semantics.md) -
  Deletes are first-class fenced events with durable root and child deletion
  custody.
- [ADR-0009: Projection schema ownership](0009-projection-schema-ownership.md) -
  Manifests own explicit versioned mappings while MongoDB remains authoritative
  for engine metadata.
- [ADR-0010: Elasticsearch write semantics](0010-elasticsearch-write-semantics.md) -
  Bulk mutations are versioned, source-fenced, idempotent, and explicitly
  complete across pinned dual-write targets.
- [ADR-0011: Kafka offset and delivery semantics](0011-kafka-offset-and-delivery-semantics.md) -
  Commits use at-least-once, highest-contiguous-completion semantics with
  durable terminal dispositions.
- [ADR-0012: Backpressure, retry, and DLQ policy](0012-backpressure-retry-dlq-policy.md) -
  Bounded queues, mode-specific retries, scoped blocking, and protected DLQ
  custody contain failures without data loss.
- [ADR-0013: Stream pipeline context resolution](0013-stream-pipeline-context-resolution.md) -
  Bounded pending, controlled source fallback, and deterministic coalescing
  resolve live events without indefinite waits.

- [ADR template](template.md) - Copy only after a concept has been reviewed and
  stamped.

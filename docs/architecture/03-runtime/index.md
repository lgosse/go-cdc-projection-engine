# Runtime processing

Stamp the live service behavior from startup through offset advancement.

- [Runtime topology](runtime-topology.md) - Process, worker, and projection
  isolation model.
- [Startup and readiness](startup-and-readiness.md) - Validation, dependency
  checks, and safe partial operation.
- [Stream pipeline](stream-pipeline.md) - Resolution, coalescing, transformation,
  and write flow.
- [Cache and reverse lookups](cache-and-reverse-lookups.md) - Population,
  invalidation, misses, and recovery.
- [Elasticsearch writes](elasticsearch-writes.md) - Scripted updates, dual writes,
  and partial bulk results; fenced idempotent semantics are accepted for v1.
- [Offsets and delivery](offsets-and-delivery.md) - Kafka commit semantics and
  crash recovery; at-least-once contiguous completion is accepted for v1.
- [Backpressure, retry, and DLQ](backpressure-retry-dlq.md) - Failure
  classification and containment; bounded mode-specific retries and scoped
  blocking are accepted for v1.

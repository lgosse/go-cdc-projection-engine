# Runtime processing

Stamp the live service behavior from startup through offset advancement.

- [Runtime topology](runtime-topology.md) - Process, worker, and projection
  isolation model; workload-class stream groups, bounded per-projection work,
  and partition-contiguous completion are accepted in
  [ADR-0041](../decisions/0041-runtime-topology.md).
- [Startup and readiness](startup-and-readiness.md) - Deterministic preflight,
  scoped blocking, and stable readiness during recoverable outages; accepted in
  [ADR-0042](../decisions/0042-startup-and-readiness.md).
- [Stream pipeline](stream-pipeline.md) - Resolution, coalescing, transformation,
  and write flow; bounded context resolution and coalescing are accepted for v1.
- [Cache and reverse lookups](cache-and-reverse-lookups.md) - Population,
  invalidation, misses, and recovery; relation-specific retention and
  generation rebuilds are accepted for v1.
- [Elasticsearch writes](elasticsearch-writes.md) - Scripted updates, dual writes,
  and partial bulk results; fenced idempotent semantics are accepted for v1.
- [Offsets and delivery](offsets-and-delivery.md) - Kafka commit semantics and
  crash recovery; at-least-once contiguous completion is accepted for v1.
- [Backpressure, retry, and DLQ](backpressure-retry-dlq.md) - Failure
  classification and containment; bounded mode-specific retries and scoped
  blocking are accepted for v1.

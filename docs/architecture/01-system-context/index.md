# System context

Stamp the engine's purpose and promises before selecting mechanisms.

- [Objectives and boundaries](objectives-and-boundaries.md) - Intended outcomes,
  actors, dependencies, and non-goals.
- [Live ingestion source](live-ingestion-source.md) - Accepted Kafka-only live
  input boundary.
- [Redis authority boundary](redis-authority-boundary.md) - Accepted
  non-authoritative cache boundary.
- [Durable state ownership](durable-state-ownership.md) - Accepted ownership
  matrix for offsets, control-plane state, and DLQ custody.
- [System guarantees](system-guarantees.md) - The semantics the engine promises
  to consumers and operators.
- [Data-store responsibilities](data-store-responsibilities.md) - Canonical role
  and durability expectations for Kafka, MongoDB, Redis, and Elasticsearch.
- [Domain language](domain-language.md) - Shared terminology for later reviews.

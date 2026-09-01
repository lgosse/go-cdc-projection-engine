# System context

Stamp the engine's purpose and promises before selecting mechanisms.

- [Objectives and boundaries](objectives-and-boundaries.md) - Intended outcomes,
  actors, dependencies, and non-goals; the engine's ownership boundary,
  allow-listed source fallback, and evidence-gated first-release scope are
  accepted in [ADR-0037](../decisions/0037-objectives-and-boundaries.md).
- [Live ingestion source](live-ingestion-source.md) - Accepted Kafka-only live
  input boundary.
- [Redis authority boundary](redis-authority-boundary.md) - Accepted
  non-authoritative cache boundary.
- [Durable state ownership](durable-state-ownership.md) - Accepted ownership
  matrix for offsets, control-plane state, and DLQ custody.
- [System guarantees](system-guarantees.md) - The semantics the engine promises
  to consumers and operators; bounded at-least-once delivery, conditional
  convergence, measurable freshness, verified migration continuity, and scoped
  exceptions are accepted in
  [ADR-0038](../decisions/0038-system-guarantees.md).
- [Data-store responsibilities](data-store-responsibilities.md) - Canonical role
  and durability expectations for Kafka, MongoDB, Redis, and Elasticsearch;
  the authority matrix, disagreement rules, and reverse-index classification are
  accepted in [ADR-0036](../decisions/0036-data-store-responsibilities.md).
- [Domain language](domain-language.md) - Shared terminology for later reviews;
  canonical terms distinguishing business records, CDC transport, logical
  projections, physical targets, and engine metadata are accepted in
  [ADR-0039](../decisions/0039-domain-language.md).

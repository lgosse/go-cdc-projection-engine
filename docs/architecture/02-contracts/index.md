# Contracts and model

Stamp every contract that makes configuration and event processing deterministic.

- [Manifest contract](manifest-contract.md) - Versioning, validation, and
  expressiveness of projection declarations.
- [CDC event envelope](cdc-event-envelope.md) - Required upstream event data.
- [Identity, time, and ordering](identity-time-ordering.md) - Canonical IDs and
  comparison rules.
- [Relationship model](relationship-model.md) - Root, child, reference, and
  multi-hop dependency semantics.
- [Transformation contract](transformation-contract.md) - Where and how derived
  values are evaluated.
- [Deletion and replay](deletion-and-replay.md) - Tombstones, removals, and
  resurrection prevention.
- [Projection schema](projection-schema.md) - Elasticsearch document and mapping
  ownership.

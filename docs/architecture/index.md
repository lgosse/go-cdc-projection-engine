---
okf_version: "0.2"
---

# Architecture review workspace

This Open Knowledge Format bundle decomposes the two source drafts into small,
linked decisions. Every proposal remains provisional until its concept is
reviewed and stamped.

## Start here

- [Review method](review-method.md) - Status vocabulary, stamping criteria, and
  the template used by every architecture concept.
- [Decision register](decisions/index.md) - Accepted decisions and the ADR
  template; currently empty by design.
- [Source conflicts](source-conflicts.md) - Contradictions that must be resolved
  before implementation.

## Architecture sections

- [System context](01-system-context/) - Purpose, boundaries, guarantees, and
  shared language.
- [Contracts and model](02-contracts/) - Manifests, events, identity, relations,
  transformations, and deletion semantics.
- [Runtime processing](03-runtime/) - Service topology, streaming, caches,
  Elasticsearch writes, offsets, and failure handling.
- [Data lifecycle](04-data-lifecycle/) - Bootstrap, schema evolution,
  migration, reconciliation, and recovery.
- [Quality attributes](05-quality-attributes/) - Capacity, availability,
  security, privacy, and compatibility.
- [Observability](06-observability/) - Telemetry conventions, signals, health,
  and alerting.
- [Operations](07-operations/) - Modes, configuration, deployment, runbooks,
  and ownership.
- [Validation](08-validation/) - Invariants, tests, failure exercises, and
  acceptance gates.

## Source drafts

- [System requirements and implementation draft](../design/system.md)
- [OpenTelemetry observability draft](../design/otel.md)

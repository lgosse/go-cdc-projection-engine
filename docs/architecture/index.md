---
okf_version: "0.2"
---

# Architecture review workspace

This Open Knowledge Format bundle decomposes the two source drafts into small,
linked decisions. Every proposal remains provisional until its concept is
reviewed and stamped.

The review workflow is encoded in the repository-local
[architecture decision review skill](../../.skills/architecture-decision-review/SKILL.md).

## Start here

- [Review method](review-method.md) - Status vocabulary, stamping criteria, and
  the accepted process used by every architecture concept; see [ADR-0044](decisions/0044-architecture-review-process.md).
- [Decision register](decisions/index.md) - Accepted decisions and the ADR
  template.
- [Implementation roadmap](../implementation-plan.md) - Gated path from the
  accepted architecture baseline to a first production release.
- [Follow-up register](follow-ups.md) - Questions that must be resolved or
  explicitly deferred during implementation.
- [Source conflicts](source-conflicts.md) - Source-draft contradictions and
  their accepted reconciliation in [ADR-0043](decisions/0043-source-draft-conflict-resolution.md).

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

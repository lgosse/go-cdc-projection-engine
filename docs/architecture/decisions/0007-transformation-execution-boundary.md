---
type: Architecture Decision Record
title: "ADR-0007: Transformation execution boundary"
description: Bloblang in Go is authoritative and Painless is limited to fenced mutations.
tags: [architecture, adr, transformations, bloblang, elasticsearch]
status: accepted
decision_id: ADR-0007
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Bloblang execution is deterministic and resource-bounded.
  - Incomplete context is deferred or repaired unless explicitly allowed by the manifest.
  - Malformed or impossible events follow the durable DLQ policy.
---

# ADR-0007: Transformation execution boundary

## Context

The source drafts execute Bloblang in Go for stream, bootstrap, and audit paths,
but also propose compiling Painless and investigating a Bloblang-to-Painless
translation boundary. Maintaining two implementations would allow null, type,
array, date, and error semantics to diverge between live and batch processing.

## Decision

Bloblang in Go is the authoritative transformation runtime. Compile a restricted,
resource-bounded subset before consumption and use the same evaluator in stream,
bootstrap, audit, repair, and migration modes.

Evaluate transformations against a canonical assembled projection
representation. Elasticsearch Painless is limited to deterministic, idempotent,
source-fenced mutations such as skeleton upserts, nested-member updates, and
stale-write suppression. Painless does not implement business derivations or a
second transformation language.

Do not publish derived values from incomplete context unless the manifest
explicitly declares that derivation safe with missing fields. Defer or repair
valid events lacking enough context. Malformed or impossible events follow the
accepted durable-DLQ policy.

## Conditions and boundaries

- The exact Bloblang subset and resource limits are follow-up decisions.
- Context assembly and pending-event lifecycle are follow-up runtime decisions.
- Transformation versioning and its relationship to index migration remain open.
- This ADR does not define the exact Painless mutation script shape.

## Alternatives considered

1. Translate Bloblang into Painless for live updates. This could use the current
   Elasticsearch document as context but risks semantic drift and a large,
   incomplete compiler.
2. Use separate Bloblang and Painless semantics by mode. This minimizes initial
   work but makes bootstrap and live projections disagree over time.

## Consequences

- One evaluator and test corpus define derived-field semantics across all modes.
- Stream processing must obtain sufficient context before evaluating dependent
  fields or defer/repair the event.
- Elasticsearch remains responsible for mutation atomicity and fencing, not
  domain calculation.
- Transformation CPU, memory, timeout, and pending-work budgets become
  observable operational concerns.

## Validation

- Run a representative transformation corpus through every operating mode and
  compare canonical outputs.
- Verify Painless mutations do not reimplement derivation logic.
- Test partial child events with missing context and confirm defer/repair rather
  than incorrect derived values.
- Test malformed and impossible transformation inputs reach durable DLQ custody.
- Benchmark resource limits and verify diagnostics identify limit violations.

## Review trigger

Revisit if one evaluator cannot meet stream latency or bootstrap throughput,
if a required transformation cannot be expressed safely in the allowed subset,
or if a product requirement demands remote Elasticsearch-side derivation.

## Related concepts

- [Transformation contract](../02-contracts/transformation-contract.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)

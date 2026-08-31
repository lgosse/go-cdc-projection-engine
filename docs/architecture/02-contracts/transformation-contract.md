---
type: Architecture Review Topic
title: Transformation contract
description: Defines supported derivations and consistent execution across stream and batch modes.
tags: [contracts, transformations, bloblang, correctness]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0007
conditions:
  - Bloblang execution is deterministic and resource-bounded.
  - Incomplete context is deferred or repaired unless explicitly allowed by the manifest.
  - Malformed or impossible events follow the durable DLQ policy.
---

# Transformation contract

## Decision

Bloblang in Go is the authoritative transformation runtime. Elasticsearch
Painless applies only deterministic, idempotent, source-fenced mutations and
does not implement a second business-transformation language.

## Accepted boundary

- Compile a restricted, resource-bounded Bloblang subset before consumption.
- Use the same evaluator in stream, bootstrap, audit, repair, and migration
  modes.
- Evaluate against a canonical assembled projection representation.
- Do not publish derived values from incomplete context unless the manifest
  explicitly declares the derivation safe with missing fields.
- Defer or repair valid events that lack enough context; classify malformed or
  impossible events through the accepted durable-DLQ policy.
- Defer the exact Bloblang subset, resource limits, context-assembly strategy,
  and pending-event lifecycle to follow-up decisions.

## Pros

- Stream, bootstrap, audit, repair, and migration share one transformation
  implementation.
- Avoids a difficult and potentially incomplete Bloblang-to-Painless compiler.
- Startup compilation catches syntax errors before consumption.
- Keeps Elasticsearch focused on fenced mutation mechanics.

## Cons and risks

- A partial child event may not contain enough context to re-evaluate a whole
  projection and must be deferred or repaired.
- Complex scripts can undermine manifest safety and performance.
- Canonical assembly in stream mode may require more cached state than drafted.
- Resource limits and pending work add operational states to observe and own.

## Questions to stamp

- What exact Bloblang operations and resource limits are allowed?
- Which dependency changes trigger recomputation?
- How are transformation versions associated with projected documents?

## Related concepts

- [Manifest contract](manifest-contract.md)
- [Relationship model](relationship-model.md)
- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)

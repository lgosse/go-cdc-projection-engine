---
type: Architecture Decision Record
title: "ADR-0012: Backpressure, retry, and DLQ policy"
description: Bounded queues, mode-specific retries, scoped blocking, and protected DLQ custody contain failures without data loss.
tags: [architecture, adr, backpressure, retry, dlq, resilience]
status: accepted
decision_id: ADR-0012
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Queue limits bound records, bytes, and age, and Kafka intake pauses before hard capacity is reached.
  - Transient failures use mode-specific bounded retry budgets and do not become poison-event DLQ records.
  - Event-local deterministic failures use protected, configurable DLQ custody in engine-owned MongoDB.
  - Blocking scope escalates from event to projection, partition, workload, or process according to failure scope.
---

# ADR-0012: Backpressure, retry, and DLQ policy

## Context

The accepted offset and Elasticsearch decisions require durable terminal custody
before Kafka progress advances. The source draft proposes bounded retries and a
DLQ but also suggests panicking after exhausted transient retries. That would
turn dependency outages into crash loops and make valid events look like bad
data.

## Decision

Apply bounded per-workload buffers measured by records, bytes, and queue age.
Pause Kafka intake before a hard capacity limit is reached; never silently drop
records. Bootstrap and replay workers use equivalent bounded work queues and
checkpoints appropriate to their mode.

Classify failures by whether the event can succeed and by the scope of the
affected dependency:

- Event-local deterministic failures—malformed envelopes, missing identity,
  invalid transformations, mapping conflicts, and impossible records—go to the
  durable DLQ and become terminal only after custody succeeds.
- Transient dependency failures—timeouts, throttling, temporary Elasticsearch or
  Redis unavailability, and network errors—use jittered exponential backoff and
  a bounded elapsed-time budget. Exhausting that budget pauses or blocks the
  affected work; it does not turn valid events into poison records.
- Configuration or projection-wide failures—missing indices, incompatible
  mappings, invalid manifests, or unavailable shared dependencies—block the
  smallest affected projection, partition, or workload rather than creating a
  DLQ record for every event.
- Corrupted invariants or unrecoverable process state may terminate the process;
  ordinary dependency outages must not create a crash loop.

Retry budgets differ by mode: live streaming prioritizes bounded latency,
bootstrap prioritizes bounded throughput, and replay may allow a longer recovery
window. Exact durations and queue capacities are operational follow-ups.

DLQ custody remains in engine-owned MongoDB. Store the original envelope or a
recoverable protected payload reference together with source coordinates,
manifest and projection versions, failure classification, target outcomes,
diagnostics, and retry history. Apply encryption/access control, configurable
retention, and redaction or secure references where source-data policy requires
them. Correctness must not depend on expired DLQ data.

## Alternatives considered

1. Retry a fixed number of times and then panic the process. This is simple but
   creates crash loops during dependency outages and misclassifies valid events.
2. Send every exhausted retry to the DLQ. This preserves offsets but floods the
   DLQ during cluster-wide incidents and makes replay diagnosis harder.
3. Use unbounded in-memory queues and rely on pod restarts for recovery. This
   risks memory exhaustion, duplicate storms, and loss of controlled backpressure.

## Consequences

- Memory growth and retry storms are bounded before they threaten process
  stability.
- Bad records are isolated and replayable without treating dependency outages as
  data corruption.
- A prolonged outage can intentionally pause a partition, projection, or
  workload and requires alerting and operator recovery.
- DLQ storage carries privacy, retention, indexing, and capacity obligations.
- Mode-specific budgets require separate operational measurements and tuning.

## Validation

- Queue limits pause intake before configured record, byte, or age thresholds
  exhaust process capacity.
- Transient outages retry with jitter and do not create crash loops or DLQ floods.
- Deterministic event failures reach protected durable DLQ custody with complete
  diagnostics before their offsets become terminal.
- DLQ or MongoDB outages never cause unsafe Kafka offset advancement.
- Projection-wide failures block the smallest correct scope without corrupting
  unrelated work.
- Live, bootstrap, and replay modes expose their retry, queue, pause, and resume
  states for testing and operations.

## Review triggers

Revisit if queue pressure or retry latency violates recovery objectives, if
failure classification repeatedly chooses the wrong scope, or if privacy and
retention requirements change the permitted DLQ payload.

## Related concepts

- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)

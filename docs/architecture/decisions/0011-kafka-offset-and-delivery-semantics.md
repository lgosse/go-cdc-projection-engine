---
type: Architecture Decision Record
title: "ADR-0011: Kafka offset and delivery semantics"
description: Kafka commits use at-least-once, highest-contiguous-completion semantics with durable terminal dispositions.
tags: [architecture, adr, kafka, offsets, delivery, dlq]
status: accepted
decision_id: ADR-0011
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Kafka commits use at-least-once, highest-contiguous-completion semantics per topic partition.
  - Durable DLQ acknowledgment is terminal completion; operator skips are distinct, authorized, and audited.
  - Rebalance draining uses a bounded configurable deadline and cannot commit after partition ownership is revoked.
---

# ADR-0011: Kafka offset and delivery semantics

## Context

Kafka delivery is at-least-once and is not transactionally coupled to
Elasticsearch or the engine-owned MongoDB metadata store. Batches can complete
out of order, Elasticsearch bulk requests can partially fail, and coalescing can
map several consumed records to one projection mutation.

## Decision

Guarantee at-least-once processing, not exactly-once processing across Kafka,
Elasticsearch, and MongoDB.

Track completion independently for every consumed offset within each topic
partition. Commit only the highest contiguous completed offset. A record is
complete only after all required Elasticsearch targets succeed or durable
terminal custody is recorded in engine-owned MongoDB.

Coalescing may combine events across partitions, but completion is recorded for
every contributing offset only after the combined mutation has a definitive
outcome. A failed combined mutation leaves all contributing offsets incomplete.

A durable DLQ acknowledgment is sufficient for terminal completion. A crash
between the MongoDB write and Kafka commit may repeat the DLQ attempt, so DLQ
custody uses both the topic/partition/offset delivery coordinate and a canonical
payload/event hash for deduplication and diagnostics.

Operator skips are distinct from ordinary DLQ records. They require explicit
authorization, durable audit evidence, and the same contiguous-completion
handling as any other terminal disposition.

During rebalance, stop fetching the revoked partition and drain in-flight work
within a bounded, configurable deadline. Commit only completed contiguous
offsets while ownership is still valid; after revocation, the previous owner
must not commit that partition.

## Alternatives considered

1. Commit the highest offset in each flushed batch. This is simple and fast but
   can permanently skip an earlier failed record.
2. Commit every record synchronously. This makes completion obvious but reduces
   throughput and still requires idempotent recovery across the non-transactional
   sinks.
3. Claim exactly-once behavior through Kafka transactions. Kafka transactions do
   not make Elasticsearch and MongoDB writes atomic, so this would overstate the
   guarantee.

## Consequences

- A blocked record holds later offsets in its partition until success, durable
  DLQ custody, or an authorized operator skip.
- Crash recovery safely replays uncommitted work through idempotent sink
  operations.
- Coalescing requires an explicit mapping from one mutation outcome back to all
  consumed offsets that contributed to it.
- DLQ storage needs retention, unique keys, hashing, and audit controls.
- Rebalance handling trades some duplicate replay for bounded failover latency.

## Validation

- An unresolved offset prevents later offsets in the same partition from being
  committed.
- Crash after Elasticsearch success but before Kafka commit safely replays the
  mutation.
- Crash after DLQ persistence but before Kafka commit does not create unbounded
  duplicate DLQ records.
- Coalesced cross-partition mutations map completion back to every contributing
  offset.
- Rebalance does not allow the previous owner to commit after partition
  revocation.
- Operator skips are authorized, audited, distinct from poison-event DLQ, and
  replay-safe.

## Review triggers

Revisit if partition blocking violates recovery objectives, if coalescing cannot
reliably map outcomes to offsets, or if a future sink provides a genuine
cross-system transaction boundary.

## Related concepts

- [Offsets and delivery semantics](../03-runtime/offsets-and-delivery.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Stream pipeline](../03-runtime/stream-pipeline.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)

---
type: Architecture Decision Record
title: "ADR-0047: Debezium operation and snapshot scope"
description: Defines the v1 Debezium operations and snapshot behavior accepted by the projection engine.
tags: [architecture, adr, cdc, debezium, snapshot, contracts]
status: accepted
decision_id: ADR-0047
accepted_on: 2026-09-24
owner: Project maintainer
conditions:
  - V1 applies only Debezium `c`, `u`, and `d` events through the live stream path.
  - Snapshot-read and unknown operation codes are unprocessable and require durable DLQ custody before Kafka offset advancement.
  - Existing data is populated through the engine bootstrap and CDC handoff, not Debezium startup data-snapshot rows.
---

# ADR-0047: Debezium operation and snapshot scope

## Status

Accepted on 2026-09-24. Owner: Project maintainer.

## Context

The first release needs one deterministic boundary between live CDC and
population of existing source data. Debezium MongoDB uses `c`, `u`, and `d` for
create, update, and delete operations; `r` represents a read event produced by
a snapshot. The source also includes a snapshot marker whose representation can
vary; the supplied example represents false as the string `"false"`.

The checked-in Terraform connector configuration sets `snapshot.mode: never`
for the configured dev, prod, and squads source connectors, and the Strimzi
Kafka Connect build pins the MongoDB connector plugin to `2.2.1.Final`. This is
declared configuration, not evidence of the configuration currently running
in any cluster. Debezium's current MongoDB documentation deprecates `never` in
favor of `no_data`; this ADR defines the engine's event contract and does not
require changing that connector setting. Any plugin/configuration upgrade must
check the supported setting against that pinned plugin version.

The engine already has a separate bootstrap scan and live CDC handoff in
[ADR-0015](0015-bootstrap-consistency-and-handoff.md). Allowing snapshot reads
into the normal stream path without choosing their fence and handoff semantics
would blur that boundary.

## Decision

- V1 applies only Debezium operation codes `c` (create), `u` (update), and `d`
  (delete) through the live stream path.
- An `r` snapshot-read event or an unknown operation code is unprocessable. It
  must not mutate a projection. The engine stores it in the durable DLQ and
  advances the Kafka offset only after DLQ custody succeeds.
- If `source.snapshot` is present, accept only JSON boolean `false` or string
  `"false"`. Any other present value is unsupported and makes the event
  unprocessable. The field may be absent on a live event.
- V1 does not consume Debezium startup data-snapshot rows. Populate existing
  data through the engine's separate bootstrap scan, live dual-write, and
  boundary-overlap replay defined by ADR-0015.
- The v1 connector profile must suppress startup data-snapshot rows. The
  checked-in Terraform configuration currently declares
  `snapshot.mode: never`; modes that emit startup `r` rows are outside the
  supported profile. The engine still rejects any `r` event it receives.
- Enforce the event rules even if connector configuration changes or an
  on-demand snapshot produces an `r` event. A connector setting is not an
  authorization to apply an operation the engine does not support.

Q-010, which asks whether snapshot-read events participate in normal fencing
or use a separate bootstrap path, is deferred. The safe interim behavior is to
DLQ `r` events without applying them. Revisit Q-010 before enabling any
connector or workflow that can emit snapshot-read events into an engine input
stream.

## Alternatives considered

1. **Apply `r` through the live stream path.** This could simplify a
   connector-driven first load, but it would mix snapshot rows with live CDC
   without a decided source boundary, ordering fence, or replay handoff. It is
   deferred until Q-010 is resolved.
2. **Use Debezium startup snapshots to populate v1 targets.** This would rely
   on connector snapshot behavior in place of the engine's source watermark,
   dual-write, overlap replay, and resumable scan contract. It is not selected
   for v1.

## Consequences

- The live apply path has a small explicit operation allow-list and malformed,
  unsupported, or snapshot-read records share the existing durable DLQ and
  offset-custody rule.
- V1 does not get automatic source-data repopulation from a Debezium startup
  snapshot. Existing data and recovery scans use the engine bootstrap,
  reconciliation, or repair lifecycle.
- Connector configuration can evolve independently only while its emitted
  records remain inside the engine contract; configuration changes do not
  relax the engine's fail-closed handling.
- Snapshot reads in the DLQ require an explicit later decision and replay path
  before they can be applied.

## Validation

- Contract and runtime tests accept `c`, `u`, and `d` with `source.snapshot`
  absent, boolean `false`, or string `"false"`.
- Tests prove `r`, unknown operations, and other present snapshot-marker values
  reach durable DLQ custody and do not advance offsets before custody succeeds.
- Bootstrap evidence proves an existing-data scan overlaps live CDC and replays
  through its source boundary before handoff, as required by ADR-0015.
- Deployment evidence verifies the effective connector configuration and
  plugin version for each environment; checked-in Terraform values alone do
  not establish runtime state.

## Review triggers

Revisit before enabling initial, on-demand, or other connector/workflow
snapshots that can emit `r` into an engine input stream; before changing the
v1 operation allow-list; or when upgrading the Debezium plugin or changing
snapshot configuration in a way that affects emitted events.

## Related concepts

- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Bootstrap consistency and handoff](0015-bootstrap-consistency-and-handoff.md)
- [Follow-up register](../follow-ups.md) (Q-001 and deferred Q-010)
- [Debezium MongoDB connector documentation](https://debezium.io/documentation/reference/connectors/mongodb.html)

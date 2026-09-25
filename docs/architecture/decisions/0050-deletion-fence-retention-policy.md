---
type: Architecture Decision Record
title: "ADR-0050: Deletion-fence retention policy"
description: Defines the supported deletion-fence lifetime and recovery behavior after it expires.
tags: [architecture, adr, deletion, replay, retention, recovery]
status: accepted
decision_id: ADR-0050
accepted_on: 2026-09-24
owner: Project maintainer
conditions:
  - Durable deletion fences are retained for at least 30 days after they are persisted.
  - Raw replay older than 30 days is unsupported; recovery outside the window rebuilds from current authoritative source state before writes resume.
  - The residual risk of an old event arriving after fence expiry and recreating deleted projection state is explicitly accepted.
  - DLQ, Kafka, and backup retention details remain separate operational follow-ups and must respect this supported replay boundary.
---

# ADR-0050: Deletion-fence retention policy

## Status

Accepted on 2026-09-24. Owner: Project maintainer.

## Context

[ADR-0008](0008-deletion-and-replay-semantics.md) established durable root and
child deletion fences, but left fence cleanup tied to the maximum supported
replay, retention, recovery, and reconciliation horizon. Proving one upper bound
across Kafka, DLQ, backups, and every recovery path would block cleanup until
those operational policies were all specified and evidenced.

The repository's Kafka IaC declares a short CDC-topic retention, but it does not
prove the effective retention of deployed topics, DLQ custody, or metadata
backups. The 30-day policy is therefore an explicit product risk boundary, not
a claim that infrastructure makes later delivery impossible.

## Decision

- Keep each durable deletion fence in engine-owned MongoDB for at least 30 days
  after the fence is durably recorded. Redis expiry or loss never shortens this
  period.
- Treat raw replay of events older than 30 days as unsupported. If recovery
  requires history outside that window, build a fresh projection from current
  authoritative source state and complete the source-boundary and validation
  steps before resuming writes. Do not resume by replaying the older raw history.
- Accept the residual risk that an old event may nevertheless arrive after its
  fence expires and recreate deleted projection state. The 30-day period is a
  chosen risk limit, not a proof that resurrection cannot happen.
- Preserve ADR-0008's identity, source-fencing, root/child delete, and Redis
  authority semantics. This ADR supersedes only its open fence-retention and
  cleanup condition and resolves Q-005.
- Keep the exact Kafka outage/recovery window, DLQ retention and replay rules,
  metadata backup/PITR evidence, and authorization for recovery with missing
  custody as separate follow-ups. Any supported replay path must stay within
  the 30-day boundary; older retained payloads may not be replayed as raw events.

## Alternatives considered

1. **Wait for an evidenced upper bound across every history path.** This gives
   stronger cleanup evidence, but requires Kafka, DLQ, backup, replay, and
   reconciliation horizons to be specified and monitored together before any
   fence can expire. It delays storage cleanup and couples this contract to
   several operational decisions.
2. **Never expire fences.** This avoids expiry-based resurrection, but makes
   fence storage grow indefinitely and retains identifiers indefinitely, which
   increases storage and privacy costs.
3. **Use a shorter or longer fixed lifetime.** This is simple to operate, but
   a shorter period raises the chance of an old event arriving after expiry;
   a longer period raises storage and identifier-retention costs. Thirty days
   is the accepted balance, with the remaining risk stated explicitly.

## Consequences

- Fence cleanup has a concrete minimum age and no longer depends on proving an
  unbounded collection of infrastructure horizons.
- Recovery outside the supported window is a current-source rebuild with a
  controlled handoff, not an attempt to replay old events against expired
  fences.
- The engine cannot claim that deletes are protected against arbitrarily old
  deliveries. Operators need to distinguish supported replay from an unsafe
  old-history replay.
- DLQ and backup policies may retain data longer than 30 days, but retention
  alone does not make raw replay after 30 days supported.
- Deletion-fence identifiers remain subject to the open legal-treatment
  follow-up.

## Validation

- A fence remains effective throughout the first 30 days after durable
  persistence, including Redis loss and restart.
- Cleanup cannot remove a fence before that minimum age.
- Replay tooling rejects or otherwise prevents raw replay older than 30 days.
- Recovery outside the window builds from current authoritative state and
  validates the new source boundary and projection before writes resume.
- DLQ, Kafka, backup, and runbook policies document how they honor the
  supported 30-day replay boundary.
- Tests and operational evidence explicitly record that delivery after fence
  expiry is outside the guarantee and can resurrect stale state.

## Review triggers

Revisit if any supported raw replay path can exceed 30 days, source rebuilds
cannot complete before writes resume, an observed late event resurrects data,
Kafka/DLQ/backup policy changes materially, or privacy requirements change the
acceptable fence-identifier retention.

## Related decisions and concepts

- [ADR-0008: Deletion and replay semantics](0008-deletion-and-replay-semantics.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)
- [Follow-up register](../follow-ups.md) (Q-005, Q-020, Q-029, Q-045, Q-046, Q-047, Q-076)

---
type: Architecture Review Topic
title: Runbooks and intervention
description: Defines the minimum operator procedures required before production use.
tags: [operations, runbooks, incident-response]
status: accepted
decision_id: ADR-0030
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Runbooks cover streaming, dependency, migration, reconciliation, DLQ, security, privacy, and disaster-recovery scenarios.
  - Every runbook uses a consistent diagnosis, action, stop-condition, rollback, evidence, and escalation structure.
  - Automatic behavior is limited to bounded safe actions; destructive or correctness-sensitive actions require explicit operator initiation or approval.
  - One authorized operator model is sufficient initially; privileged actions are attributable and audited, without mandatory two-person approval.
  - Live observability, durable MongoDB audit records, and the incident system together provide operational evidence and history.
---

# Runbooks and intervention

## Decision

Maintain an indexed runbook catalogue before production use. It covers Kafka lag
or freshness breaches, rebalances and hot partitions, MongoDB/Redis/
Elasticsearch outages, Redis loss and rebuild, blocked schema or capability
mismatch, partial dual-write failure, failed or stalled bootstrap, unsafe alias or
migration state, reconciliation drift, DLQ triage and replay, hot documents or
nested-object limits, credential rotation, privacy erasure/anonymization, and
disaster recovery or target rebuild.

Every runbook follows the same structure:

1. Trigger and customer impact.
2. Scope and authoritative state.
3. Required permissions and prerequisites.
4. Diagnosis commands and expected observations.
5. Safe automated behavior already performed by the engine.
6. Approved operator actions.
7. Stop conditions and escalation.
8. Rollback or recovery steps.
9. Evidence to capture.
10. Closure and follow-up actions.

The engine may automatically retry transient failures, apply backpressure,
pause a partition or projection, mark a migration degraded, restart after a
corrupted local invariant, and rebuild derived Redis state through the accepted
generation procedure. Operators must explicitly initiate or approve DLQ replay,
repairs, alias rollback or cutover, forced migration abort, target retirement,
privacy-erasure verification, disaster-recovery promotion, and any safety-fence
override.

Use role-based ownership even before concrete team names are assigned:

- the engine on-call owns projector behavior, offsets, fencing, retries, DLQ
  custody, and migrations;
- source-domain owners own CDC completeness, source schema, and anonymization;
- platform owners own Kafka, MongoDB, Redis, Elasticsearch, Kubernetes, and
  telemetry;
- projection owners own manifest semantics, search correctness, and business
  acceptance.

Use live metrics, traces, and structured logs for diagnosis; durable MongoDB
records for privileged actions, migration state, repairs, and DLQ operations;
and the organization incident/ticket system for chronology, decisions, and
follow-up. Link these layers with bounded run or operation identifiers and never
put raw payloads or sensitive identifiers in runbook evidence.

Runbooks are validated in a safe environment through failure-injection or
game-day exercises. A runbook is incomplete until its commands, expected alerts,
diagnostics, ownership, escalation, and recovery steps have been exercised.

## Pros

- Converts implicit failure behavior into repeatable, auditable procedures.
- Makes the boundary between automatic safety behavior and human intervention
  explicit.
- Provides evidence and acceptance material for incident and game-day exercises.
- Role-based ownership works before organizational names are finalized.

## Cons and risks

- Runbooks decay unless releases and incidents update them.
- A broad catalogue requires a clear index and consistent navigation.
- Redaction and bounded identifiers can make some investigations slower.
- Manual approval for correctness-sensitive actions adds response time.
- Failure-injection exercises consume operational capacity.

## Alternatives considered

1. Documentation-only procedures. Easy to create, but they can diverge from
   supported commands and actual failure behavior.
2. Fully automatic remediation. Reduces operator effort but risks hiding
   correctness failures or performing destructive actions without context.
3. One large operations manual. Centralized but difficult to navigate under
   incident pressure; scenario-specific runbooks with an index are preferred.
4. Team-specific undocumented procedures. Fast locally but inconsistent and
   unauditable in production.

## Consequences

- Production readiness includes runbook coverage and exercised recovery paths.
- Privileged operations need authenticated access, actor attribution, and
  durable audit records.
- Incident tooling must preserve links to metrics, traces, logs, and MongoDB
  operation records.
- Concrete team assignments and exercise cadence remain operational follow-ups.
- Safety-sensitive actions remain explicit rather than hidden in automation.

## Validation

- Each required scenario has an indexed runbook with tested commands and stop
  conditions.
- Automatic retries, pauses, degradation, and restarts match documented behavior.
- DLQ replay, repair, migration rollback, retirement, and erasure procedures are
  attributable and auditable.
- Runbook evidence contains no raw payloads, secrets, Restricted fields, or
  unprotected identifiers.
- Failure-injection or game-day exercises demonstrate recovery and escalation.
- Runbooks are reviewed when related code, manifests, dependencies, or policies
  change.

## Review trigger

Revisit if incident response repeatedly depends on undocumented actions, runbook
steps cannot be exercised safely, ownership boundaries change, or automation
needs to expand beyond the accepted safety boundary.

## Related concepts

- [Modes and configuration](modes-and-configuration.md)
- [Deployment and orchestration](deployment-and-orchestration.md)
- [Health and diagnostics](../06-observability/health-and-diagnostics.md)
- [Metrics and alerting](../06-observability/metrics-and-alerting.md)
- [Tracing and logging](../06-observability/tracing-and-logging.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)

## Follow-up questions

- Which actual teams or rotations fill the role-based ownership categories?
- Where is the canonical incident and evidence system?
- Which commands are supported operator interfaces rather than ad-hoc backend
  or Kubernetes actions?
- What game-day cadence is appropriate: per release, quarterly, or risk-triggered?
- Which runbooks must be complete before first production deployment?

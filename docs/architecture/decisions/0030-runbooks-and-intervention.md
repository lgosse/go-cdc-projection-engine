---
type: Architecture Decision Record
title: "ADR-0030: Runbooks and intervention"
description: Production operation requires indexed, exercised runbooks with explicit automation and audited intervention boundaries.
tags: [architecture, adr, operations, runbooks, incident-response]
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

# ADR-0030: Runbooks and intervention

## Context

The engine spans Kafka, MongoDB, Redis, Elasticsearch, Kubernetes, migrations,
reconciliation, DLQ custody, and privacy-sensitive data. Accepted decisions
already distinguish automatic bounded recovery from operator-controlled repairs,
replays, cutovers, retirement, and erasure behavior. Production readiness needs
these boundaries to be executable and auditable rather than implicit in code or
tribal knowledge.

## Decision

Maintain an indexed runbook catalogue covering lag and freshness breaches,
rebalances and hot partitions, dependency outages, Redis loss and rebuild,
schema or capability blocks, partial dual-write failure, bootstrap failure,
unsafe alias or migration state, reconciliation drift, DLQ triage and replay,
hot documents or nested-object limits, credential rotation, privacy
erasure/anonymization, and disaster recovery or target rebuild.

Each runbook records trigger and impact, scope and authoritative state,
permissions and prerequisites, diagnosis, safe automatic behavior, approved
operator actions, stop conditions, escalation, rollback or recovery, evidence,
and closure follow-up.

Automatic behavior is limited to bounded transient retries, backpressure,
partition or projection pause, migration degradation, restart after corrupted
local invariants, and the accepted Redis generation rebuild. Operators explicitly
initiate or approve DLQ replay, repairs, alias rollback or cutover, forced
migration abort, target retirement, privacy-erasure verification,
disaster-recovery promotion, and safety-fence overrides.

Use role-based ownership: engine on-call for projector behavior and control
state; source-domain owners for CDC completeness, source schema, and
anonymization; platform owners for Kafka, MongoDB, Redis, Elasticsearch,
Kubernetes, and telemetry; and projection owners for manifest semantics, search
correctness, and business acceptance. Concrete team assignments may be filled in
later.

Use metrics, traces, and structured logs for live diagnosis; durable MongoDB
records for privileged actions, migration state, repairs, and DLQ operations;
and the organization incident/ticket system for incident chronology and
follow-up. Link evidence with bounded run or operation identifiers and exclude
raw payloads, secrets, Restricted fields, and unprotected identifiers.

Validate runbooks in a safe environment through failure-injection or game-day
exercises. Review them whenever related code, manifests, dependencies, or
policies change.

## Alternatives considered

1. **Documentation-only procedures.** Easy to create, but they can diverge from
   supported commands and actual failure behavior.
2. **Fully automatic remediation.** Reduces operator effort but risks hiding
   correctness failures or performing destructive actions without context.
3. **One large operations manual.** Centralized but difficult to navigate under
   incident pressure; scenario-specific runbooks with an index are preferred.
4. **Team-specific undocumented procedures.** Fast locally but inconsistent and
   unauditable in production.

## Consequences

- Production readiness includes runbook coverage and exercised recovery paths.
- Privileged operations need authenticated access, actor attribution, and
  durable audit records.
- Incident tooling must link metrics, traces, logs, and MongoDB operation records.
- Concrete team assignments and exercise cadence remain operational follow-ups.
- Safety-sensitive actions remain explicit rather than hidden in automation.

## Validation

- Each required scenario has an indexed runbook with tested commands and stop
  conditions.
- Automatic retries, pauses, degradation, and restarts match documented
  behavior.
- DLQ replay, repair, migration rollback, retirement, and erasure procedures are
  attributable and auditable.
- Runbook evidence contains no raw payloads, secrets, Restricted fields, or
  unprotected identifiers.
- Failure-injection or game-day exercises demonstrate recovery and escalation.
- Runbooks are reviewed when related behavior changes.

## Review triggers

Revisit if incident response repeatedly depends on undocumented actions, runbook
steps cannot be exercised safely, ownership boundaries change, or automation
needs to expand beyond the accepted safety boundary.

## Related concepts

- [Runbooks and intervention](../07-operations/runbooks-and-intervention.md)
- [Modes and configuration](../07-operations/modes-and-configuration.md)
- [Deployment and orchestration](../07-operations/deployment-and-orchestration.md)
- [Health and diagnostics](../06-observability/health-and-diagnostics.md)
- [Metrics and alerting](../06-observability/metrics-and-alerting.md)
- [Tracing and logging](../06-observability/tracing-and-logging.md)
- [Backpressure, retry, and DLQ](../03-runtime/backpressure-retry-dlq.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)

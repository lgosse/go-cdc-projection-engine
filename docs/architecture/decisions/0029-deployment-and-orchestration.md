---
type: Architecture Decision Record
title: "ADR-0029: Deployment and orchestration"
description: Kubernetes schedules lifecycle-specific workloads while MongoDB durably coordinates migrations, checkpoints, leases, and fencing.
tags: [architecture, adr, operations, kubernetes, deployment, migration]
status: accepted
decision_id: ADR-0029
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Kubernetes schedules explicit workload types while engine-owned MongoDB remains authoritative for migration state, leases, checkpoints, and fencing.
  - Stream workloads use Deployments; bounded bootstrap, repair, DLQ replay, and migration use Jobs; audits use Jobs or CronJobs.
  - Migration is initially invoked as an idempotent executable Job rather than requiring a continuously running custom controller.
  - Stream rollouts require compatibility validation and graceful partition draining.
  - Maintenance workloads have independent resource, permission, concurrency, and alerting policies from live stream workloads.
---

# ADR-0029: Deployment and orchestration

## Context

The engine has a long-lived Kafka stream and several bounded or scheduled
operations. Kubernetes can restart and schedule these processes, but it cannot
by itself preserve migration ownership, bootstrap checkpoints, fencing, or
operator decisions across interrupted deployments. The source draft mentions
CI/CD and Helm pre-upgrade hooks, while accepted lifecycle decisions require a
durable, resumable migration state machine and graceful stream handoff.

## Decision

Use Kubernetes (K8s) primitives according to lifecycle:

- a Deployment for stream workers;
- Jobs for bootstrap, repair, DLQ replay, and migration;
- Jobs or CronJobs for audit;
- CI or a pre-deployment Job for validation.

Kubernetes owns starting, restarting, scheduling, and scaling workloads. It is
not the authoritative workflow store. Engine-owned MongoDB stores migration
phase, leases, fencing tokens, active target sets, bootstrap checkpoints,
verification results, and operator actions. Interrupted Jobs resume from this
state.

Deployment automation may use Helm, GitOps, CI/CD, or another platform
workflow, but it invokes idempotent migration operations rather than storing
state only in a transient hook. Initially, migration runs as an idempotent
executable Job; a continuously running custom controller is deferred until
migration volume, scheduling, approval, or dependency needs justify it.

Before a normal executable rollout, validate binary, manifest, dependency, and
metadata compatibility. Roll stream workers gracefully, draining Kafka
partitions before pod termination. For a projection schema migration, provision
and validate the new target, activate live dual-write, run the resumable
bootstrap Job, catch up and verify watermarks, deletes, fences, mappings, and
canonical data, then cut over the alias. Keep the old target through the
rollback window and retire it only after explicit approval and lifecycle
evidence.

Give stream Deployments and maintenance Jobs independent resources, service
accounts, scheduling and priority policies, concurrency limits, alerting, and
readiness scopes so bounded work cannot starve live CDC processing.

## Alternatives considered

1. **Helm or CI/CD hooks own migration state.** Easy to start, but interrupted
   invocations lose durable ownership and recoverability.
2. **A permanent migration controller from day one.** Provides richer scheduling
   and approval policy, but adds a highly available control-plane service before
   scale requires it.
3. **Every mode runs as a Deployment.** This complicates completion and retry
   semantics and risks indefinite maintenance processes.
4. **Bootstrap runs inside stream pods.** This couples full scans to live
   ingestion and weakens resource isolation and rollback.

## Consequences

- Kubernetes manifests explicitly declare workload type, executable, resources,
  permissions, restart behavior, and concurrency.
- MongoDB metadata needs retention, access control, leases, fencing, and audit
  support.
- Stream rollouts require graceful termination and compatibility gates.
- Maintenance workloads need separate capacity budgets and operational alerts.
- A future migration controller can reuse the same durable state model.

## Validation

- Failed or restarted Jobs resume from durable checkpoints and fencing state.
- Deployment-tool interruption does not erase migration ownership or progress.
- Stream rollouts preserve contiguous offset semantics and avoid unnecessary
  rebalance storms.
- Maintenance workloads cannot exhaust capacity reserved for live streams.
- Incompatible binary, manifest, dependency, or metadata combinations fail
  before source progress.
- A failed pre-cutover migration leaves the active target unaffected, and
  rollback returns to a current old target during the rollback window.

## Review triggers

Revisit if migration concurrency, approval requirements, dependency ordering, or
operator burden justify a dedicated controller, or if maintenance workloads
cannot be isolated within available Kubernetes capacity.

## Related concepts

- [Deployment and orchestration](../07-operations/deployment-and-orchestration.md)
- [Modes and configuration](../07-operations/modes-and-configuration.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Blue-green migration lifecycle](0016-blue-green-migration-lifecycle.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Availability and scaling](../05-quality-attributes/availability-and-scaling.md)
- [Compatibility and dependencies](../05-quality-attributes/compatibility-and-dependencies.md)
- [Health and diagnostics](../06-observability/health-and-diagnostics.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)

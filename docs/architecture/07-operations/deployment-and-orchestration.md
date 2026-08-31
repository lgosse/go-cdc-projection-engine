---
type: Architecture Review Topic
title: Deployment and orchestration
description: Defines workload types, rollout ordering, and control-plane coordination.
tags: [operations, kubernetes, deployment, migration]
sources:
  - resource: ../../design/system.md
    title: System design draft
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

# Deployment and orchestration

## Decision

Use Kubernetes (K8s) primitives according to workload lifecycle:

- a Deployment for long-lived stream workers;
- Jobs for bootstrap, repair, DLQ replay, and migration;
- Jobs or CronJobs for audit;
- CI or a pre-deployment Job for validation.

Kubernetes starts, restarts, schedules, and scales these workloads. It is not
the authoritative store for workflow progress. Engine-owned MongoDB stores
migration phase, leases, fencing tokens, active target sets, bootstrap
checkpoints, verification results, and operator actions. Interrupted Jobs resume
from that durable state rather than depending on the original Kubernetes Job
invocation.

Deployment automation may be Helm, GitOps, CI/CD, or another platform workflow,
but it must invoke idempotent migration operations rather than own migration
state in a transient hook. A continuously running custom migration controller is
not required initially; introduce one only when migration volume, scheduling,
approval, or dependency needs justify its operational cost.

For ordinary executable releases, validate binary, manifest, dependency, and
metadata compatibility before a graceful rolling stream update. Drain Kafka
partitions before pod termination, keep old and new executables compatible during
the rollout, and abort when readiness or capability checks fail.

For projection schema migrations, provision and validate the new target, activate
live dual-write, launch the resumable bootstrap Job, catch up and verify source
watermarks, deletes, fences, mappings, and canonical data, then cut over the
alias. Keep the old target through the rollback window and retire it only after
explicit approval and the accepted lifecycle evidence.

Give stream Deployments and maintenance Jobs independent resource requests and
limits, service accounts, scheduling and priority policies, concurrency limits,
alerting, and readiness scopes. This prevents bounded scans or repairs from
starving live CDC processing.

## Pros

- Matches workload lifecycle and completion semantics to Kubernetes primitives.
- Keeps migration recovery independent of one Helm or CI/CD invocation.
- Preserves durable fencing, checkpoints, and operator visibility.
- Allows maintenance capacity and permissions to be isolated from streaming.
- Supports safe rolling upgrades and zero-downtime target migrations.

## Cons and risks

- A durable workflow still needs explicit ownership and operational tooling.
- Bootstrap, audit, and repair Jobs can compete with live streams unless capacity
  isolation is configured.
- Release rollout and manifest migration are separate compatibility problems.
- A single migration executable Job provides less automation than a dedicated
  controller at larger scale.
- Kubernetes policy and deployment configuration become part of the safety
  boundary.

## Alternatives considered

1. Helm or CI/CD hooks own migration state. This is easy to start but loses
   durable ownership and recoverability when an invocation is interrupted.
2. A permanent custom migration controller runs from the beginning. This enables
   richer scheduling and approval policy but adds a highly available control-plane
   service before scale requires it.
3. Every mode runs as a Deployment. This treats bounded work as a service,
   complicates completion and retry semantics, and risks indefinite maintenance
   processes.
4. Bootstrap runs inside stream pods. This reduces workload types but couples
   full scans to live ingestion and weakens resource isolation and rollback.

## Consequences

- Kubernetes manifests must declare workload type, executable, resources,
  permissions, restart behavior, and concurrency explicitly.
- MongoDB metadata requires retention, access control, leases, fencing, and audit
  support.
- Stream rollouts need graceful termination and compatibility gates.
- Maintenance workloads need separate capacity budgets and operational alerts.
- Introducing a migration controller later remains possible without changing the
  migration state model.

## Validation

- A failed or restarted Job resumes from durable checkpoints and fencing state.
- A deployment-tool interruption does not erase migration ownership or progress.
- Stream rollouts preserve contiguous offset semantics and avoid unnecessary
  rebalance storms.
- Maintenance workloads cannot exhaust the capacity reserved for live streams.
- Incompatible binary, manifest, dependency, or metadata combinations fail before
  source progress.
- A failed pre-cutover migration leaves the active target unaffected, and a
  rollback returns to a current old target during the rollback window.

## Review trigger

Revisit if migration concurrency, approval requirements, dependency ordering, or
operator burden justify a dedicated controller, or if maintenance workloads
cannot be isolated within the available Kubernetes capacity.

## Related concepts

- [Modes and configuration](modes-and-configuration.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Blue-green migration lifecycle](../decisions/0016-blue-green-migration-lifecycle.md)
- [Bootstrap consistency](../04-data-lifecycle/bootstrap-consistency.md)
- [Availability and scaling](../05-quality-attributes/availability-and-scaling.md)
- [Compatibility and dependencies](../05-quality-attributes/compatibility-and-dependencies.md)
- [Health and diagnostics](../06-observability/health-and-diagnostics.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)

## Follow-up questions

- Which deployment mechanism will invoke migration Jobs?
- Should maintenance Jobs use a separate namespace, node pool, or resource
  quota?
- What rollout timeout and graceful-drain period fit production partitions?
- Which operations require explicit human approval before execution or target
  retirement?
- At what migration volume should a dedicated controller be introduced?

---
type: Architecture Review Topic
title: Deployment and orchestration
description: Defines workload types, rollout ordering, and control-plane coordination.
tags: [operations, kubernetes, deployment, migration]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Deployment and orchestration

## Decision to stamp

Define how stream deployments, bounded jobs, migrations, and configuration
rollouts are coordinated safely.

## Draft proposal

Use Kubernetes Deployments for stream workloads and Jobs/CronJobs for bounded
bootstrap and audit work. Treat schema migration as an explicit idempotent
workflow with durable state rather than relying solely on a Helm pre-upgrade
hook. Roll out manifests and engine versions under declared compatibility rules,
with disruption budgets and graceful partition handoff.

## Pros

- Matches workload lifecycle to Kubernetes primitives.
- Migration recovery is not tied to one Helm release invocation.
- Explicit compatibility supports staged rollout.

## Cons and risks

- A durable workflow needs ownership and possibly another controller.
- Jobs can compete with live streaming for source and sink capacity.
- Configuration rollout across many groups is operationally complex.

## Questions to stamp

- Which deployment tooling and controllers are available?
- How is job resource usage isolated from production streams?
- What rollback behavior applies to binary versus manifest releases?

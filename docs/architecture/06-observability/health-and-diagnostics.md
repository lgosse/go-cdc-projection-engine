---
type: Architecture Review Topic
title: Health and diagnostics
description: Defines process health, workload readiness, progress, and safe diagnostic surfaces.
tags: [observability, health, readiness, diagnostics]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Health and diagnostics

## Decision to stamp

Define what health endpoints mean and how operators inspect projection-level
state without relying on logs alone.

## Draft proposal

Expose minimal unauthenticated liveness and Kubernetes readiness endpoints, plus
an authenticated diagnostic view showing each projection's configuration hash,
assigned partitions, paused reason, lag/event age, active write targets,
migration phase, last successful flush, and terminal failure summary.

## Pros

- Prevents a global `200` from hiding blocked workloads.
- Makes control-plane state and progress directly inspectable.
- Keeps Kubernetes probes cheap and stable.

## Cons and risks

- Diagnostic endpoints can leak topology or sensitive identifiers.
- Aggregating dependency checks can make readiness flap.
- A detailed endpoint becomes another compatibility surface.

## Questions to stamp

- Which conditions remove a pod from service versus pause one projection?
- Who can access diagnostics?
- What data is authoritative when diagnostics and backend state disagree?

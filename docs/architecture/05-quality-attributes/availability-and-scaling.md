---
type: Architecture Review Topic
title: Availability and scaling
description: Defines availability objectives, degradation modes, and scaling behavior.
tags: [quality, availability, scaling, kubernetes]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Availability and scaling

## Decision to stamp

Define availability objectives and how replicas, Kafka rebalances, and dependency
outages affect progress and reads.

## Draft proposal

Keep search reads available independently of projector availability. Run multiple
stream replicas using cooperative assignment, use graceful revocation and
shutdown, and scale on sustained lag plus saturation rather than lag alone.
Degrade per projection when possible and bound restart loops.

## Pros

- Read availability does not depend on live ingestion.
- Cooperative scaling can reduce partition movement.
- Projection isolation limits the blast radius of bad configuration.

## Cons and risks

- More replicas do not help beyond available partitions.
- HPA changes can cause the rebalances they are trying to resolve.
- Per-projection degradation complicates readiness and alerting.

## Questions to stamp

- What SLO applies to search freshness versus worker uptime?
- How long may each dependency be unavailable before intervention?
- What is the safe minimum and maximum replica count per workload group?

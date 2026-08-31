---
type: Architecture Review Topic
title: Disaster recovery
description: Defines restoration paths for lost caches, indices, checkpoints, and Kafka history.
tags: [lifecycle, recovery, resilience]
status: in-review
---

# Disaster recovery

## Decision to stamp

Define recovery objectives and procedures for each material state-loss scenario.

## Draft proposal

Document recovery matrices for Redis loss, Elasticsearch index loss, corrupt
projection data, lost migration state, expired Kafka history, and MongoDB
unavailability. Prefer rebuilding derived state from MongoDB plus retained Kafka;
back up only state that cannot be reconstructed within the required objective.

## Pros

- Tests the claim that the engine and its projections are rebuildable.
- Exposes hidden durable state before implementation.
- Aligns backup cost with actual authority.

## Cons and risks

- Full rebuild time may exceed acceptable recovery objectives.
- Cross-service source availability can dominate restoration time.
- Recovery procedures require regular exercises to stay credible.

## Questions to stamp

- What are recovery time and recovery point objectives per mode?
- Which state is backed up versus rebuilt?
- What happens when Kafka retention has elapsed during a prolonged outage?

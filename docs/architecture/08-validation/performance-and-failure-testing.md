---
type: Architecture Review Topic
title: Performance and failure testing
description: Defines benchmark, soak, rebalance, and fault-injection evidence.
tags: [validation, performance, chaos, resilience]
status: proposed
---

# Performance and failure testing

## Decision to stamp

Define realistic workload models and failure exercises for capacity and recovery
claims.

## Draft proposal

Benchmark representative projection shapes, hot-key skew, nested cardinality,
reference fan-out, transformation cost, and dual-write load. Run soak tests plus
fault injection for dependency latency/outage, partial bulk failures, consumer
rebalance, pod termination, Redis loss, Kafka redelivery, bootstrap interruption,
and migration-controller restart.

## Pros

- Validates both steady-state capacity and catch-up behavior.
- Exposes retry amplification and memory pressure before production.
- Makes recovery claims reproducible.

## Cons and risks

- Realistic datasets may contain sensitive information.
- Distributed test results can be noisy and expensive.
- Fault injection can validate only modeled failure classes.

## Questions to stamp

- What dataset generator preserves production distributions safely?
- Which latency and recovery regressions block a release?
- Where can destructive migration and recovery exercises run?

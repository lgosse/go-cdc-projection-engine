---
type: Architecture Review Topic
title: Compatibility and dependencies
description: Defines supported platform versions and upgrade responsibility.
tags: [quality, compatibility, dependencies]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: proposed
---

# Compatibility and dependencies

## Decision to stamp

Define the supported version matrix and capability detection for Go, MongoDB,
Kafka, Redis, Elasticsearch, Kubernetes, Bloblang, and OpenTelemetry.

## Draft proposal

Publish a tested compatibility matrix with minimum and maximum major versions.
Validate required capabilities at startup and in CI integration suites. Pin
client libraries and define an upgrade cadence; do not rely only on nominal
server version strings.

## Pros

- Prevents hidden reliance on unavailable APIs or semantics.
- Makes dependency upgrades planned architecture work.
- Capability checks produce clearer startup failures.

## Cons and risks

- A multi-service integration matrix is expensive.
- Narrow support windows may conflict with platform reality.
- Server distributions can differ despite matching version numbers.

## Questions to stamp

- What versions exist in target environments?
- Which clients and transformation runtime are acceptable dependencies?
- What backward-compatibility promise does the engine itself offer?

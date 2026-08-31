---
type: Architecture Decision Record
title: "ADR-0023: Compatibility and dependencies"
description: Supported dependency combinations are capability-tested, pinned per release, and fail readiness when required behavior is unavailable.
tags: [architecture, adr, compatibility, dependencies, upgrades, capabilities]
status: accepted
decision_id: ADR-0023
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Each dependency has Required, Supported, Compatible, or Unsupported status based on tested capabilities, not version strings alone.
  - Required capability checks fail readiness before source consumption or progress advancement.
  - Client libraries, Go toolchain, Bloblang runtime, and OpenTelemetry components are pinned per engine release.
  - Rolling binary upgrades are allowed only when old and new workers share a compatible manifest and durable metadata protocol; otherwise use drain or blue-green deployment.
  - Exact platform inventories, support windows, upgrade cadence, and workload-level failure scopes remain operational follow-ups.
---

# ADR-0023: Compatibility and dependencies

## Context

The engine depends on specific behavior from MongoDB change streams, Kafka
consumer assignment, Redis expiry and generation operations, Elasticsearch
aliases/mappings/scripts, Kubernetes lifecycle primitives, the embedded Bloblang
runtime, and OpenTelemetry telemetry. Nominal version strings cannot prove that
these capabilities are available in a particular distribution or configuration.

## Decision

Publish a tested compatibility matrix for Go, MongoDB, Kafka, Redis,
Elasticsearch, Kubernetes, Bloblang, and OpenTelemetry. Classify each entry as
**Required**, **Supported**, **Compatible**, or **Unsupported**. A required
dependency capability that is absent leaves the affected workload unready and
must not allow source consumption or progress advancement.

Use capability checks in addition to version checks. Validate MongoDB replica
set/change-stream and source cluster-time support; Kafka manual commits and
cooperative assignment; Redis expiry and generation operations; Elasticsearch
bulk, alias, mapping, nested, and script behavior; Kubernetes graceful
termination and workload primitives; the supported Bloblang subset; and
OpenTelemetry export behavior.

Pin client libraries, the Go toolchain, the embedded transformation runtime, and
telemetry components per engine release. Test the oldest supported dependency,
the production platform version, and a current upgrade candidate. Do not promise
compatibility with arbitrary future major versions.

The engine promises explicit manifest, projection, and transformation versions.
Rolling binary upgrades are safe only when old and new workers can share the
manifest and durable metadata protocol. Otherwise drain the old workers or use
blue-green deployment. Existing physical indices and migration state are never
silently downgraded.

## Alternatives considered

1. Support one exact version of every dependency and require coordinated
   upgrades. This simplifies testing but reduces deployment flexibility and
   increases upgrade coupling.
2. Trust semantic version strings without capability probes. This is cheaper but
   fails with vendor distributions, disabled features, and configuration drift.
3. Track only client-library compatibility. This misses server-side APIs,
   permissions, and deployment capabilities required by the engine.

## Consequences

- A maintained compatibility matrix and integration environments become release
  responsibilities.
- Startup and readiness failures must name the missing capability and scope.
- Security patches can be applied within a pinned line; major upgrades require
  explicit compatibility testing.
- Binary, manifest, and projection versions must be coordinated during rollout.
- Unsupported capability combinations fail closed rather than silently
  degrading.

## Validation

- Missing required capabilities fail readiness before source consumption.
- The matrix is exercised against real dependency distributions and versions.
- Rolling upgrades preserve offsets, metadata, transformations, and projections.
- Incompatible binary or manifest changes use drain or blue-green deployment.
- Dependency failures produce actionable diagnostics and preserve telemetry.

## Review triggers

Revisit when target platform versions change, a dependency deprecates a required
capability, the support window becomes too costly, or rolling compatibility
cannot preserve the durable metadata and projection contracts.

## Related concepts

- [Compatibility and dependencies](../05-quality-attributes/compatibility-and-dependencies.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Transformation contract](../02-contracts/transformation-contract.md)
- [Projection schema](../02-contracts/projection-schema.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Availability and scaling](../05-quality-attributes/availability-and-scaling.md)
- [Deployment and orchestration](../07-operations/deployment-and-orchestration.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)

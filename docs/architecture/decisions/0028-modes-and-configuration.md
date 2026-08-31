---
type: Architecture Decision Record
title: "ADR-0028: Modes and configuration"
description: Separate mode-specific executables are released together, initially packaged in one image, and orchestrated as explicit Kubernetes workloads.
tags: [architecture, adr, operations, cli, configuration, kubernetes]
status: accepted
decision_id: ADR-0028
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Mode-specific executables share internal libraries and are released from the same source version.
  - The initial image may contain all executables; Kubernetes workloads invoke only their assigned executable.
  - Manifest semantics cannot be silently overridden by operational flags or environment variables.
  - Configuration is validated before source consumption and frozen for the process lifetime; restart-based changes are acceptable for now.
  - Mutating maintenance operations require explicit target/environment confirmation and durable coordination where applicable.
---

# ADR-0028: Modes and configuration

## Context

The engine has materially different operational lifecycles: continuous Kafka
streaming, bounded bootstrap and repair jobs, scheduled audits, DLQ inspection
and replay, and durable migration orchestration. Kubernetes can schedule these
as separate workloads, but the executable and configuration boundary determines
how easily accidental mode changes, permission sprawl, release skew, and
resource contention are prevented.

The source draft proposes `--mode` operations for streaming, bootstrap, audit,
and DLQ replay. Accepted decisions also require durable MongoDB migration state,
explicit capability validation, protected maintenance actions, and zero-downtime
bootstrap.

## Decision

Build separate executables for the distinct operational modes from shared
internal libraries and release them as one compatible versioned bundle. The
initial container image may package all executables; this avoids an unnecessary
image explosion while preserving explicit process entrypoints.

Use explicit mode-specific workloads:

- a Kubernetes Deployment for `stream`;
- Jobs for `bootstrap`, `repair`, DLQ replay, and migration;
- Jobs or CronJobs for audit;
- CI or a pre-deployment Job for validation.

Every workload selects its executable explicitly. There is no automatic mode
selection. Kubernetes supplies workload-specific resources, service accounts,
restart policies, scheduling, and scaling; engine-owned MongoDB remains the
authority for migration, replay, repair, checkpoint, lease, and fencing state.

Keep manifest semantics authoritative for identity, relationships,
transformations, mappings, target version, and field classifications. Resolve
operational configuration from safe defaults, deployment configuration,
environment variables, and explicit flags, while delivering secret values via
environment-provided references. Reject overrides that would silently alter
manifest semantics.

Validate mode, manifest, capabilities, target, and mode-specific settings before
source progress. Freeze the resolved configuration for the process lifetime and
record redacted configuration and manifest hashes, mode, environment, target,
and run identifier in diagnostics and durable control metadata. Restart or
redeploy for configuration changes for now, while keeping interfaces evolvable
toward safe live reload.

Require explicit target/environment confirmation, preflight validation, dry-run
support where meaningful, and an exclusive durable lease for maintenance
operations that control shared state. Never retire or delete a target
automatically.

## Alternatives considered

1. **One binary with subcommands.** This reduces artifacts and version skew but
   places all capabilities in every process and depends more heavily on runtime
   command validation to prevent mode mistakes.
2. **One image per executable.** This gives the strongest artifact and
   permission isolation but adds image, release, policy, and operational
   complexity. It remains an available evolution if needed.
3. **Automatic mode selection.** This makes invocation shorter but allows a
   deployment mistake to silently change workload behavior and is rejected.

## Consequences

- Mode lifecycles, scaling, resources, and permissions are independently
  expressible in Kubernetes.
- Shared libraries and release versions reduce semantic drift but require
  compatibility testing across stream and maintenance executables.
- The first image contains more code than each workload needs; image splitting
  can be introduced later for security, ownership, or release-cadence reasons.
- Configuration and manifest provenance become auditable operational data.
- Kubernetes restarts Jobs and pods, while durable MongoDB state makes recovery
  independent of a particular invocation.

## Validation

- Declared stream workloads cannot launch maintenance executables.
- Invalid configuration and incompatible capabilities prevent source progress.
- Interrupted Jobs resume from durable checkpoints, leases, and fencing state.
- Workload-specific resource policies prevent maintenance jobs from starving
  live streaming.
- Diagnostics expose redacted configuration and manifest hashes.
- Rolling upgrades reject incompatible executable, metadata, or manifest
  combinations.

## Review triggers

Revisit if security scanning, least-privilege requirements, independent release
cadence, or mode-specific ownership make a single all-mode image unacceptable,
or if configuration changes require live reload rather than restart.

## Related concepts

- [Modes and configuration](../07-operations/modes-and-configuration.md)
- [Deployment and orchestration](../07-operations/deployment-and-orchestration.md)
- [Blue-green migration lifecycle](0016-blue-green-migration-lifecycle.md)
- [Compatibility and dependencies](../05-quality-attributes/compatibility-and-dependencies.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)
- [Health and diagnostics](../06-observability/health-and-diagnostics.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)

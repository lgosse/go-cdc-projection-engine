---
type: Architecture Review Topic
title: Modes and configuration
description: Defines operational modes, configuration sources, validation, and safeguards.
tags: [operations, cli, configuration]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0028
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Mode-specific executables are built from shared libraries and released from the same source version.
  - The initial container image may package all mode executables; Kubernetes workloads invoke only their assigned executable.
  - Manifest semantics remain authoritative and cannot be silently overridden by operational flags or environment variables.
  - Configuration is validated before source consumption and frozen for the process lifetime; changes require restart or redeployment for now.
  - Mutating maintenance operations require explicit mode, target/environment confirmation, validation, and durable coordination where applicable.
---

# Modes and configuration

## Decision

Build separate executables for distinct operational modes while keeping their
implementation in one repository and sharing internal libraries. The initial
release is a single versioned bundle; its container image may contain all mode
executables. Kubernetes runs them as separate workloads with mode-specific
commands, resources, service accounts, restart policies, and scaling rules.

Support these explicit operations:

- `stream`: long-lived Kafka consumption and offset commits.
- `bootstrap`: bounded MongoDB scanning into a pinned target without owning the
  live Kafka cursor.
- `audit`: read-only reconciliation by default.
- `repair`: explicit, rate-limited, audited repair through the normal fenced
  write path.
- `dlq inspect`: read-only DLQ investigation.
- `dlq replay`: selected durable DLQ records replayed idempotently without
  advancing Kafka offsets.
- `migration`: drives the durable migration lifecycle and its verification
  gates; deployment automation may invoke it.
- `validate`: validates manifests, configuration, capabilities, and target
  compatibility without consuming source data.

Do not auto-detect a mode. Every workload must select its executable and mode
explicitly. Stream workloads run as Kubernetes Deployments; bootstrap, repair,
DLQ replay, and migration run as bounded Jobs; audits run as Jobs or CronJobs;
validation runs in CI or as a pre-deployment Job.

Separate configuration ownership into two domains. The manifest owns projection
semantics: identity, relationships, transformations, mappings, target version,
and field classifications. Operational configuration uses safe built-in
defaults, deployment configuration, environment variables, and explicit flags.
Environment-provided secret values remain the supported secret delivery path.
Operational overrides must not silently change manifest semantics.

Before reading Kafka or scanning MongoDB, each process validates its mode,
manifest, capabilities, target, and mode-specific settings, then freezes a
resolved configuration snapshot. Record a redacted configuration hash, manifest
hash, mode, environment, target, and run identifier in diagnostics and durable
control metadata. Configuration changes require restart or redeployment for
now; interfaces should remain evolvable toward safe live reload.

Mutating maintenance operations require explicit target and environment
confirmation, preflight validation, dry-run support where meaningful, and an
exclusive durable lease when they control shared migration, repair, or replay
state. No operation automatically retires or deletes a target.

## Pros

- Prevents a stream workload from accidentally executing a bounded or destructive
  maintenance operation.
- Preserves independent Kubernetes scaling, resource limits, and permissions.
- Keeps all modes on one shared implementation and release compatibility line.
- Makes migration, repair, and replay recoverable through durable metadata rather
  than transient deployment hooks.
- Allows the initial deployment to use one image while leaving stronger image
  separation available later.

## Cons and risks

- Multiple executables still require coordinated builds and compatibility checks.
- A single initial image contains more code and potential vulnerabilities than a
  mode-specific image.
- Separate mode interfaces become release compatibility surfaces.
- Configuration provenance and validation must be kept consistent across
  executables.
- Migration orchestration remains a control-plane concern in addition to
  Kubernetes scheduling.

## Alternatives considered

1. One binary with subcommands. This minimizes release artifacts and version
   skew, but puts every capability in every process and relies more heavily on
   command validation to prevent mode mistakes.
2. A separate image for every executable. This gives the strongest artifact and
   permission boundary, but increases image, release, policy, and operational
   complexity. It remains a future option if security or ownership requires it.
3. Automatic mode selection from configuration. This simplifies invocation but
   makes an incorrect deployment configuration capable of changing workload
   behavior silently, so it is rejected.

## Consequences

- Kubernetes manifests must pin a compatible release bundle and invoke an
  explicit executable.
- Shared metadata, manifest, and capability protocols must be compatibility
  tested before rolling upgrades or maintenance jobs run.
- Stream, bootstrap, audit, repair, replay, and migration have independent
  resource and permission policies.
- The first image is convenient to publish, but image splitting may become a
  later security or ownership decision.
- Operators get bounded, auditable commands instead of one overloaded service
  lifecycle.

## Validation

- A stream Deployment cannot start a bootstrap, repair, replay, or migration
  executable through its declared command and policy.
- Invalid configuration or incompatible capabilities prevent source progress.
- Bootstrap, repair, replay, and migration Jobs resume from durable state after
  pod or Job interruption.
- Separate workloads scale and fail independently without starving live stream
  processing.
- Resolved configuration and manifest hashes are visible in diagnostics and
  control metadata without exposing secrets or protected fields.
- Rolling upgrades reject incompatible executable and metadata combinations.

## Review trigger

Revisit if image vulnerabilities, permission boundaries, independent release
cadence, or mode-specific ownership justify splitting the initial image, or if
configuration changes require safe live reload rather than restart.

## Related concepts

- [Deployment and orchestration](deployment-and-orchestration.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Blue-green migration lifecycle](../decisions/0016-blue-green-migration-lifecycle.md)
- [Compatibility and dependencies](../05-quality-attributes/compatibility-and-dependencies.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)
- [Health and diagnostics](../06-observability/health-and-diagnostics.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)

## Follow-up questions

- Which Kubernetes policy mechanism will restrict each executable and workload?
- When should the initial all-mode image be split into separate images?
- What exact operational settings may be overridden by flags versus environment
  variables?
- Should migration orchestration eventually become a dedicated controller?

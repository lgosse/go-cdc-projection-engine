---
type: Architecture Review Topic
title: Manifest contract
description: Defines the projection manifest's scope, versioning, and validation boundary.
tags: [contracts, manifest, configuration]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: accepted
decision_id: ADR-0040
accepted_on: 2026-08-31
owner: TBD
conditions:
  - One versioned YAML manifest defines each logical projection's semantic contract.
  - Layered structural, semantic, transformation, capability, sensitivity, and capacity validation occurs before source progress.
  - Manifest semantics cannot be silently overridden by deployment configuration; secrets are references only.
  - One source event may update multiple subscribed projections, with independent terminal outcomes before shared offset progress advances.
  - Manifest-schema, projection, transformation, and engine versions remain distinct and compatibility-checked.
---

# Manifest contract

## Decision

Use one versioned YAML manifest per logical projection. It declares logical
projection identity and version; subscribed Kafka topics and source identity;
MongoDB references for bootstrap, audit, and repair; source-to-target fields and
types; root, reference, nested, and multi-hop relations; root-resolution and
reverse-index policies; cardinality and fan-out limits; Bloblang transformations
and their version; Elasticsearch mappings, settings, aliases, and target version;
field sensitivity classifications; cache, fallback, deletion, and retention
policies; and manifest-level capacity and safety limits.

The manifest may contain environment-variable or secret references such as a URI
variable name, but never literal credentials, tokens, or key material.

Deployment configuration owns actual environment values and secrets, replica and
resource settings, worker concurrency and scheduling, dependency endpoints and
pool settings, workload identity and environment, mode selection, target
confirmation, and operational retry/queue/rate limits not specific to projection
semantics. Deployment settings must not silently override manifest semantics.

Validate before Kafka consumption or MongoDB scanning:

1. YAML parsing and structural schema validation.
2. Required-field, type, and unknown-field checks.
3. Semantic graph validation for identity, root resolution, cycles, fan-out,
   target paths, and relation compatibility.
4. Transformation compilation and resource-bound checks.
5. Elasticsearch mapping/settings compatibility classification.
6. Sensitivity, destination-policy, and capacity-limit validation.
7. Dependency capability checks.

Unknown or unsupported fields, schema versions, transformations, or capabilities
block the affected workload. The engine does not silently coerce or ignore them.
Record an immutable manifest hash, manifest-schema version, projection version,
transformation version, and validation result in diagnostics and engine metadata.
Each physical target is produced from one pinned manifest set.

Allow one source event to update multiple logical projections when multiple
manifests subscribe to the topic and identity. Each projection has independent
validation and terminal outcome. A shared Kafka offset advances only after all
required projection outcomes are terminal under the accepted delivery rules. The
first production release validates one representative projection; multi-
projection fan-out remains an architectural capability rather than a scale
promise.

Keep manifest-schema, logical projection, transformation, and engine release
versions distinct. A manifest is compatible when the engine supports its declared
capabilities, migration-required when persisted meaning changes, and blocked when
malformed, ambiguous, unsupported, or incompatible. No version is silently
downgraded or interpreted using an older semantic contract.

## Pros

- Keeps the engine domain-agnostic and reviewable.
- Enables deterministic validation before consumers or scans start.
- Separates semantic projection ownership from deployment operations.
- Supports efficient multi-projection subscription without losing per-projection
  outcome visibility.
- Versioned hashes make targets and diagnostics reproducible.

## Cons and risks

- A rich manifest can become a domain-specific programming language.
- Layered validation is more complex than JSON Schema alone.
- Multi-projection fan-out complicates offset completion and observability.
- A single manifest can tightly couple source, transformation, and Elasticsearch
  concerns.

## Alternatives considered

1. One global configuration file for all projections. This couples independent
   review and rollback decisions.
2. Put source and mapping semantics in code. This violates the domain-agnostic
   objective and requires binary releases for domain changes.
3. Let deployment configuration override manifest fields. This harms review and
   reproducibility.
4. Restrict each event to one projection. This duplicates Kafka reads and limits
   efficient multi-projection operation.
5. Use JSON Schema only. This cannot validate graph, fan-out, transformation,
   sensitivity, or semantic compatibility rules.

## Consequences

- Manifest files are versioned release inputs and require compatibility evidence.
- Validation must occur before source progress and identify the affected scope.
- Projection authors need explicit policies for fallbacks, reverse indexes,
  limits, and sensitive destinations.
- Shared topic consumers need per-projection outcomes before committing offsets.
- Deployment configuration cannot be used as an undocumented semantic override.

## Validation

- A representative manifest passes structural and semantic validation before
  source progress.
- Invalid identity, cycles, fan-out, transformations, sensitivity, and
  compatibility are rejected deterministically.
- Manifest hashes and version sets are recorded on targets and diagnostics.
- One source event updating multiple projections produces independent outcomes
  and correct shared offset behavior.
- Deployment overrides cannot change manifest semantics.
- Compatible, migration-required, and blocked versions are classified
  consistently.

## Review trigger

Revisit if the manifest becomes unmanageably expressive, multi-projection fan-out
causes unacceptable coupling, a required capability cannot be declared safely,
or organization governance requires a different semantic/deployment boundary.

## Related concepts

- [Manifest contract](manifest-contract.md)
- [CDC event envelope](cdc-event-envelope.md)
- [Identity, time, and ordering](identity-time-ordering.md)
- [Relationship model](relationship-model.md)
- [Transformation contract](transformation-contract.md)
- [Projection schema](projection-schema.md)
- [Modes and configuration](../07-operations/modes-and-configuration.md)
- [Schema evolution](../04-data-lifecycle/schema-evolution.md)
- [Compatibility and dependencies](../05-quality-attributes/compatibility-and-dependencies.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)

---
type: Architecture Decision Record
title: "ADR-0040: Manifest contract"
description: Versioned manifests declare projection semantics and are validated structurally and semantically before source progress.
tags: [architecture, adr, contracts, manifest, configuration]
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

# ADR-0040: Manifest contract

## Context

The manifest connects source topics and identities, MongoDB bootstrap data,
relations, transformations, Elasticsearch mappings, sensitivity policy, and
operational limits. A single YAML document per logical projection is useful, but
without a validation boundary it can become an unreviewable domain language or
silently diverge from deployment settings.

## Decision

Use one versioned YAML manifest per logical projection. It declares logical
projection identity and version; subscribed Kafka topics and source identity;
MongoDB references for bootstrap, audit, and repair; source-to-target fields and
types; root, reference, nested, and multi-hop relations; root-resolution and
reverse-index policies; cardinality and fan-out limits; Bloblang transformations
and their version; Elasticsearch mappings, settings, aliases, and target
version; field sensitivity classifications; cache, fallback, deletion, and
retention policies; and manifest-level capacity and safety limits.

The manifest may contain environment-variable or secret references, but never
literal credentials, tokens, or key material. Deployment configuration owns
actual values and secrets, resources, scheduling, endpoints, workload identity,
mode selection, target confirmation, and operational limits not specific to
projection semantics. It cannot silently override manifest semantics.

Validate before Kafka consumption or MongoDB scanning: YAML and structural
schema; required, type, and unknown fields; semantic graph, identity, root,
cycle, fan-out, path, and relation compatibility; transformation compilation and
resource bounds; Elasticsearch compatibility; sensitivity and destination
policy; capacity limits; and dependency capabilities. Unknown or unsupported
fields, versions, transformations, or capabilities block the affected workload.

Record immutable manifest and version hashes, validation results, and the
manifest-schema, projection, transformation, and engine release versions in
diagnostics and engine metadata. No version is silently downgraded.

Allow one source event to update multiple subscribed logical projections. Each
projection has independent validation and terminal outcome; shared Kafka offset
progress advances only after all required projection outcomes are terminal. The
first production release validates one representative projection, while
multi-projection fan-out remains an architectural capability rather than a scale
promise.

## Alternatives considered

1. **One global configuration file.** This couples independent review and
   rollback decisions.
2. **Put semantics in code.** This violates the domain-agnostic objective and
   requires binary releases for domain changes.
3. **Let deployment settings override manifests.** This harms review and
   reproducibility.
4. **Restrict each event to one projection.** This duplicates Kafka reads and
   limits efficient multi-projection operation.
5. **Use JSON Schema only.** This cannot validate graph, fan-out,
   transformation, sensitivity, or semantic compatibility rules.

## Consequences

- Manifest files are versioned release inputs requiring compatibility evidence.
- Projection authors must declare fallback, reverse-index, limit, and sensitive
  destination policies.
- Shared topic consumers need per-projection outcomes before offset commit.
- Deployment configuration cannot be an undocumented semantic override.
- A rich manifest needs governance to avoid becoming an unbounded programming
  language.

## Validation

- A representative manifest passes structural and semantic validation before
  source progress.
- Invalid identity, cycles, fan-out, transformations, sensitivity, and
  compatibility are rejected deterministically.
- Manifest hashes and version sets are recorded on targets and diagnostics.
- One event updating multiple projections produces independent outcomes and
  correct shared offset behavior.
- Deployment overrides cannot change manifest semantics.
- Compatible, migration-required, and blocked versions are classified
  consistently.

## Review triggers

Revisit if the manifest becomes unmanageably expressive, multi-projection fan-out
causes unacceptable coupling, a required capability cannot be declared safely,
or governance requires a different semantic/deployment boundary.

## Related concepts

- [Manifest contract](../02-contracts/manifest-contract.md)
- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Relationship model](../02-contracts/relationship-model.md)
- [Transformation contract](../02-contracts/transformation-contract.md)
- [Projection schema](../02-contracts/projection-schema.md)
- [Modes and configuration](../07-operations/modes-and-configuration.md)
- [Schema evolution](../04-data-lifecycle/schema-evolution.md)
- [Compatibility and dependencies](../05-quality-attributes/compatibility-and-dependencies.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)

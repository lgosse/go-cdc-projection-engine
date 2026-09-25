---
type: Architecture Decision Record
title: "ADR-0062: Transformation version and target association"
description: Pins transformation semantics to each physical projection target.
tags: [architecture, adr, transformations, schema-evolution, migration]
status: accepted
decision_id: ADR-0062
accepted_on: 2026-09-25
owner: Project maintainer
conditions:
  - Each physical target has one immutable manifest and transformation identity.
  - A transformation version is bound to its normalized definition and versioned allowlist digest.
  - Changes that can alter persisted output meaning use a new physical target and the blue-green lifecycle.
  - Evaluator and binary compatibility remains governed by ADR-0023 and its rolling-deployment follow-ups.
---

# ADR-0062: Transformation version and target association

## Status

Accepted on 2026-09-25. Owner: Project maintainer.

## Context

The same transformation must produce the same projection in stream,
bootstrap, audit, repair, and migration modes. A physical target can remain
queryable while a new target is populated and verified. The engine therefore
needs to know which transformation produced each target and must not silently
mix outputs produced under different transformation semantics.

The manifest contract already records a manifest hash and transformation
version, while schema evolution classifies changes that alter persisted meaning
as reindex-required. What remained open was whether a version marker belongs on
every document or whether the engine binds the version to the target.

## Decision

Bind transformation identity to each physical target. Authoritative engine-owned
metadata (with MongoDB as authority) records the immutable manifest hash,
transformation version, and a stable digest of the canonical normalized
transformation definition together with its versioned allowlist identity.
Expose this pin in diagnostics. Do not require a public per-document
transformation-version field; a rebuildable target-side copy, if present, is
not authoritative.

A declared transformation version cannot be rebound to a different digest. A
version/digest mismatch blocks preflight before source progress. Changes that
can alter persisted output meaning require a new versioned physical target and
the accepted blue-green migration lifecycle. During dual-write, evaluate each
target using that target's pinned transformation identity. Bootstrap and verify
the new target before read-alias cutover.

An evaluator or binary upgrade that preserves transformation semantics follows
the compatibility policy in [ADR-0023](0023-compatibility-and-dependencies.md).
If an upgrade can change output meaning or the supported allowlist, treat it as
a transformation semantic change and migrate to a new target. This decision
does not settle the broader separation of manifest-schema, projection-schema,
transformation, and engine-release version families; that remains Q-057.

For example, suppose `tasks-v12` is pinned to manifest hash `M12`, transform
version `1.3`, and digest `D13`. If changing the rounding rule alters projected
`total_minutes`, preflight creates a migration to `tasks-v13` pinned to version
`1.4` and digest `D14`. While both targets are active, each gets output from its
own pinned transform. If someone edits version `1.4` but leaves its target pin
at `D14`, preflight blocks the mismatch rather than silently rewriting the
meaning of `tasks-v13`.

## Alternatives considered

1. **Store a public version marker on every document.** This makes individual
   documents easier to inspect, but adds engine metadata to every record and
   allows a target to contain mixed transformation versions unless every write
   path enforces a transition protocol. Target-level metadata remains the
   authoritative source either way.
2. **Infer the transformation from the currently deployed manifest.** This is
   simpler, but a rollout could reinterpret an existing target before its
   documents are rebuilt, making rollback and diagnosis ambiguous.
3. **Pin the manifest hash, transformation version, and normalized digest per
   physical target (selected).** This keeps one semantic identity per target
   and fits the existing blue-green lifecycle, at the cost of a full target
   migration when transformation meaning changes.

## Consequences

- Target metadata and diagnostics identify the exact manifest and transformation
  semantics used to create a target.
- Transformation changes with persisted meaning require extra target storage,
  dual-write capacity, bootstrap, verification, and cutover time.
- No public per-document version field is needed for normal operation; an
  operator inspects the target's authoritative pin.
- Rolling evaluator compatibility remains a separate release concern under
  ADR-0023 and Q-068.
- Exact version-family separation remains unresolved under Q-057.

## Validation

- Identical normalized transformation definitions and allowlist identities
  produce the same target digest regardless of YAML or JSON formatting order.
- Reusing a transformation version with a changed digest blocks preflight before
  source consumption or progress.
- A semantic transformation change creates a new physical target and uses
  target-specific transforms during dual-write.
- Alias cutover is blocked until the new target's recorded identity, bootstrap,
  high-watermark, and canonical-output verification pass.
- A target's transformation identity can be inspected without adding a public
  marker to each projected document.

## Review triggers

Revisit if document-level provenance becomes an operational requirement, if a
runtime upgrade can change output without a versioned allowlist or migration, or
if target-level pinning cannot support the selected sink or migration workflow.

## Related concepts

- [Transformation contract](../02-contracts/transformation-contract.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Schema evolution](../04-data-lifecycle/schema-evolution.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Compatibility and dependencies](../05-quality-attributes/compatibility-and-dependencies.md)
- [Bloblang subset and resource budgets](0060-bloblang-subset-and-resource-budgets.md)
- [Follow-up register](../follow-ups.md) (Q-018 and Q-057)

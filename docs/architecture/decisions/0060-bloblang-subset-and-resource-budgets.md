---
type: Architecture Decision Record
title: "ADR-0060: Bloblang subset and resource budgets"
description: Defines deterministic Bloblang operations and evidence-based resource limits.
tags: [architecture, adr, transformations, capacity]
status: accepted
decision_id: ADR-0060
accepted_on: 2026-09-25
owner: Project maintainer
conditions:
  - Transformations operate only on the canonical assembled projection and use a deterministic allowlist.
  - Mapping complexity is checked before source progress; operations over collections have finite bounds.
  - Numeric runtime limits are established from representative profiles under Q-070 before production.
  - An inability to demonstrate bounded execution blocks production enablement for that workload.
---

# ADR-0060: Bloblang subset and resource budgets

## Status

Accepted on 2026-09-25. Owner: Project maintainer.

## Context

V1 uses one Bloblang-in-Go evaluator across live processing, bootstrap, audit,
repair, and migration ([ADR-0007](0007-transformation-execution-boundary.md)).
The same canonical input must produce the same output in each mode, and a
manifest must not be able to consume unbounded CPU or memory. Bloblang also
provides operations whose results can depend on the host, wall clock, external
files, randomness, or message metadata; those operations make replay and
cross-mode comparison unreliable.

At this architecture stage, no exact evaluator version or representative
production workload has been selected. The evaluator must be pinned per engine
release under ADR-0023, and transformation/allowlist identity is associated
with each target under
[ADR-0062](0062-transformation-version-target-association.md). Numeric parser,
byte, array, latency, and memory caps require those inputs, workload profiles,
and benchmark evidence under Q-069 and Q-070.

## Decision

Compile and validate each transformation before its workload begins source
progress. Use a deterministic, versioned allowlist over the canonical assembled
projection only. Use the same evaluator and rules in stream, bootstrap, audit,
repair, and migration.

The supported operation categories are:

- Read declared fields from the canonical context and construct output objects,
  arrays, and literals.
- Use arithmetic, comparisons, boolean logic, conditionals, and match
  expressions.
- Apply deterministic casts and string operations, and date operations whose
  format and timezone are explicit.
- Map, filter, or reduce finite input arrays. Any nested traversal or generated
  output must have a finite bound under the relation and document limits.
- Omit or delete a field only when the manifest declares that output field
  optional. Use missing/null defaults only when the manifest declares the input
  safe with missing context. Transformation errors are not hidden by broad
  catch-and-default behavior.

Do not allow transformations to read environment variables, files, network or
plugin state, the wall clock, random or unique-ID generators, host identity,
batch position, transport metadata, or the unassembled raw envelope. Do not
load mapping code dynamically or use a root-level delete operation that drops a
whole event while processing otherwise succeeds. The exact built-in function
and method names are tied to the pinned evaluator and its versioned allowlist;
target transformation identity follows
[ADR-0062](0062-transformation-version-target-association.md), and evaluator
compatibility follows ADR-0023.

Bound and measure these resource dimensions:

- Mapping source size, syntax-tree size/depth, and compile cost, checked before
  source progress.
- Kafka event bytes, canonical assembled input bytes, and output document bytes
  as distinct quantities.
- Relation cardinalities, array items traversed, and output array sizes.
- Transformation CPU/latency and aggregate worker memory/concurrency under each
  operating mode.

Do not choose numeric caps here. Establish and record them under Q-070 using the
representative workload profiles maintained under Q-069. Profile ordinary,
large-but-expected, stress, and recovery workloads. Set hard limits at the
largest tested workload that still meets the accepted freshness, recovery, and
resource objectives; expected workload values may warn below those hard limits.
The cap on raw CDC event size follows ADR-0048. Relation/document hard-limit
behavior follows ADR-0056: preserve valid source work and isolate the affected
scope instead of DLQing it as malformed data.

An unsupported or over-complex mapping blocks its manifest before consumption.
If the pinned evaluator cannot safely enforce or otherwise demonstrate the
execution bound, block production enablement for the affected workload until
the bound is proven. Never silently drop an event or advance its offset without
a definitive accepted outcome.

## Alternatives considered

1. **Allow the full Bloblang surface.** This is expressive, but can make output
   depend on machine state, time, external files, randomness, or transport
   context. It also makes the maximum work of a manifest difficult to review.
2. **Allow only declarative field copies.** This is easy to bound but cannot
   express common deterministic calculations such as unit conversion, defaults,
   and bounded aggregation.
3. **Allow deterministic data-only categories with versioned allowlists.** This
   retains common transformations while keeping replay semantics and resource
   review explicit. This is the v1 choice.

## Consequences

- Projection authors can express ordinary deterministic data shaping but must
  request deliberate review to add a new operation category.
- New evaluator versions do not silently expand the supported manifest
  language; target identity follows ADR-0062 and rolling compatibility changes
  follow ADR-0023 and Q-068.
- Numeric workload ceilings are production evidence, not constants guessed in
  the contract. Manifests outside a measured safe envelope remain blocked.
- Bootstrap and live processing can compare identical canonical outputs; the
  restricted language gives up host-, time-, and randomness-dependent values.
- Benchmark evidence must include transform latency and worker resources along
  with event, canonical-document, relation, and array sizes.

## Validation

- Compile valid and invalid manifests before source progress and verify
  unsupported operations are rejected at preflight.
- Run the same deterministic corpus through stream, bootstrap, audit, repair,
  and migration and compare canonical outputs.
- Verify field deletion requires a declared optional output and missing-value
  defaults require declared safe-missing semantics.
- Verify root-level event deletion, environment/file/clock/random access,
  transport metadata, and dynamic mapping loads are rejected.
- Benchmark all listed resource dimensions under Q-069 profiles and record the
  numeric limits and evidence under Q-070 before production.
- Verify raw event and valid relation/document hard-limit outcomes follow
  ADR-0048 and ADR-0056 respectively, with no silent offset advancement.
- Demonstrate a safe execution bound for the pinned evaluator before enabling a
  production manifest.

## Review triggers

Revisit if the evaluator changes its execution model, a required deterministic
operation cannot be expressed safely, representative workloads exceed the
accepted limits, or a new mode cannot preserve identical transformation
semantics.

## Related concepts

- [Transformation contract](../02-contracts/transformation-contract.md)
- [Manifest contract](../02-contracts/manifest-contract.md)
- [Performance and capacity](../05-quality-attributes/performance-and-capacity.md)
- [Performance and failure testing](../08-validation/performance-and-failure-testing.md)
- [Transformation execution boundary](0007-transformation-execution-boundary.md)
- [Follow-up register](../follow-ups.md) (Q-016, Q-018, Q-069, Q-070)

---
type: Architecture Decision Record
title: "ADR-0053: Canonical source ID serialization"
description: Defines typed, unambiguous serialization for MongoDB source entity identifiers.
tags: [architecture, adr, identity, serialization, mongodb, contracts]
status: accepted
decision_id: ADR-0053
accepted_on: 2026-09-24
owner: Project maintainer
conditions:
  - Canonical IDs use a type tag and an unambiguous length-framed payload; the manifest declares the expected ID kind.
  - ObjectID uses its 12 raw bytes; UUID uses 16 RFC/network-order bytes; strings preserve exact UTF-8 bytes; signed BSON int32 and int64 remain distinct types.
  - Unsupported types and mismatches fail closed; BSON legacy UUID subtype 3 is accepted only with an explicit byte-order configuration.
---

# ADR-0053: Canonical source ID serialization

## Status

Accepted on 2026-09-24. Owner: Project maintainer.

## Context

The engine needs one stable identity key for a source entity across event
decoding, per-entity freshness fences, reverse lookups, and replay. Converting
every `_id` value to a plain string can make distinct BSON types collide. For
example, the BSON string `"42"` and BSON integer `42` must not silently identify
the same entity. Delimiter-joined compound keys can also be ambiguous unless
each component is framed.

## Decision

- The projection manifest declares the expected source ID kind. The engine
  validates the incoming BSON or decoded event value against it; mismatched or
  unsupported values are unprocessable and follow the durable custody policy.
- Encode every ID as a type tag plus an unambiguous length-prefixed payload.
  Encode compound identity components separately; never form the canonical
  identity by naive delimiter joining.
- Canonical payloads are:
  - **ObjectID:** the 12 raw bytes. Its lowercase hexadecimal form is used for
    readable diagnostics, not as the canonical payload.
  - **UUID:** the 16 bytes in RFC/network byte order. Valid UUID text is
    normalized to lowercase hexadecimal-and-dash form for display, and maps to
    those same 16 bytes. BSON UUID subtype 4 uses this representation. BSON
    legacy subtype 3 is accepted only when the source configuration explicitly
    identifies its driver-specific byte order; otherwise it is unprocessable.
  - **String:** the exact UTF-8 bytes. Do not trim, case-fold, or Unicode
    normalize the value.
  - **Numeric:** signed BSON `int32` and `int64` values use distinct type tags
    and canonical base-10 ASCII payloads: no leading `+`, no leading zeroes
    except the value `0`, and a leading `-` only for negative values. Do not
    convert through floating point. Reject floating-point and decimal IDs until
    an explicit representation is accepted.
- Human-readable forms such as `oid:507f...`,
  `uuid:550e8400-e29b-41d4-a716-446655440000`, `str:507f...`, `int32:42`, and
  `int64:42` illustrate the distinct type tags; they do not replace the framed
  binary encoding.

## Alternatives considered

1. **Stringify all ID values.** This is easy to inspect, but can collapse
   different source types, such as string `"42"` and integer `42`, and can lose
   UUID byte-order information.
2. **Use BSON's encoded value unchanged.** This preserves types, but couples the
   engine's identity contract to the exact BSON encoding and does not by itself
   define how textual UUIDs or legacy subtype 3 values are interpreted.
3. **Use typed, framed canonical payloads.** This adds a small encoder and
   requires manifests to state the expected kind, but gives stable identity
   across event shapes and prevents ambiguous concatenations. This is accepted.

## Consequences

- Identity and freshness fences distinguish values that look alike but have
  different BSON types.
- UUID subtype 4 and valid UUID text converge on the same canonical bytes;
  subtype 3 requires source-specific configuration to avoid guessing byte
  order.
- String IDs retain exact source semantics, including case and whitespace.
- Manifests and event validation must carry or derive the declared ID kind.
  Existing data written under a different serialization cannot be assumed to
  share identity and may require a rebuild or migration.
- Equal-revision delivery behavior is defined by
  [ADR-0054](0054-equal-source-revision-conflicts.md); this decision defines
  entity identity but does not change source ordering.

## Validation

- ObjectID `507f1f77bcf86cd799439011` and string `"507f1f77bcf86cd799439011"`
  produce different canonical IDs because their type tags differ.
- String `"42"`, `int32(42)`, and `int64(42)` produce three different canonical
  IDs.
- Integer payloads for `42`, `0`, and `-42` are exactly `42`, `0`, and `-42`;
  the `int32` and `int64` type tags still make equal numeric values distinct.
- UUID text and the equivalent subtype-4 UUID bytes produce the same canonical
  UUID ID; subtype-3 bytes without configured byte order are rejected.
- String values differing only by case, surrounding whitespace, or Unicode
  normalization remain different IDs.
- A manifest expecting ObjectID rejects a string-valued `_id`; unsupported
  floating-point and decimal IDs are not coerced into integer IDs.
- Compound IDs remain unambiguous when component values contain delimiter
  characters because each encoded component is separately framed.

## Review triggers

Revisit if a supported source uses another BSON identifier type, a legacy UUID
byte order must be supported by default, or existing projections require a
serialization migration.

## Related decisions and concepts

- [ADR-0005: Identity and ordering scope](0005-identity-and-ordering-scope.md)
- [ADR-0052: MongoDB source-ordering scope](0052-mongodb-source-ordering-scope.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Follow-up register](../follow-ups.md) (Q-008, Q-009, Q-123)

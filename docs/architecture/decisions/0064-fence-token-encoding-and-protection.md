---
type: Architecture Decision Record
title: "ADR-0064: Fence token encoding and protection"
description: Defines privacy-preserving, deterministic entity-key and mutation-fingerprint tokens for derived Elasticsearch fences.
tags: [architecture, adr, security, privacy, elasticsearch, fencing]
status: accepted
decision_id: ADR-0064
accepted_on: 2026-09-25
owner: Project maintainer
conditions:
  - Entity keys derive from the canonical scoped source identity in ADR-0053; mutation fingerprints derive from the canonical logical source mutation in ADR-0054.
  - Use domain-separated HMAC-SHA-256 with dedicated secret key material, and store each 32-byte result as unpadded Base64URL.
  - Each physical target pins a nonsecret key ID and token-format version in private engine_meta; raw identifiers and keys are never stored in Elasticsearch.
  - Key rotation is target-aware and uses blue-green migration; retain old key material until every target that uses it is retired.
  - HMAC tokens remain pseudonymous data; legal retention treatment remains under FU-104/Q-076.
---

# ADR-0064: Fence token encoding and protection

## Status

Accepted on 2026-09-25. Owner: Project maintainer.

## Context

[ADR-0055](0055-derived-elasticsearch-fence-metadata.md) puts a rebuildable
fence copy in Elasticsearch so one scripted write can atomically compare an
incoming source revision with the last mutation for that contributor. The
entity key must identify the same scoped source entity on every delivery. The
mutation fingerprint must distinguish a repeated delivery of the same logical
mutation from different content at an equal revision under
[ADR-0054](0054-equal-source-revision-conflicts.md).

These values should not expose source identifiers to someone who can inspect an
Elasticsearch document. Plain hashes would still let an observer enumerate
low-entropy identifiers such as small numeric IDs. The engine needs a
deterministic comparison token, not a reversible copy of the source ID.

## Decision

- Derive the entity-key input from the type-tagged, length-framed canonical
  scoped source identity defined by
  [ADR-0053](0053-canonical-source-id-serialization.md), including the source,
  replica-set, database, collection, and typed entity ID scope.
- Derive the mutation-fingerprint input from the canonical logical source
  mutation used by ADR-0054. It includes the source mutation's logical content
  and excludes Kafka coordinates, connector processing timestamps, and other
  delivery-only metadata.
- Compute each token with HMAC-SHA-256 using dedicated secret key material and
  a distinct, stable domain-separation label for entity keys and mutation
  fingerprints. Encode the 32-byte result as unpadded Base64URL (43 characters).
  The two token types cannot be substituted for one another.
- Store only the tokens in fence entries. At the target's root `engine_meta`,
  store a token-format version and a nonsecret key ID that identifies the HMAC
  secret version used by that physical target. Do not store the raw canonical
  identity, HMAC key, or an unkeyed copy of either value in Elasticsearch.
- Deliver the dedicated HMAC key through the existing managed-secret boundary
  in [ADR-0022](0022-security-and-privacy.md). A target remains pinned to one
  key ID. During blue-green migration, the existing target continues using its
  key while the candidate target uses its own pinned key. Keep both keys
  available through dual-write and rollback; remove an old key only after no
  retained target or rollback path depends on it.
- Keep `engine_meta` out of public APIs and ordinary telemetry. Treat HMAC
  tokens as pseudonymous, linkable values, not anonymous data. The legal
  retention treatment of identifiers in fences remains deferred under Q-076.

For example, suppose source entity `agency A17` has revision `(1700000000000,
7)` and mutation `name = "North branch"`. Its scoped identity produces a stable
entity token for that target's key. Redelivery of the same logical mutation
produces the same fingerprint, so an equal-revision write is a duplicate
no-op. If another event claims the same entity and revision but says
`name = "South branch"`, it produces a different fingerprint; the engine keeps
the current projection and follows ADR-0054's durable entity-local repair path.
An Elasticsearch reader can see that tokens match or differ, but cannot recover
`A17` or either name without the secret key.

When rotating from key `fence-v7` to `fence-v8`, the existing physical target
continues to use `fence-v7` and the new blue-green target records `fence-v8`.
The same source event therefore has different opaque tokens in the two targets;
both targets remain comparable within their own fence history. Rollback returns
to the old target and its still-available key.

## Alternatives considered

1. **Store canonical identifiers directly.** This makes inspection and manual
   repair easier, but exposes source identities in every Elasticsearch copy
   and expands access and retention risk.
2. **Store an unkeyed hash.** This avoids readable raw IDs, but an observer can
   hash likely low-entropy values and match them against stored tokens.
3. **Encrypt the values deterministically.** This is reversible and would
   require decryption access for routine comparisons, adding key-handling
   complexity the fencing script does not need.
4. **Use domain-separated HMAC tokens (selected).** This preserves stable
   equality and hides values from readers without the secret, while requiring
   explicit key custody and target-aware rotation.

## Consequences

- Repeated deliveries remain comparable within a target without persisting raw
  identifiers or source content in fence tokens.
- Different fence-token purposes cannot collide through accidental reuse of
  the same input domain; the output length and representation are fixed.
- Readers with access to an Elasticsearch target can still observe equality
  and changes between its tokens. If the same key and identity scope are used
  across targets, matching entity tokens can also be correlated across those
  targets. Access controls remain necessary.
- Key loss prevents safe writes against a target whose HMAC key is unavailable;
  restore or rebuild must provide the target's pinned key. Rotation requires
  temporary access to both old and new key versions and adds blue-green
  migration coordination.
- HMAC protection does not settle whether fence tokens may be retained after
  source anonymization or deletion. Q-076 remains the legal and retention gate.

## Validation

- The same typed scoped identity, token-format version, and key produce the
  same entity token; distinct type-tagged identities do not collapse.
- The same logical source mutation produces the same fingerprint even when
  Kafka coordinates or connector processing metadata differ.
- A changed logical mutation produces a different fingerprint and reaches the
  equal-revision conflict path defined by ADR-0054.
- Entity-key and mutation-fingerprint domain labels produce distinct tokens
  for equal input bytes.
- Tokens are 43-character unpadded Base64URL encodings of 32-byte HMAC-SHA-256
  outputs; tests and fixtures contain no production keys.
- Elasticsearch fence metadata contains the format version and nonsecret key
  ID but no raw entity ID or HMAC key; public responses omit `engine_meta`.
- During a blue-green rotation, each physical target accepts comparisons only
  using its pinned key, and rollback remains possible until the old target is
  retired.

## Review triggers

Revisit if canonical source-mutation encoding changes, the token format or
algorithm needs migration, a target cannot retain its key through the rollback
window, or privacy requirements prohibit deterministic equality correlation.

## Related concepts

- [Projection schema](../02-contracts/projection-schema.md)
- [Security and privacy](../05-quality-attributes/security-and-privacy.md)
- [Identity, time, and ordering](../02-contracts/identity-time-ordering.md)
- [Equal source revision conflicts](0054-equal-source-revision-conflicts.md)
- [Derived Elasticsearch fence metadata](0055-derived-elasticsearch-fence-metadata.md)
- [Canonical source ID serialization](0053-canonical-source-id-serialization.md)
- [Blue-green migration](../04-data-lifecycle/blue-green-migration.md)
- [Follow-up register](../follow-ups.md) (Q-124, Q-076)

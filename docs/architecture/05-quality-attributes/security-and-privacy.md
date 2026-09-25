---
type: Architecture Review Topic
title: Security and privacy
description: Defines least privilege, secret handling, data minimization, and erasure behavior.
tags: [quality, security, privacy, compliance]
status: accepted
decision_id: ADR-0022
accepted_on: 2026-08-31
owner: TBD
conditions:
  - Manifest fields declare Internal, Confidential, Restricted, or Secret sensitivity; undeclared fields are rejected.
  - Restricted fields are not candidates for Redis or Elasticsearch by default; explicit future exceptions require a documented product need.
  - Credentials and KMS-provided encryption material arrive through environment variables; rotation restarts workers for now, while interfaces must not preclude future live rotation.
  - One authorized operator model is sufficient initially; privileged actions are audited and stricter role separation may evolve later without requiring two-person approval.
  - Source anonymization propagates through the normal CDC and projection path; cached and retained derived content uses time-to-live expiry, while deletion fences are retained for at least 30 days under the accepted residual-risk policy (ADR-0050).
  - Retention relationships are accepted but exact calendar durations remain a later operational/compliance decision.
  - Legal treatment of identifiers retained in deletion fences is explicitly deferred.
---

# Security and privacy

## Decision

Use four data classifications: Internal, Confidential, Restricted, and Secret.
Every manifest field declares its classification; undeclared fields are
rejected. Redis and Elasticsearch receive only explicitly allowed fields, with
Restricted fields blocked by default and Secret values never projected. Logs and
metrics contain operational metadata only and never raw envelopes, credentials,
or sensitive values.

Use least-privilege identities per mode and environment. Stream workers may read
assigned Kafka topics and permitted source collections, write required derived
stores, and update engine metadata. Bootstrap, audit, repair, replay, and
migration operations require explicit authorization and durable audit records.
One authorized operator model is sufficient initially; future role separation is
left open, and two-person approval is not required.

Use encrypted transport and managed encryption at rest where supported. Secrets
and KMS-provided encryption material are supplied through environment variables,
never committed to manifests. Workers restart to rotate values for now; the
configuration boundary must remain evolvable toward live rotation without
designing that mechanism prematurely.

When authoritative MongoDB state is anonymized, the anonymization flows through
the normal CDC, transformation, and projection path. Cached and retained derived
content uses time-to-live expiry or earlier redaction where required. A minimal
non-content deletion fence may outlive erased content for the replay and
resurrection-protection horizon. Historical raw DLQ payloads, logs, and backups
are therefore governed by their own retention and redaction policies.

Deletion fences remain for at least 30 days after durable persistence. Raw
replay after that window is unsupported; an older recovery must rebuild from
current authoritative state before writes resume. The accepted policy carries a
residual risk of resurrection if an older event nevertheless arrives after its
fence expires ([ADR-0050](../decisions/0050-deletion-fence-retention-policy.md)).
DLQ and audit retention durations remain operational follow-ups, and the legal
treatment of identifiers retained in fences is deferred.

## Pros

- Limits the blast radius of a generic cross-domain data engine.
- Makes privacy deletion part of lifecycle correctness.
- Reduces accidental sensitive-data leakage through diagnostics.
- Makes source anonymization converge through the same tested data path.

## Cons and risks

- Fine-grained credentials and field policies increase operational overhead.
- Redaction can make poison-event debugging harder.
- Replicated projections expand the data inventory that must be governed.
- Restart-based key rotation creates a short operational interruption until live
  rotation is designed.

## Alternatives considered

1. Copy complete raw payloads into every store and rely on access controls. This
   simplifies debugging but expands breach impact, retention obligations, and
   erasure complexity.
2. Introduce a separate encrypted object store for all DLQ payloads. This may
   help with large payloads but adds another authority and recovery dependency;
   MongoDB remains the initial DLQ custody boundary.
3. Require strict role separation and two-person approval from the first
   release. This improves separation of duties but adds operational friction
   without a current requirement.

## Consequences

- Field sensitivity, destination policy, and retention become manifest and
  operational concerns rather than implicit code behavior.
- Source anonymization updates projections normally, while old retained copies
  still require TTL, redaction, or protected retention.
- Environment-provided key rotation currently requires worker restarts; future
  live rotation remains possible.
- One operator can perform privileged actions, but every action and protected
  payload access must be attributable and auditable.
- Privacy-driven expiry may require a later refinement of required reverse-index
  retention and deletion-fence identifier handling.

## Validation

- Unauthorized identities cannot read or write outside their scope.
- Secrets, raw sensitive payloads, and Restricted fields do not appear in logs,
  metrics, or default Redis/Elasticsearch outputs.
- Source anonymization removes or updates content across derived stores and
  retained failure data according to the configured retention policy.
- Replay remains possible from protected DLQ custody without exposing raw data in
  diagnostics.
- Worker restart rotation loads new environment-provided key material safely.
- Privileged actions and protected-data access are attributable and auditable.

## Review trigger

Revisit if Restricted data becomes required in Redis or Elasticsearch, if source
anonymization no longer covers authoritative privacy changes, if retention or
compliance rules change, or if restart-based key rotation threatens availability.

## Related concepts

- [Manifest contract](../02-contracts/manifest-contract.md)
- [CDC event envelope](../02-contracts/cdc-event-envelope.md)
- [Deletion and replay](../02-contracts/deletion-and-replay.md)
- [Durable state ownership](../01-system-context/durable-state-ownership.md)
- [Cache and reverse lookups](../03-runtime/cache-and-reverse-lookups.md)
- [Elasticsearch writes](../03-runtime/elasticsearch-writes.md)
- [Disaster recovery](../04-data-lifecycle/disaster-recovery.md)
- [Reconciliation and repair](../04-data-lifecycle/reconciliation-and-repair.md)

## Follow-up questions

- What exact fields, if any, receive a future Restricted-field exception?
- What calendar retention values satisfy operational and compliance requirements?
- How should required reverse indexes handle privacy-driven expiry?
- What is the legal treatment of identifiers retained in deletion fences?
- When should operator permissions evolve into separated roles?

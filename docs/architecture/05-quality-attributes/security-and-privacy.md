---
type: Architecture Review Topic
title: Security and privacy
description: Defines least privilege, secret handling, data minimization, and erasure behavior.
tags: [quality, security, privacy, compliance]
status: proposed
---

# Security and privacy

## Decision to stamp

Define trust boundaries and controls for broad source reads, Kafka payloads,
caches, projections, telemetry, and DLQ records.

## Draft proposal

Use separate least-privilege identities per mode and environment, encrypted
transport, secret references supplied by the deployment platform, field-level
manifest allowlists, and auditable migration/repair actions. Redact or reference
rather than copy sensitive raw payloads into logs and DLQs. Propagate source
deletions to caches, indices, tombstones, and retained failure data under an
explicit policy.

## Pros

- Limits the blast radius of a generic cross-domain data engine.
- Makes privacy deletion part of lifecycle correctness.
- Reduces accidental sensitive-data leakage through diagnostics.

## Cons and risks

- Fine-grained credentials and field policies increase operational overhead.
- Redaction can make poison-event debugging harder.
- Replicated projections expand the data inventory that must be governed.

## Questions to stamp

- Which data classifications may enter Redis, Elasticsearch, logs, and DLQ?
- What erasure and retention obligations apply?
- Who may launch bootstrap, repair, replay, and migration modes?

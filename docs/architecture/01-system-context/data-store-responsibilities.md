---
type: Architecture Review Topic
title: Data-store responsibilities
description: Assigns authority, durability, and recovery roles to each datastore.
tags: [context, kafka, mongodb, redis, elasticsearch]
sources:
  - resource: ../../design/system.md
    title: System design draft
status: in-review
---

# Data-store responsibilities

## Decision to stamp

Define which system is authoritative for data, delivery, operational state,
projection persistence, and recovery.

## Draft proposal

- The [Redis authority boundary](redis-authority-boundary.md) is accepted; the
  exact durable ownership is resolved by [ADR-0003](../decisions/0003-durable-state-ownership.md).
- MongoDB is the business source of truth and authoritative rebuild source.
- Kafka is the durable live-change log within a declared retention window.
- Redis accelerates lookups and reverse relations but is not the sole durable
  owner of migration or progress state.
- Elasticsearch stores disposable, versioned read models exposed through aliases.

The accepted ownership matrix keeps Kafka offsets authoritative for the consumer
cursor and assigns migration, control-plane, and DLQ state to an engine-owned
MongoDB metadata store. Redis and Elasticsearch remain rebuildable derived state.

## Pros

- Clear recovery paths follow from explicit authority.
- Cache loss does not erase control-plane truth.
- Elasticsearch can be rebuilt without becoming a business source.

## Cons and risks

- A durable control-plane store or recoverable representation is still needed.
- Kafka retention limits how long a stream-only recovery can work.
- Rebuilding Redis relations from source data may be costly.

## Questions to stamp

- Which reverse relations are caches versus required indexes?
- What happens when Kafka and MongoDB temporarily disagree?

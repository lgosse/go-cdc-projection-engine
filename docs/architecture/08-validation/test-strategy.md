---
type: Architecture Review Topic
title: Test strategy
description: Defines layered verification for manifests, event histories, stores, and operational workflows.
tags: [validation, testing, contracts]
status: proposed
---

# Test strategy

## Decision to stamp

Define the minimum automated test layers and production-like dependencies needed
for confidence.

## Draft proposal

Use unit tests for canonicalization, graph validation, reducers, and error
classification; schema and compatibility tests for manifests and CDC envelopes;
property tests over duplicate/reordered histories; integration tests with real
Kafka, MongoDB, Redis, and Elasticsearch versions; and end-to-end tests for
bootstrap, delete, migration, repair, crash, and replay workflows.

## Pros

- Matches test type to the boundary it can actually prove.
- Real-store tests catch script, mapping, offset, and protocol behavior mocks miss.
- Generated histories exercise commutativity claims systematically.

## Cons and risks

- A four-store suite is slow and operationally heavy.
- Property tests need a trustworthy reference model.
- Version-matrix testing can dominate CI time.

## Questions to stamp

- Which scenarios must run per change versus nightly?
- What source data model serves as the first end-to-end fixture?
- Which compatibility versions are release-blocking?

---
type: Architecture Review Topic
title: Runbooks and intervention
description: Defines the minimum operator procedures required before production use.
tags: [operations, runbooks, incident-response]
status: proposed
---

# Runbooks and intervention

## Decision to stamp

Define which failure and maintenance scenarios need rehearsed, auditable
operator procedures.

## Draft proposal

Require runbooks for lag growth, dependency outage, cache loss/warm-up, blocked
schema, partial dual-write, failed bootstrap, unsafe alias state, drift spike,
DLQ triage/replay, hot partition/document, credential rotation, and data erasure.
Each runbook must include diagnosis, safe actions, rollback, evidence collection,
and escalation ownership.

## Pros

- Converts failure handling from implicit code behavior into shared operations.
- Clarifies which automated actions require human approval.
- Provides acceptance material for game-day exercises.

## Cons and risks

- Runbooks decay unless incidents and releases update them.
- Too many narrowly scoped procedures can be hard to navigate under pressure.

## Questions to stamp

- Who is on call for engine versus source-domain failures?
- Which actions are reversible and which require two-person approval?
- Where will operational evidence and incident history live?

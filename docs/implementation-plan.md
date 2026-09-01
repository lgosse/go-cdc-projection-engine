---
type: Project Implementation Plan
title: Implementation roadmap
description: Phased path from the accepted architecture baseline to a first production release.
tags: [implementation, roadmap, architecture, delivery]
status: active
owner: Project maintainer
---

# Implementation roadmap

The architecture decisions are accepted, but the repository intentionally has
no implementation yet. This plan converts those decisions into gated work. It
does not select package names, interfaces, libraries, or deployment templates;
those choices are made in the design checkpoint that precedes each substantial
implementation step.

## Working principles

- Accepted ADRs define non-negotiable boundaries and guarantees.
- The original drafts under [`docs/design/`](design/) remain unchanged
  provenance; they are not silently upgraded into implementation requirements.
- One representative projection proves the engine before broad projection
  coverage is attempted.
- Every new ambiguity enters the [follow-up register](architecture/follow-ups.md)
  before code relies on an assumption.
- A milestone cannot close while an applicable blocking follow-up is unresolved.
- Implementation designs must preserve reusable, domain-neutral engine logic;
  the first projection is a proving case, not the architecture's owner.

## Agent workflow

Agents working in this repository must first read the root
[`AGENTS.md`](../AGENTS.md), this roadmap, and the applicable
[`docs/architecture/AGENTS.md`](architecture/AGENTS.md). Architecture tasks use
the local review skill and its one-topic decision-card workflow. Implementation
tasks consult the [follow-up register](architecture/follow-ups.md), resolve
applicable blocking items before the milestone gate, and write a design note
before substantial slice code. Agents must report changed files, validation,
unverified areas, and newly discovered follow-ups; they must not silently turn
an open question into an implementation assumption.

## Step 1 — Freeze the architecture baseline

### What “clean architecture baseline commit” means

It is a Git commit containing the reviewed repository and documentation state
before implementation begins. It establishes the exact architectural starting
point so later code changes can be compared against accepted decisions.

### Included

- Repository initialization files such as [`README.md`](../README.md) and
  [`.gitignore`](../.gitignore).
- The complete linked architecture tree and accepted ADRs.
- The local [architecture review skill](../.skills/architecture-decision-review/SKILL.md).
- The [follow-up register](architecture/follow-ups.md) and this roadmap.
- Decision indexes, section indexes, and the review log.
- The example CDC envelope asset used as contract provenance.

### Excluded

- Go source code, module dependencies, package layout, generated files, and
  Kubernetes manifests.
- Unreviewed implementation assumptions.
- Edits to the original source drafts.

### Procedure

1. Inspect the working tree and separate intended architecture changes from
   unrelated local edits.
2. Confirm every architecture concept is accepted or intentionally marked as a
   process/template document, and review the follow-up register.
3. Run frontmatter, relative-link, heading/status, and whitespace validation.
4. Stage only the intended repository-initialization and architecture files.
5. Create a descriptive commit, for example `docs: establish architecture
   baseline`.
6. Record the resulting commit hash and date in the project history or release
   notes. Creating a Git tag is optional and should only be done if release
   conventions require one.

### Completion gate

- The worktree contains no accidental generated or source files.
- The architecture links and decision register resolve.
- The follow-up register has an owner and gate for every unresolved question.
- The baseline commit is reproducible from a fresh checkout.

The commit itself is not created by this document; it is the explicit handoff
between architecture discovery and implementation preparation.

## Step 2 — Prepare concrete contracts

Define and review the versioned artifacts needed by every runtime mode:

- manifest structure and semantic validation;
- canonical event representation while preserving the original Debezium
  envelope;
- identity, source revision, fence, error-code, and terminal-outcome rules;
- engine-owned MongoDB metadata inventory and retention boundaries;
- Redis relation policies and Elasticsearch target/mapping contracts; and
- a representative projection fixture.

This step produces contract documents and fixtures, not a production runtime.
It closes the follow-ups gated on the contract package and records any new
questions before the vertical slice starts.

## Step 3 — Design and build one vertical slice

The vertical slice is deliberately split so “end to end” does not become “put
all logic in one application package.”

### Step 3a — Vertical-slice implementation design

Before writing slice code, create a short design note and review it against the
accepted ADRs. It must describe, without prematurely fixing syntax or package
names:

- the responsibility boundary of the reusable engine core;
- the seams between source delivery, canonical event handling, relationship
  context, transformation, fencing, target writes, custody, and offset
  completion;
- which behavior is shared across modes and which belongs to a mode or adapter;
- dependency direction and ownership of state, retries, and observability;
- how a second manifest or projection can reuse the same logic; and
- the tests that prove the boundaries rather than only the happy path.

This design note is a milestone artifact. If it reveals a choice not covered by
an accepted ADR, add or resolve a follow-up before implementation proceeds.

### Step 3b — Thin end-to-end proving path

Implement one representative projection through the smallest complete path:

`Kafka → envelope handling → identity/fencing → transformation → Elasticsearch
write → durable outcome → contiguous offset completion`

The slice must include the accepted safety boundaries from the beginning:
startup preflight, engine-owned MongoDB custody, Redis policy handling,
Elasticsearch fenced writes, health signals, and durable DLQ behavior. It should
prove real failure outcomes, not only successful indexing.

Implementation details remain intentionally deferred to Step 3a. In particular,
this plan does not prescribe package layout, interface shapes, client libraries,
or concurrency primitives.

### Step 3c — Reuse and separation checkpoint

After the first path works, exercise the same reusable logic with a second
manifest variation or small synthetic projection fixture. This is not a promise
to launch a second production projection; it is an architectural fitness test
that detects domain-specific logic, hidden global state, and mode coupling while
change is still cheap.

### Vertical-slice completion gate

- Contract and applicable follow-ups are resolved.
- Duplicate, out-of-order, delete, malformed-event, cache-miss, partial-write,
  restart, and rebalance behaviors have evidence.
- The second-manifest reuse checkpoint passes without copying engine logic.
- The design note and evidence link back to the relevant ADRs.

## Step 4 — Add lifecycle modes

Extend the proven core in this order:

1. Bootstrap with source cluster-time boundary and live dual-write.
2. Blue-green migration, verification, alias cutover, rollback, and retirement.
3. Reconciliation and repair.
4. DLQ inspection and replay.
5. Audit and validation jobs.
6. Explicit migration orchestration through Kubernetes Jobs.

Do not begin with broad multi-projection support or a permanent migration
controller. Each mode receives its own design checkpoint and milestone gate.

## Step 5 — Evidence and production hardening

Add the layered tests, workload benchmarks, targeted dependency failures,
observability dashboards, authenticated diagnostics, runbooks, metadata backup
and restore evidence, and Kubernetes workload policies required by the accepted
ADRs. The first release gate is one representative projection exercising the
complete stream, bootstrap, migration, repair, replay, and reconciliation
lifecycle.

## Immediate next action

Complete Step 1's baseline review and populate the follow-up register. Then
design Step 2's contract package before selecting implementation structure.

## Related documents

- [Architecture workspace](architecture/index.md)
- [Follow-up register](architecture/follow-ups.md)
- [Architecture review method](architecture/review-method.md)
- [Decision register](architecture/decisions/index.md)

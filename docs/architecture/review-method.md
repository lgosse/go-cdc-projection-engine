---
type: Architecture Review Process
title: Architecture review and stamping method
description: Defines how provisional architecture concepts become accepted decisions.
tags: [architecture, review, governance]
sources:
  - resource: ../design/system.md
    title: System design draft
  - resource: ../design/otel.md
    title: OpenTelemetry design draft
status: accepted
decision_id: ADR-0044
accepted_on: 2026-09-01
owner: Project maintainer
conditions:
  - One contributor may own the full review loop, but every decision and follow-up remains explicit and traceable.
  - Blocking follow-ups must be resolved before the milestone that depends on them; deferrals require an owner, trigger, and safe boundary.
  - Implementation details are not inferred from accepted boundaries; they receive a design checkpoint before code depends on them.
---

# Architecture review and stamping method

## Decision

Review one bounded architecture concept at a time. Before a decision is
accepted, present its question, evidence, proposal, alternative, trade-offs,
dependencies, open blockers, and acceptance evidence. Require an explicit user
disposition: accept, accept with conditions, modify, reject, defer, or
supersede.

After acceptance, update the concept, create a numbered ADR, update indexes and
the review log, and run frontmatter, relative-link, heading/status, and
whitespace checks. Keep the original drafts under `docs/design/` unchanged as
provenance.

Maintain every unresolved question in the [follow-up register](follow-ups.md).
Each entry has a scope, owner, status, dependency or milestone gate, trigger,
and resulting decision or evidence link. A follow-up marked blocking cannot be
passed silently: the dependent implementation milestone stays closed until it
is answered or explicitly re-scoped. A deferred item remains visible with a
safe interim rule and a concrete resume trigger.

The project maintainer is the sole decision owner and contributor for now. The
maintainer performs the same evidence and traceability checks that a larger
team would require, and may request an external review when risk or domain
ownership warrants it; a second approval is not mandatory.

Accepted architecture boundaries do not prescribe package layout or internal
mechanisms. Before implementation of a substantial slice, write and review a
design note that maps responsibilities, seams, state ownership, failure
boundaries, observability, and reuse expectations to the accepted decisions.

Statuses are `proposed`, `in-review`, `accepted`, `superseded`, and `rejected`.
Follow-up entries additionally use `open`, `in-progress`, `deferred`, and
`resolved`.

## Benefits

- Keeps decisions small, discoverable, and independently editable.
- Makes uncertainty and unresolved dependencies visible.
- Separates source-draft proposals from reviewed architecture.

## Risks and controls

- Requires disciplined cross-link, follow-up, and status maintenance.
- A single maintainer can miss a concern; evidence checklists, milestone gates,
  and optional external review provide compensating controls.
- Concepts reviewed in isolation can hide system-level trade-offs; related ADR
  links and the follow-up register make dependencies visible.

## Stamp checklist

- State the decision and non-goals precisely.
- Record at least one viable alternative and why it was not selected.
- Identify correctness invariants and operational failure modes.
- Define measurable acceptance evidence.
- Check linked concepts for consequences or contradictions.
- Record an owner and a future review trigger.

## Implementation checkpoint

Before code enters a milestone:

1. Identify the accepted ADRs and follow-ups that govern the milestone.
2. Resolve every blocking follow-up or narrow the milestone so it no longer
   depends on that question.
3. Review the implementation design note for responsibility boundaries,
   dependency direction, reusable engine behavior, and test seams.
4. Record the evidence and any newly discovered follow-ups.

At milestone completion, no blocking follow-up may remain unacknowledged for the
delivered scope. Deferred questions must retain an explicit safe behavior and a
resume trigger.

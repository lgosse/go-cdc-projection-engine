---
type: Architecture Decision Record
title: "ADR-0044: Architecture review process"
description: Defines how accepted architecture decisions and implementation follow-ups are governed by a single maintainer.
tags: [architecture, adr, governance, process]
status: accepted
accepted_on: 2026-09-01
owner: Project maintainer
---

# ADR-0044: Architecture review process

## Status

Accepted on 2026-09-01. Owner: Project maintainer.

## Context

The project has a complete provisional architecture tree and a single
contributor until at least the first release. A lightweight process must still
prevent silent assumptions, forgotten ADR follow-ups, and implementation
details that accidentally harden an accepted boundary.

## Decision

Review one bounded concept at a time using a decision card containing the
question, evidence, proposal, alternative, trade-offs, dependencies, open
blockers, and acceptance evidence. Require an explicit disposition before
stamping a decision.

For an accepted decision, update the concept, create a numbered ADR, update
indexes and the log, and run frontmatter, link, heading/status, and whitespace
checks. Preserve the original drafts under `docs/design/` unchanged.

Track every unresolved implementation question in the [follow-up register](../follow-ups.md).
Each item has an owner, scope, status, dependency or milestone gate, and resume
trigger. Blocking items must be resolved before the dependent milestone closes,
or the scope must be narrowed explicitly. Deferred items retain a safe interim
behavior and a concrete trigger.

The project maintainer is the sole decision owner and contributor for now. The
maintainer performs the evidence and traceability checks directly and may seek
external review when risk or domain ownership warrants it; a second approval is
not mandatory.

Accepted ADRs define boundaries and guarantees, not package layout or internal
mechanisms. A design checkpoint must precede implementation of each substantial
slice and must describe responsibility boundaries, dependency direction, state
ownership, failure handling, observability, reuse, and tests.

## Alternatives considered

1. **Informal issue-only tracking.** Easy initially, but questions disappear
   from the architecture record and can be missed when implementation moves
   quickly.
2. **Require a second approver for every decision.** Adds a control, but is
   incompatible with the current single-contributor team and is not necessary
   when evidence and gates are explicit.
3. **Prescribe implementation structure in every ADR.** Reduces short-term
   ambiguity but hardens mechanisms before the vertical-slice design proves
   their reuse and failure behavior.

## Consequences

- One maintainer can move the project without waiting for an artificial second
  approval while retaining an auditable decision trail.
- The follow-up register and milestone gates become part of the definition of
  done for implementation work.
- Design notes are required before substantial code, adding a small planning
  cost in exchange for reusable boundaries.
- External review remains optional and risk-driven.

## Validation

- Every accepted concept has a linked ADR, index entry, and log entry.
- Every known follow-up appears in the register with owner, class, and gate.
- A milestone cannot close with an applicable unresolved blocking item.
- Vertical-slice design notes map implementation choices to accepted ADRs.
- Deferred items retain safe interim behavior and resume triggers.

## Review triggers

Revisit if the project gains additional maintainers, release work repeatedly
passes unresolved blocking questions, external governance becomes mandatory, or
the register no longer provides useful traceability.

## Related concepts

- [Architecture review method](../review-method.md)
- [Implementation roadmap](../../implementation-plan.md)
- [Follow-up register](../follow-ups.md)
- [Ownership and governance](../07-operations/ownership-and-governance.md)
- [Release acceptance](../08-validation/release-acceptance.md)

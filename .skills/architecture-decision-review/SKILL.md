---
name: architecture-decision-review
description: Review this project's architecture concepts interactively, one bounded decision at a time, and record accepted outcomes without implementing code.
metadata:
  short-description: Review architecture decisions interactively
---

# Architecture Decision Review

Use this skill when reviewing the Markdown concepts under
`docs/architecture/` in this repository. The goal is to turn provisional
architecture proposals into explicit, reviewable decisions with the user. Do
not turn an accepted decision into source code, package layout, dependencies,
or deployment configuration unless the user separately asks for implementation.

## Working agreement

- Keep the original drafts under `docs/design/` unchanged; they are provenance,
  not the decision record.
- Work on one decision topic at a time. A topic is one concept document, not a
  whole directory or an entire subsystem.
- Prefer evidence from the current repository and linked concepts. Label an
  inference as an inference and preserve unknowns instead of inventing domain
  facts.
- Surface contradictions and dependencies before asking for a stamp. Do not
  silently reconcile conflicting requirements.
- Keep proposals implementation-neutral until the decision genuinely requires a
  mechanism. When mechanisms are discussed, compare at least one credible
  alternative.
- Never mark a topic accepted merely because the current draft sounds plausible.

## Discovery and queue

At the beginning of a review session:

1. Read `docs/architecture/index.md`, `review-method.md`, and the relevant
   section index. Read the current concept and its direct related links before
   presenting it.
2. Inspect the working tree so existing user edits are preserved.
3. If the user did not name a topic, propose the next unaccepted topic using
   dependency order. Prefer this sequence unless the user chooses otherwise:
   source conflicts; objectives and guarantees; datastore responsibilities; CDC
   envelope; identity and ordering; relationships; transformations; deletion;
   projection schema; runtime delivery and writes; bootstrap and migrations;
   quality attributes; observability; operations; validation.
4. Check the decision register for an existing ADR that supersedes or constrains
   the topic. Do not duplicate an accepted decision.

## Decision card

Before asking the user to decide, present a compact decision card containing:

1. **Decision question** — one sentence with a bounded choice.
2. **Why now** — the dependency or downstream consequence that makes this topic
   ready.
3. **Evidence** — relevant statements or examples from the source drafts and
   architecture concepts, with links. Distinguish stated facts from inferences.
4. **Current proposal** — a short, implementation-neutral option derived from
   the drafts.
5. **Alternatives** — at least one credible alternative, including the cost of
   deferring where useful.
6. **Trade-offs** — concrete pros, cons, risks, and affected concepts.
7. **Stamp questions** — only the smallest set of unanswered questions needed
   for a decision.
8. **Acceptance evidence** — what must later be tested, observed, or documented
   to show the decision is sound.

End with one explicit request for the user's disposition. Do not ask a list of
unrelated architecture questions in the same turn.

## User dispositions

Interpret the user's response as one of these dispositions, asking a concise
clarification only when it cannot be mapped safely:

- **Accept** — the user agrees with a bounded decision.
- **Accept with conditions** — record the decision and its explicit follow-up
  constraints; split unresolved material choices into new topics.
- **Modify** — restate the changed decision and show its consequences before
  recording it.
- **Reject** — record the rejected proposal and rationale; retain the topic as
  open until a replacement is selected.
- **Defer** — record the blocker, dependency, owner, and trigger for resuming;
  do not treat deferral as acceptance.
- **Supersede** — link the old ADR and explain what changed; never erase the
  historical decision.

## Recording an accepted decision

Only after the user has explicitly accepted a bounded decision:

1. Update the concept's YAML `status` to `accepted` and replace its provisional
   decision section with the agreed decision, boundaries, rationale, and explicit
   non-goals. Keep a short record of alternatives and consequences.
2. Add a numbered ADR under `docs/architecture/decisions/` using the structure in
   `decisions/template.md`. Include context, decision, alternatives, consequences,
   validation evidence, owner, review triggers, and links to affected concepts.
3. Update the relevant section index and `decisions/index.md` so the decision is
   discoverable. Add or correct ordinary Markdown links; do not use wiki-link
   syntax.
4. Mark dependent concepts `in-review` only when the accepted decision actually
   unblocks their review. Do not mass-update unrelated documents.
5. Run focused checks: frontmatter validity, relative-link resolution, changed
   headings/statuses, and `git diff --check`. Report what was checked and what
   remains unverified.

For `modify`, `reject`, or `defer`, record only the smallest durable note needed
to preserve the user's intent and next action. Do not create an ADR claiming an
accepted decision.

## Quality gate before stamping

A decision is ready to accept only when:

- the scope and non-goals are explicit;
- the selected option and at least one alternative are understandable;
- correctness, operational, security/privacy, and cost consequences have been
  considered where relevant;
- dependencies and unresolved assumptions are named;
- acceptance evidence and a review trigger exist; and
- no linked concept contradicts the proposed outcome without being called out.

If one gate is missing, keep the topic `in-review` and ask the narrowest question
that closes it.

## Review output shape

Use this compact structure in conversation:

```text
Decision N — <topic>
Question: <bounded question>
Proposal: <current option>
Alternative: <credible alternative>
Trade-offs: <pros>; <cons/risks>
Dependencies: <links or none>
Open questions: <only blockers>
Acceptance evidence: <what will prove it>

Please choose: accept, accept with conditions, modify, reject, or defer.
```

After recording, report the files changed, the new status, the next unblocked
topic, and any evidence that still needs to be produced.

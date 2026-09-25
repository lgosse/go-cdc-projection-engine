# Architecture documentation instructions

These instructions apply to the `docs/architecture/` subtree. Read the
repository-root [`AGENTS.md`](../../AGENTS.md) first.

## Review workflow

- Read [`index.md`](index.md), [`review-method.md`](review-method.md), the
  relevant section index, the current concept, and its direct related links.
- Use the repository-local
  [architecture decision review skill](../../.skills/architecture-decision-review/SKILL.md).
- Review one concept at a time. Present a decision card and wait for an
  explicit user disposition before changing a proposal to `accepted`.
- Never create an ADR that claims acceptance without explicit acceptance.
- For `accept with conditions`, record the conditions and move unresolved
  material choices into the [follow-up register](follow-ups.md).
- Present each recommendation with enough plain-language context to explain
  why the choice matters and what changes for an event or operator. When a
  concrete example would make the choice easier to see, include the input,
  selected behavior, and observable outcome; identify whether the example is
  from repository evidence or hypothetical.

## Editing rules

- Keep `docs/design/` unchanged; source drafts are provenance.
- Keep architecture concepts implementation-neutral unless the accepted choice
  genuinely requires a mechanism.
- On acceptance, update the concept status, add the numbered ADR, update the
  relevant section index, update `decisions/index.md`, and append `log.md`.
- Preserve historical ADRs. Supersede them with a new linked decision instead
  of rewriting history.
- Record new questions in [`follow-ups.md`](follow-ups.md) immediately, with a
  class, owner, milestone gate, status, and resolution or trigger.

## Quality checks

When architecture Markdown changes, run:

- `git diff --check`;
- frontmatter and relative-link validation for all architecture documents; and
- a status/index check confirming that accepted concepts have discoverable ADRs.

Report what was checked and what remains unverified. Do not implement Go code,
package layouts, dependencies, or deployment manifests in this subtree.

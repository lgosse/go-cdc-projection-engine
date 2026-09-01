# Project agent instructions

These instructions apply to all work in this repository. Higher-priority
system, developer, and user instructions always take precedence.

## Start every task

1. Read [`README.md`](README.md) and the relevant part of the
   [implementation roadmap](docs/implementation-plan.md).
2. Read the [architecture index](docs/architecture/index.md) and the
   [follow-up register](docs/architecture/follow-ups.md) when the task affects
   architecture, contracts, runtime behavior, operations, or validation.
3. Inspect `git status` and preserve unrelated user changes.
4. Identify the accepted ADRs and open follow-ups that govern the requested
   scope. Do not silently invent a rule where a follow-up is still open.

## Project boundaries

- Keep the original drafts under `docs/design/` unchanged. They are provenance,
  not editable decision records.
- Treat accepted ADRs as constraints. Do not weaken or reinterpret them without
  an explicit architecture review and a superseding decision.
- Do not add implementation structure merely because an ADR leaves mechanisms
  open. Use the roadmap's design checkpoint first.
- Keep the engine domain-neutral; the first projection is a proving case, not a
  reason to hardcode domain behavior.
- Do not broaden the requested scope or make external changes such as commits,
  tags, deployments, or messages unless the user explicitly asks.

## Architecture work

For work under `docs/architecture/`, read and follow the local
[architecture decision review skill](.skills/architecture-decision-review/SKILL.md)
and the scoped instructions in
[`docs/architecture/AGENTS.md`](docs/architecture/AGENTS.md).

Review one bounded topic at a time. Before stamping a decision, present the
decision card and wait for an explicit disposition. After acceptance, update
the concept, ADR, indexes, and log. Record every unresolved implementation
question in the [follow-up register](docs/architecture/follow-ups.md).

## Implementation work

- Do not begin a substantial slice until its design checkpoint is documented
  and applicable blocking follow-ups are resolved or the scope is explicitly
  narrowed.
- Preserve the accepted separation between source delivery, canonical event
  handling, relationship context, transformation, fencing, target writes,
  custody, and offset completion.
- Prove reuse with the roadmap's second-manifest or synthetic-projection
  checkpoint before expanding production scope.
- Add tests and evidence for the failure paths required by the applicable ADRs,
  not only the nominal path.

## Validation and reporting

- Use `apply_patch` for hand-authored file edits.
- Run the narrowest relevant checks, including `git diff --check` and the
  architecture frontmatter/link validation when Markdown architecture files
  change.
- Report changed files, validation performed, known unverified areas, and any
  new follow-up entries.

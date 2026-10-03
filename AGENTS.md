# Project Instructions

## Source of truth

- `README.md` defines the product requirements and architecture.
- The repository state is the source of truth for implementation details.
- Do not assume files or features exist unless they are present in the repository.

## Development workflow

- Work on one milestone at a time.
- Inspect the repository before modifying files.
- Make the smallest change that satisfies the current requirement.
- Do not implement future milestones proactively.
- Do not perform unrelated refactors.
- Do not add dependencies without justification.

## Verification

- Verify every completed change before considering it complete.
- Never claim tests, linting, type checking, or manual verification passed unless actually executed.
- If verification cannot be performed, report that explicitly.

## Handoff

- Stop when the current milestone is complete.
- Stop when continuing requires a significant architectural decision.
- Stop when the task becomes substantially larger than originally scoped.
- Stop when repeated failures indicate that more investigation is required.
- Stop when the context becomes unreliable.

When a session needs to continue in another context:

1. Create or update `docs/handoff.md`.
2. Document the current milestone.
3. Document what was implemented.
4. Document files created and modified.
5. Document verification.
6. Document unresolved problems and decisions.
7. Document exactly what the next session should do.

Do not copy the conversation into the handoff.

## Human approval

Ask the user before:

- Major architectural changes.
- Major dependencies.
- Database redesigns.
- Replacing technologies defined in README.md.
- Expanding the scope of the current milestone.
- Resolving ambiguous requirements with a significant assumption.

## Scope

The goal is not to maximize code written per session.

The goal is to produce small, correct, verifiable increments that can be safely continued by another session.

# Project Instructions

## Source of truth

- `README.md` defines the product requirements and architecture.
- The repository state is the source of truth for implementation details.
- Git history is the source of truth for implementation history.
- Do not assume files or features exist unless they are present in the repository.
- Do not infer requirements that are not documented or explicitly requested.

## Development workflow

- Work on one milestone at a time.
- Inspect the repository before modifying files.
- Make the smallest change that satisfies the current requirement.
- Do not implement future milestones proactively.
- Do not perform unrelated refactors.
- Do not add dependencies without justification.
- Do not create speculative models, abstractions, endpoints, or infrastructure for future features.
- Prefer simple, explicit implementations over premature abstraction.
- Before making significant architectural decisions, ask the user for approval.

## Milestones

Each milestone should produce a small, coherent, verifiable increment.

Before implementing a milestone:

1. Inspect the current repository and Git state.
2. Read the relevant requirements from `README.md`.
3. Identify the minimum files and changes required.
4. Identify any important ambiguity or architectural decision.
5. Ask for user approval if the decision is significant.

After implementing a milestone:

1. Run the appropriate verification.
2. Inspect the resulting Git diff.
3. Confirm that no unrelated changes were introduced.
4. Report what was implemented and how it was verified.
5. Create a Git commit representing the completed milestone.

Do not commit incomplete or unverified work.

## Git

- Use Git to preserve the history of completed milestones.
- Each completed milestone should correspond to a coherent Git commit.
- Commit messages should briefly describe the completed change.
- Do not create commits for unrelated changes.
- Do not commit generated files, local databases, secrets, credentials, or local model files.
- Do not rewrite or delete Git history unless explicitly requested.
- Do not assume that a milestone is complete merely because the code was written; verification is required first.

## Verification

- Verify every completed change before considering it complete.
- Never claim tests, linting, type checking, endpoint checks, or manual verification passed unless they were actually executed.
- If verification cannot be performed, report that explicitly.
- Prefer automated verification when practical.
- For API changes, verify the relevant endpoints with actual requests.
- For database changes, verify that the application can initialize and interact with the database correctly.
- If verification reveals warnings or errors introduced by the current milestone, address them before considering the milestone complete when reasonably within scope.
- Do not hide unresolved warnings, errors, or limitations.

## Development environment

- The primary development environment is Windows 11.
- The shell is Git Bash / MINGW64.
- Do not assume that Unix/Linux utilities are available.
- Use Windows-compatible commands when managing Windows processes.
- Do not use `pkill`, `fuser`, or other Unix-only process-management commands unless their availability has been explicitly verified.
- Be aware that Git Bash may transform arguments beginning with `/` when invoking native Windows commands.

For example, when using `taskkill` from Git Bash, prefer:

`MSYS_NO_PATHCONV=1 taskkill /PID <PID> /F`

rather than assuming standard Unix process-management behavior.

## Development servers

- Before starting a development server, check whether the intended port is already in use.
- Do not start duplicate instances of the same development server unnecessarily.
- Do not silently switch to another port merely because the expected port is occupied.
- If an existing server is running and responding correctly, reuse it for verification when possible.
- If a running process needs to be terminated, identify its PID first and use a Windows-compatible method.
- Do not leave multiple unnecessary instances of the same development server running.
- If process management fails, report the problem instead of repeatedly attempting incompatible commands.
- Never claim that a server was stopped unless the process was actually terminated and verified.

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
5. Document verification performed.
6. Document unresolved problems and limitations.
7. Document important decisions already made.
8. Document exactly what the next session should do.

Do not copy the conversation into the handoff.

`docs/handoff.md` is a transition document, not a duplicate of the Git history or README.

If the milestone is completely finished and no context needs to be transferred, a handoff file is not required.

## Human approval

Ask the user before:

- Major architectural changes.
- Major dependencies.
- Database redesigns.
- Replacing technologies defined in `README.md`.
- Expanding the scope of the current milestone.
- Resolving ambiguous requirements with a significant assumption.
- Introducing authentication or security-sensitive infrastructure.
- Changing the persistence strategy.
- Adding external/cloud services that conflict with the local-only architecture.

Do not ask for approval for small implementation details that are already clearly determined by `README.md` and the current milestone.

## Scope control

- The goal is not to maximize code written per session.
- The goal is to produce small, correct, verifiable increments that can be safely continued by another session.
- Do not implement features simply because they are mentioned as future goals in `README.md`.
- Only implement the current milestone.
- If a future requirement becomes relevant to the current implementation, document it rather than implementing it prematurely.
- If the current task reveals that the milestone definition is insufficient, stop and propose a revised milestone instead of expanding scope silently.

## Project-specific principles

- The application is a multi-learner local English tutoring system.
- Multiple learners must be supported conceptually, but authentication is not part of the initial implementation unless explicitly requested.
- Keep learner data isolated through explicit relationships between the relevant models.
- Do not add authentication merely because multiple learners exist.
- The application is intended to run locally and should not introduce cloud AI services or external infrastructure unless explicitly approved.
- Preserve the technologies defined in `README.md` unless a change is explicitly approved.
- Build the learning system incrementally; do not create speculative tables or services for the entire future architecture.
- The pedagogical requirements in `README.md` are product requirements, not permission to implement all future learning functionality at once.

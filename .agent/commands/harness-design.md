# Harness Design

Use this command to guide a feature from requirements to architecture to implementation progress.

The command manages three markdown files by default:

- `docs/feature-requirements.md` - user-approved requirements, feature list, acceptance criteria, and open questions
- `docs/framework-design.md` - concise, stable framework and architecture design
- `docs/implementation-todo.md` - mutable implementation todo and progress tracker

Use these filenames unless the user explicitly asks for different names or the repository already has equivalent docs. If equivalent docs exist, confirm the mapping once, then use that mapping consistently for the rest of the session.

## Operating Modes

1. **Design Mode** - use when the requirements doc or framework-design doc is missing, empty, or only contains placeholders. Goal: converge on confirmed requirements and a lightweight architecture.
2. **Execution Mode** - use when the requirements doc and framework-design doc exist with real content. Create the todo doc on entry if it is missing.
3. State the active mode before continuing.
4. If the user asks to revise requirements or architecture during execution, pause implementation and return to Design Mode for that scope.
5. Never silently switch modes. If a mode change is needed, propose it and wait for user confirmation.

## Design Mode

Goal: create or complete the three design documents through conversation.

### Requirements Document

File: `docs/feature-requirements.md`

Rules:

- Treat this as the source of truth for product requirements.
- Do not create or modify requirements based only on assumptions.
- Ask the user targeted questions before writing or changing any feature.
- Before editing this file, restate the proposed requirement changes and get explicit user confirmation.
- Keep each feature testable and implementation-independent.
- Mark uncertainty as open questions instead of inventing answers.

Writing guidance:

- Choose the document structure based on the user's project and the conversation.
- Do not force a fixed template if another shape communicates the requirements better.
- Prefer stable feature IDs such as `F-001` when features need to be tracked across design, todos, verification, and commits.
- Include enough detail to make each feature verifiable, usually covering user value, required behavior, acceptance criteria, dependencies, status, and open questions.
- Keep open questions visible instead of resolving them by assumption.

### Framework Design Document

File: `docs/framework-design.md`

Rules:

- Keep this concise, stable, and abstract.
- Prefer simple module boundaries and reusable interfaces over detailed implementation steps.
- Update it only when a feature changes the architecture meaningfully.
- Avoid frequent churn. If a small task-level detail changes, update the todo doc instead.
- When an architecture update is needed, explain the reason briefly before editing.

Suggested sections, only when useful:

- **Overview** - what the system is and is not.
- **Core abstractions** - the few concepts the rest of the code depends on.
- **Module boundaries** - ownership, packages, or directories when non-obvious.
- **Data flow / lifecycle** - only when state transitions matter.
- **Invariants** - properties that must stay true across modules.
- **Deferred decisions** - explicit choices being postponed.

Size budget: aim for one screen for small projects and 2-3 screens for medium projects. If the document grows beyond that, move implementation detail into the todo doc or the code.

### Implementation Todo Document

File: `docs/implementation-todo.md`

Rules:

- Treat this as a working progress tracker, not a design authority.
- Update it freely as implementation progresses.
- Keep tasks small enough to verify.
- Link tasks back to feature IDs from the requirements document when feature IDs exist.
- Mark current, blocked, done, and skipped work clearly.
- Choose whatever checklist or progress format makes the current implementation easiest to track.

## Execution Mode

Goal: implement confirmed features incrementally using the three documents.

### Startup

1. Re-read all three command-managed docs. Trust the docs over chat memory.
2. Cross-check feature IDs and references across the requirements, framework-design, and todo docs.
3. If the docs disagree, surface the inconsistency and ask before coding.
4. Summarize the next confirmed, unimplemented feature.
5. Ask the user to confirm that feature or choose a different one before writing code.
6. Treat the requirements doc as read-only unless the user explicitly confirms a change in this chat.
7. Touch the framework-design doc only for meaningful architectural changes.
8. Use the todo doc as the live progress tracker.

### Per-Feature Loop

For each confirmed feature:

1. Pick one feature from the requirements doc.
2. Break it into small implementation tasks in the todo doc.
3. Implement the smallest coherent slice.
4. Run local tests or targeted checks when available.
5. Start a verification subagent for the completed feature.
6. If verification passes, commit only the relevant changes.
7. If verification fails, use the verification result to iterate, then verify again.
8. Repeat until the feature passes verification and is committed, or until failure handling says to stop.

### Subagent Verification

After implementing each feature, start a verification subagent before committing. This is mandatory for implemented features.

Docs-only, formatting-only, planning-only, or todo-only edits are not implemented features and do not enter this per-feature verification loop. If such edits are made, say which local checks were run or why no checks were needed.

Use a read-only subagent when the platform supports it. The verifier must not modify files.

Pass to the subagent:

- the feature ID
- the acceptance criteria, verbatim
- relevant commands or checks to run
- user-facing entrypoints or sample inputs

Do not pass:

- file diffs or exact changed files unless required to run checks
- implementation notes, design rationale, or internal class names
- your reasoning trace
- broad project context unrelated to the feature

Ask the subagent to report:

- pass / fail / partial
- commands run
- observed behavior versus expected behavior
- bugs, risks, or surprising side effects
- minimal reproduction steps for any failure

Example prompt:

```text
Verify feature <ID> against these acceptance criteria:

<acceptance criteria, verbatim>

You may read the repository and run these checks:

<commands>

Do not modify files. Exercise the feature from the user's point of view and avoid relying on implementation details. Report pass/fail, commands run, observed versus expected behavior, and any bugs or risks.
```

### Commit Rules

Invoking this command authorizes incremental commits after each feature passes verification, but it does not override normal git safety rules.

Never:

- bypass hooks with `--no-verify`
- amend commits that were not created in this session
- amend pushed commits
- force push
- reset hard
- switch branches without user confirmation
- stage unrelated changes blindly

Before each commit:

1. Run `git status`, `git diff`, and `git diff --staged`.
2. Stage only files relevant to the verified feature, plus necessary updates to the three command-managed docs.
3. If unrelated user changes are present, leave them unstaged unless the user explicitly asks to include them.
4. Do not commit secrets, `.env` files, credentials, caches, build artifacts, or local-only files.
5. Use a concise commit message based on the delivered behavior, preferably `<feature-id>: <imperative summary>`.
6. After committing, update the todo doc if needed and continue to the next feature.

### Failure Handling

If verification fails:

1. Do not commit.
2. Summarize the failure briefly: symptom, failing check, and likely scope.
3. Record the failing check in the todo doc.
4. Fix the smallest thing that plausibly addresses the root cause.
5. Re-run local checks.
6. Start another verification subagent.
7. If the same failure, or a failure with the same likely root cause, recurs after 2 fix attempts, stop and ask the user how to proceed.

If requirements are wrong or incomplete:

1. Stop implementation.
2. Ask the user for clarification.
3. Restate the proposed requirement change.
4. Update the requirements doc only after explicit confirmation in this chat.
5. Resume execution after requirements are confirmed.

## Out Of Scope

This command does not automatically:

- choose or change branch strategy
- create or switch branches
- run long-lived dev servers or watchers
- implement multiple features in parallel
- edit requirements without explicit user confirmation
- commit unrelated user changes
- resolve ambiguous product behavior by assumption

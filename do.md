---
description: Execute a saved implementation plan artifact end-to-end until tasks, verification, and acceptance are complete
---

# Do Plan Harness

User input: `$ARGUMENTS`

## Mission

Execute a markdown implementation plan artifact end-to-end through code changes, verification, and truthful progress updates.

- The plan file is the canonical execution source of truth.
- The run is not complete until the plan is fully consumed.
- "Fully consumed" means every in-scope task is either `done` or explicitly recorded as a hard blocker, every milestone is updated, end-state completion conditions are checked against reality, and verification has been run for the implemented work.
- Start with the smallest plan context that can safely identify the next runnable work. Deepen reads only when that task needs more plan detail.

## Input Handling

- `$ARGUMENTS` must be one markdown file path or filename to execute.
- Accept these forms when they resolve to one existing markdown file:
  - absolute path
  - workspace-relative path
  - exact filename
- If `$ARGUMENTS` is empty, fail and ask the user for the plan file path.
- If `$ARGUMENTS` resolves to more than one file, fail and ask for an exact path.
- Do not treat `$ARGUMENTS` as new planning requirements. It is only the plan file to consume.

## Hard Fail Rules

Fail immediately without starting implementation if any of these are true:

1. No saved plan artifact can be identified.
2. Multiple plausible plan files match and `$ARGUMENTS` does not clearly disambiguate.
3. The resolved file is outside the workspace or is not a markdown file.
4. The file does not look like an execution-ready implementation plan artifact because it lacks the minimum capabilities needed to execute safely, such as:
   - a goal or requested outcome
   - a task or milestone structure that can be executed
   - a progress/tracker structure or clearly inferable status markers
   - verification intent or completion checks
5. The artifact is still an unfinished planning draft, is obviously ambiguous, or explicitly says implementation should not begin yet.
6. `$ARGUMENTS` is not a path-like reference to one markdown file.
7. The run begins without first ingesting enough of the plan to determine the current goal, runnable work, tracker state, and verification expectations.

If hard-failing, report the exact reason and ask the user for a valid execution-ready markdown plan file.

## Non-Negotiable Rules

1. Read enough of the plan artifact to execute safely before implementing anything.
2. Prefer progressive ingest over whole-plan ingestion by default.
3. Pull the full plan into context only when the artifact is tiny, malformed, contradictory, or the current task cannot be understood safely from targeted sections.
4. Treat the saved plan as canonical scope, execution order, and acceptance intent.
5. Do not silently widen scope, narrow scope, or swap architectures.
6. Do not stop at analysis, partial edits, or the first passing targeted test.
7. Ordinary implementation friction is not a blocker. Failing tests, broken assumptions, missing small refactors, and integration bugs are work to resolve.
8. Continue task-by-task until the plan is complete or a real external blocker is hit.
9. Real blockers are limited to things like missing credentials, unavailable external systems, destructive approval requirements, conflicting user edits that cannot be safely reconciled, or plan contradictions that cannot be resolved from repository evidence.
10. When a real blocker exists, update the plan artifact truthfully before asking the user for help.
11. Prefer the smallest correct implementation that satisfies the plan.
12. Keep the work cohesive. Do not create unnecessary abstractions, giant files, or speculative helpers.
13. Ignore unrelated dirty-worktree changes. Never revert or overwrite work you did not make unless the user explicitly asks.
14. Do not create a second project-management document. The plan artifact remains the canonical tracker.
15. Do not claim completion while any material task is still `pending`, `in_progress`, or `blocked` without explicit blocker documentation.
16. Keep execution updates compact and patch-like. Do not turn the plan into a running diary.
17. Verification must be maintained as `### Required Verification` plus `### Latest Results`, whether it lives under the newer `## Verification` section or an older equivalent section.
18. Any end-state completion section must remain criteria-focused. Do not append milestone narratives or long evidence histories there.
19. Any timestamped execution or verification entries written into the plan must use local `YY-MM-DD-HH-MM` 24-hour format and remain in chronological order with newest last.

## Plan Intake Procedure

Before substantial edits:

1. Resolve the plan path.
2. Start with targeted plan discovery:
   - metadata
   - goal/requested outcome section
   - progress/tracker section
   - verification section
   - blocker/open-question section if present
3. Grep or targeted-read headings and task IDs before deciding whether deeper reads are needed.
4. Extract at minimum:
   - metadata and current status
   - top-level goal
   - current task/milestone statuses
   - task list and dependency ordering
   - verification requirements and latest concise results
   - blockers or deferred decisions that still affect execution
5. Read deeper only when the selected runnable task cannot be executed safely from the initial targeted ingest.
6. Validate that the plan still matches the current repository well enough to execute.
7. If the plan was partially executed earlier, resume from the first unfinished runnable task instead of restarting completed work.

Progressive ingest escalation order:

1. tracker + goal + verification
2. current milestone and owning task block
3. decisions/architecture context for the active task
4. full artifact only when still necessary

## Semantic Alias Matrix

Use the same semantic mapping used by `/plan/do-partial`, `/plan/status`, and `/plan/explain`.

- goal: `Goal and Completion Signal`, `Requested Outcome`, `Executive Summary`, `Summary`, `Objective`
- scope: `Scope Boundaries`, `Scope`, `In Scope`, `Out of Scope`
- decisions: `Decisions and Open Questions`, `Resolved Clarifications`, `Ambiguity Audit`, `Constraints and Compatibility`, `Alternatives Considered`
- execution: `Execution Plan`, `Roadmap and Milestones`, `Implementation Breakdown`, `Task Breakdown`, `File and Module Surface`, `File and Module Plan`
- progress: `Progress Tracking`, `Tracker`, `Task Tracker`, `Checklist`, milestone tables, task tables
- verification: `Verification`, `Testing and Verification`, `Required Verification`, `Latest Results`
- blockers: `Risks and Blockers`, `Deferred Decisions or Blockers`, `Open Questions`, `Risks and Mitigations`

## Required Plan-File Updates

The plan artifact is not just input. It must be kept current during execution.

Update it at these moments:

1. When execution begins:
   - set metadata status to `in_progress` unless the plan already reflects an accurate active state
2. When a task starts or finishes:
   - update task status in `## Progress Tracking`
   - keep milestone checkboxes aligned
3. When a task reveals a real blocker:
   - mark the task `blocked`
   - add the blocker to the plan's blocker/open-question section
   - set metadata status to `blocked` if progress cannot continue
4. When verification actually runs:
    - update the plan's verification section with concise execution results under `### Latest Results`
5. When acceptance criteria are satisfied:
    - reflect that in the plan's end-state completion section by marking current satisfied or blocked state concisely without turning the section into an evidence log
6. When the whole plan is complete:
    - set metadata status to `completed`

Update the artifact truthfully and concisely. Do not turn it into a long diary.

If verification is still in an older section such as `## Testing and Verification`, normalize it minimally into:

- `### Required Verification`
- `### Latest Results`

Preserve the required checks under `### Required Verification` and write only current compact outcomes under `### Latest Results`.

## Execution Update Budget

Plan updates must stay small.

1. Update only the tracker rows, blocker notes, verification results, and acceptance states materially affected by the work just completed.
2. Record only material command outcomes in `### Latest Results`.
3. Record manual findings only when they prove behavior, expose a blocker, or explain residue that automation does not cover.
4. Collapse repeated reruns into one latest outcome instead of appending every failed and passing attempt.
5. Do not append repeated `check passed` notes when the tracker already proves the same state and no new risk was reduced.
6. Prefer replacing stale result bullets with a newer concise summary over accumulating a long chronological log.
7. Do not use the end-state completion section as a milestone scrapbook; keep it to current completion conditions and their present status.
8. If multiple timestamped result entries remain, order them oldest to newest so the newest item is last.

## Re-Grounding Rules

The plan is authoritative, but reality wins when the repository has moved.

1. Verify that referenced files, modules, commands, and interfaces still exist.
2. If a referenced path moved or was renamed, locate the real target and update the plan minimally before continuing.
3. If the plan and repo disagree in a way that affects correctness, prefer the smallest evidence-backed adaptation that preserves the approved outcome.
4. If the disagreement changes scope, behavior, or architecture materially, stop and ask the user instead of freelancing.

## Execution Loop

Consume the plan in dependency order.

For each runnable task:

1. Re-read the relevant plan subsections for that task.
2. Inspect only the code, tests, configs, and docs needed for that task.
3. Implement the smallest correct change set.
4. Run the most relevant verification for that task immediately.
5. Fix failures and rerun verification until the task is truly done or a hard blocker is proven.
6. Update the plan artifact status and verification notes.
7. Move to the next runnable task.

When updating the artifact after a task:

- patch the smallest truthful portion of the file
- keep `### Latest Results` concise and current
- if timestamped entries are used, append or update them so chronological order stays oldest to newest
- do not restate implementation details already captured in task rows or code

If the plan's task tracker and the detailed task breakdown disagree:

- use the more specific dependency information
- repair the tracker so it matches the actual execution order
- then continue

Do not pause after one milestone if later milestones are unblocked. Keep going until the plan is consumed.

## Verification Rules

Testing and verification are mandatory, not optional cleanup.

1. Use the explicit commands and checks listed in the plan first.
2. If the plan's commands are stale or underspecified, derive the correct commands from repository evidence and update the plan accordingly.
3. Run targeted verification after meaningful tasks so problems are caught early.
4. Run broader final verification before claiming completion. This should include all relevant checks for the changed surfaces, such as unit tests, integration tests, end-to-end tests, type checks, linters, builds, migrations, or smoke tests when applicable.
5. If the workspace provides both narrow and broad verification, do both when they materially reduce risk.
6. If a final broad verification command fails, do not stop at the failure report. Debug, fix, and rerun unless the failure is proven to be an unrelated external blocker.
7. If an unrelated pre-existing failure prevents a fully clean final run, record the evidence clearly in the plan artifact and still verify the changed surfaces as deeply as the workspace allows.
8. Manual verification steps from the plan must be executed when automation does not fully cover the behavior.

## Completion Gate

The run is complete only when all of the following are true:

1. All in-scope tasks are `done`, or any remaining blocked tasks are tied to explicit hard blockers recorded in the plan.
2. Milestone checkboxes and task tracker statuses reflect reality.
3. The implemented code matches the approved scope and does not leave obvious half-finished scaffolding.
4. Required docs, configs, migrations, scripts, and operational updates from the plan are completed.
5. The verification section reflects the actual commands and results.
6. The end-state completion section has been checked against the resulting system and remains criteria-focused; satisfied or blocked state is clear without a long evidence history.
7. Metadata status accurately reflects the outcome: `completed` when done, `blocked` only when a real blocker prevented completion.

Do not end the run while leaving the plan file claiming progress that does not match reality.

## Stop Conditions

The harness must persist until the completion gate is met.

Do not stop because of:

- complexity
- repo size
- partial success
- one passing test
- one failing test
- needing additional focused refactors
- needing to inspect more files
- needing to update the plan tracker

Stop only for:

- a proven external blocker
- a required interactive or privileged command that the user must run manually
- a material plan contradiction that requires a user decision
- a concurrent change conflict that cannot be resolved safely without user input

When stopping for one of those reasons, make the plan artifact reflect the exact blocked state first.

## Final Output

When the command run finishes:

- on success, report briefly:
  - implemented plan path
  - final plan status
  - high-level outcome
  - key verification results
- on blocked exit, report briefly:
  - plan path
  - exact blocker
  - affected task or milestone IDs
  - what user input or dependency is required next

Keep the chat response concise. The updated plan artifact is the canonical execution record.

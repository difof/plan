---
description: Execute a bounded milestone, task, or deterministic next-task slice from a saved implementation plan artifact
---

# Do Plan Partial Harness

User input: `$ARGUMENTS`

## Mission

Execute only a bounded slice of one saved markdown implementation plan artifact while keeping the plan file as the canonical execution record.

- Unlike `/plan/do`, this command does not try to consume the whole plan.
- It must still load the full plan file into chat context before doing any implementation work.
- It must update the plan artifact truthfully for the selected slice.
- The run ends when the selected milestone, task list, or next-task slice is done or explicitly blocked, not when the whole plan is complete.

## Input Handling

- `$ARGUMENTS` must begin with one markdown plan file reference.
- After the plan reference, the remaining tokens choose one of these modes:
  - `<plan>`
  - `<plan> next`
  - `<plan> next <count>`
  - `<plan> <selector>`
  - `<plan> <task-id> <task-id> ...`
- `<selector>` may be:
  - exact milestone ID like `M2`
  - exact task ID like `T7`
  - exact milestone label text or exact task label text from the plan
  - unique fuzzy text that resolves to one milestone or one task after full plan intake
- In explicit multi-task mode:
  - every selector after `<plan>` must be an exact task ID like `T7`
  - the provided task ID order is authoritative and must already be dependency-safe
  - do not auto-reorder the list
- If a count is provided:
  - it is only valid in `next` mode
  - it must be a positive integer
  - it caps the number of tasks in the selected direct dependency chain
- If a trailing integer appears after a non-`next` selector, treat that as invalid input instead of guessing whether it is part of the selector or a count override.
- If no selector is provided, use deterministic `next` mode.
- If the plan path and selector cannot be separated unambiguously from `$ARGUMENTS`, fail and ask the user for a clearer invocation.

## Hard Fail Rules

Fail immediately without starting implementation if any of these are true:

1. No saved plan artifact can be identified.
2. Multiple plausible plan files match and `$ARGUMENTS` does not clearly disambiguate one target.
3. The resolved file is outside the workspace or is not a markdown file.
4. The file does not look like an execution-ready implementation plan artifact, including missing core sections such as:
   - `## Metadata`
   - `## Scope`
   - `## Requirements`
   - `## Architecture and Design`
   - `## Task Breakdown`
   - `## Progress Tracking`
   - `## Testing and Verification`
   - `## Acceptance Criteria`
5. The artifact is still an unfinished planning draft, is obviously ambiguous, or explicitly says implementation should not begin yet.
6. The run begins without first pulling the entire markdown plan file into chat context.
7. The plan artifact lacks deterministic tracking data required for bounded execution, including any of:
   - stable milestone IDs
   - stable task IDs
   - explicit task-to-milestone ownership
   - explicit hard dependency mapping by task ID
   - canonical tracker row order that can act as a tie-breaker
8. `$ARGUMENTS` is not a path-like reference to one markdown plan file followed by an optional valid selector mode.
9. An explicit selector resolves to zero plausible targets or more than one plausible target.
10. `next` mode cannot identify a single starting task deterministically.
11. The selected milestone, task, or explicit multi-task list has unmet hard prerequisites outside the selected slice.
12. A count token is present but invalid.
13. Explicit multi-task mode contains any non-task-ID selector, duplicate task ID, or task order whose hard dependencies are not already `done` or satisfied by earlier selected tasks.

If hard-failing, report the exact reason and ask the user for a valid execution-ready markdown plan file, clearer selector, or a revised plan artifact.

## Non-Negotiable Rules

1. Read the full plan artifact before implementing anything.
2. At the start of the run, pull the entire markdown plan file into chat context in one complete read, not partial slices.
3. Treat the saved plan as canonical scope, task graph, execution order, and acceptance intent.
4. Do not silently widen the selected slice.
5. For explicit milestone selection, execute only the remaining tasks owned by that milestone.
6. For explicit task selection, execute only that task.
7. For explicit multi-task selection, execute only the provided task IDs.
8. For `next` mode, execute only the selected deterministic chain of task IDs.
8. Hard fail instead of auto-running prerequisites outside the selected slice.
9. Ordinary implementation friction inside the selected slice is work to resolve, not a blocker.
10. Stop after the selected slice is complete or a real blocker is recorded.
11. Keep the plan artifact truthful at all times.
12. Do not create a second tracker document. The plan artifact remains the canonical record.
13. Ignore unrelated dirty-worktree changes. Never revert or overwrite work you did not make unless the user explicitly asks.
14. Prefer the smallest correct implementation that satisfies the selected slice.
15. Do not mark the overall plan `completed` unless the whole plan is actually complete.
16. Do not consume downstream tasks outside the selected slice just because they became unblocked during this run.
17. Keep execution updates compact and patch-like. Do not turn the plan into a running diary.
18. `## Testing and Verification` must be maintained as two subparts: `### Required Verification` and `### Latest Results`.
19. `## Acceptance Criteria` must remain criteria-focused. Do not append milestone narratives or long evidence histories there.
20. Any timestamped execution or verification entries written into the plan must use local `YY-MM-DD-HH-MM` 24-hour format and remain in chronological order with newest last.

## Plan Intake Procedure

Before substantial edits:

1. Resolve the plan path.
2. Read the full artifact into chat context, not just the summary and not just selected sections.
3. Extract at minimum:
   - metadata and current status
   - scope and out-of-scope boundaries
   - constraints and compatibility expectations
   - roadmap and milestones
   - implementation breakdown
   - task list and dependency ordering
   - file/module targets
   - testing and verification requirements
   - acceptance criteria
   - deferred decisions or blockers
4. Build a deterministic execution model from the plan itself, including:
   - milestone IDs and labels
   - task IDs and owning milestone IDs
   - explicit hard dependencies by task ID
   - canonical task row order from `## Progress Tracking`
   - current task statuses
5. Validate that the plan still matches the current repository well enough to execute.
6. Resolve the requested mode and compute the exact selected slice before major edits begin.

After the initial full-context read, later re-grounding may use targeted reads or grep for specific sections of the same file when context pressure grows, but the run must begin with the entire plan content already in context.

## Deterministic Tracking Requirements

This command depends on a plan artifact that exposes a real task graph, not just prose.

Minimum required signals:

- every milestone has a stable unique `M#` ID
- every task has a stable unique `T#` ID
- every task belongs to exactly one milestone
- every task lists hard dependencies as task IDs or `none`
- the task tracker row order is the canonical execution tie-breaker when multiple tasks are otherwise runnable
- the same IDs are used consistently across `## Roadmap and Milestones`, `## Task Breakdown`, and `## Progress Tracking`

If those signals are missing, contradictory, or only implied in prose, stop and direct the user to regenerate or revise the plan instead of guessing.

## Target Resolution

Resolve the requested slice using this priority order:

1. exact milestone ID match
2. exact task ID match
3. exact milestone label match
4. exact task label match
5. unique fuzzy milestone or task text match

If there are multiple selectors after `<plan>`, first determine whether the invocation is explicit multi-task mode:

1. multi-task mode is only valid when every selector token is an exact task ID
2. if any selector token is not an exact task ID, fail instead of mixing selector styles
3. when multi-task mode is active, do not use label or fuzzy matching for those selectors

Rules:

1. If the selector matches both a milestone and a task plausibly, fail and ask the user to disambiguate.
2. If fuzzy matching produces multiple plausible targets, fail and ask for a clearer selector.
3. Never guess between two targets that look similarly likely.

### Explicit Milestone Mode

When a milestone is selected:

1. Select all not-yet-done tasks owned by that milestone.
2. If the milestone has no remaining tasks, report that it is already complete and stop without unnecessary rewrites.
3. If any selected task depends on unfinished hard prerequisites outside the milestone, hard fail before implementation begins.
4. Execute the selected milestone tasks in dependency order, using canonical tracker row order as the tie-breaker for same-level runnable tasks.
5. Stop when the milestone's remaining tasks are done or a real blocker is recorded.

### Explicit Task Mode

When a task is selected:

1. Select only that task.
2. If the task is already `done`, report that clearly and stop without unnecessary rewrites.
3. If the task has unfinished hard prerequisites outside itself, hard fail before implementation begins.
4. Resume `in_progress` or re-attempt `blocked` work only when repository evidence suggests the task is still the right current target.

### Explicit Multi-Task Mode

When multiple exact task IDs are selected:

1. Select only the provided task IDs, in the exact order given by the user.
2. Reject duplicate task IDs.
3. If every selected task is already `done`, report that clearly and stop without unnecessary rewrites.
4. For each selected task in user-provided order, require every hard dependency to be satisfied by either:
   - a task already marked `done` in the plan, or
   - an earlier task in the same selected list
5. If a selected task depends on a later task in the same list, hard fail instead of reordering.
6. If a selected task has any unfinished hard prerequisite outside the selected list, hard fail before implementation begins.
7. Resume `in_progress` or re-attempt `blocked` work only when repository evidence suggests the selected task list is still the right current target set.

### Deterministic `next` Mode

When no explicit selector is given, or when the selector token is `next`:

1. If exactly one task is already `in_progress`, use that task as the chain head.
2. If more than one task is `in_progress`, hard fail and ask the user to target one explicitly.
3. Otherwise choose the first `pending` runnable task by canonical tracker row order.
4. Build a direct dependent chain from that head task:
   - include the head task
   - repeatedly look for one pending task whose hard dependency list includes the immediately previous selected task
   - require all of that candidate task's hard dependencies to be satisfied by tasks already `done` in the plan or by tasks already selected in the current slice
   - if exactly one such candidate exists, append it to the slice
   - if zero such candidates exist, stop the chain
   - if more than one such candidate exists, stop at the branch point instead of guessing
5. If a count was provided, stop when the selected slice reaches that many tasks, even if the chain could continue.
6. If no runnable pending task can be identified deterministically, hard fail.

## Required Plan-File Updates

The plan artifact is input and live execution record.

Update it at these moments:

1. When bounded execution begins:
   - set metadata status to `in_progress` unless the plan already reflects a more accurate active state
2. When a selected task starts or finishes:
   - update that task's status in `## Progress Tracking`
   - keep milestone checkboxes aligned when milestone completion changes
3. When a selected task reveals a real blocker:
   - mark the task `blocked`
   - add the blocker to `## Deferred Decisions or Blockers`
   - set metadata status to `blocked` only if no further safe progress can continue in the selected slice
4. When verification actually runs:
    - update `## Testing and Verification` with concise execution results under `### Latest Results`
5. When acceptance criteria are materially affected by the selected slice:
    - reflect that in `## Acceptance Criteria` by marking current satisfied or blocked state concisely without turning the section into an evidence log
6. When the slice ends:
    - if the whole plan is now complete, set metadata status to `completed`
    - otherwise leave metadata status as `in_progress` or `blocked`, whichever is truthful

Do not rewrite unrelated task rows just to make the file look busy.

If `## Testing and Verification` is not yet split, normalize it minimally into:

- `### Required Verification`
- `### Latest Results`

Preserve the required checks under `### Required Verification` and write only current compact outcomes under `### Latest Results`.

## Execution Update Budget

Plan updates must stay small.

1. Update only the selected tracker rows, blocker notes, verification results, and acceptance states materially affected by the work just completed.
2. Record only material command outcomes in `### Latest Results`.
3. Record manual findings only when they prove behavior, expose a blocker, or explain residue that automation does not cover.
4. Collapse repeated reruns into one latest outcome instead of appending every failed and passing attempt.
5. Do not append repeated `check passed` notes when the tracker already proves the same state and no new risk was reduced.
6. Prefer replacing stale result bullets with a newer concise summary over accumulating a long chronological log.
7. Do not use `## Acceptance Criteria` as a milestone scrapbook; keep it to current completion conditions and their present status.
8. If multiple timestamped result entries remain, order them oldest to newest so the newest item is last.

## Re-Grounding Rules

The plan is authoritative, but reality wins when the repository has moved.

1. Verify that referenced files, modules, commands, and interfaces still exist.
2. If a referenced path moved or was renamed, locate the real target and update the plan minimally before continuing.
3. If the plan and repo disagree in a way that affects the selected slice only, prefer the smallest evidence-backed adaptation that preserves the approved outcome.
4. If the disagreement changes scope, behavior, architecture, or target selection materially, stop and ask the user instead of freelancing.

## Execution Loop

Consume only the selected slice.

For each selected task in runnable order:

1. Re-read the relevant plan subsections for that task.
2. Inspect only the code, tests, configs, and docs needed for that task.
3. Implement the smallest correct change set.
4. Run the most relevant verification for that task immediately.
5. Fix failures and rerun verification until the task is truly `done` or a hard blocker is proven.
6. Update the plan artifact status and verification notes.
7. Move to the next selected task if one remains.

When updating the artifact after a task:

- patch the smallest truthful portion of the file
- keep `### Latest Results` concise and current
- if timestamped entries are used, append or update them so chronological order stays oldest to newest
- do not restate implementation details already captured in task rows or code

For explicit milestone mode:

- continue through the milestone's selected tasks until the milestone slice is finished or blocked

For explicit task mode:

- stop after that task is complete or blocked

For explicit multi-task mode:

- stop after the provided task list is consumed or blocked
- do not reorder or extend the selected list during the same run

For `next` mode:

- stop after the precomputed chain is consumed or blocked
- do not extend the chain past the selected boundary during the same run

## Verification Rules

Testing and verification are mandatory for the selected slice.

1. Use the explicit commands and checks listed in the plan first.
2. If the plan's commands are stale or underspecified, derive the correct commands from repository evidence and update the plan accordingly.
3. Run targeted verification after meaningful task work so problems are caught early.
4. Before stopping, run the smallest broader checks that materially cover all changed surfaces in the selected slice.
5. If a relevant verification command fails, do not stop at the failure report. Debug, fix, and rerun unless the failure is proven to be an unrelated external blocker.
6. If an unrelated pre-existing failure prevents a fully clean final check, record the evidence clearly in the plan artifact and still verify the changed surfaces as deeply as the workspace allows.
7. Manual verification steps from the plan must be executed when automation does not fully cover the selected behavior.

## Completion Gate

The run is complete only when all of the following are true:

1. Every task in the selected slice is `done`, or any remaining selected task is explicitly `blocked` with a real blocker recorded in the plan.
2. Milestone checkboxes and selected task tracker rows reflect reality.
3. The implemented code matches the approved selected slice and does not leave obvious half-finished scaffolding for that slice.
4. Required docs, configs, scripts, or operational updates named by the selected slice are completed.
5. `## Testing and Verification` reflects the actual commands and results for the selected work.
6. `## Acceptance Criteria` has been updated truthfully for any criteria materially affected by the selected work and remains criteria-focused without a long evidence history.
7. Metadata status accurately reflects the overall plan state after this bounded run.

Do not end the run while leaving the plan file claiming progress that does not match reality.

## Stop Conditions

The harness must persist until the bounded completion gate is met.

Do not stop because of:

- complexity inside the selected slice
- one passing test
- one failing test
- needing an additional focused refactor
- needing to inspect more files that are relevant to the selected work
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
  - selected slice, such as milestone ID or task IDs
  - final plan status
  - high-level outcome
  - key verification results
- on blocked exit, report briefly:
  - plan path
  - selected slice
  - exact blocker
  - affected task IDs
  - what user input or dependency is required next

Keep the chat response concise. The updated plan artifact is the canonical execution record.

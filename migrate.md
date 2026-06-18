---
description: Migrate one saved implementation plan artifact in place to the current shared plan structure without changing its execution truth
---

# Migrate Plan Harness

User input: `$ARGUMENTS`

## Mission

Migrate one existing markdown implementation plan artifact in place so it matches the current shared plan-artifact structure closely enough for the plan command system to keep consuming it safely.

The command must:

- inspect the existing plan before deciding whether migration is needed
- hard stop with a no-op result if the artifact is already up to date
- patch the existing file when a localized migration is sufficient
- allow a full in-place rewrite only when patching would leave the artifact structurally broken, contradictory, or harder to trust
- preserve truthful execution state so `/plan/do`, `/plan/do-partial`, `/plan/status`, and `/plan/explain` can continue from the migrated artifact without re-planning
- never create a new artifact version, duplicate file, supersession record, or cross-artifact lineage

The migrated artifact remains the same plan file.

- write back to the resolved source plan path only
- preserve the existing filename
- preserve the existing internal `version` metadata value unless the user explicitly asks to change it
- do not add a new `v<N>` or timestamped filename

## Input Handling

- the provided command input must resolve to one markdown plan file path or filename to migrate
- Accept these forms when they resolve to one existing markdown file:
  - absolute path
  - workspace-relative path
  - exact filename
- if no plan file input was provided, fail and ask the user for the plan file path
- if the provided command input resolves to more than one file, fail and ask for an exact path
- do not treat the provided command input as new planning requirements; it is only the plan file to migrate

## Hard Fail Rules

Fail immediately without editing if any of these are true:

1. No existing plan artifact can be identified.
2. Multiple plausible plan files match and the provided command input does not clearly disambiguate one target.
3. The resolved file is outside the workspace or is not a markdown file.
4. The run begins without first ingesting enough of the source markdown plan file to understand its goal, work structure, current state, and verification intent.
5. The resolved file is too incomplete or broken to migrate safely because the command cannot recover enough of the plan's execution truth, such as when there is no recoverable goal, no meaningful work structure, or no recoverable progress or verification intent.
6. The provided command input is not a path-like reference to one markdown file.

If hard-failing, report the exact reason and ask the user for one valid markdown plan file that contains enough execution truth to migrate safely.

## No-Op Gate

Before drafting edits, determine whether the artifact is already up to date with the current migration target defined in this command.

If it is already up to date:

- stop immediately without editing
- report that no migration is needed
- do not ask a save question

`Up to date` means the artifact already preserves the current shared execution contract closely enough that migrating it would be style churn rather than a real structural repair.

At minimum, the artifact should already provide:

- the current canonical top-level section set in a truthful form
- deterministic milestone and task tracking with stable IDs and hard dependency mapping
- the current tracker table fields needed by bounded execution commands
- verification structured as `### Required Verification` plus `### Latest Results`
- enough current structure that the other plan commands can consume it without migration-specific guessing

Minor truthful extras such as `updated_at_local` do not make an artifact out of date by themselves.

Use this checklist strictly:

### Already Up To Date

Treat the artifact as already up to date only when all of these are true:

- the top-level section set already matches the current canonical section roles closely enough that no section-owner confusion remains
- milestone IDs and task IDs are stable and internally consistent
- each task has recoverable milestone ownership and hard dependency mapping using task IDs or `none`
- the progress tracker already includes the current required table fields or obvious equivalent fields with no execution ambiguity
- verification already uses `### Required Verification` and `### Latest Results`, or an equivalent already split shape that needs no behavioral normalization
- metadata status is truthful enough for continued execution
- `/plan/do`, `/plan/do-partial`, `/plan/status`, `/plan/explain`, and `/plan/migrate` could all consume the artifact safely without needing one-off migration interpretation beyond the shared semantic aliases
- any extra sections are clearly harmless context rather than competing owners of goal, scope, decisions, execution, progress, or verification

### Safe But Worth Normalizing

Treat the artifact as safe but worth normalizing when execution could continue today, but one or more of these are still true:

- older sections such as `Executive Summary`, `Requested Outcome`, `Scope`, `Constraints and Compatibility`, `Requirements`, `Architecture and Design`, or `Roadmap and Milestones` still split ownership in ways the current dense structure would consolidate
- verification is still in an older unsplit section or otherwise needs normalization into `### Required Verification` and `### Latest Results`
- tracker fields are present but not yet in the current compact table shape expected by bounded execution commands
- metadata is mostly truthful but is missing current structural fields that can be recovered safely
- duplicated facts across adjacent sections make future execution or status reads noisier, even though the artifact is not yet broken
- old section names are still carrying current meaning and can be normalized with a localized patch that preserves execution truth

Do not migrate solely for cosmetic wording cleanup when the artifact already satisfies the `Already Up To Date` checklist.

## Non-Negotiable Rules

1. Read enough of the source plan artifact before proposing migration.
2. Treat the source plan as an evidence-bearing execution record, not as disposable prose.
3. Preserve truthful progress, blocker state, verification state, and current completion reality.
4. Prefer the smallest coherent migration patch.
5. Recommend rewrite only when patch mode would leave contradictions, stale structure, or unreadable tracker drift.
6. Do not widen scope, add new implementation requirements, or quietly redesign the plan's intended work.
7. Do not reset real progress back to `pending` just because the structure improves.
8. Do not invent milestone IDs, task IDs, dependencies, sizing, or verification claims unless they are required for structural recovery and can be grounded from the existing artifact.
9. If a required migrated field cannot be recovered safely from the source artifact, say so plainly and ask the user instead of guessing.
10. Before every `question` tool call, emit a short markdown explanation covering:
    - current understanding
    - what is still undecided
    - why the next decision matters
11. Do not duplicate the exact question options in that pre-question explanation.
12. Do not write the migrated artifact until the user has seen a concise migration summary and explicitly approved the save strategy.
13. If the current run cannot write directly, stop after the preview and tell the user the migration is ready to save in build/write mode.

## Migration Intent

This command is not a revision-planning command.

It exists to repair structure, normalize the artifact contract, and preserve execution continuity.

Rules:

1. Keep the plan's actual implementation intent the same unless the existing artifact is too broken to preserve literally.
2. Keep task graph meaning the same wherever possible.
3. Preserve existing progress and completion truth even when task rows or sections must be reorganized.
4. Do not add `## Revision History` or cross-artifact lineage just because the file was migrated.
5. Do not force a new title or a new filename.
6. If the artifact uses old or custom headings but is otherwise recoverable, normalize them into the current canonical structure only as far as needed to make the plan safe and durable.

## Migration Target

The current canonical target structure for migrated artifacts is the dense shared shape used by the modern plan command system.

Required top-level section order after migration:

1. `# Plan: <Human Title>`
2. `## Metadata`
3. `## Goal and Completion Signal`
4. `## Current Context and Evidence`
5. `## Scope Boundaries`
6. `## Decisions and Open Questions`
7. `## Engineering Standards`
8. `## Execution Plan`
9. `## Progress Tracking`
10. `## Verification`
11. `## Risks and Blockers`
12. `## References`

Section role rules:

- `## Goal and Completion Signal` owns the objective, end state, and concrete done conditions.
- `## Current Context and Evidence` owns observed facts only.
- `## Scope Boundaries` owns `### In Scope` and `### Out of Scope` only.
- `## Decisions and Open Questions` owns resolved decisions, major tradeoffs, and still-open items.
- `## Execution Plan` owns milestones, tasks, file or module surface, and contract changes.
- `## Progress Tracking` is a compact tracker only.
- `## Verification` owns required checks and the latest concise results only.
- `## Risks and Blockers` stays short and current.

### `## Metadata`

After migration, metadata should include at least:

- version when the source artifact already has one or it can be recovered safely
- title
- file path
- created_at_local when recoverable
- updated_at_local for a migrated in-place artifact
- workspace_root when recoverable or directly inferable
- request_source in clean current prose when it can be recovered safely
- status reflecting real execution state such as `approved draft`, `in_progress`, `blocked`, or `completed`

Do not add cross-artifact lineage metadata.

### `## Execution Plan`

After migration, structure it as:

- `### Milestones`
- `### Task Breakdown`
- `### File and Module Surface`
- `### Data, API, or Contract Changes`

Milestone requirements:

- stable unique milestone IDs such as `M1`, `M2`
- a short stable human label for each milestone
- a relative size `MS` from `1-10`
- dependency-ordered milestone list
- goal, key outputs, and completion signal for each milestone
- explicit owned task IDs in planned execution order

Task requirements:

- stable unique task IDs such as `T1`, `T2`
- one owning milestone per task
- relative size `S` from `1-10`
- objective
- concrete change surface
- hard dependencies using task IDs or `none`
- expected deliverable
- verification step

Relative sizing rules:

- `MS` and `S` are relative within the plan only
- they are not time estimates
- preserve existing truthful sizing when it already exists and still fits
- only infer or recalculate sizing when structural recovery genuinely requires it

### `## Progress Tracking`

After migration, this section must include both:

1. a milestone checklist using markdown checkboxes such as `- [ ] M1 [MS:8] ...`
2. a compact task tracker table with at least these columns:
   - `ID`
   - `Milestone`
   - `S`
   - `Status`
   - `Hard Depends On`
   - `Deliverable`
   - `Verification`

Tracking rules:

- every task row appears exactly once
- task and milestone IDs must match the execution plan
- `Hard Depends On` uses task IDs only or `none`
- tracker row order is the canonical execution-order tie-breaker
- preserve truthful statuses such as `pending`, `in_progress`, `blocked`, `done`
- preserve real progress; do not reset completed work

### `## Verification`

After migration, structure verification as:

- `### Required Verification`
- `### Latest Results`

Keep only material current results in `### Latest Results`.

If timestamped results are retained, they must:

- use local `YY-MM-DD-HH-MM` 24-hour timestamps
- remain in chronological order with newest last

## Semantic Alias Matrix

Use the same semantic mapping used by `/plan/do`, `/plan/do-partial`, `/plan/status`, and `/plan/explain` when reading old or custom artifacts.

- goal: `Goal and Completion Signal`, `Requested Outcome`, `Executive Summary`, `Summary`, `Objective`
- scope: `Scope Boundaries`, `Scope`, `In Scope`, `Out of Scope`
- decisions: `Decisions and Open Questions`, `Resolved Clarifications`, `Ambiguity Audit`, `Constraints and Compatibility`, `Alternatives Considered`
- execution: `Execution Plan`, `Roadmap and Milestones`, `Implementation Breakdown`, `Task Breakdown`, `File and Module Surface`, `File and Module Plan`
- progress: `Progress Tracking`, `Tracker`, `Task Tracker`, `Checklist`, milestone tables, task tables
- verification: `Verification`, `Testing and Verification`, `Required Verification`, `Latest Results`
- blockers: `Risks and Blockers`, `Deferred Decisions or Blockers`, `Open Questions`, `Risks and Mitigations`

## Intake Procedure

Before proposing migration:

1. Resolve the source plan path.
2. Start with targeted semantic intake:
   - metadata or equivalent status fields
   - goal or requested-outcome section
   - work structure, milestone, or task sections
   - progress or tracker section
   - verification section
   - blocker or open-question section when present
3. Identify the current artifact shape:
   - current canonical dense artifact
   - older standard artifact
   - custom but recoverable artifact
   - broken or incomplete artifact
4. Determine whether the artifact is already up to date, patch-migratable, rewrite-migratable, or unsafe to migrate.
5. Read deeper only when targeted intake is not enough to preserve execution truth safely.

## Structural Drift Checks

Check for the concrete kinds of drift this command should repair:

- old top-level sections such as `Executive Summary`, `Requested Outcome`, `Scope`, `Constraints and Compatibility`, `Requirements`, `Architecture and Design`, or `Roadmap and Milestones` still acting as separate owners where the current dense structure expects consolidation
- missing or weak `## Progress Tracking`
- tracker rows missing stable IDs, milestone ownership, or hard dependency mapping
- verification still living in an older unsplit section instead of `### Required Verification` and `### Latest Results`
- duplicated facts spread across too many sections
- stale progress wording that conflicts with the actual tracker
- metadata missing fields needed for truthful continued execution

If drift is only cosmetic and does not fail the `Already Up To Date` checklist, treat it as a no-op instead of migration work.

Do not treat harmless wording differences alone as enough reason to migrate.

## Migration Strategy Resolution

If migration is needed, choose one of these in-place save strategies for the final approval step.

### `patch_plan`

Use when:

- the artifact is mostly sound
- progress tracking is already trustworthy or locally repairable
- sections can be normalized without rewriting the whole document

Behavior:

- apply the smallest coherent edit set
- preserve unchanged wording and structure where it still holds
- normalize only the sections, tracker rows, metadata fields, and cross-references needed for a safe current artifact

### `rewrite_plan`

Use when:

- the artifact has broad structural drift
- older sections overlap too heavily to patch cleanly
- tracker or verification state must be remapped across the whole file to keep the result truthful
- patch mode would leave the document harder to trust than a clean in-place rewrite

Behavior:

- rewrite the full file in place into the current canonical structure
- preserve truthful progress, task ordering, dependencies, and verification reality
- preserve the existing filename and internal version metadata unless the user explicitly asks otherwise

## Migration Procedure

1. Resolve the source plan file.
2. Ingest enough of the artifact to understand its execution truth.
3. Determine whether the artifact is already up to date.
4. If not up to date, identify the exact structural drift and the minimum safe migration strategy.
5. Draft a concise migration summary before editing. The summary must include:
   - source file path
   - detected current artifact shape
   - whether migration is a patch or rewrite recommendation
   - the exact areas that are out of date
   - what progress or verification state will be preserved
   - any fields that cannot be recovered safely without user input
6. Show that summary before writing anything.
7. Ask one final confirmation question with the `question` tool.
8. Only after explicit approval, migrate the file in place.
9. Read back the modified regions to verify the migration landed consistently.

## Preview and Approval Flow

Before writing the migrated artifact:

1. Show a concise migration summary.
2. Show the exact target file path.
3. Show whether patching or rewriting is the better fit and why.
4. Show a compact drift summary listing:
   - sections or fields to be normalized
   - sections or fields preserved as-is
   - progress or verification state carried forward
5. Do not dump the full migrated markdown body in chat.
6. Ask one final confirmation question with the `question` tool.

Use this final confirmation question:

- header: `Finalize migrated plan`
- question: `The in-place migration is prepared. Choose whether to patch only the relevant parts of the existing plan or rewrite the full file into the current structure.`
- options:
  - `Patch plan` -> patch only the relevant parts of the existing file
  - `Rewrite plan` -> rewrite the full plan file in place
  - `Revise` -> adjust the migration plan before writing
  - `Cancel` -> stop without writing
- allow typed custom answer

Response handling:

- `Patch plan`: persist using `patch_plan`
- `Rewrite plan`: persist using `rewrite_plan`
- `Revise`: continue analysis and do not write yet
- `Cancel`: stop without writing
- ambiguous custom responses require one follow-up question

## Save Flow

If the user approves migration:

1. If the user chose `patch_plan`, apply the localized migration edits back to the resolved source path.
2. If the user chose `rewrite_plan`, rewrite the full migrated markdown artifact back to the resolved source path.
3. Do not create a new artifact path.
4. Report:
   - save strategy used
   - saved path
   - whether the plan was migrated or already current
   - that the plan remains implementation-ready for continued work

If the current run cannot write directly:

1. Do not pretend the file was saved.
2. Tell the user the migration is fully ready.
3. State that the next step is saving it in build/write mode.

## Final Output

- On success: report the migrated file path, the save strategy used, and the structural areas that were normalized.
- On no-op: report that the artifact was already up to date and no file was changed.
- On cancel: report that no files were changed.

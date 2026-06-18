---
description: Revise an existing implementation plan into a standalone new version by default or an explicit in-place update
---

# Revise Plan Harness

User input: `$ARGUMENTS`

## Mission

Revise and improve one existing markdown implementation plan artifact.

By default, the result must be a stronger, more current, implementation-ready standalone plan artifact saved as a new version.

If and only if the user explicitly asks for an in-place update, update the existing plan file directly instead of creating a new version.

When revising in place, the final save decision must let the user choose between rewriting the full plan or patching only the relevant parts of the existing file.

The command must:

- ingest enough of the source plan before proposing revisions, then deepen reads only when needed
- preserve valid decisions and truthful progress that still hold
- challenge weak architecture, stale assumptions, tracker drift, and vague execution detail
- incorporate improvement points from the current chat context, referenced prior messages, explicit user guidance, and repository evidence
- default to creating a standalone new versioned plan artifact
- switch to in-place update behavior only when the user explicitly asks to update the existing file instead of creating a new version
- in `in_place` mode, default revision behavior to preserving the existing file structure and wording wherever they still hold unless the user explicitly chooses a full rewrite at final save time
- apply strict delta discipline so the revision stays proportional to the approved change
- prefer the dense canonical structure for new standalone artifacts unless preserving the older shape is materially safer for compatibility or partially executed work
- in default new-version mode, ensure the saved artifact stands alone and does not reference the source plan artifact it was revised from
- keep milestone and task tracking explicit enough for bounded execution commands to consume without guessing

The revised artifact must function as both:

- an implementation-ready technical plan
- a concise execution tracker for agents or humans doing the work

- In `new_version` mode, the final artifact must be written to the current workspace at `.agent/PLAN-v<N>-<title>-<YY-MM-DD-HH-MM>.md`.
- In `in_place` mode, the final artifact must be written back to the resolved source plan path.
- `<title>` must be lowercase kebab-case.
- In `new_version` mode, `v<N>` is workspace-global, not per-title.
- In `new_version` mode, the timestamp must use local time in `YY-MM-DD-HH-MM` format.
- The source plan is revision input, not output lineage, in `new_version` mode.
- In `in_place` mode, do not create a new version number, timestamped filename, or duplicate artifact unless the user explicitly asks for that instead.
- The revision is not complete until material ambiguity, stale assumptions, or plan weakness is either fixed or explicitly recorded as deferred with user awareness.

## Hard Fail Rules

Fail immediately without drafting revisions if any of these are true:

1. No existing plan artifact can be identified.
2. Multiple plausible source plan files match and the user, current chat, and workspace do not clearly disambiguate one target.
3. The resolved source file is outside the workspace or is not a markdown file.
4. The resolved file does not look like a saved implementation plan artifact because it lacks the minimum capabilities needed to revise safely, such as:
   - a goal or requested outcome
   - a work structure that can be revised
   - a progress/tracker structure or clearly inferable status markers
   - verification intent or completion checks
5. The run begins without first ingesting enough of the source markdown plan file to understand its goal, structure, status, and revision needs.
6. The user is actually asking for a brand new plan with no concrete source artifact to revise. In that case, stop and direct them to `/plan/create`.

If hard-failing, report the exact reason and ask the user for one valid source markdown plan file or to use `/plan/create` instead.

## Non-Negotiable Rules

1. Read enough of the source plan artifact before proposing meaningful revisions.
2. Treat the source plan as an evidence-bearing baseline, not as untouchable truth.
3. Ask as many questions as needed. Do not silently preserve weak assumptions just because they are already written down.
4. Never silently assume missing requirements, architecture, scope, compatibility, progress-state meaning, or testing detail.
5. Research before deciding. Prefer repository evidence, current plan content, implementation state, docs, config, tests, and prior local artifacts over intuition.
6. If evidence is missing, say so and ask.
7. Before every `question` tool call, emit a short markdown explanation covering:
   - current understanding
   - what is still ambiguous
   - why the next decision matters
8. Do not duplicate the exact question options in that pre-question explanation.
9. Use concise option labels and brief descriptions in `question` tool prompts so the user understands the impact of each choice.
10. If the source plan contains weak architecture, stale execution structure, over-centralization, or tracker clutter, say so plainly, explain why, propose the stronger alternative, and ask for explicit confirmation before preserving the weaker version.
11. If the user explicitly insists on the weaker option, honor it, but record the decision and tradeoff clearly in the revised artifact.
12. Prefer small cohesive modules, small focused functions, clean boundaries, high cohesion, low coupling, explicit interfaces, and strong testability.
13. Avoid giant files, giant functions, centralized god modules, or single-file dumping grounds unless the user explicitly asks for that style.
14. Default output mode is `new_version`. Use `in_place` only when the user's wording clearly directs updating the existing plan file instead of creating a new one.
15. In `new_version` mode, do not overwrite the source plan artifact. Save a new versioned artifact that reads as a standalone current plan and does not mention the source plan path, filename, version, or supersession lineage.
16. In `in_place` mode, overwrite only the resolved source plan file. Do not create a new versioned artifact, new timestamped filename, or duplicate copy unless the user explicitly changes that instruction.
17. Preserve truthful execution state when the source plan has already been partially executed. Do not reset real progress back to `pending` just because the plan structure improved.
18. In `in_place` mode, do not decide between full rewrite versus local patch silently. Present both options at final approval time, explain which one is better for this revision, and let the user choose.
19. In `in_place` patch mode, prefer the smallest coherent edit set that satisfies the approved revision. Preserve unchanged sections, section order, and wording wherever possible unless leaving them untouched would create contradictions.
20. If `in_place` patch mode would leave material inconsistencies, say so plainly, recommend rewrite mode in the final preview, but still allow the user to choose patch mode and record the remaining limits or tradeoffs in the saved artifact.
21. Do not write the final artifact until the user has seen a full preview of the proposed revised plan and explicitly approved saving it.
22. If the current run cannot write the file directly, stop after the preview and tell the user the revised plan is ready to save in build/write mode.

## Delta Discipline

Apply revision pressure to the changed parts, not the whole file by habit.

1. Preserve unchanged sections when they are still truthful.
2. Collapse stale execution residue instead of preserving every historical note verbatim.
3. Remove duplicated rationale rather than rephrasing the same point into additional sections.
4. If one section already carries a point adequately, update the owning section instead of restating it elsewhere.
5. Keep the revised artifact as short as truth allows for the approved scope.
6. For narrow revisions, prefer localized edits, compact tracker changes, and concise wording over full-plan expansion.
7. Do not re-expand a dense artifact back into a larger ceremonial shape unless the user explicitly asks for that style.

## Section Role Discipline

Every major section owns one primary job. Keep sections tight and non-overlapping.

1. `## Current Context and Evidence` is for observed facts only.
2. `## Scope Boundaries` defines boundaries only.
3. `## Decisions and Open Questions` is for user-decided answers, tradeoffs, and still-open items.
4. `## Execution Plan` explains structure, milestones, tasks, and change surfaces, not a second goal section.
5. `## Progress Tracking` is a compact tracker only, not a diary.
6. `## Verification` defines required checks and the latest concise results only.
7. `## Goal and Completion Signal` contains the objective and end-state completion conditions only, not milestone evidence history.

## Revision Priorities

When deciding what to improve, use this priority order:

1. the user's explicit revision request in the current message and `$ARGUMENTS`
2. referenced current-chat context and prior messages in this conversation
3. concrete repository evidence showing plan drift, missing detail, or changed implementation reality
4. weaknesses found inside the source plan itself

If `$ARGUMENTS` is empty or vague, infer the improvement targets from the current chat context first, then the workspace, then the source plan.

## Input Handling

- `$ARGUMENTS` may contain:
  - a source plan path or filename
  - a new improvement brief
  - both together
- Accept these source plan path forms when they resolve to one existing markdown file:
  - absolute path
  - workspace-relative path
  - exact filename
- If `$ARGUMENTS` clearly identifies a source plan file, use it.
- If `$ARGUMENTS` only describes desired improvements, infer the source plan from current chat context first, then workspace evidence, then ask the user to confirm if needed.
- If `$ARGUMENTS` is empty, infer both the likely source plan and likely improvement goals from current chat context first, then workspace files, then ask the user to confirm or correct that understanding.
- If multiple plausible source plans exist, do not guess. Ask the user to disambiguate.
- Treat improvement text as revision direction, not as a request to ignore the existing source plan.
- Ignore older workspace plan artifacts unless they are the source plan, directly referenced as comparison material, or needed to determine version history.
- Unless the user explicitly requests otherwise, treat the revision as `new_version` output.
- Do not infer `in_place` from generic verbs like `revise`, `update`, `modify`, `improve`, or `rewrite` by themselves.
- If the user supplied a new title, use it unless they later change it.
- If no new title was supplied, retain the source plan title unless the revision materially changes scope or intent.

## Output Mode Resolution

Resolve output mode before preview or save behavior.

Rules:

1. `new_version` is the default mode.
2. Use `in_place` only if the user clearly instructs that the existing file itself must be updated rather than producing a new artifact.
3. Wording that is sufficient to trigger `in_place` includes phrases such as:
   - `in-place`
   - `in place`
   - `overwrite the existing plan`
   - `update the existing file`
   - `modify this exact file`
   - `save back to the same file`
   - `do not create a new version`
   - `replace the current plan artifact`
4. Wording that is not sufficient by itself to trigger `in_place` includes phrases such as:
   - `revise the plan`
   - `update the plan`
   - `modify the plan`
   - `improve the plan`
   - `tighten the plan`
   - `rewrite the plan`
5. If the user's wording is mixed, conflicting, or still ambiguous about overwrite-vs-new-file intent, ask one direct question before saving.
6. Once output mode is resolved, follow it exactly for preview text, metadata shaping, and file persistence.

In `in_place` mode, the command must also resolve a final save strategy during the approval step:

- `rewrite_plan`: rewrite the full plan file into a refreshed self-contained artifact
- `patch_plan`: apply the smallest coherent set of edits to the existing file while preserving unchanged structure and wording where possible

## Phase 1: Source Plan Intake

Before proposing changes, ingest and understand the source plan artifact itself.

Minimum intake targets:

- source plan metadata and current status
- requested outcome and scope boundaries
- decisions, open questions, and compatibility constraints
- milestones, tasks, and task ordering
- progress tracker state
- change-surface sections such as files, modules, APIs, or contracts when present
- verification expectations and latest concise results
- completion conditions
- blockers and deferred items

Intake rules:

1. Separate what the source plan explicitly states from what it merely implies.
2. Flag stale, weak, duplicated, or contradictory sections.
3. Identify where the source plan is still strong so those decisions are preserved.
4. If the source plan already reflects partial implementation progress, treat that as something to validate, not something to erase.

## Semantic Alias Matrix

Use the same semantic mapping used by `/plan/do`, `/plan/do-partial`, `/plan/status`, and `/plan/explain` when reading old or custom artifacts.

- goal: `Goal and Completion Signal`, `Requested Outcome`, `Executive Summary`, `Summary`, `Objective`
- scope: `Scope Boundaries`, `Scope`, `In Scope`, `Out of Scope`
- decisions: `Decisions and Open Questions`, `Resolved Clarifications`, `Ambiguity Audit`, `Constraints and Compatibility`, `Alternatives Considered`
- execution: `Execution Plan`, `Roadmap and Milestones`, `Implementation Breakdown`, `Task Breakdown`, `File and Module Surface`, `File and Module Plan`
- progress: `Progress Tracking`, `Tracker`, `Task Tracker`, `Checklist`, milestone tables, task tables
- verification: `Verification`, `Testing and Verification`, `Required Verification`, `Latest Results`
- blockers: `Risks and Blockers`, `Deferred Decisions or Blockers`, `Open Questions`, `Risks and Mitigations`

## Phase 2: Ground Truth Re-Discovery

Before locking revisions, inspect the real workspace and gather evidence.

Minimum discovery targets when relevant:

- repository roots and project layout
- source roots
- test roots
- manifest and config files
- build/test/lint commands
- relevant docs, ADRs, README files, and API contracts
- existing modules that the source plan would extend, replace, or interact with
- relevant data models, interfaces, schemas, routes, commands, jobs, workflows, or UI surfaces
- implemented work that appears to satisfy or contradict the source plan's current tracker

Discovery rules:

1. Separate observed facts from inferred hypotheses.
2. If a hypothesis matters, validate it or ask.
3. Prefer reading the smallest set of high-value files that materially reduce revision ambiguity.
4. If the request depends on external behavior or standards, fetch or inspect the authoritative source when practical instead of guessing.
5. If repository reality has drifted from the source plan, revise the plan to match reality only where the evidence is clear and the approved outcome still holds.
6. If repo drift changes scope, behavior, or architecture materially, stop and ask the user instead of freelancing.

## Phase 3: Clarification Loop

Run an iterative clarification loop until there are no material silent assumptions left in the revised plan.

Question loop rules:

1. Ask in small batches. Prefer `1-4` questions per `question` tool call.
2. Prefer multiple focused rounds over one giant questionnaire.
3. Use concrete options when there are real design branches.
4. Keep custom typed answers enabled.
5. If a custom answer is ambiguous, ask a follow-up.
6. After every meaningful answer batch, restate the newly locked decisions before moving on.
7. Do not draft the final revised plan until the relevant categories below are either:
   - explicit and agreed
   - explicitly marked `N/A`
   - explicitly deferred by the user

Clarification checklist. Investigate every relevant category, not just the obvious ones:

1. which source plan file is being revised
2. what outcome the revision is trying to improve or correct
3. which parts of the source plan should be preserved versus replaced
4. scope boundaries and non-goals
5. user-facing behavior and success criteria
6. title changes if needed
7. architecture and module boundaries
8. data models, schemas, persistence, or migrations
9. public interfaces, APIs, commands, events, or protocols
10. compatibility and breaking-change expectations
11. rollout, deployment, or release constraints
12. performance, reliability, concurrency, and failure-mode expectations
13. security, privacy, secrets, and permission constraints
14. progress carry-forward expectations when work already started
15. test strategy and verification depth
16. coding standards or repo-specific conventions that materially affect the work
17. documentation and operational updates required by the change

Ask only questions that materially reduce uncertainty. Do not ask obvious noise questions when the answer is already explicit from the user, the source plan, or the codebase.

## Revision Challenge Protocol

When the requested revision direction or the source plan looks unsound:

1. Say clearly what is wrong.
2. Explain the likely cost, brittleness, or failure mode.
3. Offer the stronger alternative.
4. Ask the user whether they want the stronger design or still want the weaker one preserved.
5. If they insist on the weaker design, continue, but record:
   - the stronger rejected alternative
   - the user-confirmed weaker choice
   - the explicit tradeoffs and risks

Do not hide your engineering judgment to be polite. Be blunt, concrete, and useful.

## Revision Completeness Gate

The revised plan is not ready until all of the following are true:

1. The source plan and requested improvements are understood in concrete terms.
2. Scope and non-goals are explicit.
3. Relevant architecture decisions are explicit.
4. Required files, modules, packages, services, or areas of change are identified when the workspace allows that level of precision.
5. Any carried-forward implementation progress is mapped truthfully.
6. Testing and verification are specific enough to execute.
7. Acceptance criteria are concrete.
8. No material ambiguity remains unmentioned.

If something cannot be fully resolved, do not guess. Put it in `### Open Questions or Deferred Decisions` or `## Risks and Blockers`, whichever fits best, with the reason it is still open.

## Versioning and Filename Rules

When the user approves the revised plan for saving, apply the rules for the resolved output mode.

### `new_version`

1. Ensure the workspace `.agent/` directory exists, creating it if necessary.
2. Determine `N` using workspace-global plan history:
   - if files matching `.agent/PLAN-v*-*.md` exist, use `max(existing version) + 1`
   - otherwise, if legacy `.agent/PLAN-*.md` files exist without version numbers, treat them as historical plans and use `count(legacy plans) + 1`
   - otherwise use `v1`
3. Choose the title:
   - use the user's explicit new title if provided
   - otherwise retain the source plan title unless the approved revision materially changes intent
4. Normalize the filename title to lowercase kebab-case.
5. Use local time for the timestamp in `YY-MM-DD-HH-MM` format.
6. Never overwrite the source plan artifact.

### `in_place`

1. Write the final artifact back to the resolved source plan path.
2. Do not create a new filename, new `v<N>`, or timestamp suffix.
3. Preserve the existing file path.
4. Preserve the existing `version` metadata value unless the user explicitly asks to change the plan's internal version label.
5. Do not rename the file to match a new title unless the user explicitly asks for a rename.
6. If the artifact includes `file path` metadata, update it to the actual persisted path.

## In-Place Save Strategies

When output mode is `in_place`, the final approval step must offer two save strategies.

### `rewrite_plan`

Use when:

- the revision touches many sections
- the plan has stale structure, duplicated content, or internal contradictions
- task, milestone, or architecture reshaping is broad enough that local edits would be harder to trust than a rewrite

Behavior:

- rewrite the full file in place as the updated canonical artifact
- preserve truthful execution state, file path, and internal version metadata unless the user explicitly changes them
- use the full revised section structure needed for a clear implementation-ready artifact

### `patch_plan`

Use when:

- the requested revision is narrow or localized
- the existing plan is mostly sound
- preserving wording, local rationale, or tracker continuity is higher value than normalizing the whole document

Behavior:

- apply the smallest coherent edit set that satisfies the approved revision
- preserve unchanged sections, section order, headings, and wording wherever they still hold
- do not expand untouched sections just to make them longer or more canonical
- update only the sections, tracker rows, metadata fields, and cross-references needed to keep the file truthful and internally consistent
- if patch mode cannot fully remove a known weakness without broader rewrite, record that clearly in the artifact rather than hiding it

## Final Artifact Requirements

The revised artifact must be a markdown file and must be comprehensive enough that a future agent or human developer can start or continue implementation immediately.

It must also include a concise roadmap, task tracker, and checklist structure so execution can be tracked without inventing a separate project-management document.

The task-tracking structure must be explicit enough that `/plan/do`, `/plan/do-partial`, and `/plan/status` can consume the artifact without guessing task ownership, dependency direction, or execution order.

In `new_version` mode, the saved markdown must read as a standalone current plan. It must not contain lineage fields or prose such as `revision_of`, `supersedes`, `revised from`, `based on PLAN-v...`, source-plan filenames or paths, or any statement that requires the reader to inspect an earlier plan artifact to understand the current one.

In `in_place` mode, the saved markdown is the updated source plan itself. It must still be self-contained, but it may preserve the file's existing version label and file path because those belong to the same artifact rather than pointing at a different one.

Required section order:

Apply this required order to:

- all `new_version` artifacts
- `in_place` saves using `rewrite_plan`

For `in_place` saves using `patch_plan`, preserve the existing plan's section order and headings when they remain intelligible and truthful. Do not force a full structural rewrite solely to match the canonical order.

1. `# Plan: <Human Title>`
2. `## Metadata`
3. `## Revision History`
4. `## Goal and Completion Signal`
5. `## Current Context and Evidence`
6. `## Scope Boundaries`
7. `## Decisions and Open Questions`
8. `## Engineering Standards`
9. `## Execution Plan`
10. `## Progress Tracking`
11. `## Verification`
12. `## Risks and Blockers`
13. `## References`

Section requirements:

Every required section has one primary job. Keep sections compact and avoid duplicated prose across adjacent sections.

### `## Metadata`

Must include at least:

- version
- title
- file path
- created_at_local
- `updated_at_local` when saving `in_place`; preserve any existing `created_at_local` if the source plan already has one
- workspace_root
- request_source summarizing the user request and refinements in clean prose; in `new_version` mode, do not copy raw source plan filenames or paths into this field
- revision_mode as `new_version` or `in_place`
- revision_reason summarizing the approved improvement direction
- status reflecting real execution state:
  - `approved draft` if implementation has not begun
  - `in_progress` if work is already underway
  - `blocked` if the carried-forward plan state is blocked
  - `completed` only if the revised plan is documenting fully completed work

Additional metadata rules:

- In `new_version` mode, do not include `revision_of`, `supersedes`, source-plan path fields, or equivalent lineage metadata.
- In `in_place` mode, do not add cross-artifact lineage metadata; the revised file is the artifact.
- In `in_place` `patch_plan` mode, update only the metadata fields needed to keep the artifact accurate, such as `updated_at_local`, `request_source`, `revision_mode`, and `revision_reason`, while preserving stable fields unless the user explicitly changes them.

### `## Goal and Completion Signal`

This section replaces the older split between summary, requested outcome, requirements recap, and acceptance recap.

It must include:

- the concrete objective
- the intended end state
- the smallest set of completion conditions needed to know the revised plan is truly done

### `## Scope Boundaries`

Structure it as:

- `### In Scope`
- `### Out of Scope`

Keep this section boundary-focused. Do not turn it into architecture prose.

### `## Current Context and Evidence`

List the concrete workspace files, docs, interfaces, commands, tracker state, and implementation observations used to ground the revision.

In `new_version` mode, do not cite the source plan filename or path inside the saved artifact. Carry forward validated facts, but rewrite them as current-plan context.

### `## Revision History`

This section is required for artifacts produced by `/plan/revise`, even on the first revision.

Rules:

- keep it super compact
- keep entries in chronological order with newest last
- one entry per revise invocation
- do not turn it into a general execution diary
- do not list every tiny edit
- do not make any other command depend on this section for execution logic

Each entry must include, in one compact bullet or table row:

- local revision timestamp in `YY-MM-DD-HH-MM` 24-hour format
- revision mode used
- save strategy when `in_place`
- pre-revision plan status
- short revision reason
- short summary of what changed

In `new_version` mode, do not use this section to point at source filenames or paths. Keep it as compact user-audit context only.

### `## Decisions and Open Questions`

This section replaces the older split between clarifications, ambiguity audit, constraints, alternatives, and deferred items.

Structure it as:

- `### Resolved Decisions`
- `### Open Questions or Deferred Decisions`

Include here when relevant:

- important user decisions captured during the revision loop
- what was ambiguous, stale, weak, or contradictory in the prior plan or request
- how each issue was resolved
- major tradeoffs and rejected stronger alternatives
- what remains intentionally deferred, if anything
- a clear statement that no silent assumptions remain in the approved revised plan

In `new_version` mode, do not turn this section into file-to-file lineage notes or source-plan comparison prose.

In `in_place` `patch_plan` mode, if the user chooses patching despite a recommended rewrite, record the unresolved or intentionally preserved weaknesses and the explicit tradeoff.

### `## Engineering Standards`

Emphasize:

- small cohesive files
- small focused functions
- no large symbol dumps
- no god objects or central junk drawers
- clean interfaces and ownership boundaries
- tests for important behavior, edge cases, and regressions
- verification before claiming completion

If the user explicitly requested a lower-quality or more centralized approach, record that as a user-directed exception here.

### `## Execution Plan`

This section owns implementation structure, not repeated goals.

Structure it as:

- `### Milestones`
- `### Task Breakdown`
- `### File and Module Surface`
- `### Data, API, or Contract Changes`

### `### Milestones`

Provide a concise delivery roadmap that a human or agent can scan quickly.

Requirements:

- use milestone IDs such as `M1`, `M2`, `M3`
- milestone IDs must be stable and unique within the revised plan
- each milestone must have a short stable human label alongside its ID
- keep it short and dependency-ordered
- each milestone must state:
  - goal
  - key outputs
  - completion signal
- each milestone must also list the task IDs it owns in planned execution order
- avoid duplicating every low-level task here; this subsection is the high-level execution map

### `### Task Breakdown`

Translate the revised plan into concrete executable tasks.

Requirements:

- use task IDs such as `T1`, `T2`, `T3`
- task IDs must be stable and unique within the revised plan
- group tasks by milestone or phase when relevant
- each task should include:
  - short stable human label
  - owning milestone ID
  - objective
  - concrete change surface
  - hard dependencies using task IDs or `none`
  - expected deliverable
  - verification step
- tasks must distinguish real hard prerequisites from general notes or contextual sequencing comments
- task ordering inside each milestone must reflect the preferred execution order when multiple tasks could otherwise start around the same time
- tasks must be specific enough that a future agent can pick one up and execute it without re-planning the whole project
- if the source plan already had truthful completed work, preserve or remap it instead of regenerating a fake all-pending tracker

### `## Progress Tracking`

This section is mandatory and must be concise.

Include both:

1. a milestone checklist using markdown checkboxes like `- [ ] M1 ...`
2. a compact task tracker table with at least these columns:
   - `ID`
   - `Milestone`
   - `Status`
   - `Hard Depends On`
   - `Deliverable`
   - `Verification`

Tracking rules:

- initial status values should reflect reality, not habit
- use compact status labels such as `pending`, `in_progress`, `blocked`, `done`
- every task row must appear exactly once in the tracker table
- every task row must use the same stable task ID and milestone ID already defined elsewhere in the revised plan
- `Hard Depends On` values must use task IDs only or `none`, never vague prose like `backend first`
- tracker row order is the canonical execution-order tie-breaker when multiple tasks are otherwise runnable
- do not turn this section into a long diary or duplicate the whole plan narrative
- only include milestones and tasks that materially matter to implementation tracking
- the structure should be immediately usable by either a human developer or a future agent session

### `### File and Module Surface`

Map the intended changes to real files, packages, directories, modules, or surfaces when that information is available.

Do not repeat full task narratives here. This section should answer `where`, not re-explain `why` or `when`.

### `### Data, API, or Contract Changes`

Cover only what is relevant. If not applicable, say so explicitly.

### `## Verification`

This section owns verification planning and the latest concise results, not an execution diary.

Structure it as:

- `### Required Verification`
- `### Latest Results`

Be specific in `### Required Verification`. Include:

- unit tests
- integration or end-to-end coverage where relevant
- regression coverage
- migration or compatibility checks where relevant
- concrete commands to run when they can be known from the workspace
- manual verification steps for behaviors not fully covered by automation
- if the source plan already tracked prior verification results that still matter, preserve them concisely and separate them from future required checks

For `### Latest Results`, keep only the latest material outcomes needed to understand current state. Collapse stale pass spam into one concise current summary.

If timestamped result entries are kept here, they must:

- use local `YY-MM-DD-HH-MM` 24-hour timestamps
- remain in chronological order with newest last

### `## Risks and Blockers`

Keep this section short.

Use it for:

- meaningful risks that could change execution shape
- real blockers or deferred items that still matter to completion

Do not use it as a dumping ground for trivia.

## Preview and Approval Flow

Before writing the revised artifact:

1. Show the user a concise summary of the finalized revised plan.
2. Show the resolved output mode.
3. Show the exact target file path.
4. Show the proposed title and version behavior:
   - in `new_version` mode, show the new version and title
   - in `in_place` mode, show that the existing file will be updated in place and whether the internal version field stays unchanged
5. In `in_place` mode, add one short line explaining whether `rewrite_plan` or `patch_plan` is the better fit for this specific revision and why.
6. Show a concise delta summary listing:
   - sections changed
   - sections preserved
   - sections condensed
7. If helpful, you may mention the source plan separately as revision input, but do not describe the saved `new_version` artifact as superseding or referencing it.
8. Do not dump the full markdown plan body in chat.
9. Ask one final confirmation question with the `question` tool.

Preview summary requirements:

- include the plan goal
- include the major architecture or implementation shape
- include milestone count or phase count when relevant
- include whether progress was carried forward from the source plan
- include notable risks, deferred decisions, or user-forced tradeoffs when relevant
- in `in_place` mode, include a short recommendation line explaining whether patching or rewriting is better for this save
- include the section delta summary in compact form
- keep it concise; the saved markdown artifact is the canonical full plan
- the full plan content should be read from the saved `.md` file after persistence, not echoed in full before save

Use this final confirmation question:

- header: `Save revised plan` in `new_version` mode
- header: `Finalize in-place plan` in `in_place` mode
- question:
  - use `The revised standalone plan is finalized and ready to save as a new artifact. Should I write it now?` in `new_version` mode
  - use `The in-place revision is finalized. Choose whether to rewrite the full plan or patch only the relevant parts of the existing file.` in `in_place` mode
- options:
  - in `new_version` mode:
    - `Save` -> write the revised artifact now
    - `Revise` -> continue clarification or edit the draft before saving
    - `Cancel` -> stop without writing
  - in `in_place` mode:
    - `Rewrite plan` -> rewrite the full plan file in place
    - `Patch plan` -> patch only the relevant parts of the existing file
    - `Revise` -> continue clarification or edit the draft before saving
    - `Cancel` -> stop without writing
- allow typed custom answer

Response handling:

- `Save`: persist the revised plan according to the resolved output mode
- `Rewrite plan`: in `in_place` mode, persist using `rewrite_plan`
- `Patch plan`: in `in_place` mode, persist using `patch_plan`
- `Revise`: continue the revision loop and do not write yet
- `Cancel`: stop without writing
- clear custom approvals or revisions map the same way
- ambiguous custom responses require one follow-up question

## Save Flow

If the user approves saving:

1. In `new_version` mode:
   - create `.agent/` if it does not exist
   - compute the new filename and full path
   - write the final revised markdown artifact there
2. In `in_place` mode:
    - if the user chose `rewrite_plan`, rewrite the final revised markdown artifact back to the resolved source path
    - if the user chose `patch_plan`, apply the localized approved edits back to the resolved source path while preserving unchanged structure and wording where possible
    - do not create a new artifact path
3. Report:
    - output mode
    - save strategy when `in_place`
    - saved path
    - version
    - title
   - note that the revised plan is approved and implementation-ready

If the current run cannot write directly:

1. Do not pretend the file was saved.
2. Tell the user the revised plan is fully ready.
3. State that the next step is saving it in build/write mode.

## Failure Handling

1. If the source plan path is unclear, inspect more or ask.
2. If the source plan and current repository contradict each other, surface the contradiction and resolve it before revising further.
3. If evidence is weak, label the uncertainty and ask rather than guessing.
4. If the user keeps answers intentionally broad, convert that breadth into explicit ranges, options, or deferred decisions instead of silent assumptions.
5. If the source plan has partial execution state that cannot be mapped cleanly into the new structure, say so explicitly and ask the user how they want that progress represented.
6. Never output a shallow revision that only sounds improved.

## Final Output Requirements

When the command run finishes:

- if not yet saved, clearly state whether the blocker is unresolved ambiguity, unresolved source-plan targeting, or missing final approval
- if saved, report only the essential outcome and the saved path
- keep the chat response concise because the saved artifact is the canonical revised plan

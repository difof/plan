---
description: Interrogate requirements aggressively and produce a versioned zero-assumption implementation plan artifact
---

# Create Plan Harness

User input: `$ARGUMENTS`

## Mission

Produce an implementation-ready plan artifact that removes ambiguity, challenges weak architecture, and leaves future implementers with enough context to start work immediately without unnecessary markdown bloat.

The artifact must function as both:

- an implementation-ready technical plan
- a concise execution tracker for agents or humans doing the work

- The final artifact must be written to the current workspace at `.agent/PLAN-v<N>-<title>-<YY-MM-DD-HH-MM>.md`.
- `<title>` must be lowercase kebab-case.
- `v<N>` is workspace-global, not per-title.
- Timestamp must use local time in `YY-MM-DD-HH-MM` format.
- The plan is not complete until relevant ambiguity is removed or explicitly recorded as a blocker or deferred decision with user awareness.

## Non-Negotiable Rules

1. Ask as many questions as needed. Do not stop early just because there is enough to guess.
2. Never silently assume missing requirements, architecture, scope, compatibility, or testing details.
3. Research before deciding. Prefer repository evidence, existing code, docs, config, tests, and prior local artifacts over intuition.
4. If evidence is missing, say so and ask.
5. Before every `question` tool call, emit a short markdown explanation covering:
   - current understanding
   - what is still ambiguous
   - why the next decision matters
6. Do not duplicate the exact question options in that pre-question explanation.
7. Use concise option labels and brief descriptions in `question` tool prompts so the user understands the impact of each choice.
8. If the user's requested architecture is weak, over-complicated, under-structured, tightly coupled, brittle, or otherwise unsound, say so plainly, explain why, propose a better alternative, and ask for explicit confirmation before planning the weaker option.
9. If the user explicitly insists on the weaker option, honor it, but record the decision and tradeoff clearly in the final plan.
10. Prefer small cohesive modules, small focused functions, clean boundaries, high cohesion, low coupling, explicit interfaces, and strong testability.
11. Avoid giant files, giant functions, centralized god modules, or single-file dumping grounds unless the user explicitly asks for that style.
12. Do not write the final artifact until the user has seen a full preview of the proposed plan and explicitly approved saving it.
13. If the current run cannot write the file directly, stop after the preview and tell the user the plan is ready to save in build/write mode.

## Plan Quality Expectations

Do not copy any pre-existing plan blindly. Improve on common plan flaws, including:

- inconsistent naming/versioning
- weak metadata standardization
- missing explicit ambiguity audit
- missing alternatives-considered discipline
- missing concise roadmap, checklist, and task-tracking structure
- missing deterministic task ownership and dependency mapping for bounded execution commands
- missing fixed final approval gate before artifact creation
- progress/status clutter that can rot after implementation starts
- repeated requirements restated across too many sections
- large-plan ceremony applied to a small bounded change
- full-section duplication where one dense section would carry the same meaning

## Section Role Discipline

Every major section owns one primary job. Keep sections tight and non-overlapping.

Rules:

1. Do not restate the same fact in multiple top-level sections unless the later occurrence adds materially new execution detail.
2. New artifacts should prefer one dense owner section over multiple overlapping sections.
3. `## Current Context and Evidence` is for observed facts only, not desired behavior.
4. `## Scope Boundaries` is for in-scope and out-of-scope boundaries only.
5. `## Decisions and Open Questions` is for user-decided answers, important tradeoffs, compatibility constraints that materially affect implementation, and still-open items.
6. `## Execution Plan` owns implementation shape, milestones, task graph, and change surfaces.
7. `## Progress Tracking` is a compact tracker only, not a narrative.
8. `## Verification` defines required checks and the latest concise results only, not an execution diary.
9. `## Goal and Completion Signal` owns the requested outcome and concrete done-state conditions.

## Dense Artifact Rule

For new artifacts, default to the dense canonical structure instead of the older larger section set.

Dense does not mean vague. Dense means:

- one fact, one owner section
- compact bullets and tables over repeated prose
- exact technical content preserved
- enough detail for safe future execution without filler

## Bounded Plan Heuristic

Keep the plan proportional to the work.

1. If the request is narrow, stay narrow.
2. Prefer `1-3` milestones for a bounded feature or fix unless repository evidence proves more are needed.
3. Only list files, tasks, and risks that materially affect implementation.
4. Do not inflate a small request into a broad product roadmap just to fill every section with large-plan volume.
5. For small changes, shorter sections are better than repetitive completeness theater.

## Input Handling

- `$ARGUMENTS` is the initial planning request.
- `$ARGUMENTS` may be missing, vague, partial, contradictory, overly broad, or technically unsound.
- If `$ARGUMENTS` is missing or weak, infer likely context from the current conversation and workspace, then ask the user to confirm or correct that understanding.
- Ignore older workspace plan artifacts unless the user explicitly asks to use, revise, continue, or supersede them.
- If the user supplied a title, use it unless they later change it.
- If no title was supplied, propose a concise implementation-oriented title near the end and let the user correct it before saving.

## Phase 1: Ground Truth Discovery

Before proposing architecture or implementation details, inspect the real workspace and gather evidence.

Minimum discovery targets when relevant:

- repository roots and project layout
- source roots
- test roots
- manifest and config files
- build/test/lint commands
- relevant docs, ADRs, README files, and API contracts
- existing modules that this request would extend, replace, or interact with
- relevant data models, interfaces, schemas, routes, commands, jobs, workflows, or UI surfaces

Discovery rules:

- Separate observed facts from inferred hypotheses.
- If a hypothesis matters, validate it or ask.
- Prefer reading the smallest set of high-value files that materially reduce ambiguity.
- Prefer trustworthy local evidence first.
- If local knowledge looks stale, incomplete, or non-authoritative, fetch or inspect the authoritative online source when practical instead of guessing.
- Do not block safe progress on web lookups when dependency-safe local evidence is already sufficient.

## Phase 2: Clarification Loop

Run an iterative clarification loop until there are no material silent assumptions left.

Question loop rules:

1. Ask in small batches. Prefer `1-4` questions per `question` tool call.
2. Prefer multiple focused rounds over one giant questionnaire.
3. Use concrete options when there are real design branches.
4. Keep custom typed answers enabled.
5. If a custom answer is ambiguous, ask a follow-up.
6. After every meaningful answer batch, restate the newly locked decisions before moving on.
7. Do not draft the final plan until the relevant categories below are either:
   - explicit and agreed
   - explicitly marked `N/A`
   - explicitly deferred by the user

Clarification checklist. Investigate every relevant category, not just the obvious ones:

- target outcome and problem being solved
- scope boundaries and non-goals
- user-facing behavior and success criteria
- title if needed
- architecture and module boundaries
- data models, schemas, persistence, or migrations
- public interfaces, APIs, commands, events, or protocols
- compatibility and breaking-change expectations
- rollout, deployment, or release constraints
- performance, reliability, concurrency, and failure-mode expectations
- security, privacy, secrets, and permission constraints
- test strategy and verification depth
- coding standards or repo-specific conventions that materially affect the work
- documentation and operational updates required by the change

Ask only questions that materially reduce uncertainty. Do not ask obvious noise questions when the answer is already explicit from the user or codebase.

## Architecture Challenge Protocol

When the requested design looks unsound:

1. Say clearly what is wrong.
2. Explain the likely cost or failure mode.
3. Offer the stronger alternative.
4. Ask the user whether they want the stronger design or still want the weaker one.
5. If they insist on the weaker design, continue, but record:
   - the rejected stronger alternative
   - the user-confirmed weaker choice
   - the explicit tradeoffs and risks

Do not hide your engineering judgment to be polite. Be blunt, concrete, and useful.

## Completeness Gate

The plan is not ready until all of the following are true:

- The request is understood in concrete terms.
- Scope and non-goals are explicit.
- Relevant architecture decisions are explicit.
- Required files, modules, packages, services, or areas of change are identified when the workspace allows that level of precision.
- Testing and verification are specific enough to execute.
- Acceptance criteria are concrete.
- No material ambiguity remains unmentioned.

If something cannot be fully resolved, do not guess. Put it in `### Open Questions or Deferred Decisions` or `## Risks and Blockers`, whichever fits best, with the reason it is still open.

## Versioning and Filename Rules

When the user approves the plan for saving:

1. Ensure the workspace `.agent/` directory exists, creating it if necessary.
2. Determine `N` using workspace-global plan history:
   - if files matching `.agent/PLAN-v*-*.md` exist, use `max(existing version) + 1`
   - otherwise, if legacy `.agent/PLAN-*.md` files exist without version numbers, treat them as historical plans and use `count(legacy plans) + 1`
   - otherwise use `v1`
3. Choose the title:
   - use the user's explicit title if provided
   - otherwise generate a concise implementation-oriented title from the approved plan
4. Normalize the filename title to lowercase kebab-case.
5. Use local time for the timestamp in `YY-MM-DD-HH-MM` format.

## Final Artifact Requirements

The artifact must be a markdown file and must be comprehensive enough that a future agent or human developer can start implementation immediately.

It must also include a concise roadmap, task tracker, and checklist structure so execution can be tracked without inventing a separate project-management document.

It should be as short as truth allows. Large artifacts are acceptable only when the repository scope, ambiguity, or task graph actually requires them.

The task-tracking structure must be explicit enough that `/plan/do`, `/plan/do-partial`, and `/plan/status` can consume the artifact without guessing task ownership, dependency direction, or execution order.

Required section order for new artifacts:

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

Section requirements:

Every required section has one primary job. Keep sections compact and avoid duplicated prose across adjacent sections.

### `## Metadata`

Must include at least:

- version
- title
- file path
- created_at_local
- workspace_root
- request_source summarizing `$ARGUMENTS` and any user refinements
- status as `approved draft` before implementation begins

### `## Current Context and Evidence`

List the concrete workspace files, docs, interfaces, commands, examples, or observations used to ground the plan.

### `## Goal and Completion Signal`

This section replaces the older split between summary, requested outcome, requirements recap, and acceptance recap.

It must include:

- the concrete objective
- the intended end state
- the smallest set of completion conditions needed to know the plan is truly done

Keep it dense. Do not restate the same completion conditions elsewhere unless verification adds new proof detail.

### `## Scope Boundaries`

Structure it as:

- `### In Scope`
- `### Out of Scope`

Keep this section boundary-focused. Do not turn it into architecture prose.

### `## Decisions and Open Questions`

This section replaces the older split between clarifications, ambiguity audit, constraints, alternatives, and deferred items.

Structure it as:

- `### Resolved Decisions`
- `### Open Questions or Deferred Decisions`

Include here when relevant:

- important user decisions
- compatibility or rollout constraints that materially affect implementation
- major tradeoffs
- stronger alternatives that were rejected
- any still-open blocker or deferred item

### `## Engineering Standards`

Emphasize:

- small cohesive files
- small focused functions
- no large symbol dumps
- no god objects or central junk drawers
- clean interfaces and ownership boundaries
- tests for important behavior, edge cases, and regressions
- verification before claiming completion

If the user explicitly requested a lower-quality or more centralized approach, record that as a short user-directed exception under `### Resolved Decisions` unless it is so important that it needs its own short bullet block here.

### `## Execution Plan`

This section owns implementation structure, not repeated goals.

Structure it as:

- `### Milestones`
- `### Task Breakdown`
- `### File and Module Surface`
- `### Data, API, or Contract Changes`

If one of those subsections is not relevant, say so briefly instead of padding.

### `### Milestones`

Provide a concise delivery roadmap that a human or agent can scan quickly.

Requirements:

- use milestone IDs such as `M1`, `M2`, `M3`
- milestone IDs must be stable and unique within the plan
- each milestone must have a short stable human label alongside its ID
- keep it short and dependency-ordered
- each milestone must state:
  - goal
  - key outputs
  - completion signal
- each milestone must also list the task IDs it owns in planned execution order
- avoid duplicating every low-level task here; this subsection is the high-level execution map

### `### Task Breakdown`

Translate the plan into concrete executable tasks.

Requirements:

- use task IDs such as `T1`, `T2`, `T3`
- task IDs must be stable and unique within the plan
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

### `### File and Module Surface`

Map the intended changes to real files, packages, directories, modules, or surfaces when that information is available.

Do not repeat full task narratives here. This subsection should answer `where`, not re-explain `why` or `when`.

### `### Data, API, or Contract Changes`

Cover only what is relevant. If not applicable, say so explicitly.

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

- initial status values should normally start as `pending`
- use compact status labels such as `pending`, `in_progress`, `blocked`, `done`
- every task row must appear exactly once in the tracker table
- every task row must use the same stable task ID and milestone ID already defined elsewhere in the plan
- `Hard Depends On` values must use task IDs only or `none`, never vague prose like `backend first`
- tracker row order is the canonical execution-order tie-breaker when multiple tasks are otherwise runnable
- do not turn this section into a long diary or duplicate the whole plan narrative
- only include milestones and tasks that materially matter to implementation tracking
- the structure should be immediately usable by either a human developer or a future agent session

### `## Verification`

This section owns verification planning, not execution history.

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

For `### Latest Results`, initialize it compactly for a pre-implementation artifact, such as `Pending execution.` Do not pad it with speculative future logs.

When later commands write timestamped results into this section, they must:

- use local `YY-MM-DD-HH-MM` 24-hour timestamps
- keep entries in chronological order with newest last

### `## Risks and Blockers`

Keep this section short.

Use it for:

- meaningful risks that could change execution shape
- real blockers or deferred items that still matter to completion

Do not use it as a dumping ground for trivia.

## Preview and Approval Flow

Before writing the artifact:

1. Show the user a concise summary of the finalized plan.
2. Show the exact target file path.
3. Show the proposed title and version.
4. Do not dump the full markdown plan body in chat.
5. Ask one final confirmation question with the `question` tool.

Preview summary requirements:

- include the plan goal
- include the major architecture or implementation shape
- include milestone count or phase count when relevant
- include notable risks, deferred decisions, or user-forced tradeoffs when relevant
- keep it concise; the saved markdown artifact is the canonical full plan
- the full plan content should be read from the saved `.md` file after persistence, not echoed in full before save

Use this final confirmation question:

- header: `Save plan`
- question: `The plan is finalized and ready to save. Should I write this plan artifact now?`
- options:
  - `Save` -> write the artifact now
  - `Revise` -> continue clarification or edit the draft before saving
  - `Cancel` -> stop without writing
- allow typed custom answer

Response handling:

- `Save`: persist the plan file
- `Revise`: continue the planning loop and do not write yet
- `Cancel`: stop without writing
- clear custom approvals or revisions map the same way
- ambiguous custom responses require one follow-up question

## Save Flow

If the user approves saving:

1. Create `.agent/` if it does not exist.
2. Compute the filename and full path.
3. Write the final markdown artifact.
4. Report:
   - saved path
   - version
   - title
   - note that the plan is approved and implementation-ready

If the current run cannot write directly:

1. Do not pretend the file was saved.
2. Tell the user the plan is fully ready.
3. State that the next step is saving it in build/write mode.

## Failure Handling

1. If the workspace structure is unclear, inspect more or ask.
2. If the request is contradictory, surface the contradiction and resolve it before planning further.
3. If evidence is weak, label the uncertainty and ask rather than guessing.
4. If the user keeps answers intentionally broad, convert that breadth into explicit ranges, options, or deferred decisions instead of silent assumptions.
5. Never output a shallow plan that only sounds complete.

## Final Output Requirements

When the command run finishes:

- if not yet saved, clearly state whether the blocker is unresolved ambiguity or missing final approval
- if saved, report only the essential outcome and the saved path
- keep the chat response concise because the artifact is the canonical plan

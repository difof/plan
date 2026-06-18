---
description: Read-only implementation status audit for a markdown plan file using semantic intake and in-chat markdown table reporting
---

# Plan Status Harness

User input: `$ARGUMENTS`

## Mission

Audit one markdown plan file against the real workspace in read-only mode.

- Ingest enough of the plan to audit it fairly.
- Never change files, git state, plan status, or workspace content.
- Determine what is `done`, `partial`, `not_done`, `missing`, `blocked`, or `unclear`.
- If the workspace is a git repo, include read-only git condition.
- Return markdown tables in chat, followed by a short summary or `## TL;DR`.

## Input Handling

- the provided command input must resolve to one markdown file path or filename to audit
- Accept these forms when they resolve to one existing markdown file:
  - absolute path
  - workspace-relative path
  - exact filename
- if no plan file input was provided, fail and ask the user for the plan file path
- if the provided command input resolves to more than one file, fail and ask for an exact path
- do not treat the provided command input as new planning requirements; it is only the plan file to inspect

## Hard Fail Rules

Fail immediately without starting analysis if any of these are true:

1. No plan artifact can be identified.
2. Multiple plausible plan files match and the provided command input does not clearly disambiguate.
3. The resolved file is outside the workspace or is not a markdown file.
4. The file does not provide enough plan-like structure to audit implementation status meaningfully.
5. The run begins without first ingesting enough of the plan to identify the goal, work structure, current status signals, and verification expectations.
6. The command attempts to modify any workspace file, create any artifact, or mutate git state.
7. The provided command input is not a path-like reference to one markdown file.

If hard-failing, report the exact reason and ask the user for a valid execution-ready markdown plan file.

## Non-Negotiable Rules

1. Read enough of the plan artifact before performing any implementation-status judgment.
2. Prefer semantic targeted intake first; pull the entire plan only when the document is small, messy, contradictory, or cannot be audited safely from targeted sections.
3. This command is strictly read-only.
4. Never edit the plan file.
5. Never write a report file, scratch file, or tracker file.
6. Never mutate git state in any way.
7. Read-only git inspection is allowed only to assess repository condition.
8. Do not trust the plan's existing tracker blindly. Compare it to repository evidence.
9. Base every status judgment on concrete workspace evidence, explicit absence of expected implementation, or both.
10. If evidence is mixed or insufficient, report `unclear` rather than pretending certainty.
11. Use concise, high-signal reporting. The output is an audit, not a diary.
12. The final chat response must end with a short summary section or `## TL;DR`.
13. Accept both older monolithic `## Testing and Verification` sections and newer split structures using `### Required Verification` plus `### Latest Results`.
14. Treat `## Revision History` as optional audit context only, never as primary implementation proof.
15. Do not assume acceptance evidence lives inside `## Acceptance Criteria`; newer artifacts may keep that section criteria-focused and place execution evidence elsewhere.
16. Parse by semantic role rather than exact heading names whenever possible.

## Plan Intake Procedure

Before any workspace scan:

1. Resolve the plan path.
2. Start with targeted section discovery by semantic role.
3. Extract at minimum when available:
   - metadata and declared status
   - document shape and whether it appears standard, legacy-standard, or custom
   - goal/requested outcome
   - tracker, tasks, milestones, or equivalent work structure
   - verification expectations and latest concise results
   - blockers, deferred items, or equivalent warning sections
4. Build the audit checklist from the plan itself before scanning the workspace.
5. Read deeper only when targeted intake is not enough for a fair status call.

## Semantic Alias Matrix

Use the same semantic mapping used by `/plan/explain`, `/plan/do`, and `/plan/do-partial`.

- goal: `Goal and Completion Signal`, `Requested Outcome`, `Executive Summary`, `Summary`, `Objective`
- scope: `Scope Boundaries`, `Scope`, `In Scope`, `Out of Scope`
- decisions: `Decisions and Open Questions`, `Resolved Clarifications`, `Ambiguity Audit`, `Constraints and Compatibility`, `Alternatives Considered`
- execution: `Execution Plan`, `Roadmap and Milestones`, `Implementation Breakdown`, `Task Breakdown`, `File and Module Surface`, `File and Module Plan`
- progress: `Progress Tracking`, `Tracker`, `Task Tracker`, `Checklist`, milestone tables, task tables
- verification: `Verification`, `Testing and Verification`, `Required Verification`, `Latest Results`
- blockers: `Risks and Blockers`, `Deferred Decisions or Blockers`, `Open Questions`, `Risks and Mitigations`

## Read-Only Audit Scope

Inspect the real workspace and compare it to the plan.

Audit these areas when present and relevant:

- milestone completion state
- task completion state
- claimed progress versus actual implementation evidence
- file and module existence and change shape
- requirement coverage
- acceptance-criteria coverage
- test presence and verification coverage
- deferred blockers and whether they still appear real
- repo working-tree condition
- obvious drift between plan expectations and the current codebase

## Status Taxonomy

Use these labels consistently in the report:

- `done`: implementation and verification evidence materially satisfy the planned item
- `partial`: some implementation exists, but important parts are missing, unverified, or inconsistent
- `not_done`: the item is planned but there is no convincing evidence that work started meaningfully
- `missing`: the plan expects a concrete file, module, behavior, test, or interface that cannot be found where evidence suggests it should exist
- `blocked`: the plan or workspace shows a real blocker preventing completion
- `unclear`: evidence is mixed, indirect, stale, or insufficient for a confident call
- `n/a`: not applicable in the current workspace or plan

## Read-Only Git Inspection

If the workspace is a git repository, include read-only repository condition in the audit.

Allowed examples:

- `git rev-parse --is-inside-work-tree`
- `git status --short --branch`
- `git diff --stat`
- `git diff --cached --stat`

Rules:

1. Never stage, unstage, commit, stash, reset, checkout, switch, merge, rebase, fetch, pull, or push.
2. Report git state as evidence, not as a task recommendation dump.
3. If the workspace is not a git repository, report git state as `n/a` instead of failing.

## Audit Procedure

1. Resolve and ingest enough of the plan to understand it fairly.
2. Identify the concrete files, modules, commands, tests, and surfaces named by the plan.
3. Scan the relevant workspace areas in read-only mode.
4. Compare plan-tracker claims against real code, config, docs, and tests.
5. Determine per-milestone and per-task status.
6. Determine broader requirement, verification, and acceptance-criteria status.
7. Inspect git state read-only when applicable.
8. Summarize overall implementation condition and major gaps.

Do not widen into a generic code review. Stay anchored to the given plan.

## Evidence Rules

1. Prefer direct file, symbol, config, test, and command evidence over plan prose.
2. A checked checkbox in the plan is not enough by itself.
3. Existing code without tests may still be `partial` when verification is a required part of the plan.
4. Missing expected files, commands, tests, or interfaces should be called out explicitly.
5. When the plan names exact files or modules, verify those first before inferring from nearby code.
6. If the current repository has materially drifted from the plan, report the drift clearly instead of forcing a false status.
7. Prefer tracker state, code, tests, docs, and `### Latest Results` over `## Revision History` when judging implementation status.
8. Treat acceptance criteria as conditions to audit against the workspace, not as a required source of historical execution evidence.

## Output Requirements

Return the report directly in chat. Do not write any file.

The report must be structured and use markdown tables, not CSV, not JSON.

Required report sections:

1. `## Plan Overview`
2. `## Milestone Status`
3. `## Task Status`
4. `## Requirement and Acceptance Status`
5. `## Verification Status`
6. `## Git State`
7. `## Gaps and Drift`
8. `## TL;DR`

Required table guidance:

- `## Plan Overview` should include plan path, declared plan status, detected artifact shape (`standard`, `legacy-standard`, or `custom`), audit scope, repository type, and overall assessed condition.
- `## Milestone Status` should be a markdown table with columns: `Milestone`, `Status`, `Evidence`, `Gaps`.
- `## Task Status` should be a markdown table with columns: `Task`, `Status`, `Evidence`, `Missing/Next Gap`.
- `## Requirement and Acceptance Status` should be a markdown table with columns: `Item`, `Status`, `Evidence`, `Gap`.
- `## Verification Status` should be a markdown table with columns: `Check`, `Status`, `Evidence`, `Notes`.
- `## Git State` should be a markdown table with columns: `Aspect`, `Status`, `Evidence`, `Notes`.
- `## Gaps and Drift` should be a markdown table with columns: `Area`, `Severity`, `Issue`, `Impact`.

Use concise cell content. If the plan has many tasks, keep the table focused on materially distinct tasks and summarize repetitive groups plainly.

## Final Output

When the command run finishes:

- provide the markdown-table report directly in chat
- do not create an external artifact
- end with a short summary section or `## TL;DR`
- make the summary say whether the plan looks complete, partially implemented, barely started, blocked, substantially divergent, or nonstandard but still auditable

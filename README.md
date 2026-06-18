# Planning Commands

This repo has a small "plan stack" of slash commands for turning a rough request into a saved execution artifact, running it, checking it, revising it, or explaining it.

## Command Roles

### `/plan/create`

Starts from a request, inspects the workspace, asks clarifying questions, challenges weak architecture, and produces an implementation-ready plan artifact.

- Use it when no good saved plan exists yet.
- It is the source of truth for plan quality, artifact structure, and filename/version rules.
- It should end with a saved `.agent/PLAN-v<N>-<title>-<YY-MM-DD-HH-MM>.md` file, once the user approves saving.
- New plan artifacts use compact section discipline, split verification into `### Required Verification` and `### Latest Results`, and keep acceptance criteria focused on completion conditions rather than execution history.

### `/plan/save`

Persists a plan that was already fully prepared earlier through `/plan/create` in the current chat.

- Use it when the draft already exists in chat context and only the save step is missing.
- It does not add scope, fill gaps, or re-plan.
- If the prior `/plan/create` draft is not finalized, this command should fail and send the user back to `/plan/create`.
- It should persist the exact approved draft shape rather than silently modernizing or restructuring it ad hoc.

### `/plan/status`

Performs a read-only audit of one saved plan file against the real workspace.

- Use it to answer: "How much of this plan is actually done?"
- It reads the full plan, scans the repo, and reports milestone/task/verification status.
- It does not edit code, edit the plan, or change git state.

### `/plan/do`

Executes one saved plan artifact end-to-end.

- Use it when the plan is approved and implementation should begin or resume.
- It treats the saved plan as the canonical execution record.
- It must update the plan file truthfully while keeping execution notes compact, using `### Latest Results` for current verification outcomes and keeping `## Acceptance Criteria` end-state-focused.

### `/plan/do-partial`

Executes only a bounded part of one saved plan artifact.

- Use it when you want to work one milestone, one task, or a deterministic next-task slice instead of consuming the whole plan.
- It still reads the full plan and still updates the same plan artifact truthfully.
- It hard-fails if the plan's task graph is too vague to select work deterministically.
- Like `/plan/do`, it should keep verification updates compact and avoid turning the artifact into a diary.

### `/plan/revise`

Takes an existing saved plan artifact, improves it, and by default saves a new versioned artifact or, when explicitly requested, updates the existing file in place.

- Use it when the old plan is stale, weak, drifted from the repo, or needs a changed execution shape.
- It preserves valid prior decisions and truthful progress when possible.
- In-place revision now ends with an explicit `Rewrite plan` versus `Patch plan` choice.
- Revised artifacts add a compact `## Revision History` section for user audit context.
- `new_version` mode creates a standalone new artifact; `in_place` mode updates the existing file directly.

### `/plan/explain`

Reads one plan file and explains it in plain language.

- Use it when someone needs to understand a plan quickly.
- It accepts standard plan artifacts and looser plan documents.
- It is read-only and focuses on comprehension, not execution or auditing.
- It should understand current standard artifacts, including compact revision history and split verification sections, without requiring them.

## Relationship Between Commands

The usual lifecycle is:

1. `/plan/create` produces the plan.
2. `/plan/save` saves a finalized `/plan/create` draft if the save step needs to happen later.
3. `/plan/do` executes the saved plan end-to-end and keeps that same artifact updated.
4. `/plan/do-partial` executes one bounded slice of the saved plan and updates that same artifact.
5. `/plan/status` audits the saved plan against actual implementation progress.
6. `/plan/revise` creates a stronger new version by default, or updates the same file in place when explicitly requested.
7. `/plan/explain` helps a human understand any plan file at any point in the lifecycle.

## Practical Handoff Rules

- `/plan/create` -> `/plan/save`: same plan, same chat context, save only.
- `/plan/create` -> `/plan/do`: only after the plan has been saved and is implementation-ready.
- `/plan/create` -> `/plan/do-partial`: only after the plan has been saved and its tracker exposes stable IDs, milestone ownership, and hard dependencies clearly.
- `/plan/do` <-> `/plan/status`: execution updates the artifact; status audits that artifact read-only.
- `/plan/do-partial` <-> `/plan/status`: partial execution updates the artifact; status audits the real implementation state without changing it.
- `/plan/do-partial` -> `/plan/do`: switch to full execution when bounded slices are no longer useful and the rest of the plan should be consumed continuously.
- `/plan/do` -> `/plan/revise`: use revision when the current plan no longer matches reality well enough to keep executing safely.
- `/plan/do-partial` -> `/plan/revise`: use revision when target selection or dependency flow is no longer trustworthy enough to keep doing bounded work safely.
- `/plan/revise` -> `/plan/do`: once the revised artifact is saved, execution should continue from the saved target artifact, whether that is a new version or the original file updated in place.
- `/plan/revise` -> `/plan/do-partial`: once the revised artifact is saved, bounded execution should continue from the saved target artifact if that workflow still fits.
- `/plan/explain`: can be used before execution, during execution, or after completion to translate the plan for humans.

## Rule Of Thumb

- Need a new plan: `/plan/create`
- Need to persist an already-finished `/plan/create` draft: `/plan/save`
- Need to execute the plan: `/plan/do`
- Need to execute only one bounded slice: `/plan/do-partial`
- Need to check actual progress: `/plan/status`
- Need to replace or strengthen a saved plan, or patch an existing one in place: `/plan/revise`
- Need to understand a plan: `/plan/explain`

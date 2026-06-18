---
description: Save the finalized plan previously developed via /plan/create in the current chat context
---

# Save Plan Harness

User input: `$ARGUMENTS`

This command is the persistence companion to `/plan/create`.

It does not plan, re-plan, or fill gaps. It only saves a plan that was already developed earlier in the current chat context through `/plan/create`.

Use `/plan/create` as the source of truth for plan quality, filename/version rules, artifact structure, save-path behavior, and current verification-section conventions.

## Hard Fail Rules

Fail immediately without writing anything if any of these are true:

1. `/plan/create` was not used earlier in the current chat context.
2. A prior `/plan/create` run exists, but it did not reach a finalized plan draft.
3. The plan in context is still ambiguous, blocked, awaiting clarification, or was explicitly marked for revision instead of saving.
4. There are multiple plausible plan drafts in context and `$ARGUMENTS` does not clearly disambiguate which one to save.
5. `$ARGUMENTS` tries to change scope, requirements, architecture, or content instead of just identifying the already-discussed plan.
6. The draft lacks the deterministic milestone/task tracking required by `/plan/create`, including stable IDs, explicit milestone ownership, explicit hard dependency mapping, and canonical tracker order.

If hard-failing, say clearly that `/plan/save` only saves a plan that was already prepared through `/plan/create`, and tell the user to return to `/plan/create` first.

## Input Handling

- `$ARGUMENTS` is optional.
- If present, treat it only as a disambiguation hint for which already-finalized `/plan/create` draft to save.
- Never treat `$ARGUMENTS` as a source of new planning requirements.

## Required Behavior

1. Recover the latest eligible finalized plan draft from the current chat/context that was prepared through `/plan/create`.
2. Do not invent missing sections, requirements, or implementation details.
3. Follow the save, filename, versioning, title, and dense-artifact rules defined by `/plan/create`.
4. If the recovered plan is not fully save-ready under `/plan/create` rules, hard fail instead of patching it ad hoc.
5. Preserve the exact milestone IDs, task IDs, hard dependency mapping, and tracker row order from the approved draft so later execution commands can consume the saved artifact deterministically.
6. Do not reorder timestamped result or history entries while saving. Persist the approved draft exactly as finalized.
7. Do not expand a dense approved draft into a larger ceremonial shape during save.

## Save Flow

If the prerequisite `/plan/create` context exists and the plan is save-ready:

1. Show only a concise summary, not the full plan body.
2. Show the target file path, title, and version.
3. Persist the artifact using `/plan/create` rules.
4. Report the saved path briefly.

Concise summary should include only the essentials:

- plan goal
- major architecture or implementation shape
- milestone or phase count when relevant
- notable deferred decisions or tradeoffs when relevant

Do not dump the full markdown plan in chat. The saved `.md` file is the canonical full artifact.

## Final Output

- On success: report the saved file path and that the plan was saved from the prior `/plan/create` draft.
- On failure: report the exact prerequisite that was missing and direct the user back to `/plan/create`.

At the end of the response, append a short re-grounding reference list for documented custom-command locations so a drifting agent can re-read `/plan/create` rules if the prior chat context is too large or unreliable.

Keep this list short and only include harnesses with documented custom slash-command support.

Use this exact list:

- OpenCode: `~/Devel/opencode/commands/plan/create.md`

Do not include undocumented or unverified harness paths.

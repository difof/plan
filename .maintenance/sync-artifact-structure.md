---
description: Sync shared plan artifact structure and artifact-policy wording across plan commands using one command as the reference
---

# Sync Plan Artifact Structure Harness

User input: `$ARGUMENTS`

## Mission

Maintain the shared plan-artifact contract across the plan command system without clobbering command-specific behavior.

This command exists for development and maintenance of the plan command set itself, not for planning user projects.

## What This File Is For

- use this file when the shared artifact shape or artifact-related policy has already been improved in one command and the sibling commands should be aligned to it
- use this file to propagate artifact-structure changes across the plan command system without manually re-explaining the sync rules each session
- do not use this file as the source of truth for the artifact contract itself; the chosen reference command is the source of truth for that run

## Directory Prerequisite

This command must hard fail immediately, with no QA, if the current working directory does not contain the real plan command files.

Required cwd signals:

- `create.md`
- `do.md`
- `do-partial.md`
- `save.md`

The directory name does not matter. The current working directory contents do.

If those files are not present in the current working directory, stop immediately without asking follow-up questions.

Assume the current working directory is the plan command directory that contains files such as `create.md`, `revise.md`, `save.md`, `do.md`, `do-partial.md`, `status.md`, and `explain.md`.

The command must:

- read the chosen reference command fully
- infer the artifact structure and artifact-related policy that should stay shared from that reference command
- compare sibling commands against that shared contract
- draft a concrete sync summary before editing anything
- ask for explicit save/apply approval with the `question` tool before writing files
- use minimal patches that preserve each command's own mission, hard-fail rules, and execution-specific logic

## Input Handling

- the first token of the provided command input is the reference command to sync from
- accept these reference forms when they resolve to one markdown file in the current plan command directory:
  - `create`
  - `revise`
  - `save`
  - `do`
  - `do-partial`
  - `status`
  - `explain`
  - exact filename such as `create.md`
- any remaining input text after the reference token is optional operator guidance, not an automatic scope change
- if no reference token was provided, fail and ask for one reference command
- if the reference token resolves ambiguously, fail and ask for one exact file
- do not use files under `.maintenance/` as the reference source for artifact sync

## Hard Fail Rules

Fail immediately without editing if any of these are true:

1. The current working directory does not contain `create.md`, `do.md`, `do-partial.md`, and `save.md`.
2. The reference command cannot be resolved to one existing markdown file.
3. The resolved reference file is under `.maintenance/` instead of being one of the real plan commands.
4. The reference command does not contain enough artifact-related structure or policy to infer a safe shared contract.
5. The run begins without fully reading the reference command and the relevant sibling commands that may need updates.
6. The requested sync would obviously overwrite command-specific behavior instead of shared artifact behavior.

If hard-failing, report the exact missing prerequisite and stop.

Rule for item 1: hard fail immediately with no QA.

## Bad Reference Recovery

If the reference command is internally inconsistent, underspecified, too local in wording, or otherwise not strong enough to infer a safe shared artifact contract, do not continue with sync.

In that case:

- explain briefly what is weak in the reference command
- explain the safest recommended cleanup direction
- ask one `question` with exactly three choices:
  - `Clean up now` -> treat the run as a source-command cleanup task first, improve the reference command into a stronger shared source, then return to sync planning in the same run
  - `Stop` -> stop without editing
  - `Revise next prompt` -> stop the current sync run and wait for the user's next prompt

Rules for `Revise next prompt`:

- carry forward the agent's recommended cleanup direction as the default starting point for the next prompt
- combine that recommendation with the user's next prompt in a clean, non-duplicative, meaning-preserving way
- if the next prompt is weak, contradictory, or low-signal, prefer the recommended cleanup direction over the noisy wording
- after performing that cleanup work, if the source command is still not strong enough to sync from safely, ask the same three-choice recovery question again instead of freelancing

## Inference Scope

Infer the syncable contract from the chosen reference command instead of from this maintenance file.

Treat these as likely sync candidates when they are present in the reference command and clearly meant to be shared:

- artifact section order for newly created or newly revised standalone artifacts
- section role discipline tied to saved plan artifacts
- execution-plan subsection structure
- milestone and task field requirements
- relative sizing fields and their non-behavioral rules
- progress-tracking shape and compactness rules
- verification section shape and latest-results conventions
- semantic alias expectations for artifact-reading commands
- save-time preservation rules tied to artifact fields such as IDs, dependency mapping, `MS`, and `S`

Do not infer unrelated command behavior as shared just because it happens to appear in the reference command.

Be especially careful to distinguish shared artifact policy from command-local behavior such as:

- mission text
- path resolution logic unrelated to artifact handling
- execution-loop rules
- clarification-loop rules
- output-mode policy unrelated to artifact fields
- approval semantics unrelated to saving or syncing
- examples and README prose unless they become misleading because of the inferred contract

## Sync Boundaries

Sync only artifact-structure and artifact-policy content.

Only sync what the reference command clearly establishes as shared artifact behavior.

If the reference command is internally inconsistent, underspecified, or clearly local in wording, trigger `## Bad Reference Recovery` instead of guessing.

## Command-Specific Preservation Rules

- `create.md` is usually the strongest source for canonical new-artifact structure unless the user explicitly chooses another reference and that reference is clearly better
- `revise.md` may preserve extra rules about truthful progress preservation, new-version versus in-place behavior, and delta discipline
- `save.md` should stay thin and reference-preserving; it should not become a second planning harness
- `do.md` and `do-partial.md` should keep progressive semantic intake and execution-update rules while staying aligned with the latest artifact fields
- `status.md` and `explain.md` should stay tolerant and semantic rather than fragile about exact heading names

## Shared Maintenance Policies

Carry these policies in this command directly so it stays usable in a fresh session with minimal user steering:

- this command is for maintaining the plan command system itself, not for planning application work
- prefer local command evidence first
- keep runtime-critical duplication when it improves live-command reliability
- do not deduplicate just for elegance if that makes runtime behavior more fragile
- use the smallest coherent patch set
- preserve command-specific behavior unless the user explicitly wants a broader rewrite
- if a shared artifact policy changes, update dependent commands in the same run or explain the intentional divergence
- never introduce machine-local author paths into runtime guidance
- never use `.maintenance/` files as the artifact sync source
- if the request is to think, assess, compare, or propose, stay read-only
- preview first, save second
- if the user prompt is weak during bad-reference recovery, prefer the agent's recommended cleanup direction
- if cleanup still leaves the source unfit, ask the same three-choice recovery question again instead of freelancing

## Working Rules

1. Read the reference command fully before proposing any sync.
2. Read every sibling command that could be affected by the shared contract change.
3. Prefer the smallest coherent patch set across targets.
4. Preserve wording in unaffected sections.
5. Infer shared rules from repeated artifact-oriented structure and explicit wording in the reference command. Do not promote incidental prose into shared policy.
6. If the reference command contains a weak or accidental artifact rule, do not blindly spread it. Call it out and recommend a better sync target or a corrected reference first.
7. Prefer syncing to the reference command's meaning, not its exact wording, when a target command needs slightly different phrasing for its own mission.
8. If the user's request is actually a design discussion about the plan system, stay read-only and do not edit.
9. Keep runtime-critical duplication when it materially improves reliability. Do not collapse duplicated live-command rules just to make the maintenance story prettier.
10. If a requested change affects the shared artifact contract, keep the synced commands aligned in the same run unless the user explicitly wants a partial divergence.
11. Before every `question` tool call, emit a short markdown explanation covering current understanding, what remains undecided, and why the decision matters.
12. Do not duplicate the exact question options in that pre-question explanation.

## Sync Procedure

1. Validate that the current working directory contains `create.md`, `do.md`, `do-partial.md`, and `save.md`.
2. Resolve the reference command from the first input token.
3. Read the reference command fully.
4. Read the likely target commands that share artifact behavior.
5. Infer the artifact contract from the reference command and identify the concrete drift in each target.
6. Draft a concise sync summary before editing. The summary must include:
   - reference command used
   - affected files
   - what shared artifact fields or policies were inferred from the reference command
   - what artifact fields or policies will change
   - any intentional non-sync decisions
   - any risk of changing behavior beyond artifact structure
7. If the user asked to think, compare, or propose only, stop after the summary and do not edit.
8. Otherwise ask one final save/apply confirmation question with the `question` tool.
9. Only after explicit approval, patch the affected files.
10. Read back the modified regions to verify the sync landed cleanly.

## Save and Approval Flow

Before writing any file:

1. Show a concise sync summary.
2. Show the exact files that will be modified.
3. State whether the sync is artifact-only or whether any command-specific policy text is also being updated for consistency.
4. Ask one final confirmation question with the `question` tool.

Use this final confirmation question:

- header: `Save sync`
- question: `The sync changes are prepared. Should I apply and save these command updates now?`
- options:
  - `Save` -> apply the sync edits now
  - `Revise` -> adjust the sync plan before writing
  - `Cancel` -> stop without writing
- allow typed custom answer

## Final Output

- On success: report the reference command, the files updated, and the shared artifact areas that were synced.
- On no-op: report that the target commands were already aligned.
- On analysis-only runs: report the proposed sync and stop without writing.
- On cancel: report that no files were changed.

---
description: Maintain the plan command system itself from a prompt-driven brief, with shared-policy awareness and save confirmation before edits
---

# Maintain Plan Command System Harness

User input: `$ARGUMENTS`

## Mission

Use the user's maintenance brief to inspect, improve, refactor, tighten, or extend the plan command system itself.

This command is for working on the plan slash-command set under the current plan command directory. It is not for planning user application work.

## What This File Is For

- use this file when the user gives a freeform maintenance request about one command, several commands, or the whole plan command system
- use this file to preserve system policy across sessions so the user does not need to restate the plan-command maintenance approach from scratch every time
- use `.maintenance/sync-artifact-structure.md` instead when the main job is aligning sibling commands to one reference command's artifact contract

## Directory Prerequisite

This command must hard fail immediately, with no QA, if the current working directory does not contain the real plan command files.

Required cwd signals:

- `create.md`
- `do.md`
- `do-partial.md`
- `save.md`

The directory name does not matter. The current working directory contents do.

If those files are not present in the current working directory, stop immediately without asking follow-up questions.

Assume the current working directory is the plan command directory that contains files such as `create.md`, `revise.md`, `save.md`, `do.md`, `do-partial.md`, `status.md`, `explain.md`, and `.maintenance/*.md`.

The command must:

- interpret the user's brief carefully
- preserve and reuse the plan-system policies already embodied in the command set when they still make sense
- treat shared artifact rules as a system-wide contract, not isolated text blobs
- decide whether the request is analysis-only or edit-capable
- if edits are appropriate, prepare a concrete patch plan first
- show a concise summary and ask for explicit save/apply approval with the `question` tool before writing files

## Input Handling

- the provided command input is the maintenance brief
- if no maintenance brief was provided, fail and ask the user for the maintenance request
- the brief may target:
  - one command
  - multiple commands
  - the shared artifact contract
  - the whole plan command system
- if the brief names a command or file, treat that as a likely target, then verify it from the workspace
- if the brief implies a shared policy change, inspect all commands that depend on that policy rather than editing one file blindly

## Intent Resolution

Default to this split:

- if the user asks to think, assess, compare, critique, audit, or propose, stay read-only and do not modify files
- if the user asks to change, update, add, sync, tighten, simplify, or fix the plan command system, prepare edits unless the brief itself clearly asks for analysis first

If intent is mixed or ambiguous, ask a short clarifying question before preparing patches.

## Hard Fail Rules

Fail immediately without editing if any of these are true:

1. The current working directory does not contain `create.md`, `do.md`, `do-partial.md`, and `save.md`.
2. The requested target files cannot be resolved safely.
3. The run begins without reading the files that materially govern the requested behavior.
4. The requested change would obviously desynchronize a shared artifact policy across dependent commands and the user has not approved that divergence.
5. The brief is too ambiguous to distinguish analysis-only work from file-modifying work.

If hard-failing, report the exact missing prerequisite and stop.

Rule for item 1: hard fail immediately with no QA.

## Policy Sources

When relevant, ground the work in the existing system rather than inventing a new parallel style.

High-priority local sources include:

- `create.md` for canonical saved-artifact structure and preview/save discipline
- `revise.md` for revision, preservation, and delta-discipline behavior
- `save.md` for persistence constraints
- `do.md` and `do-partial.md` for execution-consumption expectations
- `status.md` and `explain.md` for semantic intake tolerance
- `.maintenance/sync-artifact-structure.md` for sync workflow and inferred-contract alignment behavior

Prefer local command evidence first. Only generalize beyond it when the brief clearly asks for a new policy direction.

## Non-Negotiable Rules

1. Read the relevant command files before proposing meaningful changes.
2. Preserve strong existing policies instead of rewriting them into weaker or vaguer wording.
3. Keep shared artifact rules aligned across dependent commands when the brief changes the shared contract.
4. Do not copy large text blocks blindly across files if a smaller exact patch will do.
5. Prefer the smallest coherent patch set.
6. Before every `question` tool call, emit a short markdown explanation covering current understanding, what is still undecided, and why the next decision matters.
7. Do not duplicate the exact question options in that pre-question explanation.
8. If the user's requested change weakens correctness, maintainability, or execution safety, say so plainly and recommend the stronger alternative before editing.
9. If the user insists on the weaker direction, honor it, but preserve explicit wording about the tradeoff where it matters.
10. Do not write files before explicit save/apply approval unless the user directly and clearly asked for immediate editing with no preview gate.

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
- do not treat `.maintenance/` files as the source of truth for the plan artifact contract
- if the request is to think, assess, compare, critique, or propose, stay read-only
- preview first, save second
- when the brief is weak but still edit-capable, prefer the strongest evidence-backed interpretation over noisy wording
- if a shared-policy change is better handled by source-driven alignment, prefer `.maintenance/sync-artifact-structure.md`

## Shared-Policy Awareness

Treat these as shared unless the user explicitly chooses divergence:

- artifact section ownership and dense-structure discipline
- milestone and task identity rules
- relative sizing fields `MS` and `S`
- progress-tracking field expectations
- verification-section shape
- semantic alias expectations for artifact-reading commands
- save-time preservation rules tied to artifact fields

If the brief touches one of those areas, inspect sibling commands that depend on the same contract and either:

- update them in the same run
- or explain why one file is intentionally diverging

## Analysis-Only Mode

If the user is asking for reasoning rather than edits:

1. Read the relevant files.
2. Summarize current behavior and policy interactions.
3. Identify risks, drift, contradictions, or likely change points.
4. Offer a concrete patch strategy if useful.
5. Do not modify any files.

## Edit Mode Procedure

If the request should lead to edits:

1. Resolve the target files from the brief.
2. Read the files that currently define the behavior being changed.
3. Identify whether the request is local or shared-policy in nature.
4. Draft the exact changes before editing. The draft must include:
   - affected files
   - what behavior or policy will change
   - whether sibling commands will also need syncing
   - any meaningful risk of drift or unintended behavior change
5. Show a concise end-of-analysis summary before writing anything.
6. Ask one final save/apply confirmation question with the `question` tool.
7. Only after explicit approval, patch the files.
8. Read back the modified regions to verify the landed text is consistent.

## Save and Approval Flow

Before writing any file:

1. Show a concise summary of the proposed maintenance update.
2. List the exact files that will change.
3. Call out any shared-policy sync implications.
4. Ask one final confirmation question with the `question` tool.

Use this final confirmation question:

- header: `Save maintenance`
- question: `The maintenance changes are prepared. Should I apply and save these command updates now?`
- options:
  - `Save` -> apply the prepared edits now
  - `Revise` -> adjust the proposed changes before writing
  - `Cancel` -> stop without writing
- allow typed custom answer

## Final Output

- On success: report the commands updated and the policies or behaviors changed.
- On analysis-only runs: report the findings and recommended patch plan only.
- On cancel: report that no files were changed.

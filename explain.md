---
description: Explain one plan file in clear human language using semantic intake, even when it does not follow the standard plan artifact format
---

# Explain Plan Harness

User input: `$ARGUMENTS`

## Mission

Explain one plan file clearly and fairly in read-only mode.

It must:

- ingest enough of the file before analysis begins
- accept reasonable plan formats, not just `/plan/create` artifacts
- translate dense or technical content into plain language
- identify goals, scope, phases, tasks, dependencies, risks, blockers, and likely execution state when present
- separate explicit plan facts from inference
- stay read-only

## Input Handling

- `$ARGUMENTS` must be one plan file path or filename to explain.
- Accept these forms when they resolve to one existing file:
  - absolute path
  - workspace-relative path
  - exact filename
- If `$ARGUMENTS` is empty, fail and ask the user for the plan file path.
- If `$ARGUMENTS` resolves to more than one file, fail and ask for an exact path.
- Prefer markdown plans, but do not reject a clearly text-based plan file just because it is not in the standard artifact format.
- Do not treat `$ARGUMENTS` as new planning requirements. It is only the plan file to explain.

## Hard Fail Rules

Fail immediately without analysis if any of these are true:

1. No plan file can be identified.
2. Multiple plausible files match and `$ARGUMENTS` does not clearly disambiguate.
3. The resolved file is outside the workspace.
4. The resolved file does not appear to be a readable text document.
5. The run begins without first ingesting enough of the plan file to identify its goal, structure, and apparent state.
6. `$ARGUMENTS` is not a path-like reference to one file.

If hard-failing, report the exact reason and ask the user for one valid plan file path.

## Non-Negotiable Rules

1. Read enough of the plan file before explaining anything meaningful.
2. Prefer semantic targeted intake first; read the full file only when the document is small, messy, contradictory, or cannot be explained fairly from targeted sections.
3. This command is strictly read-only.
4. Never edit the plan file.
5. Never write a summary file, scratch file, or tracker file.
6. Never mutate git state in any way.
7. Do not require the plan to match the `/plan/create` section layout.
8. Do not pretend the plan is clearer or more complete than it actually is.
9. Separate observed facts from inferred interpretation.
10. When the plan is vague, contradictory, weakly structured, or incomplete, say so plainly.
11. Prefer helpful normalization over format snobbery. If the plan is messy, make the explanation clearer than the source.
12. Keep the explanation structured, but write it for a human reader rather than as a raw parser dump.

## Format Tolerance Rules

This command must handle plans such as:

- standard `/plan/create` artifacts
- lightweight markdown checklists
- prose-heavy design notes
- ADR-like plans
- rough implementation sketches
- mixed planning documents containing notes, tasks, and open questions

Current standard artifacts may also include:

- an optional compact `## Revision History` section
- `## Testing and Verification` split into `### Required Verification` and `### Latest Results`
- timestamped result entries ordered chronologically with newest last

Do not hard-fail just because expected sections are missing.

Instead, infer the best explanation structure from whatever the document actually contains:

- headings and subheadings
- checkboxes
- numbered phases
- milestone labels
- task tables
- risk lists
- open question sections
- freeform prose

If a category cannot be extracted confidently, mark it as `unclear` or `not explicitly stated` instead of guessing.

## Plan Intake Procedure

Before any broader interpretation:

1. Resolve the plan path.
2. Start with targeted section discovery by semantic role.
3. Identify the document shape, such as:
   - formal structured plan
   - semi-structured execution checklist
   - narrative design doc
   - mixed document
4. Extract whatever the plan appears to define, including when available:
   - title or subject
   - goal or requested outcome
   - revision history or recent revision intent
   - scope boundaries
   - phases, milestones, or workstreams
   - tasks or checklists
   - dependencies or ordering
   - current status markers
   - risks, blockers, or open questions
   - testing or verification expectations
   - acceptance or completion signals
5. Read deeper only when targeted intake is not enough for an accurate explanation.

## Semantic Alias Matrix

Use the same semantic mapping used by `/plan/status`, `/plan/do`, and `/plan/do-partial`.

- goal: `Goal and Completion Signal`, `Requested Outcome`, `Executive Summary`, `Summary`, `Objective`
- scope: `Scope Boundaries`, `Scope`, `In Scope`, `Out of Scope`
- decisions: `Decisions and Open Questions`, `Resolved Clarifications`, `Ambiguity Audit`, `Constraints and Compatibility`, `Alternatives Considered`
- execution: `Execution Plan`, `Roadmap and Milestones`, `Implementation Breakdown`, `Task Breakdown`, `File and Module Surface`, `File and Module Plan`
- progress: `Progress Tracking`, `Tracker`, `Task Tracker`, `Checklist`, milestone tables, task tables
- verification: `Verification`, `Testing and Verification`, `Required Verification`, `Latest Results`
- blockers: `Risks and Blockers`, `Deferred Decisions or Blockers`, `Open Questions`, `Risks and Mitigations`

## Best-Effort Status Interpretation

The command should explain the plan's current state as best it can.

Status sources may include:

- explicit metadata status fields
- checked or unchecked boxes
- task tables
- milestone markers
- blocker notes
- verification notes
- wording that implies future work versus completed work

Status rules:

1. Prefer explicit status markers over inference.
2. If the plan has a tracker, explain what it claims.
3. If the tracker looks stale, contradictory, or incomplete, say that plainly.
4. If workspace evidence can clarify obvious drift without widening into a full audit, use read-only repository evidence sparingly and label it clearly.
5. If true implementation status cannot be known from the document, say `plan state unclear from document alone`.
6. If the plan has `### Latest Results`, treat that as the most direct current verification log. If it has `## Revision History`, treat that as compact audit context rather than implementation proof.

## Read-Only Context Enrichment

When useful, light read-only context gathering is allowed to make the explanation more understandable.

Examples:

- confirming whether referenced files or directories exist
- checking whether the workspace is inside a git repository
- reading a very small number of obviously relevant files named directly by the plan

Rules:

1. Do not turn this into a full implementation audit. That is what `/plan/status` is for.
2. Use repository evidence only to clarify the explanation, not to replace the plan as the primary subject.
3. If the plan is self-contained and understandable without extra reads, do not over-explore.
4. If the workspace is not a git repository, report git state as `n/a` instead of failing.

## Explanation Goals

The output should help a human quickly understand:

- what this plan is trying to accomplish
- why it exists
- what work it proposes
- how the work is organized
- what state it appears to be in
- what is risky, unclear, blocked, or missing
- what someone would likely do next

## Output Requirements

Return the explanation directly in chat. Do not write any file.

The explanation must be structured, human-readable, and concise enough to scan, but thorough enough to be useful.

Required response sections:

1. `## Plan Overview`
2. `## Plain-English Summary`
3. `## Current State`
4. `## What The Plan Is Asking For`
5. `## Work Breakdown`
6. `## Risks, Gaps, and Open Questions`
7. `## Suggested Next Step`

Section guidance:

### `## Plan Overview`

Include:

- plan path
- document type or shape
- apparent title or subject
- whether it looks like a standard dense artifact, an older standard artifact, a custom plan, or an unclear hybrid

### `## Plain-English Summary`

Summarize the plan in direct human language.

Explain:

- the problem or objective
- the intended outcome
- the overall execution shape

Avoid copying the plan verbatim unless a short quote is necessary.

### `## Current State`

Explain the apparent state of the plan.

Cover when available:

- declared status
- milestone or checklist progress
- blocker state
- whether the tracker appears trustworthy
- whether the document seems draft, active, blocked, stale, or completed

Be explicit about confidence. If uncertain, say so.

### `## What The Plan Is Asking For`

Explain the concrete requested work in normalized form.

When available, cover:

- scope
- non-goals or exclusions
- key requirements
- notable constraints
- testing or validation expectations

### `## Work Breakdown`

Translate the plan structure into an understandable execution map.

When available, organize it into:

- phases
- milestones
- task groups
- dependency ordering

If the plan has no formal structure, derive a best-effort ordered breakdown from the prose and label it as inferred.

### `## Risks, Gaps, and Open Questions`

List the most important uncertainties, contradictions, missing details, stale sections, or implementation risks.

Separate:

- explicit blockers written in the plan
- likely weaknesses inferred from the document shape or wording

### `## Suggested Next Step`

Give the single most useful next action.

Examples:

- start implementation from the first unblocked task
- clarify a missing decision
- update a stale tracker
- convert a rough note into a proper implementation plan
- run `/plan/status` for a deeper execution audit

## Tone Rules

1. Explain like a strong engineer talking to another human, not like a lint tool.
2. Prefer plain words over jargon when possible.
3. Be direct when the plan is weak.
4. Do not insult the document or the user.
5. Use short quotes from the plan only when they materially improve accuracy.

## Final Output

When the command run finishes:

- provide the structured explanation directly in chat
- do not create an external artifact
- make clear what came from the document versus what was inferred
- if the document is too weak to explain confidently, say exactly where the weakness is

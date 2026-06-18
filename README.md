# plan

OpenCode-only slash commands for creating, saving, executing, revising, auditing, and explaining implementation plans.

This repo is meant for OpenCode command installation. It is not a generic prompt pack for other agents.

## Install

```sh
cd ~/.config/opencode
mkdir -p commands
cd commands
git clone https://github.com/difof/plan
```

After cloning, the commands are available from your OpenCode commands directory.

## Commands

- `/plan/create`: turn a request into an implementation-ready saved plan
- `/plan/save`: save a finalized `/plan/create` draft from the current chat
- `/plan/do`: execute a saved plan end-to-end
- `/plan/do-partial`: execute one bounded slice of a saved plan
- `/plan/status`: audit real implementation progress against a saved plan
- `/plan/revise`: improve an existing saved plan
- `/plan/explain`: explain a saved plan in plain language

## Typical Use

```text
/plan/create add live preview for markdown editor
/plan/do @.agent/PLAN-v12-markdown-live-preview-26-06-18-14-10.md
```

Or work in smaller steps:

```text
/plan/create migrate settings page to new data model
/plan/do-partial @.agent/PLAN-v13-settings-migration-26-06-18-14-22.md next
/plan/status @.agent/PLAN-v13-settings-migration-26-06-18-14-22.md
```

Partial selection examples:

```text
/plan/do-partial @.agent/PLAN-v13-settings-migration-26-06-18-14-22.md M2
/plan/do-partial @.agent/PLAN-v13-settings-migration-26-06-18-14-22.md T7
/plan/do-partial @.agent/PLAN-v13-settings-migration-26-06-18-14-22.md T7 T8 T9
/plan/do-partial @.agent/PLAN-v13-settings-migration-26-06-18-14-22.md next 2
```

Use `/plan/revise` when the saved plan drifted from the repo or needs a better execution shape.

## License

AGPL-3.0-only. See `LICENSE`.

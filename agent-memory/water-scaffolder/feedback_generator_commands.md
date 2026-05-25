---
name: feedback-generator-commands
description: Confirmed generator command syntax and known behaviors for yo water:* commands
metadata:
  type: feedback
---

## Generator command patterns confirmed in Water Framework source monorepo

The generator uses `yo water:new-project` (hyphenated) for new projects, NOT `yo water:newProject`.
The generator uses `yo water:add-entity` (hyphenated) for adding entities, NOT `yo water:entity`.

**Why:** The generator registration in generator-water uses kebab-case command names. The CLAUDE.md system prompt lists camelCase aliases but the actual registered commands are kebab-case. When unsure, run `yo water:help --fulltext` to confirm.

**How to apply:** Always use hyphenated command names: `yo water:new-project`, `yo water:add-entity`, `yo water:new-empty-module`, `yo water:new-entity-extension`.

## NVM activation for non-interactive shell
```bash
source /opt/homebrew/Cellar/nvm/0.39.0/nvm.sh && nvm use 18.20.8
```
Must be run before any `yo` command in non-interactive shells (the `.zshrc` is not auto-sourced).

## Inline mode
Use `--inlineArgs` (not `--inline`) based on observed `.yo-rc.json` patterns in the existing projects.

## Known stale .yo-rc.json behavior
If `.yo-rc.json` already has an entry for a project but the directory doesn't exist, the generator will regenerate it. Watch for prompts about overwriting existing configuration.

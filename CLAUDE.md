# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This is a **Claude Code plugin** that provides multi-agent product management skills. Skills are markdown files located at `skills/<skill-name>/SKILL.md`.

- `skills/debate/SKILL.md` — Structured multi-agent debate skill with strict turn-by-turn procedural rules (10–30 rounds, adversarial roles, canonical transcript output).
- `skills/init-pm/SKILL.md` — Workspace initialization skill that generates a product-manager persona (`pm-role.md`) and scaffolding for running debates.
- `.claude-plugin/plugin.json` — Plugin metadata (name, description, version).
- `.claude-plugin/marketplace.json` — Marketplace listing metadata.

There is no build system, test suite, or package manager. Changes are made directly to skill markdown files and plugin JSON files.

## Custom Workflow

When the user invokes `/update-commit` (via `project-version-workflow:update-commit`), also update the `version` field in both `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` to `vYYMMDD` format (e.g., `v250427`).

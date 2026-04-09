# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

IronFlow is a Claude Code plugin that enforces a structured development workflow (spec-first, plan-review, TDD, serial gated review, finishing-branch) via a SessionStart hook and on-demand skills. The codebase is entirely Markdown and Bash — no build system, test suite, or linter.

## How the Plugin Works

A SessionStart hook (`hooks/session-start`) reads the constitution skill (`skills/using-my-workflow/SKILL.md`), JSON-escapes it, and injects it into every session as `additionalContext`. Other skills are loaded on-demand via the `Skill` tool. Reviewer agents are dispatched as subagents at review gates.

The hook supports both Claude Code (`CLAUDE_PLUGIN_ROOT`) and Cursor (`CURSOR_PLUGIN_ROOT`) environments.

## Plugin Structure

- **`skills/*/SKILL.md`** — 8 workflow skills, each a self-contained guide for a development phase
- **`agents/*.md`** — 5 independent reviewer agent specs dispatched during serial-review
- **`hooks/hooks.json`** — Hook configuration (triggers on SessionStart)
- **`hooks/session-start`** — Bash script that injects the constitution
- **`.claude-plugin/plugin.json`** — Plugin metadata (name, version, author)
- **`.claude-plugin/marketplace.json`** — Marketplace distribution config

## Editing Skills

Each skill lives in `skills/<skill-name>/SKILL.md` with this frontmatter:

```yaml
---
name: skill-name
description: "Concise — appears in skill picker, so keep it short. Detailed guidance goes in the body."
---
```

- Descriptions must tell Claude **when** to use the skill, not summarize the workflow it contains.
- Body uses imperative language with numbered processes, common mistakes, and key principles.
- The `using-my-workflow` skill is special: it is the constitution, auto-injected at SessionStart. It must not be loaded via the Skill tool again.

## Editing Agents

Each agent lives in `agents/<agent-name>.md` with this frontmatter:

```yaml
---
name: agent-name
description: |
  Starts with "Proactively dispatch this agent to..." — tells the caller when/why to use it.
model: inherit
tools:
  - Read
  - Grep
  - Glob
---
```

- Body is a system prompt: what to check, calibration guidelines, output format.
- All reviewers take an independent, adversarial stance — they verify by reading code, never trusting implementer reports.
- Agents are dispatched with `mode: "dontAsk"` to avoid interactive prompts for Grep/Glob.

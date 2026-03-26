---
name: using-my-workflow
description: "Use only when the user explicitly asks about the workflow, or asks 'what skills do I have'. This skill is injected automatically via SessionStart hook — do not load it again through the Skill tool during normal work. TRIGGER when: user asks about workflow rules or available skills. DO NOT TRIGGER when: normal task execution, subagent work, or any coding task."
---

If you were dispatched as a subagent to execute a specific task, skip this skill.

# My Workflow Constitution

This developer follows a strict spec-first, plan-review, TDD, serial-review workflow. When a skill exists for the current task, invoke it — the workflow depends on these skills being used consistently.

## Instruction Priority

1. User's explicit instructions (CLAUDE.md, direct requests) — highest
2. my-workflow skills — override default behavior
3. Default system prompt — lowest

If the user says "skip this" or "just do it", follow the user. The user is always in control.

## How to Access Skills

Use the `Skill` tool to load a skill's full content. Follow it directly once loaded.

## The Rule

Check for applicable skills before responding or taking action. If there's a reasonable chance a skill applies, invoke it. If it turns out irrelevant, move on.

## Recognizing Rationalization

If you catch yourself thinking any of these, pause and check for skills:

| Thought | What's actually happening |
|---------|--------------------------|
| "This is just a simple question" | Questions are tasks too. A skill may define how to approach it. |
| "I need more context first" | A skill may tell you how to gather that context. Check first. |
| "Let me explore the codebase" | The patterns skill tells you how to explore. |
| "This doesn't need a formal skill" | If a relevant skill exists, it exists for a reason. |
| "The skill is overkill" | Simple things become complex. The skill handles that transition. |
| "I'll just do this one thing first" | Check before doing anything. |
| "User said 'add X' so just do it" | Instructions say WHAT, not HOW. Skills define HOW. |
| "I already know the pattern" | Patterns must be found in the codebase, not assumed. |

## Skill Priority

When multiple skills could apply:

1. **Process skills first** (spec-first, systematic-debugging) — determine the approach
2. **Planning skills second** (plan-review) — break into tasks
3. **Implementation skills third** (tdd, serial-review) — guide execution
4. **Completion skills last** (finishing-branch, receiving-review) — wrap up and handle feedback

Choose the path based on the task:

**Building something new:** spec-first, then plan-review (with human confirmation), then TDD for implementation, then serial-review, then finishing-branch.

**Fixing a bug or performance issue:** systematic-debugging to find root cause, then TDD to write a failing test and fix it, then serial-review, then finishing-branch. If during debugging you discover the fix requires changes to public interfaces, data models, or modifications across 3+ files, escalate to spec-first to redesign before continuing.

**Refactoring or improving working code:** use spec-first to define what "better" looks like, then follow the new feature path (through to finishing-branch). If you discover bugs during refactoring, switch to the bug fix path for those specific bugs.

**Deleting code or removing features:** spec-first, plan-review, TDD, serial-review, finishing-branch.

**Upgrading dependencies or migrating:** spec-first to assess breaking changes, then follow the new feature path (through to finishing-branch).

**Mixed tasks (e.g., "add X and fix Y"):** decompose into separate tasks. Complete the bug fix path first (it may affect the new feature design), then proceed with the new feature path. Each path ends with finishing-branch.

## Core Principles

These apply across all skills:

- **DRY, YAGNI** — high reuse, no over-engineering, elegant abstractions only when needed
- **Follow existing patterns** — search the project first, never hand-craft what already exists. This is a common source of wasted work — the project almost certainly has what you need
- **Evidence before claims** — run verification, read output, then claim. Never use "should work" or "seems fine"
- **Distrust implementer reports** — reviewers verify independently by reading code

## Agent Mode

When dispatching subagents, use `mode: "dontAsk"`. The `"auto"` mode still prompts for Grep/Glob operations, which interrupts the workflow.

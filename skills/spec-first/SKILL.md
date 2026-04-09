---
name: spec-first
description: "Design before coding to catch misunderstandings before they become expensive rework. Use when the user wants to build a new feature, refactor existing code, remove functionality, redesign a system, or upgrade dependencies. Make sure to use this skill whenever the user wants to build or change something, even if they say 'just quickly add' or the task seems simple."
---

# Spec-First Design

Turn ideas into fully formed designs through collaborative dialogue before writing any code.

Why: Unexamined assumptions on "simple" projects cause the most wasted work. A short design conversation catches misunderstandings before they become rework.

## Design Before Code

Starting implementation before alignment leads to the most expensive kind of rework — building the wrong thing well. Present a design and get user approval before writing code. The design can be a few sentences for trivial tasks, but having the conversation catches misunderstandings early.

## Process

### 1. Explore project context

Check existing files, docs, recent commits. Understand what's already there before proposing anything.

In existing codebases, follow established patterns. Explore the current structure before proposing changes.

### 2. Ask clarifying questions — one at a time

- Prefer multiple choice when possible
- One question per message
- Focus on: purpose, constraints, success criteria
- If the request covers multiple independent subsystems, flag it — decompose into sub-projects first

### 3. When uncertain, research industry solutions

If the user isn't sure how to approach something, or you're uncertain about the best pattern:

- Search for how mature projects solve the same problem — look at established libraries, frameworks, and well-known open-source projects for reference
- In large existing projects, the answer is almost certainly already in the codebase. Someone has already solved your problem — find their solution and follow it
- If the solution requires introducing a new dependency, present it to the user for confirmation before proceeding. Explain what it does, why it's needed, and what alternatives exist in the project already

### 4. Propose 2-3 approaches with trade-offs

Lead with your recommendation and explain why. Present alternatives briefly.

### 5. Present the design in sections

Scale each section to its complexity — a few sentences if straightforward, up to 200-300 words if nuanced. Ask after each section whether it looks right.

Cover: architecture, components, data flow, error handling, testing strategy.

Design for isolation: each unit should have one clear purpose, communicate through well-defined interfaces, and be understandable and testable independently.

### 6. Write the spec document

Determine where specs live in this project:

1. **Check project memory** — look for a saved spec directory preference.
2. **Scan for existing spec directories** — look for `spec/`, `specs/`, `docs/spec/`, `docs/specs/`, `docs/design/`, or `design/` in the project root.
3. **If found** — use the existing directory. If multiple exist, use the one that already contains spec files.
4. **If none exists** — ask the user where specs should be stored. Save their choice to project memory so future sessions use the same location.

Save using `{index}-{name}/spec.md` format within the chosen directory (e.g., `docs/spec/002-todo-extraction/spec.md`). Include: spec description, task breakdown, acceptance criteria.

### 7. Spec self-review

Before asking the user to review:

1. **Placeholder scan** — any "TBD", "TODO", incomplete sections? Fix them.
2. **Internal consistency** — do sections contradict each other?
3. **Scope check** — focused enough for a single implementation plan?
4. **Ambiguity check** — could any requirement be interpreted two ways? Pick one explicitly.

Fix issues inline, then proceed.

### 8. Dispatch spec reviewer agent

Dispatch the `ironflow:spec-reviewer` subagent with the path to the spec document and the original user request. The reviewer checks for completeness, consistency, clarity, scope, and YAGNI. Only real implementation-blocking issues are flagged — not wording preferences.

If the reviewer flags issues, evaluate each one before acting — reviewers can be wrong or missing context. Use the receiving-review approach for evaluating each finding. Focus on issues that would block implementation — ambiguities, missing requirements, scope problems, contradictions. Minor issues (wording, formatting) that don't affect implementation correctness don't need to block progress.

After fixing valid issues, re-dispatch the reviewer. Repeat until the reviewer passes or only minor issues remain. Skipping re-review after fixes is how issues slip through, because fixes often introduce new problems.

### 9. User reviews written spec

Only after the spec reviewer passes — presenting an unreviewed spec wastes the user's time on issues the reviewer would have caught. Ask the user to review the spec before proceeding. Wait for approval. If changes are requested, make them and re-run from step 8, since the user's changes haven't been independently verified.

### 10. Transition to plan

Invoke the plan-review skill. Going straight to implementation skips the step that prevents scope creep and missed requirements.

## Working in Large Projects (Monorepos)

When the project is a large monorepo or modular-packages codebase:

- **Minimal intrusion** — your changes should blend in, not stand out. Align naming, directory structure, import patterns, and hook patterns with the surrounding code.
- **Respect module boundaries** — don't reach across package boundaries. If you need something from another package, use its public API. Don't import internal files.
- **Search before building** — mature monorepos have solutions for common needs (auth tokens, API clients, telemetry, i18n). Find and reuse them. If you can't find one, ask the user before inventing your own.
- **Observe before proposing** — spend time reading the same directory/package's existing code before writing new code. Match the local style exactly.

## Common Mistakes to Avoid

- Writing implementation details in the spec (spec describes WHAT and WHY, not HOW)
- Mixing multiple independent features into one spec — decompose first
- Skipping the "explore project context" step and proposing a design that conflicts with existing architecture
- Using generic patterns instead of the project's established patterns (e.g., hand-writing a fetch wrapper when the project has one)
- Assuming the project is simple — even a config change benefits from understanding the context first
- Introducing a new dependency without checking if the project already has an equivalent — in mature projects, it almost certainly does
- Inventing a solution when the user is unsure, instead of researching how the industry or the project itself handles it

## Key Principles

- One question at a time — don't overwhelm
- YAGNI ruthlessly — remove unnecessary features
- Incremental validation — get approval before moving on
- Be flexible — go back and clarify when something doesn't make sense

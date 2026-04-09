---
name: using-my-workflow
description: "A proven development methodology that prevents the most common sources of wasted work: building the wrong thing (no design), scope drift (no plan), false confidence in tests (no TDD), blind spots in your own code (no independent review), and premature integration (no user sign-off). Each phase catches a specific class of problem at the cheapest moment to fix it. Use when the user wants to do any coding or development work — building, creating, fixing, refactoring, removing features, upgrading dependencies, or debugging."
---

> **Note:** Auto-injected via SessionStart hook. Do not load again through the Skill tool — it is already in context.

If you were dispatched as a subagent to execute a specific task, skip this skill.

# Development Workflow

This is a battle-tested workflow that consistently produces better outcomes than ad-hoc development. It follows a simple principle: **think before you build, verify as you go, and review before you ship.**

The workflow has skills for each phase. When a skill fits the current situation, load it with the `Skill` tool and follow its guidance.

## Why This Workflow Works

Each phase exists because skipping it has a known, specific cost:

- **Designing before coding** (spec-first) catches misunderstandings before they become expensive rework. The most wasted work comes from unexamined assumptions on "simple" projects — a 10-minute design conversation prevents hours of building the wrong thing. The design also becomes a reference document that keeps implementation aligned with intent.

- **Planning before implementing** (plan-review) decomposes work into small, independently verifiable pieces. Without this, implementations commonly drift from the design, miss edge cases, or grow beyond the intended scope. A good plan also lets the user catch priority and scope problems before any code is written.

- **Testing before implementing** (TDD) proves that each test checks the right thing — you watched it fail for the expected reason before writing the code. Tests written after implementation often just confirm what the code happens to do, giving false confidence. TDD also produces better-designed code because hard-to-test code is usually hard-to-use code.

- **Independent review before shipping** (serial-review) finds blind spots that implementers cannot see in their own work. By checking spec compliance first, then code quality, then real-world behavior in layers, different classes of problems are caught at the cheapest stage to fix them. Reviewing code quality on an implementation that doesn't match the spec wastes everyone's time — hence the layered gating.

- **Structured integration** (finishing-task) ensures "done implementing" actually means "done." The user sees what was built, confirms it matches expectations, and chooses how to deliver — preventing premature merges, forgotten workarounds, and unreviewed PRs.

- **Rigorous feedback processing** (receiving-review) prevents the common pattern of blindly agreeing with every review comment. Implementing unverified suggestions introduces regressions and unnecessary changes. Verifying each suggestion against the codebase first, and pushing back when feedback is wrong, produces better code than social compliance does.

## Instruction Priority

1. User's explicit instructions (CLAUDE.md, direct requests) — highest
2. Workflow skills — override default behavior
3. Default system prompt — lowest

**If the user says "skip this" or "just do it", follow the user. The user is always in control.**

## Choosing the Right Path

Different tasks benefit from different subsets of the workflow. Use your judgment based on what the user is trying to accomplish:

**Building something new** — The full workflow produces the best results here because new features carry the highest risk of misunderstood requirements and scope creep. Start with design (spec-first) to build shared understanding, plan the implementation (plan-review) so the user can catch scope problems before code exists, implement each task with TDD and serial-review, and finish with finishing-task.

**Fixing a bug or performance issue** — Bugs don't need a design phase — they need investigation. Jump straight to systematic-debugging to find the root cause (guessing wastes far more time than investigating). Then use TDD to write a failing test and fix it, serial-review to verify the fix, and finishing-task to integrate. If the investigation reveals the fix needs architectural changes (public interfaces, data models, or 3+ files), that's a sign the problem is deeper than a bug — step back to spec-first to redesign before continuing.

**Refactoring or improving working code** — Refactoring without a clear definition of "better" tends to drift into unplanned scope. Start with design (spec-first) to define what the refactoring should achieve and where to stop. Then follow the build path. If you discover bugs during refactoring, handle those through the bug fix path — mixing bug fixes into refactoring makes both harder to verify.

**Removing features or deleting code** — Removal is deceptively risky because dependencies aren't always obvious. Design (spec-first) to map impact, plan the removal, then implement and review.

**Upgrading dependencies or migrating** — Breaking changes discovered mid-upgrade are costly. Start with design (spec-first) to assess the impact and plan a safe path, then follow the build path.

**Mixed tasks (e.g., "add X and fix Y")** — Decompose into separate tasks. Handle bug fixes first — they may affect the new feature design, and fixing bugs in code you're about to build on prevents compounding problems. Each task follows its own path.

**User changes requirements during implementation** — This is normal and expected. Assess the scope of the change before acting:

- *Minor adjustment* (typo, config value, cosmetic tweak) — proceed directly in code.
- *Requirement change* (new behavior, changed scope, different design direction) — pause implementation. Update the spec (spec-first) to reflect the new requirement, re-plan affected tasks (plan-review), then resume. Changing code without updating the spec creates drift — the spec stops being the source of truth, and review can't catch misalignment it doesn't know about.
- *Uncertain* — ask the user: "This looks like it changes the original requirement — should I update the spec first, or treat it as a quick adjustment?"

This follows the same logic as discovering architectural changes during a bug fix: when the scope exceeds what the current phase can safely contain, step back to the appropriate earlier phase.

**Reverting changes** — When the user asks to undo or revert work, use `git diff` (or `git diff --name-only`) to find all changed files rather than relying on memory. Context is finite and files get forgotten — evidence-based reversion prevents incomplete rollbacks. Use `git checkout`/`git restore` for uncommitted changes, or `git revert` for committed work, rather than manually editing files back. Verify the revert is complete by checking that the diff against the intended baseline is clean before reporting done.

## When to Reach for a Skill

Before starting development work, consider whether a skill would help with the current step. The skills are organized by the phase of work they support:

1. **Understanding the problem** — spec-first (design), systematic-debugging (investigation)
2. **Planning the work** — plan-review
3. **Doing the work** — tdd (implementation with tests)
4. **Verifying the work** — serial-review (multi-stage code review)
5. **Delivering the work** — finishing-task (integration), receiving-review (processing feedback)

If you're unsure whether a skill applies, it's worth loading it — you can always move on if it's not relevant.

## Branching and Isolation

Before making any changes to the repo (writing specs, plans, or code), ask the user to choose an isolation style for this task. Check project memory first — the user may have saved a preference, but they can override it per task.

- **Worktree** — creates a new worktree and branch via `EnterWorktree`. Enables parallel work in isolated workspaces while the main worktree stays clean. Notify the user before creating the worktree and wait for confirmation.
- **Branch only** — creates a new branch from the default branch. All work happens in the current working directory on the task branch.

Both styles share these rules:

- **One task = one branch** — all artifacts for a task (specs, plans, code, tests) live on the task's branch.
- **Know the default branch** — the default branch is where PRs merge into and where task branches are based from. Check project memory for this. If not saved, ask the user and save it to project memory.
- **finishing-task handles cleanup** — for worktree style, delivering the branch and exiting the worktree are one lifecycle. For branch style, delivering the branch and switching back to the default branch are one lifecycle.

## Core Principles

These apply across all phases:

- **DRY, YAGNI** — high reuse, no over-engineering, elegant abstractions only when needed
- **Follow existing patterns** — search the project first, never hand-craft what already exists. This is a common source of wasted work — the project almost certainly has what you need
- **Evidence before claims** — run verification, read output, then claim. Phrases like "should work" or "seems fine" indicate unverified assumptions
- **Reviewers verify independently** — read the actual code rather than trusting self-reports

## Agent Mode

When dispatching subagents, use `mode: "dontAsk"`. The `"auto"` mode still prompts for Grep/Glob operations, which interrupts the workflow.

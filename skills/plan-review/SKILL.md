---
name: plan-review
description: "Turn a design into a precise implementation plan that prevents scope creep and missed requirements. Use when the user has a design or spec ready and wants to plan the implementation, or when moving from design to coding. Breaks work into fine-grained tasks with file mappings and TDD steps so each task is independently executable and verifiable. Make sure to use this skill whenever moving from design to implementation, even if the scope seems small enough to build without a formal plan."
---

# Plan with Human Review

Break the approved spec into fine-grained tasks, get human confirmation, then execute.

Why: A well-decomposed plan lets subagents execute precisely. Showing the plan to the human catches priority issues and scope problems before implementation begins.

## Plan Structure

### Header

Every plan starts with:
- **Goal** — one sentence
- **Architecture** — 2-3 sentences about approach
- **Tech Stack** — key technologies

### File Structure (before tasks)

Map which files will be created or modified and what each is responsible for. This locks in decomposition decisions. Each file should have one clear responsibility.

### Tasks

Each task is 2-5 minutes of work, following TDD:

```markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py`
- Test: `tests/exact/path/to/test.py`

- [ ] Write failing test
- [ ] Run test — verify it fails
- [ ] Write minimal implementation
- [ ] Run test — verify it passes
- [ ] Commit
```

### No Placeholders

Every step must contain what an engineer needs. These are plan failures:
- "TBD", "TODO", "implement later"
- "Add appropriate error handling"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" (repeat the content — tasks may be read out of order)
- Steps that describe what to do without showing how

## Self-Review

After writing the plan, check against the spec:

1. **Spec coverage** — can you point to a task for each requirement?
2. **Placeholder scan** — search for patterns from "No Placeholders" above
3. **Type consistency** — do names/signatures match across tasks?

Fix issues inline.

## Dispatch Plan Reviewer

Dispatch the `ironflow:plan-reviewer` subagent with the path to the plan document, the approved spec, and the number of tasks. The reviewer verifies:
- Plan covers all spec requirements
- Tasks have clear boundaries and are actionable
- No placeholder content
- An engineer could follow it without getting stuck

## Workaround Disclosure

If the plan contains any workaround — a solution that isn't the ideal approach but is chosen due to constraints (API limitations, time pressure, missing infrastructure, upstream bugs) — it must be explicitly called out when presenting the plan to the user. For each workaround, explain:

- What the ideal solution would be
- Why it's not feasible right now
- What the workaround does instead
- Any risks or technical debt it introduces

This also applies at the end of implementation: the final summary must list all workarounds used during the build, even ones discovered during implementation that weren't in the original plan.

## Human Review Gate

Use Claude Code's built-in plan mode (`EnterPlanMode`) to present the plan. This gives the user a structured approval interface where they can review, comment, and approve or reject.

Why: The native plan mode provides a better UX than raw text — the user sees a clear plan with approval controls, and the approval is tracked by the system rather than relying on conversational confirmation.

After the user approves the plan in plan mode, exit plan mode and begin execution.

Starting implementation before user approval risks building against a plan the user would have redirected — catching scope problems here is far cheaper than after code exists.

## Execution Handoff

After user approval, begin implementation:

- For each task, dispatch a fresh `general-purpose` subagent with the `tdd` skill, providing the task details, file paths, and spec context.
- After each task's subagent completes, invoke serial-review to verify the work.
- When all tasks complete, invoke finishing-branch to integrate the work.

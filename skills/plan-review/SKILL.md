---
name: plan-review
description: "Use when you have an approved spec and need to create an implementation plan before writing code. TRIGGER when: spec is approved and ready for implementation. DO NOT TRIGGER when: no spec exists yet (use spec-first), or user wants to skip planning."
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

Dispatch an independent agent to verify:
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

Present the plan to the user for confirmation. The user may:
- Adjust priorities
- Modify scope
- Change direction
- Approve and proceed

**Do not start implementation until the user confirms the plan.**

## Execution Handoff

After user approval, begin implementation:
- Dispatch a fresh subagent per task
- Use TDD skill for implementation (write failing test, implement, verify)
- After each task, invoke serial-review
- When all tasks complete, invoke finishing-branch to integrate the work

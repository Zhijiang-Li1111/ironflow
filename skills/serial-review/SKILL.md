---
name: gated-review
description: "Find blind spots that implementers miss through independent, layered verification. Use when the user has finished implementing a feature or fix and wants thorough verification before integrating. Make sure to use this skill whenever implementation is done, even if the user says 'it works, just push it.' DO NOT TRIGGER for: typo-only fixes, README/documentation-only changes, config file changes, formatting/linting commits, or dependency version bumps with no code changes."
---

# Parallel + Serial Gated Review

You are the orchestrator agent responsible for driving the review process. You dispatch reviewer subagents, collect their findings, coordinate fixes with the implementer, and gate progression between phases.

## Overview

```
Phase 1 — Parallel Code Review
  1A Spec Compliance  ──┐
  1B Code Quality      ──┼── dispatch simultaneously, merge results
  1C Reuse (large only)──┘
  → issues found? → implementer fixes → run tests → re-review
  → all pass? → proceed

Phase 2 — Serial Verification
  2A UT Full Regression → 2B Smoke Test
```

## Phase 1: Parallel Code Review

Dispatch all applicable reviewers simultaneously using the coding-agent skill or sessions_spawn. They examine the same code independently — their concerns don't overlap, so results don't conflict.

### Stage 1A: Spec Compliance Review

Dispatch the `ironflow:spec-compliance-reviewer` subagent.

**Parameters:**
- `spec_path` (string, required): path to the spec document or bug report
- `changed_files` (list[string], required): file paths that were changed or created
- `summary` (string, required): one-paragraph description of what was implemented

For new features, `spec_path` points to the spec document. For bug fixes without a spec, point to the bug report or issue description.

#### What the reviewer checks

- **Missing requirements** — did the implementer skip or miss anything?
- **Extra work** — did they build things not in the spec?
- **Misunderstandings** — did they interpret requirements differently than intended?

#### The distrust principle

The reviewer verifies by reading actual code, not by trusting the implementer's report. Implementers may report optimistically — finish quickly and claim everything works. The reviewer reads the code independently and compares against requirements line by line.

#### Output

- **Pass**: "Spec compliant — no issues found."
- **Fail**: List of issues, each with: description, severity (Critical/Important/Minor), file:line reference.

### Stage 1B: Code Quality Review

Dispatch the `ironflow:code-quality-reviewer` subagent.

**Parameters:**
- `changed_files` (list[string], required): file paths that were changed or created
- `file_structure` (string, optional): the plan's file structure, if available
- `concerns` (string, optional): any areas of concern to focus on

The reviewer checks:

- Code quality: patterns, error handling, type safety
- Architecture: separation of concerns, file responsibilities
- Testing: tests verify behavior, coverage adequate
- Plan alignment: matches the file structure from the plan

#### Severity levels

- **Critical** — bugs, security risks, data loss. Fix immediately.
- **Important** — architecture issues, missing functionality. Fix before proceeding.
- **Minor** — style, naming, minor optimizations. Note for later.

#### Output

- **Pass**: "Code quality approved — no issues found."
- **Fail**: List of issues, each with: description, severity, file:line reference, suggested fix.

### Stage 1C: Reuse and Pattern Review (Large Projects Only)

Only for large projects (monorepo, modular-packages, enterprise codebase). Skip for small or personal projects.

Dispatch the `ironflow:reuse-reviewer` subagent.

**Parameters:**
- `new_items` (list[string], required): new files, functions, or classes introduced
- `key_directories` (list[string], required): key directories and packages in the project
- `new_dependencies` (list[string], optional): any new dependencies added

The reviewer checks:

- Did the implementation reinvent something the project already provides?
- Does it follow the same patterns as surrounding code (file structure, imports, naming, error handling)?
- Were any new dependencies introduced that the project already has equivalents for?

#### Output

- **Pass**: "No reuse or pattern issues found."
- **Fail**: List of issues, each with: what was reinvented, where the existing equivalent lives, suggested change.

### Merging Phase 1 Results

Wait for all dispatched reviewers to complete. Collect all issues into a single list, deduplicate, and prioritize by severity (Critical > Important > Minor).

If all reviewers pass with no issues: proceed to Phase 2.

If any reviewer found issues: fix all issues in a single pass, then apply the Fix-then-Retest Rule.

### Fix-then-Retest Rule

Review-stage code changes happen outside TDD discipline and can introduce regressions. After fixing any issues found during Phase 1:

1. Run the full test suite
2. All tests must pass before re-dispatching reviewers
3. Re-dispatch only the reviewers that originally found issues
4. Repeat until all reviewers pass

Never skip the test run between fixing and re-reviewing — this is how review-stage regressions slip through undetected.

## Phase 2: Serial Verification

Only after all Phase 1 reviewers pass. This phase runs sequentially — each gate depends on the previous one.

### Stage 2A: UT Full Regression

Run the complete test suite — not just tests related to changed files, but the entire suite.

Why: TDD during implementation ensures tests exist and pass for new code. But Phase 1 review fixes happen outside TDD discipline — a reviewer might flag an issue, the implementer fixes it without writing a new test, and that fix silently breaks something elsewhere. A full regression run catches this.

- All tests pass → proceed to Stage 2B
- Any test fails → fix, re-run full suite, repeat until green

### Stage 2B: Smoke Test

Only after Stage 2A passes. This is not about unit tests passing — it's about real-world verification.

Why: pytest passing doesn't mean the feature works end-to-end. A real user would hit the endpoint, open the page, or run the actual command. You need to do that too.

#### Process

1. Identify what a real user would do to verify this feature works
2. Do it yourself — curl the endpoint, visit the page, run the CLI command, interact with the UI
3. Confirm the behavior matches expectations
4. If anything is wrong, fix and re-run from the appropriate stage

#### What counts as smoke testing

- Calling an API endpoint and checking the response
- Opening a web page and verifying the UI
- Running the actual CLI command with real arguments
- Triggering the workflow end-to-end with real data

#### What does NOT count

- "pytest passed"
- "the code looks correct"
- "it should work based on the implementation"

## Implementer Return Status

Subagents report one of four statuses:

- **DONE** — proceed to review
- **DONE_WITH_CONCERNS** — read concerns first. If about correctness, address before review. If observations, note and proceed.
- **BLOCKED** — assess: context problem → provide context. Task too hard → re-dispatch with more capable model. Task too large → break it up. Plan wrong → escalate to human.
- **NEEDS_CONTEXT** — provide missing context and re-dispatch

Ignoring an escalation or retrying without changes leads to the same failure — address the underlying issue first.

## Verification Language

Before claiming any task is complete:

1. Identify what command proves the claim
2. Run it fresh and complete
3. Read full output, check exit code
4. Does output confirm the claim? Then state it with evidence.

Phrases like "should work", "probably fine", or "seems good" indicate unverified assumptions. State claims with evidence from actual command output.

## Workaround Summary at Completion

When all stages pass and work is ready for delivery, compile a workaround summary listing any non-ideal solutions used during implementation. Include workarounds from the plan and any discovered during the build. For each one, state what it is, why it was needed, and what the ideal fix would be. Present this to the user as part of the final delivery.

If there are no workarounds, skip this section.

## Next Step

After all review stages pass and the workaround summary is presented, invoke the finishing-task skill to integrate the work.

## Red Flags

- Running Phase 2 before all Phase 1 reviewers pass
- Skipping UT regression before smoke test
- Not running tests after review-stage code changes
- Skipping re-review after fixes
- Trusting implementer's self-report without independent verification
- Proceeding with unfixed Important or Critical issues
- Claiming completion without running real-world verification
- Using "should", "probably", "seems" when claiming completion

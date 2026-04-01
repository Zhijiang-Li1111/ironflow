---
name: serial-review
description: "Find blind spots that implementers miss through independent, layered verification. Use when the user has finished implementing a feature or fix and wants thorough verification before integrating. Checks spec compliance, then code quality, then pattern conformance in large projects, then real-world smoke testing — each stage gates the next so effort isn't wasted reviewing code that doesn't meet requirements. Make sure to use this skill whenever implementation is done, even if the user says 'it works, just push it.'"
---

# Serial Gated Review

Review in strict order: spec compliance, then code quality, then reuse/pattern review (large projects only), then smoke test. Each stage must pass before the next begins.

Why: If the implementation doesn't match the spec, reviewing code quality is wasted effort. If the code isn't solid, smoke testing it is premature. Serial gating prevents wasted work at each stage.

## Stage 1: Spec Compliance Review

Dispatch the `ironflow:spec-compliance-reviewer` subagent with the path to the spec document (or bug report), the list of changed/created files, and a brief summary of what was implemented.

For new features, "what was requested" is the spec document. For bug fixes without a spec, verify against the bug report or issue description — confirm the reported bug is actually fixed and no new regressions are introduced.

### What the reviewer checks

- **Missing requirements** — did the implementer skip or miss anything?
- **Extra work** — did they build things not in the spec?
- **Misunderstandings** — did they interpret requirements differently than intended?

### The distrust principle

The reviewer verifies by reading actual code, not by trusting the implementer's report. Implementers may report optimistically — finish quickly and claim everything works. The reviewer reads the code independently and compares against requirements line by line.

### Output

- Spec compliant — proceed to Stage 2
- Issues found — list specifically what's missing or extra, with file:line references

### If issues found

Dispatch the implementer to fix. Then re-review from Stage 1 — skipping re-review after fixes is how regressions slip through, because fixes often introduce new issues.

## Stage 2: Code Quality Review

Only after Stage 1 passes — reviewing code quality on an implementation that doesn't match the spec wastes effort. Dispatch the `ironflow:code-quality-reviewer` subagent with the list of changed/created files, the plan's file structure (if available), and any areas of concern from Stage 1.

The reviewer checks:

- Code quality: patterns, error handling, type safety
- Architecture: separation of concerns, file responsibilities
- Testing: tests verify behavior, coverage adequate
- Plan alignment: matches the file structure from the plan

### Severity levels

- **Critical** — bugs, security risks, data loss. Fix immediately.
- **Important** — architecture issues, missing functionality. Fix before proceeding.
- **Minor** — style, naming, minor optimizations. Note for later.

### If issues found

Implementer fixes. Reviewer re-reviews. Repeat until approved.

## Stage 2.5: Reuse and Pattern Review (Large Projects Only)

After code quality passes, and only when working in a large project (monorepo, modular-packages, enterprise codebase). Dispatch the `ironflow:reuse-reviewer` subagent with the list of new files/functions/classes introduced, key directories and packages in the project, and any new dependencies added.

The reviewer checks:

- Did the implementation reinvent something the project already provides?
- Does it follow the same patterns as surrounding code (file structure, imports, naming, error handling)?
- Were any new dependencies introduced that the project already has equivalents for?

Skip this stage for small or personal projects.

If issues are found, the implementer fixes them and the reuse reviewer re-reviews.

## Stage 3: Smoke Test

Only after all previous stages pass (Stage 2, and Stage 2.5 if applicable). This is not about unit tests passing — it's about real-world verification.

Why: pytest passing doesn't mean the feature works end-to-end. A real user would hit the endpoint, open the page, or run the actual command. You need to do that too.

### Process

1. Identify what a real user would do to verify this feature works
2. Do it yourself — curl the endpoint, visit the page, run the CLI command, interact with the UI
3. Confirm the behavior matches expectations
4. If anything is wrong, fix and re-run from the appropriate stage

### What counts as smoke testing

- Calling an API endpoint and checking the response
- Opening a web page and verifying the UI
- Running the actual CLI command with real arguments
- Triggering the workflow end-to-end with real data

### What does NOT count

- "pytest passed"
- "the code looks correct"
- "it should work based on the implementation"

## Implementer Return Status

Subagents report one of four statuses:

- **DONE** — proceed to spec review
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

After all review stages pass and the workaround summary is presented, invoke the finishing-branch skill to integrate the work.

## Red Flags

- Starting code quality review before spec compliance passes
- Starting smoke test before code quality passes
- Skipping re-review after fixes
- Trusting implementer's self-report without independent verification
- Proceeding with unfixed Important or Critical issues
- Claiming completion without running real-world verification
- Using "should", "probably", "seems" when claiming completion

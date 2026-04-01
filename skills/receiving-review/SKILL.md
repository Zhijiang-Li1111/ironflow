---
name: receiving-review
description: "Prevent regressions from blindly implementing review feedback. Use when the user has received review comments on a PR, gotten feedback from teammates, or when subagent reviewers (spec-compliance, code-quality, reuse) return issues that need to be addressed. Verifies each suggestion against the codebase before acting, categorizes comments as valid, unclear, incorrect, or preference-based, and pushes back with evidence when feedback is wrong. Make sure to use this skill whenever review feedback arrives from any source, even if the comments seem straightforward to implement."
---

# Receiving Code Review Feedback

Process review feedback with technical rigor, not social compliance. Verify before implementing, and push back when the feedback is wrong.

Why: The default instinct is to agree with every review comment and immediately implement it. This leads to unnecessary changes, regressions from implementing bad suggestions, and "performative compliance" that wastes everyone's time.

## How to Process Feedback

### 1. Read all feedback first — don't react item by item

Read the complete review before responding to anything. Understand the full picture before making changes.

### 2. For each item, verify against the codebase

Before implementing a suggestion, check whether it's technically correct:
- Does the reviewer have full context, or are they missing information?
- Does the suggestion actually improve the code, or is it a preference?
- Would implementing it break existing functionality?
- Does it conflict with the project's established patterns?

### 3. Categorize each item

- **Valid and clear** — implement it
- **Valid but unclear** — ask for clarification before implementing. Guessing at what the reviewer meant leads to implementing the wrong change.
- **Incorrect** — push back with technical reasoning. Show evidence (code, tests, documentation) for why the current approach is correct.
- **Preference, not improvement** — acknowledge it, explain why you chose differently, and move on unless the reviewer insists

### 4. Implement fixes, then re-verify

After making changes based on valid feedback, run tests and verification again. Review fixes can introduce regressions. If the changes are significant (not just naming or style), re-run the relevant serial-review stages to verify the fixes don't break spec compliance or code quality.

## Common Pitfalls

- Performative responses ("Great point!", "You're absolutely right!") waste time and signal compliance over understanding. Respond with technical substance instead.
- Implementing suggestions without verifying them first risks introducing regressions — reviewers can be wrong, and their suggestions may not account for context they haven't seen.
- Partial fixes create inconsistency — if a suggestion requires changes across multiple places, apply all of them or none.
- Style arguments are rarely productive — if the project has a convention, follow it; if not, defer to the reviewer on style.

## When to Push Back

- The suggestion would break existing functionality
- The reviewer is missing context that changes the conclusion
- The suggestion violates YAGNI — adding complexity for hypothetical future needs
- The suggestion conflicts with the project's established patterns
- The suggestion is technically incorrect

When pushing back, provide evidence: point to code, tests, or documentation that supports your position. A respectful technical disagreement is more valuable than blind compliance.

## Completion

After implementing valid fixes:

1. Re-run the relevant serial-review stages on changed files — fixes can introduce new issues or break spec compliance.
2. Once verification passes, return to finishing-branch to update the PR description or integrate.
3. For PR feedback specifically, reply to reviewer comments explaining what was addressed and push the updated code. Reviewers need to see their feedback was processed, not just silently fixed.

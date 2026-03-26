---
name: receiving-review
description: "Use when receiving code review feedback from reviewers, teammates, or PR comments, before implementing any suggestions. TRIGGER when: review feedback arrives (from subagent reviewers, human reviewers, or PR comments). DO NOT TRIGGER when: you are the one dispatching a review (use serial-review instead)."
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
- **Valid but unclear** — ask for clarification before implementing. Do not guess what the reviewer meant.
- **Incorrect** — push back with technical reasoning. Show evidence (code, tests, documentation) for why the current approach is correct.
- **Preference, not improvement** — acknowledge it, explain why you chose differently, and move on unless the reviewer insists

### 4. Implement fixes, then re-verify

After making changes based on valid feedback, run tests and verification again. Review fixes can introduce regressions. If the changes are significant (not just naming or style), re-run the relevant serial-review stages to verify the fixes don't break spec compliance or code quality.

## What Not to Do

- Don't say "Great point!" or "You're absolutely right!" — these are performative, not technical
- Don't implement suggestions you haven't verified — a reviewer can be wrong
- Don't implement partial fixes — if a suggestion requires changes across multiple places, do all of them or none
- Don't argue about style preferences — if the project has a convention, follow it; if not, defer to the reviewer on style

## When to Push Back

- The suggestion would break existing functionality
- The reviewer is missing context that changes the conclusion
- The suggestion violates YAGNI — adding complexity for hypothetical future needs
- The suggestion conflicts with the project's established patterns
- The suggestion is technically incorrect

When pushing back, provide evidence: point to code, tests, or documentation that supports your position. A respectful technical disagreement is more valuable than blind compliance.

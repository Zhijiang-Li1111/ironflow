---
name: systematic-debugging
description: "Find the actual root cause instead of guessing — systematic investigation takes 15-30 minutes versus 2-3 hours of trial-and-error. Use when the user reports a bug, a test is failing, a build breaks, something behaves unexpectedly, or performance has degraded. Guides a methodical process: reproduce, trace the data flow, form and test hypotheses one at a time, then fix with TDD. Make sure to use this skill whenever something isn't working as expected, even if the user thinks they already know the cause."
---

# Systematic Debugging

Find the root cause before attempting fixes. Random fixes waste time and create new bugs.

Why: Guessing at fixes feels productive but leads to thrashing — 2-3 hours of trial and error versus 15-30 minutes of systematic investigation. The first fix you try sets the pattern for the rest of the session. Do it right from the start.

## Process

### Phase 1: Root Cause Investigation

Before attempting any fix:

1. **Read error messages carefully** — don't skip past them. Stack traces contain file paths, line numbers, and often the exact problem. Read completely.

2. **Reproduce consistently** — can you trigger the bug reliably? What are the exact steps? If not reproducible, gather more data instead of guessing.

3. **Check recent changes** — git diff, recent commits, new dependencies, config changes. What changed that could cause this?

4. **Trace the data flow** — where does the bad value originate? What called this function with the wrong input? Keep tracing backward until you find the source. Fix at the source, not at the symptom.

5. **In multi-component systems** — add diagnostic logging at each component boundary before attempting fixes. Run once, read the logs, identify which layer breaks. Then investigate that specific layer.

### Phase 2: Pattern Analysis

1. **Find working examples** — locate similar working code in the same codebase. What's different between the working version and the broken one?

2. **Compare against references** — if implementing a pattern, read the reference implementation completely. Don't skim.

3. **List every difference** — however small. Don't assume "that can't matter."

### Phase 3: Hypothesis and Testing

1. **Form a single hypothesis** — "I think X is the cause because Y." Be specific.

2. **Test with the smallest possible change** — one variable at a time. Don't fix multiple things at once.

3. **Verify** — did it work? Yes → Phase 4. No → form a new hypothesis. Don't pile more fixes on top.

### Phase 4: Fix with TDD

If the fix involves changes across multiple files or components, create a brief fix plan first — an ordered list of what to change and in what sequence. This doesn't require the full plan-review skill, just a numbered list to keep the fix organized.

1. **Write a failing test** that reproduces the bug — invoke the tdd skill via the Skill tool
2. **Verify it fails** for the right reason
3. **Implement a single fix** — address the root cause, one change only, no "while I'm here" improvements
4. **Verify the test passes** and no other tests broke
5. **Run serial-review** — invoke the serial-review skill via the Skill tool

### When 3+ fixes have failed

Stop attempting more fixes. Three failed attempts usually mean the problem is architectural, not a simple bug. Also escalate early if the fix requires changes to public interfaces, data models, or modifications across 3+ files — even before 3 failures.

Signs that the problem is architectural:
- Each fix reveals new problems in different places
- Fixes require "massive refactoring"
- Each fix creates new symptoms elsewhere
- The fix touches public interfaces, data models, or 3+ files

Discuss with the user before continuing. This is not a failed hypothesis — it's a wrong architecture. If the user agrees the problem is architectural, escalate to spec-first to redesign the affected area before attempting further fixes.

## Recognizing Shortcuts

| Shortcut | What to do instead |
|----------|-------------------|
| "Quick fix for now, investigate later" | Investigate now. Quick fixes become permanent. |
| "Just try changing X and see" | Form a hypothesis first. What evidence supports X as the cause? |
| "It's probably X, let me fix that" | "Probably" means you haven't verified. Verify first. |
| "I'll add multiple changes and run tests" | One change at a time. Can't isolate what worked otherwise. |
| "Skip the test, I'll manually verify" | Manual verification doesn't persist. Write the test. |
| "I see the problem, let me fix it" | Seeing symptoms is not understanding root cause. Trace the data flow. |
| "The issue is simple, don't need process" | Simple bugs have root causes too. The process is fast for simple bugs. |

## How This Connects to the Full Workflow

Bug fixes skip spec-first and plan-review — they go straight to:
1. **systematic-debugging** (this skill) — find root cause
2. **tdd** — write failing test, fix, verify
3. **serial-review** — spec compliance checks against the bug report (not a spec document), then code quality, then smoke test
4. **finishing-branch** — integrate the work

For large bugs that require design changes, escalate to spec-first after root cause is understood.

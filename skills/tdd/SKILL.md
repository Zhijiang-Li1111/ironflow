---
name: tdd
description: "Use when implementing any feature or bug fix, before writing implementation code. TRIGGER when: about to write production code, implementing a plan task, fixing a bug. DO NOT TRIGGER when: writing config files, throwaway prototypes, or user explicitly says to skip TDD."
---

# Test-Driven Development

Write the test first. Watch it fail. Write minimal code to pass.

Why: If you didn't watch the test fail, you don't know if it tests the right thing. Tests written after code pass immediately — that proves nothing about correctness.

## The Cycle

### RED — Write one failing test

- One behavior per test
- Clear name that describes the behavior
- Use real code, not mocks (unless unavoidable)

### Verify RED — Run and confirm failure

Run the test. Confirm:
- It fails (not errors from typos)
- The failure message is what you expect
- It fails because the feature is missing

If the test passes immediately, you're testing existing behavior. Fix the test.

### GREEN — Write minimal code to pass

Write the simplest code that makes the test pass. Don't add features, don't refactor other code, don't "improve" beyond what the test requires.

### Verify GREEN — Run and confirm passing

Confirm:
- The new test passes
- All existing tests still pass
- No errors or warnings in output

### REFACTOR — Clean up while green

- Remove duplication
- Improve names
- Extract helpers if needed

Keep tests green throughout. Don't add behavior during refactor.

### Repeat

Next failing test for the next behavior.

## Recognizing Shortcuts

If you wrote code before the test — delete it. Start over with the test.

"Keep it as reference" is a rationalization. Adapting existing code means you're testing after, not before. Delete means delete.

| Shortcut | What to do |
|----------|------------|
| Wrote code before test | Delete code. Write test first. |
| Test passes immediately | Test is wrong. Fix it. |
| "Too simple to test" | Simple code breaks. Test takes 30 seconds. |
| "I'll test after" | Tests-after answer "what does this do?" not "what should this do?" |
| "Need to explore first" | Fine. Throw away exploration, start with TDD. |
| "Test is hard to write" | Listen to the test — hard to test usually means hard to use. Simplify the design. |

## Bug Fix Pattern

1. Write a failing test that reproduces the bug
2. Verify it fails
3. Fix the bug with minimal change
4. Verify it passes
5. Commit

Never fix bugs without a test.

## Verification Checklist

Before marking work complete:

- [ ] Every new function has a test
- [ ] Watched each test fail before implementing
- [ ] Each test failed for the expected reason
- [ ] Wrote minimal code to pass each test
- [ ] All tests pass
- [ ] Output clean (no errors, warnings)
- [ ] Tests verify behavior, not mock behavior

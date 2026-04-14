---
name: tdd
description: "Catch bugs at the moment code is written, not after. Use when the user wants to implement a feature, fix a bug, or write production code with confidence. Make sure to use this skill whenever writing production code, even if the change seems small or straightforward."
---

# Test-Driven Development

Write the test first. Watch it fail. Write minimal code to pass.

Why: If you didn't watch the test fail, you don't know if it tests the right thing. Tests written after code pass immediately — that proves nothing about correctness.

## The Cycle

### RED — Write one failing test

- One behavior per test
- Clear name that describes the behavior
- Use real code, not mocks (unless unavoidable)

In compiled/typed languages, the test won't build if the system under test doesn't exist yet. Create a minimal stub — just the signature with an empty body, zero return value, or `notImplemented` throw. The goal is to make the code compile so the test can actually run and fail.

### Verify RED — Run and confirm failure

Run the test. Confirm:
- It fails (not errors from typos or missing imports)
- The failure message is what you expect
- It fails because the behavior is missing, not because the code doesn't compile

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

"Keep it as reference" leads to adapting existing code to fit the test, which means you're testing after, not before. Start fresh from the test.

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

Fixing bugs without a test means you can't prove the bug is fixed, and it may regress silently later.

## Verification Checklist

Before marking work complete:

- [ ] Every new function has a test
- [ ] Watched each test fail before implementing
- [ ] Each test failed for the expected reason
- [ ] Wrote minimal code to pass each test
- [ ] All tests pass
- [ ] Output clean (no errors, warnings)
- [ ] Tests verify behavior, not mock behavior

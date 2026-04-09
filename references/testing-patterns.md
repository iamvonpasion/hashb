# Testing Patterns

Quick-reference for test structure, naming, and common anti-patterns.
Used by `/tdd`, `/review`, `/eng`, and `/audit`.

## Test Structure: Arrange-Act-Assert

```
// Arrange — set up test scenario
// Act — perform the action being tested
// Assert — verify the outcome
```

Each test verifies one behavior. One assertion per concept.

## Test Pyramid

| Level | Share | Speed | Scope |
|-------|-------|-------|-------|
| Unit tests | ~80% | Milliseconds | Single function/class, no I/O |
| Integration tests | ~15% | Seconds | Component boundaries, localhost only |
| E2E tests | ~5% | Minutes | Full user flows, real browser |

## Naming Convention

Test names read like specifications: `[unit] [expected behavior] [condition]`

Good: `"user registration rejects duplicate email addresses"`
Bad: `"test1"`, `"should work"`, `"handles input"`

## Test Doubles (simplest first)

1. **Real implementation** — use when fast and deterministic
2. **Fake** — simplified working implementation (in-memory DB)
3. **Stub** — returns canned responses
4. **Mock** — verifies interactions (use sparingly)

Prefer real implementations. Mock only at system boundaries (databases, external APIs, network).

## DAMP Over DRY

In tests, clarity beats deduplication. Each test should read like a specification
without tracing shared helpers. Duplicate setup is acceptable when it improves
readability.

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|---|---|---|
| Testing implementation details | Refactoring breaks tests despite unchanged behavior | Test inputs and outputs only |
| Flaky tests | Erode confidence in the suite | Deterministic assertions, isolate state |
| Testing framework code | Wasted effort verifying third-party behavior | Test only your application code |
| Snapshot abuse | Large snapshots rarely reviewed, break on any change | Use targeted assertions |
| No test isolation | Tests pass alone but fail together | Each test owns its own setup/teardown |
| Over-mocking | Tests pass while production breaks | Prefer real implementations |
| Tests passing on first run | May not be testing intended behavior | Verify test fails without implementation (RED) |

## The Prove-It Pattern (Bug Fixes)

1. Write a test demonstrating the bug (must FAIL)
2. Confirm the test fails — proving the bug exists
3. Implement the fix
4. Verify the test PASSES
5. Run the full suite for regressions

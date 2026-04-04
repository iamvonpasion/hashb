---
id: testing
description: >
  Testing conventions for test files — behavior testing, descriptive names,
  test isolation, edge case coverage, AI-specific testing practices.
paths:
  - "**/*.test.*"
  - "**/*.spec.*"
  - "**/__tests__/**"
  - "**/test_*.py"
  - "**/tests/**"
---

# Testing Rules

## Core principles

- Use descriptive test names that explain the expected behavior, not the implementation
- Test behavior and outputs — avoid testing implementation details (method calls, internal state)
- Keep tests independent — no shared mutable state between tests
- Mock external services (APIs, databases, file systems), not internal modules

## Error and edge case coverage

- Test both happy path and error cases for every public function
- Test edge cases explicitly: nil/null, empty strings, empty collections, zero, negative numbers, boundary values, maximum lengths, and malformed input
- Test concurrent access if the code is used in multi-threaded or async contexts
- Test error paths with the same rigor as happy paths — AI-generated code fails most often on error handling

## AI-aware testing practices

- When AI generates both code and tests, review tests for implementation bias — tests should verify the specification, not mirror the implementation
- Use property-based testing for functions with invariants or contracts (e.g., "output is always sorted", "round-trip encode/decode is identity")
- Prefer testing with realistic data over synthetic data — AI-generated test data often misses real-world patterns
- Verify that assertions test meaningful behavior, not just "function returns something"

## Design principles for testability

Universal principles that make code testable. Skills adapt these to the consumer's
declared Stack at review/eng time — no stack-specific patterns here.

- **Single Responsibility** — one reason to change per unit (class, component, module, function)
- **Explicit Dependencies** — declare what you need, inject it. Never instantiate dependencies inline in business logic
- **Composition over Inheritance** — build from small, composable pieces. Deep hierarchies create brittle, hard-to-test code
- **Separation of Concerns** — logic, data, and presentation stay apart
- **Immutability by Default** — don't mutate shared state. Mutable shared state is the #1 source of flaky tests
- **Small Interfaces / Narrow Contracts** — don't force unused dependencies on consumers of your code

## Risk-based test strategy

- **Always test:** Business logic, data mutations, auth/authz, payment flows, error handling, API contracts, state machines
- **Test when non-trivial:** Transformations, validations, edge-case-heavy utilities, concurrent/async code
- **Skip testing:** Simple getters/setters, pass-through wrappers, framework boilerplate, one-line mappings
- Integration tests for critical cross-boundary flows (DB, external APIs, message queues) — use real dependencies where feasible, mocks where not
- The goal is confidence to ship, not coverage metrics. Test what would break trust if it failed in production

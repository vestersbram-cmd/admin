---
name: test-driven-development
description: Red-green-refactor methodology for writing tests before production code, ensuring correctness through iterative verification.
---

# Test-Driven Development

Write tests before production code. Let tests drive design and catch regressions.

## When to Use

- Starting a new feature where behavior can be specified up front
- Fixing a bug — write a failing test that reproduces it first
- Refactoring existing code — tests verify behavior stays intact
- Adding functionality to existing modules — extend the test suite first
- Any time correctness matters more than speed of initial delivery

## Core Cycle

### 1. Red — Write a Failing Test

- Write the smallest possible test that demonstrates the desired behavior.
- Run it. It must fail.
- If it passes, the test is wrong or the feature already exists.

### 2. Green — Make It Pass

- Write the minimal production code to make the test pass.
- Do not over-engineer. Hardcode if needed. Optimize later.
- All existing tests must still pass.

### 3. Refactor — Clean Up

- Remove duplication. Improve names. Simplify logic.
- Run the full test suite after every refactor step.
- Keep tests passing throughout.

## Principles

- **One concept per test** — each test should verify exactly one behavior.
- **Descriptive names** — test names should read like requirements: `should_return_error_when_input_is_empty`.
- **Fast feedback** — tests should run in seconds, not minutes.
- **Isolate dependencies** — use test doubles (mocks, stubs, fakes) for external services and side effects.
- **Test behavior, not implementation** — assert on outputs and state, not internal method calls.

## Workflow

1. Read the requirement or bug report.
2. Identify the smallest behavior that can be tested independently.
3. Write the test. Run it. Confirm it fails for the right reason.
4. Implement the minimal code to pass.
5. Run all tests. Confirm everything passes.
6. Refactor. Run tests again. Confirm they still pass.
7. Repeat until the feature is complete.

## What to Test

| Layer | What to Test | What to Avoid |
|---|---|---|
| Unit | Single function/class, isolated | Testing implementation details, private methods |
| Integration | Module interactions, data flow | Testing framework internals, external services directly |
| End-to-end | Critical user paths | Full UI automation for every edge case |

## Common Pitfalls

- **Writing tests after the code** — defeats the purpose. The test won't shape the design.
- **Testing too much at once** — large tests are hard to diagnose when they fail.
- **Mocking everything** — over-mocking hides real integration bugs.
- **Ignoring slow tests** — slow tests stop being run. Fix speed before it festers.
- **Skipping the refactor step** — green is not done. Refactor keeps the codebase healthy.

## Completion Criteria

- Every new behavior has at least one test that would fail if the behavior broke.
- The full test suite passes.
- No production code exists that isn't covered by a test (except trivial glue).
- Tests are fast enough to run before every commit.

---
name: behavior-driven-development
description: Use when implementing any feature or bugfix, before writing implementation code
---

# Behavior-Driven Development (BDD)

## Exceptions (Ask Human Partner First)

- Throwaway prototypes
- Generated code
- Configuration files

## Iron Law

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

Write code before the test? Delete it. Start over. No "reference", no "adapting".

## Contract First, Helper Last

- Start from **public entry points**: API / command / route / service method / provider contract.
- Test **observable outcomes and scenarios**, not internal steps.
- Group tests by **context and behavior**, not by function names.
- **Never** test private helpers unless they ARE the stable contract boundary.

Ask: *"What user-visible or contract-visible behavior am I protecting?"*
If unclear, the test is too close to the implementation.

## Red-Green-Refactor

### RED — One Failing Scenario

Write one minimal failing test at the contract boundary.

```typescript
test('should retry failed operations 3 times before succeeding', async () => {
  let attempts = 0;
  const operation = () => {
    attempts++;
    if (attempts < 3) throw new Error('fail');
    return 'success';
  };
  const result = await retryOperation(operation);
  expect(result).toBe('success');
  expect(attempts).toBe(3);
});
```

**Run it. Watch it fail.** Then verify:

- Test **fails** (assertion failure), not **errors** (import/syntax/runtime crash)
- Failure message matches expectation
- Fails because feature is missing, not typos
- Test passes immediately? → You're testing existing behavior. Fix the test.
- Test errors? → Fix the error first, re-run until it fails correctly.

### GREEN — Minimal Code to Pass

Write the simplest code that makes the test green. No extras.

**Run it.** Then verify:

- Target test passes
- All other tests still pass
- Output clean (no errors, no warnings)
- Test still fails? → Fix code, not test.
- Other tests broke? → Fix now before continuing.

### REFACTOR — Clean Up (Tests Stay Green)

Remove duplication, improve names, extract helpers. No new behavior.

### Repeat

Next scenario → next failing test.

## BDD Test Structure (Mandatory)

- `describe` → context/scenario, `it`/`test` → behavior assertion
- For pytest: class grouping = scenario, not method buckets
- Name **what should happen**, not what is called
- Prefer `should ...` or `when ... then ...` phrasing

```typescript
describe('EmailSubmission', () => {
  describe('when email is empty', () => {
    it('should return validation error', async () => {
      const result = await submitForm({ email: '' });
      expect(result.error).toBe('Email required');
    });
  });
});
```

```python
class TestLiveBars:
    def test_should_drain_buffer_and_insert_new_rows(self):
        ...
    def test_should_not_write_when_buffer_is_empty(self):
        ...
```

## Verification Checklist

Before marking complete:

- [ ] Every new behavior has a failing test first
- [ ] Watched each test fail for the expected reason
- [ ] Wrote minimal code to pass
- [ ] All tests pass, output clean
- [ ] Mocks only where unavoidable; tested real behavior
- [ ] Tests organized by scenario/behavior, not implementation
- [ ] Edge cases and errors covered

Can't check all boxes? Start over.

## Anti-Patterns

Read @testing-anti-patterns.md before adding mocks or test utilities. Key rules:

| Don't | Do |
|-------|----|
| Assert on mock elements | Test real component behavior |
| Add test-only methods to production | Put in test utilities |
| Mock without understanding side effects | Understand deps first, mock minimally |
| Create partial/incomplete mocks | Mirror real API structure completely |
| Test private helpers by default | Test public contract |
| Group tests by internal methods | Group by scenario/behavior |

## Red Flags — STOP and Start Over

Any of these means you've left BDD. Delete code, restart with a failing test:

- Wrote production code before test
- Test passes immediately (testing existing behavior, not new)
- Can't explain why the test failed
- Test named after `_private_helper` or internal method
- Test class grouped by implementation step, not behavior
- Asserting on mock elements instead of real behavior
- Rationalizing "just this once" / "too simple to test" / "I'll test after"
- Keeping pre-written code as "reference" instead of deleting

## Bug Fix Workflow

Bug found? → Write a failing test that reproduces it first. Then follow Red-Green-Refactor. Never fix bugs without a test.

## When Stuck

| Problem | Solution |
|---------|----------|
| Don't know how to test | Write the wished-for API first. Ask your human partner. |
| Test too complicated | Design too complicated. Simplify interface. |
| Must mock everything | Code too coupled. Use dependency injection. |
| Bug found | Write failing test reproducing it. Then fix. |

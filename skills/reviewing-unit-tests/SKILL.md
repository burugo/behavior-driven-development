---
name: reviewing-unit-tests
description: "Use in a dedicated subagent for post-TDD blind unit-test review against requirements and public contracts to find missing cases, wrong assertions, and brittle tests."
---

# Reviewing Unit Tests

Blind-review unit tests using only `requirements + public contract + tests` to find three classes of issues: `missing tests`, `wrong tests`, and `brittle tests`.

## Use When

- The main agent has written the tests, verified they fail, then added the minimum implementation needed to make them pass.
- An isolated reviewer is needed to check for missing counterexamples, boundaries, or error paths.
- You need to judge whether the tests protect external behavior or merely mirror the implementation.

**How to invoke it:** The main agent dispatches an isolated subagent through the Task tool, and that subagent loads this skill. The main agent must not load it directly, because direct loading exposes the production implementation in the current context and breaks blind-review isolation.

Do not use it for:
- Coverage audits based on implementation diffs.
- Reviewing TDD ceremony, commit slicing, or process compliance.
- Reverse-engineering the contract by peeking at the implementation.

## Inputs

Default inputs are only:
- `Requirements`
- `Public contract`
- `Existing tests`
- `Assumptions` (if any)

Do not read the production implementation by default. Only read it when the user explicitly asks for an implementation-aware review.

## Hard Rules

- `Tests do not define contract.` Existing tests must not be used to redefine the requirements.
- Any unspecified behavior is a `contract hole`, not a `missing test`.
- Only externally observable behavior counts as contract; internal call order, private helpers, and temporary intermediate state do not count by default.
- For every `Blocking` or `Important` finding, show at least one concrete false-positive path or harmless refactor that the current tests would fail to catch correctly.
- Do not score the tests, report coverage percentages, or write vague praise.
- Once a stop condition is triggered, stop the review. Do not continue assigning severities while also saying the result is uncertain.

## Stop Conditions

Stop immediately when:
- There is no usable `Public contract`.
- The requirements contain material ambiguity.
- The provided tests are actually integration or end-to-end tests.
- The scope is too large to evaluate within a clear boundary.

When a stop condition is triggered, output only:
- What input is missing.
- Why a reliable review is not possible yet.
- What smallest scope the review should be narrowed to.

## Workflow

1. Lock the boundary: which contract is under review, which test files are in scope, and which behaviors are included.
2. Extract `behavior obligations`: check only obligations that the contract or requirements explicitly promise or directly imply. `happy path`, `boundary values`, `invalid input`, `error semantics`, `observable invariants`, `observable side effects`, `repeated calls`, `idempotence`, `rollback/cleanup after failure`, and `resource lifetime` are candidate categories, not a mandatory checklist.
3. Map the existing tests to those obligations, counting only externally observable assertions.
4. Mark findings:
   - `Missing tests`: The contract promises the behavior, but no test protects it.
   - `Wrong tests`: A test exists, but its assertion is too weak, incorrect, or still allows a broken implementation to pass.
   - `Brittle tests`: The test is tied to implementation details, breaks under harmless refactors, and does not actually protect the contract.
5. Sort findings by severity:
   - `Blocking`: The core contract is unprotected, or an obviously broken implementation could still pass.
   - `Important`: A key boundary or error path is missing, or a highly brittle test appears on an important path.
   - `Nice-to-have`: Raise these only when they reduce confusion or maintenance risk; do not turn them into style review.
6. Output using `review-template.md`.

## Review Heuristics

Prioritize these as likely problems:
- Assertions that only check `is not None`, `len() > 0`, or generic truthiness.
- Vague assertions that hide the important result.
- Recreating implementation logic to compute the expected value.
- Over-mocking so the tests no longer cover the real contract.
- Asserting on private helpers or internal collaborator call order.
- Tests that depend on shared state, execution order, or polluted fixtures.
- Tests that rely on wall-clock time, `sleep` timing, randomness, scheduler order, locale/timezone, filesystem ordering, environment leakage, or live external services without the contract explicitly controlling that variability.
- Smart fixtures, builders, or test helpers that hide contract-relevant setup or derive expected results instead of making behavior explicit.
- Table-driven or parameterized tests that mix unrelated obligations or make it hard to tell which behavior failed.
- Assertions that only verify collaborator calls even though the contract exposes a direct observable result.

If the contract does not define an input, boundary, or error semantic:
- Do not classify it as a `missing test`.
- Put it under `Contract holes / assumptions`.

## Output

Use `review-template.md`.

Each finding should include, when possible:
- The relevant `behavior obligation`.
- Evidence from the existing tests, or a clear statement that no test covers it.
- Why the current assertion is insufficient, incorrect, or brittle.
- For `Blocking` and `Important` findings, a concrete broken implementation that could still pass, or a harmless refactor that would break the test.
- A minimal direction for fixing it, when needed.

If no issues are found, state clearly: `No clear missing, wrong, or brittle tests found.` Then add one sentence about residual risk.

## Forbidden

- Do not output a scorecard, coverage matrix, or completion checklist.
- Do not pad the report with lists of things that are already covered well.
- Do not suggest adding direct tests for a private helper unless it is itself the public contract.
- Do not force tests to cover a branch just because the implementation contains one.
- Do not fix an already over-mocked test by adding even more mocks.

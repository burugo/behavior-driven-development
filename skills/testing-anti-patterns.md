# Testing Anti-Patterns

**Load this reference when:** writing or changing tests, adding mocks, or tempted to add test-only methods to production code.

## The Iron Laws

```
1. NEVER test mock behavior
2. NEVER add test-only methods to production classes
3. NEVER mock without understanding dependencies
```

## Anti-Pattern 1: Testing Mock Behavior

```typescript
// ❌ BAD
test('renders sidebar', () => {
  render(<Page />);
  expect(screen.getByTestId('sidebar-mock')).toBeInTheDocument();
});

// ✅ GOOD
describe('Page', () => {
  describe('when sidebar is rendered', () => {
    it('should display navigation', () => {
      render(<Page />);
      expect(screen.getByRole('navigation')).toBeInTheDocument();
    });
  });
});
// OR if sidebar must be mocked: don't assert on the mock, test Page's behavior
```

### Gate Function

```
BEFORE asserting on any mock element:
  Ask: "Am I testing real behavior or just mock existence?"
  IF mock existence: STOP - delete assertion or unmock the component
```

## Anti-Pattern 2: Test-Only Methods in Production

```typescript
// ❌ BAD: destroy() only used in tests
class Session {
  async destroy() {
    await this._workspaceManager?.destroyWorkspace(this.id);
  }
}
afterEach(() => session.destroy());

// ✅ GOOD: keep production class clean
// In test-utils/
export async function cleanupSession(session: Session) {
  const workspace = session.getWorkspaceInfo();
  if (workspace) await workspaceManager.destroyWorkspace(workspace.id);
}
afterEach(() => cleanupSession(session));
```

### Gate Function

```
BEFORE adding any method to production class:
  Ask: "Is this only used by tests?" → IF yes: STOP, put it in test utilities
  Ask: "Does this class own this resource's lifecycle?" → IF no: STOP, wrong class
```

## Anti-Pattern 3: Mocking Without Understanding

```typescript
// ❌ BAD: Mock silently breaks test logic
test('detects duplicate server', () => {
  vi.mock('ToolCatalog', () => ({
    discoverAndCacheTools: vi.fn().mockResolvedValue(undefined)
    // Prevents config write the test depends on!
  }));
  await addServer(config);
  await addServer(config);  // Should throw - but won't
});

// ✅ GOOD: Mock at the right level
describe('ServerConfig', () => {
  describe('when adding a server that already exists', () => {
    it('should throw a duplicate server error', async () => {
      vi.mock('MCPServerManager'); // Mock slow startup, not config write
      await addServer(config);
      await expect(addServer(config)).rejects.toThrow('duplicate');
    });
  });
});
```

### Gate Function

```
BEFORE mocking any method:
  1. Ask: "What side effects does the real method have?"
  2. Ask: "Does this test depend on any of those side effects?"
  3. Ask: "Do I fully understand what this test needs?"

  IF depends on side effects:
    Mock at lower level (the actual slow/external operation)
    NOT the high-level method the test depends on

  IF unsure: Run test with real implementation FIRST, observe what happens, THEN mock minimally

  Red flags: "I'll mock this to be safe" / "This might be slow, better mock it"
```

## Anti-Pattern 4: Incomplete Mocks

```typescript
// ❌ BAD: Partial mock - missing fields downstream code uses
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' }
  // Missing: metadata.requestId that downstream code accesses
};

// ✅ GOOD: Mirror real API completely
const mockResponse = {
  status: 'success',
  data: { userId: '123', name: 'Alice' },
  metadata: { requestId: 'req-789', timestamp: 1234567890 }
};
```

### Gate Function

```
BEFORE creating mock responses:
  1. Examine actual API response from docs/examples
  2. Include ALL fields system might consume downstream
  3. Verify mock matches real response schema completely

  If uncertain: include all documented fields
```

## Anti-Pattern 5: Testing Private Helpers by Default

```python
# ❌ BAD: Testing internal decomposition
class TestNormalizeBars:
    def test_normalize_uses_factor_dict(self):
        rows = provider._normalize_bars(...)
        assert rows[0][6] == 8.45

# ✅ GOOD: Test the promised behavior
class TestLiveBars:
    def test_should_drain_buffer_dedup_and_insert_rows(self):
        rows = asyncio.run(provider.fetch_live_bars(...))
        assert rows == inserted_rows
```

### Gate Function

```
BEFORE writing a direct test for any helper:
  Ask: "Is this helper itself the stable contract?" → IF no: STOP, test the public entry point
  Ask: "Can I express this as an externally observable behavior?" → IF yes: write that scenario instead
```

## Anti-Pattern 6: Grouping Tests by Internal Methods

```python
# ❌ BAD: Mirrors implementation structure
class TestParseBatchResult:
    def test_parse_batch_result(self): ...
class TestNormalizeBars:
    def test_normalize_single_bar(self): ...

# ✅ GOOD: Grouped by scenario/behavior
class TestGalaxyLiveBars:
    def test_should_drain_buffer_and_insert_new_rows(self): ...
    def test_should_not_insert_duplicate_rows(self): ...
```

### Gate Function

```
BEFORE creating a new test class/module section:
  Ask: "Am I grouping by scenario/behavior or by method name?"
  IF by method/helper: STOP - regroup by user-visible behavior or public contract
```

## Anti-Pattern 7: Tests as Afterthought

### Gate Function

```
BEFORE claiming implementation complete:
  Ask: "Does every new behavior have a failing scenario that was written first?"
  IF no: STOP - write the scenario, watch it fail, then implement
```

## When Mocks Become Too Complex

Warning signs → consider integration tests with real components instead:
- Mock setup longer than test logic
- Mocking everything to make test pass
- Test breaks when mock changes
- Can't explain why mock is needed

## Quick Reference

| Anti-Pattern | Fix |
|--------------|-----|
| Assert on mock elements | Test real component or unmock it |
| Test-only methods in production | Move to test utilities |
| Mock without understanding | Understand deps first, mock minimally |
| Incomplete mocks | Mirror real API completely |
| Test private helpers by default | Start from public contract |
| Group tests by internal methods | Group by scenario and behavior |
| Tests as afterthought | Write failing scenario first |
| Over-complex mocks | Consider integration tests |

## Red Flags

- Assertion checks for `*-mock` test IDs
- Test names centered on `_private_helper` or internal transformation names
- Test classes named after implementation steps instead of behaviors
- Methods only called in test files
- Mock setup is >50% of test
- Test fails when you remove mock
- Can't explain why mock is needed
- Mocking "just to be safe"

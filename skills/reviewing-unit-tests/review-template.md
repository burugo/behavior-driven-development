## Unit Test Review

### Blocking issues
- `[file:line]` Obligation: `...`
  - Evidence: `existing test` or `missing coverage`
  - Risk: `why a bad implementation can still pass`
  - False-positive path: `concrete broken implementation or harmless refactor that current tests mishandle`
  - Fix direction: `smallest high-leverage correction`

### Important issues
- `[file:line]` Obligation: `...`
  - Evidence: `existing test` or `missing coverage`
  - Risk: `why this gap matters`
  - False-positive path: `concrete broken implementation or harmless refactor that current tests mishandle`
  - Fix direction: `smallest useful correction`

### Nice-to-have
- `[file:line]` Risk: `only if it reduces misdirection, brittleness, or maintenance cost`

### Contract holes / assumptions
- `what is not defined clearly enough to judge test completeness`

### Suggested high-leverage tests
- `test_name`: input / condition / expected observable result
- `test_name`: input / condition / expected observable result

## Rules

- Keep findings concrete.
- For every `Blocking` or `Important` issue, include one concrete false-positive path or harmless refactor scenario.
- Prefer 1-3 high-leverage tests, not a shopping list.
- Fill `Suggested high-leverage tests` only when a Blocking/Important finding or a contract hole requires it; otherwise omit that section.
- Do not output scores, percentages, or a coverage matrix.
- If there are no findings, write: `No clear missing, wrong, or brittle tests found.`

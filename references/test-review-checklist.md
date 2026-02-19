# Test Review Checklist

## Severity Levels

| Level | Name | Description | Action |
|-------|------|-------------|--------|
| **P0** | Critical | Test gives false confidence — passes but doesn't verify correctness | Must fix immediately |
| **P1** | High | Major quality issue — fragile, over-mocked, or missing critical coverage | Should fix before merge |
| **P2** | Medium | Maintainability concern — readability, naming, structure | Fix in this PR or follow-up |
| **P3** | Low | Style or minor suggestion | Optional improvement |

---

## 1. Four Pillars Assessment

### Protection Against Regressions
- [ ] Tests cover critical business rules, not just happy paths
- [ ] Edge cases and boundary conditions are tested
- [ ] Error paths are tested (not just "it throws" but state after error)
- [ ] Non-trivial code is exercised (not just getters/setters)
- [ ] Tests would catch real bugs if code changes

**Questions to ask:**
- "If I introduce a bug in line X, will any test fail?"
- "Are the most important business rules covered?"

### Resistance to Refactoring
- [ ] Tests verify behavior, not implementation details
- [ ] No assertions on method call sequences (unless unmanaged dependency)
- [ ] No mocking of internal collaborators
- [ ] Private methods are not tested directly
- [ ] Tests would survive renaming/restructuring of internals

**Red flag**: If refactoring internal implementation (without changing behavior) would break tests → P1 finding.

### Fast Feedback
- [ ] No `sleep()`, `time.sleep()`, or real delays in unit tests
- [ ] No real network calls or database queries in unit tests
- [ ] No file system I/O in unit tests (use fakes or in-memory)
- [ ] Individual test runs in under 100ms
- [ ] Full unit suite runs in under 30 seconds

### Maintainability
- [ ] Tests are readable and self-contained
- [ ] Setup is minimal and relevant to the scenario
- [ ] Assertions are clear and meaningful
- [ ] Test names describe the behavior, not the implementation
- [ ] No duplicated setup — extracted to helpers/builders where appropriate

---

## 2. Antipattern Detection

### Critical (P0)
- [ ] **The Liar**: Tests without meaningful assertions
- [ ] **Shared Mutable State**: Tests that affect each other's state

### High (P1)
- [ ] **The Giant**: Tests with multiple Act steps
- [ ] **The Mockery**: More mock setup than actual test logic
- [ ] **The Inspector**: Testing private methods/fields directly
- [ ] **Fragile Tests**: Tests coupled to implementation details
- [ ] **Ice Cream Cone**: Overreliance on E2E, few unit tests

### Medium (P2)
- [ ] **The Slow Poke**: Unnecessary delays or real I/O
- [ ] **Second Class Citizen**: Copy-paste, magic values, no helpers
- [ ] **Conditional Logic**: if/for/try-catch in test body

### Low (P3)
- [ ] **100% Coverage Trap**: Tests for trivial code
- [ ] Minor naming inconsistencies
- [ ] Excessive comments in straightforward tests

---

## 3. Business Logic Coverage

### Coverage quality (not just code coverage)
- [ ] Critical business rules have dedicated tests
- [ ] Decision table scenarios are covered (all condition combinations)
- [ ] Boundary values tested at edges of valid ranges
- [ ] Domain invariants are verified
- [ ] State machine transitions tested (valid + invalid)
- [ ] Concurrent/race condition scenarios addressed if applicable

### Missing coverage detection
- [ ] Look for untested public methods with non-trivial logic
- [ ] Check if error/exception paths are tested
- [ ] Verify that validation rules have corresponding tests
- [ ] Check for complex conditionals (if/switch) without test coverage
- [ ] Look for domain events/side effects without verification

---

## 4. Test Doubles Usage

### Appropriate usage
- [ ] Mocks used only for unmanaged out-of-process dependencies
- [ ] Stubs return realistic values (not `null` / `undefined` everywhere)
- [ ] Fakes preferred over mocks for complex dependencies
- [ ] Real objects used for value objects, DTOs, in-process collaborators
- [ ] No mocking of standard library / language primitives

### Problematic patterns
- [ ] Mocking the class under test (always wrong)
- [ ] Deeply nested mock returns (`mock.a.b.c.return_value`)
- [ ] Verifying exact call counts/order on internal methods
- [ ] Mock setup longer than the test itself
- [ ] Mocking what you don't own (third-party libraries directly)

---

## 5. Naming and Readability

### Test names
- [ ] Names describe behavior, not method names
- [ ] Names include the condition/scenario
- [ ] Names include the expected outcome
- [ ] Consistent naming convention throughout the suite
- [ ] Names are readable as specifications

### Test body
- [ ] Clear AAA (Arrange-Act-Assert) structure
- [ ] Single Act per test
- [ ] Assertions verify specific expected values, not just `is not None`
- [ ] Magic values have context (via variable names or comments)
- [ ] Setup relevance is clear (no hidden shared setup affecting the test)

---

## 6. Test Isolation

- [ ] Tests can run in any order
- [ ] Tests can run in parallel
- [ ] No shared mutable state between tests
- [ ] Each test creates its own data
- [ ] Cleanup happens in teardown, not in the next test's setup
- [ ] No dependency on external services (database, API, file system) in unit tests
- [ ] Time-dependent tests use injected clocks, not real time

---

## Review Output Format

```markdown
## Test Review Summary

**Files reviewed**: X test files, Y test cases
**Overall assessment**: [APPROVE / REQUEST_CHANGES / COMMENT]
**Coverage quality**: [Good / Adequate / Insufficient]

---

## Findings

### P0 - Critical
(none or list)

### P1 - High
1. **[test_file:line]** Brief title
   - **Antipattern**: Which antipattern
   - **Problem**: What's wrong
   - **Impact**: Why it matters
   - **Fix**: Suggested correction

### P2 - Medium
...

### P3 - Low
...

---

## Coverage Gaps

- [List of business rules / paths not covered by any test]

## Positive Observations

- [What's done well — reinforce good practices]

## Suggested Improvements

- [Structural improvements, test helpers, patterns to adopt]
```

---

## Quick-Start Review Sequence

When reviewing tests, follow this order:

1. **Scan for Liars** (P0) — do all tests have meaningful assertions?
2. **Check isolation** (P0) — any shared mutable state?
3. **Check doubles usage** (P1) — over-mocking? mocking internals?
4. **Check fragility** (P1) — do tests verify behavior or implementation?
5. **Check coverage quality** (P1) — are critical paths tested?
6. **Check readability** (P2) — names, structure, clarity
7. **Check speed** (P2) — any unnecessary delays or real I/O?
8. **Check for trivial tests** (P3) — testing getters/constructors?

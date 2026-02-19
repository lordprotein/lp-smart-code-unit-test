---
name: lp-smart-code-unit-test
description: "Write ideal unit tests and review existing ones. Two modes: generate new tests from code analysis, or review existing tests for antipatterns and quality issues. Language-agnostic, based on best practices from Beck, Fowler, Khorikov, and Google."
---

# Unit Test Expert

## Overview

Generate high-quality unit tests or review existing tests using industry best practices. Works in two modes:

- **Mode 1: Generate Tests** — analyze code and produce comprehensive tests
- **Mode 2: Review Tests** — evaluate existing tests for quality, coverage, and antipatterns

Default to **Generate** mode unless the user explicitly asks to review existing tests or the input is clearly test code.

## Severity Levels

| Level | Name | Description | Action |
|-------|------|-------------|--------|
| **P0** | Critical | False confidence — test passes but doesn't verify correctness | Must fix immediately |
| **P1** | High | Major quality issue — fragile, over-mocked, missing critical coverage | Should fix before merge |
| **P2** | Medium | Maintainability concern — readability, naming, structure | Fix in this PR or follow-up |
| **P3** | Low | Style or minor suggestion | Optional improvement |

---

## Mode 1: Generate Tests

### 1) Preflight context

- Use `git status -sb`, `git diff --stat`, and `git diff` to scope changes. If user specifies files, analyze those instead.
- Identify the programming language, test framework, and assertion library used in the project.
- Search for existing tests (`rg -g "*test*" -g "*spec*" --files`) to discover:
  - Test directory structure and file naming conventions
  - Test framework and assertion patterns in use
  - Helper functions, builders, fixtures, or factories already defined
  - Naming conventions for test methods
- If no existing tests found, ask user about preferred framework or propose a standard one for the language.

**Edge cases:**
- **No changes**: If `git diff` is empty, ask user which files/modules to generate tests for.
- **Large diff (>500 lines)**: Summarize modules first, then ask user to prioritize which to test.
- **Test files in diff**: Switch to Mode 2 (Review) for the test files, Mode 1 for production files.

### 2) Classify code (Khorikov matrix)

- Load `references/testing-principles.md` for the classification matrix.
- Categorize each unit of code:

| Category | Test strategy |
|----------|--------------|
| **Domain model / algorithms** | Unit test extensively — highest priority |
| **Trivial code** (getters, DTOs, one-line delegations) | Skip — no tests needed |
| **Controllers / orchestrators** | Integration tests only — don't unit test |
| **Overcomplicated code** (high complexity + many dependencies) | Recommend refactoring first, then test |

- Report the classification to the user before generating tests.
- If code is overcomplicated, suggest applying Humble Object pattern to extract testable logic.

### 3) Determine test scenarios

- Load `references/business-logic-testing.md` for scenario identification techniques.
- For each domain logic unit, identify:
  - **Happy paths**: Core business rules, primary use cases
  - **Decision table scenarios**: All condition combinations for complex rules
  - **Boundary conditions**: Min, max, zero, empty, null, off-by-one
  - **Error paths**: Invalid input, constraint violations, exception handling
  - **Invariants**: Conditions that must always hold after any operation
  - **State transitions**: Valid and invalid transitions (if applicable)
- Prioritize: business-critical paths first, edge cases second, defensive checks last.

### 4) Generate tests

- Load `references/test-design-patterns.md` for patterns.
- Load `references/test-doubles-guide.md` for double selection.
- Apply these rules:

**Structure:**
- Follow AAA pattern (Arrange-Act-Assert) with clear visual separation
- One Act per test — never test multiple behaviors in one test
- No conditional logic in tests (no if/for/try-catch)
- Group tests logically (by behavior/feature, using describe/context if framework supports)

**Naming:**
- Use project's existing naming convention if detected
- Otherwise use descriptive names: `test_[scenario]_[expected_result]` or `test_[behavior_description]`
- Name must describe WHAT is tested and WHAT the expected outcome is
- Use domain language, not implementation terms

**Test doubles:**
- Default to real objects for in-process dependencies
- Use stubs only when real setup is impractical and behavior is irrelevant to this test
- Mock ONLY unmanaged out-of-process dependencies (external APIs, SMTP, payment gateways)
- Create adapters/wrappers for third-party libraries — never mock what you don't own
- Prefer fakes over mocks when dependency has complex behavior

**Data setup:**
- Use builders/factories if they exist in the project
- Create new builders if the same object setup repeats 3+ times
- Use parameterized tests for decision tables and boundary value sets
- Every magic value in a test should have a clear purpose (via variable name or context)

**Assertions:**
- Assert on specific expected values, not just `is not None` or `doesn't throw`
- For error paths, assert on system state AFTER the error (not just the exception)
- One logical concept per test (may be multiple assert statements)
- Use the project's assertion style (expect/assert/should)

### 5) Self-check against antipatterns

- Load `references/antipatterns-checklist.md`.
- Before outputting, verify each generated test against:
  - [ ] Not a Liar — has meaningful assertions
  - [ ] Not a Giant — single Act, focused assertions
  - [ ] Not a Mockery — minimal, justified mocks
  - [ ] Not an Inspector — tests public behavior only
  - [ ] Not Fragile — would survive refactoring
  - [ ] No shared mutable state
  - [ ] No conditional logic
  - [ ] Not testing trivial code
- If any check fails, fix the test before outputting.

### 6) Output format

```markdown
## Test Generation Summary

**Source**: X files analyzed, Y functions/methods identified
**Classification**: Domain logic: A, Trivial (skipped): B, Controllers (skip for unit test): C
**Tests generated**: Z test cases

---

## Code Classification

| File / Function | Category | Test strategy |
|----------------|----------|---------------|
| `order.calculate_total()` | Domain model | Unit test ✓ |
| `OrderDTO` | Trivial | Skip |
| `OrderController.create()` | Controller | Integration test (not generated) |

---

## Generated Tests

### [filename_test.ext]

[Test code with comments explaining the scenario for non-obvious cases]

---

## Coverage Notes

- **Tested**: [List of business rules covered]
- **Not tested (trivial)**: [List of skipped trivial code]
- **Needs integration test**: [List of controllers/orchestrators]
- **Needs refactoring first**: [List of overcomplicated code, if any]

---

## Next Steps

1. **Run tests** — Shall I run the generated tests to verify they pass?
2. **Add more scenarios** — Any edge cases or business rules I missed?
3. **Create test helpers** — Shall I extract builders/factories for reusable setup?
```

---

## Mode 2: Review Tests

### 1) Preflight context

- Collect existing test files: from `git diff` or user-specified files.
- Identify test framework, assertion library, and project conventions.
- Read the corresponding production code to understand what's being tested.

**Edge cases:**
- **No test files found**: Inform user and offer to switch to Mode 1 (Generate).
- **Mixed test and production code in diff**: Review tests (Mode 2), generate tests for uncovered production code (Mode 1).

### 2) Evaluate against checklist

- Load `references/test-review-checklist.md` for the full checklist.
- Load `references/antipatterns-checklist.md` for antipattern detection.
- Load `references/test-doubles-guide.md` for doubles assessment.

Follow the quick-start review sequence:
1. **Scan for Liars** (P0) — meaningful assertions in every test?
2. **Check isolation** (P0) — shared mutable state?
3. **Check doubles usage** (P1) — over-mocking? mocking internals?
4. **Check fragility** (P1) — behavior or implementation verification?
5. **Check coverage quality** (P1) — critical paths tested?
6. **Check readability** (P2) — names, AAA structure, clarity
7. **Check speed** (P2) — unnecessary delays or real I/O?
8. **Check for trivial tests** (P3) — testing getters/constructors?

### 3) Assess severity

Assign severity to each finding:
- **P0**: False confidence, shared mutable state
- **P1**: Over-mocking, fragile tests, missing critical coverage, testing implementation details
- **P2**: Readability issues, slow tests, poor naming, second-class test code
- **P3**: Trivial test coverage, minor style issues

### 4) Output format

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
   - **Fix**: Suggested correction with code example

### P2 - Medium
2. (continue numbering)

### P3 - Low
...

---

## Coverage Gaps

- [Business rules / paths not covered by any test]

## Positive Observations

- [What's done well — reinforce good practices]

---

## Next Steps

I found X issues (P0: _, P1: _, P2: _, P3: _).

**How would you like to proceed?**

1. **Fix all** — I'll implement all suggested fixes
2. **Fix P0/P1 only** — Address critical and high priority issues
3. **Fix specific items** — Tell me which issues to fix
4. **No changes** — Review complete, no implementation needed

Please choose an option or provide specific instructions.
```

**Important**: Do NOT implement any changes until user explicitly confirms. This is a review-first workflow.

---

## Resources

### references/

| File | Purpose |
|------|---------|
| `testing-principles.md` | Four pillars, FIRST, test pyramid, code classification, behavior vs implementation |
| `test-design-patterns.md` | AAA, Given-When-Then, builders, parameterization, naming, organization |
| `test-doubles-guide.md` | Dummy/Stub/Spy/Mock/Fake taxonomy, Classical vs London, decision tree |
| `business-logic-testing.md` | Decision tables, boundaries, invariants, state machines, error paths |
| `antipatterns-checklist.md` | 12 antipatterns with examples, detection signals, and fixes |
| `test-review-checklist.md` | Structured checklist for reviewing existing tests by severity |

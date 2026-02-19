# LP Smart Code Unit Test

A comprehensive unit testing skill for AI agents. Generates ideal tests and reviews existing ones using best practices from Kent Beck, Martin Fowler, Vladimir Khorikov, and Google.

## Installation

```bash
npx skills add lordprotein/lp-smart-code-unit-test
```

Or via Agent Skills:

```bash
npx agent-skills-cli install @lordprotein/lp-smart-code-unit-test
```

## Features

- **Two Modes** — Generate new tests or review existing ones
- **Code Classification** — Khorikov matrix: domain model, trivial, controllers, overcomplicated
- **Business Logic Focus** — Decision tables, boundary analysis, invariants, state machines
- **Smart Test Doubles** — Classical school by default, mocks only for unmanaged dependencies
- **Antipattern Detection** — 12 antipatterns with severity levels and fixes
- **Language Agnostic** — Works with any language and test framework
- **Project-Aware** — Discovers existing conventions, helpers, and patterns

## Usage

After installation, run:

```
/lp-smart-code-unit-test
```

### Mode 1: Generate Tests

The skill analyzes your code changes (or specified files) and generates comprehensive unit tests:

1. Classifies code by testability (domain logic, trivial, controllers)
2. Identifies test scenarios (happy paths, boundaries, errors, invariants)
3. Generates tests following AAA pattern with proper naming
4. Self-checks against antipatterns before output

### Mode 2: Review Tests

The skill reviews existing tests for quality issues:

1. Evaluates against the Four Pillars (Khorikov)
2. Detects antipatterns (The Liar, The Mockery, Fragile Tests, etc.)
3. Assesses business logic coverage gaps
4. Reports findings by severity (P0-P3)
5. Asks for confirmation before implementing fixes

## Workflow

### Generate Mode
1. **Preflight** — Analyze code, detect language/framework, find conventions
2. **Classify** — Khorikov matrix (domain model / trivial / controller / overcomplicated)
3. **Scenarios** — Business rules, boundaries, error paths, invariants
4. **Generate** — AAA pattern, proper doubles, meaningful names
5. **Self-check** — Verify against antipatterns checklist
6. **Output** — Tests with classification and coverage notes

### Review Mode
1. **Preflight** — Collect test files, read production code
2. **Evaluate** — Check against review checklist
3. **Severity** — Assign P0-P3 to each finding
4. **Output** — Findings with suggested fixes
5. **Confirm** — Ask user before implementing changes

## Severity Levels

| Level | Name | Action |
|-------|------|--------|
| P0 | Critical | Must fix — false confidence or test pollution |
| P1 | High | Should fix — fragile, over-mocked, missing coverage |
| P2 | Medium | Fix or follow-up — readability, naming, structure |
| P3 | Low | Optional — style, minor suggestions |

## Structure

```
lp-smart-code-unit-test/
├── SKILL.md                           # Main skill definition
├── agents/
│   └── agent.yaml                     # Agent interface config
└── references/
    ├── testing-principles.md          # Four pillars, FIRST, pyramid, classification
    ├── test-design-patterns.md        # AAA, builders, parameterization, naming
    ├── test-doubles-guide.md          # Dummy/Stub/Spy/Mock/Fake guide
    ├── business-logic-testing.md      # Decision tables, boundaries, invariants
    ├── antipatterns-checklist.md       # 12 antipatterns with detection and fixes
    └── test-review-checklist.md       # Structured review checklist
```

## Knowledge Base

Based on:
- **Vladimir Khorikov** — Unit Testing: Principles, Practices, and Patterns
- **Kent Beck** — Test-Driven Development: By Example
- **Martin Fowler** — Refactoring, TestDouble, TestPyramid
- **Gerard Meszaros** — xUnit Test Patterns
- **Google** — Testing Blog, Software Engineering at Google
- **Steve Freeman & Nat Pryce** — Growing Object-Oriented Software, Guided by Tests

## License

MIT

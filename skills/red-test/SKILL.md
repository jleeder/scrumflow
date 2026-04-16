---
name: scrumflow-red-test
description: |
  ScrumFlow tool skill. Converts BDD scenarios (Given/When/Then) into failing test code for the project's detected test framework and style. Called by the Engineer agent.
allowed-tools: ["read", "edit", "search"]
---

# Red Test Skill

Convert approved BDD scenarios (Given/When/Then) into actual failing test code in the project's test framework and style.

## Purpose
This skill transforms BDD specification into executable test code that initially fails, establishing the red phase of the Red-Green-Refactor cycle.

## Input
- **BDD scenario text** – Given/When/Then formatted scenarios
- **Detected framework** – jest, pytest, rspec, vitest, mocha, or other
- **Task context** – What feature/module is being tested

## Output
- Test file(s) written to the appropriate test directory
- Tests are failing (red state)
- Structured output confirming test file creation and initial failure status

## Behaviour

### Framework Detection
1. Search project root for configuration files:
   - `jest.config.js`, `jest.config.ts`, `jest.config.mjs`
   - `pytest.ini`, `setup.cfg`, `pyproject.toml`
   - `.rspec`, `spec_helper.rb`
   - `vitest.config.ts`, `vitest.config.js`
   - `mocha.opts`, `.mocharc.json`
2. If multiple configs exist, infer from package.json `"test"` script or tsconfig target
3. If framework cannot be detected, report clearly and ask for manual specification

### Test File Naming and Location
1. Analyze existing test files to detect conventions:
   - JavaScript: `*.test.ts`, `*.test.js`, `*.spec.ts`
   - Python: `test_*.py`, `*_test.py`
   - Ruby: `*_spec.rb`
2. Place new test files in detected test directory:
   - JavaScript: `__tests__/`, `tests/`, `src/**/__tests__/`
   - Python: `tests/`, `test/`
   - Ruby: `spec/`
3. Name the test file matching the source file being tested, with convention suffix

### Test Style Detection
1. Read existing test files to identify patterns:
   - Jest/Vitest/Mocha: `describe()` blocks, `it()` or `test()` calls, `expect()` assertions
   - Pytest: `def test_*()` functions, `assert` statements
   - RSpec: `describe` blocks, `it` examples, `expect()` matchers
2. Match all imports, setup patterns, mock patterns, helper usage
3. If no existing tests found, use framework defaults

### Scenario to Test Mapping
1. Parse each BDD scenario:
   - Extract Given clauses → setup/arrange phase
   - Extract When clauses → action/act phase
   - Extract Then clauses → assertion/assert phase
2. Create one test case per scenario
3. Use scenario name as test description: `it('should: [scenario name]')`
4. Map Given steps to arrange phase (setup state, mocks, fixtures)
5. Map When steps to act phase (call function, trigger event, submit form)
6. Map Then steps to assertions (expect statements)

### Test Must Be Red
1. After writing test code, run test suite to confirm tests fail
2. If any test would pass without production code being written, flag it as a specification error
3. Report which tests failed and why (what assertion failed, what code is missing)
4. Do not proceed if tests are already passing

### Code Integration
1. Follow existing import patterns in the codebase:
   - If tests import from `@/utils`, use that pattern
   - If tests import modules absolutely, use that pattern
   - If tests use path aliases, use those
2. Follow existing helper usage:
   - If there's a test factory/builder, use it
   - If there's a setupBeforeEach, follow that pattern
   - If there's a custom render function, use it
3. Follow existing mock patterns:
   - If the codebase uses jest.mock(), use that
   - If it uses dependency injection, use that
   - If it uses sinon stubs, use that
4. Match variable naming conventions and test organization style

### Do Not Write Production Code
- Only write test code
- Test files should fail because the production code being tested doesn't exist or is incomplete
- If test implementation requires writing production code to pass, stop and ask instead

### Output Report
```
**Red Test Complete**
- Framework: [detected]
- Test file: [path/to/test/file.test.ts]
- Tests written: [count]
- Status: [count] failing ✓

**Failing Tests:**
- Scenario 1: [reason for failure]
- Scenario 2: [reason for failure]
```

## Error Handling
- If framework cannot be detected → Report clearly with available evidence
- If test naming convention is ambiguous → Ask for clarification
- If existing tests cannot be analyzed → Use framework defaults
- If test code would pass before production code exists → Flag as specification error

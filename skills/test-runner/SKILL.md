---
name: scrumflow-test-runner
description: |
  ScrumFlow tool skill. Executes the test suite and reports pass/fail status with failure details. Called by the Engineer agent to verify red and green phases.
allowed-tools: ["execute"]
---

# Test Runner Skill

Execute the project's test suite and report pass/fail results.

## Purpose
This skill runs the project's tests and provides structured output on test results, used during the Red-Green-Refactor cycle to verify test states.

## Input
- **command** – The exact shell command to run tests (e.g., `npm test -- src/auth.test.ts`). If provided, this bypasses all auto-detection and filtering.
- **Optional path filter** – Run only tests relevant to a specific task (e.g., `src/auth/**`)
- **Optional framework hint** – If auto-detection fails, the test framework name

## Output
- **Structured report** with:
  - Total tests run
  - Number passing
  - Number failing
  - Failure details (test names, assertion errors, stack traces)
  - Exit code (0 for all pass, non-zero for any failure)

## Behaviour

### Execution Priority
1. **Explicit command provided:** If the `command` input is present, execute it directly using the `execute` tool. Parse the output and return the results.
2. **Framework Detection (Fallback):** If no command is provided, proceed with framework detection.

### Framework Detection (Fallback)
1. Search project root for test runner configuration:
   - JavaScript/TypeScript:
     - `npm test` script in `package.json`
     - `jest.config.js`, `jest.config.ts`
     - `vitest.config.ts`, `vitest.config.js`
     - `mocha.opts`, `.mocharc.json`
   - Python:
     - `pytest.ini`, `setup.cfg`, `pyproject.toml` with pytest config
     - `tox.ini`
   - Ruby:
     - `.rspec`, `spec_helper.rb`
     - Presence of `Gemfile` with rspec
2. Infer test runner from package.json `"test"` script if available
3. If detection fails, report clearly with evidence and ask for framework specification
4. Store detected framework for this session to avoid re-detection

### Test Discovery and Filtering
1. **No filter provided** – Run entire test suite
2. **With path filter** – Run only tests matching the path:
   - Filter: `src/auth/**` → run only tests in `test/**/auth/**` or `src/**/auth/**/__tests__/**`
   - Filter: `services/UserService.ts` → run tests matching `UserService` in name
   - Detection: Infer test naming from framework convention (test_*.py, *.test.ts, *_spec.rb)
3. Do not run tests that are intentionally skipped (describe.skip, it.skip, pytest.mark.skip)
4. Report which tests are being run when filtering is used

### Execution
1. **JavaScript/TypeScript**:
   - Detect test script from package.json: `npm test`, `npm run test:unit`, `yarn test`, `pnpm test`
   - If custom test script detected, use that
   - Run with output that includes test names and failure details
   - Command: `npm test -- [--testPathPattern=pattern]` (or framework-specific equivalent)
2. **Python**:
   - Run `pytest` (if pytest.ini or pyproject.toml exists)
   - If filter provided: `pytest [path] -v`
   - If no filter: `pytest -v`
   - Capture verbose output with test names and assertion errors
3. **Ruby**:
   - Run `rspec` (if .rspec or spec_helper.rb exists)
   - If filter provided: `rspec [path]`
   - If no filter: `rspec`
   - Capture output with test names and failure details
4. Set environment variables for testing:
   - `NODE_ENV=test` (JavaScript)
   - `PYTHONPATH` set if needed (Python)
   - Suppress any pre-test hooks that would slow execution (skip DB migrations unless in test file)

### Result Parsing and Reporting
1. Parse test output to extract:
   - Total tests executed
   - Number passed (✓ or .)
   - Number failed (✗ or F)
   - Number skipped (if applicable)
   - Failure details: test name, assertion message, stack trace
2. Structure the report:
   ```
   **Test Results**
   Framework: [jest/pytest/rspec/vitest/mocha]
   Command: [actual command run]
   
   Summary:
   - Total: [n] tests
   - Passing: [n] ✓
   - Failing: [n] ✗
   - Skipped: [n]
   
   Status: [PASS] or [FAIL]
   
   **Failing Tests:**
   
   1. Test name: "should authenticate with valid credentials"
      Error: expect(result).toBe(true)
      Stack: src/auth.test.ts:45:12
      Message: Expected true, got undefined
   
   2. Test name: "should reject invalid token"
      Error: Error: InvalidToken not thrown
      Stack: src/token.test.ts:62:8
   ```
3. Include enough detail for developer to understand failures without re-running

### Performance Optimization
1. When filter is provided, run only filtered tests:
   - `npm test -- --testNamePattern="auth"` (faster than full suite)
   - `pytest src/auth/` (runs only directory)
   - `rspec spec/auth/` (runs only directory)
2. Skip setup tasks if appropriate:
   - Skip DB migrations that are per-test if running subset
   - Skip seed data unless in test file
3. Report filtering applied: "Running tests matching 'auth/**'"

### Edge Cases and Errors
1. **Framework cannot be detected**:
   - Report available evidence: "No jest.config found, no pytest.ini, no .rspec found"
   - List files in root: "Files: package.json, pyproject.toml"
   - Ask user to specify framework
2. **Test runner not installed**:
   - Message: "jest is not installed in node_modules"
   - Suggest: `npm install --save-dev jest`
3. **Syntax errors in tests**:
   - Report which test file has syntax error
   - Report the error message
   - Do not continue running other tests
4. **Filter matches no tests**:
   - Report: "No tests found matching pattern 'src/missing/**'"
   - Verify pattern is correct
5. **Timeout on slow tests**:
   - Report: "[n] tests passed, [n] tests timed out"
   - Suggest increasing timeout or identifying slow test

### Do Not Interpret Results
- This skill reports results only
- Do not make judgments like "tests are good now" or "this is acceptable"
- Do not modify code based on test results
- Pass judgment of test quality/coverage back to Engineer agent

### Output Format
Always return structured output that can be parsed:
```
FRAMEWORK: [detected framework]
COMMAND: [command executed]
EXIT_CODE: [0 or non-zero]
TOTAL: [number]
PASSED: [number]
FAILED: [number]
SKIPPED: [number]
STATUS: [PASS|FAIL]

[Detailed failure output or "All tests passed"]
```

## Error Handling
- If test runner not found → Report clearly with evidence and ask for framework
- If test syntax error → Report file and error, stop
- If timeout → Report which tests and suggest increase
- If filter matches no tests → Report pattern and ask for verification

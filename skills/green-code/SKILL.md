---
name: scrumflow-green-code
description: |
  ScrumFlow tool skill. Writes the minimal production code needed to make failing tests pass. Does not add features beyond what the tests require. Called by the Engineer agent.
allowed-tools: ["read", "edit", "search"]
---

# Green Code Skill

Write the minimum production code needed to make currently failing tests pass.

## Purpose
This skill implements the production code required to satisfy the test specifications, completing the Green phase of the Red-Green-Refactor cycle.

## Input
- **Failing test output** – Test failure messages and assertions that are not met
- **Task documentation** – Implementation guidance, file structure, patterns, abstractions
- **Test file(s)** – The tests that must pass (read-only)

## Output
- Production code written to appropriate source files
- All provided tests passing
- Structured report of files written and functions/classes created

## Behaviour

### Understanding Test Failures
1. Read the test output carefully:
   - What assertion failed?
   - What function/class/method is missing?
   - What error message does it give?
   - What input does it expect?
   - What output should it produce?
2. Do not skim — understand the exact contract the test expects
3. If test failure output is unclear, re-run test with verbose output

### Read Task Documentation
1. Consult the task-doc for:
   - Which file(s) should be created or modified
   - What abstractions or patterns to use
   - What edge cases to handle
   - Any performance or architectural constraints
2. Use task-doc as the source of truth for file locations and structure
3. If task-doc specifies `src/auth/TokenService.ts`, create that file in that location

### Minimum Code to Pass Tests
1. Write only the code necessary to make tests pass
2. Do not add:
   - Speculative features not tested
   - Error handling for cases not tested
   - Logging, metrics, or observability unless tested
   - Comment documentation unless tested
3. Follow the principle: **If a test doesn't require it, don't write it**
4. Example: If test expects a function that returns a boolean, return a boolean — don't add validation, caching, or retry logic

### Code Conventions and Patterns
1. Follow existing code in the repository:
   - Naming conventions: camelCase, PascalCase, snake_case (match existing)
   - Module structure: flat, nested, or barrel exports (match existing)
   - Function style: arrow functions or function declarations (match existing)
   - Class patterns: class fields, constructor initialization (match existing)
   - Error handling: exceptions, result objects, error codes (match existing)
2. Reuse existing utility functions, helpers, and libraries
3. Match the indentation, spacing, and formatting of surrounding code
4. Use the same import/export style as existing code

### Do Not Modify Tests
1. Test files are read-only from this skill's perspective
2. If a test seems wrong:
   - Flag it clearly
   - Do not attempt to "fix" the test
   - Explain the issue to the Engineer agent
   - Ask for clarification before proceeding
3. If test and task-doc contradict:
   - Flag the conflict explicitly
   - Show both statements
   - Ask which one is authoritative
   - Do not guess

### Detecting Required Files and Functions
1. From test imports, infer the module structure:
   - `import { authenticate } from '../auth'` → create `src/auth.ts` or `src/auth/index.ts`
   - `import UserService from '@/services/UserService'` → create `src/services/UserService.ts`
2. From test calls, infer function/class signatures:
   - `expect(fn(a, b)).toBe(result)` → function takes two parameters
   - `new User({ name: 'John' })` → constructor takes object with name property
   - `await service.save()` → method is async
3. From assertions, infer return types:
   - `expect(...).toBe(true)` → returns boolean
   - `expect(...).toEqual({ id: 1 })` → returns object with id property
   - `expect(...).rejects.toThrow()` → throws exception

### Implementation Steps
1. Create files in the correct locations (per task-doc)
2. Implement functions/classes/methods in the order they're called in tests
3. For each function:
   - Accept parameters as test expects
   - Return type as test expects
   - Handle happy path thoroughly
   - Handle unhappy paths only if tested
4. Run tests after each function to verify progress
5. Do not move to the next function until current tests pass

### Edge Cases and Error Handling
1. Only implement error handling for cases tested
2. If test expects an exception: implement the logic to throw it
3. If test expects a default value: implement logic to return it
4. If test doesn't specify error behavior: implement minimum viable behavior
5. Flag any edge cases from task-doc that aren't tested

### Output Report
```
**Green Code Complete**
- Files created: [count]
  - src/auth/TokenService.ts (authenticate function)
  - src/auth/TokenValidator.ts (validate function)
- Files modified: [count]
  - src/index.ts (exported new modules)
- Tests passing: [count]/[total] ✓

**Functions Implemented:**
- authenticate(credentials: Credentials): Promise<Token>
- validate(token: Token): boolean
```

## Error Handling
- If test failure is unclear → Verbose test output or ask for clarification
- If task-doc conflicts with test → Flag explicitly and ask which is authoritative
- If required file location is ambiguous → Ask before creating
- If implementation would require external dependencies not in package.json → Flag and ask
- If test expects different behavior than task-doc describes → Surface the conflict

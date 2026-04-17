---
name: scrumflow-commit-crafter
description: |
  ScrumFlow tool skill. Generates a well-structured atomic commit message and creates the git commit for a completed task. Called by the Engineer agent after green phase is verified.
allowed-tools: ["read", "execute"]
---

# Commit Crafter Skill

Generate a meaningful atomic commit message and create a git commit for a completed task.

## Purpose
This skill creates well-structured git commits that clearly communicate what changed and why, making the project history readable and useful for future debugging and understanding.

## Input
- **Task metadata**:
  - Task ID (e.g., `TASK-001`)
  - Story ID (e.g., `STORY-001`)
  - Task document (for understanding the WHY)
- **Current working state**:
  - Modified and new files in the repository
  - Git status/diff output showing what changed

## Output
- A single atomic git commit with a well-crafted message
- Commit message follows conventional commits format
- Commit includes only the relevant files (not `.scrum-flow/`)

## Behaviour

### Pre-commit Validation
1. Check git status:
   - List all changed files
   - Report if any untracked files exist that should be committed
   - Report if any unstaged changes exist
2. Verify no `.scrum-flow/` files are staged:
   - `.scrum-flow/` directory should never be committed to the repository
   - If found in staging area, unstage it: `git reset HEAD .scrum-flow/`
3. Verify the change set is cohesive:
   - All files should relate to the same feature/fix
   - If multiple unrelated changes exist, flag it and ask for clarification

### Read Task Documentation
1. Open the task document and read:
   - What feature/bug/refactoring is this task?
   - What is the user problem being solved?
   - What acceptance criteria were defined?
   - What edge cases or decisions were discussed?
2. Understand the WHY — the motivation and context
3. Note any key decisions or trade-offs made
4. Identify edge cases that were handled

### Analyze Git Diff
1. Run `git diff --staged` (or `git diff` if nothing is staged yet):
   - Understand what files changed
   - Understand what functions/classes/methods were added
   - Understand what existing code was modified
   - Understand what lines were removed
2. Identify the scope: which module/component is affected?
   - Examples: `auth`, `user-service`, `database`, `ui-form`
3. Identify the type of change:
   - `feat` – new feature
   - `fix` – bug fix
   - `test` – test-only changes (no production code)
   - `refactor` – restructuring without changing behavior
   - `chore` – build, dependencies, configuration
4. Summarize the change in one sentence

### Commit Message Format

#### Subject Line
- **Length**: Maximum 50 characters
- **Format**: `<type>(<scope>): <what changed>`
- **Types**: `feat`, `fix`, `test`, `refactor`, `chore`
- **Scope**: The module/component affected (lowercase, single word or hyphenated)
- **What changed**: Imperative mood, present tense, lowercase
- **Examples**:
  - `feat(auth): add password reset token model`
  - `fix(cart): prevent duplicate items in checkout`
  - `test(api): add integration tests for user endpoint`
  - `refactor(config): simplify environment variable loading`
  - `chore(deps): upgrade jest to v29`

#### Blank Line
- Separate subject from body with exactly one blank line
- This allows git log to format properly

#### Body
- **Length**: 2–4 sentences, wrapped at ~72 characters
- **Purpose**: Explain WHY the change was made, not WHAT was changed
- **Content**:
  - The story/issue this implements
  - Key decisions made during implementation
  - Edge cases or gotchas handled
  - Constraints or trade-offs
- **Tone**: Clear, concise, professional
- **Example**:
  ```
  Implements the token storage layer for the password reset flow (STORY-001).
  Tokens are single-use with 1-hour expiry, stored with a unique index to prevent
  reuse. Uses crypto.randomBytes for sufficient entropy without external dependencies.
  Old tokens for the same user are invalidated on new request.
  ```

#### Footer
- **Format**: `Implements: <STORY-ID> / <TASK-ID>`
- **Purpose**: Link the commit to the tracking system
- **Example**: `Implements: STORY-001 / TASK-001`

#### Full Example
```
feat(auth): add password reset token model

Implements the token storage layer for the password reset flow (STORY-001).
Tokens are single-use with 1-hour expiry, stored with a unique index to prevent
reuse. Uses crypto.randomBytes for sufficient entropy without external dependencies.
Old tokens for the same user are invalidated on new request.

Implements: STORY-001 / TASK-001
```

### What NOT to Include
- Avoid vague subject lines: ~~"Updated files"~~, ~~"Fix stuff"~~, ~~"WIP"~~
- Avoid repeating the WHAT in the body (the diff shows that)
- Avoid commit messages that are code comments
- Avoid listing every file changed — git shows that
- Avoid mentioning build/test/CI changes unless unusual

### Staging Files
1. Run `git add` for all changed production files:
   - Include all modified source files
   - Include all new source files
   - Include any fixture/config files modified
   - Include package.json/lock files if dependencies changed
2. Explicitly exclude `.scrum-flow/`:
   ```
   git reset HEAD .scrum-flow/
   ```
3. Verify staging with `git status`:
   - All intended files should be staged
   - No `.scrum-flow/` files should be staged
   - No unintended files should be staged
4. Report what is staged before committing

### Creating the Commit
1. Use `git commit -m` with the full message (subject + body + footer)
2. Format the message as a multi-line string (using here-doc or newlines)
3. Ensure the message is properly formatted
4. Example command:
   ```
   git commit -m "feat(auth): add password reset token model

   Implements the token storage layer for the password reset flow (STORY-001).
   Tokens are single-use with 1-hour expiry, stored with a unique index to prevent
   reuse. Uses crypto.randomBytes for sufficient entropy without external dependencies.
   Old tokens for the same user are invalidated on new request.

   Implements: STORY-001 / TASK-001"
   ```

### Post-commit Validation
1. Verify commit was created:
   ```
   git log -1 --format="%H %s"
   ```
2. Report successful commit with:
   - Commit hash (shortened)
   - Commit subject line
   - Files included
   - Task IDs linked

### Output Report
```
**Commit Created**
- Hash: abc1234
- Subject: feat(auth): add password reset token model
- Files: 2 changed
  - src/auth/TokenModel.ts (new)
  - src/auth/index.ts (modified)
- Linked: STORY-001 / TASK-001

Message:
feat(auth): add password reset token model

Implements the token storage layer for the password reset flow (STORY-001).
Tokens are single-use with 1-hour expiry, stored with a unique index to prevent
reuse. Uses crypto.randomBytes for sufficient entropy without external dependencies.
Old tokens for the same user are invalidated on new request.

Implements: STORY-001 / TASK-001
```

## Error Handling
- If multiple unrelated changes exist → Flag and ask for clarification
- If `.scrum-flow/` files are staged → Unstage them automatically
- If no files are staged → Report error and ask what to commit
- If task document is missing → Ask for task metadata before committing
- If commit message is unclear → Ask Engineer agent for clarification
- If git commands fail → Report error clearly

---
name: scrumflow-pr-writer
description: |
  ScrumFlow tool skill. Generates a pull request description from the feature branch's commit history and story artifacts. Called by the Orchestrator during post-review (Phase 6).
allowed-tools: ["read", "execute"]
---

# PR Writer Skill

Generate a PR description from the branch's unmerged commits and story artifacts.

## Purpose
This skill creates a comprehensive PR description that helps reviewers understand the context, implementation decisions, and acceptance criteria for a completed task or story.

## Input
- **Git history**:
  - `git log main..HEAD` output (commits on current branch not on main)
  - Commit subjects, bodies, and authors
- **Story artifacts**:
  - Story files from `.scrum-flow/stories/` directory
  - Story descriptions (user story format)
  - Acceptance criteria
  - BDD scenarios and specifications
  
## Output
- A complete PR description in markdown format
- Ready to paste into GitHub/GitLab/Bitbucket pull request form
- Includes What, Why, How, Acceptance Criteria, BDD Scenarios, and Notes sections

## Behaviour

### Extract Commit Information
1. Run `git log main..HEAD --format="%h %s%n%b"`:
   - Extract commit subjects (one-line summaries)
   - Extract commit bodies (detailed explanations of WHY)
   - Extract all commit messages in chronological order
2. Identify the type of each commit:
   - `feat(...)` → feature commit
   - `fix(...)` → bug fix commit
   - `test(...)` → test-only commit
   - `refactor(...)` → code restructuring
   - `chore(...)` → build/config/dependency commit
3. Collect all unique scopes mentioned (e.g., `auth`, `user-service`)
4. Identify the primary change type (is this mostly features, fixes, or refactoring?)

### Read Story Artifacts
1. Check `.scrum-flow/stories/` for story files:
   - Story JSON/YAML files containing user story format
   - Each story should have: title, description, acceptance criteria, scenarios
2. For each story found:
   - Extract the user story description (As a [user] I want [feature] so that [benefit])
   - Extract acceptance criteria (bulleted list of requirements)
   - Extract BDD scenarios (Given/When/Then format)
3. If no story files found, use commits as source of truth instead

### Derive "What"
- **Source**: Story file descriptions + commit subjects
- **Content**: 2-3 sentence summary of what this PR implements
- **Format**: Clear, user-facing description of features/fixes
- **Example**:
  ```
  Adds password reset functionality to the authentication system. Users can now
  request a password reset via email link, which generates a single-use token
  with 1-hour expiry. The reset form validates new password strength and updates
  the account securely.
  ```
- **Rules**:
  - Write from the user perspective, not implementation
  - Focus on the new capability or fix
  - Mention all major features included
  - Use simple, jargon-free language

### Derive "Why"
- **Source**: Story descriptions (the "so that" clause) + commit message bodies
- **Content**: The user problem being solved or business value delivered
- **Format**: 1-2 sentences explaining the motivation
- **Example**:
  ```
  Users frequently forget their passwords and lack a secure way to recover account
  access. This feature reduces support burden and improves user satisfaction by
  providing a self-service recovery flow.
  ```
- **Rules**:
  - Answer: "Why does the product need this?"
  - Include business value if available
  - Be concise but specific
  - Avoid technical jargon

### Derive "How"
- **Source**: Commit message bodies (they explain decisions made)
- **Content**: Key implementation decisions and technical approach
- **Format**: 2-3 bullet points or short sentences
- **Example**:
  ```
  - Uses crypto.randomBytes for secure token generation (no external dependencies)
  - Stores tokens with compound unique index on user_id + token_value
  - Implements token expiry via database index on created_at for cleanup
  - Sends reset emails asynchronously to avoid request timeouts
  ```
- **Rules**:
  - Extract from commit message bodies (they contain "why" decisions)
  - Focus on architectural/design decisions
  - Mention any trade-offs made
  - List constraints or dependencies handled
  - Keep technical level appropriate for reviewers

### Acceptance Criteria Section
- **Source**: Story artifacts or commit messages mentioning requirements
- **Format**: Bulleted checklist that reviewers can verify
- **Example**:
  ```
  - [ ] User can request password reset from login page
  - [ ] Reset email contains secure, single-use token link
  - [ ] Token expires after 1 hour
  - [ ] Reset form validates password strength (min 8 chars, mixed case)
  - [ ] Old tokens are invalidated when new reset is requested
  - [ ] All flows have passing test coverage (>95%)
  ```
- **Rules**:
  - Use checkbox format for easy review tracking
  - Each criterion should be testable/verifiable
  - Include all acceptance criteria from original story
  - Add test coverage if relevant
  - Order logically (user flow order is good)

### BDD Scenarios Section
- **Source**: Story artifacts containing Given/When/Then scenarios
- **Format**: List of scenario names (not full Given/When/Then, just names)
- **Example**:
  ```
  - User requests reset with valid email
  - User requests reset with non-existent email
  - User clicks reset link with expired token
  - User clicks reset link twice with same token
  - User sets new password below minimum length
  - User successfully resets and logs in with new password
  ```
- **Rules**:
  - List scenario names/titles only (they're documented in the story)
  - Mention that all scenarios have passing tests
  - Add note if any scenarios are known to be incomplete
  - Order logically by test flow

### Notes Section
- **Content**: Anything worth flagging to the reviewer
- **Examples**:
  - Gotchas or edge cases handled
  - Known limitations or future improvements
  - Dependencies on other PRs or issues
  - Breaking changes or migrations needed
  - Configuration or environment setup required
  - Performance implications
- **Format**: Bullet points or short paragraphs
- **Example**:
  ```
  - Token cleanup runs via database scheduled job (separate from this PR)
  - Password reset emails use existing email service (no new dependencies)
  - Database migration for tokens table is included but requires review
  - Rate limiting on reset requests is out of scope (STORY-042)
  ```

### PR Description Template

```
## What
[2-3 sentence summary of what this PR implements — derived from stories]

## Why
[The user problem being solved — from the "so that" clause of the stories]

## How
[Key implementation decisions — derived from commit messages]

## Acceptance Criteria
[Checklist from the original stories — so reviewer can verify]
- [ ] criterion 1
- [ ] criterion 2

## BDD Scenarios Covered
[List of scenario names from the BDD spec that are now implemented]

## Notes
[Any gotchas, follow-up work, or decisions worth flagging to the reviewer]
```

### Filling Each Section
1. **What**: Synthesize commit subjects + story descriptions
2. **Why**: Extract from story "so that" clauses or high-level story descriptions
3. **How**: Summarize key decisions from commit message bodies
4. **Acceptance Criteria**: Copy from story artifacts, format as checklist
5. **BDD Scenarios**: List scenario names from story files
6. **Notes**: Surface any gotchas, limitations, or follow-ups from commits or stories

### Do Not Include
- Links to internal tools (JIRA, etc.) unless context requires them
- Full commit hashes or detailed git archaeology
- Code snippets or diffs (GitHub shows those automatically)
- Speculation about future work (unless noted as follow-up)
- Apologies or defensive language
- Author names or email addresses

### Do Not Invent Information
- If story information is missing, note it: "No story artifact found; derived from commits"
- If BDD scenarios don't exist, omit the section
- If no notes are needed, omit the section
- Do not guess at business value if not documented
- Do not add acceptance criteria not in the original story

### Example Full PR Description
```
## What
Adds secure password reset functionality to the authentication system. Users can now
request a reset via email link, which generates a single-use token with 1-hour expiry.
The reset form validates password strength and updates the account securely.

## Why
Users frequently forget their passwords and lacked a secure way to recover account access.
This feature reduces support burden and improves user satisfaction by providing a
self-service recovery flow.

## How
- Uses crypto.randomBytes for secure token generation without external dependencies
- Stores tokens with compound index on user_id + token_value to prevent reuse
- Implements token expiry via database index for automatic cleanup
- Sends reset emails asynchronously to avoid request timeouts
- All code paths covered by BDD integration tests

## Acceptance Criteria
- [ ] User can request password reset from login page
- [ ] Reset email contains secure, single-use token link
- [ ] Token expires after 1 hour of creation
- [ ] Reset form validates password strength (min 8 chars, mixed case)
- [ ] Old tokens are invalidated when user requests new reset
- [ ] All BDD scenarios have passing test coverage

## BDD Scenarios Covered
- User requests reset with valid email
- User requests reset with non-existent email (no error leaked)
- User clicks reset link with expired token
- User clicks reset link twice with same token (second fails)
- User sets new password below minimum length
- User successfully resets and logs in with new password

## Notes
Token cleanup runs via database scheduled job (not in this PR). Password reset emails
use the existing email service (no new dependencies). Database migration for tokens
table is included and ready for production.
```

### Output Format
Return the complete PR description as markdown, ready to paste into the pull request form.

## Error Handling
- If no story artifacts found → Derive from commits and note: "Derived from commit messages (no story artifacts)"
- If commit messages lack context → Note what information is incomplete
- If BDD scenarios don't exist → Omit BDD section
- If acceptance criteria missing → Note: "No acceptance criteria documented"
- If multiple stories in one PR → Summarize all features and note: "Implements [story count] stories"

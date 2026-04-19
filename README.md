# ScrumFlow

A GitHub Copilot agentic plugin that orchestrates end-to-end feature development using a pipeline of specialized AI agents, enforcing software engineering best practices with human approval gates at every critical milestone.

## What It Does

ScrumFlow turns a raw feature idea into production-ready code by running it through a structured 6-phase pipeline:

```
Feature Idea
  ↓ Product Owner    → [Gate 1] Approve user story
  ↓ Test Author      → [Gate 2] Approve BDD spec
  ↓ Architect        → [Gate 3] Approve task plan
  ↓ Engineer(s)      → parallel TDD implementation
  ↓ Code Reviewer    → [Gate 4] Approve code review
  ↓ Push + PR
```

You stay in control. No phase runs without your explicit approval.

## Prerequisites

- VS Code with GitHub Copilot (agent mode enabled)
- A git repository with at least one commit
- A supported test framework (Jest, Vitest, Pytest, RSpec, Mocha, etc.)

## Installation

Install directly from GitHub with Copilot CLI:

```bash
copilot plugin install jleeder/scrumflow
```

Then verify installation:

```bash
copilot plugin list
```

This plugin manifest loads:
- Root orchestrator from `SKILL.md`
- Reusable tool skills from `skills/*/SKILL.md`
- Role-scoped agent skills from `agents/*/SKILL.md`

## Usage

Open GitHub Copilot chat in agent mode and describe what you want to build. ScrumFlow activates on natural triggers:

> "implement this feature"  
> "build this with ScrumFlow"  
> "start a new feature: \<description\>"  
> "run the pipeline on this"  
> "continue my ScrumFlow session"

From there, Copilot will guide you through each phase, pausing at each gate to show you the artifact and ask for approval before proceeding.

## Pipeline Phases

### Phase 1 — Product Owner (`scrumflow-product-owner`)
Converts your raw idea into a scoped user story. Surfaces ambiguity through structured questions before any code is written. Produces a `STORY-NNN.md` artifact for your review.

**Gate 1:** Approve or refine the story before BDD spec work begins.

### Phase 2 — Test Author (`scrumflow-test-author`)
Reads the approved story and writes a complete BDD specification in Given/When/Then format. No implementation details — just the behavioral contract.

**Gate 2:** Approve or refine the BDD spec before architecture work begins.

### Phase 3 — Architect (`scrumflow-architect`)
Reads the approved BDD spec and produces a complete technical plan: task decomposition, dependency mapping, parallel vs. sequential execution order, and rich per-task documentation that Engineer subagents will use.

**Gate 3:** Approve or refine the plan before implementation begins.

### Phase 4 — Engineer(s) (`scrumflow-engineer`)
One subagent per task, running in parallel in isolated git worktrees (managed in `.worktrees/`). Each Engineer follows a strict Red → Green → Commit TDD cycle:
1. Convert BDD scenarios to failing tests (`scrumflow-red-test`)
2. Run tests, confirm red state (`scrumflow-test-runner`)
3. Write minimum production code to pass (`scrumflow-green-code`)
4. Run tests, confirm green state (`scrumflow-test-runner`)
5. Craft an atomic commit (`scrumflow-commit-crafter`)

### Phase 5 — Code Reviewer (`scrumflow-code-reviewer`)
Reviews the full implementation against the approved story, BDD spec, and task-docs. Checks correctness, BDD coverage, security, edge cases, and test quality. Produces a structured `REVIEW-NNN.md` report.

**Gate 4:** Approve or request changes before the branch is pushed.

### Phase 6 — Push + PR
Merges worktrees, pushes the branch, and optionally generates a comprehensive pull request description (`scrumflow-pr-writer`). Ticket sync (`scrumflow-ticket-sync`) can post gate summaries to Jira, Linear, GitHub Issues, or Azure DevOps.

## State Directory

All pipeline state is tracked in `.scrum-flow/` inside your worktree (never committed):

```
.scrum-flow/
├── state.json          # Current phase, gate approvals, task status
├── stories/            # STORY-NNN.md
├── tests/              # BDD-NNN.md
├── tasks/              # TASK-NNN-docs.md + task list
├── reviews/            # REVIEW-NNN.md
└── drafts/             # Working files during active phases
```

Add `.scrum-flow/` to your `.gitignore`.

## Agents

| Agent | Skill ID | Purpose |
|---|---|---|
| Product Owner | `scrumflow-product-owner` | Story definition |
| Test Author | `scrumflow-test-author` | BDD specification |
| Architect | `scrumflow-architect` | Technical planning |
| Engineer | `scrumflow-engineer` | TDD implementation (parallel) |
| Code Reviewer | `scrumflow-code-reviewer` | Code review |

## Skills (Tools)

| Skill | Skill ID | Purpose |
|---|---|---|
| Red Test | `scrumflow-red-test` | Convert BDD scenarios to failing tests |
| Test Runner | `scrumflow-test-runner` | Execute test suite, report pass/fail |
| Green Code | `scrumflow-green-code` | Write minimum code to pass tests |
| Commit Crafter | `scrumflow-commit-crafter` | Generate atomic conventional commits |
| PR Writer | `scrumflow-pr-writer` | Generate pull request descriptions |
| Ticket Sync | `scrumflow-ticket-sync` | Sync gate approvals to issue trackers |

## Design Principles

- **Human-in-the-loop**: Agents cannot advance past a gate without explicit approval.
- **TDD by default**: Code is always written in response to failing tests.
- **Isolation**: Each Engineer runs in its own git worktree in `.worktrees/` to avoid conflicts.
- **Rich context transfer**: The Architect writes detailed task-docs so cheaper Engineer models have full context without needing to re-reason from scratch.
- **Model-cost optimization**: Premium models handle planning and review; cheaper models handle implementation and tooling.
- **Atomic commits**: One commit per task, linked to story and task IDs.
- **Non-blocking integrations**: Ticket sync failures never stop the pipeline.

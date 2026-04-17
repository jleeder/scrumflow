---
title: "[TASK DECOMPOSITION TITLE]"
id: TASKS-NNN
story_id: STORY-NNN
status: draft
created: YYYY-MM-DD
author: agent:architect
gate_approved_by: ""
gate_approved_at: ""
tags: [tasks]
---

# [[TASKS-NNN]] — [Slug Title]

Linked to [[STORY-NNN]]

## Task List

| Task ID | Name | Dependencies | Parallelism |
|---|---|---|---|
| TASK-NNN-1 | [Task description] | None | independent |
| TASK-NNN-2 | [Task description] | TASK-NNN-1 | depends-on-TASK-NNN-1 |
| TASK-NNN-3 | [Task description] | None | independent |
| TASK-NNN-4 | [Task description] | TASK-NNN-2, TASK-NNN-3 | depends-on-TASK-NNN-2, TASK-NNN-3 |

## Execution Plan

```
Phase 1 (Parallel)
├── TASK-NNN-1 [Implementation: core logic]
└── TASK-NNN-3 [Test setup: fixtures and helpers]
         ↓
Phase 2 (Sequential)
└── TASK-NNN-2 [Integration: wire up TASK-NNN-1 output]
         ↓
Phase 3 (Sequential)
└── TASK-NNN-4 [Validation: end-to-end test execution]
```

### Parallelism Summary
- **2 tasks** can run in parallel in Phase 1 (TASK-NNN-1 and TASK-NNN-3)
- **2 tasks** must run sequentially (TASK-NNN-2 depends on TASK-NNN-1; TASK-NNN-4 depends on both)
- **Estimated duration**: Phase 1 (concurrent) + Phase 2 + Phase 3

## Individual Task Documentation

> [!note]
> Each task listed above has its own detailed documentation file:
> - `TASK-NNN-1-<slug>-docs.md` — Implementation details, code patterns, acceptance criteria
> - `TASK-NNN-2-<slug>-docs.md` — Integration points, dependencies, acceptance criteria
> - `TASK-NNN-3-<slug>-docs.md` — Test fixtures, setup scripts, verification steps
> - `TASK-NNN-4-<slug>-docs.md` — End-to-end validation, acceptance criteria
>
> These files live in `.scrum-flow/tasks/` alongside this decomposition file. Each is a standalone artifact with its own frontmatter and acceptance criteria.

> [!warning]
> Use Gate #3 approval (Architect) to validate task breakdown, dependencies, and parallelism before engineering begins. Ensure no task has circular dependencies and that parallelism estimates are realistic.

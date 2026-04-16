---
title: "[BDD SPEC TITLE]"
id: BDD-NNN
story_id: STORY-NNN
status: draft
created: YYYY-MM-DD
author: agent:test-author
gate_approved_by: ""
gate_approved_at: ""
tags: [bdd-spec]
---

# [[BDD-NNN]] — [Slug Title]

## Feature: [Feature Name]

Linked to [[STORY-NNN]]

### Background
```gherkin
Given [precondition]
And [precondition]
```

### Scenario: [Scenario Title]
```gherkin
Given [initial state]
When [action taken]
Then [expected outcome]
And [expected outcome]
```

### Scenario: [Scenario Title]
```gherkin
Given [initial state]
When [action taken]
Then [expected outcome]
```

### Scenario: [Edge Case or Error Condition]
```gherkin
Given [initial state]
When [action taken]
Then [expected outcome]
And [expected outcome]
```

## Test Implementation Notes

- **Framework**: [Cucumber/Behave/pytest-bdd/other]
- **Location**: `.scrum-flow/tests/BDD-NNN-<slug>/`
- **CI Status**: [Will be linked post-implementation]

> [!note]
> This is a template for BDD specifications. Replace all bracketed placeholders with concrete Gherkin syntax. Each scenario must be testable and independently verifiable. Use Gate #2 approval (Test Author) to validate coverage and clarity before engineering implementation.

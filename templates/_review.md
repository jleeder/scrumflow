---
title: "[CODE REVIEW TITLE]"
id: REVIEW-NNN
story_id: STORY-NNN
status: draft
recommendation: ""
created: YYYY-MM-DD
author: agent:code-reviewer
gate_approved_by: ""
gate_approved_at: ""
tags: [review]
---

# [[REVIEW-NNN]] — [Slug Title]

Linked to [[STORY-NNN]]

## Summary

[2–3 sentence verdict on implementation quality, whether acceptance criteria are met, and overall readiness for merge.]

Example:
> Implementation satisfies all three acceptance criteria from STORY-NNN. Code is well-structured and test coverage is comprehensive. Ready to merge with noted minor suggestions for documentation.

## Architectural Compliance

| Dimension | Met? | Notes |
|---|:---:|---|
| File Structure | ✓ or ✗ | Adherence to specified paths |
| Patterns & Abstractions | ✓ or ✗ | Usage of mandated design patterns |
| Technical Constraints | ✓ or ✗ | Adherence to library/performance constraints |

[Notes on deviation from Architect's task-docs, if any]

## Per-Story Assessment

### [[STORY-NNN]] — [Story Title]

| Criterion | Met? | Notes |
|---|:---:|---|
| Criterion 1 | ✓ or ✗ | Implementation detail or concern |
| Criterion 2 | ✓ or ✗ | Implementation detail or concern |
| Criterion 3 | ✓ or ✗ | Implementation detail or concern |

## Findings

### 1. **[Finding Title]** — Severity: Critical
- **Location**: `src/module/file.py`, lines 42–58
- **Description**: [What was found, why it matters]
- **Suggestion**: [How to fix it]

### 2. **[Finding Title]** — Severity: Major
- **Location**: `src/api/endpoint.js`, function `handleRequest()`
- **Description**: [What was found, why it matters]
- **Suggestion**: [How to fix it]

### 3. **[Finding Title]** — Severity: Minor
- **Location**: `tests/integration/test_flow.py`, line 127
- **Description**: [What was found, why it matters]
- **Suggestion**: [How to fix it]

## Test Assessment

### Test Suite Quality

| Dimension | Assessment | Notes |
|---|---|---|
| Coverage | [Good/Fair/Poor] | [% coverage, gaps if any] |
| BDD Alignment | [Good/Fair/Poor] | [Do tests verify all Gherkin scenarios?] |
| Edge Cases | [Good/Fair/Poor] | [Are edge cases and error paths tested?] |
| Performance | [Good/Fair/Poor] | [Any slow tests or timeout risks?] |

### BDD Scenario Verification

- **Scenarios defined**: [Count from BDD-NNN]
- **Scenarios passing**: [Count]
- **Scenarios failing**: [Count and list]
- **Coverage gap**: [Any scenarios untested or uncovered by implementation?]

## Recommendation

### **Status**: [approve | approve-with-notes | needs-work]

**Recommendation**: **[APPROVE | APPROVE WITH NOTES | NEEDS WORK]**

**Rationale**:
[Clear explanation of the recommendation. If approve-with-notes or needs-work, explain what must be addressed before merge.]

Example for approve-with-notes:
> Implementation is solid and meets all acceptance criteria. The three findings above are minor and can be addressed in follow-up PRs without blocking merge. Priority: Fix finding #1 (documentation) in next release.

Example for needs-work:
> Critical finding #1 must be fixed before merge. Resubmit for re-review after addressing the database performance issue.

> [!note]
> This is a template for code review reports. Replace all bracketed placeholders with concrete findings, assessments, and a clear recommendation. Gate #4 (Code Reviewer) uses this report to make the final merge decision.

---
name: scrumflow-ticket-sync
description: |
  ScrumFlow tool skill. Pushes structured artifact summaries to external ticket trackers (Jira, Linear, GitHub Issues) via MCP. Optional; called by the Orchestrator at gate approvals.
allowed-tools: ["read"]
---

# Ticket Sync

**Version**: 1.0  
**Type**: Stateless utility skill  
**Called by**: Orchestrator (opt-in, at Gates #1, #2, #3, and post-review)

---

## Purpose

Push structured artifact summaries to external ticket trackers (Jira, Linear, GitHub Issues, etc.) via MCP. Enables the pipeline to keep upstream issue tracking systems synchronized with ScrumFlow decision gates without blocking the pipeline on sync failures.

---

## Trigger Points

1. **Gate #1 (Product Owner approval)**: Story approved
2. **Gate #2 (Test Author approval)**: BDD specs approved
3. **Gate #3 (Architect approval)**: Task decomposition approved
4. **Post-review (optional)**: PR opened and CI passing

---

## Input

```
{
  "artifact_path": "/path/to/.scrum-flow/stories/STORY-001 — example-slug.md",
  "gate_context": {
    "gate_number": 1,
    "status": "approved",
    "approved_by": "product-owner",
    "approved_at": "2026-04-09T14:30:00Z"
  },
  "mcp_config": {
    "provider": "jira",  // or "linear", "github", "azure-devops"
    "ticket_id": "PROJ-123",
    "api_endpoint": "https://...",
    "credentials": { ... }  // Provided by MCP connection
  }
}
```

---

## Behavior

### 1. Check MCP Availability
- Determine if a ticket sync MCP is connected and enabled
- If no MCP connection exists: skip silently and return `{ "skipped": true, "reason": "no_mcp_available" }`
- If MCP is unavailable: log a warning and continue without blocking

### 2. Read and Parse Artifact
- Read the artifact file at `artifact_path`
- Extract title, ID, summary, and gate-relevant metadata from YAML frontmatter and body
- Handle all artifact types: story, BDD spec, task list, or review

### 3. Generate Gate-Specific Comment

#### Gate #1 (Story Approved)
```
✅ Story Approved

[STORY-001 — Feature slug]

User Story: As a [role], I want [capability], so that [benefit]

Acceptance Criteria (3):
- Criterion 1
- Criterion 2
- Criterion 3

Status: Approved by Product Owner
Timestamp: 2026-04-09 14:30 UTC

[Link to ScrumFlow artifact]
```

#### Gate #2 (BDD Spec Approved)
```
✅ BDD Specifications Approved

[BDD-001 — Feature slug] → [STORY-001]

Scenario Count: 4 scenarios defined for [Feature Name]

- Scenario: [Scenario Title]
- Scenario: [Scenario Title]
- Scenario: [Scenario Title]
- Scenario: [Edge Case]

Status: Approved by Test Author
Timestamp: 2026-04-09 15:00 UTC

[Link to ScrumFlow artifact]
```

#### Gate #3 (Tasks Approved)
```
✅ Technical Plan Approved

[TASKS-001 — Architecture slug] → [STORY-001]

Task Decomposition: 4 tasks, 2 can run in parallel

Parallel Phase: TASK-001, TASK-003
Sequential: TASK-002 → TASK-004

Estimated Duration: [based on task docs]

Status: Approved by Architect
Timestamp: 2026-04-09 15:45 UTC

[Link to ScrumFlow artifact]
```

#### Post-Review (PR Opened)
```
✅ PR Opened & All BDD Scenarios Passing

[REVIEW-001 — Code Review] → [STORY-001]

Pull Request: [PR link]
Recommendation: Approved

Test Results:
- BDD Scenarios: 4/4 passing
- Unit Test Coverage: 92%
- CI Status: ✓ All checks passing

Status: Ready to merge
Timestamp: 2026-04-09 16:20 UTC

[Link to ScrumFlow artifact]
```

### 4. Post Comment to Ticket
- Use the MCP connection to post the structured comment to the linked ticket
- Format message appropriately for the target system (Jira, Linear, GitHub, etc.)
- Include markdown formatting where supported
- Add a footer link back to the ScrumFlow artifact

### 5. Error Handling
- **MCP unavailable**: Skip silently, log a non-blocking warning
- **Authentication fails**: Return error status but do not block pipeline
- **Ticket not found**: Log error; suggest checking ticket ID in ScrumFlow config
- **Network timeout**: Retry once after 2 seconds; if still fails, skip and log
- **All failures**: Never block the pipeline; always allow engineering work to continue

---

## Output

```
{
  "success": true,
  "posted_to": "Jira",
  "ticket_id": "PROJ-123",
  "comment_id": "12345",
  "gate": 1,
  "artifact_id": "STORY-001",
  "timestamp": "2026-04-09T14:35:00Z"
}
```

### Error Response (non-blocking)
```
{
  "success": false,
  "error": "mcp_unavailable",
  "message": "No ticket sync MCP connected; continuing without sync",
  "gate": 1,
  "artifact_id": "STORY-001",
  "timestamp": "2026-04-09T14:35:00Z"
}
```

---

## Implementation Details

### Supported Ticket Systems
- **Jira Cloud**: Uses REST API v3 (`POST /rest/api/3/issue/{issueIdOrKey}/comments`)
- **Linear**: Uses GraphQL API (`createComment` mutation)
- **GitHub Issues**: Uses REST API (`POST /repos/{owner}/{repo}/issues/{issue_number}/comments`)
- **Azure DevOps**: Uses REST API (`POST /_apis/wit/workitems/{id}/comments`)

### Templating Strategy
- Use artifact YAML frontmatter to auto-populate gate metadata
- Extract body sections intelligently (User Story, Acceptance Criteria, etc.)
- Truncate long content if needed (Jira comment max: 32,767 chars; Linear: 4,000 chars)
- Always include a direct link to the artifact in `.scrum-flow/`

### Idempotency
- Check if a comment already exists for this artifact + gate combination before posting
- Use artifact ID + gate number as a deduplication key
- If duplicate detected: return success without posting again

---

## Invocation (Orchestrator)

```python
# Pseudocode
orchestrator.run_skill("ticket-sync", {
  "artifact_path": story_file,
  "gate_context": { "gate_number": 1, "status": "approved", ... },
  "mcp_config": mcp_connection_details
})
```

---

## Notes

- **Non-blocking by design**: If ticket sync fails, the pipeline continues. Sync is a convenience, not a blocker.
- **Stateless**: Each invocation is independent; no state stored between calls.
- **Opt-in**: Only runs if an MCP connection is explicitly provided by the Orchestrator.
- **Audit trail**: All posts are timestamped and linked back to ScrumFlow artifacts for traceability.

---

## Example Use Case

1. Product Owner approves STORY-001 (Gate #1)
2. Orchestrator calls ticket-sync with story artifact + Jira config
3. Skill reads STORY-001.md, extracts title and acceptance criteria
4. Generates a Gate #1 comment summarizing the approval
5. Posts comment to Jira ticket PROJ-123
6. Returns success; Orchestrator logs the sync and continues to next gate
7. If Jira is unreachable, skill logs a warning and returns gracefully; the pipeline is unaffected

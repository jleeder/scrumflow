# AGENTS.md — ScrumFlow Development Guide

This file governs how AI agents (GitHub Copilot and others) should contribute to ScrumFlow itself. ScrumFlow is a GitHub Copilot agentic plugin. All development on it must be consistent with the principles of designing well-structured, human-in-the-loop agentic workflows for GitHub Copilot.

---

## What "Agentic Plugin" Means Here

ScrumFlow is not a VS Code extension. It is a collection of Copilot **instruction files** (`SKILL.md`) that teach GitHub Copilot specialized roles and behaviors. These files are discovered by Copilot at runtime and invoked by name as chat participants or tool skills.

Copilot reads them as grounded context. They must be:
- Unambiguous in their role scope
- Explicit about input/output contracts
- Clear about when to pause for human approval
- Honest about what they cannot do

---

## Repository Structure

```
SKILL.md                        # Top-level orchestrator instructions
agents/
  <role>.agent.md               # One agent per discrete pipeline role
skills/
  setup/SKILL.md                # Project initialization and configuration
  <tool>/SKILL.md               # One skill per atomic reusable capability
templates/
  _<artifact>.md                # Output templates referenced by agents/skills
```

### Rules

- **One role per agent.** Agents must not cross boundaries (e.g., the Architect must not write code; the Engineer must not write stories).
- **One capability per skill.** Skills are composable tools. They do one thing and return structured output.
- **Templates are contracts.** Every agent that produces a persistent artifact must use the corresponding template in `templates/`. Do not change a template without updating all agents that reference it.
- **No orphaned files.** Every `SKILL.md` must be referenced either from the top-level `SKILL.md` (orchestrator) or from an agent that invokes it as a tool.

---

## Writing Agent Instructions (`agents/*.agent.md`)

### Required Sections

Each agent `.agent.md` must contain:

0. **Initialization & Discovery** — The agent MUST begin its execution by:
   - **Announcing identity:** "I am the [Role] for this ScrumFlow phase."
   - **Scanning for local context:** Searching the project workspace (e.g., `.agents/skills/`, `.github/copilot-instructions.md`, or specific `docs/` folders) for project-specific skills, guidelines, or templates relevant to its domain.
   - **Announcing discovery:** "I have discovered and will be leveraging the following local skills/context: [List]."
1. **Role identity** — A single sentence stating what this agent is and is not responsible for.
2. **Inputs** — Explicit list of what the agent reads (state files, artifacts, user input).
3. **Outputs** — Explicit list of what the agent writes and where (`.scrum-flow/<subdir>/`).
4. **Gate behavior** — Whether this agent produces a gate artifact and what approval looks like.
5. **Failure modes** — What the agent does when it cannot proceed (always: surface the blocker to the human, never silently skip).
6. **Tool invocations** — Which skills this agent calls, in what order, and under what conditions.

### Gate Design

Gates are the core safety mechanism. When writing an agent that owns a gate:

- **Never auto-approve.** The agent must present the artifact and wait for an explicit `yes`, `approve`, or equivalent from the human before proceeding.
- **Make the artifact reviewable.** Output must be in a readable format (Markdown), not buried in code blocks or JSON.
- **Support revision loops.** Accept feedback and re-draft; do not force the human to restart the phase.
- **Record the approval.** Write the gate outcome to `.scrum-flow/state.json` so downstream agents know the gate passed.

### Model Guidance

Document the recommended model tier in each agent's `SKILL.md`:

| Tier | Use for |
|---|---|
| Premium (e.g., Claude Sonnet, GPT-4o) | Product Owner, Architect, Code Reviewer — reasoning-heavy phases |
| Standard (e.g., GPT-4.1, Claude Haiku) | Engineer, skill tools — execution-heavy phases |

Do not hard-code model names. Use tier labels so the config can be updated without touching agent instructions.

---

## Writing Skill Instructions (`skills/*/SKILL.md`)

Skills are stateless tools. They receive an input, perform one action, and return structured output. They must not:

- Maintain their own state files
- Make gate decisions
- Call other skills (skills are leaves, not orchestrators)
- Produce side effects outside of their defined output contract

Each skill `SKILL.md` must specify:

1. **Purpose** — One sentence.
2. **Input contract** — Exact fields/files/values the skill expects.
3. **Output contract** — Exact fields/files/values the skill returns.
4. **Error output** — A defined error shape when the skill cannot complete.
5. **Auto-detection logic** — If the skill adapts to the environment (e.g., test framework detection), document the detection strategy.

---

## State and Artifact Conventions

All pipeline state lives in `.scrum-flow/` and is **never committed to git**.

| Path | Owner | Format |
|---|---|---|
| `.scrum-flow/state.json` | Orchestrator | JSON — current phase, gate statuses, task statuses |
| `.scrum-flow/config.json` | User / setup | JSON — model tiers, git settings, MCP endpoint config |
| `.scrum-flow/stories/STORY-NNN.md` | Product Owner | Markdown using `templates/_story.md` |
| `.scrum-flow/tests/BDD-NNN.md` | Test Author | Markdown using `templates/_bdd-spec.md` |
| `.scrum-flow/tasks/TASK-NNN-docs.md` | Architect | Markdown using `templates/_tasks.md` |
| `.scrum-flow/reviews/REVIEW-NNN.md` | Code Reviewer | Markdown using `templates/_review.md` |
| `.scrum-flow/drafts/` | Any agent | Ephemeral working files, cleared between phases |

**Numbering:** IDs are zero-padded three-digit integers (`001`, `002`, …). The orchestrator assigns IDs sequentially at the start of each phase.

---

## Parallel Execution (Engineer Phase)

When adding features that touch the Engineer phase or task dispatch:

- Each Engineer subagent runs in an isolated git worktree. Never have two Engineers share a worktree.
- Task dependency order comes from the Architect's task-doc. Respect it.
- An Engineer may only begin once its upstream dependencies are in `completed` state in `state.json`.
- Engineers do not communicate with each other. Context is passed only through task-docs and the shared git history.

---

## Copilot-Specific Design Rules

These rules apply specifically because this plugin runs inside GitHub Copilot's agent infrastructure:

1. **Context window discipline.** Agent instructions must be concise. Copilot has a finite context window. Avoid narrative padding; every sentence in a `SKILL.md` must earn its place.

2. **Skill IDs must be stable.** Once a skill ID (e.g., `scrumflow-red-test`) is used by an agent, renaming it is a breaking change. Treat skill IDs as public API.

3. **No hallucination-prone open-ended tasks.** Every agent instruction must specify exactly what to produce, not leave it to the model's judgment. Vague instructions produce inconsistent output.

4. **Fail loudly, not silently.** If an agent cannot complete its phase (missing artifact, bad state, ambiguous input), it must tell the human what is missing and stop. Never guess past a blocker.

5. **Tools over prose for structured data.** When an agent needs to write state, use tool calls to write JSON. Do not encode state in prose messages that could be lost.

6. **Idempotency.** Re-running an agent on an already-completed phase must be safe. Agents should detect existing artifacts and ask before overwriting.

---

## Adding a New Agent

1. Create `agents/<role>.agent.md` following the required sections above.
2. Assign a stable skill ID following the `scrumflow-<role>` naming convention.
3. Register the agent in the top-level `scrumflow.agent.md` orchestrator — add it to the pipeline phase table and any relevant trigger conditions.
4. Add a template to `templates/` if the agent produces a new artifact type.
5. Document the gate behavior (or explicitly note there is no gate for this agent).
6. Update `README.md` to include the new agent in the agents table.

## Adding a New Skill

1. Create `skills/<capability>/SKILL.md` following the skill requirements above.
2. Assign a stable skill ID following the `scrumflow-<capability>` naming convention.
3. Document which agent(s) invoke this skill in both the skill file and the calling agent's file.
4. Update `README.md` to include the new skill in the skills table.

---

## Changing Existing Behavior

- **Changing a gate:** Any change to gate approval criteria or artifact format must be reviewed against all downstream agents that consume that artifact.
- **Changing a template:** Updating a template is a contract change. Bump the template version in the file header and update all agents that reference it.
- **Changing state.json shape:** Treat `state.json` fields as a versioned schema. Document the field in this file and ensure the orchestrator and all agents that read/write it are updated together.

---

## What Not to Do

- Do not add features to ScrumFlow by modifying only one agent in isolation — trace the full pipeline impact.
- Do not bypass gates in agent instructions "for speed" or "for simple cases."
- Do not store secrets or credentials in `.scrum-flow/config.json` — only endpoint URLs and non-sensitive configuration.
- Do not let an agent write to another agent's output directory (e.g., an Engineer must not write to `stories/`).
- Do not commit `.scrum-flow/` contents. Ever.

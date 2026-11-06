---
name: mflow
description: Use mFlow boards, tasks, Knowledge Base/ADRs, and the mFlow coding agent from Hermes. Use whenever the user asks to manage mFlow projects/tasks, read or write Knowledge Base docs (including ADRs), delegate coding work, or semantically search mFlow context.
license: MIT
compatibility: Requires the mflow MCP server bundled with this plugin (streamable HTTP at https://mcp.m-flow.io/mcp) and an authenticated mFlow account.
---

# mFlow

Work with mFlow through the bundled `mflow` MCP server. All tool names are
`snake_case`. Most tools require **IDs, not names** — resolve names first.

## Orientation

1. `list_standalone_projects` — list all projects the user can access.
2. `get_standalone_project` — fetch one project. Statuses and labels are
   returned as objects with **ids**; custom columns have **UUID ids**.
3. When a user refers to a project by name, resolve it to its id first. If
   ambiguous, list and ask. Pass the project id to every subsequent call.

## Tasks

- `get_standalone_task_by_key` — fetch a task by its readable key, e.g.
  `MVPB-123`. Prefer this when the user gives a key.
- `list_standalone_tasks` — list/filter tasks for a project.
- `create_standalone_tasks` — create tasks, including breakdowns. Max **50
  tasks per call**. Nest subtasks with `localRef`/`parentRef`. Validation is
  **all-or-nothing**: if any task is invalid, no tasks are created.
- `update_standalone_task` — update fields. `labelIds` is a **full replace**:
  always send the complete desired label list, not just the labels being added.
- `create_standalone_comment` — add a comment/note to a task.
- Confirm with the user before any `delete_*` action.

## Knowledge Base

- `list_knowledge_docs` — list docs. Takes a **project id** or the literal
  `"__global__"` for global docs. Types: `convention`, `workflow`, `metric`,
  `guide`, `adr`.
- **Always list before create.** If a doc already exists, use
  `update_knowledge_doc` to update it instead of creating a duplicate.
- ADR content MUST contain these three sections:
  `## Context`, `## Decision`, `## Consequences`.
- Keep new ADRs `proposed` until the user confirms them `accepted`. Accepted
  ADRs are injected into agent context and synced to the repo `docs/adr/`.
- **Never delete accepted ADRs.** To change a decision, supersede it: set
  `adrStatus: superseded` and point `supersededBy` at the replacement ADR.
- Confirm before `delete_*` or `transfer_knowledge_base` with `mode: move`.

## Coding agent delegation

- `assign_to_coding_agent` — delegate coding work to mFlow's coding agent.
  Required: `taskKey` or `taskId`. Optional:
  - `tier`: `light` (2GB / 1CPU), `mid` (default), `heavy` (7GB / 4CPU)
  - `repoUrl`: repo to clone, if any
  - `noRepoOutput`: `zip` or `new_repo`
  - `extraPrompt`: extra instructions for the agent
  - `aiProvider`: which AI provider to use
- The call **returns immediately**. Poll `get_coding_agent_status`
  afterwards — **not in a tight loop**. While queued, `queuePosition` tells
  the position in the queue.
- `noRepoOutput: new_repo` requires GitHub to be connected.
- A stuck `queued` status usually means the sandboxes are busy; wait and
  re-check later.

## Semantic search

- Use `ask_mflow` **only as a last resort**. It is slow and may change data.
  Until a dedicated `search_knowledge` tool ships, it is the only semantic
  search across mFlow.

## Pitfalls

- **401 on any tool** → the user needs to re-authenticate
  (`hermes mcp login mflow`).
- **API needs ids, not names** → resolve names to ids first.
- **`labelIds` is a full replace** → send the complete label list.
- **Confirm destructive actions** before `delete_*` and
  `transfer_knowledge_base mode: move`.

## Recipes

See `references/recipes.md` for copy-paste prompts: board triage, spec → task
breakdown, ADR compliance review, recording an ADR, coding-agent handoff, and
a daily stand-up digest.

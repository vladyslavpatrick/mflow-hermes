# mFlow MCP tools reference

Tool names are `snake_case` and are exposed by the `mflow` MCP server.
Most tools need **ids, not names**; resolve names with
`list_standalone_projects` / `get_standalone_project` first.

| Tool | Purpose | Notes |
| --- | --- | --- |
| `list_standalone_projects` | List accessible projects | First call for orientation |
| `get_standalone_project` | Get one project | Statuses/labels have ids; custom columns have UUID ids |
| `get_standalone_task_by_key` | Get task by readable key | e.g. `MVPB-123` |
| `list_standalone_tasks` | List/filter tasks | Requires project id |
| `create_standalone_tasks` | Create tasks/breakdowns | Max 50/call; nest with `localRef`/`parentRef`; all-or-nothing validation |
| `update_standalone_task` | Update a task | `labelIds` is a full replace |
| `create_standalone_comment` | Comment on a task | |
| `list_knowledge_docs` | List Knowledge Base docs | Project id or `"__global__"`; types: `convention`, `workflow`, `metric`, `guide`, `adr` |
| `update_knowledge_doc` | Update an existing doc | Prefer over creating duplicates |
| `assign_to_coding_agent` | Delegate to mFlow coding agent | `taskKey`/`taskId`, `tier`, `repoUrl`, `noRepoOutput`, `extraPrompt`, `aiProvider`; returns immediately |
| `get_coding_agent_status` | Poll coding agent status | Do not poll in a tight loop; `queuePosition` while queued |
| `ask_mflow` | Semantic search / agentic query | Last resort: slow, may change data |
| `delete_*` | Destructive deletes | Confirm with user first |
| `transfer_knowledge_base` | Transfer KB | Confirm before `mode: move` |

Tier sizes for `assign_to_coding_agent`:

| Tier | Resources |
| --- | --- |
| `light` | 2GB / 1CPU |
| `mid` | default |
| `heavy` | 7GB / 4CPU |

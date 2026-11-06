# mFlow for Hermes Agent

Use [mFlow](https://m-flow.io) from Nous Research's **Hermes Agent**: manage
mFlow boards and tasks, read/write the Knowledge Base (including ADRs), and
delegate coding work to mFlow's coding agent.

This is a **portable Agent Plugins v1** package. It contains no NestJS/NATS
code — it points Hermes at the hosted mFlow MCP server
(`https://mcp.m-flow.io/mcp`) and ships a skill that teaches Hermes how to use
it.

## Repository layout

```
plugin.json
mcp.json
skills/mflow/SKILL.md
skills/mflow/references/recipes.md
skills/mflow/references/tools.md
examples/config.oauth.yaml
README.md
LICENSE
```

`plugin.json` must remain at the repo root so
`hermes plugins install owner/repo` works.

## Install

```bash
hermes plugins install vladyslavpatrick/mflow-hermes --enable
```

Portable plugins are disabled by default unless installed with `--enable`. If
you installed without it:

```bash
hermes plugins enable mflow
```

Then choose an authentication method below.

## Authentication

### Option A — OAuth (works today)

Portable packages cannot declare `auth: oauth` in `mcp.json`, so OAuth uses a
same-named **native** entry in `~/.hermes/config.yaml`. Native config
overrides the package's server entry.

Add this to `~/.hermes/config.yaml` (see
[`examples/config.oauth.yaml`](examples/config.oauth.yaml)):

```yaml
mcp_servers:
  mflow:
    url: "https://mcp.m-flow.io/mcp"
    auth: oauth
    timeout: 120
    # tools: { exclude: ["delete_*", "transfer_knowledge_base"] }   # optional
    # trust: untrusted   # optional: approve every write
```

Then authenticate once:

```bash
hermes mcp login mflow
```

mFlow OAuth is authorization_code + PKCE with a Google sign-in consent page.
Access tokens last **90 days** and there is **no refresh grant** yet, so
re-run `hermes mcp login mflow` every 90 days. For headless boxes, follow the
Hermes **"OAuth over SSH"** guide.

### Option B — `MFLOW_TOKEN` (for headless Hermes: gateway/cron/VPS)

Once mFlow Personal Access Tokens (PATs) are available, put a PAT in Hermes'
env file and the bundled `mcp.json` works with zero native config:

```bash
# ~/.hermes/.env
MFLOW_TOKEN=mf_pat_xxxxxxxxxxxxxxxx
```

`${MFLOW_TOKEN}` in `mcp.json` headers is interpolated from the user's env or
`~/.hermes/.env`. If it is unset, Hermes fails closed and names the variable.

| Variable | Required | Description |
| --- | --- | --- |
| `MFLOW_TOKEN` | Only for Option B | mFlow Personal Access Token used as a Bearer token. Never commit it. |

> **Status:** PAT creation/listing/revocation and mcp-server PAT verification
> are backend work in the mFlow monorepo. Until it ships, use Option A.

## Verify

```bash
# Confirm the mflow server is registered
hermes mcp list

# In an interactive session, reload MCP servers
/reload-mcp

# Smoke test
hermes chat -q "List my mFlow projects"
```

You can also ask Hermes to use the skill explicitly:

```text
Use the mflow skill to list my mFlow projects and their task counts.
```

## What the skill covers

- **Orientation:** `list_standalone_projects` → `get_standalone_project`.
  mFlow APIs need **ids, not names**; custom columns have UUID ids.
- **Tasks:** `get_standalone_task_by_key`, `list_standalone_tasks`,
  `create_standalone_tasks` (max 50 per call, nesting via
  `localRef`/`parentRef`, all-or-nothing validation),
  `update_standalone_task` (`labelIds` is a full replace),
  `create_standalone_comment`.
- **Knowledge Base:** `list_knowledge_docs` (project id or `"__global__"`;
  types `convention|workflow|metric|guide|adr`), list before create, use
  `update_knowledge_doc` to avoid duplicates. ADRs must have
  `## Context`, `## Decision`, `## Consequences`; keep `proposed` until the
  user confirms `accepted`; never delete accepted ADRs — supersede them
  (`adrStatus: superseded`, `supersededBy`).
- **Coding agent:** `assign_to_coding_agent` (`taskKey`/`taskId`, `tier`
  `light` 2GB/1CPU | `mid` default | `heavy` 7GB/4CPU, `repoUrl`,
  `noRepoOutput` `zip|new_repo`, `extraPrompt`, `aiProvider`) returns
  immediately; poll `get_coding_agent_status` (not in a tight loop;
  `queuePosition` while queued).
- **Semantic search:** `ask_mflow` only as a last resort — slow, may change
  data, and is currently the only semantic search.
- **Pitfalls:** 401 → re-auth; confirm before `delete_*` /
  `transfer_knowledge_base mode: move`.

See [`skills/mflow/SKILL.md`](skills/mflow/SKILL.md),
[`skills/mflow/references/tools.md`](skills/mflow/references/tools.md), and
[`skills/mflow/references/recipes.md`](skills/mflow/references/recipes.md).

## Troubleshooting

| Symptom | Fix |
| --- | --- |
| `401` / auth error on any mFlow tool | Re-authenticate: `hermes mcp login mflow` (OAuth tokens expire after 90 days) |
| Error naming `${MFLOW_TOKEN}` / missing env var | Set `MFLOW_TOKEN` in `~/.hermes/.env`, or add the native OAuth entry from Option A |
| `conflicts with native config; skipping` warning | Expected when you use the OAuth override — native config intentionally overrides the package's `mcp.json` server entry |
| Coding agent stuck in `queued` | Sandboxes are busy; wait and poll again later (not in a tight loop) |
| `noRepoOutput: new_repo` fails | GitHub must be connected for the coding agent to create/push a repo |
| Tool not found / `search_knowledge` missing | A dedicated semantic-search MCP tool is not shipped yet; use `ask_mflow` as a fallback |

## Known gaps

- A dedicated `search_knowledge` MCP tool is not available yet; semantic
  search goes through `ask_mflow`.
- PAT-based auth (`MFLOW_TOKEN`) depends on mFlow backend work; OAuth is the
  working path today.

## License

MIT — see [LICENSE](LICENSE).

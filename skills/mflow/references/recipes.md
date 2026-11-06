# mFlow recipes

Copy-paste prompts for common mFlow workflows in Hermes.

## Board triage

```text
Use the mflow skill to triage my mFlow board. List my projects, then for the
one I pick list the tasks in the active sprint/backlog. Flag overdue tasks,
tasks without owners, and tasks that have been stuck in the same status for
more than a week. Output a short triage report.
```

## Spec → task breakdown

```text
Use the mflow skill to turn this spec into tasks in mFlow:
<paste spec>
Ask me which project to use, then create a nested task breakdown (max 50
tasks per call). Use localRef/parentRef for parent/child nesting. After
creating, show me the task keys and hierarchy.
```

## ADR compliance review

```text
Use the mflow skill to review Knowledge Base ADRs in project <name>. List the
docs, then check every ADR has ## Context, ## Decision, and ## Consequences,
is not missing required consequences, and flag any accepted ADRs that look
stale. Do not modify anything without my approval.
```

## Record an ADR

```text
Use the mflow skill to record an ADR for this decision:
<decision>
List existing knowledge docs in the project first. If an ADR already covers
this, recommend updating it. Otherwise create a proposed ADR with ## Context,
## Decision, and ## Consequences. Keep it proposed until I confirm accepted.
```

## Coding-agent handoff

```text
Use the mflow skill to delegate task MVPB-123 to the mFlow coding agent with
tier heavy. Ask me for any repoUrl or extraPrompt if needed. After assigning,
poll get_coding_agent_status occasionally and report when it completes.
```

## Daily stand-up digest (Hermes cron)

```text
Use the mflow skill to produce a stand-up digest for project <name>: tasks
created in the last 24h, tasks moved to done, new comments, and any coding
agent assignments with their current status. Keep it under 200 words.
```

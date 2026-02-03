# Multi-Agent Routing

## Key Concepts

### Why multi-agent

- Isolation: different personas, permissions, workspaces (e.g. family vs work).
- Per-agent: workspace, sessions, auth profiles, sandbox, tool policy.
openclaw agents add work
/agents

See: [Multi-Agent Routing](https://docs.openclaw.ai/concepts/multi-agent)

### agents.list + bindings

- **agents.list[]:** id, default, name, workspace, agentDir, model, identity, groupChat, sandbox, tools, etc.
- **bindings[]:** route inbound messages to agentId by `match.channel`, `match.accountId`, `match.peer`, `match.guildId`/`match.teamId`. Deterministic match order in docs.

See: [Multi-agent routing (config)](https://docs.openclaw.ai/gateway/configuration#multi-agent-routing-agents-list--bindings)

### Per-agent access profiles

- Full access (sandbox off), read-only tools + workspace, or messaging-only (no filesystem). Examples in config docs.

See: [Per-agent access profiles](https://docs.openclaw.ai/gateway/configuration#per-agent-access-profiles-multi-agent)

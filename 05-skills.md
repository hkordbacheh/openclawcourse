# Skills System

## Key Concepts

### What skills are

- AgentSkills-compatible folders: each skill is a directory with **SKILL.md** (YAML frontmatter + instructions). They teach the agent how to use tools.
- Per-agent skills live in <workspace>/skills for that agent only.
- Shared skills live in ~/.openclaw/skills (managed/local) and are visible to all agents on the same machine.
- Set user-invocable: true to invoke skills manually.

See: [Skills](https://docs.openclaw.ai/tools/skills)

### Token impact

- Base overhead when ≥1 skill; per-skill cost (~97 chars + name/description/location). Roughly ~24 tokens per skill.

See: [Token impact](https://docs.openclaw.ai/tools/skills#token-impact-skills-list)

### ClawHub

- Public skills registry: https://clawhub.com. Install: `clawhub install <slug>`; update: `clawhub update --all`.
- Treat third-party skills as untrusted; read before enabling.

[Security notes](https://docs.openclaw.ai/tools/skills#security-notes).
[ClawHub](https://docs.openclaw.ai/tools/clawhub)

### Write an email skill

https://myaccount.google.com/apppasswords

- SMTP_EMAIL
- SMTP_PASSWORD

# Lesson 5: Skills System (~25 min)

**Learning objectives:** Learners understand the AgentSkills format (SKILL.md + YAML frontmatter), skill resolution (workspace → ~/.openclaw/skills → bundled), how to write a minimal custom skill, and how eligibility (os, requiredBinaries, env) and token cost work.

---

## Key Concepts

### What skills are

- AgentSkills-compatible folders: each skill is a directory with **SKILL.md** (YAML frontmatter + instructions). They teach the agent how to use tools.

See: [Skills](https://docs.openclaw.ai/tools/skills)

### Locations and precedence

- **Bundled** (ship with install) → **Managed** (`~/.openclaw/skills`) → **Workspace** (`<workspace>/skills`). Highest wins on name conflict.
- Optional: `skills.load.extraDirs` in config.

See: [Locations and precedence](https://docs.openclaw.ai/tools/skills#locations-and-precedence)

### Format

- Required frontmatter: `name`, `description`. Optional: `homepage`, `user-invocable`, `disable-model-invocation`, `command-dispatch`, `command-tool`, `command-arg-mode`.
- Use `{baseDir}` in instructions for the skill folder path.

See: [Format (AgentSkills + Pi-compatible)](https://docs.openclaw.ai/tools/skills#format-agentskills--pi-compatible)

### Gating (load-time filters)

- `metadata.openclaw`: `os`, `requires.bins`, `requires.anyBins`, `requires.env`, `requires.config`, `primaryEnv`, `install`.
- Skills are filtered at load time; sandboxed agents need binaries inside the container too.

See: [Gating](https://docs.openclaw.ai/tools/skills#gating-load-time-filters)

### Config overrides

- `skills.entries.<name>.enabled`, `apiKey`, `env`, `config` in `~/.openclaw/openclaw.json`.

See: [Config overrides](https://docs.openclaw.ai/tools/skills#config-overrides--openclawopenclawjson)

### Token impact

- Base overhead when ≥1 skill; per-skill cost (~97 chars + name/description/location). Roughly ~24 tokens per skill.

See: [Token impact](https://docs.openclaw.ai/tools/skills#token-impact-skills-list)

### ClawHub

- Public skills registry: https://clawhub.com. Install: `clawhub install <slug>`; update: `clawhub update --all`.

See: [ClawHub](https://docs.openclaw.ai/tools/clawhub)

---

## Lesson Plan

| Segment | Duration | Activity |
|---------|----------|----------|
| What skills are | 3 min | Definition; precedence (workspace > managed > bundled). |
| SKILL.md format | 7 min | Show one bundled skill; frontmatter + body; gating metadata. |
| Write a minimal skill | 10 min | Create `workspace/skills/my-skill/SKILL.md` with name, description, short instructions; reload and confirm it appears. |
| ClawHub + token cost | 5 min | Browse clawhub.com; install one skill; mention token formula. |

---

## Doc References

- [Skills](https://docs.openclaw.ai/tools/skills)
- [Skills Config](https://docs.openclaw.ai/tools/skills-config)
- [ClawHub](https://docs.openclaw.ai/tools/clawhub)

---

## Checklist (Instructor)

- [ ] Treat third-party skills as untrusted; read before enabling. [Security notes](https://docs.openclaw.ai/tools/skills#security-notes).
- [ ] Session snapshot: skill list is fixed when session starts; changes apply on next session (or when skills watcher triggers).
- [ ] Optional: show `skills.load.watch` for auto-refresh during development.

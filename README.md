# agent-skills

A SOLID-principled Claude Code plugin providing reusable skills for common engineering workflows.

## Skills

| Skill | Command | Purpose |
|---|---|---|
| `summarize-changes` | `/agent-skills:summarize-changes` | Summarize uncommitted git changes |
| `code-review` | `/agent-skills:code-review` | Review code quality for a file or diff |
| `run-tests` | `/agent-skills:run-tests [pattern]` | Run tests, optionally filtered by pattern |
| `deploy-app` | `/agent-skills:deploy-app <env>` | Deploy to a target environment |

## SOLID Design

- **S** — Each skill has a single, focused responsibility
- **O** — Extended via `$ARGUMENTS` without editing skill files
- **L** — `paths` scoping ensures skills only activate where their contract holds
- **I** — `allowed-tools` grants only the minimum tools each skill needs
- **D** — Skills detect context and degrade gracefully when dependencies are absent

## Installation

### Local (this session)
```bash
claude --plugin-dir /Users/ajin/workspace/agent-skill
```

### Permanent (add to project)
Add to `.claude/settings.json`:
```json
{
  "plugins": ["/Users/ajin/workspace/agent-skill"]
}
```

## Development

```bash
git clone https://github.com/ajinsanut/agent-skills
cd agent-skills
# Edit skills, then reload inside Claude Code:
# /reload-plugins
```

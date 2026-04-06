# Atlas — Divide & Conquer Development Framework

Atlas is a set of Claude Code skills that solve context bloat in large projects by isolating feature development into separate directories (Feature Labs) that communicate with the main project through well-defined contracts.

## Project Structure

```
plugins/atlas/skills/     ← Claude Code skill definitions (SKILL.md files)
plugins/atlas/templates/  ← Templates for contracts, rules, CLAUDE.md files
.claude-plugin/           ← Marketplace manifest for plugin distribution
```

## Skills

| Skill | Purpose |
|-------|---------|
| `/atlas init` | Initialize Atlas in a new or existing project |
| `/atlas lab create` | Create an isolated feature lab |
| `/atlas lab split` | Split a feature lab into sub-labs |
| `/atlas lab merge` | Claude-assisted merge back to parent |
| `/atlas lab sync` | Sync contracts from parent to lab |
| `/atlas lab list` | List all active labs |
| `/atlas contract` | Manage contracts |
| `/atlas rules` | Manage coding rules |

## Key Concepts

- **Main Pipeline** — The core project with `.atlas/` directory containing contracts, rules, and config
- **Feature Lab** — Isolated directory at `{project}-labs/{lab-name}/` containing only relevant contracts
- **Contract** — Language-agnostic interface definition that specifies integration boundaries
- **Labs Repo** — Sibling git repo `{project-name}-labs/` that holds all feature labs

## Conventions

- Skills are markdown-based (SKILL.md) with YAML frontmatter
- Contracts are markdown with optional code blocks — framework-agnostic
- Labs live OUTSIDE the main project for true Claude context isolation
- Each lab has its own scoped CLAUDE.md so Claude only sees what's relevant

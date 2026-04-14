# agent-skills

Tailored for **Java Spring** (primary) and **TypeScript React** (secondary) projects.

## Project Structure

```
skills/          Core skills — one directory per skill, each with SKILL.md
agents/          Agent personas (markdown files)
hooks/           Session lifecycle hooks
references/      Supplementary checklists and reference material
.claude/commands/  Slash commands that invoke skills
.claude-plugin/  Plugin configuration for Claude Code marketplace
.github/         CI workflows
```

## Skills

| Directory | Skill Name | Phase |
|-----------|-----------|-------|
| `skills/using-agent-skills/` | using-agent-skills | Meta |
| `skills/code-review/` | code-review | Review |
| `skills/test-driven-development/` | test-driven-development | Build |
| `skills/debugging/` | debugging | Debug |
| `skills/andrej-karpathy-skills/` | andrej-karpathy-skills | Discipline |

## Conventions

- Every skill lives at `skills/<name>/SKILL.md`
- The `name` in frontmatter must match the directory name exactly
- Skill names are lowercase, hyphen-separated
- Every skill must have YAML frontmatter with `name` and `description`
- The `description` field: state what the skill does (third person), then when to use it. Max 1024 characters.
- Supporting files (examples, scripts) go in the skill directory. Create only when needed.

## Boundaries

- Always follow the skill anatomy format (see `CONTRIBUTING.md`)
- Never duplicate content between skills — cross-reference by skill name instead
- Do not add skills that duplicate existing ones without a clear differentiation
- The `using-agent-skills` meta-skill must stay accurate — update it when adding skills

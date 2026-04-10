# Contributing

## Adding a New Skill

Skills must meet a quality bar before being accepted: **Specific** (one clear workflow, not general advice), **Verifiable** (ends with a checklist that produces evidence), **Battle-tested** (reflects real engineering practice, not theory), and **Minimal** (as short as it can be while being complete).

### Directory Structure

```
skills/<skill-name>/
  SKILL.md          (required)
  <supporting>.md   (only if content exceeds 100 lines in SKILL.md)
  scripts/          (only if executable scripts are needed)
```

### SKILL.md Format

```markdown
---
name: skill-name-with-hyphens
description: What the skill does (third person). Use when [specific trigger conditions]. Max 1024 chars.
---

# Skill Title

## Overview
1-2 sentences: what this skill does and why it matters.

## When to Use
- Bullet list of specific trigger conditions
- When NOT to use this skill

## [Core Process / The Workflow]
Numbered steps. Be specific — "run `npm test`" beats "verify tests".
Use ASCII diagrams for workflows where helpful.

## Common Rationalizations
| Excuse | Reality |
|--------|---------|

## Red Flags
- Behavioral patterns that indicate this skill is being violated

## Verification
- [ ] Exit criterion with evidence requirement
```

### What NOT to Include

- Vague advice ("write good code", "think carefully")
- Content that duplicates another skill — cross-reference instead
- Supporting files in the skill directory if the content is under 100 lines total
- Theoretical content without actionable steps

### After Adding a Skill

Update these files to register the new skill:
1. `CLAUDE.md` — skills table
2. `AGENTS.md` — intent-to-skill mapping and skill table
3. `skills/using-agent-skills/SKILL.md` — discovery flowchart and quick reference table
4. Optionally: `.claude/commands/<command>.md`

### Submitting

1. Fork the repository
2. Create a branch: `feat/skill-<name>`
3. Add your skill following this guide
4. Open a pull request with a brief description of the skill and why it belongs here

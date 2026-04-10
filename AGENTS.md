# Agent Integration Guide

This file is read by AI coding agents (Claude Code, OpenCode, Cursor, Copilot) to understand how to work with this repository.

## Repository Overview

This repository contains engineering skills for AI coding agents. Each skill is a structured workflow that tells the agent how to approach a specific category of task.

**Three skills are available:**

| Skill | When to invoke | Slash command |
|-------|---------------|---------------|
| `code-review` | Reviewing any code change, PR, or diff | `/review` |
| `test-driven-development` | Writing new code or fixing a bug | `/tdd` |
| `debugging` | Something is broken or behaving unexpectedly | `/debug` |

## Intent-to-Skill Mapping

When the user's intent matches a skill, you MUST invoke that skill before proceeding.

| User intent | Skill to invoke |
|-------------|----------------|
| "Review this", "Check this PR", "What do you think of this code?" | `code-review` |
| "Add a feature", "Implement X", "Write a function for Y" | `test-driven-development` |
| "Fix this bug", "Why is this failing?", "This test is broken" | `debugging` |
| "Write tests for", "Add test coverage" | `test-driven-development` |

## Core Rules

1. **Check for an applicable skill before starting any task.** If one matches, read and follow it fully.
2. **Follow skill steps in order.** Do not skip steps because the task seems simple.
3. **Do not partially apply a skill.** Apply it completely or not at all.
4. **When multiple skills apply** (e.g., fix a bug AND review the fix): apply in sequence — `debugging` first, then `code-review`.

## Anti-Rationalization

The following statements are always incorrect:
- "This is too small for a skill"
- "I'll apply the skill after I get started"
- "The skill doesn't quite fit this situation"
- "I know how to do this without the skill"

Skills exist precisely for situations where the work feels familiar. Discipline is most valuable when the task seems easy.

## Adding a New Skill

1. Create directory: `skills/<skill-name>/`
2. Create `skills/<skill-name>/SKILL.md` with YAML frontmatter (`name`, `description`)
3. Follow the anatomy in `CONTRIBUTING.md`
4. Add a row to the tables in this file and in `CLAUDE.md`
5. Add a branch to the discovery flowchart in `skills/using-agent-skills/SKILL.md`
6. Optionally add a slash command in `.claude/commands/<command-name>.md`

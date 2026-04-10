---
description: Conduct a five-axis code review — correctness, readability, architecture, security, performance
---

Invoke the agent-skills:code-review skill.

Review the current changes (staged changes, recent commits, or specified files) across all five axes:

1. **Correctness** — Does it match the spec? Edge cases handled? Tests adequate?
2. **Readability** — Clear names? Straightforward logic? Easy to follow?
3. **Architecture** — Follows existing patterns? Clean boundaries? Right abstraction level?
4. **Security** — Input validated? Secrets safe? Auth checked?
5. **Performance** — No N+1 queries? No unbounded operations?

Categorize every finding as Critical, Important, or Suggestion.
Output a structured review with specific file:line references and fix recommendations.
Include at least one positive observation about what was done well.
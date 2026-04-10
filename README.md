# agent-skills

Engineering skills for AI coding agents — code review, test-driven development, and systematic debugging. Tailored for **Java Spring** and **TypeScript React** projects.

```
  Review ──── TDD ──── Debug
```

Install once. Your AI agent applies disciplined engineering workflows on every task.

---

## Skills

| Skill | When it activates | Command |
|-------|------------------|---------|
| `code-review` | Reviewing code, PRs, or diffs | `/review` |
| `test-driven-development` | Writing features or fixing bugs | `/tdd` |
| `debugging` | Something is broken or failing | `/debug` |

Plus a meta-skill (`using-agent-skills`) injected automatically at session start to guide skill selection.

---

## Install

### Claude Code (Marketplace)

```
/plugin marketplace add jyadegari/agent-skills
/plugin install agent-skills@jyadegari-agent-skills
```

### Claude Code (Local)

```bash
git clone https://github.com/jyadegari/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

### Other Agents

**Cursor:** Copy skill files into `.cursor/rules/`:
```bash
cp skills/*/SKILL.md .cursor/rules/
```

**OpenCode / Windsurf / Copilot:** See [AGENTS.md](./AGENTS.md) for integration guidance.

---

## How It Works

Every Claude Code session starts with the `using-agent-skills` meta-skill loaded automatically via a session hook. This tells the agent which skill to apply based on the task at hand.

Each skill is a structured workflow with:
- Clear trigger conditions (when to use it)
- Step-by-step process
- Anti-rationalization table (excuses agents use to skip steps, and why they're wrong)
- Red flags (signs the skill is being violated)
- Verification checklist (exit criteria with evidence requirements)

---

## Skill Detail

### `/review` — Code Review

Five-axis review: correctness, readability, architecture, security, and performance. Each axis includes **Spring-specific checks** (transaction boundaries, JPA fetch types, bean scopes, `@Valid`, CORS) and **React-specific checks** (hook dependencies, re-renders, state ownership, `dangerouslySetInnerHTML`). Output categorized as Critical/Important/Suggestion with file:line references.

### `/tdd` — Test-Driven Development

RED-GREEN-REFACTOR cycle with framework-specific guidance:
- **Spring:** JUnit 5 + Mockito service tests, `@WebMvcTest` controller tests, `@DataJpaTest` + Testcontainers repository tests
- **React:** Vitest + Testing Library component tests, `renderHook` for custom hooks, MSW for API mocking

### `/debug` — Debugging

Six-step triage: reproduce → localize → reduce → fix root cause → guard with a test → verify. Includes:
- **Spring error catalog:** `LazyInitializationException`, N+1 queries, `NoSuchBeanDefinitionException`, circular dependencies, security 401/403
- **React error catalog:** infinite re-renders, stale closures, invalid hook calls, unmounted component updates

---

## Adding Skills

See [CONTRIBUTING.md](./CONTRIBUTING.md) for the skill anatomy and submission process.

---

## License

MIT — [Javad Yadegari](https://github.com/jyadegari)

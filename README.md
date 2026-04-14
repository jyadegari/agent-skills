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

Different installation methods give you different capabilities. The table below shows what each method includes:

| Method | Skills (SKILL.md) | Slash commands (`/review`, `/tdd`, `/debug`, `/karpathy`) |
|--------|:-----------------:|:---------------------------------------------------------:|
| `npx skills add` | Yes | No |
| Claude Code Marketplace (`/plugin install`) | Yes | Yes |
| Claude Code local (`--plugin-dir`) | Yes | Yes |
| Cursor (manual copy) | Yes | No — Cursor has no slash command system |
| Windsurf / Copilot / OpenCode | Yes | No — these agents have no slash command system |

---

### Any Agent (Recommended)

Works with Claude Code, Cursor, Copilot, Windsurf, Gemini CLI, and OpenCode:

```bash
npx skills add jyadegari/agent-skills
```

**What you get:** Skill files are copied into your project so the agent can read and apply them. Slash commands are **not** included — invoke skills by name in your prompt instead (e.g. "apply the code-review skill").

Install a specific skill only:

```bash
npx skills add jyadegari/agent-skills --skill code-review
npx skills add jyadegari/agent-skills --skill test-driven-development
npx skills add jyadegari/agent-skills --skill debugging
npx skills add jyadegari/agent-skills --skill andrej-karpathy-skills
```

See all available skills before installing:

```bash
npx skills add jyadegari/agent-skills --list
```

**Optional: add slash commands globally**
If you use Claude Code and want `/review`, `/tdd`, `/debug`, and `/karpathy` available in every project, copy the commands to your global Claude directory after installing:

```bash
cp node_modules/jyadegari-agent-skills/.claude/commands/*.md ~/.claude/commands/
```

---

### Claude Code (Marketplace)

```
/plugin marketplace add jyadegari/agent-skills
/plugin install agent-skills@jyadegari-agent-skills
```

**What you get:** Skills and slash commands are both registered automatically in your current project. `/review`, `/tdd`, `/debug`, and `/karpathy` are available immediately.

---

### Claude Code (Local)

```bash
git clone https://github.com/jyadegari/agent-skills.git
claude --plugin-dir /path/to/agent-skills
```

**What you get:** Claude Code reads the plugin directory directly — skills and slash commands are both available in every session launched with that flag.

To make the commands available in all projects without the flag, copy them globally:

```bash
cp /path/to/agent-skills/.claude/commands/*.md ~/.claude/commands/
```

---

### Cursor

Copy skill files into `.cursor/rules/`:
```bash
cp skills/*/SKILL.md .cursor/rules/
```

**What you get:** Skill content is loaded as Cursor rules. Cursor has no slash command system — invoke skills by referencing the skill name in your prompt (e.g. "@code-review skill").

---

### Other Agents (Windsurf, Copilot, OpenCode)

See [AGENTS.md](./AGENTS.md) for integration guidance.

**What you get:** Skill files only. Invoke skills by name in your prompt. None of these agents support slash commands from external packages.

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

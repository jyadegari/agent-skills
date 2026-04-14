---
name: andrej-karpathy-skills
description: Enforces Andrej Karpathy's LLM coding discipline — think before coding, simplicity first, surgical changes, and goal-driven execution. Use at the start of any coding task to reduce over-engineering, scope creep, and speculative changes. Most valuable when the task feels familiar and discipline is tempting to skip.
---

# Andrej Karpathy Coding Discipline

## Attribution

> These guidelines are adapted from Andrej Karpathy's behavioral guidelines for LLM-assisted coding.
> Source repository: [forrestchang/andrej-karpathy-skills](https://github.com/forrestchang/andrej-karpathy-skills/tree/main)
> Original guidelines authored by [Andrej Karpathy](https://github.com/karpathy).
> Adapted here as a structured agent skill with full credit to the original authors.

## Overview

A set of four discipline rules that counteract the most common failure modes in LLM-assisted coding: assuming instead of asking, adding code that wasn't requested, touching code that wasn't broken, and shipping without verifiable success criteria. Apply them in order at the start of any task.

## When to Use

- Before writing any implementation code
- Before editing existing code
- When the task feels routine — discipline matters most when it seems unnecessary
- When a task could be interpreted multiple ways

Do NOT treat these rules as a substitute for a specific skill (`test-driven-development`, `debugging`, `code-review`). Apply this skill alongside them, not instead of them.

## The Four Rules

### Rule 1: Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them — don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### Rule 2: Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### Rule 3: Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it — don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless explicitly asked.

Test: every changed line should trace directly to the user's request.

### Rule 4: Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan before starting:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Weak criteria ("make it work") require constant clarification. Strong criteria let you loop independently.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "I know what they mean, no need to ask" | Silent assumptions cause rewrites. Ask first. |
| "I'll add this feature while I'm here — it'll be useful" | Unused code is a liability. Build only what was requested. |
| "I'll clean up this adjacent mess while I'm touching the file" | Every unasked change is a risk. Leave it alone or flag it. |
| "It should work — I don't need to verify" | "Should work" is not verified. Run it. Read the output. |
| "This task is too simple to need discipline" | Simple tasks are where discipline slips most often. |

## Red Flags

- You are writing code before fully understanding the request
- The diff includes changes to files not mentioned in the task
- You added a helper, utility, or abstraction for a one-time use
- You are refactoring code that was already working
- You completed the task without running or checking the output

## Verification

Before marking any task complete, confirm:

- [ ] Assumptions were stated explicitly before implementation began
- [ ] Every line in the diff traces directly to the user's request
- [ ] No adjacent code was "improved" beyond what was asked
- [ ] Success criteria were defined upfront and have been met
- [ ] The output was actually run or checked — not just assumed correct

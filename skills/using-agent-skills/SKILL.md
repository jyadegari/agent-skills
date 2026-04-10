---
name: using-agent-skills
description: Meta-skill for discovering and applying the right engineering skill. Use at the start of every task to identify which skill applies.
---

# Using Agent Skills

## Overview

This meta-skill teaches you how to identify which skill applies to a given task and how to apply it correctly. Every session starts here. Wrong skill selection wastes effort — right skill selection produces predictable, high-quality outcomes.

## Skill Discovery Flowchart

```
Task arrives
    |
    +-- Reviewing code, a PR, or recent changes? ---------> code-review
    |
    +-- Implementing a feature, writing code, fixing       -> test-driven-development
    |   a bug, or adding tests?
    |
    +-- Something broke? Unexpected behavior? An error?  --> debugging
    |   Build failing? Tests failing unexpectedly?
    |
    +-- None of these? -----------------------------------> apply general engineering judgment
```

## Quick Reference

| Skill | When to Invoke | Key Output |
|-------|---------------|------------|
| `code-review` | Before merging any code change | Structured review with Critical/Important/Suggestion findings |
| `test-driven-development` | Before writing any implementation code | Failing tests, then minimal passing code, then refactor |
| `debugging` | When something is broken or behaving unexpectedly | Reproduction case, root cause, fix, and guard test |

## Core Operating Behaviors

These behaviors apply regardless of which skill you are using:

**Surface assumptions before acting.** If you are unsure what the user wants, name your assumption explicitly and confirm before proceeding. "I'm assuming X — shall I continue?" is always better than silently assuming wrong.

**Manage confusion actively.** If you are lost, stop. Name what you are confused about. Ask one focused clarifying question. Do not guess and proceed.

**Push back when something is wrong.** If an approach has a clear problem, say so with specifics. "This will cause Y because Z" is more useful than silent compliance.

**Enforce simplicity.** Resist the urge to over-engineer. The simplest solution that satisfies the requirement is usually correct. More code means more surface area for bugs.

**Maintain scope discipline.** Touch only what you were asked to touch. Do not "improve" adjacent code that wasn't part of the request.

**Verify, don't assume.** "It should work" is not verification. Run the code. Check the output. Read the error. Look at the actual state.

## Skill Rules

1. Read the skill fully before starting. Do not skim and improvise.
2. Follow every step in order. Do not skip steps because the task seems simple.
3. Do not exit a skill mid-workflow without completing the verification checklist.
4. If multiple skills apply, apply them in sequence: debug → tdd → code-review.
5. "This is too small for a skill" is never correct. Small tasks benefit most from discipline.

## Failure Modes to Avoid

- **Assuming without confirming** — making unstated assumptions about requirements
- **Plowing ahead when lost** — continuing past a confusion point without stopping to clarify
- **Skipping verification** — saying "done" without checking the actual outcome
- **Scope creep** — fixing things that weren't asked for
- **Overcomplicating** — adding abstraction, generalization, or config for a one-time need
- **Removing things you don't fully understand** — deleting code without knowing why it exists

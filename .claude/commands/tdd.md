---
description: Apply test-driven development — write a failing test first, then implement, then refactor
---

Invoke the agent-skills:test-driven-development skill.

For new features:
1. Write a failing test that describes the desired behavior (RED)
2. Write the minimal code to make the test pass (GREEN)
3. Refactor without changing behavior, keeping all tests passing (REFACTOR)

For bug fixes, use the Prove-It Pattern:
1. Write a test that reproduces the bug (must fail first)
2. Fix the root cause in production code
3. Confirm the test now passes
4. Confirm no other tests regressed

Do NOT write implementation code before a test exists for the behavior.
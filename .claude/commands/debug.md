---
description: Systematically debug a failure — reproduce, localize, fix root cause, and guard with a test
---

Invoke the agent-skills:debugging skill.

Follow the triage checklist in order:
1. **Reproduce** — Confirm the failure is reliable. Read the full error and stack trace.
2. **Localize** — Find the exact line in your code closest to the root of the error.
3. **Reduce** — Find the minimal case that reproduces the failure.
4. **Fix root cause** — Address the source, not a symptom. Make the smallest change needed.
5. **Guard** — Write a test that fails before the fix and passes after.
6. **Verify** — Run all tests. Confirm the original scenario now works end-to-end.

Do NOT add try-catch to suppress errors. Do NOT change multiple things at once.
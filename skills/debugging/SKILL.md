---
name: debugging
description: Systematically diagnose and fix broken code. Use when something is failing, behaving unexpectedly, or throwing an error.
---

# Debugging

## Overview

Debugging is a systematic process, not a guessing game. The goal is to identify the exact root cause of a failure, fix it at the source, and guard against it returning. Jumping to fixes before understanding the root cause produces patches that break something else or mask the real problem.

## When to Use

- Something is throwing an error or exception
- A test is failing unexpectedly
- A build is broken
- Code is producing wrong output
- Behavior changed after a recent code change
- A bug report has been filed

Do NOT use this skill to add features or refactor code. If you identify a bug while reviewing code, switch to this skill to address the bug separately.

## The Stop-the-Line Rule

When something breaks, stop. Do not push forward and hope it resolves. Do not add more code on top of broken code. Do not change multiple things at once and hope one of them fixes it.

Stop. Read the error. Understand it. Then proceed.

Every error message is data. Read it fully — including the stack trace — before touching anything.

## The Triage Checklist

Work through this checklist in order. Do not skip steps.

### Step 1: Reproduce

Can you reliably reproduce the failure?

- Run the failing test, command, or workflow and confirm it fails consistently.
- Note the exact error message, including line numbers and stack trace.
- If you cannot reproduce it, you cannot fix it safely. Find reproduction steps first.

### Step 2: Localize

Where in the code does the failure originate?

- Read the stack trace bottom-up (bottom = where the error was thrown, middle = call chain, top = your code).
- Find the line in your code that is closest to the root of the error.
- Add a minimal log or assertion to confirm your hypothesis about where it fails.

### Step 3: Reduce

What is the minimal case that reproduces the failure?

- Remove everything from the reproduction case that is not essential to the failure.
- A simpler reproduction reveals the cause more clearly.
- If you can reproduce the failure in 5 lines of code, the fix becomes obvious.

### Step 4: Fix the Root Cause

Fix the actual source of the problem, not a symptom.

- A symptom fix: catching and swallowing the error.
- A root cause fix: addressing why the error occurs in the first place.
- If the root cause is in a dependency or framework, understand why before working around it.
- Make the smallest change that fixes the root cause. Do not refactor adjacent code.

### Step 5: Guard

Write a test that would have caught this bug.

- Follow the Prove-It Pattern from the `test-driven-development` skill.
- The test should fail before your fix and pass after.
- Commit the test alongside the fix so the bug cannot silently return.

### Step 6: Verify

Confirm the fix is complete.

- Run the specific failing test or scenario — it must now pass.
- Run all tests — nothing else must be broken.
- Re-read the original error message. Is it fully addressed, or just suppressed?

## Error-Specific Patterns

### Test Failure Triage

```
1. Read the assertion error — what value was expected vs. actual?
2. Is the test correct? (wrong assertion, wrong fixture, wrong setup?)
3. Is the implementation correct? (wrong logic, missing edge case?)
4. Run only this test in isolation — does it still fail? (rules out order dependency)
5. Add a log to see intermediate state at the failure point.
```

### Build Failure Triage

```
1. Read the first error, not the last (errors cascade — the first is the cause).
2. Is this a new error or an existing one? (git diff to see recent changes)
3. Does it reproduce on a clean install? (rules out local environment issues)
4. Search the error message exactly — it's usually a known issue with a known fix.
```

### Runtime Error Triage

```
1. Read the full stack trace — find your code in it.
2. Check what inputs reached the failing line (null? wrong type? unexpected value?).
3. Add an assertion before the failing line to catch the bad input earlier.
4. Is this a logic error (wrong result) or a precondition error (bad input)?
5. If a precondition error: where should the input have been validated?
```

## Spring-Specific Debugging

### Common Spring Errors

| Error | Likely Cause | Fix |
|-------|-------------|-----|
| `NoSuchBeanDefinitionException` | Missing `@Component`/`@Service`/`@Bean`, or not in component scan path | Check annotations and `@ComponentScan` base packages |
| `LazyInitializationException` | Accessing unloaded JPA collection outside transaction | Use `@EntityGraph`, `JOIN FETCH`, or `@Transactional` on the calling method |
| `DataIntegrityViolationException` | Unique constraint, FK violation, or not-null column | Check the SQL constraint name in the message — it tells you which column |
| `HttpMessageNotReadableException` | Malformed JSON in request body | Check `@RequestBody` class fields match the JSON structure |
| `MethodArgumentNotValidException` | `@Valid` failed on request body | Read `errors` list — each entry names the field and constraint |
| Circular dependency on startup | Two beans inject each other | Refactor to break the cycle; use `@Lazy` only as last resort |

### Spring Debugging Techniques

```
1. Enable SQL logging: spring.jpa.show-sql=true + spring.jpa.properties.hibernate.format_sql=true
2. Enable request logging: logging.level.org.springframework.web=DEBUG
3. Use @Transactional(propagation) trace: logging.level.org.springframework.transaction=TRACE
4. Check bean creation: logging.level.org.springframework.beans=DEBUG
5. For 401/403: logging.level.org.springframework.security=DEBUG
```

**The N+1 query problem (most common Spring performance bug):**
```
Symptom:  One query for parent, then one query PER child entity
Detect:   Enable SQL logging, count queries for a single API call
Fix:      @EntityGraph(attributePaths = {"children"}) on the repository method
          OR: SELECT p FROM Parent p JOIN FETCH p.children
Verify:   SQL log shows a single query with JOIN, not N+1 separate queries
```

## React-Specific Debugging

### Common React Errors

| Error | Likely Cause | Fix |
|-------|-------------|-----|
| "Too many re-renders" | State update inside render body (not in handler/effect) | Move state update into `useEffect` or event handler |
| "Cannot update unmounted component" | Async callback setting state after unmount | Use cleanup function in `useEffect`, or AbortController |
| "Invalid hook call" | Hook called conditionally, in a loop, or outside a component | Move hook to top level of component; check for duplicate React versions |
| "Objects are not valid as React child" | Rendering `{object}` instead of `{object.property}` | Render specific properties, or `JSON.stringify` for debugging |
| Stale closure in useEffect | Missing dependency in deps array | Add the variable to the dependency array; use `useCallback` if it's a function |
| Infinite useEffect loop | Object/array in deps that gets recreated each render | Memoize with `useMemo`, or extract the specific primitive values needed |

### React Debugging Techniques

```
1. React DevTools: inspect component tree, props, state, hooks
2. console.log in render: see how often a component re-renders and with what props
3. React.StrictMode: catches side effects by double-invoking render (dev only)
4. Network tab: verify API requests — URL, headers, request body, response status
5. MSW (Mock Service Worker): intercept and mock API responses for isolated testing
6. why-did-you-render: identify unnecessary re-renders in development
```

**The stale closure problem (most common React bug):**
```
Symptom:  useEffect or callback reads outdated state/prop value
Detect:   console.log inside the closure shows old value
Fix:      Add the variable to the useEffect dependency array
          OR: use a ref (useRef) for values needed in event listeners
Verify:   Logged value matches the current state after re-render
```

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "I'll just try a few things and see what sticks" | Random changes waste time and can introduce new bugs. Read the error first. |
| "It's probably an environment issue" | It might be. Prove it. Don't use this as an excuse to stop investigating. |
| "The stack trace is confusing, I'll just add a try-catch" | Catching an error doesn't fix it. It hides it and makes future debugging harder. |
| "I already fixed it, no need to write a test" | Without a test, this bug will return. The test takes 5 minutes and protects you permanently. |
| "I'll look at this later" | Unresolved failures accumulate. Fix it now or create a tracked issue — don't ignore it. |

## Red Flags

- Multiple different things were changed simultaneously to "try to fix" the problem
- The fix is a try-catch that swallows the error without logging or addressing it
- No test was written to guard against this bug recurring
- The reproduction case was never verified before fixing
- The full stack trace was never read
- The fix was applied to a symptom, not the root cause

## Verification

Before closing the bug, confirm:

- [ ] The exact error message was read in full, including the stack trace
- [ ] The failure was reproduced reliably before any fix was attempted
- [ ] The root cause (not a symptom) was identified and fixed
- [ ] A test was written that fails before the fix and passes after
- [ ] All other tests still pass after the fix
- [ ] The original failing scenario now works end-to-end
- [ ] The fix is the minimum change needed — no unrelated code was touched

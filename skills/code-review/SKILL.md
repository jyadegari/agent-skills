---
name: code-review
description: Conduct a thorough multi-axis code review. Use when reviewing any code change, pull request, or diff before it is merged.
---

# Code Review

## Overview

A code review is a systematic examination of a code change across five quality axes. The goal is to catch correctness issues, prevent technical debt, and share knowledge — not to find fault. Every review should be actionable: every finding should state what to fix and why.

## When to Use

- Before merging any pull request or code change
- When asked to review a file, function, or diff
- After implementing a feature (self-review before submitting)
- When `git diff`, staged changes, or a PR URL are present in context

Do NOT use this skill as a substitute for running tests. Review and testing are complementary, not interchangeable.

## The Five Review Axes

### 1. Correctness
Does the code do what it claims to do?

- Does it match the specification or intent?
- Are edge cases handled? (null, empty, overflow, concurrent access)
- Do the tests actually cover the critical paths?
- Are there off-by-one errors, type mismatches, or logic inversions?

**Spring-specific:**
- Are `@Transactional` boundaries correct? (read-only where appropriate, propagation level)
- Do JPA entities have correct cascade types? (`CascadeType.ALL` on a `@ManyToOne` is almost always wrong)
- Are Spring bean scopes correct? (singleton with mutable state = concurrency bug)
- Do `@Async` methods return `Future`/`CompletableFuture`, not `void` when result is needed?

**React-specific:**
- Are hook dependencies correct in `useEffect`, `useMemo`, `useCallback`?
- Are state updates batched or causing unnecessary re-renders?
- Does the component handle loading, error, and empty states?
- Are event handlers properly cleaned up in `useEffect` return?

### 2. Readability
Can a new team member understand this code in 60 seconds?

- Are names descriptive and consistent with the codebase conventions?
- Is the logic easy to follow, or does it require deciphering?
- Are there magic numbers or unexplained constants?
- Is the code organized in the order a reader would expect?

### 3. Architecture
Does this fit cleanly into the existing system?

- Does it follow the existing patterns and conventions in this codebase?
- Are abstractions at the right level? (not too thin, not too thick)
- Does it introduce unnecessary coupling or dependencies?
- Are boundaries (module, service, layer) respected?

**Spring-specific:**
- Is the layering correct? (Controller → Service → Repository — no skipping layers)
- Are DTOs used at API boundaries, not entities? (entity leaking = tight coupling + security risk)
- Does the new code use constructor injection, not field injection (`@Autowired` on fields)?
- Are cross-cutting concerns (logging, validation, auth) handled via aspects/filters, not duplicated?

**React-specific:**
- Is state owned by the right component? (lift only when shared, keep local otherwise)
- Are API calls in the right place? (data fetching in hooks/containers, not in presentational components)
- Is shared state in context/store only when genuinely shared? (not everything belongs in global state)
- Are components small enough to reason about? (>200 lines = probably needs splitting)

### 4. Security
Could this introduce a vulnerability?

- Is all user input validated and sanitized before use?
- Are secrets, tokens, or credentials ever hardcoded or logged?
- Is authorization checked at every entry point?
- Are there SQL injection, XSS, SSRF, or path traversal risks?
- Does error handling leak internal details to callers?

**Spring-specific:**
- Are endpoints secured with `@PreAuthorize` or security config? (open by default = vulnerability)
- Is `@Valid` / `@Validated` on all `@RequestBody` and `@PathVariable` where needed?
- Are native SQL queries parameterized? (`@Query` with string concatenation = SQL injection)
- Is CORS configured explicitly, not `allowedOrigins("*")`?
- Are sensitive fields excluded from JSON serialization? (`@JsonIgnore` on passwords, tokens)

**React-specific:**
- Is user input sanitized before rendering? (avoid `dangerouslySetInnerHTML`)
- Are API tokens stored securely? (not in localStorage for sensitive tokens)
- Are URL parameters validated before use in API calls?
- Is CSRF protection in place for state-changing requests?

### 5. Performance
Could this degrade under load?

- Are there N+1 queries (fetching in a loop instead of batch)?
- Are there unbounded operations (no pagination, no limit)?
- Is expensive work done on the critical path when it could be async or cached?
- Are large objects copied unnecessarily?

**Spring-specific:**
- Are JPA fetch types correct? (`FetchType.EAGER` on collections = N+1 by default)
- Is `@Transactional(readOnly = true)` used for read-only operations? (enables DB optimizations)
- Are database queries using indexes? (new `@Query` without checking the execution plan)
- Is pagination used for list endpoints? (unbounded `findAll()` = OOM under load)
- Are `@Cacheable` boundaries correct? (cache invalidation on writes)

**React-specific:**
- Are expensive computations memoized with `useMemo`?
- Are lists rendered with stable `key` props? (index as key = unnecessary re-renders)
- Are large lists virtualized? (rendering 10,000 DOM nodes = frozen UI)
- Are images lazy-loaded below the fold?
- Does the component avoid re-rendering on every parent render? (`React.memo` where measurably helpful)

## The Review Process

**Step 1: Understand context.** Before reading the diff, read the PR description, issue, or task. Know what this change is supposed to accomplish. A review without context produces noise.

**Step 2: Review tests first.** Read the tests before the implementation. Tests describe intent. If the tests are missing, wrong, or untestable, flag that before reviewing anything else.

**Step 3: Review implementation.** Walk through the implementation systematically. Apply all five axes. Note every finding — even minor ones. You will categorize them next.

**Step 4: Categorize findings.**

| Category | Meaning | Action required |
|----------|---------|-----------------|
| **Critical** | Bug, security issue, data loss risk, or spec mismatch | Must fix before merge |
| **Important** | Maintainability issue, missing test, architectural concern | Should fix before merge |
| **Suggestion** | Style, naming, minor improvement | Optional — author's call |

**Step 5: Write the review.** Output findings with specific file:line references. For each Critical or Important finding, state what the problem is, why it matters, and what to do. For Suggestions, keep it brief.

**Step 6: Verify.** Before submitting the review, re-read it. Does every Critical finding have a clear fix? Are Suggestions genuinely optional? Does the review include something positive?

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "It's a small change, no need for a thorough review" | Small changes introduce bugs too. The five-axis check is fast for small changes. |
| "The tests pass, so it's fine" | Tests only prove what they test. A review checks what tests don't cover. |
| "I'll just note it as a suggestion to avoid conflict" | Downgrading a Critical to a Suggestion misleads the author and allows bugs to merge. |
| "I don't understand this part, I'll skip it" | If you don't understand it, ask. Unreviewed code is unreviewed code. |
| "The original code was worse, this is better" | "Better than before" is not the bar. The bar is "correct, readable, secure, and performant." |

## Red Flags

- Review output has no Critical findings on a large change — likely insufficient depth
- All findings are style-level Suggestions — axes 1 (correctness) and 4 (security) were probably skipped
- Review was done without reading the PR description or linked issue
- No test coverage was checked
- Security axis was skipped because "it's not a security-sensitive feature"

## Verification

Before completing the review, confirm:

- [ ] All five axes were applied, not just the most obvious one
- [ ] Tests were read before implementation
- [ ] Every Critical finding has a specific file:line reference and a suggested fix
- [ ] The review includes at least one positive observation (what was done well)
- [ ] Suggestions are genuinely optional — nothing Critical was downgraded to avoid discomfort
- [ ] The review can be understood by someone who hasn't seen the code before

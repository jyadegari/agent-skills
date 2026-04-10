---
name: test-driven-development
description: Apply test-driven development to produce correct, well-tested code. Use when implementing any feature, fixing a bug, or adding tests to existing code.
---

# Test-Driven Development

## Overview

Test-driven development (TDD) is a discipline where you write a failing test before writing any implementation code. This produces code that is correct by construction, testable by design, and easy to refactor. The discipline is the point — skipping the test-first step defeats the purpose.

## When to Use

- Before writing any new implementation code
- When fixing a bug (write a test that reproduces it first)
- When adding tests to existing untested code
- When refactoring (verify tests pass before and after)

Do NOT write the implementation first and add tests later. "I'll add tests after" produces tests that mirror the implementation rather than specifying the requirement.

## The TDD Cycle

```
   RED          GREEN        REFACTOR
    |             |              |
Write a     Write minimal   Clean up the
failing  --> code to make --> code without
test        the test pass   changing behavior
    |                            |
    +----------------------------+
              Repeat
```

### RED: Write a Failing Test

1. Write a test that describes the desired behavior, not the implementation.
2. Run it. Confirm it fails for the right reason (not a compile error or wrong assertion).
3. If it passes without any implementation, the test is wrong — fix it.

A good test:
- Has a descriptive name: `should_return_empty_list_when_no_items_match`
- Is independent: does not rely on state from other tests
- Tests behavior, not internals: calls the public API, not private methods
- Has one reason to fail: one assertion per concept

**Spring example (JUnit 5):**
```java
@Test
@DisplayName("should return 404 when user does not exist")
void getUserById_notFound() {
    // This test must fail before the endpoint is implemented
    mockMvc.perform(get("/api/users/999"))
        .andExpect(status().isNotFound());
}
```

**React example (Vitest + Testing Library):**
```tsx
it('should show error message when form submission fails', async () => {
  // This test must fail before the error handling is implemented
  render(<LoginForm />);
  await userEvent.click(screen.getByRole('button', { name: /submit/i }));
  expect(screen.getByRole('alert')).toHaveTextContent(/required/i);
});
```

### GREEN: Write Minimal Code to Pass

1. Write only the code needed to make the failing test pass.
2. Resist the urge to generalize or add features not yet tested.
3. Run the test. It must pass. If it doesn't, fix the implementation — not the test.
4. Run all tests. No previously passing test should now fail.

### REFACTOR: Clean Up Without Changing Behavior

1. Eliminate duplication. Extract repeated logic into well-named functions.
2. Improve names. A variable named `x` or `temp` is a future maintenance problem.
3. Remove dead code. Don't leave commented-out experiments.
4. Run all tests after each refactor step. If they fail, you changed behavior — undo and retry.

## The Prove-It Pattern (Bug Fixes)

For bug fixes, never modify production code until you have a failing test that reproduces the bug.

```
1. Read the bug report. Reproduce the bug manually.
2. Write a test that fails exactly because of this bug.
3. Run the test — confirm it fails.
4. Fix the production code.
5. Run the test — confirm it now passes.
6. Run all tests — confirm nothing regressed.
```

This pattern guarantees:
- The bug is understood (you can reproduce it)
- The fix is verified (the test passes)
- The bug cannot silently return (the test guards against regression)

## The Test Pyramid

```
        /\
       /  \   E2E Tests (few — slow, brittle, expensive)
      /----\
     /      \  Integration Tests (some — test component interactions)
    /--------\
   /          \  Unit Tests (many — fast, isolated, precise)
  /____________\
```

Aim for approximately 80% unit, 15% integration, 5% end-to-end. Inversion of this pyramid (many E2E tests, few units) produces slow, flaky, hard-to-debug test suites.

**Spring test types:**
| Layer | Test type | Annotations | Speed |
|-------|-----------|-------------|-------|
| Service logic | Unit test | None (plain JUnit + Mockito) | Fast |
| Repository | Integration | `@DataJpaTest` | Medium |
| Controller | Slice test | `@WebMvcTest` | Medium |
| Full flow | Integration | `@SpringBootTest` | Slow |

**React test types:**
| Layer | Test type | Tool | Speed |
|-------|-----------|------|-------|
| Utility / hook logic | Unit test | Vitest | Fast |
| Component rendering | Component test | Vitest + Testing Library | Fast |
| User flows | Integration | Testing Library + MSW | Medium |
| Full app flows | E2E | Playwright / Cypress | Slow |

## Writing Good Tests

**Arrange-Act-Assert.** Structure every test in three parts:
```
// Arrange: set up the preconditions
// Act: call the code under test
// Assert: verify the outcome
```

**DAMP over DRY.** Tests should be Descriptive And Meaningful to read, even if that means some repetition. A test that is hard to read is a test that will be misunderstood and deleted.

**Test state, not interactions.** Assert what came out (return values, state changes, side effects), not how the code internally did it. Testing implementation details makes tests brittle.

**One assertion per concept.** A test with 10 assertions is 10 tests in a trench coat. When it fails, you don't know which of the 10 things broke. Split it.

**Descriptive names.** `test_user_creation` says nothing. `test_creating_user_with_duplicate_email_returns_conflict_error` says everything.

## Framework-Specific Guidance

### Spring (JUnit 5 + Mockito)

**Service tests — mock the repository, not the service:**
```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {
    @Mock OrderRepository orderRepository;
    @InjectMocks OrderService orderService;

    @Test
    @DisplayName("should throw when order total exceeds credit limit")
    void placeOrder_exceedsCreditLimit() {
        var order = new Order(BigDecimal.valueOf(10_000));
        assertThrows(CreditLimitExceededException.class,
            () -> orderService.place(order));
    }
}
```

**Controller tests — use `@WebMvcTest`, not `@SpringBootTest`:**
```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    @Autowired MockMvc mockMvc;
    @MockBean UserService userService;

    @Test
    void createUser_withInvalidEmail_returns400() throws Exception {
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"email": "not-an-email", "name": "Test"}
                    """))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.errors[0].field").value("email"));
    }
}
```

**Repository tests — use `@DataJpaTest` with Testcontainers for real DB:**
```java
@DataJpaTest
@Testcontainers
class UserRepositoryTest {
    @Container
    static PostgreSQLContainer<?> pg = new PostgreSQLContainer<>("postgres:16");

    @Autowired UserRepository userRepository;

    @Test
    void findByEmail_returnsEmptyWhenNotFound() {
        Optional<User> result = userRepository.findByEmail("nobody@test.com");
        assertThat(result).isEmpty();
    }
}
```

### React (Vitest + React Testing Library)

**Component tests — test what the user sees, not implementation:**
```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

it('should disable submit button while form is submitting', async () => {
  render(<CheckoutForm />);
  const submitBtn = screen.getByRole('button', { name: /place order/i });

  await userEvent.click(submitBtn);
  expect(submitBtn).toBeDisabled();
});
```

**Hook tests — use `renderHook`:**
```tsx
import { renderHook, act } from '@testing-library/react';

it('should toggle value on each call', () => {
  const { result } = renderHook(() => useToggle(false));

  act(() => result.current.toggle());
  expect(result.current.value).toBe(true);

  act(() => result.current.toggle());
  expect(result.current.value).toBe(false);
});
```

**API mocking — use MSW (Mock Service Worker), not manual fetch mocks:**
```tsx
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  http.get('/api/users/:id', ({ params }) => {
    return HttpResponse.json({ id: params.id, name: 'Test User' });
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

## Test Anti-Patterns

| Anti-pattern | Problem |
|-------------|---------|
| Test written after implementation | Tests mirror implementation, not requirements — doesn't catch logic errors |
| Shared mutable state between tests | Tests pass/fail depending on order — non-deterministic |
| Testing private methods directly | Tests break on refactor even when behavior is correct |
| Mocking everything | Tests prove the mocks work, not the code |
| Assertions on irrelevant details | Tests break on unrelated changes — high maintenance cost |
| `// TODO: add tests later` | Later never comes |

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "This is too simple to need a test" | If it's trivial, writing the test takes 2 minutes. If it breaks, you'll wish you had the test. |
| "I'll write tests after I get it working" | You won't. And if you do, those tests will reflect your bugs, not catch them. |
| "TDD slows me down" | TDD slows you down for 30 minutes and saves you 3 hours of debugging next week. |
| "Tests are brittle, they break when I refactor" | Tests that break on refactor test implementation, not behavior. Fix the tests. |
| "The code is too complex to test" | Untestable code is poorly designed code. TDD forces better design. |

## Red Flags

- Implementation code exists before any test was written
- Tests were added after the implementation was "done"
- All tests pass but coverage of edge cases is zero
- Bug was fixed without a reproducing test
- Test names describe implementation rather than behavior
- Test suite takes more than a few minutes to run (too many slow tests)

## Verification

Before completing the implementation, confirm:

- [ ] Tests were written before implementation for every new behavior
- [ ] Every failing test fails for the right reason (behavior, not syntax)
- [ ] Every implementation change was motivated by a specific failing test
- [ ] Bug fixes have a test that reproduces the bug before the fix
- [ ] All tests pass after the final implementation
- [ ] No test was deleted or weakened to make it pass
- [ ] Test names describe behavior, not implementation
- [ ] The test suite runs and passes in isolation (no order dependencies)

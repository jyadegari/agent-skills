# Testing Patterns Reference

Supplementary reference for the `test-driven-development` skill. Patterns for Java Spring (JUnit 5 + Mockito) and TypeScript React (Vitest + Testing Library).

---

## Java Spring Patterns

### Service Unit Test (Mockito)

```java
@ExtendWith(MockitoExtension.class)
class ProductServiceTest {
    @Mock ProductRepository productRepository;
    @InjectMocks ProductService productService;

    @Test
    @DisplayName("should throw when product not found by ID")
    void findById_notFound() {
        when(productRepository.findById(99L)).thenReturn(Optional.empty());

        assertThrows(ProductNotFoundException.class,
            () -> productService.findById(99L));

        verify(productRepository).findById(99L);
    }

    @Test
    @DisplayName("should return product when found")
    void findById_found() {
        var product = new Product(1L, "Widget", BigDecimal.TEN);
        when(productRepository.findById(1L)).thenReturn(Optional.of(product));

        var result = productService.findById(1L);

        assertThat(result.getName()).isEqualTo("Widget");
    }
}
```

### Controller Slice Test (@WebMvcTest)

```java
@WebMvcTest(ProductController.class)
class ProductControllerTest {
    @Autowired MockMvc mockMvc;
    @MockBean ProductService productService;

    @Test
    void getProduct_returns200() throws Exception {
        var product = new ProductDto(1L, "Widget", BigDecimal.TEN);
        when(productService.findById(1L)).thenReturn(product);

        mockMvc.perform(get("/api/products/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("Widget"))
            .andExpect(jsonPath("$.price").value(10));
    }

    @Test
    void getProduct_returns404WhenNotFound() throws Exception {
        when(productService.findById(99L)).thenThrow(new ProductNotFoundException(99L));

        mockMvc.perform(get("/api/products/99"))
            .andExpect(status().isNotFound());
    }

    @Test
    void createProduct_returns400ForInvalidInput() throws Exception {
        mockMvc.perform(post("/api/products")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {"name": "", "price": -1}
                    """))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.errors").isNotEmpty());
    }
}
```

### Repository Test (@DataJpaTest + Testcontainers)

```java
@DataJpaTest
@Testcontainers
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class ProductRepositoryTest {
    @Container
    static PostgreSQLContainer<?> pg = new PostgreSQLContainer<>("postgres:16");

    @DynamicPropertySource
    static void dbProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", pg::getJdbcUrl);
        registry.add("spring.datasource.username", pg::getUsername);
        registry.add("spring.datasource.password", pg::getPassword);
    }

    @Autowired ProductRepository productRepository;

    @Test
    void findByCategory_returnsMatchingProducts() {
        productRepository.save(new Product("Widget", "electronics", BigDecimal.TEN));
        productRepository.save(new Product("Book", "books", BigDecimal.ONE));

        var electronics = productRepository.findByCategory("electronics");

        assertThat(electronics).hasSize(1);
        assertThat(electronics.get(0).getName()).isEqualTo("Widget");
    }
}
```

### Integration Test (@SpringBootTest)

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class OrderFlowIntegrationTest {
    @Container
    static PostgreSQLContainer<?> pg = new PostgreSQLContainer<>("postgres:16");

    @Autowired TestRestTemplate restTemplate;

    @Test
    void fullOrderFlow() {
        // Create product
        var product = restTemplate.postForEntity("/api/products",
            new CreateProductRequest("Widget", BigDecimal.TEN), ProductDto.class);
        assertThat(product.getStatusCode()).isEqualTo(HttpStatus.CREATED);

        // Place order
        var order = restTemplate.postForEntity("/api/orders",
            new CreateOrderRequest(product.getBody().getId(), 2), OrderDto.class);
        assertThat(order.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(order.getBody().getTotal()).isEqualByComparingTo(BigDecimal.valueOf(20));
    }
}
```

---

## TypeScript React Patterns

### Component Test (Vitest + Testing Library)

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { ProductCard } from './ProductCard';

describe('ProductCard', () => {
  const product = { id: 1, name: 'Widget', price: 9.99 };

  it('should render product name and price', () => {
    render(<ProductCard product={product} />);

    expect(screen.getByText('Widget')).toBeInTheDocument();
    expect(screen.getByText('$9.99')).toBeInTheDocument();
  });

  it('should call onAddToCart when button is clicked', async () => {
    const onAddToCart = vi.fn();
    render(<ProductCard product={product} onAddToCart={onAddToCart} />);

    await userEvent.click(screen.getByRole('button', { name: /add to cart/i }));

    expect(onAddToCart).toHaveBeenCalledWith(product.id);
  });

  it('should show out of stock badge when quantity is zero', () => {
    render(<ProductCard product={{ ...product, quantity: 0 }} />);

    expect(screen.getByText(/out of stock/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /add to cart/i })).toBeDisabled();
  });
});
```

### Custom Hook Test

```tsx
import { renderHook, act, waitFor } from '@testing-library/react';
import { useDebounce } from './useDebounce';

it('should debounce value updates', async () => {
  const { result, rerender } = renderHook(
    ({ value }) => useDebounce(value, 300),
    { initialProps: { value: 'hello' } }
  );

  expect(result.current).toBe('hello');

  rerender({ value: 'hello world' });
  // Value should not have changed yet
  expect(result.current).toBe('hello');

  // Wait for debounce
  await waitFor(() => {
    expect(result.current).toBe('hello world');
  });
});
```

### API Integration Test (MSW)

```tsx
import { http, HttpResponse } from 'msw';
import { setupServer } from 'msw/node';
import { render, screen, waitFor } from '@testing-library/react';
import { ProductList } from './ProductList';

const server = setupServer(
  http.get('/api/products', () => {
    return HttpResponse.json([
      { id: 1, name: 'Widget', price: 9.99 },
      { id: 2, name: 'Gadget', price: 19.99 },
    ]);
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

it('should render products from API', async () => {
  render(<ProductList />);

  await waitFor(() => {
    expect(screen.getByText('Widget')).toBeInTheDocument();
    expect(screen.getByText('Gadget')).toBeInTheDocument();
  });
});

it('should show error message when API fails', async () => {
  server.use(
    http.get('/api/products', () => {
      return new HttpResponse(null, { status: 500 });
    })
  );

  render(<ProductList />);

  await waitFor(() => {
    expect(screen.getByRole('alert')).toHaveTextContent(/failed to load/i);
  });
});
```

### Form Test

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

it('should show validation errors for empty required fields', async () => {
  render(<CheckoutForm />);

  await userEvent.click(screen.getByRole('button', { name: /submit/i }));

  expect(screen.getByText(/name is required/i)).toBeInTheDocument();
  expect(screen.getByText(/email is required/i)).toBeInTheDocument();
});

it('should submit form with valid data', async () => {
  const onSubmit = vi.fn();
  render(<CheckoutForm onSubmit={onSubmit} />);

  await userEvent.type(screen.getByLabelText(/name/i), 'Test User');
  await userEvent.type(screen.getByLabelText(/email/i), 'test@example.com');
  await userEvent.click(screen.getByRole('button', { name: /submit/i }));

  expect(onSubmit).toHaveBeenCalledWith({
    name: 'Test User',
    email: 'test@example.com',
  });
});
```

---

## Common Assertion Patterns

### Java (AssertJ — preferred over raw JUnit assertions)

```java
// Equality
assertThat(result).isEqualTo(42);
assertThat(dto).usingRecursiveComparison().isEqualTo(expected);

// Nullability
assertThat(result).isNotNull();
assertThat(result).isNull();

// Collections
assertThat(list).hasSize(3);
assertThat(list).contains(item);
assertThat(list).extracting("name").containsExactly("A", "B", "C");

// Exceptions
assertThatThrownBy(() -> service.find(99L))
    .isInstanceOf(NotFoundException.class)
    .hasMessageContaining("99");

// JSON (MockMvc)
.andExpect(jsonPath("$.name").value("Widget"))
.andExpect(jsonPath("$.items").isArray())
.andExpect(jsonPath("$.items", hasSize(3)))
```

### TypeScript (Vitest + jest-dom)

```typescript
// Equality
expect(result).toBe(42);
expect(result).toEqual({ id: 1, name: 'Widget' });

// DOM assertions (jest-dom)
expect(element).toBeInTheDocument();
expect(element).toBeVisible();
expect(element).toBeDisabled();
expect(element).toHaveTextContent(/hello/i);
expect(element).toHaveAttribute('href', '/products');

// Arrays
expect(arr).toHaveLength(3);
expect(arr).toContain('item');

// Async
await waitFor(() => expect(screen.getByText('Loaded')).toBeInTheDocument());

// Mock assertions
expect(mockFn).toHaveBeenCalledTimes(1);
expect(mockFn).toHaveBeenCalledWith(expect.objectContaining({ id: 1 }));
```

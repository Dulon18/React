# ⚛️ Week 4 — Real-world React

> **Goal:** Write production-quality React code by optimizing performance with `useMemo` and `useCallback`, testing components with React Testing Library, adding type safety with TypeScript, and deploying your app live to the web.

---

## 📋 Table of Contents

1. [Performance Optimization](#1-performance-optimization)
2. [Testing with React Testing Library](#2-testing-with-react-testing-library)
3. [TypeScript Basics](#3-typescript-basics)
4. [Deployment](#4-deployment)
5. [Week 4 Project — Full Todo App](#week-4-project--full-todo-app)
6. [Weekly Checklist](#weekly-checklist)

---

## 1. Performance Optimization

### How React Re-rendering Works

Before optimizing, you need to understand *what* you are optimizing. React re-renders a component whenever:

1. Its **state** changes
2. Its **props** change
3. Its **parent** re-renders

The third point is the one that causes most performance issues. When a parent re-renders, **all of its children re-render too** — even if their props did not change.

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <ExpensiveChild /> {/* Re-renders every time count changes — even though it has no props! */}
    </>
  );
}
```

React is fast enough that most re-renders are harmless. **Only optimize when you can measure a real slowdown.** Premature optimization adds complexity without benefit.

---

### React.memo — Memoize Components

`React.memo` wraps a component and tells React: *"Only re-render this component if its props actually changed."* If the parent re-renders but passes the same props, the memoized child is skipped.

```jsx
import { memo } from 'react';

// Without memo — re-renders every time the parent re-renders
function ProductCard({ product }) {
  console.log("Rendering:", product.name);
  return (
    <div>
      <h3>{product.name}</h3>
      <p>${product.price}</p>
    </div>
  );
}

// With memo — only re-renders when `product` prop changes
const ProductCard = memo(function ProductCard({ product }) {
  console.log("Rendering:", product.name);
  return (
    <div>
      <h3>{product.name}</h3>
      <p>${product.price}</p>
    </div>
  );
});
```

```jsx
function ProductList() {
  const [filter, setFilter] = useState("all");
  const products = useProducts(); // Imagine this returns a large list

  return (
    <>
      <FilterBar filter={filter} onChange={setFilter} />
      {products.map((product) => (
        // ProductCard is memoized — won't re-render when filter changes
        <ProductCard key={product.id} product={product} />
      ))}
    </>
  );
}
```

> ⚠️ `React.memo` uses **shallow comparison** for props. If you pass a new object or array literal on every render, the comparison will always find a difference and memoization won't help. This is where `useMemo` and `useCallback` come in.

---

### useMemo — Memoize Expensive Calculations

`useMemo` caches the **result** of a function call. It only re-runs the function when its dependencies change.

```jsx
import { useMemo } from 'react';

const memoizedValue = useMemo(() => expensiveCalculation(a, b), [a, b]);
```

**Without useMemo — recalculates on every render:**

```jsx
function ProductList({ products, searchQuery }) {
  // This filter runs on EVERY render — even when unrelated state changes
  const filteredProducts = products.filter((p) =>
    p.name.toLowerCase().includes(searchQuery.toLowerCase())
  );

  return <ul>{filteredProducts.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

**With useMemo — only recalculates when products or searchQuery changes:**

```jsx
function ProductList({ products, searchQuery }) {
  const filteredProducts = useMemo(() => {
    console.log("Filtering products..."); // Only logs when deps change
    return products.filter((p) =>
      p.name.toLowerCase().includes(searchQuery.toLowerCase())
    );
  }, [products, searchQuery]); // ← dependencies

  return <ul>{filteredProducts.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

**Another common use — stabilizing object references for other memoized components:**

```jsx
function Dashboard({ userId }) {
  const [theme, setTheme] = useState("light");

  // Without useMemo: new object created every render → breaks React.memo on child
  // With useMemo: same object reference if userId hasn't changed
  const userConfig = useMemo(() => ({
    userId,
    permissions: ["read", "write"],
    dashboardVersion: 2,
  }), [userId]);

  return <MemoizedWidget config={userConfig} />;
}
```

---

### useCallback — Memoize Functions

`useCallback` caches a **function definition**. Without it, every render creates a brand-new function — which breaks `React.memo` on children that receive the function as a prop.

```jsx
import { useCallback } from 'react';

const memoizedFn = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

**The problem without useCallback:**

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  // New function object created on EVERY render
  const handleDelete = (id) => {
    console.log("Deleting", id);
  };

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>Re-render Parent</button>
      {/* MemoizedChild re-renders anyway because handleDelete is a new reference each time */}
      <MemoizedChild onDelete={handleDelete} />
    </>
  );
}
```

**Fixed with useCallback:**

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  // Same function reference across renders — MemoizedChild stays memoized
  const handleDelete = useCallback((id) => {
    console.log("Deleting", id);
  }, []); // No dependencies — function never needs to be recreated

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>Re-render Parent</button>
      <MemoizedChild onDelete={handleDelete} /> {/* Skipped ✓ */}
    </>
  );
}
```

**useCallback with dependencies:**

```jsx
function SearchPage() {
  const [query, setQuery]   = useState("");
  const [page, setPage]     = useState(1);

  // Recreated only when query or page changes
  const fetchResults = useCallback(async () => {
    const res = await fetch(`/api/search?q=${query}&page=${page}`);
    return res.json();
  }, [query, page]);

  useEffect(() => {
    fetchResults();
  }, [fetchResults]); // Safe to include because it's memoized

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

---

### useMemo vs useCallback

| Hook | Caches | Use when |
|------|--------|----------|
| `useMemo` | The **result** of a function | Expensive computation, stabilizing object/array references |
| `useCallback` | The **function itself** | Passing callbacks as props to memoized children, stable deps for useEffect |

```jsx
// useMemo — caches the VALUE
const sortedList = useMemo(() => [...items].sort(), [items]);

// useCallback — caches the FUNCTION
const handleSort = useCallback(() => setSorted(true), []);
```

---

### Code Splitting and Lazy Loading

Instead of loading your entire app upfront, split it into chunks that load on demand. Use `React.lazy` with `Suspense`:

```jsx
import { lazy, Suspense } from 'react';
import { Routes, Route } from 'react-router-dom';

// These components are only downloaded when the user navigates to that route
const Home     = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Settings = lazy(() => import('./pages/Settings'));

function App() {
  return (
    <Suspense fallback={<div>Loading page...</div>}>
      <Routes>
        <Route path="/"          element={<Home />} />
        <Route path="/dashboard" element={<Dashboard />} />
        <Route path="/settings"  element={<Settings />} />
      </Routes>
    </Suspense>
  );
}
```

---

### Performance Checklist

Before reaching for `useMemo` or `useCallback`, ask yourself:

1. **Is there actually a performance problem?** Use React DevTools Profiler to measure first.
2. **Is the calculation genuinely expensive?** Filtering 10 items is not expensive. Sorting 100,000 items is.
3. **Does the component re-render too often?** Check with `console.log` or the Profiler.
4. **Is a memoized child receiving a new object/function reference each render?** Use `useMemo`/`useCallback`.

---

## 2. Testing with React Testing Library

### Why Test?

Tests give you confidence that your app works — and keeps working — as you make changes. React Testing Library (RTL) encourages testing your components **the way a user would use them**: by finding elements on screen and interacting with them, rather than testing implementation details.

> **Core philosophy:** *"The more your tests resemble the way your software is used, the more confidence they can give you."* — Kent C. Dodds

---

### Installation

```bash
# With Vite
npm install --save-dev vitest @testing-library/react @testing-library/user-event @testing-library/jest-dom jsdom

# With Create React App (already included)
npm install --save-dev @testing-library/user-event
```

**vite.config.js** — add test config:

```js
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/setupTests.js',
  },
});
```

**src/setupTests.js:**

```js
import '@testing-library/jest-dom';
```

**package.json** — add test script:

```json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage"
  }
}
```

---

### Core RTL Queries

RTL gives you methods to find elements the way a user would — by visible text, label, role, or placeholder. Prefer them in this order:

| Priority | Query | Use when |
|----------|-------|----------|
| 1st | `getByRole` | Buttons, inputs, headings, links — most elements have a role |
| 2nd | `getByLabelText` | Form inputs associated with a label |
| 3rd | `getByPlaceholderText` | Inputs with a placeholder |
| 4th | `getByText` | Non-interactive elements: paragraphs, spans, divs |
| 5th | `getByTestId` | Last resort — add `data-testid` when nothing else works |

```jsx
// Variants: getBy (throws if not found), queryBy (returns null), findBy (async/returns promise)
const button  = screen.getByRole('button', { name: /submit/i });
const input   = screen.getByLabelText('Email');
const heading = screen.getByText('Welcome back');
const loader  = screen.queryByText('Loading...'); // Returns null if not found — good for asserting absence
const data    = await screen.findByText('John Doe'); // Waits for async updates
```

---

### Your First Test — A Simple Button

```jsx
// components/Button.jsx
export function Button({ onClick, children, disabled }) {
  return (
    <button onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
}
```

```jsx
// components/Button.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Button } from './Button';

describe('Button', () => {
  test('renders with correct text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByRole('button', { name: /click me/i })).toBeInTheDocument();
  });

  test('calls onClick when clicked', async () => {
    const user = userEvent.setup();
    const handleClick = vi.fn(); // Mock function

    render(<Button onClick={handleClick}>Submit</Button>);
    await user.click(screen.getByRole('button', { name: /submit/i }));

    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  test('is disabled when disabled prop is true', () => {
    render(<Button disabled>Submit</Button>);
    expect(screen.getByRole('button', { name: /submit/i })).toBeDisabled();
  });
});
```

---

### Testing a Counter Component

```jsx
// components/Counter.jsx
import { useState } from 'react';

export function Counter({ initialCount = 0 }) {
  const [count, setCount] = useState(initialCount);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(c => c - 1)}>Decrement</button>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

```jsx
// components/Counter.test.jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { Counter } from './Counter';

describe('Counter', () => {
  test('renders initial count', () => {
    render(<Counter initialCount={5} />);
    expect(screen.getByText('Count: 5')).toBeInTheDocument();
  });

  test('increments count when Increment is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter />);

    await user.click(screen.getByRole('button', { name: /increment/i }));
    expect(screen.getByText('Count: 1')).toBeInTheDocument();
  });

  test('decrements count when Decrement is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter initialCount={3} />);

    await user.click(screen.getByRole('button', { name: /decrement/i }));
    expect(screen.getByText('Count: 2')).toBeInTheDocument();
  });

  test('resets count to 0 when Reset is clicked', async () => {
    const user = userEvent.setup();
    render(<Counter initialCount={10} />);

    await user.click(screen.getByRole('button', { name: /reset/i }));
    expect(screen.getByText('Count: 0')).toBeInTheDocument();
  });
});
```

---

### Testing Forms

```jsx
// components/LoginForm.test.jsx
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  test('shows validation errors when submitted empty', async () => {
    const user = userEvent.setup();
    render(<LoginForm onSubmit={vi.fn()} />);

    await user.click(screen.getByRole('button', { name: /login/i }));

    expect(screen.getByText(/email is required/i)).toBeInTheDocument();
    expect(screen.getByText(/password is required/i)).toBeInTheDocument();
  });

  test('shows error for invalid email format', async () => {
    const user = userEvent.setup();
    render(<LoginForm onSubmit={vi.fn()} />);

    await user.type(screen.getByLabelText(/email/i), 'not-an-email');
    await user.click(screen.getByRole('button', { name: /login/i }));

    expect(screen.getByText(/valid email/i)).toBeInTheDocument();
  });

  test('calls onSubmit with values when form is valid', async () => {
    const user = userEvent.setup();
    const handleSubmit = vi.fn();
    render(<LoginForm onSubmit={handleSubmit} />);

    await user.type(screen.getByLabelText(/email/i), 'alice@example.com');
    await user.type(screen.getByLabelText(/password/i), 'securepassword');
    await user.click(screen.getByRole('button', { name: /login/i }));

    await waitFor(() => {
      expect(handleSubmit).toHaveBeenCalledWith({
        email: 'alice@example.com',
        password: 'securepassword',
      });
    });
  });
});
```

---

### Testing Async Components (API Calls)

```jsx
// components/UserProfile.test.jsx
import { render, screen, waitFor } from '@testing-library/react';
import { UserProfile } from './UserProfile';

// Mock the global fetch function
global.fetch = vi.fn();

describe('UserProfile', () => {
  afterEach(() => {
    vi.clearAllMocks();
  });

  test('shows loading state initially', () => {
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => ({ id: 1, name: 'Alice', email: 'alice@example.com' }),
    });

    render(<UserProfile userId={1} />);
    expect(screen.getByText(/loading/i)).toBeInTheDocument();
  });

  test('displays user data after successful fetch', async () => {
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => ({ id: 1, name: 'Alice', email: 'alice@example.com' }),
    });

    render(<UserProfile userId={1} />);

    await waitFor(() => {
      expect(screen.getByText('Alice')).toBeInTheDocument();
      expect(screen.getByText('alice@example.com')).toBeInTheDocument();
    });
  });

  test('displays error message when fetch fails', async () => {
    fetch.mockResolvedValueOnce({ ok: false, status: 404 });

    render(<UserProfile userId={999} />);

    await waitFor(() => {
      expect(screen.getByText(/user not found/i)).toBeInTheDocument();
    });
  });
});
```

---

### Testing with Context

When a component depends on a Context Provider, wrap it in the Provider inside the test:

```jsx
// test-utils.jsx — custom render that wraps with all providers
import { render } from '@testing-library/react';
import { CartProvider } from './context/CartContext';
import { BrowserRouter } from 'react-router-dom';

function AllProviders({ children }) {
  return (
    <BrowserRouter>
      <CartProvider>
        {children}
      </CartProvider>
    </BrowserRouter>
  );
}

export function renderWithProviders(ui, options) {
  return render(ui, { wrapper: AllProviders, ...options });
}
```

```jsx
// components/CartSummary.test.jsx
import { screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { renderWithProviders } from '../test-utils';
import { CartSummary } from './CartSummary';
import { ProductCard } from './ProductCard';

test('updates cart total when item is added', async () => {
  const user = userEvent.setup();
  const product = { id: 1, name: 'Laptop', price: 999 };

  renderWithProviders(
    <>
      <ProductCard product={product} />
      <CartSummary />
    </>
  );

  await user.click(screen.getByRole('button', { name: /add to cart/i }));

  expect(screen.getByText('$999.00')).toBeInTheDocument();
  expect(screen.getByText('1 item')).toBeInTheDocument();
});
```

---

### Common Jest / Vitest Matchers

```jsx
// Existence
expect(element).toBeInTheDocument();
expect(element).not.toBeInTheDocument();

// Visibility
expect(element).toBeVisible();
expect(element).not.toBeVisible();

// State
expect(button).toBeDisabled();
expect(input).toBeEnabled();
expect(checkbox).toBeChecked();

// Content
expect(element).toHaveTextContent('Hello');
expect(input).toHaveValue('alice@example.com');

// Attributes & Style
expect(element).toHaveAttribute('href', '/about');
expect(element).toHaveClass('active');

// Functions (mocks)
expect(mockFn).toHaveBeenCalled();
expect(mockFn).toHaveBeenCalledTimes(2);
expect(mockFn).toHaveBeenCalledWith({ id: 1 });
```

---

## 3. TypeScript Basics

### Why TypeScript?

TypeScript adds **static types** to JavaScript. The TypeScript compiler catches type errors before your code ever runs in the browser — like a spell-checker for your logic.

Benefits in React:
- Autocomplete for props and state in your editor
- Errors when you pass the wrong prop type
- Self-documenting components — the types tell you exactly what a component expects
- Safer refactoring across large codebases

---

### Adding TypeScript to a Vite Project

```bash
# New project with TypeScript
npm create vite@latest my-app -- --template react-ts

# Add TypeScript to an existing project
npm install --save-dev typescript @types/react @types/react-dom
npx tsc --init
```

Rename your files from `.jsx` to `.tsx` and `.js` to `.ts`.

---

### Basic TypeScript Types

```ts
// Primitive types
let name: string  = "Alice";
let age: number   = 25;
let active: boolean = true;

// Arrays
let scores: number[]    = [95, 87, 100];
let names: string[]     = ["Alice", "Bob"];
let mixed: (string | number)[] = ["Alice", 25]; // Union type

// Objects
let user: { name: string; age: number } = { name: "Alice", age: 25 };

// Optional properties
let product: { name: string; description?: string } = { name: "Laptop" };

// Union types — value can be one of several types
let id: string | number = "abc123";
id = 42; // Also valid

// Literal types — only specific values allowed
let direction: "north" | "south" | "east" | "west" = "north";
let status: "idle" | "loading" | "success" | "error" = "idle";

// null and undefined
let maybeNull: string | null = null;
let maybeUndefined: number | undefined = undefined;
```

---

### Interfaces and Type Aliases

Use these to define the shape of objects — especially component props:

```ts
// Interface — can be extended (preferred for objects and props)
interface User {
  id: number;
  name: string;
  email: string;
  avatar?: string; // Optional
  readonly createdAt: Date; // Cannot be changed after creation
}

// Extend an interface
interface AdminUser extends User {
  role: "admin" | "superadmin";
  permissions: string[];
}

// Type alias — more flexible, can represent any type
type ID = string | number;
type Status = "idle" | "loading" | "success" | "error";
type Nullable<T> = T | null;

// Type alias for an object (same as interface for simple cases)
type Product = {
  id: number;
  name: string;
  price: number;
  category: string;
};
```

---

### Typing Component Props

```tsx
// Basic props interface
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: "primary" | "secondary" | "danger";
  disabled?: boolean;
  size?: "sm" | "md" | "lg";
}

function Button({ label, onClick, variant = "primary", disabled = false, size = "md" }: ButtonProps) {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant} btn-${size}`}
    >
      {label}
    </button>
  );
}

// Usage — TypeScript will error if you pass wrong types
<Button label="Submit" onClick={() => console.log("clicked")} />
<Button label="Delete" onClick={handleDelete} variant="danger" />
// <Button label={42} /> ← TypeScript Error: Type 'number' is not assignable to type 'string'
```

---

### Typing useState

TypeScript can usually infer the type from the initial value. For complex types, provide it explicitly:

```tsx
// Inferred — TypeScript knows count is number
const [count, setCount] = useState(0);

// Inferred — TypeScript knows name is string
const [name, setName] = useState("");

// Explicit — needed when initial value is null or ambiguous
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<Product[]>([]);
const [status, setStatus] = useState<"idle" | "loading" | "success" | "error">("idle");

// Example with null initial value
function UserProfile() {
  const [user, setUser] = useState<User | null>(null);

  useEffect(() => {
    fetchUser(1).then(setUser);
  }, []);

  if (!user) return <p>Loading...</p>;

  // TypeScript now knows user is User (not null) — safe to access properties
  return <h1>{user.name}</h1>;
}
```

---

### Typing useReducer

```tsx
// Define state shape
interface CartState {
  items: CartItem[];
  isOpen: boolean;
}

interface CartItem {
  id: number;
  name: string;
  price: number;
  qty: number;
}

// Define all possible actions as a discriminated union
type CartAction =
  | { type: "ADD_ITEM";    payload: Omit<CartItem, "qty"> }
  | { type: "REMOVE_ITEM"; payload: number }
  | { type: "UPDATE_QTY";  payload: { id: number; qty: number } }
  | { type: "CLEAR_CART" }
  | { type: "TOGGLE_CART" };

// Reducer is fully typed — TypeScript catches unknown action types
function cartReducer(state: CartState, action: CartAction): CartState {
  switch (action.type) {
    case "ADD_ITEM":
      return {
        ...state,
        items: [...state.items, { ...action.payload, qty: 1 }],
      };
    case "REMOVE_ITEM":
      return {
        ...state,
        items: state.items.filter((i) => i.id !== action.payload),
      };
    case "CLEAR_CART":
      return { ...state, items: [] };
    case "TOGGLE_CART":
      return { ...state, isOpen: !state.isOpen };
    default:
      return state;
  }
}

const initialState: CartState = { items: [], isOpen: false };

function CartProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  // ...
}
```

---

### Typing Event Handlers

```tsx
function SearchForm() {
  const [query, setQuery] = useState("");

  // Input change event
  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    setQuery(e.target.value);
  }

  // Textarea change event
  function handleTextarea(e: React.ChangeEvent<HTMLTextAreaElement>) {
    console.log(e.target.value);
  }

  // Select change event
  function handleSelect(e: React.ChangeEvent<HTMLSelectElement>) {
    console.log(e.target.value);
  }

  // Form submit event
  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    console.log("Searching for:", query);
  }

  // Button click event
  function handleClick(e: React.MouseEvent<HTMLButtonElement>) {
    console.log("Clicked at:", e.clientX, e.clientY);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={query} onChange={handleChange} />
      <button onClick={handleClick} type="submit">Search</button>
    </form>
  );
}
```

---

### Typing useRef

```tsx
// DOM ref — type is the HTML element type
const inputRef = useRef<HTMLInputElement>(null);
const divRef   = useRef<HTMLDivElement>(null);
const formRef  = useRef<HTMLFormElement>(null);

// Usage
function AutoFocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    inputRef.current?.focus(); // Optional chaining because ref starts as null
  }, []);

  return <input ref={inputRef} />;
}

// Mutable ref (not a DOM ref) — initialize with the value, not null
const timerRef = useRef<ReturnType<typeof setInterval> | null>(null);
const countRef = useRef<number>(0);
```

---

### Typing Custom Hooks

```tsx
// useFetch with generics — works with any data shape
interface FetchResult<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

function useFetch<T>(url: string): FetchResult<T> {
  const [data, setData]       = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError]     = useState<string | null>(null);

  useEffect(() => {
    const controller = new AbortController();
    setLoading(true);

    fetch(url, { signal: controller.signal })
      .then((res) => {
        if (!res.ok) throw new Error("Fetch failed");
        return res.json() as Promise<T>;
      })
      .then(setData)
      .catch((err) => {
        if (err.name !== "AbortError") setError(err.message);
      })
      .finally(() => setLoading(false));

    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}

// Usage — TypeScript knows exactly what shape `data` is
interface Post { id: number; title: string; body: string; }

function PostList() {
  const { data: posts, loading, error } = useFetch<Post[]>(
    "https://jsonplaceholder.typicode.com/posts"
  );

  if (loading) return <p>Loading...</p>;
  if (error)   return <p>{error}</p>;

  return (
    <ul>
      {posts?.map((post) => (
        // TypeScript knows post.id and post.title are available
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}
```

---

## 4. Deployment

### Building for Production

Before deploying, create an optimized production build. Vite minifies code, removes development warnings, and splits your app into small chunks:

```bash
npm run build
```

This creates a `dist/` folder containing everything your server needs to serve. Preview it locally before deploying:

```bash
npm run preview
```

---

### Deploying to Vercel

Vercel is the fastest way to deploy a React app. It detects your framework automatically, builds it, and gives you a live URL.

**Method 1 — Via Vercel CLI (fastest):**

```bash
npm install -g vercel
vercel login
vercel         # Deploy to preview URL
vercel --prod  # Deploy to production
```

**Method 2 — Via GitHub (recommended for real projects):**

1. Push your code to a GitHub repository
2. Go to [vercel.com](https://vercel.com) and sign in with GitHub
3. Click **"Add New Project"** → Import your repository
4. Vercel auto-detects Vite/React — click **Deploy**
5. Every future `git push` to `main` automatically redeploys

---

### Deploying to Netlify

Similar to Vercel, Netlify deploys from GitHub automatically.

**Via Netlify CLI:**

```bash
npm install -g netlify-cli
netlify login
npm run build
netlify deploy --dir=dist          # Preview deploy
netlify deploy --dir=dist --prod   # Production deploy
```

**Via GitHub:**

1. Push to GitHub
2. Go to [app.netlify.com](https://app.netlify.com) → **"Add new site"** → **"Import from Git"**
3. Select your repo
4. Set Build command: `npm run build` | Publish directory: `dist`
5. Click **Deploy site**

---

### Environment Variables

Never hardcode API keys or secrets in your source code. Use environment variables instead:

```bash
# .env.local  (never commit this file — add it to .gitignore)
VITE_API_URL=https://api.myapp.com
VITE_GITHUB_TOKEN=ghp_yourtoken
```

> **Important:** In Vite, environment variables must start with `VITE_` to be available in your app code.

```tsx
// Access them in your code via import.meta.env
const apiUrl = import.meta.env.VITE_API_URL;
const token  = import.meta.env.VITE_GITHUB_TOKEN;

async function fetchData() {
  const res = await fetch(`${apiUrl}/users`, {
    headers: { Authorization: `Bearer ${token}` },
  });
  return res.json();
}
```

**Setting env vars on Vercel:**
1. Go to your project dashboard → **Settings** → **Environment Variables**
2. Add each variable with its value
3. Redeploy — the variables are now available at build time

**Setting env vars on Netlify:**
1. Go to **Site settings** → **Environment variables**
2. Add each variable
3. Trigger a new deploy

---

### Handling Client-Side Routing on Deployment

React Router uses the browser's History API for navigation. When a user visits `myapp.com/about` directly, the server looks for an `about.html` file — which doesn't exist in a SPA. You need to tell the server to always return `index.html`.

**Netlify** — create a `public/_redirects` file:

```
/*  /index.html  200
```

**Vercel** — create a `vercel.json` file in your project root:

```json
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}
```

---

### Continuous Deployment Workflow

Once connected to GitHub, your entire workflow becomes:

```
Write code → git add . → git commit -m "feat: add search filter"
→ git push origin main → Vercel/Netlify auto-builds → Live in ~60 seconds
```

This is the professional development workflow used by real teams.

---

## Week 4 Project — Full Todo App

The capstone project for the entire month. Bring together everything from all 4 weeks: components, hooks, context, routing, TypeScript types, tests, and a live deployment.

### Features

- Add, complete, and delete todos
- Filter by All / Active / Completed
- Persist todos to `localStorage` (using a custom hook)
- Optimized with `useMemo` for filtered list computation
- Fully typed with TypeScript
- Tested with React Testing Library
- Deployed live on Vercel or Netlify

### File Structure

```
src/
├── types/
│   └── index.ts               ← All TypeScript interfaces
├── hooks/
│   ├── useTodos.ts            ← useReducer + localStorage logic
│   └── useLocalStorage.ts     ← Generic localStorage hook
├── components/
│   ├── TodoInput.tsx          ← Controlled input for adding todos
│   ├── TodoItem.tsx           ← Single todo with toggle + delete
│   ├── TodoList.tsx           ← Memoized filtered list
│   ├── TodoFilter.tsx         ← All / Active / Completed buttons
│   └── TodoStats.tsx          ← Items remaining count
├── __tests__/
│   ├── TodoInput.test.tsx
│   ├── TodoItem.test.tsx
│   └── TodoList.test.tsx
└── App.tsx
```

### types/index.ts

```ts
export interface Todo {
  id: string;
  text: string;
  completed: boolean;
  createdAt: Date;
}

export type FilterType = "all" | "active" | "completed";

export type TodoAction =
  | { type: "ADD_TODO";       payload: string }
  | { type: "TOGGLE_TODO";    payload: string }
  | { type: "DELETE_TODO";    payload: string }
  | { type: "CLEAR_COMPLETED" }
  | { type: "SET_TODOS";      payload: Todo[] };
```

### hooks/useTodos.ts

```ts
import { useReducer, useEffect, useMemo } from 'react';
import { Todo, TodoAction, FilterType } from '../types';

function todoReducer(state: Todo[], action: TodoAction): Todo[] {
  switch (action.type) {
    case "ADD_TODO":
      return [
        { id: crypto.randomUUID(), text: action.payload, completed: false, createdAt: new Date() },
        ...state,
      ];
    case "TOGGLE_TODO":
      return state.map((t) =>
        t.id === action.payload ? { ...t, completed: !t.completed } : t
      );
    case "DELETE_TODO":
      return state.filter((t) => t.id !== action.payload);
    case "CLEAR_COMPLETED":
      return state.filter((t) => !t.completed);
    case "SET_TODOS":
      return action.payload;
    default:
      return state;
  }
}

export function useTodos(filter: FilterType) {
  const [todos, dispatch] = useReducer(todoReducer, []);

  // Load from localStorage on mount
  useEffect(() => {
    try {
      const saved = localStorage.getItem("todos");
      if (saved) {
        dispatch({ type: "SET_TODOS", payload: JSON.parse(saved) });
      }
    } catch {
      console.error("Failed to load todos from localStorage");
    }
  }, []);

  // Save to localStorage whenever todos change
  useEffect(() => {
    localStorage.setItem("todos", JSON.stringify(todos));
  }, [todos]);

  // Memoized filtered list — only recalculates when todos or filter changes
  const filteredTodos = useMemo(() => {
    switch (filter) {
      case "active":    return todos.filter((t) => !t.completed);
      case "completed": return todos.filter((t) => t.completed);
      default:          return todos;
    }
  }, [todos, filter]);

  const activeCount    = useMemo(() => todos.filter((t) => !t.completed).length, [todos]);
  const completedCount = useMemo(() => todos.filter((t) => t.completed).length, [todos]);

  return { todos, filteredTodos, activeCount, completedCount, dispatch };
}
```

### App.tsx

```tsx
import { useState } from 'react';
import { FilterType } from './types';
import { useTodos } from './hooks/useTodos';
import TodoInput  from './components/TodoInput';
import TodoList   from './components/TodoList';
import TodoFilter from './components/TodoFilter';
import TodoStats  from './components/TodoStats';

export default function App() {
  const [filter, setFilter] = useState<FilterType>("all");
  const { filteredTodos, activeCount, completedCount, dispatch } = useTodos(filter);

  return (
    <div style={{ maxWidth: 560, margin: "60px auto", padding: "0 20px" }}>
      <h1 style={{ fontSize: 48, fontWeight: 100, color: "#af2f2f", marginBottom: 24 }}>
        todos
      </h1>

      <TodoInput
        onAdd={(text) => dispatch({ type: "ADD_TODO", payload: text })}
      />

      {filteredTodos.length > 0 && (
        <>
          <TodoList
            todos={filteredTodos}
            onToggle={(id) => dispatch({ type: "TOGGLE_TODO", payload: id })}
            onDelete={(id) => dispatch({ type: "DELETE_TODO", payload: id })}
          />
          <TodoStats
            activeCount={activeCount}
            completedCount={completedCount}
            filter={filter}
            onFilterChange={setFilter}
            onClearCompleted={() => dispatch({ type: "CLEAR_COMPLETED" })}
          />
        </>
      )}

      {filteredTodos.length === 0 && (
        <p style={{ textAlign: "center", color: "#aaa", padding: 32 }}>
          {filter === "completed" ? "No completed todos yet." : "Nothing to do! Add a task above."}
        </p>
      )}
    </div>
  );
}
```

### Sample Test — TodoInput.test.tsx

```tsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import TodoInput from '../components/TodoInput';

describe('TodoInput', () => {
  test('calls onAdd with trimmed text when Enter is pressed', async () => {
    const user = userEvent.setup();
    const handleAdd = vi.fn();
    render(<TodoInput onAdd={handleAdd} />);

    const input = screen.getByPlaceholderText(/what needs to be done/i);
    await user.type(input, 'Buy groceries{Enter}');

    expect(handleAdd).toHaveBeenCalledWith('Buy groceries');
    expect(input).toHaveValue(''); // Input cleared after submit
  });

  test('does not call onAdd when input is empty', async () => {
    const user = userEvent.setup();
    const handleAdd = vi.fn();
    render(<TodoInput onAdd={handleAdd} />);

    await user.type(screen.getByRole('textbox'), '{Enter}');
    expect(handleAdd).not.toHaveBeenCalled();
  });

  test('trims whitespace before calling onAdd', async () => {
    const user = userEvent.setup();
    const handleAdd = vi.fn();
    render(<TodoInput onAdd={handleAdd} />);

    await user.type(screen.getByRole('textbox'), '   Walk the dog   {Enter}');
    expect(handleAdd).toHaveBeenCalledWith('Walk the dog');
  });
});
```

---

## Weekly Checklist

- [ ] Use React DevTools Profiler to identify a slow component
- [ ] Wrap an expensive list with `React.memo`
- [ ] Cache a filtered/sorted list with `useMemo`
- [ ] Stabilize a callback prop with `useCallback`
- [ ] Set up Vitest + React Testing Library
- [ ] Write tests for a component (renders, interactions, edge cases)
- [ ] Write a test for an async component (mock fetch, waitFor)
- [ ] Set up TypeScript in your project
- [ ] Type all component props with interfaces
- [ ] Type `useState`, `useReducer`, and `useRef` explicitly
- [ ] Type a custom hook with generics
- [ ] Add environment variables with `VITE_` prefix
- [ ] Run `npm run build` and preview locally
- [ ] Deploy the Todo App live to Vercel or Netlify
- [ ] Fix client-side routing for production (`_redirects` or `vercel.json`)

---

## 📚 Resources

| Resource | Description |
|----------|-------------|
| [React DevTools](https://react.dev/learn/react-developer-tools) | Profile and inspect component renders |
| [Vitest Docs](https://vitest.dev) | Fast unit test runner for Vite projects |
| [Testing Library Docs](https://testing-library.com/docs/react-testing-library/intro) | Official RTL documentation |
| [Common Mistakes with RTL](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library) | Kent C. Dodds — must-read for testers |
| [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) | Complete TypeScript reference |
| [TypeScript + React Cheatsheet](https://react-typescript-cheatsheet.netlify.app) | Quick reference for React + TS patterns |
| [Vercel Docs](https://vercel.com/docs) | Deployment documentation |
| [Netlify Docs](https://docs.netlify.com) | Netlify deployment documentation |

---

##  Congratulations — You Made It!

Over the past 4 weeks, you have built a solid React foundation:

| Week | What You Learned |
|------|-----------------|
| Week 1 | JSX, components, props, useState, events |
| Week 2 | useEffect, useRef, custom hooks, conditional rendering |
| Week 3 | useReducer, Context API, React Router, forms & validation |
| Week 4 | Performance, testing, TypeScript, deployment |

You have 4 real projects in your portfolio: a **Weather Card UI**, a **GitHub User Search**, a **Shopping Cart**, and a fully deployed **Todo App**. You now have everything you need to build real-world React applications.

**What to learn next:**
- **TanStack Query** — server state management (replaces most useEffect fetching)
- **Next.js** — full-stack React with server-side rendering and file-based routing
- **Zustand or Redux Toolkit** — for even larger scale state management
- **Tailwind CSS** — utility-first styling that pairs perfectly with React
- **Storybook** — component documentation and visual testing

Happy coding! 🚀

# ⚛️ Week 3 — State Management

> **Goal:** Scale your application state with `useReducer`, share data globally with Context API, navigate between pages with React Router v6, and build robust forms with validation — all tied together in a full Shopping Cart project.

---

## 📋 Table of Contents

1. [useReducer](#1-usereducer)
2. [Context API](#2-context-api)
3. [React Router v6](#3-react-router-v6)
4. [Forms & Validation](#4-forms--validation)
5. [Week 3 Project — Shopping Cart](#week-3-project--shopping-cart)
6. [Weekly Checklist](#weekly-checklist)

---

## 1. useReducer

### What is useReducer?

`useReducer` is a React hook for managing **complex state logic**. It is an alternative to `useState` — instead of setting state directly, you dispatch **actions** that describe what happened, and a **reducer function** decides how the state should change.

If you've heard of Redux, `useReducer` follows the exact same pattern — just built into React with no extra library needed.

---

### When to Use useReducer vs useState

| Situation | Use |
|-----------|-----|
| Simple value (a number, a string, a boolean) | `useState` |
| Multiple independent values | `useState` (multiple calls) |
| State updates depend on the previous state | `useReducer` |
| Multiple pieces of state updated together | `useReducer` |
| Complex logic with many action types | `useReducer` |
| State logic is hard to follow with `useState` | `useReducer` |

---

### Basic Syntax

```jsx
import { useReducer } from 'react';

const [state, dispatch] = useReducer(reducer, initialState);
```

- `state` — the current state value
- `dispatch` — a function you call to send an action
- `reducer` — a pure function `(state, action) => newState`
- `initialState` — the starting value of the state

---

### The Reducer Function

A reducer is a **pure function** — it takes the current state and an action, and returns the **new** state. It never mutates the old state directly.

```jsx
function reducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + 1 };
    case "DECREMENT":
      return { count: state.count - 1 };
    case "RESET":
      return { count: 0 };
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
}
```

---

### Simple Counter with useReducer

```jsx
import { useReducer } from 'react';

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case "INCREMENT":
      return { count: state.count + 1 };
    case "DECREMENT":
      return { count: state.count - 1 };
    case "RESET":
      return { count: 0 };
    case "SET":
      return { count: action.payload };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <h2>Count: {state.count}</h2>
      <button onClick={() => dispatch({ type: "DECREMENT" })}>-</button>
      <button onClick={() => dispatch({ type: "RESET" })}>Reset</button>
      <button onClick={() => dispatch({ type: "INCREMENT" })}>+</button>
      <button onClick={() => dispatch({ type: "SET", payload: 100 })}>
        Set to 100
      </button>
    </div>
  );
}
```

---

### Actions and Payloads

Actions are plain objects with a `type` field. They can also carry extra data called a **payload**:

```jsx
// Action with no payload
dispatch({ type: "CLEAR_CART" });

// Action with a payload
dispatch({ type: "ADD_ITEM", payload: { id: 1, name: "Laptop", price: 999 } });

// Action with multiple payload fields
dispatch({ type: "UPDATE_QUANTITY", payload: { id: 1, quantity: 3 } });
```

---

### Real-world Example — Todo List with useReducer

This shows how `useReducer` shines when you have multiple action types all affecting the same state:

```jsx
import { useReducer, useState } from 'react';

const initialState = {
  todos: [],
  filter: "all", // "all" | "active" | "completed"
};

function reducer(state, action) {
  switch (action.type) {
    case "ADD_TODO":
      return {
        ...state,
        todos: [
          ...state.todos,
          { id: Date.now(), text: action.payload, completed: false },
        ],
      };

    case "TOGGLE_TODO":
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        ),
      };

    case "DELETE_TODO":
      return {
        ...state,
        todos: state.todos.filter((todo) => todo.id !== action.payload),
      };

    case "SET_FILTER":
      return { ...state, filter: action.payload };

    case "CLEAR_COMPLETED":
      return {
        ...state,
        todos: state.todos.filter((todo) => !todo.completed),
      };

    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

export default function TodoApp() {
  const [state, dispatch] = useReducer(reducer, initialState);
  const [input, setInput] = useState("");

  function handleAdd(e) {
    e.preventDefault();
    if (!input.trim()) return;
    dispatch({ type: "ADD_TODO", payload: input.trim() });
    setInput("");
  }

  const visibleTodos = state.todos.filter((todo) => {
    if (state.filter === "active")    return !todo.completed;
    if (state.filter === "completed") return todo.completed;
    return true;
  });

  return (
    <div style={{ maxWidth: 480, margin: "0 auto", padding: 24 }}>
      <h1>Todo App</h1>

      <form onSubmit={handleAdd} style={{ display: "flex", gap: 8 }}>
        <input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="What needs to be done?"
          style={{ flex: 1, padding: 8 }}
        />
        <button type="submit">Add</button>
      </form>

      <div style={{ display: "flex", gap: 8, margin: "16px 0" }}>
        {["all", "active", "completed"].map((f) => (
          <button
            key={f}
            onClick={() => dispatch({ type: "SET_FILTER", payload: f })}
            style={{ fontWeight: state.filter === f ? "bold" : "normal" }}
          >
            {f.charAt(0).toUpperCase() + f.slice(1)}
          </button>
        ))}
      </div>

      <ul style={{ listStyle: "none", padding: 0 }}>
        {visibleTodos.map((todo) => (
          <li key={todo.id} style={{ display: "flex", gap: 8, padding: "8px 0", borderBottom: "1px solid #eee" }}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch({ type: "TOGGLE_TODO", payload: todo.id })}
            />
            <span style={{ flex: 1, textDecoration: todo.completed ? "line-through" : "none", color: todo.completed ? "#aaa" : "inherit" }}>
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: "DELETE_TODO", payload: todo.id })}>
              ✕
            </button>
          </li>
        ))}
      </ul>

      {state.todos.some((t) => t.completed) && (
        <button onClick={() => dispatch({ type: "CLEAR_COMPLETED" })}>
          Clear completed
        </button>
      )}

      <p style={{ color: "#888", fontSize: 13, marginTop: 8 }}>
        {state.todos.filter((t) => !t.completed).length} items left
      </p>
    </div>
  );
}
```

---

### Refactoring from useState to useReducer

Here is a clear before/after to show when the switch is worth it:

```jsx
// ❌ Before — multiple useState calls that update together
function ShoppingCart() {
  const [items, setItems]         = useState([]);
  const [total, setTotal]         = useState(0);
  const [discount, setDiscount]   = useState(0);
  const [isLoading, setIsLoading] = useState(false);

  function addItem(item) {
    const newItems = [...items, item];
    setItems(newItems);
    setTotal(newItems.reduce((sum, i) => sum + i.price, 0));
  }
  // All these state updates are scattered across multiple functions...
}

// ✅ After — single useReducer with one place for all logic
const initialState = { items: [], total: 0, discount: 0, isLoading: false };

function cartReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM": {
      const newItems = [...state.items, action.payload];
      return {
        ...state,
        items: newItems,
        total: newItems.reduce((sum, i) => sum + i.price, 0),
      };
    }
    // All other cases here — predictable, testable, readable
  }
}
```

---

## 2. Context API

### The Problem: Prop Drilling

When data needs to be passed down through many nested components, you end up threading props through components that don't even use them — this is called **prop drilling**.

```jsx
// ❌ Prop drilling — user has to pass through Layout and Sidebar just to reach Avatar
function App() {
  const user = { name: "Alice", avatar: "/alice.jpg" };
  return <Layout user={user} />;
}

function Layout({ user }) {
  return <Sidebar user={user} />; // Layout doesn't use user — just passes it down
}

function Sidebar({ user }) {
  return <Avatar user={user} />; // Sidebar doesn't use user either
}

function Avatar({ user }) {
  return <img src={user.avatar} alt={user.name} />; // This is the only one that uses it
}
```

**Context** solves this — it lets you broadcast data to any component in the tree, no matter how deeply nested, without passing props at every level.

---

### Creating and Using Context

There are 3 steps: **create**, **provide**, and **consume**.

**Step 1 — Create the context**

```jsx
// context/UserContext.js
import { createContext } from 'react';

export const UserContext = createContext(null);
// null is the default value used when there is no Provider above in the tree
```

**Step 2 — Provide the value**

Wrap the part of your component tree that needs access to the context with a `Provider`. Any component inside this wrapper can read the value.

```jsx
// App.jsx
import { useState } from 'react';
import { UserContext } from './context/UserContext';

function App() {
  const [user, setUser] = useState({ name: "Alice", avatar: "/alice.jpg" });

  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Layout />
    </UserContext.Provider>
  );
}
```

**Step 3 — Consume the value with useContext**

Any component nested inside the Provider can read the context value directly, without receiving it as a prop:

```jsx
// components/Avatar.jsx
import { useContext } from 'react';
import { UserContext } from '../context/UserContext';

function Avatar() {
  const { user } = useContext(UserContext); // Access directly — no prop needed!

  return <img src={user.avatar} alt={user.name} style={{ borderRadius: "50%", width: 40 }} />;
}
```

---

### Building a Theme Context (Light / Dark Mode)

A complete, practical example of Context in action:

```jsx
// context/ThemeContext.jsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext();

// Custom hook to consume the context cleanly
export function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error("useTheme must be used within a ThemeProvider");
  }
  return context;
}

// Provider component that wraps your app
export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");

  function toggleTheme() {
    setTheme((prev) => (prev === "light" ? "dark" : "light"));
  }

  const value = { theme, toggleTheme };

  return (
    <ThemeContext.Provider value={value}>
      <div
        style={{
          background: theme === "light" ? "#ffffff" : "#1a1a2e",
          color:      theme === "light" ? "#000000" : "#eaeaea",
          minHeight:  "100vh",
          padding:    24,
        }}
      >
        {children}
      </div>
    </ThemeContext.Provider>
  );
}
```

```jsx
// App.jsx
import { ThemeProvider } from './context/ThemeContext';
import Header from './components/Header';
import MainContent from './components/MainContent';

function App() {
  return (
    <ThemeProvider>
      <Header />
      <MainContent />
    </ThemeProvider>
  );
}
```

```jsx
// components/Header.jsx
import { useTheme } from '../context/ThemeContext';

function Header() {
  const { theme, toggleTheme } = useTheme();

  return (
    <header style={{ display: "flex", justifyContent: "space-between", padding: "12px 0" }}>
      <h1>My App</h1>
      <button onClick={toggleTheme}>
        {theme === "light" ? "🌙 Dark" : "☀️ Light"} Mode
      </button>
    </header>
  );
}
```

---

### Combining Context with useReducer

For complex global state, pair Context (to share state) with `useReducer` (to manage it). This is the closest pattern to Redux — without any extra library:

```jsx
// context/CartContext.jsx
import { createContext, useContext, useReducer } from 'react';

const CartContext = createContext();

function cartReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM": {
      const exists = state.items.find((i) => i.id === action.payload.id);
      if (exists) {
        return {
          ...state,
          items: state.items.map((i) =>
            i.id === action.payload.id ? { ...i, qty: i.qty + 1 } : i
          ),
        };
      }
      return { ...state, items: [...state.items, { ...action.payload, qty: 1 }] };
    }
    case "REMOVE_ITEM":
      return { ...state, items: state.items.filter((i) => i.id !== action.payload) };
    case "CLEAR_CART":
      return { ...state, items: [] };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

export function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });

  const total = state.items.reduce((sum, i) => sum + i.price * i.qty, 0);

  return (
    <CartContext.Provider value={{ state, dispatch, total }}>
      {children}
    </CartContext.Provider>
  );
}

export function useCart() {
  return useContext(CartContext);
}
```

```jsx
// Any component, anywhere in the tree
import { useCart } from '../context/CartContext';

function ProductCard({ product }) {
  const { dispatch } = useCart();

  return (
    <div>
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => dispatch({ type: "ADD_ITEM", payload: product })}>
        Add to Cart
      </button>
    </div>
  );
}

function CartSummary() {
  const { state, total } = useCart();

  return (
    <div>
      <p>{state.items.length} items in cart</p>
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}
```

---

### When NOT to Use Context

Context is not a silver bullet. Avoid it for:

- **Frequently changing values** (e.g. mouse position, scroll position) — every context consumer re-renders when the value changes. Use local state or a dedicated library instead.
- **Data used by only a few nearby components** — just pass props. Context adds indirection without benefit.
- **Server data / async state** — use TanStack Query or SWR instead.

---

## 3. React Router v6

### What is React Router?

React apps are **Single Page Applications (SPAs)** — the browser loads one HTML file and JavaScript renders different views. React Router lets you map URL paths to different components, giving users the experience of navigating between pages without a full page reload.

---

### Installation

```bash
npm install react-router-dom
```

---

### Basic Setup

Wrap your entire app in `<BrowserRouter>`. Then use `<Routes>` and `<Route>` to define which component renders for which URL path.

```jsx
// main.jsx
import { BrowserRouter } from 'react-router-dom';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')).render(
  <BrowserRouter>
    <App />
  </BrowserRouter>
);
```

```jsx
// App.jsx
import { Routes, Route } from 'react-router-dom';
import Home       from './pages/Home';
import About      from './pages/About';
import Products   from './pages/Products';
import NotFound   from './pages/NotFound';
import Navbar     from './components/Navbar';

function App() {
  return (
    <>
      <Navbar />
      <Routes>
        <Route path="/"         element={<Home />} />
        <Route path="/about"    element={<About />} />
        <Route path="/products" element={<Products />} />
        <Route path="*"         element={<NotFound />} />
      </Routes>
    </>
  );
}
```

---

### Navigation — Link and NavLink

Never use `<a href>` tags for internal navigation — they cause a full page reload. Use `<Link>` instead:

```jsx
import { Link, NavLink } from 'react-router-dom';

function Navbar() {
  return (
    <nav style={{ display: "flex", gap: 16, padding: "12px 24px", background: "#f5f5f5" }}>
      {/* Link — basic navigation */}
      <Link to="/">Home</Link>
      <Link to="/about">About</Link>

      {/* NavLink — automatically adds an "active" class when the URL matches */}
      <NavLink
        to="/products"
        style={({ isActive }) => ({
          fontWeight: isActive ? "bold" : "normal",
          color:      isActive ? "#0066cc" : "inherit",
        })}
      >
        Products
      </NavLink>
    </nav>
  );
}
```

---

### URL Parameters

Use `:paramName` in the route path to capture dynamic segments of the URL. Read them with `useParams`.

```jsx
// Route definition
<Route path="/products/:productId" element={<ProductDetail />} />

// ProductDetail component
import { useParams } from 'react-router-dom';

function ProductDetail() {
  const { productId } = useParams();
  const { data: product, loading, error } = useFetch(`/api/products/${productId}`);

  if (loading) return <p>Loading...</p>;
  if (error)   return <p>Product not found.</p>;

  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p>${product.price}</p>
    </div>
  );
}
```

---

### Programmatic Navigation — useNavigate

Navigate from inside event handlers or after async operations using `useNavigate`:

```jsx
import { useNavigate } from 'react-router-dom';

function CheckoutForm() {
  const navigate = useNavigate();

  async function handleSubmit(e) {
    e.preventDefault();
    await submitOrder();
    navigate("/order-success"); // Redirect after successful submission
  }

  function handleCancel() {
    navigate(-1); // Go back to the previous page (like browser back button)
  }

  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
      <button type="submit">Place Order</button>
      <button type="button" onClick={handleCancel}>Cancel</button>
    </form>
  );
}
```

---

### Query Parameters — useSearchParams

Query parameters (the `?key=value` part of a URL) are read and updated with `useSearchParams`. Useful for search, filtering, and pagination:

```jsx
import { useSearchParams } from 'react-router-dom';

function ProductsPage() {
  const [searchParams, setSearchParams] = useSearchParams();
  const category = searchParams.get("category") || "all";
  const page     = Number(searchParams.get("page")) || 1;

  function handleCategoryChange(newCategory) {
    setSearchParams({ category: newCategory, page: 1 });
  }

  // URL becomes: /products?category=electronics&page=1
  return (
    <div>
      <div style={{ display: "flex", gap: 8 }}>
        {["all", "electronics", "clothing", "books"].map((cat) => (
          <button
            key={cat}
            onClick={() => handleCategoryChange(cat)}
            style={{ fontWeight: category === cat ? "bold" : "normal" }}
          >
            {cat}
          </button>
        ))}
      </div>
      {/* Render filtered products */}
    </div>
  );
}
```

---

### Nested Routes and Layouts

Nested routes let you render child routes inside a parent layout — perfect for dashboards with shared sidebars:

```jsx
// App.jsx
import { Routes, Route, Outlet } from 'react-router-dom';

function DashboardLayout() {
  return (
    <div style={{ display: "flex" }}>
      <aside style={{ width: 200, background: "#f0f0f0", padding: 16 }}>
        <Link to="/dashboard/overview">Overview</Link>
        <Link to="/dashboard/analytics">Analytics</Link>
        <Link to="/dashboard/settings">Settings</Link>
      </aside>
      <main style={{ flex: 1, padding: 24 }}>
        <Outlet /> {/* Child route renders here */}
      </main>
    </div>
  );
}

function App() {
  return (
    <Routes>
      <Route path="/"          element={<Home />} />
      <Route path="/dashboard" element={<DashboardLayout />}>
        <Route index              element={<Overview />} />    {/* /dashboard */}
        <Route path="analytics"  element={<Analytics />} />   {/* /dashboard/analytics */}
        <Route path="settings"   element={<Settings />} />    {/* /dashboard/settings */}
      </Route>
    </Routes>
  );
}
```

---

### Protected Routes

Redirect users who are not logged in away from private pages:

```jsx
import { Navigate, Outlet } from 'react-router-dom';

function ProtectedRoute({ isAuthenticated }) {
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  return <Outlet />;
}

// Usage
<Routes>
  <Route path="/login" element={<Login />} />
  <Route element={<ProtectedRoute isAuthenticated={isLoggedIn} />}>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/profile"   element={<Profile />} />
  </Route>
</Routes>
```

---

## 4. Forms & Validation

### Controlled vs Uncontrolled Inputs

**Controlled inputs** bind the input's value to React state. React is the single source of truth.

**Uncontrolled inputs** let the DOM manage the value itself. You read it with a `ref` when needed.

For most forms, use **controlled inputs** — they give you real-time access to the value and make validation straightforward.

```jsx
// Controlled — React owns the value
const [email, setEmail] = useState("");
<input value={email} onChange={(e) => setEmail(e.target.value)} />

// Uncontrolled — DOM owns the value
const inputRef = useRef(null);
<input ref={inputRef} />
// Read it when needed: inputRef.current.value
```

---

### Building a Complete Login Form

```jsx
import { useState } from 'react';

function LoginForm() {
  const [values, setValues] = useState({ email: "", password: "" });
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [successMessage, setSuccessMessage] = useState("");

  // Single handler for all fields — uses input name attribute
  function handleChange(e) {
    const { name, value } = e.target;
    setValues((prev) => ({ ...prev, [name]: value }));
    // Clear the error for this field as the user types
    if (errors[name]) {
      setErrors((prev) => ({ ...prev, [name]: "" }));
    }
  }

  function validate() {
    const newErrors = {};

    if (!values.email.trim()) {
      newErrors.email = "Email is required.";
    } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(values.email)) {
      newErrors.email = "Please enter a valid email address.";
    }

    if (!values.password) {
      newErrors.password = "Password is required.";
    } else if (values.password.length < 8) {
      newErrors.password = "Password must be at least 8 characters.";
    }

    return newErrors;
  }

  async function handleSubmit(e) {
    e.preventDefault();
    const validationErrors = validate();

    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return; // Stop — do not submit
    }

    setIsSubmitting(true);
    try {
      await new Promise((res) => setTimeout(res, 1500)); // Simulate API call
      setSuccessMessage("Logged in successfully!");
    } catch {
      setErrors({ general: "Login failed. Please try again." });
    } finally {
      setIsSubmitting(false);
    }
  }

  if (successMessage) {
    return <p style={{ color: "green" }}>{successMessage}</p>;
  }

  return (
    <form onSubmit={handleSubmit} style={{ maxWidth: 400, margin: "0 auto" }}>
      <h2>Login</h2>

      {errors.general && (
        <p style={{ color: "red", background: "#fff0f0", padding: 12, borderRadius: 6 }}>
          {errors.general}
        </p>
      )}

      <div style={{ marginBottom: 16 }}>
        <label htmlFor="email" style={{ display: "block", marginBottom: 4 }}>
          Email
        </label>
        <input
          id="email"
          name="email"
          type="email"
          value={values.email}
          onChange={handleChange}
          style={{
            width: "100%",
            padding: 10,
            border: `1px solid ${errors.email ? "red" : "#ccc"}`,
            borderRadius: 6,
          }}
        />
        {errors.email && (
          <p style={{ color: "red", fontSize: 13, margin: "4px 0 0" }}>{errors.email}</p>
        )}
      </div>

      <div style={{ marginBottom: 24 }}>
        <label htmlFor="password" style={{ display: "block", marginBottom: 4 }}>
          Password
        </label>
        <input
          id="password"
          name="password"
          type="password"
          value={values.password}
          onChange={handleChange}
          style={{
            width: "100%",
            padding: 10,
            border: `1px solid ${errors.password ? "red" : "#ccc"}`,
            borderRadius: 6,
          }}
        />
        {errors.password && (
          <p style={{ color: "red", fontSize: 13, margin: "4px 0 0" }}>{errors.password}</p>
        )}
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        style={{ width: "100%", padding: 12, fontSize: 16, cursor: "pointer" }}
      >
        {isSubmitting ? "Logging in..." : "Login"}
      </button>
    </form>
  );
}
```

---

### Building a Reusable useForm Hook

Extract form logic into a reusable custom hook so you never have to repeat it:

```jsx
// hooks/useForm.js
import { useState } from 'react';

export function useForm(initialValues, validate) {
  const [values, setValues]   = useState(initialValues);
  const [errors, setErrors]   = useState({});
  const [touched, setTouched] = useState({});

  function handleChange(e) {
    const { name, value, type, checked } = e.target;
    setValues((prev) => ({
      ...prev,
      [name]: type === "checkbox" ? checked : value,
    }));
  }

  function handleBlur(e) {
    const { name } = e.target;
    setTouched((prev) => ({ ...prev, [name]: true }));
    const validationErrors = validate(values);
    setErrors(validationErrors);
  }

  function handleSubmit(onSubmit) {
    return (e) => {
      e.preventDefault();
      const validationErrors = validate(values);
      setErrors(validationErrors);
      // Mark all fields as touched to show all errors
      setTouched(Object.keys(initialValues).reduce((acc, key) => ({ ...acc, [key]: true }), {}));
      if (Object.keys(validationErrors).length === 0) {
        onSubmit(values);
      }
    };
  }

  function reset() {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  }

  // Show an error only if the field has been touched
  const visibleErrors = Object.keys(errors).reduce((acc, key) => {
    if (touched[key]) acc[key] = errors[key];
    return acc;
  }, {});

  return { values, errors: visibleErrors, handleChange, handleBlur, handleSubmit, reset };
}
```

```jsx
// Usage — clean and zero boilerplate
import { useForm } from './hooks/useForm';

function validate(values) {
  const errors = {};
  if (!values.name.trim()) errors.name = "Name is required.";
  if (!values.email.trim()) errors.email = "Email is required.";
  else if (!/\S+@\S+\.\S+/.test(values.email)) errors.email = "Invalid email.";
  if (!values.message.trim()) errors.message = "Message is required.";
  return errors;
}

function ContactForm() {
  const { values, errors, handleChange, handleBlur, handleSubmit, reset } = useForm(
    { name: "", email: "", message: "" },
    validate
  );

  function onSubmit(values) {
    console.log("Form submitted:", values);
    reset();
  }

  return (
    <form onSubmit={handleSubmit(onSubmit)} style={{ maxWidth: 480 }}>
      <h2>Contact Us</h2>

      {[
        { name: "name",    label: "Name",    type: "text" },
        { name: "email",   label: "Email",   type: "email" },
      ].map(({ name, label, type }) => (
        <div key={name} style={{ marginBottom: 16 }}>
          <label htmlFor={name}>{label}</label>
          <input
            id={name}
            name={name}
            type={type}
            value={values[name]}
            onChange={handleChange}
            onBlur={handleBlur}
            style={{ display: "block", width: "100%", padding: 8, marginTop: 4 }}
          />
          {errors[name] && <p style={{ color: "red", fontSize: 13 }}>{errors[name]}</p>}
        </div>
      ))}

      <div style={{ marginBottom: 16 }}>
        <label htmlFor="message">Message</label>
        <textarea
          id="message"
          name="message"
          value={values.message}
          onChange={handleChange}
          onBlur={handleBlur}
          rows={4}
          style={{ display: "block", width: "100%", padding: 8, marginTop: 4 }}
        />
        {errors.message && <p style={{ color: "red", fontSize: 13 }}>{errors.message}</p>}
      </div>

      <button type="submit" style={{ padding: "10px 24px" }}>
        Send Message
      </button>
    </form>
  );
}
```

---

### Form Input Types

How to handle different input types in controlled forms:

```jsx
function AllInputTypesExample() {
  const [form, setForm] = useState({
    name:       "",
    age:        "",
    gender:     "male",
    agree:      false,
    plan:       "free",
    color:      "#ff0000",
    birthdate:  "",
  });

  function handleChange(e) {
    const { name, value, type, checked } = e.target;
    setForm((prev) => ({
      ...prev,
      [name]: type === "checkbox" ? checked : value,
    }));
  }

  return (
    <form>
      {/* Text */}
      <input name="name" type="text" value={form.name} onChange={handleChange} />

      {/* Number */}
      <input name="age" type="number" value={form.age} onChange={handleChange} />

      {/* Select dropdown */}
      <select name="gender" value={form.gender} onChange={handleChange}>
        <option value="male">Male</option>
        <option value="female">Female</option>
        <option value="other">Other</option>
      </select>

      {/* Checkbox */}
      <input name="agree" type="checkbox" checked={form.agree} onChange={handleChange} />

      {/* Radio buttons */}
      {["free", "pro", "enterprise"].map((plan) => (
        <label key={plan}>
          <input
            name="plan"
            type="radio"
            value={plan}
            checked={form.plan === plan}
            onChange={handleChange}
          />
          {plan}
        </label>
      ))}

      {/* Color picker */}
      <input name="color" type="color" value={form.color} onChange={handleChange} />

      {/* Date */}
      <input name="birthdate" type="date" value={form.birthdate} onChange={handleChange} />
    </form>
  );
}
```

---

## Week 3 Project — Shopping Cart

Build a multi-page shopping cart app using everything from this week.

### Features
- Product listing page with Add to Cart button
- Cart page showing items, quantities, and totals
- Checkout page with a validated form
- Navigation between pages with React Router
- Global cart state with Context + useReducer

### File Structure

```
src/
├── context/
│   └── CartContext.jsx        ← useReducer + Context
├── hooks/
│   └── useForm.js             ← reusable form hook
├── pages/
│   ├── ProductsPage.jsx       ← product grid
│   ├── CartPage.jsx           ← cart items + totals
│   └── CheckoutPage.jsx       ← validated checkout form
├── components/
│   ├── Navbar.jsx             ← NavLink navigation + cart badge
│   └── ProductCard.jsx        ← single product display
└── App.jsx                    ← Routes setup
```

### CartContext.jsx

```jsx
import { createContext, useContext, useReducer } from 'react';

const CartContext = createContext();

function cartReducer(state, action) {
  switch (action.type) {
    case "ADD_ITEM": {
      const exists = state.items.find((i) => i.id === action.payload.id);
      if (exists) {
        return {
          ...state,
          items: state.items.map((i) =>
            i.id === action.payload.id ? { ...i, qty: i.qty + 1 } : i
          ),
        };
      }
      return { ...state, items: [...state.items, { ...action.payload, qty: 1 }] };
    }
    case "REMOVE_ITEM":
      return { ...state, items: state.items.filter((i) => i.id !== action.payload) };
    case "UPDATE_QTY":
      return {
        ...state,
        items: state.items.map((i) =>
          i.id === action.payload.id ? { ...i, qty: action.payload.qty } : i
        ).filter((i) => i.qty > 0),
      };
    case "CLEAR_CART":
      return { ...state, items: [] };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

export function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });
  const itemCount = state.items.reduce((sum, i) => sum + i.qty, 0);
  const total     = state.items.reduce((sum, i) => sum + i.price * i.qty, 0);
  return (
    <CartContext.Provider value={{ state, dispatch, itemCount, total }}>
      {children}
    </CartContext.Provider>
  );
}

export const useCart = () => useContext(CartContext);
```

### App.jsx

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { CartProvider } from './context/CartContext';
import Navbar        from './components/Navbar';
import ProductsPage  from './pages/ProductsPage';
import CartPage      from './pages/CartPage';
import CheckoutPage  from './pages/CheckoutPage';

export default function App() {
  return (
    <BrowserRouter>
      <CartProvider>
        <Navbar />
        <div style={{ maxWidth: 1100, margin: "0 auto", padding: "24px 20px" }}>
          <Routes>
            <Route path="/"         element={<ProductsPage />} />
            <Route path="/cart"     element={<CartPage />} />
            <Route path="/checkout" element={<CheckoutPage />} />
          </Routes>
        </div>
      </CartProvider>
    </BrowserRouter>
  );
}
```

---

## Weekly Checklist

- [ ] Refactor a `useState` component to use `useReducer`
- [ ] Write a reducer with at least 4 different action types
- [ ] Create a Context with a custom `useContext` hook
- [ ] Build a global theme toggle (light/dark) with Context
- [ ] Combine `useReducer` + Context for global cart state
- [ ] Set up React Router with at least 3 routes
- [ ] Use `NavLink` with active styling in a navigation bar
- [ ] Read a URL parameter with `useParams`
- [ ] Navigate programmatically after form submission with `useNavigate`
- [ ] Build a validated form with real-time error clearing
- [ ] Complete the Shopping Cart project

---

## 📚 Resources

| Resource | Description |
|----------|-------------|
| [react.dev — useReducer](https://react.dev/reference/react/useReducer) | Official useReducer reference |
| [react.dev — Context](https://react.dev/learn/passing-data-deeply-with-context) | Deep dive into Context API |
| [React Router v6 Docs](https://reactrouter.com/en/main) | Full React Router documentation |
| [React Router Tutorial](https://reactrouter.com/en/main/start/tutorial) | Hands-on tutorial from the maintainers |
| [Zod](https://zod.dev) | TypeScript-first schema validation (great for forms) |
| [React Hook Form](https://react-hook-form.com) | Library that makes forms even easier — worth knowing |

---

> **Up next → Week 4: Real-world React** — Optimize performance with `useMemo` and `useCallback`, write tests with React Testing Library, add TypeScript, and deploy your app live.

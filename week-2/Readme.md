# ⚛️ Week 2 — Hooks & Effects

> **Goal:** Master side effects with `useEffect`, access the DOM with `useRef`, build reusable logic with custom hooks, and control what gets rendered with conditional rendering.

---

## 📋 Table of Contents

1. [useEffect](#1-useeffect)
2. [useRef](#2-useref)
3. [Custom Hooks](#3-custom-hooks)
4. [Conditional Rendering](#4-conditional-rendering)
5. [Week 2 Project — GitHub User Search](#week-2-project--github-user-search)
6. [Weekly Checklist](#weekly-checklist)

---

## 1. useEffect

### What is a Side Effect?

A **side effect** is anything a component does that reaches outside of its own render cycle. Examples include:

- Fetching data from an API
- Setting a document title
- Starting a timer
- Adding/removing a DOM event listener
- Writing to `localStorage`

React's render function should be **pure** — given the same props and state, it should always return the same JSX. Side effects don't belong inside the render. That's exactly what `useEffect` is for.

---

### Basic Syntax

```jsx
import { useEffect } from 'react';

useEffect(() => {
  // Your side effect code goes here
}, [dependencies]);
```

`useEffect` takes two arguments:
1. A **callback function** — the effect to run
2. A **dependency array** — controls *when* the effect re-runs

---

### The Dependency Array

The dependency array is the most important part of `useEffect`. It has three forms:

**Form 1 — No dependency array: runs after every render**

```jsx
useEffect(() => {
  console.log("Runs after every single render");
});
```

**Form 2 — Empty array `[]`: runs only once (on mount)**

```jsx
useEffect(() => {
  console.log("Runs only once when the component first appears");
}, []);
```

**Form 3 — Array with values: runs when those values change**

```jsx
const [count, setCount] = useState(0);

useEffect(() => {
  console.log("Runs whenever 'count' changes:", count);
}, [count]);
```

> **Rule:** Every value from your component that is used inside the effect must be listed in the dependency array. The ESLint plugin `eslint-plugin-react-hooks` will warn you if you forget.

---

### Fetching Data with useEffect

The most common use of `useEffect` is fetching data from an API when a component loads:

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    setLoading(true);
    setError(null);

    fetch(`https://jsonplaceholder.typicode.com/users/${userId}`)
      .then((res) => {
        if (!res.ok) throw new Error("User not found");
        return res.json();
      })
      .then((data) => {
        setUser(data);
        setLoading(false);
      })
      .catch((err) => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]); // Re-fetches whenever userId prop changes

  if (loading) return <p>Loading...</p>;
  if (error)   return <p>Error: {error}</p>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

---

### Using async/await Inside useEffect

`useEffect`'s callback cannot be `async` directly (because async functions return a Promise, and useEffect expects either nothing or a cleanup function). Instead, define an async function inside and call it immediately:

```jsx
useEffect(() => {
  // ❌ Wrong — useEffect callback cannot be async
  // async () => { ... }

  // ✅ Correct — define async inside, then call it
  async function fetchData() {
    try {
      const res = await fetch("https://api.example.com/data");
      const data = await res.json();
      setData(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  fetchData();
}, []);
```

---

### The Cleanup Function

`useEffect` can return a **cleanup function**. React runs this cleanup:
- Before the component is removed from the DOM (unmount)
- Before the effect runs again (when dependencies change)

This prevents memory leaks, dangling subscriptions, and stale data.

```jsx
// Example 1: Cleanup a timer
useEffect(() => {
  const timer = setInterval(() => {
    console.log("Tick");
  }, 1000);

  return () => {
    clearInterval(timer); // Cleanup when component unmounts
  };
}, []);
```

```jsx
// Example 2: Abort a fetch when the component unmounts or userId changes
useEffect(() => {
  const controller = new AbortController();

  async function fetchUser() {
    try {
      const res = await fetch(`/api/users/${userId}`, {
        signal: controller.signal,
      });
      const data = await res.json();
      setUser(data);
    } catch (err) {
      if (err.name !== "AbortError") {
        setError(err.message);
      }
    }
  }

  fetchUser();

  return () => {
    controller.abort(); // Cancel the in-flight request on cleanup
  };
}, [userId]);
```

```jsx
// Example 3: Remove an event listener
useEffect(() => {
  function handleResize() {
    setWidth(window.innerWidth);
  }

  window.addEventListener("resize", handleResize);

  return () => {
    window.removeEventListener("resize", handleResize); // Cleanup
  };
}, []);
```

---

### Setting the Document Title

A simple but practical use of `useEffect`:

```jsx
function PageTitle({ title }) {
  useEffect(() => {
    document.title = title;

    // Reset on unmount
    return () => {
      document.title = "My App";
    };
  }, [title]);

  return <h1>{title}</h1>;
}
```

---

### useEffect Lifecycle Summary

```
Component mounts   → effect runs (if [] or no array)
State/prop changes → cleanup runs → effect runs again (if deps listed)
Component unmounts → cleanup runs
```

---

## 2. useRef

### What is useRef?

`useRef` gives you a **mutable container** that persists across renders. It has two main uses:

1. **Accessing a DOM element directly** (like `document.getElementById` but the React way)
2. **Storing a value that survives re-renders without causing one** (unlike `useState`)

```jsx
import { useRef } from 'react';

const myRef = useRef(initialValue);
// myRef.current === initialValue
```

The key difference from `useState`: changing `ref.current` does **not** trigger a re-render.

---

### Use Case 1 — Accessing DOM Elements

Attach a ref to a JSX element using the `ref` attribute. After the component renders, `ref.current` points to the actual DOM node.

```jsx
import { useRef } from 'react';

function SearchBar() {
  const inputRef = useRef(null);

  function handleFocus() {
    inputRef.current.focus(); // Directly calls focus() on the DOM input
  }

  function handleClear() {
    inputRef.current.value = "";
    inputRef.current.focus();
  }

  return (
    <div>
      <input ref={inputRef} type="text" placeholder="Search..." />
      <button onClick={handleFocus}>Focus</button>
      <button onClick={handleClear}>Clear</button>
    </div>
  );
}
```

```jsx
// Another example — auto-focus an input when the component mounts
function AutoFocusInput() {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current.focus();
  }, []); // Runs once after mount

  return <input ref={inputRef} type="text" />;
}
```

---

### Use Case 2 — Storing Values Without Re-rendering

Sometimes you need to remember a value across renders, but you don't want that value to *cause* a re-render when it changes. Classic examples are timer IDs, previous values, and render counts.

```jsx
// Tracking how many times a component re-renders
import { useState, useRef, useEffect } from 'react';

function RenderCounter() {
  const [count, setCount] = useState(0);
  const renderCount = useRef(0);

  useEffect(() => {
    renderCount.current += 1; // Mutating ref.current — no re-render triggered
  });

  return (
    <div>
      <p>Count: {count}</p>
      <p>This component has rendered {renderCount.current} times</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}
```

```jsx
// Storing a timer ID so it can be cleared later
function Stopwatch() {
  const [seconds, setSeconds] = useState(0);
  const [running, setRunning] = useState(false);
  const timerRef = useRef(null);

  function start() {
    setRunning(true);
    timerRef.current = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
  }

  function stop() {
    setRunning(false);
    clearInterval(timerRef.current);
  }

  function reset() {
    stop();
    setSeconds(0);
  }

  return (
    <div>
      <p>{seconds}s</p>
      <button onClick={start} disabled={running}>Start</button>
      <button onClick={stop} disabled={!running}>Stop</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

---

### Use Case 3 — Tracking Previous State

A common pattern is storing the previous value of a prop or state:

```jsx
import { useState, useRef, useEffect } from 'react';

function PriceDisplay({ price }) {
  const prevPriceRef = useRef(price);

  useEffect(() => {
    prevPriceRef.current = price; // Update after every render
  });

  const prevPrice = prevPriceRef.current;
  const increased = price > prevPrice;

  return (
    <div>
      <p>Current: ${price}</p>
      <p>Previous: ${prevPrice}</p>
      <p style={{ color: increased ? "green" : "red" }}>
        {increased ? "▲ Up" : "▼ Down"}
      </p>
    </div>
  );
}
```

---

### useState vs useRef — When to Use Which

| Situation | Use |
|-----------|-----|
| Value displayed in UI | `useState` — triggers re-render to show update |
| Timer ID, socket, previous value | `useRef` — persists without re-render |
| Direct DOM access (focus, scroll) | `useRef` |
| Form input value (controlled) | `useState` |
| Tracking if component is mounted | `useRef` |

---

## 3. Custom Hooks

### What is a Custom Hook?

A **custom hook** is a regular JavaScript function whose name starts with `use` and that can call other hooks inside it. They let you extract and reuse stateful logic across multiple components.

Without custom hooks, you'd copy-paste the same `useState` + `useEffect` patterns everywhere. Custom hooks solve this.

---

### Your First Custom Hook — `useLocalStorage`

```jsx
import { useState } from 'react';

function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  function setValue(value) {
    try {
      setStoredValue(value);
      localStorage.setItem(key, JSON.stringify(value));
    } catch (err) {
      console.error(err);
    }
  }

  return [storedValue, setValue];
}

// Usage — works just like useState but persists to localStorage
function ThemeToggle() {
  const [theme, setTheme] = useLocalStorage("theme", "light");

  return (
    <div data-theme={theme}>
      <p>Current theme: {theme}</p>
      <button onClick={() => setTheme(theme === "light" ? "dark" : "light")}>
        Toggle Theme
      </button>
    </div>
  );
}
```

---

### `useFetch` — The Most Useful Custom Hook

Extract data-fetching logic into a reusable hook:

```jsx
import { useState, useEffect } from 'react';

function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    if (!url) return;

    const controller = new AbortController();
    setLoading(true);
    setError(null);

    async function fetchData() {
      try {
        const res = await fetch(url, { signal: controller.signal });
        if (!res.ok) throw new Error(`HTTP error! status: ${res.status}`);
        const json = await res.json();
        setData(json);
      } catch (err) {
        if (err.name !== "AbortError") {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchData();

    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}

// Usage in any component — clean and no repetition
function PostList() {
  const { data: posts, loading, error } = useFetch(
    "https://jsonplaceholder.typicode.com/posts"
  );

  if (loading) return <p>Loading posts...</p>;
  if (error)   return <p>Error: {error}</p>;

  return (
    <ul>
      {posts.slice(0, 5).map((post) => (
        <li key={post.id}>{post.title}</li>
      ))}
    </ul>
  );
}

// Reuse in a completely different component — same hook, zero duplication
function UserList() {
  const { data: users, loading, error } = useFetch(
    "https://jsonplaceholder.typicode.com/users"
  );

  if (loading) return <p>Loading users...</p>;
  if (error)   return <p>Error: {error}</p>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name} — {user.email}</li>
      ))}
    </ul>
  );
}
```

---

### `useDebounce` — Delay Fast-Changing Values

Useful when you want to wait until the user stops typing before firing an API call:

```jsx
import { useState, useEffect } from 'react';

function useDebounce(value, delay = 500) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer); // Cleanup resets the timer on each keystroke
  }, [value, delay]);

  return debouncedValue;
}

// Usage — only fires the search after the user stops typing for 500ms
function SearchPage() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 500);

  const { data, loading } = useFetch(
    debouncedQuery
      ? `https://jsonplaceholder.typicode.com/posts?title=${debouncedQuery}`
      : null
  );

  return (
    <div>
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search posts..."
      />
      {loading && <p>Searching...</p>}
      {data && <p>{data.length} results found</p>}
    </div>
  );
}
```

---

### `useWindowSize` — Track Browser Window Dimensions

```jsx
import { useState, useEffect } from 'react';

function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    function handleResize() {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }

    window.addEventListener("resize", handleResize);

    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return size;
}

// Usage
function ResponsiveLayout() {
  const { width } = useWindowSize();

  return (
    <div>
      <p>Window width: {width}px</p>
      {width < 768 ? <MobileNav /> : <DesktopNav />}
    </div>
  );
}
```

---

### Rules of Hooks

All hooks — built-in and custom — must follow these two rules:

**Rule 1: Only call hooks at the top level**

Never call hooks inside loops, conditions, or nested functions. React relies on the order of hook calls to be consistent on every render.

```jsx
// ❌ Wrong — hook inside a condition
if (isLoggedIn) {
  const [data, setData] = useState(null); // Breaks hook order
}

// ✅ Correct — always at the top level
const [data, setData] = useState(null);
// Use the condition inside the hook or handler
```

**Rule 2: Only call hooks inside React functions**

Call hooks only inside functional components or other custom hooks. Never in regular JS functions, class components, or event handlers.

```jsx
// ❌ Wrong — hook in a regular function
function regularFunction() {
  const [state, setState] = useState(0); // Not allowed
}

// ✅ Correct — hook in a component or custom hook
function MyComponent() {
  const [state, setState] = useState(0); // Fine
}
```

---

## 4. Conditional Rendering

### What is Conditional Rendering?

Conditional rendering means showing different UI based on a condition — just like an `if` statement in regular JavaScript. React doesn't have special template directives for this; you use regular JavaScript logic inside JSX.

---

### Method 1 — `if` / `else` Statements

Use a standard `if` statement before the return for complex conditions:

```jsx
function Dashboard({ isLoggedIn, user }) {
  if (!isLoggedIn) {
    return <LoginPage />;
  }

  if (!user) {
    return <p>Loading user data...</p>;
  }

  return <UserDashboard user={user} />;
}
```

This is the clearest approach when you have multiple conditions or complex logic.

---

### Method 2 — Ternary Operator `? :`

Best for simple two-way conditions inline in JSX:

```jsx
function StatusBadge({ isActive }) {
  return (
    <span style={{ color: isActive ? "green" : "red" }}>
      {isActive ? "Active" : "Inactive"}
    </span>
  );
}
```

```jsx
function App({ isLoading, data }) {
  return (
    <div>
      {isLoading ? (
        <Spinner />
      ) : (
        <DataTable data={data} />
      )}
    </div>
  );
}
```

---

### Method 3 — Logical AND `&&`

Use `&&` when you want to render something or render nothing at all:

```jsx
function Notification({ message, show }) {
  return (
    <div>
      <h1>Dashboard</h1>
      {show && <p className="alert">{message}</p>}
    </div>
  );
}
```

> ⚠️ **Watch out for the zero problem!** If the left side is `0` (falsy), React will actually render `0` instead of nothing.

```jsx
// ❌ Bug — renders "0" when items.length is 0
{items.length && <List items={items} />}

// ✅ Fix — convert to a boolean explicitly
{items.length > 0 && <List items={items} />}
```

---

### Method 4 — Nullish Coalescing and Early Returns

Return `null` from a component to render nothing:

```jsx
function ErrorMessage({ error }) {
  if (!error) return null; // Renders nothing

  return (
    <div className="error-box">
      <p>{error}</p>
    </div>
  );
}
```

---

### Rendering Loading, Error, and Empty States

A real-world pattern that combines all of the above:

```jsx
function PostFeed() {
  const { data: posts, loading, error } = useFetch(
    "https://jsonplaceholder.typicode.com/posts"
  );

  // Loading state
  if (loading) {
    return (
      <div className="skeleton">
        <p>Loading posts...</p>
      </div>
    );
  }

  // Error state
  if (error) {
    return (
      <div className="error">
        <p>Something went wrong: {error}</p>
        <button onClick={() => window.location.reload()}>Retry</button>
      </div>
    );
  }

  // Empty state
  if (!posts || posts.length === 0) {
    return <p>No posts found.</p>;
  }

  // Success state
  return (
    <ul>
      {posts.map((post) => (
        <li key={post.id}>
          <h3>{post.title}</h3>
          <p>{post.body}</p>
        </li>
      ))}
    </ul>
  );
}
```

---

### Combining Everything — Tabbed UI

A practical example using `useState` + conditional rendering:

```jsx
import { useState } from 'react';

const TABS = ["Profile", "Posts", "Settings"];

function TabbedLayout() {
  const [activeTab, setActiveTab] = useState("Profile");

  return (
    <div>
      <nav style={{ display: "flex", gap: 8 }}>
        {TABS.map((tab) => (
          <button
            key={tab}
            onClick={() => setActiveTab(tab)}
            style={{
              fontWeight: activeTab === tab ? "bold" : "normal",
              borderBottom: activeTab === tab ? "2px solid blue" : "none",
            }}
          >
            {tab}
          </button>
        ))}
      </nav>

      <div style={{ marginTop: 16 }}>
        {activeTab === "Profile"  && <ProfileTab />}
        {activeTab === "Posts"    && <PostsTab />}
        {activeTab === "Settings" && <SettingsTab />}
      </div>
    </div>
  );
}

function ProfileTab()  { return <p>Your profile information.</p>; }
function PostsTab()    { return <p>Your posts will appear here.</p>; }
function SettingsTab() { return <p>Account settings.</p>; }
```

---

## Week 2 Project — GitHub User Search

Build a GitHub profile search app using the real GitHub API. This project exercises `useEffect` for data fetching, error and loading states, `useRef` for input focus, a `useFetch` custom hook, and conditional rendering.

### Features
- Search for any GitHub username
- Display avatar, name, bio, public repos, followers, and following
- Show a loading spinner while fetching
- Show a friendly error message for 404 (user not found)
- Auto-focus the search input on page load

### Full Code

```jsx
// hooks/useFetch.js
import { useState, useEffect } from 'react';

export function useFetch(url) {
  const [data, setData]       = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError]     = useState(null);

  useEffect(() => {
    if (!url) return;

    const controller = new AbortController();
    setLoading(true);
    setError(null);
    setData(null);

    async function fetchData() {
      try {
        const res = await fetch(url, { signal: controller.signal });
        if (res.status === 404) throw new Error("User not found.");
        if (!res.ok) throw new Error("Something went wrong. Try again.");
        const json = await res.json();
        setData(json);
      } catch (err) {
        if (err.name !== "AbortError") setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    fetchData();
    return () => controller.abort();
  }, [url]);

  return { data, loading, error };
}
```

```jsx
// App.jsx
import { useState, useRef, useEffect } from 'react';
import { useFetch } from './hooks/useFetch';

function Spinner() {
  return <p style={{ color: "#888" }}>Searching...</p>;
}

function GitHubCard({ user }) {
  return (
    <div style={{
      border: "1px solid #e1e4e8",
      borderRadius: 12,
      padding: 24,
      maxWidth: 480,
      marginTop: 24,
      display: "flex",
      gap: 20,
      alignItems: "flex-start"
    }}>
      <img
        src={user.avatar_url}
        alt={user.login}
        style={{ width: 80, height: 80, borderRadius: "50%" }}
      />
      <div>
        <h2 style={{ margin: 0 }}>{user.name || user.login}</h2>
        <p style={{ color: "#586069", margin: "4px 0" }}>@{user.login}</p>
        {user.bio && <p>{user.bio}</p>}
        <div style={{ display: "flex", gap: 24, marginTop: 8 }}>
          <Stat label="Repos"     value={user.public_repos} />
          <Stat label="Followers" value={user.followers} />
          <Stat label="Following" value={user.following} />
        </div>
        <a
          href={user.html_url}
          target="_blank"
          rel="noreferrer"
          style={{ display: "inline-block", marginTop: 12, color: "#0366d6" }}
        >
          View on GitHub →
        </a>
      </div>
    </div>
  );
}

function Stat({ label, value }) {
  return (
    <div>
      <strong>{value}</strong>
      <p style={{ margin: 0, fontSize: 12, color: "#586069" }}>{label}</p>
    </div>
  );
}

export default function App() {
  const [username, setUsername] = useState("");
  const [searchUrl, setSearchUrl] = useState(null);
  const inputRef = useRef(null);

  const { data: user, loading, error } = useFetch(searchUrl);

  // Auto-focus the input when the page loads
  useEffect(() => {
    inputRef.current.focus();
  }, []);

  // Update document title when a user is found
  useEffect(() => {
    if (user) {
      document.title = `${user.name || user.login} — GitHub Search`;
    } else {
      document.title = "GitHub User Search";
    }
  }, [user]);

  function handleSearch(e) {
    e.preventDefault();
    if (!username.trim()) return;
    setSearchUrl(`https://api.github.com/users/${username.trim()}`);
  }

  return (
    <div style={{ maxWidth: 600, margin: "60px auto", padding: "0 20px" }}>
      <h1>GitHub User Search</h1>

      <form onSubmit={handleSearch} style={{ display: "flex", gap: 8 }}>
        <input
          ref={inputRef}
          type="text"
          value={username}
          onChange={(e) => setUsername(e.target.value)}
          placeholder="Enter a GitHub username..."
          style={{ flex: 1, padding: "8px 12px", fontSize: 16, borderRadius: 6, border: "1px solid #ccc" }}
        />
        <button
          type="submit"
          disabled={loading}
          style={{ padding: "8px 20px", fontSize: 16, borderRadius: 6, cursor: "pointer" }}
        >
          Search
        </button>
      </form>

      {loading && <Spinner />}
      {error   && <p style={{ color: "red", marginTop: 16 }}>❌ {error}</p>}
      {user    && <GitHubCard user={user} />}

      {!loading && !error && !user && (
        <p style={{ color: "#888", marginTop: 24 }}>
          Type a GitHub username above and press Search.
        </p>
      )}
    </div>
  );
}
```

---

## Weekly Checklist

- [ ] Understand the 3 forms of the dependency array in `useEffect`
- [ ] Fetch data from an API inside `useEffect`
- [ ] Use `async/await` correctly inside `useEffect`
- [ ] Add a cleanup function using `AbortController`
- [ ] Use `useRef` to focus a DOM element
- [ ] Use `useRef` to store a timer ID
- [ ] Build the `useFetch` custom hook from scratch
- [ ] Build the `useDebounce` custom hook
- [ ] Render loading, error, and empty states conditionally
- [ ] Complete the GitHub User Search project

---

## 📚 Resources

| Resource | Description |
|----------|-------------|
| [react.dev — useEffect](https://react.dev/reference/react/useEffect) | Official `useEffect` deep-dive |
| [react.dev — useRef](https://react.dev/reference/react/useRef) | Official `useRef` reference |
| [react.dev — Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks) | Guide to writing your own hooks |
| [TkDodo's Blog](https://tkdodo.eu/blog) | Best practices for data fetching in React |
| [JSONPlaceholder](https://jsonplaceholder.typicode.com) | Free fake REST API for practice |
| [GitHub REST API](https://docs.github.com/en/rest) | Real API used in the Week 2 project |

---

> **Up next → Week 3: State Management** — Scale your app with `useReducer`, share global state with Context API, navigate between pages with React Router, and build validated forms.

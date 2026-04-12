# ⚛️ Week 1 — React Foundations

> **Goal:** Understand how React works under the hood and get comfortable writing JSX, building components, managing state, and handling user events.

---

## 📋 Table of Contents

1. [JSX & Rendering](#1-jsx--rendering)
2. [Components & Props](#2-components--props)
3. [State & useState](#3-state--usestate)
4. [Event Handling](#4-event-handling)
5. [Week 1 Project — Weather Card UI](#week-1-project--weather-card-ui)
6. [Weekly Checklist](#weekly-checklist)

---

## 1. JSX & Rendering

### What is JSX?

JSX (JavaScript XML) is a syntax extension for JavaScript that lets you write HTML-like code inside `.jsx` or `.js` files. It is **not** valid JavaScript on its own — a tool called **Babel** compiles it into regular JavaScript before the browser runs it.

```jsx
// What you write (JSX)
const element = <h1>Hello, React!</h1>;

// What Babel compiles it to (plain JS)
const element = React.createElement("h1", null, "Hello, React!");
```

You never need to call `React.createElement` manually — JSX handles that for you. But understanding this helps you know what's actually happening behind the scenes.

---

### JSX Rules

JSX looks like HTML but has a few important differences:

**1. Return a single root element**

Every component must return one parent element. Use a `<div>` or a Fragment (`<>`) to wrap multiple elements.

```jsx
// ❌ Wrong — two root elements
return (
  <h1>Title</h1>
  <p>Paragraph</p>
);

// ✅ Correct — wrapped in a fragment
return (
  <>
    <h1>Title</h1>
    <p>Paragraph</p>
  </>
);
```

**2. Use `className` instead of `class`**

Since `class` is a reserved word in JavaScript, JSX uses `className`.

```jsx
// ❌ Wrong
<div class="container">

// ✅ Correct
<div className="container">
```

**3. Self-close empty tags**

Tags with no children must be self-closed with a slash.

```jsx
// ❌ Wrong
<img src="photo.jpg">
<input type="text">

// ✅ Correct
<img src="photo.jpg" />
<input type="text" />
```

**4. Use curly braces `{}` for JavaScript expressions**

Anything inside `{}` is evaluated as JavaScript.

```jsx
const name = "Alice";
const age = 25;

return (
  <p>
    My name is {name} and I am {age * 2} years old in two years.
  </p>
);
```

---

### How React Renders to the DOM

React uses a **Virtual DOM** — a lightweight JavaScript copy of the real DOM. When your data changes, React:

1. Rebuilds the Virtual DOM
2. Compares it with the previous version (this is called **diffing**)
3. Updates only the parts of the real DOM that actually changed

This makes React very fast compared to directly manipulating the DOM yourself.

Your entire React app is mounted inside a single HTML element — usually `<div id="root">` in `index.html`:

```jsx
// main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

---

## 2. Components & Props

### What is a Component?

A component is a reusable, self-contained piece of UI. Think of it like a custom HTML tag that you define yourself. Every React app is built by composing components together.

A **functional component** is simply a JavaScript function that returns JSX:

```jsx
function Greeting() {
  return <h1>Hello, World!</h1>;
}
```

Component names **must start with a capital letter**. React uses this to tell components apart from regular HTML tags (`<div>` vs `<MyDiv>`).

---

### Props (Properties)

Props are how you pass data **into** a component — like arguments to a function. They make components reusable.

```jsx
// Defining a component that accepts props
function Greeting(props) {
  return <h1>Hello, {props.name}!</h1>;
}

// Using it with different prop values
function App() {
  return (
    <>
      <Greeting name="Alice" />
      <Greeting name="Bob" />
      <Greeting name="Charlie" />
    </>
  );
}
```

**Output:**
```
Hello, Alice!
Hello, Bob!
Hello, Charlie!
```

---

### Destructuring Props

Instead of writing `props.name` everywhere, you can destructure props directly in the function signature:

```jsx
// Without destructuring
function UserCard(props) {
  return (
    <div>
      <h2>{props.name}</h2>
      <p>{props.role}</p>
      <p>{props.email}</p>
    </div>
  );
}

// With destructuring (cleaner)
function UserCard({ name, role, email }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{role}</p>
      <p>{email}</p>
    </div>
  );
}

// Usage
<UserCard name="Alice" role="Developer" email="alice@example.com" />
```

---

### Default Props

You can set fallback values for props that might not be passed:

```jsx
function Button({ label = "Click me", color = "blue" }) {
  return <button style={{ backgroundColor: color }}>{label}</button>;
}

// Uses defaults
<Button />

// Overrides defaults
<Button label="Submit" color="green" />
```

---

### Props are Read-Only

A component must **never modify its own props**. Props flow one way — from parent to child. This is called **unidirectional data flow** and is a core React principle.

```jsx
// ❌ Never do this
function BadComponent(props) {
  props.name = "Changed"; // This will cause an error
  return <p>{props.name}</p>;
}
```

---

### Composing Components

Real apps are built by nesting components inside each other:

```jsx
function Avatar({ src, alt }) {
  return <img src={src} alt={alt} style={{ borderRadius: "50%", width: 60 }} />;
}

function UserInfo({ name, title }) {
  return (
    <div>
      <h3>{name}</h3>
      <p>{title}</p>
    </div>
  );
}

function ProfileCard({ user }) {
  return (
    <div className="card">
      <Avatar src={user.avatar} alt={user.name} />
      <UserInfo name={user.name} title={user.title} />
    </div>
  );
}

// Usage
const user = { name: "Alice", title: "Engineer", avatar: "/alice.jpg" };
<ProfileCard user={user} />
```

---

### The `children` Prop

Every component automatically receives a special `children` prop — whatever is placed between its opening and closing tags:

```jsx
function Card({ children }) {
  return (
    <div className="card" style={{ border: "1px solid #ccc", padding: 16 }}>
      {children}
    </div>
  );
}

// Usage — any JSX between the tags becomes `children`
<Card>
  <h2>My Title</h2>
  <p>Some content here.</p>
</Card>
```

---

## 3. State & useState

### What is State?

**Props** are data passed *into* a component from outside. **State** is data that a component manages *itself* internally. When state changes, React automatically re-renders the component to show the new value.

Think of state as a component's memory.

---

### The `useState` Hook

`useState` is a built-in React hook that adds state to a functional component. It returns two things: the current state value, and a function to update it.

```jsx
import { useState } from 'react';

function Counter() {
  // Declare a state variable called "count", starting at 0
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

The pattern `const [value, setValue] = useState(initialValue)` uses **array destructuring**. The names `value` and `setValue` are up to you — by convention the setter is always `set` + the variable name.

---

### Never Mutate State Directly

Always use the setter function. Never modify the state variable directly — React won't know the state changed and won't re-render.

```jsx
// ❌ Wrong — React won't re-render
count = count + 1;

// ✅ Correct — triggers a re-render
setCount(count + 1);
```

---

### State with Objects

When your state is an object, you must spread the existing values and only change what you need:

```jsx
function UserForm() {
  const [user, setUser] = useState({ name: "", email: "" });

  function updateName(newName) {
    // ❌ Wrong — overwrites the entire object, losing email
    setUser({ name: newName });

    // ✅ Correct — spreads existing fields and only updates name
    setUser({ ...user, name: newName });
  }

  return (
    <input
      value={user.name}
      onChange={(e) => setUser({ ...user, name: e.target.value })}
    />
  );
}
```

---

### State with Arrays

Similarly, never mutate arrays in state. Use methods that return new arrays:

```jsx
function TodoList() {
  const [todos, setTodos] = useState(["Buy groceries", "Read a book"]);

  function addTodo(newTodo) {
    // ✅ Spread creates a new array
    setTodos([...todos, newTodo]);
  }

  function removeTodo(index) {
    // ✅ filter creates a new array
    setTodos(todos.filter((_, i) => i !== index));
  }

  return (
    <ul>
      {todos.map((todo, i) => (
        <li key={i}>
          {todo}
          <button onClick={() => removeTodo(i)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

---

### The `key` Prop in Lists

When rendering a list with `.map()`, React needs a unique `key` prop on each item. This helps React identify which items changed, were added, or removed.

```jsx
const fruits = ["Apple", "Banana", "Cherry"];

// ❌ Avoid using the index as key when the list can be reordered
fruits.map((fruit, i) => <li key={i}>{fruit}</li>)

// ✅ Use a unique, stable ID when possible
const items = [{ id: 1, name: "Apple" }, { id: 2, name: "Banana" }];
items.map((item) => <li key={item.id}>{item.name}</li>)
```

---

### Functional Updates

When a new state value depends on the old one, use the **functional form** of the setter. This is safer when updates happen rapidly (like in quick button clicks):

```jsx
// ❌ Could cause bugs with rapid updates
setCount(count + 1);

// ✅ Always gets the most recent value
setCount(prevCount => prevCount + 1);
```

---

### Multiple State Variables

You can call `useState` multiple times — each call manages a completely separate piece of state:

```jsx
function LoginForm() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);

  // Each state variable updates independently
}
```

---

## 4. Event Handling

### Listening to Events

In React you attach event listeners directly in JSX using **camelCase** attribute names. You pass a function reference — not a function call.

```jsx
function Button() {
  function handleClick() {
    alert("Button was clicked!");
  }

  return (
    // ✅ Pass the function reference
    <button onClick={handleClick}>Click me</button>

    // ❌ Don't call it — this runs immediately when the component renders
    // <button onClick={handleClick()}>Click me</button>
  );
}
```

---

### Common Event Types

| JSX Event | Fires When |
|-----------|------------|
| `onClick` | Element is clicked |
| `onChange` | Input value changes |
| `onSubmit` | Form is submitted |
| `onKeyDown` | Keyboard key is pressed |
| `onMouseEnter` | Mouse enters an element |
| `onFocus` | Element receives focus |
| `onBlur` | Element loses focus |

---

### The Event Object

React event handlers automatically receive a **synthetic event** object — a cross-browser wrapper around the native browser event. It works the same in all browsers.

```jsx
function SearchInput() {
  function handleChange(event) {
    console.log(event.target.value); // Current value of the input
    console.log(event.target.name);  // Name attribute of the input
  }

  return <input type="text" onChange={handleChange} />;
}
```

---

### Controlled Inputs

A **controlled input** is one whose value is bound to state. React is the single source of truth for the input's value:

```jsx
function SearchBar() {
  const [query, setQuery] = useState("");

  return (
    <div>
      <input
        type="text"
        value={query}                          // Bound to state
        onChange={(e) => setQuery(e.target.value)} // Updates state on every keystroke
        placeholder="Search..."
      />
      <p>You typed: {query}</p>
    </div>
  );
}
```

---

### Preventing Default Behavior

For forms, call `event.preventDefault()` to stop the browser from reloading the page on submit:

```jsx
function LoginForm() {
  const [email, setEmail] = useState("");

  function handleSubmit(event) {
    event.preventDefault(); // Stop page reload
    console.log("Submitted with email:", email);
    // Send data to your API here
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

---

### Passing Arguments to Handlers

When you need to pass extra data to an event handler (like an item ID), use an arrow function wrapper:

```jsx
function ProductList() {
  const products = [
    { id: 1, name: "Laptop" },
    { id: 2, name: "Phone" },
  ];

  function handleDelete(id) {
    console.log("Deleting product:", id);
  }

  return (
    <ul>
      {products.map((product) => (
        <li key={product.id}>
          {product.name}
          {/* Arrow function lets us pass the id argument */}
          <button onClick={() => handleDelete(product.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

---

### Putting It All Together — Interactive Counter

Here is a complete example combining JSX, components, state, and events:

```jsx
import { useState } from 'react';

function Counter({ initialValue = 0, step = 1 }) {
  const [count, setCount] = useState(initialValue);

  function increment() {
    setCount(prev => prev + step);
  }

  function decrement() {
    setCount(prev => prev - step);
  }

  function reset() {
    setCount(initialValue);
  }

  return (
    <div style={{ textAlign: "center", padding: 24 }}>
      <h2 style={{ fontSize: 48 }}>{count}</h2>
      <div style={{ display: "flex", gap: 8, justifyContent: "center" }}>
        <button onClick={decrement}>- {step}</button>
        <button onClick={reset}>Reset</button>
        <button onClick={increment}>+ {step}</button>
      </div>
    </div>
  );
}

function App() {
  return (
    <div>
      <h1>Counter Demo</h1>
      <Counter initialValue={0} step={1} />
      <Counter initialValue={10} step={5} />
    </div>
  );
}

export default App;
```

---

## Week 1 Project — Weather Card UI

Build a static weather dashboard using only what you've learned this week. No API calls yet — use hardcoded data.

### Requirements

- A `WeatherCard` component that accepts `city`, `temperature`, `condition`, and `humidity` as props
- A `WeatherIcon` component that renders a different emoji based on the `condition` prop (`"sunny"`, `"cloudy"`, `"rainy"`)
- A `WeatherDashboard` component that renders 3 `WeatherCard` components with different data
- A toggle button that switches between Celsius and Fahrenheit (use `useState` + conversion logic)

### Starter Code

```jsx
// App.jsx
import { useState } from 'react';

const weatherData = [
  { city: "Dhaka",    temp: 32, condition: "sunny",  humidity: 70 },
  { city: "London",   temp: 14, condition: "cloudy",  humidity: 85 },
  { city: "New York", temp: 22, condition: "rainy",   humidity: 60 },
];

function WeatherIcon({ condition }) {
  const icons = { sunny: "☀️", cloudy: "☁️", rainy: "🌧️" };
  return <span style={{ fontSize: 40 }}>{icons[condition]}</span>;
}

function WeatherCard({ city, temp, condition, humidity, unit }) {
  const displayTemp = unit === "F" ? Math.round(temp * 9/5 + 32) : temp;

  return (
    <div style={{ border: "1px solid #ccc", borderRadius: 8, padding: 20, width: 200 }}>
      <WeatherIcon condition={condition} />
      <h3>{city}</h3>
      <p style={{ fontSize: 32, fontWeight: "bold" }}>
        {displayTemp}°{unit}
      </p>
      <p>Humidity: {humidity}%</p>
    </div>
  );
}

function App() {
  const [unit, setUnit] = useState("C");

  return (
    <div style={{ padding: 40 }}>
      <h1>Weather Dashboard</h1>
      <button onClick={() => setUnit(unit === "C" ? "F" : "C")}>
        Switch to °{unit === "C" ? "F" : "C"}
      </button>
      <div style={{ display: "flex", gap: 16, marginTop: 24 }}>
        {weatherData.map((data) => (
          <WeatherCard key={data.city} {...data} unit={unit} />
        ))}
      </div>
    </div>
  );
}

export default App;
```

---

## Weekly Checklist

Use this to track your progress through Week 1:

- [ ] Set up a new Vite React project (`npm create vite@latest my-app -- --template react`)
- [ ] Understand what JSX compiles to
- [ ] Build and render a functional component
- [ ] Pass at least 4 different prop types (string, number, boolean, function)
- [ ] Use `useState` to manage a counter
- [ ] Build a controlled text input
- [ ] Handle a form `onSubmit` with `preventDefault`
- [ ] Render a list with `.map()` and correct `key` props
- [ ] Complete the Weather Card UI project
- [ ] Push your code to a GitHub repository

---

## 📚 Resources

| Resource | Description |
|----------|-------------|
| [react.dev/learn](https://react.dev/learn) | Official interactive React tutorial — start here |
| [Scrimba React Course](https://scrimba.com/learn/learnreact) | Hands-on video + code exercises |
| [CodeSandbox](https://codesandbox.io) | In-browser React playground, no setup needed |
| [React DevTools](https://react.dev/learn/react-developer-tools) | Browser extension to inspect components and state |
| [javascript.info](https://javascript.info) | Brush up on ES6+ (destructuring, spread, arrow functions) |

---

> **Up next → Week 2: Hooks & Effects** — Learn `useEffect` for data fetching, `useRef` for DOM access, and how to write your own custom hooks.

#  1-Month React Learning Plan

A structured, project-driven curriculum to go from React beginner to job-ready developer in 4 weeks.

---

## 📅 Overview

| Week | Theme | Project |
|------|-------|---------|
| Week 1 | Foundations | Weather Card UI |
| Week 2 | Hooks & Effects | GitHub User Search |
| Week 3 | State Management | Shopping Cart |
| Week 4 | Real-world React | Full Todo App (deployed) |

---

## Week 1 — Foundations

**Goal:** Get comfortable with JSX, components, props, and state.

### Topics
- **JSX & Rendering** — How JSX compiles to `React.createElement` calls
- **Components** — Functional components, props, and composing UIs
- **State & useState** — Local state, state updates, and re-rendering triggers
- **Event Handling** — `onClick`, `onChange`, and synthetic events

### Checklist
- [ ] Install Node.js & scaffold a project with Vite (`npm create vite@latest`)
- [ ] Build your first functional component
- [ ] Pass props between 3 nested components
- [ ] Toggle state with a button

### Project: Weather Card UI
A static component tree that displays weather data. Focus on clean props passing and composing multiple components.

### Resources
- [react.dev](https://react.dev) — official docs
- [Scrimba React course](https://scrimba.com/learn/learnreact)
- [CodeSandbox](https://codesandbox.io) — in-browser playground

---

## Week 2 — Hooks & Effects

**Goal:** Manage side effects, fetch data, and write reusable custom hooks.

### Topics
- **useEffect** — Fetching data, subscriptions, and cleanup functions
- **useRef** — DOM access and persisting values without re-render
- **Custom Hooks** — Extract reusable stateful logic into `useFoo` patterns
- **Conditional Rendering** — `&&`, ternary, and early returns inside JSX

### Checklist
- [ ] Fetch data with `useEffect` and include a cleanup function
- [ ] Build a `useFetch` custom hook
- [ ] Handle loading and error states
- [ ] Use `useRef` to programmatically focus an input

### Project: GitHub User Search
Search the GitHub API by username. Displays avatar, bio, and repo count. Handles loading spinners and 404 errors.

### Resources
- [react.dev — Hooks reference](https://react.dev/reference/react)
- [TkDodo's React Query Blog](https://tkdodo.eu/blog)
- [JSONPlaceholder](https://jsonplaceholder.typicode.com) — free mock API

---

## Week 3 — State Management

**Goal:** Scale state across your app with reducers, context, and routing.

### Topics
- **useReducer** — Complex state logic with `dispatch` and action patterns
- **Context API** — Share state globally without prop drilling
- **React Router** — Client-side routing with routes, links, and URL params
- **Forms & Validation** — Controlled inputs, `onSubmit`, and basic validation

### Checklist
- [ ] Refactor a `useState` implementation to `useReducer`
- [ ] Create a global theme context (light/dark mode)
- [ ] Set up 3 routes with React Router v6
- [ ] Build a validated checkout form

### Project: Shopping Cart
Browse a product list, add/remove items, update quantities, and navigate to a checkout page. State managed with `useReducer` and Context.

### Resources
- [React Router v6 docs](https://reactrouter.com)
- [TanStack Query](https://tanstack.com/query) — intro reading
- [Zod](https://zod.dev) — schema-based validation

---

## Week 4 — Real-world React

**Goal:** Optimize performance, write tests, add TypeScript, and ship your app.

### Topics
- **Performance** — `useMemo`, `useCallback`, `React.memo`, and lazy loading
- **Testing** — React Testing Library fundamentals and `userEvent`
- **TypeScript Basics** — Typing props, state, and event handlers
- **Deployment** — Build pipelines, Vercel / Netlify, environment variables

### Checklist
- [ ] Memoize an expensive computation with `useMemo`
- [ ] Write 3 component tests with React Testing Library
- [ ] Add TypeScript to your project (`tsc --init`)
- [ ] Deploy your app live on Vercel or Netlify

### Project: Full Todo App
A complete CRUD application with local persistence, unit tests, TypeScript types, and a live deployment link. The capstone of the curriculum.

### Resources
- [Vitest + RTL docs](https://vitest.dev)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [Vercel getting started](https://vercel.com/docs)

---

## 🛠 Prerequisites

- Basic HTML, CSS, and JavaScript (ES6+)
- Node.js v18+ installed
- A code editor (VS Code recommended)

## 💡 Tips for Success

1. **Code every day** — even 30 minutes of consistent practice beats long weekend sessions.
2. **Build the projects** — reading docs is not enough; the projects are where understanding solidifies.
3. **Don't skip Week 4** — testing and deployment are what separate hobby projects from professional work.
4. **Read error messages** — React's error messages are genuinely helpful. Treat them as learning material.
5. **Use the React DevTools** — install the browser extension early and use it constantly.

---

## 📁 Suggested Folder Structure

```
my-react-journey/
├── week-1-foundations/
│   └── weather-card/
├── week-2-hooks/
│   └── github-search/
├── week-3-state/
│   └── shopping-cart/
└── week-4-production/
    └── todo-app/
```

---

*Happy coding! By the end of Week 4, you'll have 4 real projects and the foundations to build anything with React.*

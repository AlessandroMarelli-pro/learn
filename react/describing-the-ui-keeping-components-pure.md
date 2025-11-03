# Keeping Components Pure

## Overview
Some JavaScript functions are "pure," meaning they only perform calculations and nothing more. By keeping your components pure, you can avoid bugs and unpredictable behavior as your codebase grows. React is designed around this concept.

## What You Will Learn
- Understanding purity and how it prevents bugs
- Maintaining component purity by avoiding changes during rendering
- Using Strict Mode to identify component violations

---

## What is a Pure Function?

A **pure function** has two essential characteristics:

### 1. "It minds its own business"
The function doesn't change any objects or variables that existed before it was called.

### 2. "Same inputs, same output"
Given the same inputs, a pure function always returns the same result.

**Mathematical analogy:** Consider the formula `y = 2x`
- If `x = 2`, then `y` will always be `4`
- If `x = 3`, then `y` will always be `6`
- Predictable and consistent

### Pure Function Example
```jsx
function double(number) {
  return 2 * number;
}
```

This is pure because:
- It doesn't modify anything outside itself
- `double(3)` always returns `6`

---

## React Components Should Be Pure

**React assumes all components are pure functions.**

Given the same inputs (props), a component must always return the same JSX.

### Pure Component Example
```jsx
function Recipe({ drinkers }) {
  return (
    <ol>
      <li>Boil {drinkers} cups of water.</li>
      <li>Add {drinkers} spoons of tea and {0.5 * drinkers} spoons of spice.</li>
      <li>Add {0.5 * drinkers} cups of milk to boil and sugar to taste.</li>
    </ol>
  );
}
```

**Why it's pure:**
- Calling `Recipe` with `drinkers={2}` always produces the same JSX
- Calling `Recipe` with `drinkers={4}` always produces the same JSX
- No external state is modified

### Impure Component Example

```jsx
let guest = 0;

function Cup() {
  // ❌ Bad: modifying a preexisting variable
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```

**Why it's impure:**
- Reading and writing a variable (`guest`) declared outside the component
- Each call produces different results
- The component's behavior depends on the order and frequency of calls

> **⚠️ PITFALL**
>
> **Never modify preexisting variables during render!** This makes components unpredictable. Calling the component multiple times produces different results, and other components reading the same variable get inconsistent data.

---

## Side Effects: Changes "On the Side"

**Side effects** are changes that happen outside of the component's calculation:
- Updating the screen
- Starting an animation
- Changing data
- Making network requests
- Logging to console

### Side Effects Belong in Event Handlers

**Event handlers** are functions that run in response to user actions (clicks, typing, etc.). Even though they're defined inside components, **event handlers don't need to be pure**.

```jsx
function AddToCart({ product }) {
  const handleClick = () => {
    // ✅ Side effects are fine in event handlers
    addToShoppingCart(product);
    showNotification('Added to cart!');
  };

  return <button onClick={handleClick}>Add to Cart</button>;
}
```

### When You Can't Use Event Handlers: useEffect

If you can't find an appropriate event handler for a side effect, use `useEffect` as a **last resort**. This tells React to execute it after rendering.

```jsx
import { useEffect } from 'react';

function DataLogger({ data }) {
  useEffect(() => {
    // ✅ Runs after render
    logToAnalytics(data);
  }, [data]);

  return <div>{data}</div>;
}
```

**Important:** However, this approach should be your last resort. When possible, express your logic with rendering alone.

---

## Local Mutation: The Component's "Little Secret"

**Local mutation** (modifying variables created during the current render) is perfectly fine.

```jsx
function Cup({ guest }) {
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaGathering() {
  let cups = [];
  // ✅ Good: modifying a variable we just created
  for (let i = 1; i <= 12; i++) {
    cups.push(<Cup key={i} guest={i} />);
  }
  return cups;
}
```

**Why this is okay:**
- The `cups` array is created during the same render
- No code outside this component knows about it
- It's the component's "little secret"

**The key difference:**
- ❌ **Mutating preexisting variables** → Impure
- ✅ **Mutating locally created variables** → Pure

---

## Where You Can Cause Side Effects

Changes to the screen, animations, and data changes are called **side effects**. They happen "on the side," not during rendering.

### Rendering Must Calculate JSX Only

**During rendering, your component should:**
- ✅ Calculate and return JSX
- ✅ Read props, state, and context
- ✅ Create local variables and manipulate them

**During rendering, your component should NOT:**
- ❌ Modify preexisting variables
- ❌ Make network requests
- ❌ Update the DOM directly
- ❌ Set timers
- ❌ Read/write shared state

> **⚠️ PITFALL**
>
> Don't mutate **props**, **state**, or **context**. React treats them as read-only snapshots. Never change them directly—use state setters instead.

---

## Using Strict Mode to Detect Impurity

React offers **Strict Mode** to help find impure components during development.

### How Strict Mode Works

In development, Strict Mode **calls each component function twice**. If a component is impure, calling it twice will produce different results, helping you identify the problem.

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

**What happens:**
- Pure components: Calling twice has no effect (same result both times)
- Impure components: Different results reveal the impurity

**Note:** Strict Mode has no effect in production. It only helps during development.

---

## Benefits of Purity

### 1. Components Can Run Anywhere
Pure components can run in any environment—servers, client devices, anywhere—because they always return the same result for the same input.

### 2. Performance Optimization
React can skip rendering components whose inputs haven't changed. This is safe because pure functions always return the same results.

### 3. Safe to Interrupt Rendering
If data changes during rendering, React can restart rendering without wasting time finishing the outdated calculation.

### 4. Future Features
Purity enables React's future features and optimizations. It's the foundation of React's modern architecture.

---

## Common Violations and Solutions

| Violation | Problem | Solution |
|-----------|---------|----------|
| Modifying global variable | Unpredictable behavior | Pass as prop or use state |
| Mutating props | Breaks React's assumptions | Treat props as read-only |
| Mutating state directly | React won't re-render | Use state setter function |
| Making API calls during render | Inconsistent renders | Use `useEffect` or event handlers |
| DOM manipulation during render | Conflicts with React's rendering | Use refs and `useEffect` |

---

## Practical Examples

### Example 1: Impure (Bad)
```jsx
let count = 0;

function Counter() {
  // ❌ Modifying external variable
  count = count + 1;
  return <div>Count: {count}</div>;
}
```

### Example 2: Pure (Good)
```jsx
function Counter({ count }) {
  // ✅ Same input always produces same output
  return <div>Count: {count}</div>;
}

// Parent manages state
function App() {
  const [count, setCount] = useState(0);
  return (
    <>
      <Counter count={count} />
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </>
  );
}
```

### Example 3: Local Mutation (Good)
```jsx
function ItemList({ items }) {
  // ✅ Creating and modifying local variable
  const processedItems = [];

  for (const item of items) {
    processedItems.push({
      ...item,
      displayName: item.name.toUpperCase()
    });
  }

  return (
    <ul>
      {processedItems.map(item => (
        <li key={item.id}>{item.displayName}</li>
      ))}
    </ul>
  );
}
```

---

## Interview Questions to Consider

1. **Q: What makes a function pure?**
   - A: A pure function (1) doesn't modify variables outside its scope, and (2) always returns the same output for the same inputs.

2. **Q: Why should React components be pure?**
   - A: Purity enables reliable rendering, performance optimizations, server rendering, and safe render interruption. It prevents bugs and makes components predictable.

3. **Q: Where should side effects go in React?**
   - A: Side effects belong in event handlers (primary choice) or `useEffect` (last resort). Never in the render phase.

4. **Q: What's the difference between mutating a preexisting variable and local mutation?**
   - A: Mutating preexisting variables (defined outside the component) makes it impure. Local mutation (variables created during render) is fine because it's contained within that render.

5. **Q: What does Strict Mode do?**
   - A: In development, Strict Mode calls component functions twice to help detect impurity. Pure components produce the same result both times; impure ones don't.

6. **Q: Can you modify props or state directly?**
   - A: No! Props and state are read-only snapshots. Mutating them breaks React's assumptions. Use state setter functions to update state.

7. **Q: Why can't you make API calls during render?**
   - A: API calls are side effects that can produce different results or modify external state. This makes renders unpredictable and can cause bugs or performance issues.

---

## Key Takeaways

- Pure functions always return the same result for the same inputs and don't modify external state
- **React components should be pure functions** of their props
- Modifying preexisting variables during render makes components impure and unpredictable
- **Side effects belong in event handlers** or `useEffect`, not in render
- Local mutation (modifying variables created during render) is perfectly fine
- Never mutate props, state, or context directly
- Use **Strict Mode** to catch impurity bugs during development
- Purity enables performance optimizations, server rendering, and future React features
- Rendering should calculate JSX—event handlers should change state
- When in doubt: calculate during render, change during events

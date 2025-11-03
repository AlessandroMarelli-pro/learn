# Render and Commit

## Overview
Before your components display on screen, they must be rendered by React. Understanding the steps in this process helps you think about how your code executes and explain its behavior.

## What You Will Learn
- What rendering means in React
- When and why React renders a component
- The steps involved in displaying a component on screen
- Why rendering does not always produce a DOM update

---

## The Restaurant Analogy

Think of React components like a restaurant:
- **Components** are cooks in the kitchen, preparing dishes
- **React** is the waiter, taking orders and delivering food
- **The process** has three steps:

1. **Triggering** a render (taking the order to the kitchen)
2. **Rendering** the component (preparing the order)
3. **Committing** to the DOM (placing the order on the table)

---

## Step 1: Trigger a Render

There are **two reasons** for a component to render:

### Reason 1: Initial Render

When your app starts, you need to trigger the initial render:

```jsx
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

**What happens:**
1. Call `createRoot` with the target DOM node
2. Call the `render` method with your component
3. React displays your component on screen

### Reason 2: Component's State Has Updated

Once a component has been initially rendered, you can trigger additional renders by **updating its state** with a setter function:

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Clicked {count} times
    </button>
  );
}
```

**What happens:**
- Calling `setCount` queues a render
- React re-renders the component with new state
- Like a restaurant guest ordering tea, dessert, etc., after the first order

**State updates trigger renders automatically!**

---

## Step 2: React Renders Your Components

### What is "Rendering"?

**"Rendering" means React calls your component functions** to figure out what to display on screen.

- **On initial render:** React calls the root component
- **On subsequent renders:** React calls the component whose state update triggered the render

### The Process is Recursive

If a component returns other components, React renders those next, continuing until there are no more nested components:

```jsx
function App() {
  return <Gallery />; // React renders Gallery next
}

function Gallery() {
  return (
    <>
      <Profile /> {/* React renders Profile next */}
      <Profile />
      <Profile />
    </>
  );
}
```

React keeps rendering nested components until it reaches the bottom of the tree.

### What React Does During Rendering

**Initial render:**
- React creates DOM nodes for all components
- Uses `document.createElement()` to build the DOM tree

**Re-renders:**
- React calculates which properties changed since the previous render
- Doesn't do anything with that information yet (that's Step 3!)
- Just calculates the differences

> **⚠️ CRITICAL: RENDERING MUST BE PURE**
>
> **Rendering must always be a pure calculation:**
>
> 1. **Same inputs, same output** — Given the same inputs (props, state, context), a component should always return the same JSX.
>
> 2. **It minds its own business** — It should not change any objects or variables that existed before rendering.
>
> **Why this matters:**
> - Breaking these rules leads to confusing bugs and unpredictable behavior
> - React's Strict Mode calls components twice to help catch impure functions
> - Optimization features depend on components being pure
>
> **Examples:**
> ```jsx
> // ✅ Pure: Always returns same output for same input
> function Greeting({ name }) {
>   return <h1>Hello, {name}!</h1>;
> }
>
> // ❌ Impure: Modifies external variable
> let counter = 0;
> function ImpureCounter() {
>   counter++; // Don't do this!
>   return <div>{counter}</div>;
> }
> ```

---

## Step 3: React Commits Changes to the DOM

After rendering (calling your components), React modifies the DOM.

### Initial Render

For the initial render, React uses `appendChild()` DOM API to put all the DOM nodes it created on screen.

### Re-renders

For re-renders, React applies the **minimal necessary operations** to make the DOM match the latest rendering output.

**Key principle:**
**"React only changes the DOM nodes if there's a difference between renders."**

### Example: Input Value Persists

```jsx
function Clock({ time }) {
  return (
    <>
      <h1>{time}</h1>
      <input />
    </>
  );
}
```

**What happens:**
- Every second, `time` prop changes, triggering a re-render
- React updates the `<h1>` content (because it changed)
- React **does not touch** the `<input>` (because it didn't change)
- Text you type in the input stays there!

**Why:** React compares the new and old render outputs. Since `<input />` appears identically in both, React doesn't recreate or reset it.

---

## Step 4: Browser Paint

After React commits changes to the DOM, the browser repaints the screen.

**Terminology note:**
- **"Rendering"** in React = calling your component functions
- **"Painting"** = browser drawing pixels on screen

These are different steps! React rendering ≠ browser painting.

---

## Why Rendering Doesn't Always Update the DOM

**Important concept:** A render doesn't guarantee a DOM update!

**Scenario:**
```jsx
function App() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(0)}>Reset</button>
      <p>Count: {count}</p>
    </div>
  );
}
```

If `count` is already `0` and you click "Reset":
1. ✅ Render is **triggered** (state setter called)
2. ✅ React **renders** the component (calls `App()`)
3. ❌ React **skips commit** (output is identical to previous render)

**Result:** No DOM changes, better performance!

---

## The Complete Flow

### Initial Render Flow

1. **User opens app**
2. **Trigger:** `root.render(<App />)` is called
3. **Render:** React calls `App()` and all nested components
4. **Commit:** React creates DOM nodes with `appendChild()`
5. **Paint:** Browser displays the UI

### State Update Flow

1. **User clicks button**
2. **Trigger:** State setter function is called (e.g., `setCount(1)`)
3. **Render:** React calls the component again with new state
4. **Commit:** React calculates differences and updates only changed DOM nodes
5. **Paint:** Browser repaints the updated parts

---

## Practical Examples

### Example 1: Re-render Without DOM Update

```jsx
function Message({ text }) {
  console.log('Rendering Message'); // Logs on every render

  return (
    <div>
      <p>{text}</p>
      <input placeholder="Type here..." />
    </div>
  );
}

function App() {
  const [count, setCount] = useState(0);

  return (
    <>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <Message text="Hello" />
    </>
  );
}
```

**What happens when you click the button:**
- `App` re-renders (state changed)
- `Message` re-renders (parent re-rendered)
- Console logs "Rendering Message"
- But the DOM for `<Message>` doesn't change (same output)
- Your input text persists!

### Example 2: Partial DOM Updates

```jsx
function Clock() {
  const [time, setTime] = useState(new Date().toLocaleTimeString());

  useEffect(() => {
    const interval = setInterval(() => {
      setTime(new Date().toLocaleTimeString());
    }, 1000);
    return () => clearInterval(interval);
  }, []);

  return (
    <>
      <h1>Current time: {time}</h1>
      <input />
    </>
  );
}
```

**What happens every second:**
1. State updates (new time)
2. React renders `Clock` component
3. React compares old and new output
4. Only the `<h1>` text node is updated
5. `<input>` is unchanged, preserving user input

---

## Interview Questions to Consider

1. **Q: What are the three steps in React's rendering process?**
   - A: (1) Trigger a render, (2) Render the component (call the function), (3) Commit changes to the DOM.

2. **Q: What are the two ways to trigger a render?**
   - A: (1) Initial render when the app starts, (2) State updates via setter functions.

3. **Q: What does "rendering" mean in React?**
   - A: React calling your component functions to figure out what should be displayed on screen.

4. **Q: Does every render result in a DOM update?**
   - A: No! React only updates the DOM if there's a difference between the previous and new render output.

5. **Q: Why must rendering be a pure calculation?**
   - A: To ensure predictable behavior, enable optimizations, and prevent bugs. Same inputs should always produce same output, and components shouldn't modify external variables.

6. **Q: What happens during the commit phase?**
   - A: React applies changes to the DOM. For initial render, it creates all DOM nodes. For re-renders, it applies minimal necessary operations to match the latest output.

7. **Q: If a component re-renders but produces the same output, what happens?**
   - A: React compares the outputs and skips the DOM update entirely, improving performance.

8. **Q: What's the difference between React rendering and browser painting?**
   - A: React rendering is calling component functions and calculating what should be displayed. Browser painting is drawing pixels on screen after the DOM is updated.

9. **Q: Why does text in an input field persist when the component re-renders?**
   - A: If the `<input>` element appears identically in both renders, React doesn't recreate it or modify its properties, so user input is preserved.

---

## Key Takeaways

- React's process has three steps: **Trigger → Render → Commit**
- **Two triggers:** Initial render and state updates
- **Rendering** = React calling your component functions
- Rendering is **recursive**—React renders nested components until reaching the bottom
- **Components must be pure** during rendering (same inputs → same output)
- **Commit phase:** React updates the DOM with minimal necessary changes
- React **only changes DOM nodes if there's a difference** between renders
- Not every render produces a DOM update!
- After React commits, the **browser paints** the screen
- Understanding this flow helps debug performance and behavior issues
- State updates automatically queue renders
- React optimizes by comparing outputs and skipping unnecessary DOM work

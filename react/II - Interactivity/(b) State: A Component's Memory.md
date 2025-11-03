# State: A Component's Memory

## Overview
Components often need to "remember" information and change what's displayed based on interactions. Regular variables don't work for this because they don't persist between renders. React provides **state** as a component's memory.

## What You Will Learn
- Add state variables using the `useState` Hook
- Understand the pair of values `useState` returns
- Implement multiple state variables in a single component
- Recognize why state is considered local to components

---

## Why Regular Variables Aren't Sufficient

Consider a simple image gallery with a "Next" button. You might try using a regular variable:

```jsx
function Gallery() {
  let index = 0;

  function handleClick() {
    index = index + 1;
  }

  return (
    <>
      <button onClick={handleClick}>Next</button>
      <h2>Image #{index}</h2>
    </>
  );
}
```

**This doesn't work! Why?**

### Problem 1: Local Variables Don't Persist

**"Local variables don't persist between renders."**

When React renders the component a second time, it renders from scratch—it doesn't consider any changes to local variables. The `index` variable resets to `0` on every render.

### Problem 2: Changes Don't Trigger Re-renders

**"Changes to local variables won't trigger renders."**

React doesn't know it needs to render the component again with new data. Updating `index` doesn't tell React to re-render.

---

## The Solution: useState Hook

To update a component with new data, you need two things:

1. **Retain data** between renders
2. **Trigger React** to render the component with new data

The **`useState` Hook** provides both:
- A **state variable** to retain data
- A **state setter function** to update the variable and trigger a re-render

---

## How to Use useState

### Basic Syntax

```jsx
import { useState } from 'react';

function Gallery() {
  const [index, setIndex] = useState(0);

  function handleClick() {
    setIndex(index + 1);
  }

  return (
    <>
      <button onClick={handleClick}>Next</button>
      <h2>Image #{index}</h2>
    </>
  );
}
```

### What useState Returns

```jsx
const [index, setIndex] = useState(0);
```

**`useState` returns an array with exactly two values:**

1. **State variable** (`index`): The current value
2. **Setter function** (`setIndex`): Function to update the value and trigger a re-render

**Initial value**: `0` is the initial state (used only on first render)

### How It Works Step-by-Step

**First render:**
1. You call `useState(0)`
2. React returns `[0, setIndex]`
3. React remembers `0` as the current state value

**User clicks button:**
1. Event handler calls `setIndex(index + 1)` → `setIndex(1)`
2. React stores `1` as the new state value
3. React triggers a re-render

**Second render:**
1. React sees `useState(0)` again
2. But React remembers you set state to `1`
3. Returns `[1, setIndex]` instead
4. Component renders with `index = 1`

---

## Naming Convention

The convention is:
```jsx
const [something, setSomething] = useState(initialValue);
```

**Examples:**
- `const [count, setCount] = useState(0);`
- `const [name, setName] = useState('');`
- `const [isActive, setIsActive] = useState(false);`

This naming pattern is universal in React codebases and improves readability.

---

## Multiple State Variables

You can use `useState` multiple times in a single component:

```jsx
function Gallery() {
  const [index, setIndex] = useState(0);
  const [showMore, setShowMore] = useState(false);

  function handleNextClick() {
    setIndex(index + 1);
  }

  function handleMoreClick() {
    setShowMore(!showMore);
  }

  return (
    <>
      <button onClick={handleNextClick}>Next</button>
      <button onClick={handleMoreClick}>
        {showMore ? 'Hide' : 'Show'} details
      </button>
      <h2>Image #{index}</h2>
      {showMore && <p>Details about the image...</p>}
    </>
  );
}
```

**Best practice:** Use multiple state variables when their state is unrelated (like `index` and `showMore`).

### State Variables Can Hold Any Type

```jsx
const [count, setCount] = useState(0);              // number
const [name, setName] = useState('Taylor');         // string
const [isActive, setIsActive] = useState(false);    // boolean
const [items, setItems] = useState([]);             // array
const [user, setUser] = useState({ name: 'Bob' }); // object
```

---

## How React Knows Which State to Return

You might wonder: How does React know which state variable corresponds to which `useState` call?

**Answer: React relies on stable call order.**

Hooks depend on being called in the **same order** on every render of the same component. This is why Hooks must be called at the top level.

```jsx
// ✅ Good: Hooks at top level
function Form() {
  const [name, setName] = useState('');
  const [age, setAge] = useState(0);
  // ...
}

// ❌ Bad: Hook inside condition
function Form() {
  if (someCondition) {
    const [name, setName] = useState(''); // Order changes!
  }
  // ...
}
```

> **⚠️ PITFALL**
>
> **Hooks must be called at the top level of components.**
>
> Hooks—functions starting with `use`—can only be called at the top level of your components or your own custom Hooks. You **cannot** call Hooks inside:
> - Conditions (`if` statements)
> - Loops (`for`, `while`)
> - Nested functions
>
> This ensures React can correctly match each `useState` call to its state value across renders.

---

## State is Isolated and Private

### Each Component Instance Gets Its Own State

**"If you render the same component twice, each copy will have completely isolated state!"**

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      Clicked {count} times
    </button>
  );
}

function App() {
  return (
    <>
      <Counter />
      <Counter />
    </>
  );
}
```

**What happens:**
- Each `<Counter />` has its own `count` state
- Clicking one counter doesn't affect the other
- State is tied to the **component's position in the tree**, not the function

### State is Private

**"State is fully private to the component declaring it."**

The parent component **cannot access or change** a child's state directly. State is encapsulated within the component.

```jsx
function ParentComponent() {
  return <ChildComponent />; // Parent can't access ChildComponent's state
}

function ChildComponent() {
  const [count, setCount] = useState(0); // Private to ChildComponent
  // ...
}
```

**To share state:** Lift it up to a common parent and pass it down as props.

---

## When Does React Re-render?

React re-renders a component when:
1. **State changes**: A setter function is called
2. **Props change**: Parent passes new props
3. **Parent re-renders**: Component re-renders if parent does (unless optimized)

**Key insight:** Calling a state setter function is the primary way to trigger a re-render.

---

## Practical Examples

### Example 1: Form Input
```jsx
function NameForm() {
  const [name, setName] = useState('');

  return (
    <input
      value={name}
      onChange={(e) => setName(e.target.value)}
    />
  );
}
```

### Example 2: Toggle Visibility
```jsx
function Accordion() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <button onClick={() => setIsOpen(!isOpen)}>
        {isOpen ? 'Hide' : 'Show'}
      </button>
      {isOpen && <p>Accordion content...</p>}
    </>
  );
}
```

### Example 3: Multiple Independent States
```jsx
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [filter, setFilter] = useState('all');
  const [inputValue, setInputValue] = useState('');

  // Each state variable manages independent data
}
```

---

## Interview Questions to Consider

1. **Q: Why don't regular variables work for component data that changes?**
   - A: Regular variables don't persist between renders (they reset) and changes to them don't trigger React to re-render the component.

2. **Q: What does useState return?**
   - A: An array with two elements: the current state value and a setter function to update that value.

3. **Q: What happens when you call a state setter function?**
   - A: React stores the new state value and triggers a re-render of the component. On the next render, `useState` returns the updated value.

4. **Q: Can you have multiple state variables in one component?**
   - A: Yes! You can call `useState` multiple times. It's good practice when state variables are unrelated.

5. **Q: How does React know which state belongs to which useState call?**
   - A: React relies on the stable call order of Hooks. Each `useState` call must happen in the same order on every render.

6. **Q: Why can't you call Hooks inside conditions or loops?**
   - A: Hooks depend on call order to match state values. Conditional or loop-based calls would change the order between renders, breaking state management.

7. **Q: If you render the same component twice, do they share state?**
   - A: No, each component instance has completely isolated state. Changes to one don't affect the other.

8. **Q: Can a parent component access a child's state?**
   - A: No, state is private to the component that declares it. To share state, lift it up to a common parent.

9. **Q: What's the naming convention for useState?**
   - A: `const [something, setSomething] = useState(initialValue)`, where the setter function is the state variable name prefixed with "set".

---

## Key Takeaways

- Regular variables reset on every render and don't trigger re-renders
- **`useState` Hook** provides persistent state and triggers re-renders
- `useState(initialValue)` returns `[stateValue, setterFunction]`
- Calling a setter function updates state and causes a re-render
- Use multiple state variables for unrelated data
- State variables can hold any JavaScript value (primitives, objects, arrays)
- **Hooks must be called at the top level**—never in conditions, loops, or nested functions
- React uses stable call order to match `useState` calls to state values
- **State is isolated** per component instance—same component rendered twice = separate state
- **State is private** to the component—parents can't access it directly
- Naming convention: `[value, setValue]`
- State is React's way of giving components "memory" across renders

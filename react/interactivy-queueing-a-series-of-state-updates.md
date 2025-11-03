# Queueing a Series of State Updates

## Overview
Setting a state variable queues a re-render, but sometimes you want to perform multiple operations on the value before the next render. Understanding how React batches state updates helps you do this correctly.

## What You Will Learn
- Understanding what "batching" means and how React uses it for multiple state updates
- Applying several updates to the same state variable sequentially
- The distinction between passing values versus updater functions

---

## React Batches State Updates

When you call a state setter, React doesn't update state immediately. Instead, it **batches** updates together.

### What is Batching?

**Batching** means React waits until **all code in event handlers has run** before processing state updates.

```jsx
function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <button onClick={() => {
      setNumber(number + 1);
      setNumber(number + 1);
      setNumber(number + 1);
    }}>
      +3
    </button>
  );
}
```

**What happens:**
- All three `setNumber` calls run first
- React waits for the event handler to finish
- React processes all updates in one batch
- React re-renders once (not three times!)

**Result:** Counter increments by 1, not 3 (because all three calls use `number = 0`).

### The Restaurant Analogy

Think of React like a waiter at a restaurant:
- The waiter doesn't run to the kitchen after each item
- They wait for you to finish your entire order
- Then they take it all to the kitchen at once

Similarly:
- React doesn't re-render after each state update
- It waits for all event handler code to complete
- Then it processes all updates together

**Benefits:**
- Prevents "half-finished" renders with only some updates applied
- Better performance (one re-render instead of many)
- Consistent state during execution

---

## Updating the Same State Multiple Times

What if you really want to update the same state variable multiple times before the next render?

### The Problem with Direct Values

```jsx
setNumber(number + 1); // setNumber(0 + 1)
setNumber(number + 1); // setNumber(0 + 1)
setNumber(number + 1); // setNumber(0 + 1)
```

All three use the snapshot value (`number = 0`), so this only increments by 1.

### The Solution: Updater Functions

Instead of passing the next state value, pass a **function** that calculates it from the previous state:

```jsx
setNumber(n => n + 1);
setNumber(n => n + 1);
setNumber(n => n + 1);
```

**Now what happens:**
1. First `n => n + 1` receives `n = 0`, returns `1`
2. Second `n => n + 1` receives `n = 1`, returns `2`
3. Third `n => n + 1` receives `n = 2`, returns `3`
4. React re-renders with `number = 3`

---

## Updater Functions Explained

An **updater function** is a function you pass to a state setter that:
- Takes the previous state as an argument
- Returns the next state

### Syntax

```jsx
setNumber(n => n + 1);
//         ↑    ↑
//         |    └─ Return next state
//         └────── Previous state parameter
```

### How React Processes the Queue

When you call state setters, React queues operations:

| Queued Update | Previous State (`n`) | Returns | Result |
|---------------|----------------------|---------|--------|
| `n => n + 1` | 0 | 0 + 1 = 1 | 1 |
| `n => n + 1` | 1 | 1 + 1 = 2 | 2 |
| `n => n + 1` | 2 | 2 + 1 = 3 | 3 |

After event handler completes, React triggers a re-render with `number = 3`.

---

## Naming Conventions for Updater Functions

Common patterns for naming the parameter:

### Pattern 1: First Letter
```jsx
setEnabled(e => !e)
setLastName(ln => ln.toUpperCase())
setFriendCount(fc => fc * 2)
```

### Pattern 2: Full Name
```jsx
setEnabled(enabled => !enabled)
setLastName(lastName => lastName.toUpperCase())
```

### Pattern 3: "prev" Prefix
```jsx
setEnabled(prevEnabled => !prevEnabled)
setLastName(prevLastName => prevLastName.toUpperCase())
```

**Recommendation:** Use whichever is most readable for your team.

---

## Value vs. Updater Function

### Passing a Value

```jsx
setNumber(5);
```

React queues: **"Replace with 5"**
- Ignores previous state
- Next render will use `5`

### Passing an Updater Function

```jsx
setNumber(n => n + 1);
```

React queues: **"Apply this function to calculate next state"**
- Uses previous queued state
- Allows chaining multiple updates

---

## Mixed Updates Example

What happens when you mix values and updater functions?

```jsx
function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <button onClick={() => {
      setNumber(number + 5);   // Replace with 5
      setNumber(n => n + 1);   // Apply: 5 + 1 = 6
      setNumber(42);           // Replace with 42
    }}>
      Increase
    </button>
  );
}
```

**Queue processing:**

| Update | Type | Previous | Action | Result |
|--------|------|----------|--------|--------|
| `number + 5` | Replace | - | Replace with `5` | 5 |
| `n => n + 1` | Function | 5 | 5 + 1 | 6 |
| `42` | Replace | - | Replace with `42` | 42 |

**Final state:** `42`

**Key insight:** Direct values act as "replace" operations, discarding what's in the queue so far.

---

## Complete Example with Explanation

```jsx
function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);  // Queue: n => n + 1 (0 + 1 = 1)
        setNumber(n => n + 1);  // Queue: n => n + 1 (1 + 1 = 2)
        setNumber(n => n + 1);  // Queue: n => n + 1 (2 + 1 = 3)
      }}>
        +3
      </button>
    </>
  );
}
```

**Click once:** Counter goes from 0 to 3 in one re-render!

---

## Updater Functions Must Be Pure

> **⚠️ IMPORTANT**
>
> Updater functions must be **pure**:
> - They should only calculate and return the next state
> - No side effects (API calls, mutations, etc.)
> - Same input should always produce same output
>
> **Why:** React may call updater functions multiple times (e.g., in Strict Mode) to verify purity.
>
> ```jsx
> // ✅ Good: Pure calculation
> setCount(c => c + 1)
>
> // ❌ Bad: Side effect
> setCount(c => {
>   logToAnalytics(c); // Side effect!
>   return c + 1;
> })
> ```

---

## When Batching Happens

### Batching Within Event Handlers

React batches all state updates **within the same event handler**:

```jsx
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  setName('Updated');
  // All batched together → one re-render
}
```

### No Batching Across Separate Events

Each user action triggers its own batch:

```jsx
// Click 1: setCount(1) → re-render
// Click 2: setCount(2) → re-render
// Click 3: setCount(3) → re-render
```

Different clicks = different batches = separate re-renders.

---

## Practical Examples

### Example 1: Counter with Multiple Increments
```jsx
function StepCounter() {
  const [count, setCount] = useState(0);

  function increment3() {
    setCount(c => c + 1);
    setCount(c => c + 1);
    setCount(c => c + 1);
  }

  return (
    <>
      <h1>Count: {count}</h1>
      <button onClick={increment3}>+3</button>
    </>
  );
}
```

### Example 2: Toggle with Reset
```jsx
function Toggle() {
  const [enabled, setEnabled] = useState(false);

  return (
    <button onClick={() => {
      setEnabled(e => !e);  // Toggle
      setEnabled(e => !e);  // Toggle back
      setEnabled(e => !e);  // Toggle again
      // Final: opposite of initial
    }}>
      {enabled ? 'ON' : 'OFF'}
    </button>
  );
}
```

### Example 3: Mixed Updates
```jsx
function Calculator() {
  const [result, setResult] = useState(0);

  return (
    <button onClick={() => {
      setResult(10);           // Replace with 10
      setResult(r => r * 2);   // 10 * 2 = 20
      setResult(r => r + 5);   // 20 + 5 = 25
    }}>
      Calculate
    </button>
  );
}
```

---

## Interview Questions to Consider

1. **Q: What is batching in React?**
   - A: React waits until all code in an event handler completes before processing state updates. This groups multiple updates into a single re-render.

2. **Q: Why doesn't calling `setCount(count + 1)` three times increment by 3?**
   - A: All three calls use the same `count` value from the current render's snapshot. They all queue the same value, so only one increment happens.

3. **Q: How do you update the same state multiple times before the next render?**
   - A: Use updater functions: `setCount(c => c + 1)` instead of `setCount(count + 1)`.

4. **Q: What's the difference between `setNumber(5)` and `setNumber(n => n + 5)`?**
   - A: `setNumber(5)` replaces the state with 5. `setNumber(n => n + 5)` adds 5 to the previous queued state value.

5. **Q: What does an updater function receive as its parameter?**
   - A: The previous state value from the queue (either the initial state or the result of the previous updater function).

6. **Q: Can you mix direct values and updater functions?**
   - A: Yes, but direct values act as "replace" operations, discarding previous queue values. Updater functions apply to whatever state is in the queue.

7. **Q: Must updater functions be pure?**
   - A: Yes! They should only calculate and return the next state, with no side effects. React may call them multiple times.

8. **Q: When does React batch state updates?**
   - A: Within the same event handler. Separate events (like different button clicks) trigger separate batches and re-renders.

9. **Q: Why is batching beneficial?**
   - A: It prevents "half-finished" renders with inconsistent state and improves performance by reducing the number of re-renders.

---

## Key Takeaways

- React **batches** state updates within event handlers for performance
- Setting state doesn't update it immediately—React queues updates
- **Direct values** replace state: `setNumber(5)` → "replace with 5"
- **Updater functions** chain updates: `setNumber(n => n + 1)` → "add 1 to previous"
- Use updater functions when updating the same state multiple times
- Updater functions receive the **previous queued state**, not the snapshot
- Updater functions must be **pure** (no side effects)
- Common naming: `setCount(c => ...)`, `setCount(count => ...)`, `setCount(prevCount => ...)`
- Batching prevents unnecessary re-renders and ensures consistent state
- Each event handler gets its own batch—separate clicks don't share batches
- Mixed updates: functions apply sequentially, values replace everything before them

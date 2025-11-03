# State as a Snapshot

## Overview
State might seem like a regular JavaScript variable that you can read and write, but it behaves more like a snapshot. Setting state doesn't change the existing state variable—it triggers a re-render with a new snapshot.

## What You Will Learn
- How state updates trigger re-rendering cycles
- Timing and mechanics of state changes
- Why state doesn't reflect changes immediately after calling the setter function
- How event handlers capture a fixed "snapshot" of state values

---

## Setting State Triggers Renders

You might think clicking a button changes a variable directly, but that's not how React works.

```jsx
function Form() {
  const [isSent, setIsSent] = useState(false);

  if (isSent) {
    return <h1>Message sent!</h1>;
  }

  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      setIsSent(true);
    }}>
      <button type="submit">Send</button>
    </form>
  );
}
```

**What happens when you click "Send":**
1. `setIsSent(true)` is called
2. React schedules a re-render with `isSent = true`
3. React calls `Form()` again with the new state value
4. The new render sees `isSent = true` and returns different JSX

**Key insight:** Setting state **requests a re-render**, it doesn't change the state in the current render.

---

## Rendering Takes a Snapshot in Time

**"Rendering"** means React calls your component function. The JSX you return is like a **snapshot** of the UI at that point in time.

Everything in that render—props, event handlers, local variables—is calculated using **the state values from that render**.

### How It Works

1. React calls your component
2. Your component calculates JSX using current state
3. React updates the screen to match
4. Event handlers in that JSX "remember" the state values from that render

```jsx
function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
      }}>
        +1
      </button>
    </>
  );
}
```

**When you click the button:**
- `number` is `0` in this render
- You call `setNumber(0 + 1)`
- React schedules a re-render with `number = 1`
- `number` stays `0` in the current render

---

## State Over Time: It's Fixed Within a Render

> **⚠️ CRITICAL CONCEPT**
>
> **Setting state only changes it for the _next_ render.**
>
> During a single render, state variables act like constants. Their values never change mid-render, no matter how many times you call the setter function.

### Example: State Doesn't Update Immediately

```jsx
function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
      }}>
        +3
      </button>
    </>
  );
}
```

**What you might expect:** Clicking increments by 3 (from 0 to 3).

**What actually happens:** Clicking increments by 1.

**Why?**
- In this render, `number` is `0`
- All three calls expand to `setNumber(0 + 1)`
- React queues three updates: `setNumber(1)`, `setNumber(1)`, `setNumber(1)`
- Next render: `number` is `1` (not 3!)

---

## State Variables Are Like Constants

Within a single render, **state values never change**, even after calling the setter.

```jsx
const [number, setNumber] = useState(0);

return (
  <button onClick={() => {
    setNumber(number + 5);
    alert(number); // Still 0!
  }}>
    Increase
  </button>
);
```

**Output:** Alert shows `0`, even though you just called `setNumber(number + 5)`.

**Why:** The `alert` uses `number` from this render's snapshot, which is `0`. The state setter schedules a change for the **next** render.

---

## State Snapshot Persists Even with Delays

State snapshots remain fixed even when code runs asynchronously:

```jsx
function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <button onClick={() => {
      setNumber(number + 5);
      setTimeout(() => {
        alert(number);
      }, 3000);
    }}>
      Increase
  </button>
);
}
```

**What happens:**
1. You click the button when `number` is `0`
2. `setNumber(0 + 5)` schedules a re-render with `5`
3. `setTimeout` schedules an alert in 3 seconds
4. Component re-renders with `number = 5`
5. After 3 seconds, the alert shows `0` (not `5`!)

**Why:** The alert "closes over" the `number` value from when it was created (the render where `number` was `0`).

---

## Practical Example: Form with Delayed Send

```jsx
function Form() {
  const [to, setTo] = useState('Alice');
  const [message, setMessage] = useState('Hello');

  function handleSubmit(e) {
    e.preventDefault();
    setTimeout(() => {
      alert(`You said ${message} to ${to}`);
    }, 5000);
  }

  return (
    <form onSubmit={handleSubmit}>
      <label>
        To:{' '}
        <select value={to} onChange={e => setTo(e.target.value)}>
          <option value="Alice">Alice</option>
          <option value="Bob">Bob</option>
        </select>
      </label>
      <textarea value={message} onChange={e => setMessage(e.target.value)} />
      <button type="submit">Send</button>
    </form>
  );
}
```

**Scenario:**
1. Select "Alice" and type "Hello"
2. Click "Send"
3. Immediately change recipient to "Bob"
4. Wait 5 seconds

**What appears in the alert:** "You said Hello to Alice"

**Why:** The alert captures the state values from the render when "Send" was clicked, not the current values.

---

## How React Manages State Snapshots

Think of state like a photograph:

```jsx
// Render 1: number = 0
<button onClick={() => setNumber(0 + 1)}>
  +1
</button>

// User clicks button

// Render 2: number = 1
<button onClick={() => setNumber(1 + 1)}>
  +1
</button>
```

Each render has its own:
- State values
- Event handlers (with state values "baked in")
- Local variables

**Event handlers "remember" the state from the render they were created in.**

---

## Multiple State Updates in One Event Handler

```jsx
function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <button onClick={() => {
      setNumber(number + 1); // setNumber(0 + 1)
      setNumber(number + 1); // setNumber(0 + 1)
      setNumber(number + 1); // setNumber(0 + 1)
    }}>
      Click me
    </button>
  );
}
```

**All three calls use the same `number` value (0) from this render.**

React batches these updates:
- Queues: `1`, `1`, `1`
- Next render: `number = 1`

**To increment multiple times, use the updater function pattern (covered in the next section).**

---

## Why This Design Makes Sense

### Prevents Race Conditions

State snapshots prevent confusing bugs:

```jsx
// Without snapshots (hypothetical)
function sendMessage() {
  setRecipient('Alice');
  // Some async operation...
  // What if recipient changed in the meantime?
  sendEmailTo(recipient); // Which value? 😕
}

// With snapshots (React)
function sendMessage() {
  setRecipient('Alice');
  // recipient is still the original value in this closure
  sendEmailTo(recipient); // Always uses the snapshot value ✅
}
```

### Predictable Behavior

Event handlers always see consistent state values, making code easier to reason about.

---

## Interview Questions to Consider

1. **Q: What happens when you call a state setter function?**
   - A: It schedules a re-render with the new state value for the next render. It doesn't change the state in the current render.

2. **Q: If you call `setNumber(number + 1)` three times in one event handler, what happens?**
   - A: All three calls use the same `number` value from the current render. React batches them, and the next render increments by 1, not 3.

3. **Q: Why does an alert inside setTimeout show the old state value?**
   - A: The setTimeout callback "closes over" the state value from when it was created (the render's snapshot), not the current state.

4. **Q: What is a state "snapshot"?**
   - A: Each render has a fixed set of state values. Event handlers and variables from that render use those specific values, like a snapshot of state at that point in time.

5. **Q: Can state values change during a single render?**
   - A: No, state values are constant within a render. They only change between renders when React calls your component again with new state.

6. **Q: Why doesn't this work: `setNumber(number + 1); setNumber(number + 1); setNumber(number + 1);`?**
   - A: Because `number` has the same value for all three calls in that render. Each evaluates to the same number, so you're effectively calling `setNumber(1)` three times.

7. **Q: In the form example, why does the alert show the old recipient even though you changed it?**
   - A: The `handleSubmit` function captured the state values from when "Send" was clicked. The setTimeout callback uses those captured values (the snapshot), not the current state.

8. **Q: How does React prevent race conditions with state?**
   - A: By making state values fixed within each render, event handlers always see consistent data. Async operations use the state snapshot from when they were initiated.

---

## Key Takeaways

- **State is a snapshot**, not a live variable
- **Setting state triggers a re-render**, it doesn't change state immediately
- **State values are fixed within a render**—they never change mid-render
- Event handlers "remember" state values from their render
- Multiple `setState` calls in one handler use the same state value
- **State only updates between renders**, not during them
- Async code (setTimeout, promises) captures state from when it was created
- This design prevents race conditions and makes code predictable
- Think of each render as having its own "version" of state
- To update state multiple times in one handler, use updater functions (next topic)
- State snapshots ensure consistency even with delays and async operations

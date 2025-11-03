# Referencing Values with Refs

## You Will Learn

- How to add a ref to your component using `useRef`
- How updating a ref differs from updating state
- When to use refs versus state
- How to avoid common mistakes with refs

---

## What are Refs?

**Refs** are a way to reference values that don't need to trigger a re-render when they change. Unlike state, updating a ref's value doesn't cause the component to re-render.

**Key characteristics:**

- **Persist across renders**: Ref values survive component re-renders
- **Mutable**: You can modify `ref.current` at any time
- **No re-renders**: Changing `ref.current` doesn't trigger a component update
- **Direct access**: You can read and write `ref.current` directly

---

## Adding a Ref to Your Component

Use the `useRef` Hook to create a ref. Pass the initial value as an argument:

```jsx
import { useRef } from 'react';

function MyComponent() {
  const myRef = useRef(initialValue);
  // ...
}
```

**`useRef` returns an object:**

```jsx
{
  current: initialValue
}
```

You access and modify the value through the `current` property:

```jsx
function Counter() {
  const countRef = useRef(0);
  
  function handleClick() {
    countRef.current += 1;  // Update the ref
    console.log(countRef.current);  // Read the ref
    // Component doesn't re-render!
  }
  
  return <button onClick={handleClick}>Click me</button>;
}
```

---

## Refs vs. State

Understanding when to use refs versus state is crucial for writing efficient React components.

### Key Differences

| Feature | State (`useState`) | Ref (`useRef`) |
|---------|-------------------|----------------|
| Triggers re-render? | ✅ Yes | ❌ No |
| Returns | `[value, setter]` | `{ current: value }` |
| Updates | Use setter function | Direct assignment |
| Persists across renders? | ✅ Yes | ✅ Yes |
| Use for rendering? | ✅ Yes | ❌ No |

### Example: Stopwatch Using Both

A stopwatch needs both state and refs:

```jsx
import { useState, useRef } from 'react';

function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());  // Triggers re-render to update display
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>Start</button>
      <button onClick={handleStop}>Stop</button>
    </>
  );
}
```

**Why this works:**

- **State (`startTime`, `now`)**: Used for rendering the display, so changes must trigger re-renders
- **Ref (`intervalRef`)**: Stores the interval ID which doesn't affect rendering, so it doesn't need to trigger re-renders

**If we used state for the interval ID:**

```jsx
// ❌ BAD: Unnecessary re-renders
const [intervalId, setIntervalId] = useState(null);

intervalId = setInterval(() => {
  setNow(Date.now());
});
setIntervalId(intervalId);  // Causes re-render even though we don't need to update UI
```

This would cause an extra re-render every time the interval ID is set, which is wasteful.

---

## When to Use Refs

### ✅ Use Refs For:

1. **Storing timeout/interval IDs**
   ```jsx
   const timeoutRef = useRef(null);
   timeoutRef.current = setTimeout(() => {}, 1000);
   clearTimeout(timeoutRef.current);
   ```

2. **Storing previous values**
   ```jsx
   const prevValueRef = useRef(value);
   useEffect(() => {
     prevValueRef.current = value;
   });
   ```

3. **Tracking if component is mounted**
   ```jsx
   const isMountedRef = useRef(true);
   useEffect(() => {
     return () => {
       isMountedRef.current = false;
     };
   }, []);
   ```

4. **Storing DOM element references** (covered in detail in the next lesson)

5. **Storing any mutable value that doesn't affect rendering**
   ```jsx
   const renderCountRef = useRef(0);
   renderCountRef.current += 1;  // Track renders without causing re-renders
   ```

### ❌ Don't Use Refs For:

1. **Values needed for rendering** → Use state
2. **Values that should trigger UI updates** → Use state
3. **Derived values** → Use variables or `useMemo`
4. **Values needed in JSX** → Use state or props

---

## Common Patterns

### Pattern 1: Storing a Previous Value

Sometimes you need to compare the current value with the previous value:

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();

  useEffect(() => {
    prevCountRef.current = count;
  });

  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCountRef.current}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**Note**: `prevCountRef.current` will be `undefined` on the first render, which is fine.

### Pattern 2: Tracking Render Count

Track how many times a component has rendered without causing extra renders:

```jsx
function MyComponent() {
  const renderCountRef = useRef(0);
  renderCountRef.current += 1;
  
  console.log(`Rendered ${renderCountRef.current} times`);
  
  return <div>Component</div>;
}
```

### Pattern 3: Storing Timeout IDs

Store timeout/interval IDs to clear them later:

```jsx
function DebouncedInput() {
  const [value, setValue] = useState('');
  const timeoutRef = useRef(null);

  function handleChange(e) {
    setValue(e.target.value);
    
    clearTimeout(timeoutRef.current);
    timeoutRef.current = setTimeout(() => {
      console.log('Debounced value:', e.target.value);
    }, 500);
  }

  return <input value={value} onChange={handleChange} />;
}
```

---

## Pitfalls

> **⚠️ PITFALL: Don't Read or Write `ref.current` During Rendering**
>
> Reading or writing `ref.current` during the render phase makes your component unpredictable and hard to debug.
>
> ```jsx
> // ❌ NEVER DO THIS
> function MyComponent() {
>   const countRef = useRef(0);
>   countRef.current += 1;  // Writing during render!
>   
>   return <div>Count: {countRef.current}</div>;  // Reading during render!
> }
> ```
>
> **Problems:**
> - React may call your component multiple times during rendering (Strict Mode)
> - Unpredictable behavior
> - Hard to debug
> - Breaks React's rendering rules
>
> ```jsx
> // ✅ DO THIS: Read/write refs in event handlers or effects
> function MyComponent() {
>   const countRef = useRef(0);
>   
>   function handleClick() {
>     countRef.current += 1;  // OK: Writing in event handler
>     console.log(countRef.current);  // OK: Reading in event handler
>   }
>   
>   return <button onClick={handleClick}>Click</button>;
> }
> ```
>
> **Rule**: Only read or write `ref.current` in:
> - Event handlers
> - Effects (`useEffect`)
> - Other functions called during render (not during the render phase itself)

---

> **⚠️ PITFALL: Don't Use Refs for Values That Should Trigger Re-renders**
>
> If changing a value should update the UI, use state, not a ref.
>
> ```jsx
> // ❌ BAD: Using ref for value that should trigger render
> function Counter() {
>   const countRef = useRef(0);
>   
>   function handleClick() {
>     countRef.current += 1;
>     // UI won't update! User won't see the change.
>   }
>   
>   return (
>     <div>
>       <p>Count: {countRef.current}</p>  {/* Always shows 0 */}
>       <button onClick={handleClick}>Increment</button>
>     </div>
>   );
> }
> ```
>
> ```jsx
> // ✅ GOOD: Use state for values that affect rendering
> function Counter() {
>   const [count, setCount] = useState(0);
>   
>   function handleClick() {
>     setCount(count + 1);  // Triggers re-render, UI updates
>   }
>   
>   return (
>     <div>
>       <p>Count: {count}</p>  {/* Updates correctly */}
>       <button onClick={handleClick}>Increment</button>
>     </div>
>   );
> }
> ```
>
> **Remember**: If the value is used in JSX or affects what's rendered, use state.

---

> **⚠️ PITFALL: Don't Overuse Refs**
>
> Refs are an escape hatch. Most of the time, state is what you need. Overusing refs makes components harder to understand and test.
>
> ```jsx
> // ❌ OVERUSE: Using refs for everything
> function Form() {
>   const nameRef = useRef('');
>   const emailRef = useRef('');
>   const passwordRef = useRef('');
>   
>   function handleSubmit() {
>     // All values stored in refs, form never updates visually
>     console.log(nameRef.current, emailRef.current, passwordRef.current);
>   }
>   
>   return (
>     <form onSubmit={handleSubmit}>
>       <input ref={nameRef} />  {/* Won't be controlled! */}
>       <input ref={emailRef} />
>       <input ref={passwordRef} />
>       <button>Submit</button>
>     </form>
>   );
> }
> ```
>
> ```jsx
> // ✅ BETTER: Use state for form inputs
> function Form() {
>   const [name, setName] = useState('');
>   const [email, setEmail] = useState('');
>   const [password, setPassword] = useState('');
>   
>   function handleSubmit(e) {
>     e.preventDefault();
>     console.log(name, email, password);
>   }
>   
>   return (
>     <form onSubmit={handleSubmit}>
>       <input value={name} onChange={(e) => setName(e.target.value)} />
>       <input value={email} onChange={(e) => setEmail(e.target.value)} />
>       <input value={password} onChange={(e) => setPassword(e.target.value)} />
>       <button>Submit</button>
>     </form>
>   );
> }
> ```
>
> **Guideline**: Default to state. Use refs only when you specifically need a value that doesn't trigger re-renders.

---

> **⚠️ PITFALL: Remember That `ref.current` Can Be `null`**
>
> When using refs with DOM elements (covered in next lesson), `ref.current` starts as `null` and only gets set after the component mounts.
>
> ```jsx
> // ❌ PROBLEM: Accessing ref.current before it's set
> function MyComponent() {
>   const inputRef = useRef(null);
>   
>   // inputRef.current is null here!
>   inputRef.current.focus();  // Error: Cannot read property 'focus' of null
>   
>   return <input ref={inputRef} />;
> }
> ```
>
> ```jsx
> // ✅ SOLUTION: Access in useEffect or event handler
> function MyComponent() {
>   const inputRef = useRef(null);
>   
>   useEffect(() => {
>     // Component has mounted, ref.current is now set
>     inputRef.current?.focus();  // Safe with optional chaining
>   }, []);
>   
>   return <input ref={inputRef} />;
> }
> ```
>
> **Rule**: Always check if `ref.current` is `null` before using it, especially with DOM refs.

---

## Ref as Instance Variable

In class components, you could use instance variables that persisted across renders. Refs provide the same capability in function components:

**Class component (old way):**

```jsx
class MyComponent extends React.Component {
  intervalId = null;  // Instance variable
  
  componentDidMount() {
    this.intervalId = setInterval(() => {}, 1000);
  }
  
  componentWillUnmount() {
    clearInterval(this.intervalId);
  }
}
```

**Function component with refs (modern way):**

```jsx
function MyComponent() {
  const intervalRef = useRef(null);
  
  useEffect(() => {
    intervalRef.current = setInterval(() => {}, 1000);
    return () => clearInterval(intervalRef.current);
  }, []);
}
```

Refs give function components the same persistent storage that instance variables provided in class components.

---

## Complete Example: Debounced Search

Here's a complete example showing refs in action:

```jsx
import { useState, useRef } from 'react';

function SearchBox() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const timeoutRef = useRef(null);

  function handleChange(e) {
    const newQuery = e.target.value;
    setQuery(newQuery);

    // Clear previous timeout
    if (timeoutRef.current) {
      clearTimeout(timeoutRef.current);
    }

    // Set new timeout
    timeoutRef.current = setTimeout(() => {
      // Simulate API call
      setResults([
        `Result 1 for "${newQuery}"`,
        `Result 2 for "${newQuery}"`,
      ]);
    }, 300);
  }

  return (
    <div>
      <input
        type="text"
        value={query}
        onChange={handleChange}
        placeholder="Search..."
      />
      <ul>
        {results.map((result, index) => (
          <li key={index}>{result}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Why refs here?**

- The timeout ID doesn't need to trigger a re-render
- We need to store it to clear it when the query changes
- State would cause unnecessary re-renders every time we set the timeout ID

---

## Interview Questions to Consider

### Conceptual Understanding

1. **What is a ref in React, and how does it differ from state?**
   - A ref is a way to store a mutable value that persists across renders but doesn't trigger re-renders when changed. State triggers re-renders, while refs don't. Use refs for values that don't affect rendering; use state for values that do.

2. **When would you use `useRef` instead of `useState`?**
   - Use `useRef` when you need to store a value that:
     - Doesn't need to trigger a re-render when changed
     - Persists across renders
     - Is used internally (like timeout IDs, previous values, DOM references)
   - Use `useState` when the value affects what's rendered or needs to trigger updates.

3. **What does `useRef` return, and how do you access the stored value?**
   - `useRef` returns an object with a `current` property: `{ current: initialValue }`. You access and modify the value through `ref.current`.

4. **Can you modify `ref.current` directly?**
   - Yes, unlike state, you can directly assign to `ref.current`: `ref.current = newValue`. This doesn't trigger a re-render.

### Practical Application

5. **What's wrong with this code?**
   ```jsx
   function Component() {
     const countRef = useRef(0);
     countRef.current += 1;
     return <div>{countRef.current}</div>;
   }
   ```
   - You shouldn't read or write `ref.current` during rendering. This violates React's rules and can cause unpredictable behavior. Move the read/write to event handlers or effects.

6. **Implement a counter that tracks both the current count (with state) and the previous count (with a ref):**
   ```jsx
   function Counter() {
     const [count, setCount] = useState(0);
     const prevCountRef = useRef();
     
     useEffect(() => {
       prevCountRef.current = count;
     });
     
     return (
       <div>
         <p>Current: {count}</p>
         <p>Previous: {prevCountRef.current}</p>
         <button onClick={() => setCount(count + 1)}>Increment</button>
       </div>
     );
   }
   ```

7. **Why use a ref for storing a timeout ID instead of state?**
   - Storing a timeout ID in state would trigger an unnecessary re-render every time the timeout is set or cleared. Since the timeout ID doesn't affect what's rendered, a ref is more appropriate and efficient.

### Advanced Thinking

8. **How do refs relate to instance variables in class components?**
   - Refs provide function components with the same capability that instance variables provided in class components: persistent storage across renders that doesn't trigger re-renders. Both allow storing mutable values that survive component re-renders.

9. **Can you have multiple refs in one component?**
   - Yes, you can call `useRef` multiple times to create multiple refs:
   ```jsx
   function Component() {
     const ref1 = useRef(0);
     const ref2 = useRef(null);
     const ref3 = useRef('initial');
     // Use each ref independently
   }
   ```

10. **What happens if you don't initialize a ref?**
    - If you call `useRef()` without an argument, `ref.current` will be `undefined`. If you call `useRef(null)`, `ref.current` will be `null`. Both are valid, but `null` is often preferred for DOM refs to make null checks clearer.

---

## Summary

Refs are a powerful feature in React that allow you to reference values that don't need to trigger re-renders:

- **Created with `useRef`**: Returns an object with a `current` property
- **Persist across renders**: Values survive component re-renders
- **No re-renders**: Changing `ref.current` doesn't update the UI
- **Use for**: Timeout IDs, previous values, DOM references, any mutable value that doesn't affect rendering
- **Don't use for**: Values needed in JSX or that should trigger UI updates (use state instead)

**Key rules:**
- Don't read or write `ref.current` during rendering (only in event handlers or effects)
- Use refs sparingly—state is usually what you need
- Remember that `ref.current` can be `null` initially (especially with DOM refs)

Refs are an escape hatch for when you need to store values that shouldn't trigger re-renders. Use them wisely, and default to state for most cases.


# Lifecycle of Reactive Effects

## You Will Learn

- How an Effect's lifecycle is different from a component's lifecycle
- How to think about each individual Effect independently
- When and why React re-synchronizes an Effect
- How React knows which dependencies your Effect needs
- What happens with stale closures and how to fix them

---

## Effects Have Their Own Lifecycle

An Effect's lifecycle is separate from your component's lifecycle. A component may mount, update, or unmount multiple times. An Effect can:

1. **Start synchronizing** - Set up a connection, subscribe, fetch data
2. **Stop synchronizing** - Clean up: disconnect, unsubscribe, cancel requests

This cycle can repeat many times throughout a component's lifetime.

### Component Lifecycle vs. Effect Lifecycle

**Component lifecycle:**
- Mount → Update → Update → Unmount

**Effect lifecycle (can repeat many times):**
- Start → Stop → Start → Stop → Start → Stop

An Effect synchronizes with external systems based on reactive values (props, state, derived values).

---

## How Effects Synchronize

Effects synchronize your component's reactive values with external systems. When reactive values change, React re-runs the Effect to keep things in sync.

### Example: Chat Room Connection

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();

    return () => {
      connection.disconnect();
    };
  }, [roomId]);
}
```

**What happens:**

1. **Component renders** with `roomId = "general"`
2. **Effect runs** → Connects to "general" room
3. **User changes** `roomId` to "travel"
4. **Component re-renders** with `roomId = "travel"`
5. **Cleanup runs** → Disconnects from "general" room
6. **Effect runs again** → Connects to "travel" room
7. **Component unmounts**
8. **Cleanup runs** → Disconnects from "travel" room

**Key insight**: Each Effect independently manages one synchronization.

---

## Each Effect Represents a Separate Synchronization

You can have multiple Effects in one component, each managing a different synchronization:

```jsx
function ChatRoom({ roomId, theme }) {
  // Effect 1: Synchronizes room connection
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  // Effect 2: Synchronizes theme
  useEffect(() => {
    document.body.className = theme;
    return () => {
      document.body.className = '';
    };
  }, [theme]);

  return <h1>Welcome to {roomId}!</h1>;
}
```

**Each Effect:**
- Has its own lifecycle
- Manages its own synchronization
- Runs independently
- Has its own cleanup

**Benefits:**
- Clear separation of concerns
- Each Effect is easier to understand
- Dependencies are explicit and minimal

---

## React Identifies Reactive Values

React identifies which values are "reactive" (come from component scope) and requires you to list them as dependencies.

### What Makes a Value Reactive?

A value is reactive if it comes from:
- **Props**: `roomId`, `userId`, etc.
- **State**: Values returned from `useState`
- **Derived values**: Variables computed from props/state
- **Context values**: Values from `useContext`

### Example: Identifying Reactive Values

```jsx
function ChatRoom({ roomId, serverUrl }) {
  const [message, setMessage] = useState('');
  const theme = useContext(ThemeContext);

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId, theme);
    connection.connect();

    return () => connection.disconnect();
  }, [serverUrl, roomId, theme]); // All reactive values must be listed

  // ...
}
```

**React's rules:**
- Any value from component scope used inside Effect must be in dependencies
- React will warn if you forget one (via ESLint plugin)
- Non-reactive values (like constants, functions defined outside component) don't need to be in dependencies

---

## Effects Re-run When Dependencies Change

When a dependency value changes, React:
1. Runs cleanup (if provided)
2. Runs the Effect again with new values

### Example: Re-synchronization

```jsx
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying]);

  return <video ref={ref} src={src} />;
}
```

**Lifecycle:**

1. **Initial render**: `isPlaying = false` → Effect runs → `pause()`
2. **User clicks play**: `isPlaying = true`
3. **Re-render**: Component renders with `isPlaying = true`
4. **Effect re-runs**: Effect runs with new `isPlaying` → `play()`
5. **User clicks pause**: `isPlaying = false`
6. **Re-render**: Component renders with `isPlaying = false`
7. **Effect re-runs**: Effect runs with new `isPlaying` → `pause()`

**No cleanup needed** in this example because play/pause are idempotent operations.

---

## Cleanup Functions

Cleanup functions run:
1. **Before re-running the Effect** (when dependencies change)
2. **Before unmounting** (when component is removed)

### When Cleanup is Needed

**You need cleanup for:**
- Subscriptions (WebSocket, events)
- Timers/intervals
- Network requests (cancel if component unmounts)
- DOM manipulation that needs reversal

**You don't need cleanup for:**
- Idempotent operations (play/pause, focus, scroll)
- Logging/analytics
- Non-persistent operations

### Example: Subscription with Cleanup

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();

    connection.on('message', (msg) => {
      showNotification(msg);
    });

    return () => {
      connection.disconnect(); // Cleanup: disconnect on re-run or unmount
    };
  }, [roomId]);
}
```

**Lifecycle when `roomId` changes:**

1. Component re-renders with new `roomId`
2. **Cleanup runs** → Disconnects from old room
3. **Effect runs** → Connects to new room

**Lifecycle on unmount:**

1. Component unmounts
2. **Cleanup runs** → Disconnects from room

---

## Stale Closures

A "stale closure" happens when an Effect uses an old value of a reactive dependency.

### The Problem

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      console.log(count); // Always logs the initial count (0)!
    }, 1000);

    return () => clearInterval(intervalId);
  }, []); // Missing count dependency!
}
```

**Problem**: `count` is used inside the Effect but not in dependencies. The interval callback captures the initial `count` value (0) and never sees updates.

### The Solution

**Option 1: Add to dependencies**

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      console.log(count); // Now uses current count
    }, 1000);

    return () => clearInterval(intervalId);
  }, [count]); // Include count - Effect re-runs when count changes
}
```

**Note**: This creates a new interval every second (not ideal for this case).

**Option 2: Use functional state updates**

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setCount(c => c + 1); // Functional update - always uses latest
    }, 1000);

    return () => clearInterval(intervalId);
  }, []); // Empty deps OK - setCount is stable
}
```

**Option 3: Use ref for mutable value**

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);

  useEffect(() => {
    countRef.current = count; // Always update ref
  });

  useEffect(() => {
    const intervalId = setInterval(() => {
      console.log(countRef.current); // Always current value
    }, 1000);

    return () => clearInterval(intervalId);
  }, []); // Empty deps OK - ref.current is always current
}
```

---

## Execution Order

Understanding the order of execution helps you reason about your Effects.

### Multiple Effects Run in Declaration Order

```jsx
function Component() {
  useEffect(() => {
    console.log('Effect 1');
    return () => console.log('Cleanup 1');
  });

  useEffect(() => {
    console.log('Effect 2');
    return () => console.log('Cleanup 2');
  });

  return <div>Content</div>;
}
```

**On mount:**
```
Effect 1
Effect 2
```

**On unmount:**
```
Cleanup 2 (reverse order)
Cleanup 1
```

**Why reverse order?** Cleanup runs in reverse order to undo setup in the opposite direction.

### Effect Runs After Render

```jsx
function Component() {
  const [count, setCount] = useState(0);

  console.log('Render:', count);

  useEffect(() => {
    console.log('Effect:', count);
  }, [count]);

  return <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>;
}
```

**Execution order:**
1. Render: 0
2. Effect: 0
3. User clicks
4. Render: 1
5. Effect: 1

Effects always run after the browser has painted the screen.

---

## Complete Example: Chat Room Lifecycle

Here's a complete example showing the full lifecycle:

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    console.log('🟢 Connecting to', roomId);
    const connection = createConnection(roomId);
    connection.connect();

    connection.on('message', (msg) => {
      setMessages(msgs => [...msgs, msg]);
    });

    return () => {
      console.log('🔴 Disconnecting from', roomId);
      connection.disconnect();
    };
  }, [roomId]);

  return (
    <>
      <h1>Welcome to {roomId}!</h1>
      <MessageList messages={messages} />
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(true);

  return (
    <>
      <label>
        Choose room:{' '}
        <select value={roomId} onChange={(e) => setRoomId(e.target.value)}>
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

**Lifecycle:**

1. **App mounts** → `ChatRoom` mounts with `roomId="general"`
   - 🟢 Connecting to general

2. **User changes room to "travel"**
   - Component re-renders with `roomId="travel"`
   - 🔴 Disconnecting from general
   - 🟢 Connecting to travel

3. **User closes chat**
   - Component unmounts
   - 🔴 Disconnecting from travel

4. **User opens chat again**
   - Component mounts with `roomId="travel"` (last selected)
   - 🟢 Connecting to travel

---

## Pitfalls

> **⚠️ PITFALL: Missing Dependencies Causes Stale Closures**
>
> If you use a reactive value inside an Effect but don't include it in dependencies, you'll have a stale closure that never updates.
>
> ```jsx
> // ❌ PROBLEM: Missing count in dependencies
> function Counter() {
>   const [count, setCount] = useState(0);
>
>   useEffect(() => {
>     const intervalId = setInterval(() => {
>       console.log(count); // Always logs 0 - stale closure!
>     }, 1000);
>
>     return () => clearInterval(intervalId);
>   }, []); // Missing count!
> }
> ```
>
> **Problem**: The Effect captures the initial `count` value (0) in a closure and never sees updates.
>
> ```jsx
> // ✅ SOLUTION 1: Add to dependencies
> function Counter() {
>   const [count, setCount] = useState(0);
>
>   useEffect(() => {
>     const intervalId = setInterval(() => {
>       console.log(count); // Uses current count
>     }, 1000);
>
>     return () => clearInterval(intervalId);
>   }, [count]); // Include count
> }
>
> // ✅ SOLUTION 2: Use functional update (if updating state)
> function Counter() {
>   const [count, setCount] = useState(0);
>
>   useEffect(() => {
>     const intervalId = setInterval(() => {
>       setCount(c => c + 1); // Always uses latest
>     }, 1000);
>
>     return () => clearInterval(intervalId);
>   }, []); // Empty deps OK - setCount is stable
> }
> ```
>
> **Rule**: Include all reactive values you use inside the Effect in the dependency array. React's ESLint plugin will warn you if you forget one.

---

> **⚠️ PITFALL: Forgetting Cleanup for Subscriptions**
>
> If your Effect sets up a subscription, timer, or connection, you must clean it up. Otherwise, you'll have memory leaks and multiple subscriptions running.
>
> ```jsx
> // ❌ PROBLEM: No cleanup
> function ChatRoom({ roomId }) {
>   useEffect(() => {
>     const connection = createConnection(roomId);
>     connection.connect();
>     // No cleanup! Connection stays open forever
>   }, [roomId]);
> }
> ```
>
> **Problem**: Every time `roomId` changes, a new connection is created, but old ones are never closed. Memory leak!
>
> ```jsx
> // ✅ SOLUTION: Always clean up subscriptions
> function ChatRoom({ roomId }) {
>   useEffect(() => {
>     const connection = createConnection(roomId);
>     connection.connect();
>
>     return () => {
>       connection.disconnect(); // Cleanup!
>     };
>   }, [roomId]);
> }
> ```
>
> **Rule**: If your Effect sets up something that persists (subscription, timer, connection), always return a cleanup function.

---

> **⚠️ PITFALL: Including Non-Reactive Values in Dependencies**
>
> Including values that aren't reactive (constants, functions defined outside component) in dependencies is unnecessary and can cause confusion.
>
> ```jsx
> // ❌ UNNECESSARY: Including non-reactive values
> const API_URL = 'https://api.example.com'; // Constant, not reactive
>
> function Component() {
>   const [data, setData] = useState(null);
>
>   useEffect(() => {
>     fetch(API_URL).then(res => res.json()).then(setData);
>   }, [API_URL]); // API_URL never changes, so this is unnecessary
> }
> ```
>
> ```jsx
> // ✅ CORRECT: Don't include constants
> const API_URL = 'https://api.example.com';
>
> function Component() {
>   const [data, setData] = useState(null);
>
>   useEffect(() => {
>     fetch(API_URL).then(res => res.json()).then(setData);
>   }, []); // Empty deps - API_URL is constant
> }
> ```
>
> **Rule**: Only include values that come from component scope (props, state, context, derived values). Constants and functions defined outside components don't need to be in dependencies.

---

> **⚠️ PITFALL: Creating New Objects/Functions in Render**
>
> Creating new objects or functions during render and using them in Effects causes the Effect to re-run on every render, even if the values are logically the same.
>
> ```jsx
> // ❌ PROBLEM: New object every render
> function Component({ userId }) {
>   const options = { userId }; // New object every render!
>
>   useEffect(() => {
>     fetchUser(options);
>   }, [options]); // Effect runs every render!
> }
> ```
>
> **Problem**: `options` is a new object on every render, so React sees it as changed, causing the Effect to run unnecessarily.
>
> ```jsx
> // ✅ SOLUTION 1: Include primitive values directly
> function Component({ userId }) {
>   useEffect(() => {
>     fetchUser({ userId });
>   }, [userId]); // Only userId, not the object
> }
>
> // ✅ SOLUTION 2: Use useMemo if object is complex
> function Component({ userId, filters }) {
>   const options = useMemo(() => ({ userId, ...filters }), [userId, filters]);
>
>   useEffect(() => {
>     fetchUser(options);
>   }, [options]);
> }
> ```
>
> **Rule**: Include primitive dependencies directly. For objects/arrays, either destructure and include primitives, or use `useMemo` to memoize the object.

---

> **⚠️ PITFALL: Cleanup Functions Not Running in Expected Order**
>
> Understanding cleanup order is important. Cleanup runs in reverse order of Effect declaration, and cleanup runs before the next Effect execution.
>
> ```jsx
> // Example showing cleanup order
> function Component() {
>   useEffect(() => {
>     console.log('Effect 1');
>     return () => console.log('Cleanup 1');
>   }, []);
>
>   useEffect(() => {
>     console.log('Effect 2');
>     return () => console.log('Cleanup 2');
>   }, []);
> }
> ```
>
> **On mount**: Effect 1, Effect 2
> **On unmount**: Cleanup 2, Cleanup 1 (reverse order)
>
> **Why reverse?** Cleanup undoes setup, so it should happen in reverse order.
>
> **Rule**: Don't assume cleanup order. Write cleanup functions to be independent, or if they must interact, be explicit about dependencies.

---

## Summary: Effect Lifecycle

Understanding the lifecycle of reactive Effects is crucial:

- **Effects have their own lifecycle**: Start synchronizing → Stop synchronizing (can repeat many times)
- **Each Effect is independent**: Multiple Effects in one component each manage their own synchronization
- **Reactive values must be dependencies**: Props, state, context values used in Effects must be in the dependency array
- **Effects re-run when dependencies change**: React runs cleanup, then re-runs Effect with new values
- **Cleanup is essential**: For subscriptions, timers, and connections, always return cleanup functions
- **Watch for stale closures**: Missing dependencies cause stale closures that never update
- **Execution order matters**: Effects run after render, cleanup runs in reverse order

**Key principles:**

- Include all reactive values in dependencies
- Always clean up subscriptions, timers, and connections
- Understand when cleanup runs (before re-run or unmount)
- Watch for stale closures and fix them by including dependencies or using refs/functional updates

Effects are powerful, but understanding their lifecycle helps you write correct, efficient code.


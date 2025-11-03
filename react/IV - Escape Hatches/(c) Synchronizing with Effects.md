# Synchronizing with Effects

## You Will Learn

- What Effects are and how they differ from event handlers
- How to declare an Effect in your component
- How to specify when an Effect should re-run with dependencies
- How to clean up Effects to avoid bugs
- When Effects run and in what order

---

## What are Effects?

**Effects** let you run code after rendering to synchronize your component with systems outside of React. They're called "Effects" because they're typically used for side effects—operations that affect things outside your component.

**Examples of when you need Effects:**

- **Fetching data**: Loading data from a server
- **Setting up subscriptions**: Connecting to WebSocket, chat server, etc.
- **Manipulating the DOM**: Focusing inputs, scrolling, integrating with non-React libraries
- **Logging**: Analytics, performance monitoring
- **Synchronizing with external systems**: Browser APIs, third-party libraries

**Effects run after the browser has painted the screen**, allowing you to perform side effects that shouldn't block the UI.

---

## Effects vs. Event Handlers

Understanding when to use Effects vs. event handlers is crucial:

| Aspect | Event Handlers | Effects |
|--------|---------------|---------|
| **When they run** | In response to user actions | After rendering |
| **Trigger** | User clicks, types, etc. | Render completion |
| **Purpose** | Handle specific user interactions | Synchronize with external systems |
| **Example** | Button click handler | Fetch data on mount |

### Example: Event Handler vs. Effect

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // Event handler: Responds to user clicking "Send"
  function handleSendClick() {
    sendMessage(message);
  }

  // Effect: Synchronizes with chat server when roomId changes
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);

  return (
    <>
      <input value={message} onChange={(e) => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>Send</button>
    </>
  );
}
```

**Key difference**: Event handlers respond to user actions; Effects synchronize with external systems based on rendering.

---

## Declaring an Effect

### Step 1: Import `useEffect`

```jsx
import { useEffect } from 'react';
```

### Step 2: Call `useEffect` at the Top Level

Call `useEffect` at the top level of your component, not inside conditions or loops:

```jsx
function MyComponent() {
  useEffect(() => {
    // Effect code here
  });
  
  return <div>...</div>;
}
```

**Rule**: Always call Hooks at the top level of your component.

### Step 3: Write Your Effect Code

Place the code that synchronizes with the external system inside the Effect:

```jsx
function VideoPlayer({ src, isPlaying }) {
  const videoRef = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      videoRef.current.play();
    } else {
      videoRef.current.pause();
    }
  });

  return <video ref={videoRef} src={src} />;
}
```

---

## Specifying Effect Dependencies

By default, Effects run after **every** render. To control when an Effect runs, specify a dependency array as the second argument.

### No Dependencies (Runs After Every Render)

```jsx
useEffect(() => {
  // Runs after every render
  console.log('Component rendered');
});
```

**Use case**: Rarely needed. Usually you want to specify dependencies.

### Empty Dependencies (Runs Once on Mount)

```jsx
useEffect(() => {
  // Runs only once when component mounts
  console.log('Component mounted');
}, []);
```

**Use case**: Setting up subscriptions, fetching initial data, one-time setup.

### With Dependencies (Runs When Dependencies Change)

```jsx
function VideoPlayer({ src, isPlaying }) {
  useEffect(() => {
    // Runs when isPlaying changes
    if (isPlaying) {
      videoRef.current.play();
    } else {
      videoRef.current.pause();
    }
  }, [isPlaying]); // Dependency array

  return <video src={src} />;
}
```

**How it works**: React compares the dependency values. If any dependency changed since the last render, React runs the Effect again.

### Multiple Dependencies

```jsx
function ChatRoom({ roomId, serverUrl }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // Both are dependencies
}
```

**Rule**: Include all values from component scope (props, state, variables) that are used inside the Effect.

---

## Example: Video Player

Here's a complete example showing Effects in action:

```jsx
import { useState, useEffect, useRef } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }, [isPlaying]);

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

**What happens:**

1. Component renders with `isPlaying={false}`
2. Effect runs, calls `pause()` (video starts paused)
3. User clicks button, `isPlaying` becomes `true`
4. Component re-renders
5. Effect runs again (because `isPlaying` changed), calls `play()`

---

## Cleanup Functions

Some Effects need cleanup to prevent memory leaks, cancel network requests, or disconnect subscriptions.

### Returning a Cleanup Function

Return a function from your Effect to clean up:

```jsx
useEffect(() => {
  // Setup code
  const connection = createConnection();
  connection.connect();

  // Cleanup function
  return () => {
    connection.disconnect();
  };
}, []);
```

**When cleanup runs:**

1. **Before re-running the Effect** (if dependencies changed)
2. **Before component unmounts** (when component is removed)

### Example: Chat Room Connection

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();

    return () => {
      connection.disconnect(); // Cleanup: disconnect when roomId changes or component unmounts
    };
  }, [roomId]);

  return <h1>Welcome to {roomId}!</h1>;
}
```

**Lifecycle:**

1. Component mounts → Effect runs → Connection created
2. `roomId` changes → Cleanup runs (disconnect old) → Effect runs (connect new)
3. Component unmounts → Cleanup runs (disconnect)

### Example: Interval with Cleanup

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const intervalId = setInterval(() => {
      setSeconds((s) => s + 1);
    }, 1000);

    return () => {
      clearInterval(intervalId); // Cleanup: clear interval on unmount
    };
  }, []);

  return <div>Seconds: {seconds}</div>;
}
```

**Why cleanup matters**: Without cleanup, the interval keeps running even after the component unmounts, causing memory leaks and errors.

---

## Common Use Cases

### 1. Fetching Data

```jsx
function Profile({ userId }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    let cancelled = false;

    fetchUser(userId).then((userData) => {
      if (!cancelled) {
        setUser(userData);
      }
    });

    return () => {
      cancelled = true; // Cancel if userId changes or component unmounts
    };
  }, [userId]);

  if (!user) return <div>Loading...</div>;
  return <div>{user.name}</div>;
}
```

### 2. Subscribing to Events

```jsx
function WindowSize() {
  const [size, setSize] = useState({ width: window.innerWidth, height: window.innerHeight });

  useEffect(() => {
    function handleResize() {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    }

    window.addEventListener('resize', handleResize);

    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);

  return <div>Window size: {size.width} x {size.height}</div>;
}
```

### 3. Controlling Non-React Components

```jsx
function Map({ center, zoom }) {
  const mapRef = useRef(null);

  useEffect(() => {
    if (mapRef.current === null) {
      mapRef.current = new MapLibreGL.Map({
        container: mapRef.current,
        center: center,
        zoom: zoom,
      });
    }

    const map = mapRef.current;
    map.setCenter(center);
    map.setZoom(zoom);

    return () => {
      map.remove(); // Cleanup on unmount
    };
  }, [center, zoom]);

  return <div ref={mapRef} style={{ width: '100%', height: '400px' }} />;
}
```

### 4. Focusing Elements

```jsx
function SearchInput({ query }) {
  const inputRef = useRef(null);

  useEffect(() => {
    inputRef.current?.focus();
  }, []); // Only on mount

  return <input ref={inputRef} defaultValue={query} />;
}
```

---

## Pitfalls

> **⚠️ PITFALL: Forgetting the Dependency Array**
>
> If you don't specify dependencies, the Effect runs after every render, which can cause infinite loops or performance issues.
>
> ```jsx
> // ❌ PROBLEM: Missing dependency array
> function Component({ userId }) {
>   const [user, setUser] = useState(null);
>   
>   useEffect(() => {
>     fetchUser(userId).then(setUser);
>     // Runs after EVERY render, causing infinite loop!
>   });
>   
>   return <div>{user?.name}</div>;
> }
> ```
>
> **Problem**: After `setUser` is called, component re-renders, Effect runs again, fetches again, infinite loop!
>
> ```jsx
> // ✅ SOLUTION: Include dependencies
> function Component({ userId }) {
>   const [user, setUser] = useState(null);
>   
>   useEffect(() => {
>     fetchUser(userId).then(setUser);
>   }, [userId]); // Only runs when userId changes
>   
>   return <div>{user?.name}</div>;
> }
> ```
>
> **Rule**: Always include all dependencies that are used inside the Effect. If the Effect should run only once, use an empty array `[]`.

---

> **⚠️ PITFALL: Including Unnecessary Dependencies**
>
> Including dependencies that don't need to be there causes the Effect to run more often than necessary.
>
> ```jsx
> // ❌ PROBLEM: Unnecessary dependency
> function VideoPlayer({ src, isPlaying }) {
>   const [count, setCount] = useState(0);
>   
>   useEffect(() => {
>     if (isPlaying) {
>       videoRef.current.play();
>     }
>   }, [isPlaying, count]); // count is not used in Effect!
>   
>   return (
>     <>
>       <video ref={videoRef} src={src} />
>       <button onClick={() => setCount(count + 1)}>Count: {count}</button>
>     </>
>   );
> }
> ```
>
> **Problem**: Effect runs every time `count` changes, even though it doesn't use `count`. This is wasteful.
>
> ```jsx
> // ✅ SOLUTION: Only include dependencies you actually use
> function VideoPlayer({ src, isPlaying }) {
>   const [count, setCount] = useState(0);
>   
>   useEffect(() => {
>     if (isPlaying) {
>       videoRef.current.play();
>     }
>   }, [isPlaying]); // Only isPlaying, not count
>   
>   return (
>     <>
>       <video ref={videoRef} src={src} />
>       <button onClick={() => setCount(count + 1)}>Count: {count}</button>
>     </>
>   );
> }
> ```
>
> **Rule**: Only include values in the dependency array that are actually read inside the Effect.

---

> **⚠️ PITFALL: Missing Dependencies in the Array**
>
> If you use a value inside an Effect but don't include it in dependencies, React will warn you, and you may have stale closures or bugs.
>
> ```jsx
> // ❌ PROBLEM: Missing dependency
> function ChatRoom({ roomId }) {
>   const [serverUrl, setServerUrl] = useState('https://server.com');
>   
>   useEffect(() => {
>     const connection = createConnection(serverUrl, roomId);
>     connection.connect();
>     return () => connection.disconnect();
>   }, [roomId]); // Missing serverUrl!
>   
>   // ...
> }
> ```
>
> **Problem**: If `serverUrl` changes, the Effect won't re-run, and the connection will use the old `serverUrl`.
>
> ```jsx
> // ✅ SOLUTION: Include all dependencies
> function ChatRoom({ roomId }) {
>   const [serverUrl, setServerUrl] = useState('https://server.com');
>   
>   useEffect(() => {
>     const connection = createConnection(serverUrl, roomId);
>     connection.connect();
>     return () => connection.disconnect();
>   }, [serverUrl, roomId]); // Both dependencies included
>   
>   // ...
> }
> ```
>
> **Rule**: React's ESLint plugin will warn you about missing dependencies. Fix all warnings—they indicate real bugs.

---

> **⚠️ PITFALL: Not Cleaning Up Effects**
>
> Effects that set up subscriptions, timers, or connections must clean up to prevent memory leaks.
>
> ```jsx
> // ❌ PROBLEM: No cleanup
> function ChatRoom({ roomId }) {
>   useEffect(() => {
>     const connection = createConnection(roomId);
>     connection.connect();
>     // No cleanup! Connection stays open after unmount
>   }, [roomId]);
>   
>   return <h1>Welcome to {roomId}!</h1>;
> }
> ```
>
> **Problem**: When component unmounts or `roomId` changes, the old connection stays open, causing memory leaks and bugs.
>
> ```jsx
> // ✅ SOLUTION: Always clean up
> function ChatRoom({ roomId }) {
>   useEffect(() => {
>     const connection = createConnection(roomId);
>     connection.connect();
>     
>     return () => {
>       connection.disconnect(); // Cleanup!
>     };
>   }, [roomId]);
>   
>   return <h1>Welcome to {roomId}!</h1>;
> }
> ```
>
> **Rule**: If your Effect sets up something (subscription, timer, connection), always return a cleanup function.

---

> **⚠️ PITFALL: Using Effects for Event Handlers**
>
> Effects run after rendering. If you need to respond to a user action, use an event handler instead.
>
> ```jsx
> // ❌ WRONG: Using Effect for button click
> function Form() {
>   const [submitted, setSubmitted] = useState(false);
>   
>   useEffect(() => {
>     if (submitted) {
>       sendFormData(); // This runs after render, not on click!
>     }
>   }, [submitted]);
>   
>   return (
>     <form onSubmit={(e) => {
>       e.preventDefault();
>       setSubmitted(true); // Effect will run, but delayed
>     }}>
>       <button type="submit">Submit</button>
>     </form>
>   );
> }
> ```
>
> ```jsx
> // ✅ CORRECT: Use event handler
> function Form() {
>   function handleSubmit(e) {
>     e.preventDefault();
>     sendFormData(); // Runs immediately on submit
>   }
>   
>   return (
>     <form onSubmit={handleSubmit}>
>       <button type="submit">Submit</button>
>     </form>
>   );
> }
> ```
>
> **Rule**: Use event handlers for user interactions. Use Effects for synchronizing with external systems based on rendering.

---

> **⚠️ PITFALL: Calling Hooks Conditionally or in Loops**
>
> Hooks, including `useEffect`, must be called at the top level of your component, not conditionally or in loops.
>
> ```jsx
> // ❌ NEVER DO THIS
> function Component({ shouldFetch }) {
>   if (shouldFetch) {
>     useEffect(() => {
>       fetchData();
>     }, []);
>   }
>   
>   return <div>Content</div>;
> }
> ```
>
> ```jsx
> // ✅ CORRECT: Always call at top level
> function Component({ shouldFetch }) {
>   useEffect(() => {
>     if (shouldFetch) {
>       fetchData();
>     }
>   }, [shouldFetch]);
>   
>   return <div>Content</div>;
> }
> ```
>
> **Rule**: Hooks must be called in the same order on every render. Put conditions inside the Effect, not around the Hook call.

---

## When Effects Run

Understanding the order of execution helps you reason about your code:

### Render Cycle

1. **Component renders** (JSX is computed)
2. **Browser paints screen** (user sees the update)
3. **Effects run** (synchronize with external systems)

### Effect Execution Order

If you have multiple Effects:

```jsx
function Component() {
  useEffect(() => {
    console.log('Effect 1');
  });

  useEffect(() => {
    console.log('Effect 2');
  });

  return <div>Content</div>;
}
```

**Execution order:**
1. Component renders
2. Browser paints
3. All Effects run (in the order they're declared)

### Cleanup Order

When dependencies change or component unmounts:

1. **Cleanup functions run** (in reverse order of Effect declaration)
2. **Effects run** (in order of declaration)

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

  // When component unmounts:
  // Output: Cleanup 2, then Cleanup 1
}
```

---

## Complete Example: Chat Room

Here's a complete example showing Effects with cleanup:

```jsx
import { useState, useEffect } from 'react';

function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();

    connection.on('message', (msg) => {
      setMessages((msgs) => [...msgs, msg]);
    });

    return () => {
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
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select value={roomId} onChange={(e) => setRoomId(e.target.value)}>
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

**What happens:**

1. Component mounts → Effect runs → Connection to "general" room
2. User switches to "travel" → Cleanup runs (disconnect "general") → Effect runs (connect "travel")
3. Component unmounts → Cleanup runs (disconnect)

---

## Interview Questions to Consider

### Conceptual Understanding

1. **What is an Effect, and how does it differ from an event handler?**
   - Effects are code that runs after rendering to synchronize your component with external systems. Event handlers run in response to user actions. Effects run automatically based on rendering; event handlers run when users interact.

2. **When do Effects run?**
   - Effects run after the browser has painted the screen. By default, they run after every render, but you can control this with dependency arrays to run only when specific values change.

3. **What is the purpose of the dependency array in `useEffect`?**
   - The dependency array tells React when to re-run the Effect. If any dependency value changed since the last render, React runs the Effect again. An empty array `[]` means the Effect runs only once on mount.

4. **Why do you need cleanup functions in Effects?**
   - Cleanup functions prevent memory leaks and bugs. They run before the Effect re-runs (when dependencies change) or before the component unmounts, allowing you to cancel subscriptions, clear timers, or disconnect from external systems.

### Practical Application

5. **Write an Effect that fetches user data when `userId` changes:**
   ```jsx
   function UserProfile({ userId }) {
     const [user, setUser] = useState(null);
     
     useEffect(() => {
       let cancelled = false;
       
       fetchUser(userId).then((userData) => {
         if (!cancelled) {
           setUser(userData);
         }
       });
       
       return () => {
         cancelled = true;
       };
     }, [userId]);
     
     return user ? <div>{user.name}</div> : <div>Loading...</div>;
   }
   ```

6. **What's wrong with this Effect?**
   ```jsx
   function Component() {
     const [count, setCount] = useState(0);
     
     useEffect(() => {
       setCount(count + 1);
     });
     
     return <div>{count}</div>;
   }
   ```
   - Missing dependency array. The Effect runs after every render, calls `setCount`, which triggers another render, causing an infinite loop. Add `[]` if you want it to run once, or remove the Effect if you don't need it.

7. **Implement a subscription Effect that cleans up properly:**
   ```jsx
   function Component() {
     useEffect(() => {
       const subscription = subscribe();
       
       return () => {
         subscription.unsubscribe();
       };
     }, []);
     
     return <div>Content</div>;
   }
   ```

### Advanced Thinking

8. **Can you have multiple Effects in one component?**
   - Yes, you can call `useEffect` multiple times. Each Effect runs independently, and you can separate concerns into different Effects (one for data fetching, one for subscriptions, etc.).

9. **What happens if you don't include a dependency that you use inside the Effect?**
   - React will warn you (if you have the ESLint plugin). The Effect may use stale values, causing bugs. For example, if you use `userId` but don't include it, the Effect might use an old `userId` value.

10. **Should you use an Effect to update state based on props?**
    - Usually no. If you can compute the state from props during render, use `useMemo` or compute it directly. Use Effects only when you need to synchronize with external systems. However, if you need to "reset" state when a prop changes, an Effect might be appropriate.

---

## Summary

Effects allow you to synchronize React components with external systems:

- **Declare Effects**: Import `useEffect` and call it at the top level of your component
- **Dependencies**: Use the dependency array to control when Effects run—include all values from component scope that you use inside the Effect
- **Cleanup**: Return a cleanup function from Effects that set up subscriptions, timers, or connections
- **Timing**: Effects run after the browser paints, not during rendering
- **Purpose**: Use Effects for side effects (data fetching, subscriptions, DOM manipulation), not for event handling

**Key principles:**

- Always include dependencies you use inside the Effect
- Always clean up subscriptions, timers, and connections
- Don't use Effects for event handlers—use event handlers instead
- Hooks must be called at the top level, not conditionally or in loops

Effects are a powerful tool for integrating React with the outside world, but they should be used judiciously and with proper cleanup to avoid bugs and memory leaks.


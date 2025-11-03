# Separating Events from Effects

## You Will Learn

- The difference between event handlers and Effects
- Why you might need code that reads the latest props/state without "reacting" to it
- How to separate non-reactive logic from your Effects
- When and how to use `useEffectEvent` to access latest values

---

## Events vs. Effects: The Fundamental Difference

**Event handlers** run in response to specific user actions (clicks, typing, etc.). They run only when the event occurs.

**Effects** synchronize with external systems and re-run when their dependencies change. They're reactive—they respond to changes in props, state, or other reactive values.

**The problem**: Sometimes you need an Effect to read the latest value of a prop/state without re-running when that value changes.

---

## The Problem: Unwanted Re-renders

Consider a shopping cart that logs page visits with the current cart item count:

```jsx
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems); // Log visit with current cart count
  }, [url, numberOfItems]); // Problem: Re-runs every time cart changes!
}
```

**Problem**: The Effect re-runs every time `numberOfItems` changes (user adds/removes items), causing unnecessary logging. You only want to log when the `url` changes.

**What we want**: 
- Re-run Effect when `url` changes (user navigates)
- Don't re-run when `numberOfItems` changes (user modifies cart)
- But still read the latest `numberOfItems` when logging

---

## Solution: Effect Events with `useEffectEvent`

`useEffectEvent` lets you extract non-reactive logic from Effects. An Effect Event can read the latest props/state without causing the Effect to re-run.

### How It Works

```jsx
import { useEffect, useEffectEvent } from 'react';

function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  // Effect Event: Can read latest values without being reactive
  const onVisit = useEffectEvent((visitedUrl) => {
    logVisit(visitedUrl, numberOfItems); // Always reads latest numberOfItems
  });

  // Effect: Only re-runs when url changes
  useEffect(() => {
    onVisit(url);
  }, [url]); // Only url in dependencies, not numberOfItems!
}
```

**Key points:**

1. **`useEffectEvent`** creates an Effect Event function
2. **Effect Events are not reactive** - they don't cause Effects to re-run
3. **They always read latest values** - no stale closures
4. **You can pass arguments** to Effect Events
5. **Only call Effect Events from Effects** - not from event handlers or during render

### Step-by-Step Breakdown

1. Create the Effect Event with `useEffectEvent`
   ```jsx
   const onVisit = useEffectEvent((visitedUrl) => {
     logVisit(visitedUrl, numberOfItems);
   });
   ```

2. Call it from your Effect
   ```jsx
   useEffect(() => {
     onVisit(url); // Only runs when url changes
   }, [url]);
   ```

3. The Effect Event always has access to the latest `numberOfItems`, but the Effect only re-runs when `url` changes

---

## Complete Example: Shopping Cart Page Visit Logging

```jsx
import { useEffect, useEffectEvent, useContext } from 'react';
import { ShoppingCartContext } from './ShoppingCartContext';

function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  // Effect Event: Reads latest numberOfItems without being reactive
  const onVisit = useEffectEvent((visitedUrl) => {
    logVisit(visitedUrl, {
      numberOfItems: numberOfItems, // Always latest value
      timestamp: Date.now(),
    });
  });

  // Effect: Only re-runs when url changes
  useEffect(() => {
    onVisit(url);
  }, [url]); // Only url, not numberOfItems

  return <h1>Welcome to {url}</h1>;
}
```

**What happens:**

1. User navigates to `/products` → Effect runs → Logs visit with current cart count
2. User adds item to cart → `numberOfItems` changes → Effect does NOT re-run
3. User navigates to `/checkout` → Effect runs → Logs visit with current cart count

---

## Common Patterns

### Pattern 1: Reading Latest State Without Reacting

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // Effect Event: Always reads latest message
  const onMessage = useEffectEvent((room) => {
    sendMessage(room, message); // Latest message, even if changed
  });

  useEffect(() => {
    // Only re-run when roomId changes
    // But always use latest message
    onMessage(roomId);
  }, [roomId]); // Only roomId, not message
}
```

### Pattern 2: Combining Reactive and Non-Reactive Logic

```jsx
function VideoPlayer({ src, isPlaying }) {
  const [volume, setVolume] = useState(0.5);
  const videoRef = useRef(null);

  // Effect Event: Reads latest volume without being reactive
  const onPlay = useEffectEvent(() => {
    if (videoRef.current) {
      videoRef.current.volume = volume; // Always latest volume
    }
  });

  // Effect: Re-runs when isPlaying changes
  useEffect(() => {
    if (isPlaying) {
      onPlay(); // Sets current volume
      videoRef.current?.play();
    } else {
      videoRef.current?.pause();
    }
  }, [isPlaying]); // Only isPlaying, not volume
}
```

### Pattern 3: Conditional Logic Based on Latest Values

```jsx
function Form({ userId }) {
  const [draft, setDraft] = useState('');

  // Effect Event: Checks latest draft
  const saveDraft = useEffectEvent((id) => {
    if (draft.trim() !== '') {
      persistDraft(id, draft); // Always uses latest draft
    }
  });

  useEffect(() => {
    saveDraft(userId);
  }, [userId]); // Only userId, not draft
}
```

---

## Effect Events vs. Regular Functions

### Regular Function (Reactive)

```jsx
function Component({ userId }) {
  const [count, setCount] = useState(0);

  // Regular function: If used in Effect, must be in dependencies
  function logData() {
    console.log(userId, count);
  }

  useEffect(() => {
    logData();
  }, [userId, count]); // Must include count or will have stale closure
}
```

**Problem**: If `count` changes, Effect re-runs even if you don't want it to.

### Effect Event (Non-Reactive)

```jsx
function Component({ userId }) {
  const [count, setCount] = useState(0);

  // Effect Event: Reads latest values without being reactive
  const onLog = useEffectEvent(() => {
    console.log(userId, count); // Always latest, no stale closure
  });

  useEffect(() => {
    onLog();
  }, [userId]); // Only userId, Effect doesn't re-run when count changes
}
```

**Benefits**: 
- Reads latest `count` without being reactive to it
- Effect only re-runs when `userId` changes
- No stale closures

---

## When to Use Effect Events

### ✅ Use Effect Events When:

- You need to read the latest value of a prop/state in an Effect
- But you don't want the Effect to re-run when that value changes
- You want to avoid stale closures without adding dependencies
- You're extracting event-like logic from Effects

### ❌ Don't Use Effect Events When:

- You DO want the Effect to re-run when the value changes (use regular dependencies)
- You need the logic in event handlers (use regular functions)
- The logic should run during render (use regular variables/functions)
- You're trying to avoid including necessary dependencies (fix the root issue instead)

---

## Rules for Effect Events

1. **Only call from Effects**: Effect Events should only be called from inside Effects, not from event handlers or during render
2. **Always read latest values**: Effect Events always access the latest props/state, avoiding stale closures
3. **Not reactive**: Changes to values read inside Effect Events don't cause Effects to re-run
4. **Pass arguments**: You can pass arguments to Effect Events when calling them
5. **Must be stable**: Effect Events are stable references (don't change between renders)

---

## Pitfalls

> **⚠️ PITFALL: Calling Effect Events from Event Handlers**
>
> Effect Events should only be called from Effects, not from event handlers. If you need to call logic from an event handler, use a regular function.
>
> ```jsx
> // ❌ WRONG: Calling Effect Event from event handler
> function Component() {
>   const onVisit = useEffectEvent(() => {
>     logVisit();
>   });
>
>   function handleClick() {
>     onVisit(); // Don't do this!
>   }
>
>   return <button onClick={handleClick}>Click</button>;
> }
> ```
>
> ```jsx
> // ✅ CORRECT: Use regular function for event handlers
> function Component() {
>   function handleClick() {
>     logVisit(); // Regular function in event handler
>   }
>
>   return <button onClick={handleClick}>Click</button>;
> }
>
> // OR: Extract shared logic
> function Component() {
>   const onVisit = useEffectEvent(() => {
>     logVisit(); // For use in Effects
>   });
>
>   function handleClick() {
>     logVisit(); // Regular function call for events
>   }
>
>   return <button onClick={handleClick}>Click</button>;
> }
> ```
>
> **Rule**: Effect Events are for Effects. Event handlers use regular functions.

---

> **⚠️ PITFALL: Using Effect Events to Avoid Necessary Dependencies**
>
> Don't use Effect Events to hide missing dependencies. If an Effect should re-run when a value changes, include it in the dependency array.
>
> ```jsx
> // ❌ WRONG: Hiding necessary dependency
> function ChatRoom({ roomId }) {
>   const [serverUrl, setServerUrl] = useState('https://server.com');
>
>   const connect = useEffectEvent((room) => {
>     createConnection(serverUrl, room); // Should re-run when serverUrl changes!
>   });
>
>   useEffect(() => {
>     connect(roomId);
>   }, [roomId]); // Missing serverUrl!
> }
> ```
>
> **Problem**: If `serverUrl` changes, connection doesn't update. This is a bug!
>
> ```jsx
> // ✅ CORRECT: Include necessary dependencies
> function ChatRoom({ roomId }) {
>   const [serverUrl, setServerUrl] = useState('https://server.com');
>
>   useEffect(() => {
>     const connection = createConnection(serverUrl, roomId);
>     connection.connect();
>     return () => connection.disconnect();
>   }, [roomId, serverUrl]); // Both dependencies needed
> }
> ```
>
> **Rule**: Effect Events are for values you read but don't want to react to. If you should react to a change, use dependencies.

---

> **⚠️ PITFALL: Not Understanding When Effect Events Read Latest Values**
>
> Effect Events read the latest values when they're called, not when they're defined. Make sure you understand when they execute.
>
> ```jsx
> function Component() {
>   const [count, setCount] = useState(0);
>
>   const onIncrement = useEffectEvent(() => {
>     // This reads count at the time onIncrement is CALLED,
>     // not when it's defined
>     setCount(count + 1); // Might have stale closure issues!
>   });
>
>   useEffect(() => {
>     const timer = setInterval(() => {
>       onIncrement(); // Calls onIncrement every second
>     }, 1000);
>
>     return () => clearInterval(timer);
>   }, []);
> }
> ```
>
> **Problem**: Even with Effect Event, `setCount(count + 1)` has closure issues because `count` is read at call time, but state updates are asynchronous.
>
> ```jsx
> // ✅ CORRECT: Use functional update
> function Component() {
>   const [count, setCount] = useState(0);
>
>   const onIncrement = useEffectEvent(() => {
>     setCount(c => c + 1); // Functional update - always uses latest
>   });
>
>   useEffect(() => {
>     const timer = setInterval(() => {
>       onIncrement();
>     }, 1000);
>
>     return () => clearInterval(timer);
>   }, []);
> }
> ```
>
> **Rule**: Effect Events help with reading values, but for state updates, still use functional updates when needed.

---

> **⚠️ PITFALL: Overusing Effect Events**
>
> Effect Events are a specialized tool. Don't use them for every function in your Effects. Most of the time, regular functions or proper dependencies are better.
>
> ```jsx
> // ❌ OVERUSE: Effect Event for everything
> function Component({ userId, theme, language }) {
>   const onUpdate = useEffectEvent(() => {
>     updateUser(userId, theme, language);
>   });
>
>   useEffect(() => {
>     onUpdate();
>   }, []); // Should re-run when userId, theme, or language changes!
> }
> ```
>
> ```jsx
> // ✅ BETTER: Use dependencies appropriately
> function Component({ userId, theme, language }) {
>   useEffect(() => {
>     updateUser(userId, theme, language);
>   }, [userId, theme, language]); // React to all changes
> }
> ```
>
> **Rule**: Only use Effect Events when you specifically need to read a value without reacting to it. Default to proper dependencies.

---

## Alternatives to Effect Events

### Alternative 1: Ref + Effect

If `useEffectEvent` isn't available (experimental API), you can use a ref:

```jsx
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;
  const numberOfItemsRef = useRef(numberOfItems);

  // Keep ref in sync
  useEffect(() => {
    numberOfItemsRef.current = numberOfItems;
  });

  useEffect(() => {
    logVisit(url, numberOfItemsRef.current); // Read from ref
  }, [url]); // Only url in dependencies
}
```

**Trade-offs**: More boilerplate, but works without experimental APIs.

### Alternative 2: Restructure Logic

Sometimes you can restructure to avoid the problem:

```jsx
// Instead of Effect Event, move logic to where it's needed
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    logVisit(url, numberOfItems);
  }, [url, numberOfItems]); // If re-running is acceptable
}
```

**When**: If occasional re-runs aren't a problem, simple dependencies work fine.

---

## Complete Example: Form Auto-Save

Here's a complete example showing Effect Events in practice:

```jsx
import { useState, useEffect, useEffectEvent } from 'react';

function EditPost({ postId }) {
  const [title, setTitle] = useState('');
  const [body, setBody] = useState('');

  // Effect Event: Saves draft but doesn't react to draft changes
  const saveDraft = useEffectEvent((id) => {
    if (title.trim() !== '' || body.trim() !== '') {
      persistDraft(id, { title, body }); // Always uses latest title/body
    }
  });

  // Effect: Auto-saves when postId changes, but not when title/body changes
  useEffect(() => {
    saveDraft(postId);
    
    // Also save on unmount
    return () => {
      saveDraft(postId);
    };
  }, [postId]); // Only postId, not title or body

  return (
    <form>
      <input
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Title"
      />
      <textarea
        value={body}
        onChange={(e) => setBody(e.target.value)}
        placeholder="Body"
      />
    </form>
  );
}
```

**What happens:**

1. User types in title/body → Draft updates in state → No Effect re-run
2. User switches to different post → `postId` changes → Effect runs → Saves draft with latest title/body
3. Component unmounts → Cleanup runs → Final save with latest values

---

## Interview Questions to Consider

### Conceptual Understanding

1. **What is the difference between an event handler and an Effect?**
   - Event handlers run in response to user actions (clicks, typing). Effects synchronize with external systems and re-run when dependencies change. Event handlers are event-driven; Effects are reactive.

2. **What is an Effect Event (`useEffectEvent`), and when would you use it?**
   - An Effect Event is a function that can read the latest props/state without causing Effects to re-run. Use it when you need to read a value in an Effect but don't want the Effect to re-run when that value changes.

3. **How do Effect Events differ from regular functions?**
   - Regular functions used in Effects must be included in dependencies or you'll have stale closures. Effect Events always read the latest values without needing to be in dependencies, allowing selective reactivity.

4. **When should you NOT use Effect Events?**
   - Don't use them to avoid necessary dependencies, in event handlers (use regular functions), or when you actually want the Effect to re-run when a value changes.

### Practical Application

5. **Refactor this code to use Effect Events:**
   ```jsx
   function Component({ userId }) {
     const [count, setCount] = useState(0);
     
     useEffect(() => {
       logAction(userId, count);
     }, [userId, count]); // Re-runs every time count changes
   }
   ```
   - Solution:
   ```jsx
   function Component({ userId }) {
     const [count, setCount] = useState(0);
     
     const onLog = useEffectEvent((id) => {
       logAction(id, count); // Reads latest count
     });
     
     useEffect(() => {
       onLog(userId);
     }, [userId]); // Only re-runs when userId changes
   }
   ```

6. **What's wrong with this Effect Event usage?**
   ```jsx
   function Component() {
     const onVisit = useEffectEvent(() => {
       logVisit();
     });
     
     function handleClick() {
       onVisit(); // Called from event handler
     }
     
     return <button onClick={handleClick}>Click</button>;
   }
   ```
   - Effect Events should only be called from Effects, not event handlers. Use a regular function in the event handler instead.

7. **When would you choose to use a ref pattern instead of Effect Events?**
   - When `useEffectEvent` isn't available (experimental API) or you need a simpler solution. Refs require keeping the ref in sync with the value, adding more boilerplate.

### Advanced Thinking

8. **How do Effect Events solve the stale closure problem?**
   - Effect Events are recreated on every render with access to the latest props/state. When called, they always read the current values, not the values from when the Effect was first created.

9. **Can you use Effect Events to avoid all dependencies?**
   - No, and you shouldn't try. Effect Events are for values you read but don't want to react to. If a value should cause the Effect to re-run, it must be in the dependency array. Using Effect Events to hide necessary dependencies is a bug.

10. **What's the execution order when an Effect Event is called?**
    - Effect Events are called synchronously when invoked from an Effect. They read the latest values at the time of the call, not at the time the Effect was set up.

---

## Summary

Separating events from Effects is about managing reactivity precisely:

- **Event handlers**: Run in response to user actions
- **Effects**: Re-run when dependencies change (reactive)
- **Effect Events**: Read latest values without being reactive

**Key principles:**

- Use Effect Events when you need to read a value in an Effect but don't want the Effect to re-run when that value changes
- Only call Effect Events from Effects, not from event handlers or during render
- Don't use Effect Events to avoid necessary dependencies—if you should react to a change, include it in dependencies
- Effect Events solve the "read latest without reacting" problem, avoiding both stale closures and unnecessary re-runs

**Use cases:**
- Logging with current state that shouldn't trigger re-runs
- Reading latest values for conditional logic
- Combining reactive and non-reactive logic in one Effect

Effect Events are a powerful tool for fine-grained control over when Effects re-run while still accessing the latest component values.


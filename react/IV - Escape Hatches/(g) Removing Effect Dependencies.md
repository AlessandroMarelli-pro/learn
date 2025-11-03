# Removing Effect Dependencies

## You Will Learn

- How to remove unnecessary dependencies from Effects
- How to prove a value isn't reactive and doesn't need to be a dependency
- How to fix Effects that have too many dependencies
- What to do about objects and functions in dependencies
- Why you shouldn't suppress dependency warnings

---

## The Dependency Rule

**React's rule**: Every reactive value used inside an Effect must be in the dependency array.

**But sometimes**: The linter suggests dependencies that you don't actually want the Effect to react to. The challenge is removing unnecessary dependencies while keeping necessary ones.

**Key principle**: If a value truly doesn't change between renders (or you can prove it won't change), you can remove it from dependencies. But you must prove it's not reactive.

---

## Strategy 1: Move Static Values Outside Component

If a value never changes, move it outside the component. This proves it's not reactive.

### ❌ Problem: Constant Inside Component

```jsx
function ChatRoom() {
  const serverUrl = 'https://server.com'; // Constant, but inside component
  
  useEffect(() => {
    const connection = createConnection(serverUrl);
    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl]); // serverUrl in dependencies (unnecessary)
}
```

**Problem**: Even though `serverUrl` never changes, React sees it as a reactive value because it's declared inside the component.

### ✅ Solution: Move Outside Component

```jsx
const serverUrl = 'https://server.com'; // Moved outside - not reactive

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl);
    connection.connect();
    return () => connection.disconnect();
  }, []); // No dependencies needed - serverUrl is not reactive
}
```

**Why this works**: Values declared outside the component aren't reactive. React knows they can't change between renders.

---

## Strategy 2: Move Values Inside the Effect

If a value is only used in the Effect and derived from props, you can compute it inside the Effect.

### ❌ Problem: Derived Value in Dependencies

```jsx
function ChatRoom({ roomId, serverUrl }) {
  const options = { serverUrl, roomId }; // New object every render
  
  useEffect(() => {
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [options]); // options changes every render!
}
```

**Problem**: `options` is a new object on every render, causing the Effect to re-run unnecessarily.

### ✅ Solution: Compute Inside Effect

```jsx
function ChatRoom({ roomId, serverUrl }) {
  useEffect(() => {
    const options = { serverUrl, roomId }; // Compute inside Effect
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // Only primitive dependencies
}
```

**Why this works**: Compute derived values inside the Effect from primitive dependencies.

---

## Strategy 3: Remove Unnecessary Object/Function Dependencies

Objects and functions defined inside components are recreated on every render. If they're only used in Effects, you can often restructure to avoid including them.

### ❌ Problem: Function in Dependencies

```jsx
function TodoList({ todos }) {
  const options = {
    showCompleted: true,
    sortBy: 'date',
  };
  
  function getVisibleTodos() {
    return todos.filter(todo => {
      if (!options.showCompleted && todo.completed) return false;
      return true;
    });
  }
  
  useEffect(() => {
    const visible = getVisibleTodos();
    // Use visible todos
  }, [getVisibleTodos]); // Function recreated every render!
}
```

**Problem**: `getVisibleTodos` is a new function every render, causing Effect to re-run constantly.

### ✅ Solution 1: Move Function Inside Effect

```jsx
function TodoList({ todos }) {
  const options = {
    showCompleted: true,
    sortBy: 'date',
  };
  
  useEffect(() => {
    function getVisibleTodos() {
      return todos.filter(todo => {
        if (!options.showCompleted && todo.completed) return false;
        return true;
      });
    }
    
    const visible = getVisibleTodos();
    // Use visible todos
  }, [todos]); // Only todos, not the function
}
```

### ✅ Solution 2: Use useCallback (if function is reused)

```jsx
function TodoList({ todos }) {
  const options = {
    showCompleted: true,
    sortBy: 'date',
  };
  
  const getVisibleTodos = useCallback(() => {
    return todos.filter(todo => {
      if (!options.showCompleted && todo.completed) return false;
      return true;
    });
  }, [todos]); // Function only changes when todos changes
  
  useEffect(() => {
    const visible = getVisibleTodos();
    // Use visible todos
  }, [getVisibleTodos]); // Now stable (changes only when todos changes)
}
```

**When to use each**:
- **Move inside Effect**: If function is only used in that Effect
- **useCallback**: If function is used in multiple places

---

## Strategy 4: Use useCallback for Stable Function References

If a function must be passed as a dependency (e.g., to a child component), use `useCallback` to stabilize it.

### ❌ Problem: Function Recreated Every Render

```jsx
function SearchResults({ query }) {
  function getResults() {
    return fetch(`/api/search?q=${query}`).then(r => r.json());
  }
  
  useEffect(() => {
    getResults().then(setResults);
  }, [getResults]); // getResults changes every render
}
```

### ✅ Solution: useCallback

```jsx
function SearchResults({ query }) {
  const getResults = useCallback(() => {
    return fetch(`/api/search?q=${query}`).then(r => r.json());
  }, [query]); // Only changes when query changes
  
  useEffect(() => {
    getResults().then(setResults);
  }, [getResults]); // Now stable
}
```

**Note**: In this case, you might not even need `useCallback`—you could just include `query` in dependencies:

```jsx
function SearchResults({ query }) {
  useEffect(() => {
    fetch(`/api/search?q=${query}`).then(r => r.json()).then(setResults);
  }, [query]); // Simpler - just query
}
```

---

## Strategy 5: Use useMemo for Expensive Computations

For expensive computations that are used in Effects, `useMemo` can prevent unnecessary recalculations.

### ❌ Problem: Expensive Computation Every Render

```jsx
function DataVisualization({ data, filters }) {
  const processedData = expensiveProcessing(data, filters);
  
  useEffect(() => {
    updateChart(processedData);
  }, [processedData]); // Recalculates every render if data/filters change
}
```

### ✅ Solution: useMemo

```jsx
function DataVisualization({ data, filters }) {
  const processedData = useMemo(() => {
    return expensiveProcessing(data, filters);
  }, [data, filters]); // Only recalculates when data or filters change
  
  useEffect(() => {
    updateChart(processedData);
  }, [processedData]); // Only runs when processedData actually changes
}
```

**When to use**: Only for expensive computations. Don't use `useMemo` for simple operations—the overhead isn't worth it.

---

## Strategy 6: Extract Static Parts of Objects

If only part of an object is reactive, extract the reactive parts.

### ❌ Problem: Entire Object in Dependencies

```jsx
function VideoPlayer({ src, isPlaying }) {
  const options = {
    src,
    isPlaying,
    volume: 0.5, // Constant
    playbackRate: 1.0, // Constant
  };
  
  useEffect(() => {
    player.setOptions(options);
  }, [options]); // Entire object changes when src or isPlaying changes
}
```

### ✅ Solution: Extract Reactive Parts

```jsx
function VideoPlayer({ src, isPlaying }) {
  useEffect(() => {
    player.setOptions({
      src,
      isPlaying,
      volume: 0.5, // Constant - doesn't need to be reactive
      playbackRate: 1.0, // Constant - doesn't need to be reactive
    });
  }, [src, isPlaying]); // Only reactive values
}
```

---

## Strategy 7: Don't Suppress Warnings

**Never suppress linter warnings** about missing dependencies. Instead, fix the root cause.

### ❌ Wrong: Suppressing Warnings

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://server.com');
  
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [roomId]); // Suppressed warning for serverUrl
}
```

**Problems**:
- Hides bugs (if `serverUrl` changes, connection doesn't update)
- Makes code harder to maintain
- Can cause subtle bugs

### ✅ Correct: Fix the Dependencies

```jsx
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://server.com');
  
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl, roomId]); // Include all dependencies
}
```

**Rule**: If the linter warns about a missing dependency, it's usually right. Fix it properly, don't suppress it.

---

## Complete Example: Chat Room with All Strategies

Here's a complete example showing multiple strategies:

```jsx
// Strategy 1: Move constants outside
const DEFAULT_SERVER_URL = 'https://server.com';

function ChatRoom({ roomId, userId }) {
  // Strategy 2: Derived values inside Effect
  useEffect(() => {
    // Compute options inside Effect
    const options = {
      serverUrl: DEFAULT_SERVER_URL, // Constant from outside
      roomId, // From props
      userId, // From props
    };
    
    const connection = createConnection(options);
    connection.connect();
    
    return () => {
      connection.disconnect();
    };
  }, [roomId, userId]); // Only primitive dependencies
  
  return <h1>Welcome to {roomId}!</h1>;
}
```

---

## Decision Tree: Should This Be a Dependency?

```
Is the value used inside the Effect?
├─ No → Don't include it
└─ Yes → Is the value reactive (from props/state/context)?
    ├─ No (constant, outside component) → Don't include it
    └─ Yes → Should the Effect re-run when it changes?
        ├─ No → Use useEffectEvent or move logic
        └─ Yes → Include it in dependencies
```

---

## Pitfalls

> **⚠️ PITFALL: Suppressing Linter Warnings**
>
> Never suppress React's exhaustive-deps warnings. They indicate real bugs or issues that should be fixed properly.
>
> ```jsx
> // ❌ NEVER DO THIS
> function Component({ userId, roomId }) {
>   useEffect(() => {
>     fetchData(userId, roomId);
>     // eslint-disable-next-line react-hooks/exhaustive-deps
>   }, [userId]); // Suppressed warning for roomId
> }
> ```
>
> **Problems**:
> - Hides bugs (if `roomId` changes, data doesn't update)
> - Makes maintenance harder
> - Can cause production bugs
>
> ```jsx
> // ✅ FIX IT PROPERLY
> function Component({ userId, roomId }) {
>   useEffect(() => {
>     fetchData(userId, roomId);
>   }, [userId, roomId]); // Include all dependencies
> }
> ```
>
> **Rule**: Fix warnings, don't suppress them. The linter is usually right.

---

> **⚠️ PITFALL: Including Objects/Functions That Change Every Render**
>
> Objects and functions defined inside components are recreated every render. Including them in dependencies causes Effects to re-run unnecessarily.
>
> ```jsx
> // ❌ PROBLEM: Object recreated every render
> function Component({ userId }) {
>   const options = { userId, timestamp: Date.now() }; // New object every render!
>   
>   useEffect(() => {
>     fetchData(options);
>   }, [options]); // Runs every render!
> }
> ```
>
> ```jsx
> // ✅ SOLUTION: Extract reactive parts
> function Component({ userId }) {
>   useEffect(() => {
>     const options = { userId, timestamp: Date.now() };
>     fetchData(options);
>   }, [userId]); // Only userId
> }
> ```
>
> **Rule**: Don't include objects/functions in dependencies if they're recreated every render. Compute them inside the Effect or use `useMemo`/`useCallback` if needed.

---

> **⚠️ PITFALL: Moving Reactive Values Outside Component**
>
> Moving reactive values (props, state) outside components doesn't make them non-reactive—it breaks your component!
>
> ```jsx
> // ❌ WRONG: Can't move props outside
> function ChatRoom({ roomId }) {
>   // roomId is a prop - can't move outside!
> }
>
> // This doesn't work:
> const roomId = 'general'; // Not reactive, but wrong - hardcoded!
> ```
>
> ```jsx
> // ✅ CORRECT: Only move constants, not props/state
> const DEFAULT_ROOM = 'general'; // Constant OK
>
> function ChatRoom({ roomId }) {
>   const actualRoom = roomId || DEFAULT_ROOM; // Use prop, fallback to constant
>   
>   useEffect(() => {
>     createConnection(actualRoom);
>   }, [actualRoom]);
> }
> ```
>
> **Rule**: Only move non-reactive values (constants, configuration) outside. Props and state must stay inside and be in dependencies if used.

---

> **⚠️ PITFALL: Overusing useMemo/useCallback**
>
> `useMemo` and `useCallback` have overhead. Don't use them for simple operations or when they don't actually solve a problem.
>
> ```jsx
> // ❌ UNNECESSARY: Simple calculation doesn't need useMemo
> function Component({ count }) {
>   const doubled = useMemo(() => count * 2, [count]);
>   
>   useEffect(() => {
>     console.log(doubled);
>   }, [doubled]);
> }
> ```
>
> ```jsx
> // ✅ SIMPLER: Just compute directly or use in Effect
> function Component({ count }) {
>   useEffect(() => {
>     console.log(count * 2); // Simple - no need for useMemo
>   }, [count]);
> }
> ```
>
> **Rule**: Use `useMemo`/`useCallback` only when:
> - The computation is expensive
> - You need to prevent unnecessary re-renders of child components
> - The dependency would cause performance issues

---

> **⚠️ PITFALL: Removing Necessary Dependencies**
>
> Don't remove dependencies just because you don't want the Effect to re-run. If a value should cause the Effect to re-run, it must be a dependency.
>
> ```jsx
> // ❌ WRONG: Removed necessary dependency
> function ChatRoom({ roomId, serverUrl }) {
>   useEffect(() => {
>     const connection = createConnection(serverUrl, roomId);
>     connection.connect();
>     return () => connection.disconnect();
>   }, [roomId]); // Missing serverUrl!
> }
> ```
>
> **Problem**: If `serverUrl` changes, connection doesn't update. This is a bug!
>
> ```jsx
> // ✅ CORRECT: Include all reactive dependencies
> function ChatRoom({ roomId, serverUrl }) {
>   useEffect(() => {
>     const connection = createConnection(serverUrl, roomId);
>     connection.connect();
>     return () => connection.disconnect();
>   }, [serverUrl, roomId]); // Both dependencies
> }
> ```
>
> **Rule**: If a value is used in the Effect and is reactive, it must be in dependencies. Use `useEffectEvent` if you want to read it without reacting to it.

---

## Summary: Strategies for Removing Dependencies

**To remove an unnecessary dependency:**

1. **Move constants outside** - Values that never change can be declared outside the component
2. **Compute inside Effect** - Derived values can be computed inside the Effect
3. **Extract reactive parts** - Don't include entire objects, just the reactive properties
4. **Use useCallback/useMemo** - For functions/objects that need to be stable but depend on reactive values
5. **Don't suppress warnings** - Fix the root cause instead

**Remember**:
- Only remove dependencies that are truly not reactive
- If a value is reactive and used in the Effect, it must be in dependencies (or use `useEffectEvent`)
- The linter is usually right—fix warnings, don't suppress them
- Objects/functions recreated every render should be avoided in dependencies

**Key principle**: Dependencies should reflect what the Effect actually needs to react to. Remove only those that are truly unnecessary.


# Reusing Logic with Custom Hooks

## You Will Learn

- How to extract component logic into reusable custom Hooks
- How custom Hooks share logic but not state
- How custom Hooks are isolated—each call gets its own state
- When extracting a custom Hook is beneficial
- Naming conventions and rules for custom Hooks

---

## What are Custom Hooks?

**Custom Hooks** are JavaScript functions that start with `use` and can call other Hooks. They let you extract component logic into reusable functions.

**Key characteristics:**
- Named with `use` prefix (React convention)
- Can call other Hooks (useState, useEffect, etc.)
- Share logic, not state (each call is isolated)
- Make components cleaner and logic reusable

**Why use them?**
- **Avoid duplication**: Share logic across multiple components
- **Cleaner components**: Extract complex logic out of components
- **Easier testing**: Test logic separately from components
- **Better organization**: Group related logic together

---

## Creating a Custom Hook

### Basic Structure

```jsx
import { useState, useEffect } from 'react';

function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);

  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}
```

**Rules:**
1. **Name starts with `use`** - React convention, allows linter to enforce Hook rules
2. **Can call other Hooks** - useState, useEffect, useContext, etc.
3. **Return value** - Can return anything (value, array, object, nothing)
4. **Called at top level** - Must follow Hook rules (not conditionally)

---

## Using a Custom Hook

Once created, use it like any other Hook:

```jsx
function StatusBar() {
  const isOnline = useOnlineStatus(); // Use the custom Hook
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus(); // Same Hook, different component

  function handleSaveClick() {
    console.log('✅ Progress saved');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}
```

**Each component:**
- Calls the Hook independently
- Gets its own isolated state
- Shares the same logic

---

## Custom Hooks Share Logic, Not State

**Important**: Each call to a custom Hook creates its own isolated state.

```jsx
function ComponentA() {
  const isOnline = useOnlineStatus(); // Has its own isOnline state
  return <div>Component A: {isOnline ? 'Online' : 'Offline'}</div>;
}

function ComponentB() {
  const isOnline = useOnlineStatus(); // Has its OWN SEPARATE isOnline state
  return <div>Component B: {isOnline ? 'Online' : 'Offline'}</div>;
}
```

**Both components:**
- Use the same logic (listening to online/offline events)
- Have separate state instances
- Update independently (though they'll have the same value since they're listening to the same browser event)

**Why this matters**: Custom Hooks are functions—each call is independent, just like calling a regular function multiple times.

---

## Common Custom Hook Patterns

### Pattern 1: State + Effect Hook

Combine state and side effects:

```jsx
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    function handleResize() {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}

// Usage
function Component() {
  const { width, height } = useWindowSize();
  return <div>Window: {width} x {height}</div>;
}
```

### Pattern 2: Data Fetching Hook

Encapsulate data fetching logic:

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    setLoading(true);
    setError(null);

    fetch(url)
      .then(res => res.json())
      .then(data => {
        if (!cancelled) {
          setData(data);
          setLoading(false);
        }
      })
      .catch(err => {
        if (!cancelled) {
          setError(err);
          setLoading(false);
        }
      });

    return () => {
      cancelled = true;
    };
  }, [url]);

  return { data, loading, error };
}

// Usage
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <div>{user.name}</div>;
}
```

### Pattern 3: Returning Multiple Values

Return an object or array:

```jsx
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = () => setValue(v => !v);
  const setTrue = () => setValue(true);
  const setFalse = () => setValue(false);

  return { value, toggle, setTrue, setFalse };
}

// Usage
function Component() {
  const { value: isOpen, toggle, setTrue, setFalse } = useToggle();

  return (
    <>
      <button onClick={toggle}>Toggle</button>
      <button onClick={setTrue}>Open</button>
      <button onClick={setFalse}>Close</button>
      {isOpen && <div>Content</div>}
    </>
  );
}
```

### Pattern 4: Hook with Parameters

Custom Hooks can accept parameters:

```jsx
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}

// Usage
function SearchInput() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 500);

  useEffect(() => {
    if (debouncedQuery) {
      // Fetch search results
      fetchResults(debouncedQuery);
    }
  }, [debouncedQuery]);

  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

---

## When to Extract a Custom Hook

### ✅ Extract When:

1. **Multiple components use the same logic**
   ```jsx
   // If ComponentA, ComponentB, ComponentC all use the same logic
   // Extract it into a custom Hook
   ```

2. **Complex logic makes component hard to read**
   ```jsx
   // If your component has 50 lines of useEffect logic
   // Extract it into a custom Hook
   ```

3. **Logic is self-contained and testable**
   ```jsx
   // If the logic can be tested independently
   // It's a good candidate for a custom Hook
   ```

4. **Logic involves multiple Hooks working together**
   ```jsx
   // If you have useState + useEffect + useRef all working together
   // Extract into a custom Hook
   ```

### ❌ Don't Extract When:

1. **Simple, one-time use logic**
   ```jsx
   // If only one component uses it and it's simple
   // Keep it in the component
   ```

2. **Logic is tightly coupled to one component**
   ```jsx
   // If the logic is specific to one component's UI
   // Probably don't extract
   ```

3. **Extracting makes code harder to understand**
   ```jsx
   // If jumping to a Hook file makes debugging harder
   // Consider keeping it inline
   ```

---

## Naming Conventions

### ✅ Correct Naming

```jsx
function useOnlineStatus() { } // ✅ Starts with 'use'
function useToggle() { }       // ✅ Starts with 'use'
function useFetch() { }        // ✅ Starts with 'use'
function useDebounce() { }     // ✅ Starts with 'use'
```

### ❌ Incorrect Naming

```jsx
function getOnlineStatus() { } // ❌ Doesn't start with 'use'
function onlineStatus() { }    // ❌ Doesn't start with 'use'
function use_online_status() { } // ❌ Use camelCase, not snake_case
```

**Why the `use` prefix matters:**
- React's linter can identify Hooks and enforce rules
- Makes it clear this function follows Hook rules
- Convention that other developers understand

---

## Complete Example: Online Status Hook

Here's a complete example showing a custom Hook in action:

```jsx
import { useState, useEffect } from 'react';

function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }

    function handleOffline() {
      setIsOnline(false);
    }

    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return isOnline;
}

// Multiple components using the same Hook
function StatusBar() {
  const isOnline = useOnlineStatus();
  return (
    <div>
      {isOnline ? '🟢 Online' : '🔴 Offline'}
    </div>
  );
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    saveProgress();
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save' : 'Reconnecting...'}
    </button>
  );
}

function NotificationsList() {
  const isOnline = useOnlineStatus();
  
  if (!isOnline) {
    return <div>No internet connection. Notifications will appear when online.</div>;
  }
  
  return <NotificationList />;
}
```

**Benefits:**
- Logic defined once, used in multiple places
- Each component has its own state instance
- Easy to test the Hook independently
- Components are cleaner and focused on rendering

---

## Pitfalls

> **⚠️ PITFALL: Not Following Hook Rules**
>
> Custom Hooks must follow the same rules as built-in Hooks: only call them at the top level, not conditionally or in loops.
>
> ```jsx
> // ❌ NEVER DO THIS
> function Component({ shouldTrack }) {
>   if (shouldTrack) {
>     const status = useOnlineStatus(); // Conditional Hook call!
>   }
> }
> ```
>
> **Problem**: Hooks must be called in the same order on every render. Conditional calls violate this rule.
>
> ```jsx
> // ✅ CORRECT: Always call at top level
> function Component({ shouldTrack }) {
>   const isOnline = useOnlineStatus(); // Always called
>   
>   if (!shouldTrack) {
>     return null; // Early return is OK
>   }
>   
>   return <div>Status: {isOnline}</div>;
> }
> ```
>
> **Rule**: Always call Hooks at the top level of your component, in the same order on every render.

---

> **⚠️ PITFALL: Not Naming with `use` Prefix**
>
> Custom Hooks must start with `use` to follow React conventions and allow the linter to work correctly.
>
> ```jsx
> // ❌ WRONG: Doesn't start with 'use'
> function getOnlineStatus() {
>   const [isOnline, setIsOnline] = useState(true);
>   // ...
>   return isOnline;
> }
> ```
>
> **Problems**:
> - React linter won't recognize it as a Hook
> - Other developers won't know it follows Hook rules
> - Breaks React conventions
>
> ```jsx
> // ✅ CORRECT: Starts with 'use'
> function useOnlineStatus() {
>   const [isOnline, setIsOnline] = useState(true);
>   // ...
>   return isOnline;
> }
> ```
>
> **Rule**: Always name custom Hooks with a `use` prefix.

---

> **⚠️ PITFALL: Sharing State Between Hook Calls**
>
> Each call to a custom Hook creates its own isolated state. Don't try to share state between calls.
>
> ```jsx
> // ❌ WRONG: Trying to share state
> let sharedState = null; // Outside Hook
>
> function useSharedState() {
>   if (!sharedState) {
>     sharedState = useState(0); // Trying to share
>   }
>   return sharedState;
> }
> ```
>
> **Problem**: This breaks React's rules and causes bugs. Each Hook call must have its own state.
>
> ```jsx
> // ✅ CORRECT: Each call gets its own state
> function useCounter() {
>   const [count, setCount] = useState(0); // New state for each call
>   return [count, setCount];
> }
>
> function ComponentA() {
>   const [count, setCount] = useCounter(); // Own state instance
>   return <div>A: {count}</div>;
> }
>
> function ComponentB() {
>   const [count, setCount] = useCounter(); // Separate state instance
>   return <div>B: {count}</div>;
> }
> ```
>
> **Rule**: Each call to a custom Hook is independent. State is not shared between calls.

---

> **⚠️ PITFALL: Extracting Logic That's Too Simple**
>
> Don't extract every piece of logic into a custom Hook. Sometimes keeping it in the component is simpler.
>
> ```jsx
> // ❌ OVER-ENGINEERING: Simple logic doesn't need a Hook
> function useCount() {
>   const [count, setCount] = useState(0);
>   return [count, setCount];
> }
>
> function Component() {
>   const [count, setCount] = useCount(); // Just use useState directly!
> }
> ```
>
> ```jsx
> // ✅ SIMPLER: Just use useState directly
> function Component() {
>   const [count, setCount] = useState(0);
> }
> ```
>
> **Rule**: Only extract when there's real value—multiple uses, complex logic, or better organization.

---

> **⚠️ PITFALL: Returning Too Much or Too Little**
>
> Consider what your custom Hook should return. Return what's needed, no more, no less.
>
> ```jsx
> // ❌ CONFUSING: Returns too much unrelated stuff
> function useUser(userId) {
>   const [user, setUser] = useState(null);
>   const [loading, setLoading] = useState(true);
>   const [theme, setTheme] = useState('light'); // Not related to user!
>   const [notifications, setNotifications] = useState([]); // Not related!
>   
>   return { user, loading, theme, notifications }; // Too much!
> }
> ```
>
> ```jsx
> // ✅ BETTER: Focused, returns related values
> function useUser(userId) {
>   const [user, setUser] = useState(null);
>   const [loading, setLoading] = useState(true);
>   const [error, setError] = useState(null);
>   
>   // Fetch user logic
>   
>   return { user, loading, error }; // Focused return
> }
> ```
>
> **Rule**: Custom Hooks should have a single responsibility. Return related values that make sense together.

---

> **⚠️ PITFALL: Not Cleaning Up Effects in Custom Hooks**
>
> If your custom Hook uses Effects, make sure to clean them up properly.
>
> ```jsx
> // ❌ PROBLEM: No cleanup
> function useWindowSize() {
>   const [size, setSize] = useState({ width: 0, height: 0 });
>
>   useEffect(() => {
>     window.addEventListener('resize', () => {
>       setSize({ width: window.innerWidth, height: window.innerHeight });
>     });
>     // No cleanup! Memory leak!
>   }, []);
>
>   return size;
> }
> ```
>
> ```jsx
> // ✅ CORRECT: Proper cleanup
> function useWindowSize() {
>   const [size, setSize] = useState({ width: 0, height: 0 });
>
>   useEffect(() => {
>     function handleResize() {
>       setSize({ width: window.innerWidth, height: window.innerHeight });
>     }
>
>     window.addEventListener('resize', handleResize);
>     return () => {
>       window.removeEventListener('resize', handleResize);
>     };
>   }, []);
>
>   return size;
> }
> ```
>
> **Rule**: Always clean up subscriptions, event listeners, timers, and other side effects in custom Hooks.

---

## Testing Custom Hooks

Custom Hooks can be tested independently using React's testing utilities:

```jsx
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

test('increments counter', () => {
  const { result } = renderHook(() => useCounter());
  
  expect(result.current[0]).toBe(0);
  
  act(() => {
    result.current[1](); // Call setCount
  });
  
  expect(result.current[0]).toBe(1);
});
```

**Benefits of testing Hooks separately:**
- Test logic without rendering components
- Faster tests
- Easier to test edge cases
- Better code coverage

---

## Interview Questions to Consider

### Conceptual Understanding

1. **What is a custom Hook, and why would you use one?**
   - A custom Hook is a JavaScript function that starts with `use` and can call other Hooks. It allows you to extract and reuse component logic across multiple components, making code more maintainable and testable.

2. **Do custom Hooks share state between components?**
   - No. Each call to a custom Hook creates its own isolated state. Custom Hooks share logic, not state. If `ComponentA` and `ComponentB` both call `useOnlineStatus()`, they each get their own independent `isOnline` state.

3. **Why must custom Hooks start with `use`?**
   - The `use` prefix is a React convention that allows the linter to identify Hooks and enforce Hook rules (like calling them at the top level). It also signals to other developers that the function follows Hook rules.

4. **What are the rules for custom Hooks?**
   - Custom Hooks must follow the same rules as built-in Hooks:
     - Only call them at the top level (not conditionally or in loops)
     - Can call other Hooks
     - Must start with `use` prefix

### Practical Application

5. **Create a custom Hook for debouncing a value:**
   ```jsx
   function useDebounce(value, delay) {
     const [debouncedValue, setDebouncedValue] = useState(value);
     
     useEffect(() => {
       const handler = setTimeout(() => {
         setDebouncedValue(value);
       }, delay);
       
       return () => {
         clearTimeout(handler);
       };
     }, [value, delay]);
     
     return debouncedValue;
   }
   ```

6. **What's wrong with this custom Hook?**
   ```jsx
   function getOnlineStatus() {
     const [isOnline, setIsOnline] = useState(true);
     // ...
     return isOnline;
   }
   ```
   - It doesn't start with `use`, so React won't recognize it as a Hook and the linter won't enforce Hook rules. It should be `useOnlineStatus`.

7. **Can you call a custom Hook conditionally?**
   - No, you cannot. Custom Hooks must be called at the top level of your component, in the same order on every render. Conditional calls violate React's Hook rules.

### Advanced Thinking

8. **How would you share state between multiple components using a custom Hook?**
   - You wouldn't—custom Hooks don't share state. Each call is independent. To share state, you'd use Context, a state management library, or lift state up to a common parent.

9. **When should you NOT extract logic into a custom Hook?**
   - Don't extract when:
     - Logic is only used in one component
     - Logic is too simple (just a useState call)
     - Logic is tightly coupled to one component's UI
     - Extracting makes the code harder to understand

10. **How do you test custom Hooks?**
    - Use React's testing utilities like `renderHook` from `@testing-library/react`. This allows you to test Hook logic independently without rendering components.

---

## Summary

Custom Hooks are a powerful way to extract and reuse component logic:

- **Definition**: Functions starting with `use` that can call other Hooks
- **State isolation**: Each call creates its own isolated state—Hooks share logic, not state
- **Naming**: Must start with `use` to follow React conventions
- **Rules**: Follow the same rules as built-in Hooks (top-level calls, no conditionals)
- **When to extract**: When multiple components use the same logic, or when logic is complex
- **Benefits**: Reusability, cleaner components, easier testing, better organization

**Key principles:**

- Each Hook call is independent—state is not shared
- Always name with `use` prefix
- Follow Hook rules (top-level calls)
- Only extract when there's real value
- Always clean up Effects in custom Hooks
- Return focused, related values

Custom Hooks are one of React's most powerful features for code organization and reusability. Use them to keep your components clean and your logic maintainable.


# Extracting State Logic into a Reducer

## You Will Learn

- What a reducer function is and why you might want to use one
- How to refactor from `useState` to `useReducer`
- When using a reducer is advantageous
- How to write well-structured reducers

---

## Consolidating State Logic with a Reducer

As components grow in complexity, managing state updates across multiple event handlers can become challenging. The `useReducer` Hook allows you to consolidate state update logic into a single function called a **reducer**, making your code more organized and easier to maintain.

**Key Benefits of Reducers:**

- **Centralized logic**: All state updates are in one place
- **Predictable updates**: Actions describe what happened, not how to update
- **Easier testing**: Pure functions are easy to test
- **Better for complex state**: Especially when next state depends on previous state

---

## Refactoring from `useState` to `useReducer`

The refactoring process involves three steps: replacing state setters with action dispatches, writing the reducer function, and using the reducer in your component.

### Step 1: Move from Setting State to Dispatching Actions

Instead of directly setting state in event handlers, dispatch **actions** that describe what the user did. Actions are plain JavaScript objects with a `type` property.

**Before (using `useState`):**

```jsx
function handleAddTask(text) {
  setTasks([...tasks, { id: nextId++, text, done: false }]);
}

function handleChangeTask(task) {
  setTasks(
    tasks.map((t) => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    })
  );
}

function handleDeleteTask(taskId) {
  setTasks(tasks.filter((t) => t.id !== taskId));
}
```

**After (using `useReducer`):**

```jsx
function handleAddTask(text) {
  dispatch({
    type: 'added',
    id: nextId++,
    text: text,
  });
}

function handleChangeTask(task) {
  dispatch({
    type: 'changed',
    task: task,
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: 'deleted',
    id: taskId,
  });
}
```

**Key Difference**: Event handlers now describe **what happened** (added, changed, deleted) rather than **how to update the state**. This separation of concerns makes the code more maintainable.

---

### Step 2: Write a Reducer Function

A reducer function specifies **how the state updates in response to each action**. It takes the current state and an action as arguments, and returns the next state.

```jsx
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

**Reducer Rules:**

1. **Pure function**: No side effects, no mutations, no API calls
2. **Returns new state**: Always return a new object/array, never mutate the existing state
3. **Handles all actions**: Include a default case to throw an error for unknown actions
4. **Read-only state**: The reducer should not modify objects or arrays in the state

**Why the switch statement?** Using a switch statement makes it easy to add new action types and clearly shows all possible state transitions.

---

### Step 3: Use the Reducer in Your Component

Replace `useState` with `useReducer` to use your reducer function:

```jsx
import { useReducer } from 'react';

function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  // ... other handlers

  return (
    <>
      <h1>Task List</h1>
      {/* Component JSX */}
    </>
  );
}
```

**`useReducer` takes two arguments:**

1. **Reducer function**: The function that specifies how state updates
2. **Initial state**: The starting state value

**`useReducer` returns:**

1. **Current state**: The current state value
2. **Dispatch function**: A function to dispatch actions

---

## When to Use a Reducer

Consider using `useReducer` instead of `useState` when:

### ✅ Complex State Logic

If you have multiple state values that need to be updated together, or complex update logic:

```jsx
// Multiple useState calls that are related
const [name, setName] = useState('');
const [email, setEmail] = useState('');
const [age, setAge] = useState(0);

// Better: Single reducer managing related state
const [formData, dispatch] = useReducer(formReducer, {
  name: '',
  email: '',
  age: 0,
});
```

### ✅ Next State Depends on Previous State

When the next state calculation requires the previous state:

```jsx
// useState: Can be error-prone
setCount(count + 1); // What if count is stale?
setCount(prevCount => prevCount + 1); // Better, but still verbose

// useReducer: Clear and predictable
dispatch({ type: 'increment' });
// Reducer handles: return { ...state, count: state.count + 1 };
```

### ✅ Many Event Handlers Update State

If you have many event handlers that each update state, centralizing in a reducer improves organization:

```jsx
// useState: Logic scattered across handlers
function handleSubmit() { setState(...); }
function handleChange() { setState(...); }
function handleDelete() { setState(...); }
function handleEdit() { setState(...); }

// useReducer: Logic centralized
function handleSubmit() { dispatch({ type: 'submit' }); }
function handleChange() { dispatch({ type: 'change' }); }
function handleDelete() { dispatch({ type: 'delete' }); }
function handleEdit() { dispatch({ type: 'edit' }); }
```

**When NOT to use a reducer:**

- Simple state updates that don't depend on previous state
- State that's completely independent of other state
- Small components with straightforward logic

---

## Writing a Well-Structured Reducer

### Keep Reducers Pure

Reducers must be **pure functions**—they should only calculate the next state based on the current state and action, without side effects.

```jsx
// ❌ BAD: Side effects in reducer
function reducer(state, action) {
  localStorage.setItem('tasks', JSON.stringify(state)); // Side effect!
  return { ...state, count: state.count + 1 };
}

// ✅ GOOD: Pure function
function reducer(state, action) {
  return { ...state, count: state.count + 1 };
}
// Side effects should be in event handlers or useEffect
```

### Return New State, Don't Mutate

Always return a **new** state object/array. Never mutate the existing state:

```jsx
// ❌ BAD: Mutating state
function reducer(state, action) {
  state.count++; // Mutation!
  return state;
}

// ✅ GOOD: New state object
function reducer(state, action) {
  return { ...state, count: state.count + 1 };
}
```

### Handle Unknown Actions

Always include a default case that throws an error for unknown action types:

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + 1 };
    default:
      throw Error('Unknown action: ' + action.type);
  }
}
```

This helps catch typos and ensures all actions are handled.

### Use Action Objects Consistently

Use consistent action object structures. Common patterns:

```jsx
// Simple action with type only
dispatch({ type: 'reset' });

// Action with payload
dispatch({ type: 'added', id: 1, text: 'Task' });

// Action with nested data
dispatch({ type: 'changed', task: { id: 1, text: 'Updated' } });
```

---

## Pitfalls

> **⚠️ PITFALL: Don't Mutate State in Reducers**
>
> Reducers must be pure functions that return new state objects. Mutating state directly will cause bugs and unpredictable behavior.
>
> ```jsx
> // ❌ NEVER DO THIS
> function tasksReducer(tasks, action) {
>   switch (action.type) {
>     case 'added': {
>       tasks.push({  // Mutation!
>         id: action.id,
>         text: action.text,
>         done: false,
>       });
>       return tasks;  // Returning mutated state
>     }
>     // ...
>   }
> }
> ```
>
> **Why it's wrong**: Mutating state can cause React to miss updates, break time-travel debugging, and lead to unpredictable UI behavior.
>
> ```jsx
> // ✅ DO THIS INSTEAD
> function tasksReducer(tasks, action) {
>   switch (action.type) {
>     case 'added': {
>       return [  // Return new array
>         ...tasks,
>         {
>           id: action.id,
>           text: action.text,
>           done: false,
>         },
>       ];
>     }
>     // ...
>   }
> }
> ```
>
> **Solution**: Always return a new object or array. Use spread operators (`...`), `map`, `filter`, and other immutable array/object methods.

---

> **⚠️ PITFALL: Don't Put Side Effects in Reducers**
>
> Reducers should only compute the next state. API calls, localStorage updates, timers, and other side effects belong in event handlers or `useEffect`.
>
> ```jsx
> // ❌ NEVER DO THIS
> function tasksReducer(tasks, action) {
>   switch (action.type) {
>     case 'added': {
>       fetch('/api/tasks', { method: 'POST', body: JSON.stringify(action) }); // Side effect!
>       return [...tasks, action.task];
>     }
>   }
> }
> ```
>
> ```jsx
> // ✅ DO THIS INSTEAD
> function tasksReducer(tasks, action) {
>   switch (action.type) {
>     case 'added': {
>       return [...tasks, action.task];
>     }
>   }
> }
>
> // Handle side effects in event handlers or useEffect
> function handleAddTask(text) {
>   const newTask = { id: nextId++, text, done: false };
>   dispatch({ type: 'added', task: newTask });
>   // Side effect here is fine
>   fetch('/api/tasks', { method: 'POST', body: JSON.stringify(newTask) });
> }
> ```

---

> **⚠️ PITFALL: Don't Dispatch Actions Outside of Event Handlers or Effects**
>
> You should only call `dispatch` from event handlers or `useEffect`. Don't dispatch during render or in reducer functions.
>
> ```jsx
> // ❌ NEVER DO THIS
> function Component() {
>   const [state, dispatch] = useReducer(reducer, initialState);
>
>   // Don't dispatch during render!
>   if (someCondition) {
>     dispatch({ type: 'update' }); // This will cause infinite loops!
>   }
>
>   return <div>{state.value}</div>;
> }
> ```
>
> ```jsx
> // ✅ DO THIS INSTEAD
> function Component() {
>   const [state, dispatch] = useReducer(reducer, initialState);
>
>   useEffect(() => {
>     if (someCondition) {
>       dispatch({ type: 'update' }); // OK in useEffect
>     }
>   }, [someCondition]);
>
>   return <div>{state.value}</div>;
> }
> ```

---

## Complete Example: Task App with Reducer

Here's a complete example showing a task management app using `useReducer`:

```jsx
import { useReducer } from 'react';

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false },
];

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}
```

---

## Interview Questions to Consider

### Conceptual Understanding

1. **What is a reducer function in React?**
   - A reducer is a pure function that takes the current state and an action, and returns the next state. It centralizes state update logic, making it more predictable and testable.

2. **What's the difference between `useState` and `useReducer`?**
   - `useState` is simpler and better for independent state values with straightforward updates. `useReducer` is better for complex state logic, when the next state depends on the previous state, or when you want to centralize update logic.

3. **What is an "action" in the context of reducers?**
   - An action is a plain JavaScript object that describes what happened (e.g., `{ type: 'added', id: 1, text: 'Task' }`). It contains a `type` property and any additional data needed for the state update.

4. **Why must reducers be pure functions?**
   - Pure functions ensure predictable state updates, make debugging easier, enable time-travel debugging, and prevent side effects that could cause bugs. They always return the same output for the same input.

### Practical Application

5. **Refactor this component from `useState` to `useReducer`:**
   ```jsx
   function Counter() {
     const [count, setCount] = useState(0);
     return (
       <>
         <p>{count}</p>
         <button onClick={() => setCount(count + 1)}>+</button>
         <button onClick={() => setCount(count - 1)}>-</button>
         <button onClick={() => setCount(0)}>Reset</button>
       </>
     );
   }
   ```
   - Solution:
   ```jsx
   function counterReducer(count, action) {
     switch (action.type) {
       case 'increment':
         return count + 1;
       case 'decrement':
         return count - 1;
       case 'reset':
         return 0;
       default:
         throw Error('Unknown action');
     }
   }
   
   function Counter() {
     const [count, dispatch] = useReducer(counterReducer, 0);
     return (
       <>
         <p>{count}</p>
         <button onClick={() => dispatch({ type: 'increment' })}>+</button>
         <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
         <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
       </>
     );
   }
   ```

6. **What's wrong with this reducer?**
   ```jsx
   function reducer(state, action) {
     if (action.type === 'increment') {
       state.count++;
       return state;
     }
     return state;
   }
   ```
   - The reducer is mutating state directly. It should return a new state object: `return { ...state, count: state.count + 1 };`

7. **When would you choose `useReducer` over `useState`?**
   - Choose `useReducer` when:
     - State logic is complex with multiple related values
     - The next state depends on the previous state
     - You have many event handlers updating state
     - You want to centralize and test state update logic
   - Choose `useState` for simple, independent state values

### Advanced Thinking

8. **Can you have side effects in a reducer?**
   - No, reducers must be pure functions. Side effects (API calls, localStorage, timers) should be in event handlers or `useEffect` hooks. The reducer should only compute the next state.

9. **How do you handle asynchronous operations with reducers?**
   - Reducers handle synchronous state updates only. For async operations:
     - Dispatch an action when the async operation starts
     - Perform the async operation in an event handler or `useEffect`
     - Dispatch another action with the result when it completes
     - The reducer handles both the "loading" and "completed" states

10. **Can you use multiple reducers in one component?**
    - Yes, you can call `useReducer` multiple times in a component, just like `useState`. Each reducer manages its own piece of state independently.

---

## Summary

Using `useReducer` allows you to manage complex state logic in a more organized and predictable way:

- **Reducers centralize state updates**: All state transition logic lives in one place
- **Actions describe events**: Event handlers dispatch actions describing what happened, not how to update state
- **Pure functions**: Reducers are easy to test and debug because they're pure
- **Better for complex state**: Especially useful when state has multiple related values or depends on previous state

The refactoring process involves three steps: replace state setters with dispatch calls, write a reducer function, and use `useReducer` in your component. Remember to keep reducers pure, avoid mutations, and handle all action types properly.


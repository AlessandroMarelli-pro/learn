# Scaling Up with Reducer and Context

## You Will Learn

- How to combine a reducer with context to manage complex state
- How to avoid passing state and dispatch through props (prop drilling)
- How to keep context and state logic in a separate file for better organization

---

## The Problem: Prop Drilling with Reducers

When using `useReducer`, you might find yourself passing both the state and the `dispatch` function down through many component layers via props. This is called **prop drilling**, and it can make your code cumbersome and hard to maintain.

```jsx
// Prop drilling example - state and dispatch passed through many levels
function App() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  return <AddTask onAddTask={handleAddTask} />;
}

function AddTask({ onAddTask }) {
  // onAddTask was passed from App through multiple layers
  return <button onClick={() => onAddTask('New task')}>Add</button>;
}
```

**Solution**: Combine `useReducer` with Context to avoid prop drilling entirely.

---

## Combining Reducer with Context

By combining a reducer with context, you can:

- **Centralize state management**: Use a reducer for complex state logic
- **Avoid prop drilling**: Use context to provide state and dispatch to any component
- **Scale your app**: Handle complex state management patterns cleanly

### Step 1: Create the Context

Create a context to hold both the state and the dispatch function:

```jsx
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

**Why two contexts?** Splitting state and dispatch into separate contexts helps optimize performance. Components that only need to dispatch actions won't re-render when state changes.

### Step 2: Put State and Dispatch into Context

Create a provider component that uses `useReducer` and provides both values:

```jsx
import { useReducer } from 'react';
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

### Step 3: Use the Context Anywhere in the Tree

Any component can now access state or dispatch without prop drilling:

```jsx
import { useContext } from 'react';
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

function TaskList() {
  const tasks = useContext(TasksContext);
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>{task.text}</li>
      ))}
    </ul>
  );
}

function AddTask() {
  const dispatch = useContext(TasksDispatchContext);
  return (
    <button
      onClick={() => {
        dispatch({
          type: 'added',
          id: nextId++,
          text: 'New task',
        });
      }}
    >
      Add task
    </button>
  );
}
```

---

## Moving Everything Into One File

For better organization, you can move all the context and state logic into a single dedicated file.

### Complete Example: TasksContext.js

```jsx
import { createContext, useContext, useReducer } from 'react';

// Contexts
const TasksContext = createContext(null);
const TasksDispatchContext = createContext(null);

// Reducer function
export function tasksReducer(tasks, action) {
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

// Initial state
const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false },
];

// Provider component
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

// Custom hooks for consuming context
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}
```

### Using the Custom Hooks

Components can now use the custom hooks for cleaner code:

```jsx
import { useTasks, useTasksDispatch } from './TasksContext.js';

function TaskList() {
  const tasks = useTasks();
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>{task.text}</li>
      ))}
    </ul>
  );
}

function AddTask() {
  const dispatch = useTasksDispatch();
  return (
    <button
      onClick={() => {
        dispatch({
          type: 'added',
          id: nextId++,
          text: 'New task',
        });
      }}
    >
      Add task
    </button>
  );
}
```

### Wrapping the App

Wrap your app with the provider at a high level:

```jsx
import { TasksProvider } from './TasksContext.js';

export default function App() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

---

## Why Split State and Dispatch Contexts?

Splitting state and dispatch into separate contexts is an **optimization pattern**:

**Benefits:**

- **Performance**: Components that only dispatch actions won't re-render when state changes
- **Flexibility**: Components can choose to subscribe to state, dispatch, or both
- **Better separation**: Clear distinction between reading state and modifying it

**Single Context (simpler, but less optimized):**

```jsx
// ❌ All consumers re-render on any state change
const TasksContext = createContext(null);

<TasksContext.Provider value={{ tasks, dispatch }}>
  {children}
</TasksContext.Provider>
```

**Split Contexts (optimized):**

```jsx
// ✅ Only state consumers re-render on state changes
// Dispatch consumers never re-render
<TasksContext.Provider value={tasks}>
  <TasksDispatchContext.Provider value={dispatch}>
    {children}
  </TasksDispatchContext.Provider>
</TasksContext.Provider>
```

**When to use each:**

- **Single context**: For simple apps or when most components need both state and dispatch
- **Split contexts**: For performance-sensitive apps or when many components only dispatch actions

---

## Pitfalls

> **⚠️ PITFALL: Don't Provide Default Values for Context Used with useReducer**
>
> When using context with `useReducer`, the context will always be provided by the Provider component. Don't provide default values that might mask missing Provider errors.
>
> ```jsx
> // ❌ AVOID: Default values hide missing Provider errors
> const TasksContext = createContext([]);  // Default to empty array
> const TasksDispatchContext = createContext(() => {});  // Default to no-op function
> ```
>
> If a component uses the context without a Provider, it will silently use the default value, making bugs hard to catch.
>
> ```jsx
> // ✅ DO THIS: null default, throw error if Provider missing
> const TasksContext = createContext(null);
> const TasksDispatchContext = createContext(null);
>
> export function useTasks() {
>   const tasks = useContext(TasksContext);
>   if (tasks === null) {
>     throw new Error('useTasks must be used within TasksProvider');
>   }
>   return tasks;
> }
> ```
>
> **Why this helps**: The custom hook throws a clear error if used outside the Provider, making bugs immediately obvious.

---

> **⚠️ PITFALL: Don't Create Context Value Objects in Render**
>
> Creating new objects in the Provider's value prop on every render causes all consumers to re-render, even if the state hasn't changed.
>
> ```jsx
> // ❌ AVOID: New object created every render
> export function TasksProvider({ children }) {
>   const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
>   
>   return (
>     <TasksContext.Provider value={{ tasks, dispatch }}>
>       {children}
>     </TasksContext.Provider>
>   );
> }
> ```
>
> Every render creates a new `{ tasks, dispatch }` object, causing unnecessary re-renders.
>
> ```jsx
> // ✅ DO THIS: Split contexts or use stable references
> export function TasksProvider({ children }) {
>   const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
>   
>   return (
>     <TasksContext.Provider value={tasks}>
>       <TasksDispatchContext.Provider value={dispatch}>
>         {children}
>       </TasksDispatchContext.Provider>
>     </TasksContext.Provider>
>   );
> }
> ```
>
> `dispatch` from `useReducer` is stable (never changes), and `tasks` only changes when the reducer returns new state. This prevents unnecessary re-renders.

---

> **⚠️ PITFALL: Don't Put Context Provider Too High in the Tree**
>
> Providing context at the root level when only a subtree needs it causes unnecessary re-renders and makes the context available where it shouldn't be.
>
> ```jsx
> // ❌ AVOID: Provider at root when only part of app needs it
> function App() {
>   return (
>     <TasksProvider>
>       <Header />  {/* Doesn't need tasks context */}
>       <Sidebar />  {/* Doesn't need tasks context */}
>       <MainContent>  {/* Only this needs tasks context */}
>         <TaskList />
>       </MainContent>
>     </TasksProvider>
>   );
> }
> ```
>
> ```jsx
> // ✅ DO THIS: Provider closest to where it's needed
> function App() {
>   return (
>     <>
>       <Header />
>       <Sidebar />
>       <TasksProvider>
>         <MainContent>
>           <TaskList />
>         </MainContent>
>       </TasksProvider>
>     </>
>   );
> }
> ```
>
> **Why this matters**: Keeping providers closer to consumers improves performance and prevents accidental access to context where it shouldn't be used.

---

> **⚠️ PITFALL: Don't Mix Multiple Concerns in One Reducer**
>
> Putting unrelated state updates in a single reducer makes the code harder to understand and maintain.
>
> ```jsx
> // ❌ AVOID: Multiple concerns in one reducer
> function appReducer(state, action) {
>   switch (action.type) {
>     case 'added_task':
>       return { ...state, tasks: [...state.tasks, action.task] };
>     case 'changed_user':
>       return { ...state, user: action.user };
>     case 'updated_settings':
>       return { ...state, settings: action.settings };
>     // ...
>   }
> }
> ```
>
> ```jsx
> // ✅ DO THIS: Separate reducers for separate concerns
> function tasksReducer(tasks, action) {
>   // Only task-related logic
> }
>
> function userReducer(user, action) {
>   // Only user-related logic
> }
>
> function settingsReducer(settings, action) {
>   // Only settings-related logic
> }
>
> // Use separate contexts/providers for each
> <TasksProvider>
>   <UserProvider>
>     <SettingsProvider>
>       {children}
>     </SettingsProvider>
>   </UserProvider>
> </TasksProvider>
> ```
>
> **Why this helps**: Each reducer focuses on one concern, making code easier to test, debug, and maintain.

---

## Custom Hooks Pattern

Creating custom hooks for consuming context is a best practice:

```jsx
export function useTasks() {
  const context = useContext(TasksContext);
  if (context === null) {
    throw new Error('useTasks must be used within TasksProvider');
  }
  return context;
}

export function useTasksDispatch() {
  const context = useContext(TasksDispatchContext);
  if (context === null) {
    throw new Error('useTasksDispatch must be used within TasksProvider');
  }
  return context;
}
```

**Benefits:**

- **Error handling**: Clear error messages if used outside Provider
- **Type safety**: Better TypeScript support
- **Convenience**: Easier to import and use
- **Consistency**: Standardized way to consume context

**Usage:**

```jsx
// Instead of: useContext(TasksContext)
// Use: useTasks()
const tasks = useTasks();
const dispatch = useTasksDispatch();
```

---

## Complete Example: Task Management App

Here's a complete example showing the reducer + context pattern:

**TasksContext.js:**

```jsx
import { createContext, useContext, useReducer } from 'react';

const TasksContext = createContext(null);
const TasksDispatchContext = createContext(null);

export function tasksReducer(tasks, action) {
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

const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false },
];

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}

export function useTasks() {
  const context = useContext(TasksContext);
  if (context === null) {
    throw new Error('useTasks must be used within TasksProvider');
  }
  return context;
}

export function useTasksDispatch() {
  const context = useContext(TasksDispatchContext);
  if (context === null) {
    throw new Error('useTasksDispatch must be used within TasksProvider');
  }
  return context;
}
```

**App.js:**

```jsx
import { TasksProvider } from './TasksContext.js';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function App() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

**AddTask.js:**

```jsx
import { useState } from 'react';
import { useTasksDispatch } from './TasksContext.js';

let nextId = 3;

export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useTasksDispatch();

  return (
    <>
      <input
        placeholder="Add task"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={() => {
          setText('');
          dispatch({
            type: 'added',
            id: nextId++,
            text: text,
          });
        }}
      >
        Add
      </button>
    </>
  );
}
```

**TaskList.js:**

```jsx
import { useTasks, useTasksDispatch } from './TasksContext.js';

export default function TaskList() {
  const tasks = useTasks();
  const dispatch = useTasksDispatch();

  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}

function Task({ task }) {
  const dispatch = useTasksDispatch();
  const [isEditing, setIsEditing] = useState(false);
  const [taskText, setTaskText] = useState(task.text);

  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => {
          dispatch({
            type: 'changed',
            task: {
              ...task,
              done: e.target.checked,
            },
          });
        }}
      />
      {isEditing ? (
        <>
          <input
            value={taskText}
            onChange={(e) => setTaskText(e.target.value)}
          />
          <button
            onClick={() => {
              dispatch({
                type: 'changed',
                task: {
                  ...task,
                  text: taskText,
                },
              });
              setIsEditing(false);
            }}
          >
            Save
          </button>
        </>
      ) : (
        <>
          {task.text}
          <button onClick={() => setIsEditing(true)}>Edit</button>
        </>
      )}
      <button
        onClick={() => {
          dispatch({
            type: 'deleted',
            id: task.id,
          });
        }}
      >
        Delete
      </button>
    </label>
  );
}
```

---

## When to Use This Pattern

Use reducer + context when:

### ✅ Complex State Management

You have complex state logic that benefits from a reducer, and multiple components need access to it:

```jsx
// Good candidate: Shopping cart with complex logic
<CartProvider>
  <ProductList />  {/* Reads cart state */}
  <CartSummary />  {/* Reads and calculates from cart */}
  <CheckoutButton />  {/* Dispatches cart actions */}
</CartProvider>
```

### ✅ Avoiding Prop Drilling

You're passing state and setters through many component layers:

```jsx
// Prop drilling problem
<App>
  <Layout state={state} setState={setState}>
    <Header state={state} setState={setState}>
      <Nav state={state} setState={setState}>
        <MenuItem state={state} setState={setState} />
      </Nav>
    </Header>
  </Layout>
</App>

// Solution with context
<AppStateProvider>
  <Layout>
    <Header>
      <Nav>
        <MenuItem />  {/* Uses context directly */}
      </Nav>
    </Header>
  </Layout>
</AppStateProvider>
```

### ✅ Shared State Across Many Components

Multiple components at different tree levels need to read or modify the same state:

```jsx
// Theme state used everywhere
<ThemeProvider>
  <App />  {/* All components can access theme */}
</ThemeProvider>
```

**When NOT to use this pattern:**

- **Simple local state**: Use `useState` for component-specific state
- **Parent-child only**: If only parent and child need the state, props are simpler
- **Global state library**: For very large apps, consider Redux or Zustand

---

## Interview Questions to Consider

### Conceptual Understanding

1. **What is the benefit of combining `useReducer` with Context?**
   - Combining them allows you to manage complex state with a reducer while avoiding prop drilling. Context makes state and dispatch available to any component in the tree without passing props through every level.

2. **Why split state and dispatch into separate contexts?**
   - Performance optimization. Components that only dispatch actions won't re-render when state changes. This prevents unnecessary re-renders in components that only need to trigger actions.

3. **What is prop drilling, and how does Context solve it?**
   - Prop drilling is passing props through multiple component layers even when intermediate components don't use them. Context solves it by allowing components to access values directly without prop passing.

4. **When should you keep context and reducer logic in a separate file?**
   - When the state logic is complex, used across multiple components, or when you want to improve code organization and reusability. Separating concerns makes the codebase more maintainable.

### Practical Application

5. **Implement a context provider that uses `useReducer` to manage a counter state:**
   ```jsx
   import { createContext, useContext, useReducer } from 'react';
   
   const CounterContext = createContext(null);
   const CounterDispatchContext = createContext(null);
   
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
   
   export function CounterProvider({ children }) {
     const [count, dispatch] = useReducer(counterReducer, 0);
     return (
       <CounterContext.Provider value={count}>
         <CounterDispatchContext.Provider value={dispatch}>
           {children}
         </CounterDispatchContext.Provider>
       </CounterContext.Provider>
     );
   }
   
   export function useCounter() {
     const context = useContext(CounterContext);
     if (context === null) {
       throw new Error('useCounter must be used within CounterProvider');
     }
     return context;
   }
   
   export function useCounterDispatch() {
     const context = useContext(CounterDispatchContext);
     if (context === null) {
       throw new Error('useCounterDispatch must be used within CounterProvider');
     }
     return context;
   }
   ```

6. **What's wrong with this context implementation?**
   ```jsx
   function AppProvider({ children }) {
     const [state, dispatch] = useReducer(reducer, initialState);
     return (
       <AppContext.Provider value={{ state, dispatch }}>
         {children}
       </AppContext.Provider>
     );
   }
   ```
   - Creating a new object `{ state, dispatch }` on every render causes all consumers to re-render unnecessarily. Use split contexts or memoize the value object.

7. **How would you refactor this to use context instead of prop drilling?**
   ```jsx
   function App() {
     const [user, setUser] = useState(null);
     return <Layout user={user} setUser={setUser} />;
   }
   
   function Layout({ user, setUser }) {
     return <Header user={user} setUser={setUser} />;
   }
   
   function Header({ user, setUser }) {
     return <UserMenu user={user} setUser={setUser} />;
   }
   ```
   - Create a UserContext and UserProvider, then use `useContext` in components that need user state, eliminating the need to pass props through Layout and Header.

### Advanced Thinking

8. **How do you handle multiple reducers with context?**
   - Create separate contexts and providers for each concern. Nest providers if needed:
   ```jsx
   <TasksProvider>
     <UserProvider>
       <ThemeProvider>
         {children}
       </ThemeProvider>
     </UserProvider>
   </TasksProvider>
   ```
   Keep reducers separate for different concerns rather than combining everything into one mega-reducer.

9. **What performance considerations should you keep in mind when using context with reducers?**
   - Split state and dispatch contexts to prevent unnecessary re-renders. Keep providers as close to consumers as possible. Consider memoizing expensive computations. Avoid creating new objects/arrays in the provider value unless state actually changed.

10. **Can you use `useReducer` with multiple contexts in the same component?**
    - Yes, you can use multiple context providers and access multiple contexts in a component:
    ```jsx
    function MyComponent() {
      const tasks = useTasks();
      const user = useUser();
      const theme = useTheme();
      // Use all three contexts
    }
    ```

---

## Summary

Combining `useReducer` with Context provides a powerful pattern for managing complex state in React applications:

- **Eliminates prop drilling**: Context makes state and dispatch available anywhere in the tree
- **Centralizes state logic**: Reducers organize complex state updates in one place
- **Improves organization**: Separate files for context and reducer logic keep code maintainable
- **Optimizes performance**: Split contexts prevent unnecessary re-renders
- **Scales well**: This pattern works for applications of all sizes

The pattern involves creating contexts for state and dispatch, a Provider component using `useReducer`, and custom hooks for consuming the context. Remember to avoid common pitfalls like creating new objects in render, providing unnecessary default values, and mixing unrelated concerns in one reducer.


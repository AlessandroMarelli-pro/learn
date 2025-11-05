# React Documentation Summary

This directory contains comprehensive summaries of the official React documentation, organized by topic. Each summary follows a consistent format with learning objectives, core concepts, code examples, pitfalls, and interview questions.

## Overview

React is a JavaScript library for building user interfaces. It allows you to build complex UIs from small, reusable pieces called components. This documentation covers the essential concepts you need to master React, from creating your first component to managing complex state and side effects.

---

### Important topics summary

I'll create a comprehensive React interview preparation guide with both simple and technical explanations.

# React - Interview Preparation

## What is React?

### Simple Explanation

React is a JavaScript library for building user interfaces, especially for web applications. Think of it like LEGO blocks - you build small, reusable pieces (components) and combine them to create complex interfaces.

**Example:** A Facebook page has many parts - header, sidebar, post feed, comments. Each is a separate React component that can be reused and updated independently.

**Key idea:** When data changes, React automatically updates the parts of the page that need to change, without reloading the entire page.

### Technical Explanation

React is a declarative, component-based JavaScript library developed by Facebook (Meta) in 2013 for building user interfaces. It implements a virtual DOM for efficient updates and uses a unidirectional data flow pattern.

**Core characteristics:**

- **Component-based:** UI built from isolated, reusable pieces
- **Declarative:** Describe what UI should look like, React handles updates
- **Virtual DOM:** Efficient diffing algorithm for minimal DOM updates
- **JSX:** JavaScript syntax extension for writing HTML-like code
- **Unidirectional data flow:** Data flows down, events flow up
- **React is a library, not a framework:** Focused on view layer only

**Virtual DOM Process:**

1. State changes trigger re-render
2. React creates new virtual DOM tree
3. Diffs new tree with previous tree (reconciliation)
4. Calculates minimal changes needed
5. Batches updates to real DOM

---

## Components

### Simple Explanation

Components are like custom HTML tags that you create. Instead of just using `<div>` and `<button>`, you can create `<UserProfile>`, `<ShoppingCart>`, or `<CommentSection>`.

**Example:**

```jsx
// Reusable button component
<Button color="blue" size="large">Click Me</Button>
<Button color="red" size="small">Delete</Button>
```

### Technical Explanation

#### Functional Components (Modern Approach)

```jsx
// Simple functional component
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

// Arrow function component
const Welcome = (props) => {
  return <h1>Hello, {props.name}</h1>;
};

// With destructuring
const Welcome = ({ name }) => {
  return <h1>Hello, {name}</h1>;
};

// Implicit return (no curly braces)
const Welcome = ({ name }) => <h1>Hello, {name}</h1>;
```

#### Class Components (Legacy, but still in use)

```jsx
import React, { Component } from 'react';

class Welcome extends Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

#### Real-world Component Example

```jsx
// UserCard component
function UserCard({ user }) {
  return (
    <div className="user-card">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.email}</p>
      <button onClick={() => console.log('Follow', user.id)}>Follow</button>
    </div>
  );
}

// Usage
function App() {
  const user = {
    id: 1,
    name: 'John Doe',
    email: 'john@example.com',
    avatar: 'https://example.com/avatar.jpg',
  };

  return <UserCard user={user} />;
}
```

#### Component Composition

```jsx
// Small, focused components
function Avatar({ src, alt }) {
  return <img className="avatar" src={src} alt={alt} />;
}

function UserInfo({ name, email }) {
  return (
    <div className="user-info">
      <h2>{name}</h2>
      <p>{email}</p>
    </div>
  );
}

// Composed component
function UserCard({ user }) {
  return (
    <div className="user-card">
      <Avatar src={user.avatar} alt={user.name} />
      <UserInfo name={user.name} email={user.email} />
      <button>Follow</button>
    </div>
  );
}
```

---

## Props

### Simple Explanation

Props (properties) are how you pass data from a parent component to a child component. It's like function arguments - the parent gives information to the child.

**Example:** Like telling a delivery driver the address - you pass the address as a prop.

```jsx
<DeliveryDriver address="123 Main St" />
```

### Technical Explanation

Props are immutable (read-only) data passed from parent to child. They flow in one direction (top-down).

```jsx
// Parent component
function App() {
  const userName = 'Alice';
  const userAge = 25;

  return <UserProfile name={userName} age={userAge} />;
}

// Child component
function UserProfile(props) {
  return (
    <div>
      <h1>{props.name}</h1>
      <p>Age: {props.age}</p>
    </div>
  );
}

// With destructuring (preferred)
function UserProfile({ name, age }) {
  return (
    <div>
      <h1>{name}</h1>
      <p>Age: {age}</p>
    </div>
  );
}
```

#### Default Props

```jsx
function Button({ text = 'Click me', color = 'blue' }) {
  return <button className={`btn-${color}`}>{text}</button>;
}

// Usage
<Button /> // Uses defaults
<Button text="Submit" color="green" />
```

#### Props Validation (PropTypes)

```jsx
import PropTypes from 'prop-types';

function UserProfile({ name, age, email, isActive }) {
  return (
    <div>
      <h1>{name}</h1>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
      {isActive && <span>Active</span>}
    </div>
  );
}

UserProfile.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number.isRequired,
  email: PropTypes.string,
  isActive: PropTypes.bool,
};

UserProfile.defaultProps = {
  isActive: false,
};
```

#### Children Prop

```jsx
// Component that wraps content
function Card({ children, title }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="card-content">{children}</div>
    </div>
  );
}

// Usage
<Card title="User Information">
  <p>Name: John Doe</p>
  <p>Email: john@example.com</p>
  <button>Edit</button>
</Card>;
```

#### Render Props Pattern

```jsx
function DataFetcher({ url, render }) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch(url)
      .then((res) => res.json())
      .then((data) => {
        setData(data);
        setLoading(false);
      });
  }, [url]);

  return render({ data, loading });
}

// Usage
<DataFetcher
  url="/api/users"
  render={({ data, loading }) =>
    loading ? <p>Loading...</p> : <UserList users={data} />
  }
/>;
```

---

## State

### Simple Explanation

State is data that can change over time within a component. It's like the component's memory. When state changes, the component re-renders to show the new data.

**Example:** A light switch - its state is either "on" or "off", and clicking it changes the state.

### Technical Explanation

State is mutable data managed within a component. Changes to state trigger re-renders.

#### useState Hook

```jsx
import { useState } from 'react';

function Counter() {
  // Declare state variable
  const [count, setCount] = useState(0);

  // count: current state value
  // setCount: function to update state
  // 0: initial value

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

#### Multiple State Variables

```jsx
function UserForm() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({ name, email, age });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Name"
      />
      <input
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="number"
        value={age}
        onChange={(e) => setAge(Number(e.target.value))}
        placeholder="Age"
      />
      <button type="submit">Submit</button>
    </form>
  );
}
```

#### Object State

```jsx
function UserProfile() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0,
  });

  const updateField = (field, value) => {
    setUser((prevUser) => ({
      ...prevUser,
      [field]: value,
    }));
  };

  return (
    <div>
      <input
        value={user.name}
        onChange={(e) => updateField('name', e.target.value)}
        placeholder="Name"
      />
      <input
        value={user.email}
        onChange={(e) => updateField('email', e.target.value)}
        placeholder="Email"
      />
      <input
        type="number"
        value={user.age}
        onChange={(e) => updateField('age', Number(e.target.value))}
        placeholder="Age"
      />
    </div>
  );
}
```

#### Array State

```jsx
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');

  const addTodo = () => {
    if (input.trim()) {
      setTodos([...todos, { id: Date.now(), text: input, completed: false }]);
      setInput('');
    }
  };

  const toggleTodo = (id) => {
    setTodos(
      todos.map((todo) =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo,
      ),
    );
  };

  const deleteTodo = (id) => {
    setTodos(todos.filter((todo) => todo.id !== id));
  };

  return (
    <div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && addTodo()}
      />
      <button onClick={addTodo}>Add</button>

      <ul>
        {todos.map((todo) => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span
              style={{
                textDecoration: todo.completed ? 'line-through' : 'none',
              }}
            >
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

#### State Updates are Asynchronous

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  const incrementThreeTimes = () => {
    // WRONG - won't work as expected
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    // Only increments by 1 because all use the same 'count' value

    // CORRECT - use functional update
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
    setCount((prev) => prev + 1);
    // Increments by 3
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementThreeTimes}>+3</button>
    </div>
  );
}
```

#### Lazy Initial State

```jsx
// Expensive computation
function getInitialState() {
  console.log('Computing initial state...');
  return Array(1000)
    .fill(0)
    .map((_, i) => i);
}

function Component() {
  // BAD - runs on every render
  const [data, setData] = useState(getInitialState());

  // GOOD - only runs once
  const [data, setData] = useState(() => getInitialState());

  return <div>{data.length}</div>;
}
```

---

## Lifecycle & Effects

### Simple Explanation

Components have a lifecycle - they are created (mounted), updated, and removed (unmounted). Effects let you run code at specific points in this lifecycle.

**Example:** Like a TV - when you turn it on (mount), it shows content (render), you can change channels (update), and when you turn it off (unmount), it stops.

### Technical Explanation

#### useEffect Hook

```jsx
import { useEffect, useState } from 'react';

function Component() {
  const [count, setCount] = useState(0);

  // Runs after every render
  useEffect(() => {
    console.log('Component rendered');
  });

  // Runs only on mount (once)
  useEffect(() => {
    console.log('Component mounted');
  }, []); // Empty dependency array

  // Runs when count changes
  useEffect(() => {
    console.log('Count changed to:', count);
  }, [count]); // Dependency array with count

  // Cleanup function
  useEffect(() => {
    console.log('Setting up...');

    return () => {
      console.log('Cleaning up...');
    };
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

#### Data Fetching

```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    const fetchUser = async () => {
      try {
        setLoading(true);
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();

        if (!cancelled) {
          setUser(data);
          setError(null);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err.message);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };

    fetchUser();

    // Cleanup function to prevent state updates after unmount
    return () => {
      cancelled = true;
    };
  }, [userId]); // Re-fetch when userId changes

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;
  if (!user) return <p>No user found</p>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

#### Event Listeners

```jsx
function WindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    // Add event listener
    window.addEventListener('resize', handleResize);

    // Cleanup - remove event listener
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []); // Empty array = only setup/cleanup once

  return (
    <div>
      Window size: {size.width} x {size.height}
    </div>
  );
}
```

#### Timers

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(false);

  useEffect(() => {
    if (!isRunning) return;

    const interval = setInterval(() => {
      setSeconds((prev) => prev + 1);
    }, 1000);

    // Cleanup - clear interval when component unmounts or isRunning changes
    return () => clearInterval(interval);
  }, [isRunning]);

  return (
    <div>
      <p>Seconds: {seconds}</p>
      <button onClick={() => setIsRunning(!isRunning)}>
        {isRunning ? 'Pause' : 'Start'}
      </button>
      <button onClick={() => setSeconds(0)}>Reset</button>
    </div>
  );
}
```

#### Class Component Lifecycle (Legacy)

```jsx
class LifecycleDemo extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    console.log('1. Constructor');
  }

  componentDidMount() {
    console.log('3. Component Mounted');
    // Like useEffect with []
    // Good for: API calls, subscriptions
  }

  componentDidUpdate(prevProps, prevState) {
    console.log('5. Component Updated');
    // Like useEffect with dependencies
    if (prevState.count !== this.state.count) {
      console.log('Count changed');
    }
  }

  componentWillUnmount() {
    console.log('6. Component Will Unmount');
    // Like useEffect cleanup
    // Good for: cleanup, removing listeners
  }

  render() {
    console.log('2. Render / 4. Re-render');
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Increment
        </button>
      </div>
    );
  }
}
```

---

## Hooks

### Simple Explanation

Hooks are special functions that let you "hook into" React features in functional components. They all start with "use" (useState, useEffect, etc.).

Before hooks (pre-2019), you needed class components to use state and lifecycle methods. Hooks made functional components more powerful.

### Technical Explanation

#### Built-in Hooks Overview

| Hook                  | Purpose                            |
| --------------------- | ---------------------------------- |
| `useState`            | Add state to functional components |
| `useEffect`           | Side effects (lifecycle)           |
| `useContext`          | Access context values              |
| `useReducer`          | Complex state logic                |
| `useCallback`         | Memoize functions                  |
| `useMemo`             | Memoize values                     |
| `useRef`              | Persist values, DOM references     |
| `useLayoutEffect`     | Synchronous effects                |
| `useImperativeHandle` | Customize ref exposure             |
| `useDebugValue`       | Custom hook debugging              |

#### useReducer

```jsx
import { useReducer } from 'react';

// Reducer function
function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    case 'set':
      return { count: action.payload };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'set', payload: 10 })}>
        Set to 10
      </button>
    </div>
  );
}
```

#### Complex State with useReducer

```jsx
const initialState = {
  todos: [],
  filter: 'all',
};

function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: Date.now(),
            text: action.payload,
            completed: false,
          },
        ],
      };

    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo,
        ),
      };

    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter((todo) => todo.id !== action.payload),
      };

    case 'SET_FILTER':
      return {
        ...state,
        filter: action.payload,
      };

    default:
      return state;
  }
}

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  const [input, setInput] = useState('');

  const addTodo = () => {
    if (input.trim()) {
      dispatch({ type: 'ADD_TODO', payload: input });
      setInput('');
    }
  };

  const filteredTodos = state.todos.filter((todo) => {
    if (state.filter === 'active') return !todo.completed;
    if (state.filter === 'completed') return todo.completed;
    return true;
  });

  return (
    <div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && addTodo()}
      />
      <button onClick={addTodo}>Add</button>

      <div>
        <button
          onClick={() => dispatch({ type: 'SET_FILTER', payload: 'all' })}
        >
          All
        </button>
        <button
          onClick={() => dispatch({ type: 'SET_FILTER', payload: 'active' })}
        >
          Active
        </button>
        <button
          onClick={() => dispatch({ type: 'SET_FILTER', payload: 'completed' })}
        >
          Completed
        </button>
      </div>

      <ul>
        {filteredTodos.map((todo) => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() =>
                dispatch({ type: 'TOGGLE_TODO', payload: todo.id })
              }
            />
            <span
              style={{
                textDecoration: todo.completed ? 'line-through' : 'none',
              }}
            >
              {todo.text}
            </span>
            <button
              onClick={() =>
                dispatch({ type: 'DELETE_TODO', payload: todo.id })
              }
            >
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

#### useRef

```jsx
import { useRef, useEffect, useState } from 'react';

function Examples() {
  // 1. DOM reference
  const inputRef = useRef(null);

  const focusInput = () => {
    inputRef.current.focus();
  };

  // 2. Persist value without re-rendering
  const renderCount = useRef(0);

  useEffect(() => {
    renderCount.current += 1;
  });

  // 3. Store previous value
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();

  useEffect(() => {
    prevCountRef.current = count;
  }, [count]);

  const prevCount = prevCountRef.current;

  return (
    <div>
      <h3>1. DOM Reference</h3>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>

      <h3>2. Render Count (doesn't cause re-render)</h3>
      <p>This component has rendered {renderCount.current} times</p>

      <h3>3. Previous Value</h3>
      <p>
        Current: {count}, Previous: {prevCount}
      </p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

#### useMemo

```jsx
import { useMemo, useState } from 'react';

function ExpensiveComponent() {
  const [count, setCount] = useState(0);
  const [input, setInput] = useState('');

  // Expensive calculation
  const expensiveValue = useMemo(() => {
    console.log('Computing expensive value...');
    let result = 0;
    for (let i = 0; i < 1000000000; i++) {
      result += i;
    }
    return result + count;
  }, [count]); // Only recompute when count changes

  // This won't trigger recomputation when input changes
  return (
    <div>
      <p>Expensive Value: {expensiveValue}</p>
      <button onClick={() => setCount(count + 1)}>Increment Count</button>

      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="Type here (won't trigger expensive calculation)"
      />
    </div>
  );
}
```

#### useCallback

```jsx
import { useCallback, useState, memo } from 'react';

// Child component (memoized)
const ExpensiveChild = memo(({ onItemClick }) => {
  console.log('ExpensiveChild rendered');
  return (
    <div>
      {Array(100)
        .fill(0)
        .map((_, i) => (
          <button key={i} onClick={() => onItemClick(i)}>
            Item {i}
          </button>
        ))}
    </div>
  );
});

function Parent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([]);

  // Without useCallback - new function on every render
  // const handleItemClick = (id) => {
  //   console.log('Item clicked:', id);
  // };

  // With useCallback - same function reference
  const handleItemClick = useCallback((id) => {
    console.log('Item clicked:', id);
    setItems((prev) => [...prev, id]);
  }, []); // No dependencies = never recreated

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>

      {/* ExpensiveChild won't re-render when count changes */}
      <ExpensiveChild onItemClick={handleItemClick} />
    </div>
  );
}
```

#### Custom Hooks

```jsx
// Custom hook for form handling
function useForm(initialValues) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});

  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues((prev) => ({
      ...prev,
      [name]: value,
    }));
  };

  const resetForm = () => {
    setValues(initialValues);
    setErrors({});
  };

  return {
    values,
    errors,
    setErrors,
    handleChange,
    resetForm,
  };
}

// Usage
function LoginForm() {
  const { values, handleChange, resetForm } = useForm({
    email: '',
    password: '',
  });

  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Submitting:', values);
    resetForm();
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="email"
        value={values.email}
        onChange={handleChange}
        placeholder="Email"
      />
      <input
        name="password"
        type="password"
        value={values.password}
        onChange={handleChange}
        placeholder="Password"
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

#### Custom Hook - Data Fetching

```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    let cancelled = false;

    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        const json = await response.json();

        if (!cancelled) {
          setData(json);
          setError(null);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err.message);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };

    fetchData();

    return () => {
      cancelled = true;
    };
  }, [url]);

  return { data, loading, error };
}

// Usage
function UserList() {
  const { data: users, loading, error } = useFetch('/api/users');

  if (loading) return <p>Loading...</p>;
  if (error) return <p>Error: {error}</p>;

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

#### Custom Hook - Local Storage

```jsx
function useLocalStorage(key, initialValue) {
  // Get initial value from localStorage or use initialValue
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  // Update localStorage when value changes
  const setValue = (value) => {
    try {
      const valueToStore =
        value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue];
}

// Usage
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [language, setLanguage] = useLocalStorage('language', 'en');

  return (
    <div>
      <select value={theme} onChange={(e) => setTheme(e.target.value)}>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>

      <select value={language} onChange={(e) => setLanguage(e.target.value)}>
        <option value="en">English</option>
        <option value="fr">French</option>
      </select>
    </div>
  );
}
```

---

## Context API

### Simple Explanation

Context is like a "global variable" for your component tree. Instead of passing props through every level (prop drilling), you can make data available to any component that needs it.

**Example:** User authentication - instead of passing the current user through 10 components, you create a UserContext and any component can access it directly.

### Technical Explanation

Context provides a way to share values between components without explicitly passing props through every level of the tree.

#### Basic Context

```jsx
import { createContext, useContext, useState } from 'react';

// 1. Create Context
const ThemeContext = createContext();

// 2. Create Provider Component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  };

  const value = {
    theme,
    toggleTheme,
  };

  return (
    <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>
  );
}

// 3. Create Custom Hook for easy access
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// 4. Use in components
function App() {
  return (
    <ThemeProvider>
      <Toolbar />
    </ThemeProvider>
  );
}

function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

function ThemedButton() {
  const { theme, toggleTheme } = useTheme();

  return (
    <button
      style={{
        background: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#000' : '#fff',
      }}
      onClick={toggleTheme}
    >
      Toggle Theme (Current: {theme})
    </button>
  );
}
```

#### Authentication Context

```jsx
const AuthContext = createContext();

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Check if user is logged in (e.g., from localStorage)
    const storedUser = localStorage.getItem('user');
    if (storedUser) {
      setUser(JSON.parse(storedUser));
    }
    setLoading(false);
  }, []);

  const login = async (email, password) => {
    try {
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ email, password }),
      });
      const userData = await response.json();
      setUser(userData);
      localStorage.setItem('user', JSON.stringify(userData));
      return { success: true };
    } catch (error) {
      return { success: false, error: error.message };
    }
  };

  const logout = () => {
    setUser(null);
    localStorage.removeItem('user');
  };

  const value = {
    user,
    login,
    logout,
    isAuthenticated: !!user,
    loading,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// Usage
function App() {
  return (
    <AuthProvider>
      <Navigation />
      <MainContent />
    </AuthProvider>
  );
}

function Navigation() {
  const { user, logout } = useAuth();

  return (
    <nav>
      {user ? (
        <>
          <span>Welcome, {user.name}</span>
          <button onClick={logout}>Logout</button>
        </>
      ) : (
        <Link to="/login">Login</Link>
      )}
    </nav>
  );
}

function ProtectedRoute({ children }) {
  const { isAuthenticated, loading } = useAuth();

  if (loading) return <p>Loading...</p>;
  if (!isAuthenticated) return <Navigate to="/login" />;
  return children;
}
```

#### Multiple Contexts

```jsx
function App() {
  return (
    <AuthProvider>
      <ThemeProvider>
        <LanguageProvider>
          <NotificationProvider>
            <Router>
              <Routes />
            </Router>
          </NotificationProvider>
        </LanguageProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}
```

#### Context with useReducer

```jsx
const CartContext = createContext();

const cartReducer = (state, action) => {
  switch (action.type) {
    case 'ADD_ITEM':
      const existingIndex = state.items.findIndex(
        (item) => item.id === action.payload.id,
      );

      if (existingIndex >= 0) {
        const newItems = [...state.items];
        newItems[existingIndex].quantity += action.payload.quantity;
        return { ...state, items: newItems };
      }

      return {
        ...state,
        items: [...state.items, action.payload],
      };

    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter((item) => item.id !== action.payload),
      };

    case 'UPDATE_QUANTITY':
      return {
        ...state,
        items: state.items.map((item) =>
          item.id === action.payload.id
            ? { ...item, quantity: action.payload.quantity }
            : item,
        ),
      };

    case 'CLEAR_CART':
      return { ...state, items: [] };

    default:
      return state;
  }
};

function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [] });

  const addItem = (item) => {
    dispatch({ type: 'ADD_ITEM', payload: item });
  };

  const removeItem = (id) => {
    dispatch({ type: 'REMOVE_ITEM', payload: id });
  };

  const updateQuantity = (id, quantity) => {
    dispatch({ type: 'UPDATE_QUANTITY', payload: { id, quantity } });
  };

  const clearCart = () => {
    dispatch({ type: 'CLEAR_CART' });
  };

  const total = state.items.reduce(
    (sum, item) => sum + item.price * item.quantity,
    0,
  );

  const value = {
    items: state.items,
    addItem,
    removeItem,
    updateQuantity,
    clearCart,
    total,
    itemCount: state.items.length,
  };

  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}

function useCart() {
  const context = useContext(CartContext);
  if (!context) {
    throw new Error('useCart must be used within CartProvider');
  }
  return context;
}

// Usage
function ProductCard({ product }) {
  const { addItem } = useCart();

  return (
    <div>
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => addItem({ ...product, quantity: 1 })}>
        Add to Cart
      </button>
    </div>
  );
}

function CartSummary() {
  const { items, total, itemCount } = useCart();

  return (
    <div>
      <p>Items: {itemCount}</p>
      <p>Total: ${total.toFixed(2)}</p>
    </div>
  );
}
```

---

## Performance Optimization

### Simple Explanation

React can be slow if you're not careful. Performance optimization is about making your app faster by preventing unnecessary work.

**Key idea:** Only re-render components when they actually need to update.

### Technical Explanation

#### React.memo

Prevents component re-render if props haven't changed.

```jsx
import { memo } from 'react';

// Without memo - re-renders every time parent renders
function ExpensiveComponent({ data, onAction }) {
  console.log('ExpensiveComponent rendered');
  return <div>{data.name}</div>;
}

// With memo - only re-renders when props change
const ExpensiveComponent = memo(function ExpensiveComponent({
  data,
  onAction,
}) {
  console.log('ExpensiveComponent rendered');
  return <div>{data.name}</div>;
});

// Custom comparison function
const ExpensiveComponent = memo(
  function ExpensiveComponent({ data, onAction }) {
    console.log('ExpensiveComponent rendered');
    return <div>{data.name}</div>;
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip render)
    // Return false if props are different (re-render)
    return prevProps.data.id === nextProps.data.id;
  },
);
```

#### useMemo for Expensive Calculations

```jsx
function DataTable({ data, filterText }) {
  // Without useMemo - filters on every render
  // const filteredData = data.filter(item =>
  //   item.name.toLowerCase().includes(filterText.toLowerCase())
  // );

  // With useMemo - only filters when data or filterText changes
  const filteredData = useMemo(() => {
    console.log('Filtering data...');
    return data.filter((item) =>
      item.name.toLowerCase().includes(filterText.toLowerCase()),
    );
  }, [data, filterText]);

  return (
    <ul>
      {filteredData.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

#### useCallback for Function Stability

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([]);

  // Without useCallback - new function on every render
  // const handleAddItem = (item) => {
  //   setItems(prev => [...prev, item]);
  // };

  // With useCallback - same function reference
  const handleAddItem = useCallback((item) => {
    setItems((prev) => [...prev, item]);
  }, []); // Empty deps = never recreated

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <ChildComponent onAddItem={handleAddItem} />
    </div>
  );
}

const ChildComponent = memo(({ onAddItem }) => {
  console.log('ChildComponent rendered');
  return <button onClick={() => onAddItem('new item')}>Add Item</button>;
});
```

#### Code Splitting & Lazy Loading

```jsx
import { lazy, Suspense } from 'react';

// Regular import - loaded immediately
// import HeavyComponent from './HeavyComponent';

// Lazy import - loaded only when needed
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  const [show, setShow] = useState(false);

  return (
    <div>
      <button onClick={() => setShow(!show)}>Toggle Heavy Component</button>

      {show && (
        <Suspense fallback={<div>Loading...</div>}>
          <HeavyComponent />
        </Suspense>
      )}
    </div>
  );
}
```

#### Route-based Code Splitting

```jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<div>Loading page...</div>}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

#### Virtualization for Long Lists

```jsx
import { FixedSizeList } from 'react-window';

// Without virtualization - renders all 10,000 items
function RegularList({ items }) {
  return (
    <ul>
      {items.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// With virtualization - only renders visible items
function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>{items[index].name}</div>
  );

  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}
```

#### Debouncing & Throttling

```jsx
import { useState, useCallback } from 'react';
import { debounce } from 'lodash';

function SearchComponent() {
  const [results, setResults] = useState([]);

  // Debounce - wait until user stops typing
  const handleSearch = useCallback(
    debounce(async (query) => {
      const response = await fetch(`/api/search?q=${query}`);
      const data = await response.json();
      setResults(data);
    }, 500), // Wait 500ms after last keystroke
    [],
  );

  return (
    <div>
      <input
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="Search..."
      />
      <ul>
        {results.map((item) => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

#### Avoid Inline Functions and Objects

```jsx
// BAD - Creates new function/object on every render
function Parent() {
  return (
    <Child
      onClick={() => console.log('clicked')}
      style={{ color: 'red' }}
      user={{ name: 'John' }}
    />
  );
}

// GOOD - Stable references
function Parent() {
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);

  const style = useMemo(() => ({ color: 'red' }), []);

  const user = useMemo(() => ({ name: 'John' }), []);

  return <Child onClick={handleClick} style={style} user={user} />;
}
```

---

## Common Patterns

### Simple Explanation

Design patterns are reusable solutions to common problems in React development.

### Technical Explanation

#### 1. Compound Components

```jsx
// Components that work together to form a cohesive UI
function Select({ children, value, onChange }) {
  return (
    <div className="select">
      {React.Children.map(children, (child) => {
        if (React.isValidElement(child)) {
          return React.cloneElement(child, {
            selected: child.props.value === value,
            onClick: () => onChange(child.props.value),
          });
        }
        return child;
      })}
    </div>
  );
}

function Option({ value, children, selected, onClick }) {
  return (
    <div className={`option ${selected ? 'selected' : ''}`} onClick={onClick}>
      {children}
    </div>
  );
}

// Usage
function App() {
  const [selected, setSelected] = useState('');

  return (
    <Select value={selected} onChange={setSelected}>
      <Option value="apple">Apple</Option>
      <Option value="banana">Banana</Option>
      <Option value="orange">Orange</Option>
    </Select>
  );
}
```

#### 2. Controlled vs Uncontrolled Components

```jsx
// Controlled - React controls the value
function ControlledInput() {
  const [value, setValue] = useState('');

  return <input value={value} onChange={(e) => setValue(e.target.value)} />;
}

// Uncontrolled - DOM controls the value
function UncontrolledInput() {
  const inputRef = useRef();

  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };

  return (
    <>
      <input ref={inputRef} defaultValue="initial" />
      <button onClick={handleSubmit}>Submit</button>
    </>
  );
}
```

#### 3. Higher-Order Components (HOC)

```jsx
// HOC - function that takes a component and returns a new component
function withLoading(Component) {
  return function WithLoadingComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <div>Loading...</div>;
    }
    return <Component {...props} />;
  };
}

// Usage
function UserList({ users }) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

const UserListWithLoading = withLoading(UserList);

function App() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  return <UserListWithLoading isLoading={loading} users={users} />;
}
```

#### 4. Render Props

```jsx
function MouseTracker({ render }) {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };

    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  return render(position);
}

// Usage
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <div>
          Mouse position: {x}, {y}
        </div>
      )}
    />
  );
}
```

#### 5. Container/Presentational Pattern

```jsx
// Presentational Component - only displays data
function UserListView({ users, onUserClick }) {
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id} onClick={() => onUserClick(user.id)}>
          {user.name}
        </li>
      ))}
    </ul>
  );
}

// Container Component - handles logic and state
function UserListContainer() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('/api/users')
      .then((res) => res.json())
      .then((data) => {
        setUsers(data);
        setLoading(false);
      });
  }, []);

  const handleUserClick = (id) => {
    console.log('User clicked:', id);
  };

  if (loading) return <div>Loading...</div>;

  return <UserListView users={users} onUserClick={handleUserClick} />;
}
```

---

## Best Practices

### 1. Component Organization

```jsx
// Good structure
function UserProfile({ user }) {
  // 1. Hooks at the top
  const [isEditing, setIsEditing] = useState(false);
  const [formData, setFormData] = useState(user);

  // 2. Effects
  useEffect(() => {
    setFormData(user);
  }, [user]);

  // 3. Event handlers
  const handleEdit = () => {
    setIsEditing(true);
  };

  const handleSave = async () => {
    await updateUser(formData);
    setIsEditing(false);
  };

  // 4. Render helpers
  const renderEditMode = () => (
    <form>
      <input
        value={formData.name}
        onChange={(e) => setFormData({ ...formData, name: e.target.value })}
      />
      <button onClick={handleSave}>Save</button>
    </form>
  );

  const renderViewMode = () => (
    <div>
      <h1>{user.name}</h1>
      <button onClick={handleEdit}>Edit</button>
    </div>
  );

  // 5. Main render
  return (
    <div className="user-profile">
      {isEditing ? renderEditMode() : renderViewMode()}
    </div>
  );
}
```

### 2. Key Props for Lists

```jsx
// BAD - using index as key
{
  items.map((item, index) => <li key={index}>{item.name}</li>);
}

// GOOD - using unique ID
{
  items.map((item) => <li key={item.id}>{item.name}</li>);
}

// If no ID available, generate stable keys
{
  items.map((item) => (
    <li key={`${item.name}-${item.timestamp}`}>{item.name}</li>
  ));
}
```

### 3. Conditional Rendering

```jsx
function Component({ isLoading, error, data }) {
  // 1. Early returns
  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!data) return <EmptyState />;

  // 2. Ternary for simple conditions
  return <div>{data.isPremium ? <PremiumBadge /> : <FreeBadge />}</div>;

  // 3. && for showing/hiding
  return (
    <div>
      {data.notifications.length > 0 && (
        <NotificationBadge count={data.notifications.length} />
      )}
    </div>
  );
}
```

### 4. Props Destructuring

```jsx
// Good - destructure in parameter
function UserCard({ name, email, avatar }) {
  return (
    <div>
      <img src={avatar} alt={name} />
      <h2>{name}</h2>
      <p>{email}</p>
    </div>
  );
}

// With default values
function Button({ text = 'Click', variant = 'primary', onClick }) {
  return (
    <button className={`btn-${variant}`} onClick={onClick}>
      {text}
    </button>
  );
}
```

### 5. Error Boundaries

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h1>Something went wrong</h1>
          <p>{this.state.error?.message}</p>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
function App() {
  return (
    <ErrorBoundary>
      <MyComponent />
    </ErrorBoundary>
  );
}
```

---

## Common Interview Questions

### 1. What is the Virtual DOM?

**Simple:** A JavaScript copy of the real DOM that React uses to figure out what changed, so it can update only those parts.

**Technical:** The Virtual DOM is a lightweight representation of the real DOM kept in memory. React uses a diffing algorithm to compare the new Virtual DOM with the previous one, calculates the minimal set of changes needed, and batches updates to the real DOM for optimal performance.

### 2. What's the difference between state and props?

**Simple:**

- Props: Data passed FROM parent TO child (like function arguments)
- State: Data managed INSIDE a component (like variables)

**Technical:**

- Props are immutable and flow unidirectionally from parent to child
- State is mutable and managed within the component using useState/useReducer
- Props trigger re-renders when changed by parent
- State changes trigger re-renders of the component and its children

### 3. When would you use useCallback vs useMemo?

**Simple:**

- useCallback: When you want to keep the same function reference
- useMemo: When you want to avoid recalculating an expensive value

**Technical:**

- useCallback returns a memoized callback function
- useMemo returns a memoized value
- useCallback(fn, deps) is equivalent to useMemo(() => fn, deps)
- Use useCallback when passing callbacks to optimized child components to prevent unnecessary renders
- Use useMemo for expensive computations

### 4. How do you handle forms in React?

**Controlled components:** React controls the form state

```jsx
const [value, setValue] = useState('');
<input value={value} onChange={(e) => setValue(e.target.value)} />;
```

**Uncontrolled components:** DOM controls the form state

```jsx
const inputRef = useRef();
<input ref={inputRef} />;
// Access with inputRef.current.value
```

### 5. What are React keys and why are they important?

Keys help React identify which items have changed, are added, or are removed. They should be stable, unique, and consistent across renders. Never use array index as key if the list can be reordered.

---

This comprehensive guide covers React fundamentals through advanced concepts. Good luck with your interviews, Ale!

```

## Table of Contents

### I - Describing the UI

Learn how to build the visual structure of your React application.

- [(a) Your First Component](<./I%20-%20Describing%20the%20UI/(a)%20Your%20First%20Component.md>) - Understanding what components are and how to create your first one
- [(b) Importing and Exporting Components](<./I%20-%20Describing%20the%20UI/(b)%20Importing%20and%20Exporting%20Components.md>) - Organizing components across files
- [(c) Writing Markup with JSX](<./I%20-%20Describing%20the%20UI/(c)%20Writing%20Markup%20with%20JSX.md>) - The syntax for writing React components
- [(d) JavaScript in JSX with Curly Braces](<./I%20-%20Describing%20the%20UI/(d)%20JavaScript%20in%20JSX%20with%20Curly%20Braces.md>) - Using JavaScript expressions in JSX
- [(e) Passing Props to a Component](<./I%20-%20Describing%20the%20UI/(e)%20Passing%20Props%20to%20a%20Component.md>) - How to pass data from parent to child components
- [(f) Conditional Rendering](<./I%20-%20Describing%20the%20UI/(f)%20Conditional%20Rendering.md>) - Showing different UI based on conditions
- [(g) Rendering Lists](<./I%20-%20Describing%20the%20UI/(g)%20Rendering%20Lists.md>) - Displaying arrays of data in your UI
- [(h) Keeping Components Pure](<./I%20-%20Describing%20the%20UI/(h)%20Keeping%20Components%20Pure.md>) - Writing predictable, testable components
- [(i) Understanding Your UI as a Tree](<./I%20-%20Describing%20the%20UI/(i)%20Understanding%20Your%20UI%20as%20a%20Tree.md>) - The component tree and how React renders

### II - Interactivity

Understand how to make your components interactive and manage component state.

- [(a) Responding to Events](<./II%20-%20Interactivity/(a)%20Responding%20to%20Events.md>) - Handling user interactions like clicks and input
- [(b) State: A Component's Memory](<./II%20-%20Interactivity/(b)%20State%20A%20Component's%20Memory.md>) - Using state to remember information
- [(c) Render and Commit](<./II%20-%20Interactivity/(c)%20Render%20and%20Commit.md>) - How React updates the UI in three steps
- [(d) State as a Snapshot](<./II%20-%20Interactivity/(d)%20State%20as%20a%20Snapshot.md>) - Understanding how state works in React
- [(e) Queueing a Series of State Updates](<./II%20-%20Interactivity/(e)%20Queueing%20a%20Series%20of%20State%20Updates.md>) - Batching state updates correctly
- [(f) Updating Objects in State](<./II%20-%20Interactivity/(f)%20Updating%20Objects%20in%20State.md>) - Working with object state immutably
- [(g) Updating Arrays in State](<./II%20-%20Interactivity/(g)%20Updating%20Arrays%20in%20State.md>) - Working with array state immutably

### III - Managing States

Advanced patterns for managing state across your application.

- [(a) Reacting to Input with State](<./III%20-%20Managing%20states/(a)%20Reacting%20to%20Input%20with%20State.md>) - Building controlled components and forms
- [(b) Choosing the State Structure](<./III%20-%20Managing%20states/(b)%20Choosing%20the%20State%20Structure.md>) - Best practices for organizing state
- [(c) Sharing State Between Components](<./III%20-%20Managing%20states/(c)%20Sharing%20State%20Between%20Components.md>) - Lifting state up to share data
- [(d) Preserving and Resetting State](<./III%20-%20Managing%20states/(d)%20Preserving%20and%20Resetting%20State.md>) - How React preserves or resets component state
- [(e) Extracting State Logic into a Reducer](<./III%20-%20Managing%20states/(e)%20Extracting%20State%20Logic%20into%20a%20Reducer.md>) - Using `useReducer` for complex state logic
- [(f) Passing Data Deeply with Context](<./III%20-%20Managing%20states/(f)%20Passing%20Data%20Deeply%20with%20Context.md>) - Avoiding prop drilling with Context API
- [(g) Scaling Up with Reducer and Context](<./III%20-%20Managing%20states/(g)%20Scaling%20Up%20with%20Reducer%20and%20Context.md>) - Combining reducers and context for complex apps

### IV - Escape Hatches

Advanced features for when you need to step outside React's normal patterns.

- [(a) Referencing Values with Refs](<./IV%20-%20Escape%20Hatches/(a)%20Referencing%20Values%20with%20Refs.md>) - Storing values that don't trigger re-renders
- [(b) Manipulating the DOM with Refs](<./IV%20-%20Escape%20Hatches/(b)%20Manipulating%20the%20DOM%20with%20Refs.md>) - Accessing DOM nodes directly when needed
- [(c) Synchronizing with Effects](<./IV%20-%20Escape%20Hatches/(c)%20Synchronizing%20with%20Effects.md>) - Using `useEffect` to sync with external systems
- [(d) You Might Not Need an Effect](<./IV%20-%20Escape%20Hatches/(d)%20You%20Might%20Not%20Need%20an%20Effect.md>) - When to avoid Effects and use alternatives
- [(e) Lifecycle of Reactive Effects](<./IV%20-%20Escape%20Hatches/(e)%20Lifecycle%20of%20Reactive%20Effects.md>) - Understanding how Effects synchronize and clean up
- [(f) Separating Events from Effects](<./IV%20-%20Escape%20Hatches/(f)%20Separating%20Events%20from%20Effects.md>) - Using `useEffectEvent` to read values without reacting
- [(g) Removing Effect Dependencies](<./IV%20-%20Escape%20Hatches/(g)%20Removing%20Effect%20Dependencies.md>) - Strategies for optimizing Effect dependencies
- [(h) Reusing Logic with Custom Hooks](<./IV%20-%20Escape%20Hatches/(h)%20Reusing%20Logic%20with%20Custom%20Hooks.md>) - Extracting and sharing component logic

---

## Summary

This documentation collection covers React fundamentals from components and JSX to advanced patterns like reducers, context, and custom hooks. Each section includes:

- **Learning objectives** - What you'll understand after reading
- **Core concepts** - Detailed explanations with examples
- **Code examples** - Practical, real-world implementations
- **Pitfalls** - Common mistakes and how to avoid them (with ⚠️ warnings)
- **Interview questions** - Questions to test your understanding

### Key Learning Path

1. **Start with Describing the UI** - Learn the basics of components and JSX
2. **Move to Interactivity** - Understand events, state, and how React updates
3. **Master Managing State** - Handle complex state patterns and data flow
4. **Explore Escape Hatches** - Use advanced features when needed

### Prerequisites

- Basic JavaScript knowledge (ES6+)
- Understanding of HTML/CSS
- Familiarity with functional programming concepts (helpful but not required)

---

_Based on the official React documentation at [react.dev](https://react.dev)_
```

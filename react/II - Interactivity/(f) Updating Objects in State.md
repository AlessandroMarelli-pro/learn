# Updating Objects in State

## Overview
State can hold any JavaScript value, including objects. But you shouldn't change objects that you hold in React state directly. Instead, when you want to update an object, create a new one (or make a copy of an existing one), and then set state to use that copy.

## What You Will Learn
- Correctly modify objects stored in React state
- Update nested objects without mutation
- Understand immutability concepts and maintain them
- Use Immer library to simplify object updates

---

## What's a Mutation?

**Mutation** means changing the contents of an object directly:

```jsx
const position = { x: 0, y: 0 };
position.x = 5; // ❌ This is mutation
```

While technically possible in JavaScript, you should treat React state objects **as if they were immutable**—like numbers, booleans, and strings.

---

## Treat State as Read-Only

When you store objects in state, **never change them directly**. React won't detect the change and won't re-render.

### ❌ Wrong: Mutating State

```jsx
function MovingDot() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  return (
    <div
      onPointerMove={e => {
        position.x = e.clientX; // ❌ Don't mutate state!
        position.y = e.clientY; // ❌ Don't mutate state!
      }}
    >
      <div style={{ left: position.x, top: position.y }} />
    </div>
  );
}
```

**Problem:** React doesn't know the object changed. No re-render happens. The UI won't update.

### ✅ Correct: Creating New Objects

```jsx
function MovingDot() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  return (
    <div
      onPointerMove={e => {
        setPosition({
          x: e.clientX,
          y: e.clientY
        }); // ✅ Create new object and set state
      }}
    >
      <div style={{ left: position.x, top: position.y }} />
    </div>
  );
}
```

**What happens:**
1. Create a brand new object with updated values
2. Pass it to `setPosition`
3. React sees a new object and triggers a re-render
4. UI updates with new position

> **⚠️ CRITICAL RULE**
>
> **Treat state as read-only. Never mutate objects in state.**
>
> Instead of mutating:
> ```jsx
> obj.x = 10; // ❌ Don't mutate
> setState(obj);
> ```
>
> Create new objects:
> ```jsx
> setState({ ...obj, x: 10 }); // ✅ Create new object
> ```

---

## Using the Spread Syntax

Manually creating new objects for every property is tedious. Use the **spread syntax** (`...`) to copy existing properties:

```jsx
const [person, setPerson] = useState({
  firstName: 'Barbara',
  lastName: 'Hepworth',
  email: 'bhepworth@sculpture.com'
});

function handleFirstNameChange(e) {
  setPerson({
    ...person,                    // Copy all existing properties
    firstName: e.target.value     // Override this one
  });
}
```

**How it works:**
1. `...person` spreads all existing properties
2. `firstName: e.target.value` overwrites just the firstName
3. Result: New object with updated firstName, other properties unchanged

---

## Updating a Single Field

You can update just one property while keeping others intact:

```jsx
function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  return (
    <>
      <label>
        First name:
        <input
          value={person.firstName}
          onChange={e => setPerson({
            ...person,
            firstName: e.target.value
          })}
        />
      </label>
      <label>
        Email:
        <input
          value={person.email}
          onChange={e => setPerson({
            ...person,
            email: e.target.value
          })}
        />
      </label>
    </>
  );
}
```

Each input updates only its field, leaving others unchanged.

---

## Updating Nested Objects

Objects can contain other objects (nested structures):

```jsx
const [person, setPerson] = useState({
  name: 'Niki de Saint Phalle',
  artwork: {
    title: 'Blue Nana',
    city: 'Hamburg',
    image: 'https://example.com/image.jpg'
  }
});
```

### ❌ Wrong: Shallow Spread for Nested Objects

```jsx
setPerson({
  ...person,
  artwork.city: 'New Delhi' // ❌ Syntax error!
});

// Or:
setPerson({
  ...person,
  artwork: {
    city: 'New Delhi' // ❌ Loses title and image!
  }
});
```

### ✅ Correct: Spread at Each Level

```jsx
setPerson({
  ...person,                    // Copy person
  artwork: {                    // Replace artwork
    ...person.artwork,          // Copy artwork properties
    city: 'New Delhi'           // Override city
  }
});
```

**Breakdown:**
1. `...person` copies top-level properties
2. `artwork: { ... }` creates new artwork object
3. `...person.artwork` copies existing artwork properties
4. `city: 'New Delhi'` overrides just the city

> **⚠️ PITFALL**
>
> **Spread syntax is shallow—it only copies one level deep.**
>
> For nested objects, you must spread at each level:
> ```jsx
> setPerson({
>   ...person,              // Level 1
>   artwork: {
>     ...person.artwork,    // Level 2
>     city: 'New Delhi'
>   }
> });
> ```

---

## Deeply Nested Objects

With deeply nested structures, spreading becomes verbose:

```jsx
const [person, setPerson] = useState({
  name: 'Alex',
  address: {
    street: '123 Main St',
    city: {
      name: 'New York',
      state: 'NY'
    }
  }
});

// Update city name (3 levels deep!)
setPerson({
  ...person,
  address: {
    ...person.address,
    city: {
      ...person.address.city,
      name: 'Boston'
    }
  }
});
```

**This gets unwieldy quickly!**

---

## Using Immer for Simpler Updates

**Immer** is a library that lets you write concise update logic using mutation-like syntax, while maintaining immutability behind the scenes.

### Installation

```bash
npm install use-immer
```

### Basic Usage

```jsx
import { useImmer } from 'use-immer';

function Form() {
  const [person, updatePerson] = useImmer({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg'
    }
  });

  function handleCityChange(e) {
    updatePerson(draft => {
      draft.artwork.city = e.target.value; // ✅ "Mutate" the draft!
    });
  }

  return (
    <input value={person.artwork.city} onChange={handleCityChange} />
  );
}
```

**How Immer works:**
1. You receive a `draft` object (a special Proxy)
2. You "mutate" the draft freely
3. Immer records all changes
4. Immer produces a new immutable object with those changes

**Benefits:**
- Concise syntax for nested updates
- Reads like normal JavaScript
- Maintains immutability automatically

### Example: Multiple Nested Updates

```jsx
updatePerson(draft => {
  draft.name = 'New Name';
  draft.artwork.title = 'New Title';
  draft.artwork.city = 'New City';
  // All changes applied to a new immutable object
});
```

Much cleaner than multiple spreads!

---

## Why Immutability Matters

### 1. Debugging
When you log state, you get a snapshot at that moment:

```jsx
console.log(person); // { name: 'Alice', age: 30 }
setPerson({ ...person, age: 31 });
console.log(person); // Still { name: 'Alice', age: 30 } (snapshot preserved)
```

With mutation, logs get overwritten, making debugging harder.

### 2. Optimizations
React can quickly check if state changed by comparing references:

```jsx
if (prevState === newState) {
  // No change, skip re-render
}
```

With mutation, the reference stays the same, so React can't detect changes.

### 3. New Features
Future React features depend on state being treated as snapshots. Mutation breaks this assumption.

### 4. Feature Implementation
Features like undo/redo, time-travel debugging, and state history are easier with immutable state.

### 5. Simplicity
React doesn't need special object wrappers or proxy systems—plain JavaScript objects work perfectly.

---

## Local Mutation is OK

You **can** mutate objects you've just created in the same scope:

```jsx
const nextPosition = {};
nextPosition.x = e.clientX; // ✅ OK: Just created this object
nextPosition.y = e.clientY;
setPosition(nextPosition);
```

**Why this is fine:**
- The object isn't in state yet
- No other code references it
- It's a "local mutation" before it enters state

**Equivalent to:**
```jsx
setPosition({
  x: e.clientX,
  y: e.clientY
});
```

**The rule:** Only avoid mutating objects **already in state**.

---

## Practical Examples

### Example 1: Form with Multiple Fields
```jsx
function UserProfile() {
  const [user, setUser] = useState({
    username: '',
    email: '',
    bio: ''
  });

  return (
    <>
      <input
        value={user.username}
        onChange={e => setUser({ ...user, username: e.target.value })}
      />
      <input
        value={user.email}
        onChange={e => setUser({ ...user, email: e.target.value })}
      />
      <textarea
        value={user.bio}
        onChange={e => setUser({ ...user, bio: e.target.value })}
      />
    </>
  );
}
```

### Example 2: Nested Settings with Immer
```jsx
import { useImmer } from 'use-immer';

function Settings() {
  const [settings, updateSettings] = useImmer({
    theme: {
      mode: 'dark',
      colors: {
        primary: '#007bff',
        secondary: '#6c757d'
      }
    },
    notifications: {
      email: true,
      push: false
    }
  });

  return (
    <>
      <button onClick={() => updateSettings(draft => {
        draft.theme.mode = draft.theme.mode === 'dark' ? 'light' : 'dark';
      })}>
        Toggle Theme
      </button>
      <button onClick={() => updateSettings(draft => {
        draft.notifications.push = !draft.notifications.push;
      })}>
        Toggle Push Notifications
      </button>
    </>
  );
}
```

---

## Interview Questions to Consider

1. **Q: Why can't you mutate objects in React state?**
   - A: React won't detect the change and won't trigger a re-render. React only re-renders when you pass a new object to the state setter.

2. **Q: What's the correct way to update an object in state?**
   - A: Create a new object (using spread syntax or other methods) and pass it to the state setter: `setState({ ...obj, property: newValue })`.

3. **Q: What does the spread operator do?**
   - A: It copies all properties from one object to another: `{ ...obj }` creates a shallow copy of `obj`.

4. **Q: How do you update a nested object in state?**
   - A: Spread at each level: `setState({ ...state, nested: { ...state.nested, property: value } })`.

5. **Q: What's the limitation of spread syntax?**
   - A: It's shallow—it only copies one level deep. Nested objects need multiple spreads.

6. **Q: What is Immer and why would you use it?**
   - A: Immer is a library that lets you write concise updates using mutation-like syntax while maintaining immutability. It's helpful for deeply nested state.

7. **Q: Can you ever mutate objects in React?**
   - A: Yes, you can mutate objects you've just created (local mutation) before putting them in state. You just can't mutate objects already in state.

8. **Q: Why does immutability help with debugging?**
   - A: Console logs preserve state snapshots. With mutation, all references get updated, making it harder to see historical values.

9. **Q: How does immutability enable optimizations?**
   - A: React can quickly compare object references (`prevState === newState`) to determine if state changed, rather than deeply comparing all properties.

---

## Key Takeaways

- **Never mutate objects in state directly**—React won't detect changes
- **Create new objects** instead: `setState({ ...obj, property: value })`
- Use **spread syntax** (`...`) to copy objects while updating specific properties
- **Spread is shallow**—for nested objects, spread at each level
- For deeply nested updates, consider using **Immer** library
- Immer lets you write mutation-like syntax while maintaining immutability
- **Local mutation is OK** for newly created objects not yet in state
- Immutability enables debugging, optimizations, and future React features
- Treat state objects like immutable values (numbers, strings, booleans)
- Understanding object references is crucial for proper state management

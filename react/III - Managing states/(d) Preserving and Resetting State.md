# Preserving and Resetting State

## You Will Learn

- How React decides whether to preserve or reset a component's state
- How to force React to reset a component's state using the `key` prop
- How component position in the render tree affects state preservation
- The relationship between keys, component types, and state preservation

---

## State is Tied to a Position in the Render Tree

React maintains state based on a component's **position in the UI tree**, not its name or function identity. As long as a component remains at the same position between renders, React preserves its state. If a component is removed from the tree or replaced by a different component at the same position, React resets its state.

**Key Principle**: React associates state with the position of the component in the render tree, not with the component itself.

### Example: Independent Counters

Each component at a different position maintains its own independent state:

```jsx
import { useState } from 'react';

export default function App() {
  return (
    <div>
      <Counter />
      <Counter />
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  return (
    <div>
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

Even though both components use the same `<Counter />` component, each has its own state because they occupy different positions in the render tree.

### Example: Conditional Rendering Resets State

When a component is conditionally rendered and then unmounted, its state is reset when it remounts:

```jsx
import { useState } from 'react';

export default function App() {
  const [showCounter, setShowCounter] = useState(true);

  return (
    <div>
      {showCounter && <Counter />}
      <button onClick={() => setShowCounter(!showCounter)}>
        Toggle Counter
      </button>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  return (
    <div>
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

When you toggle the counter off and back on, the `score` resets to 0 because React destroys and recreates the component.

---

## How to Force React to Reset a Component's State

Sometimes you want React to reset a component's state even when it's rendered at the same position. You can achieve this by giving it a different `key` prop.

**The `key` prop tells React**: "This is a different component instance, even though it looks the same." When the `key` changes, React resets the component's state.

### Example: Resetting a Chat Form When Switching Recipients

In a chat application, you want the input field to reset when switching between recipients:

```jsx
import { useState } from 'react';

export default function Messenger() {
  const [recipient, setRecipient] = useState(contacts[0]);

  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={recipient}
        onSelect={(contact) => setRecipient(contact)}
      />
      <Chat key={recipient.email} contact={recipient} />
    </div>
  );
}

const contacts = [
  { name: 'Taylor', email: 'taylor@mail.com' },
  { name: 'Alice', email: 'alice@mail.com' },
  { name: 'Bob', email: 'bob@mail.com' },
];
```

By setting `key={recipient.email}`, React treats each recipient's chat as a new component instance. When you switch recipients, the `Chat` component's state (including the input field value) is reset.

**Without the `key` prop**, React would preserve the input field's value when switching recipients because the component stays at the same position.

---

## How Keys and Types Affect Whether the State Is Preserved

React uses **both** the component type and the `key` prop to decide whether to preserve or reset state:

| Situation                          | State Behavior                        |
| ---------------------------------- | ------------------------------------- |
| Same component type, same key      | ✅ State is preserved                 |
| Same component type, different key | ❌ State is reset                     |
| Different component type           | ❌ State is reset (regardless of key) |

### Example: Different Component Types Reset State

Switching between different component types at the same position always resets state:

```jsx
import { useState } from 'react';

export default function App() {
  const [isPaused, setIsPaused] = useState(false);

  return (
    <div>
      {isPaused ? <p>See you later!</p> : <Counter />}
      <label>
        <input
          type="checkbox"
          checked={isPaused}
          onChange={(e) => setIsPaused(e.target.checked)}
        />
        Take a break
      </label>
    </div>
  );
}

function Counter() {
  const [score, setScore] = useState(0);
  return (
    <div>
      <h1>{score}</h1>
      <button onClick={() => setScore(score + 1)}>Add one</button>
    </div>
  );
}
```

When you toggle between `Counter` and `<p>`, React treats them as different component types. The `Counter`'s state resets every time it's re-rendered because React sees a different type at that position.

---

## Pitfalls

> **⚠️ PITFALL: Never Define Components Inside Other Components**
>
> Defining a component inside another component can cause React to lose state on every re-render. This happens because each render creates a new component function, and React treats it as a completely different component type.
>
> ```jsx
> // ❌ NEVER DO THIS
> export default function MyComponent() {
>   const [counter, setCounter] = useState(0);
>
>   function MyTextField() {
>     // Don't define components inside other components!
>     const [text, setText] = useState('');
>     return <input value={text} onChange={(e) => setText(e.target.value)} />;
>   }
>
>   return (
>     <>
>       <MyTextField />
>       <button onClick={() => setCounter(counter + 1)}>
>         Clicked {counter} times
>       </button>
>     </>
>   );
> }
> ```
>
> **Problem**: Every time `MyComponent` re-renders, a new `MyTextField` function is created. React sees this as a new component type and resets its state, causing the input field to lose its value whenever the counter updates.
>
> ```jsx
> // ✅ DO THIS INSTEAD
> function MyTextField() {
>   // Define at the top level
>   const [text, setText] = useState('');
>   return <input value={text} onChange={(e) => setText(e.target.value)} />;
> }
>
> export default function MyComponent() {
>   const [counter, setCounter] = useState(0);
>
>   return (
>     <>
>       <MyTextField />
>       <button onClick={() => setCounter(counter + 1)}>
>         Clicked {counter} times
>       </button>
>     </>
>   );
> }
> ```
>
> **Solution**: Always define components at the top level, outside of other components. This ensures React recognizes them as the same component type across renders and preserves their state.

---

## Interview Questions to Consider

### Conceptual Understanding

1. **How does React determine whether to preserve or reset a component's state?**

   - React associates state with the component's position in the render tree. As long as a component remains at the same position between renders, React preserves its state. If the position changes or the component is replaced, React resets the state.

2. **What role does the `key` prop play in state preservation?**

   - The `key` prop helps React identify when a component instance is "new" even if it's at the same position. When the `key` changes, React treats it as a different component and resets its state. This is useful when you want to reset state when switching between different items (like chat recipients).

3. **What determines state preservation: component type, key, or position?**
   - All three factors matter:
     - React uses both component type AND key to identify component instances
     - The position in the render tree determines where the component is rendered
     - Same type + same key + same position = state preserved
     - Different type OR different key = state reset

### Practical Application

4. **Why does this code reset the counter's state when toggling?**

   ```jsx
   {
     showCounter && <Counter />;
   }
   <button onClick={() => setShowCounter(!showCounter)}>Toggle</button>;
   ```

   - When `showCounter` is `false`, the `Counter` component is removed from the render tree. When it's added back, React treats it as a new component instance and resets its state.

5. **How would you fix a chat application where the input field doesn't reset when switching recipients?**

   ```jsx
   <Chat contact={recipient} />
   ```

   - Add a `key` prop based on the recipient's unique identifier:

   ```jsx
   <Chat key={recipient.id} contact={recipient} />
   ```

   This forces React to reset the `Chat` component's state whenever the recipient changes.

6. **What's wrong with this code?**
   ```jsx
   function App() {
     function MyInput() {
       const [value, setValue] = useState('');
       return (
         <input value={value} onChange={(e) => setValue(e.target.value)} />
       );
     }
     return <MyInput />;
   }
   ```
   - `MyInput` is defined inside `App`, so React creates a new component function on every render. This causes React to reset the input's state because it sees a "new" component type each time.

### Advanced Thinking

7. **If you have two `<Counter />` components at different positions, do they share state?**

   - No, each component maintains its own independent state because they occupy different positions in the render tree. React preserves state per position, not per component function.

8. **What happens if you render the same component type with different keys at different positions?**

   - Each instance maintains its own state because they're at different positions. The key doesn't matter in this case since they're not at the same position. Keys are primarily important when components are at the same position but represent different logical items.

9. **Why does switching between `<Counter />` and `<p>See you later!</p>` always reset the counter's state?**

   - React sees different component types (`Counter` vs `p` element) at the same position. Different component types always result in state reset, regardless of keys.

10. **Can you have the same component preserve state when moving to a different position?**
    - No. React associates state with position, not with the component function itself. Moving a component to a different position in the tree will cause React to reset its state because it's a different position.

---

## Summary

Understanding how React preserves and resets state is crucial for building predictable applications:

- **State is tied to position**: React preserves state based on a component's position in the render tree
- **Keys reset state**: Using different `key` props forces React to reset state, useful when switching between different items
- **Type matters**: Different component types always reset state, even at the same position
- **Never nest components**: Defining components inside other components causes state loss on every re-render

By leveraging component positions, keys, and types appropriately, you gain fine-grained control over when React preserves or resets component state.

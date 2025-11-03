# Manipulating the DOM with Refs

## You Will Learn

- How to access a DOM node managed by React with the `ref` attribute
- How the `ref` JSX attribute relates to the `useRef` Hook
- How to access another component's DOM node using `forwardRef`
- When it's safe to modify the DOM managed by React

---

## Why Access the DOM?

React automatically updates the DOM to match your render output, so you usually don't need to manipulate it directly. However, sometimes you need access to DOM elements to:

- **Focus an input**: Programmatically focus a text input
- **Scroll to an element**: Scroll to a specific section of the page
- **Measure size/position**: Get dimensions or coordinates of elements
- **Use browser APIs**: Call DOM APIs that React doesn't expose

For these cases, you need a **ref** to the DOM node.

---

## Getting a Ref to a DOM Node

### Step 1: Import `useRef`

```jsx
import { useRef } from 'react';
```

### Step 2: Declare the Ref

Create a ref inside your component:

```jsx
const myRef = useRef(null);
```

**Initial value**: The `null` initial value is important. `ref.current` will be `null` initially and only gets set after React creates the DOM node.

### Step 3: Attach the Ref to a JSX Element

Pass the ref as the `ref` attribute to the JSX tag:

```jsx
<div ref={myRef}>
  {/* Content */}
</div>
```

**How it works:**

1. `useRef` returns an object: `{ current: null }`
2. When React creates the DOM node for `<div>`, it puts a reference to that node into `myRef.current`
3. You can then access the DOM node from event handlers or effects

---

## Example: Focusing a Text Input

Here's a complete example of using a ref to focus an input:

```jsx
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

**Step-by-step:**

1. **Declare `inputRef`** with `useRef(null)`
2. **Pass it as `<input ref={inputRef}>`** - React sets `inputRef.current` to the DOM node
3. **In `handleClick`**, read `inputRef.current` and call `focus()` on it
4. **Attach `handleClick`** to the button's `onClick`

**Important**: Access `ref.current` in event handlers or effects, not during rendering.

---

## Example: Scrolling to an Element

You can use refs to scroll to specific elements:

```jsx
import { useRef } from 'react';

export default function Page() {
  const sectionRef = useRef(null);

  function handleClick() {
    sectionRef.current.scrollIntoView({ behavior: 'smooth' });
  }

  return (
    <>
      <button onClick={handleClick}>
        Scroll to section
      </button>
      <div style={{ height: '150vh' }}>
        {/* Content with enough height to enable scrolling */}
      </div>
      <div ref={sectionRef}>
        <h2>Target Section</h2>
        <p>This section will scroll into view</p>
      </div>
    </>
  );
}
```

**Other scrolling methods:**

- `scrollIntoView({ behavior: 'smooth', block: 'center' })`
- `scrollTo({ top: 0, behavior: 'smooth' })`
- `scrollTop = 0`

---

## Accessing Another Component's DOM Node

By default, React components don't expose their DOM nodes. To access a DOM node inside a child component, use `forwardRef` to forward the ref from the parent to the child's DOM element.

### Using `forwardRef`

```jsx
import { forwardRef, useRef } from 'react';

// Child component forwards the ref
const MyInput = forwardRef(function MyInput(props, ref) {
  return <input {...props} ref={ref} />;
});

// Parent component uses the ref
export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

**How `forwardRef` works:**

1. `forwardRef` wraps your component function
2. Your component receives `props` and `ref` as the second parameter
3. Pass `ref` to the DOM element you want to expose
4. Parent can now access the child's DOM node through the ref

### Why `forwardRef` is Needed

Without `forwardRef`, you can't pass a ref to a custom component:

```jsx
// ❌ This doesn't work - ref is not a regular prop
function MyInput(props) {
  return <input {...props} ref={props.ref} />;  // Won't work
}

// ✅ This works with forwardRef
const MyInput = forwardRef(function MyInput(props, ref) {
  return <input {...props} ref={ref} />;
});
```

**Note**: `ref` is a special prop handled by React, not a regular prop like `className` or `onClick`.

---

## Example: Custom Input Component with Ref

Here's a complete example showing how to create a reusable input component that exposes its DOM node:

```jsx
import { forwardRef, useRef } from 'react';

// Custom input component that forwards refs
const MyInput = forwardRef(function MyInput({ label, ...props }, ref) {
  return (
    <label>
      {label}
      <input {...props} ref={ref} />
    </label>
  );
});

export default function Form() {
  const firstNameRef = useRef(null);
  const lastNameRef = useRef(null);

  function handleFirstNameFocus() {
    firstNameRef.current.focus();
  }

  function handleLastNameFocus() {
    lastNameRef.current.focus();
  }

  return (
    <form>
      <MyInput
        label="First name:"
        ref={firstNameRef}
      />
      <MyInput
        label="Last name:"
        ref={lastNameRef}
      />
      <button type="button" onClick={handleFirstNameFocus}>
        Focus first name
      </button>
      <button type="button" onClick={handleLastNameFocus}>
        Focus last name
      </button>
    </form>
  );
}
```

**Benefits:**

- Reusable input component
- Parent can control focus
- Clean separation of concerns

---

## Common Use Cases for DOM Refs

### 1. Focusing Elements

```jsx
const inputRef = useRef(null);
inputRef.current?.focus();
```

### 2. Scrolling

```jsx
const elementRef = useRef(null);
elementRef.current?.scrollIntoView({ behavior: 'smooth' });
```

### 3. Measuring Elements

```jsx
const divRef = useRef(null);
const width = divRef.current?.offsetWidth;
const height = divRef.current?.offsetHeight;
```

### 4. Calling Browser APIs

```jsx
const videoRef = useRef(null);
videoRef.current?.play();
videoRef.current?.pause();
```

### 5. Integration with Third-Party Libraries

```jsx
const canvasRef = useRef(null);

useEffect(() => {
  if (canvasRef.current) {
    const chart = new Chart(canvasRef.current, config);
    return () => chart.destroy();
  }
}, []);
```

---

## When It's Safe to Modify the DOM

### ✅ Safe: Non-Destructive Actions

These actions don't conflict with React's DOM management:

- **Focusing**: `element.focus()`, `element.blur()`
- **Scrolling**: `element.scrollIntoView()`, `window.scrollTo()`
- **Reading**: `element.offsetWidth`, `element.getBoundingClientRect()`
- **Media controls**: `video.play()`, `audio.pause()`
- **Canvas operations**: Drawing on canvas elements

```jsx
function MyComponent() {
  const inputRef = useRef(null);
  
  useEffect(() => {
    // Safe: Non-destructive action
    inputRef.current?.focus();
  }, []);
  
  return <input ref={inputRef} />;
}
```

### ✅ Safe: Modifying Elements React Doesn't Touch

If React has no reason to update a part of the DOM, you can safely modify it:

```jsx
function MyComponent() {
  const containerRef = useRef(null);
  
  useEffect(() => {
    // Safe: React never updates children of this div (it's always empty in JSX)
    if (containerRef.current) {
      const thirdPartyWidget = document.createElement('div');
      containerRef.current.appendChild(thirdPartyWidget);
      
      return () => {
        containerRef.current?.removeChild(thirdPartyWidget);
      };
    }
  }, []);
  
  return <div ref={containerRef} />;  // Always empty in JSX
}
```

### ❌ Dangerous: Destructive Actions on React-Managed Elements

These actions can conflict with React and cause crashes or inconsistencies:

- **Removing elements**: `element.remove()`
- **Adding/removing children**: `appendChild()`, `removeChild()` on React-managed elements
- **Changing text content**: `textContent = '...'` on elements React manages
- **Modifying attributes React controls**: Changing `className`, `style`, etc. that React manages

---

## Pitfalls

> **⚠️ PITFALL: Don't Modify DOM Elements That React Manages**
>
> Manually modifying DOM elements that React controls can cause crashes, inconsistent UI, and bugs that are hard to debug.
>
> ```jsx
> // ❌ DANGEROUS: Manual DOM manipulation
> import { useState, useRef } from 'react';
>
> export default function Counter() {
>   const [show, setShow] = useState(true);
>   const ref = useRef(null);
>
>   return (
>     <div>
>       <button onClick={() => setShow(!show)}>
>         Toggle with setState
>       </button>
>       <button
>         onClick={() => {
>           ref.current.remove();  // ❌ Removing element React manages
>         }}
>       >
>         Remove from the DOM
>       </button>
>       {show && <p ref={ref}>Hello world</p>}
>     </div>
>   );
> }
> ```
>
> **Problem**: After manually removing the element, trying to toggle with `setShow` will crash because React expects the element to exist but it's gone.
>
> ```jsx
> // ✅ CORRECT: Let React manage the DOM
> export default function Counter() {
>   const [show, setShow] = useState(true);
>
>   return (
>     <div>
>       <button onClick={() => setShow(!show)}>
>         Toggle with setState
>       </button>
>       {show && <p>Hello world</p>}  {/* React manages this */}
>     </div>
>   );
> }
> ```
>
> **Rule**: If React manages an element (it's in your JSX), don't manually add, remove, or modify it. Use React state and JSX instead.

---

> **⚠️ PITFALL: Don't Access `ref.current` During Rendering**
>
> `ref.current` may be `null` during the initial render. Always access refs in event handlers or effects, not during the render phase.
>
> ```jsx
> // ❌ PROBLEM: Accessing ref during render
> function MyComponent() {
>   const inputRef = useRef(null);
>   
>   // ref.current is null here!
>   inputRef.current.focus();  // Error: Cannot read property 'focus' of null
>   
>   return <input ref={inputRef} />;
> }
> ```
>
> ```jsx
> // ✅ SOLUTION: Access ref in useEffect or event handler
> function MyComponent() {
>   const inputRef = useRef(null);
>   
>   useEffect(() => {
>     // Component has mounted, ref.current is now set
>     inputRef.current?.focus();  // Safe with optional chaining
>   }, []);
>   
>   return <input ref={inputRef} />;
> }
> ```
>
> **Rule**: Only access `ref.current` in:
> - Event handlers
> - `useEffect` hooks
> - Other functions called after render (not during render)

---

> **⚠️ PITFALL: Remember That Refs Don't Trigger Re-renders**
>
> Changing `ref.current` doesn't cause a component to re-render. If you need the UI to update, use state instead.
>
> ```jsx
> // ❌ BAD: Using ref to track value that should update UI
> function Counter() {
>   const countRef = useRef(0);
>   
>   function handleClick() {
>     countRef.current += 1;
>     // UI won't update! User won't see the change.
>   }
>   
>   return (
>     <div>
>       <p>Count: {countRef.current}</p>  {/* Always shows 0 */}
>       <button onClick={handleClick}>Increment</button>
>     </div>
>   );
> }
> ```
>
> ```jsx
> // ✅ GOOD: Use state for values that affect rendering
> function Counter() {
>   const [count, setCount] = useState(0);
>   
>   function handleClick() {
>     setCount(count + 1);  // Triggers re-render, UI updates
>   }
>   
>   return (
>     <div>
>       <p>Count: {count}</p>  {/* Updates correctly */}
>       <button onClick={handleClick}>Increment</button>
>     </div>
>   );
> }
> ```
>
> **Remember**: Refs are for values that don't affect rendering (like DOM nodes, timeout IDs). State is for values that do affect rendering.

---

> **⚠️ PITFALL: Don't Overuse Refs for DOM Manipulation**
>
> Most of the time, you should let React manage the DOM. Only use refs when you need to "step outside React" for browser APIs that React doesn't expose.
>
> ```jsx
> // ❌ OVERUSE: Using refs for everything
> function Form() {
>   const nameRef = useRef(null);
>   const emailRef = useRef(null);
>   
>   function handleSubmit() {
>     // Manually reading form values
>     const name = nameRef.current.value;
>     const email = emailRef.current.value;
>     // ...
>   }
>   
>   return (
>     <form onSubmit={handleSubmit}>
>       <input ref={nameRef} />  {/* Uncontrolled input */}
>       <input ref={emailRef} />
>     </form>
>   );
> }
> ```
>
> ```jsx
> // ✅ BETTER: Use controlled inputs with state
> function Form() {
>   const [name, setName] = useState('');
>   const [email, setEmail] = useState('');
>   
>   function handleSubmit(e) {
>     e.preventDefault();
>     // Use state values directly
>     console.log(name, email);
>   }
>   
>   return (
>     <form onSubmit={handleSubmit}>
>       <input
>         value={name}
>         onChange={(e) => setName(e.target.value)}
>       />
>       <input
>         value={email}
>         onChange={(e) => setEmail(e.target.value)}
>       />
>     </form>
>   );
> }
> ```
>
> **Guideline**: Default to React's declarative approach (controlled components). Use refs only when you need direct DOM access for browser APIs (focus, scroll, media controls, etc.).

---

> **⚠️ PITFALL: Don't Forget to Use `forwardRef` for Custom Components**
>
> You can't pass a ref to a custom component directly. You must use `forwardRef` to expose the DOM node.
>
> ```jsx
> // ❌ WON'T WORK: ref is not a regular prop
> function MyInput(props) {
>   return <input {...props} />;
> }
>
> function Form() {
>   const inputRef = useRef(null);
>   return <MyInput ref={inputRef} />;  // ref will be undefined!
> }
> ```
>
> ```jsx
> // ✅ CORRECT: Use forwardRef
> const MyInput = forwardRef(function MyInput(props, ref) {
>   return <input {...props} ref={ref} />;
> });
>
> function Form() {
>   const inputRef = useRef(null);
>   return <MyInput ref={inputRef} />;  // Works!
> }
> ```
>
> **Rule**: If you want to expose a DOM node from a custom component, wrap it with `forwardRef` and pass the ref to the DOM element.

---

## Complete Example: Form with Multiple Inputs

Here's a complete example showing refs for DOM manipulation:

```jsx
import { useRef, forwardRef } from 'react';

const Input = forwardRef(function Input({ label, ...props }, ref) {
  return (
    <label>
      {label}
      <input {...props} ref={ref} />
    </label>
  );
});

export default function Form() {
  const firstNameRef = useRef(null);
  const lastNameRef = useRef(null);
  const emailRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    // Validate and submit form
  }

  function handleFirstNameFocus() {
    firstNameRef.current?.focus();
  }

  function handleLastNameFocus() {
    lastNameRef.current?.focus();
  }

  function handleEmailFocus() {
    emailRef.current?.focus();
  }

  return (
    <form onSubmit={handleSubmit}>
      <Input
        label="First name:"
        ref={firstNameRef}
        type="text"
        required
      />
      <Input
        label="Last name:"
        ref={lastNameRef}
        type="text"
        required
      />
      <Input
        label="Email:"
        ref={emailRef}
        type="email"
        required
      />
      <button type="submit">Submit</button>
      <div>
        <button type="button" onClick={handleFirstNameFocus}>
          Focus First Name
        </button>
        <button type="button" onClick={handleLastNameFocus}>
          Focus Last Name
        </button>
        <button type="button" onClick={handleEmailFocus}>
          Focus Email
        </button>
      </div>
    </form>
  );
}
```

---

## Best Practices

### 1. Use Refs for Browser APIs Only

Only use refs when you need browser APIs that React doesn't expose:
- Focus/blur management
- Scrolling
- Media playback
- Canvas/WebGL
- Third-party library integration

### 2. Prefer Controlled Components

For form inputs, prefer controlled components (state) over uncontrolled components (refs):

```jsx
// ✅ Better: Controlled
const [value, setValue] = useState('');
<input value={value} onChange={(e) => setValue(e.target.value)} />

// ❌ Avoid: Uncontrolled (unless you have a specific need)
const inputRef = useRef(null);
<input ref={inputRef} />
```

### 3. Always Check for `null`

Before accessing `ref.current`, check if it exists:

```jsx
// ✅ Safe
if (ref.current) {
  ref.current.focus();
}

// ✅ Also safe (optional chaining)
ref.current?.focus();
```

### 4. Access Refs at the Right Time

Access refs in:
- Event handlers
- `useEffect` hooks
- Callback functions

Never during the render phase.

---

## Interview Questions to Consider

### Conceptual Understanding

1. **What is the difference between using `ref` for DOM nodes vs. using refs for values?**
   - When you pass `ref` to a JSX element (like `<div ref={myRef}>`), React automatically sets `ref.current` to the DOM node. For value refs, you manually set `ref.current`. Both use `useRef`, but DOM refs are managed by React, while value refs are managed by you.

2. **When is it safe to modify the DOM directly?**
   - It's safe to modify the DOM when:
     - You're doing non-destructive actions (focus, scroll, reading)
     - You're modifying elements React doesn't manage (empty containers that React never touches)
     - Never safe: modifying elements React controls (removing, adding children, changing text/attributes)

3. **Why do you need `forwardRef` to access a child component's DOM node?**
   - `ref` is a special prop in React, not a regular prop. To pass a ref through a custom component to its DOM element, you must use `forwardRef` to properly forward it. Without `forwardRef`, React won't pass the ref to the child component.

4. **What happens if you try to access `ref.current` during rendering?**
   - During the initial render, `ref.current` will be `null` because React hasn't created the DOM node yet. Accessing it during render can cause errors. Always access refs in event handlers or `useEffect` after the component has mounted.

### Practical Application

5. **Implement a component that focuses an input when a button is clicked:**
   ```jsx
   import { useRef } from 'react';
   
   function FocusInput() {
     const inputRef = useRef(null);
     
     function handleClick() {
       inputRef.current?.focus();
     }
     
     return (
       <>
         <input ref={inputRef} />
         <button onClick={handleClick}>Focus Input</button>
       </>
     );
   }
   ```

6. **Create a custom `Input` component that can receive a ref from a parent:**
   ```jsx
   import { forwardRef } from 'react';
   
   const Input = forwardRef(function Input(props, ref) {
     return <input {...props} ref={ref} />;
   });
   
   // Usage in parent
   function Form() {
     const inputRef = useRef(null);
     return <Input ref={inputRef} />;
   }
   ```

7. **What's wrong with this code?**
   ```jsx
   function Component() {
     const divRef = useRef(null);
     divRef.current.style.color = 'red';  // During render!
     return <div ref={divRef}>Hello</div>;
   }
   ```
   - Accessing `ref.current` during rendering. On the first render, `divRef.current` is `null`, so this will crash. Move the style change to `useEffect` or an event handler.

### Advanced Thinking

8. **How would you measure the size of a DOM element using a ref?**
   ```jsx
   function MeasurableDiv() {
     const divRef = useRef(null);
     
     useEffect(() => {
       if (divRef.current) {
         const { width, height } = divRef.current.getBoundingClientRect();
         console.log(`Width: ${width}, Height: ${height}`);
       }
     }, []);
     
     return <div ref={divRef}>Content</div>;
   }
   ```

9. **Can you have multiple refs in one component?**
   - Yes, you can call `useRef` multiple times to create multiple refs:
   ```jsx
   function Component() {
     const inputRef = useRef(null);
     const buttonRef = useRef(null);
     const divRef = useRef(null);
     // Use each ref independently
   }
   ```

10. **When would you use an uncontrolled input (with ref) vs. a controlled input (with state)?**
    - **Uncontrolled with ref**: When integrating with third-party libraries, when you don't need React to manage the value, or for simple forms where you only need the value on submit.
    - **Controlled with state**: Most of the time—when you need validation, conditional rendering based on value, or real-time UI updates based on input.

---

## Summary

Refs provide a way to access DOM nodes directly, which is necessary for browser APIs that React doesn't expose:

- **Accessing DOM nodes**: Pass `ref={myRef}` to JSX elements to get a reference to the DOM node
- **`forwardRef`**: Use `forwardRef` to expose DOM nodes from custom components
- **Safe operations**: Focus, scroll, reading dimensions are safe. Avoid removing, adding, or modifying React-managed elements
- **Access timing**: Only access `ref.current` in event handlers or `useEffect`, never during rendering
- **Use wisely**: Refs are an escape hatch—default to React's declarative approach and use refs only when necessary

**Key principles:**

- Don't modify DOM elements React manages
- Access refs at the right time (not during render)
- Use `forwardRef` to expose child component DOM nodes
- Prefer controlled components over uncontrolled ones
- Refs don't trigger re-renders—use state if the UI needs to update

Refs are a powerful tool for DOM manipulation when used correctly and sparingly.


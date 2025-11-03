# Responding to Events

## Overview
React lets you add event handlers to your JSX. Event handlers are your own functions that respond to user interactions like clicking, hovering, focusing form inputs, and more.

## What You Will Learn
- Different ways to write event handlers
- Passing event handling logic from parent to child components
- How events propagate and methods to control propagation

---

## Adding Event Handlers

### What Are Event Handlers?

**Event handlers** are custom functions that execute in response to user interactions such as:
- Clicking
- Hovering
- Focusing form inputs
- Typing
- Submitting forms

### How to Add an Event Handler

**Step 1: Define a function inside your component**
```jsx
function Button() {
  function handleClick() {
    alert('You clicked me!');
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

**Step 2: Pass it as a prop to the JSX element**

Notice that you **pass the function**, not call it:
- ✅ `onClick={handleClick}`
- ❌ `onClick={handleClick()}`

### Naming Convention

Handler functions typically follow this pattern:
- **`handle` + event name**
- Examples: `handleClick`, `handleMouseEnter`, `handleSubmit`, `handleChange`

---

## Passing Functions vs. Calling Functions

> **⚠️ PITFALL**
>
> **Critical difference between passing and calling:**
>
> ```jsx
> // ✅ Correct: Passes the function
> <button onClick={handleClick}>Click me</button>
>
> // ❌ Wrong: Calls the function immediately during render
> <button onClick={handleClick()}>Click me</button>
> ```
>
> The difference is subtle but crucial:
> - **First example:** The `handleClick` function is passed as an event handler. React stores it and calls it when the user clicks.
> - **Second example:** The `()` at the end **fires the function immediately during rendering**, without any clicks. This causes the function to run every time the component renders.

### Why This Matters

Functions passed to event handlers should be:
- **Passed, not called**
- Executed only when the event occurs, not during rendering

---

## Different Ways to Define Event Handlers

### Method 1: Separate Function
```jsx
function Button() {
  function handleClick() {
    alert('Clicked!');
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

### Method 2: Inline Function
```jsx
function Button() {
  return (
    <button onClick={() => alert('Clicked!')}>
      Click me
    </button>
  );
}
```

### Method 3: Inline Arrow Function
```jsx
function AlertButton({ message }) {
  return (
    <button onClick={() => alert(message)}>
      Show alert
    </button>
  );
}
```

**When to use inline handlers:**
- Short, simple logic
- Need to pass arguments to a handler
- One-off handlers not reused elsewhere

---

## Reading Props in Event Handlers

Event handlers are defined inside components, so they have **access to props through closure**:

```jsx
function AlertButton({ message, children }) {
  function handleClick() {
    alert(message); // Can access props!
  }

  return <button onClick={handleClick}>{children}</button>;
}

function App() {
  return (
    <div>
      <AlertButton message="Playing!">Play Movie</AlertButton>
      <AlertButton message="Uploading!">Upload Image</AlertButton>
    </div>
  );
}
```

---

## Passing Event Handlers as Props

Parent components can pass event handlers to children, making components reusable with different behaviors:

```jsx
function Button({ onClick, children }) {
  return <button onClick={onClick}>{children}</button>;
}

function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`Playing ${movieName}!`);
  }

  return <Button onClick={handlePlayClick}>Play "{movieName}"</Button>;
}

function UploadButton() {
  function handleUploadClick() {
    alert('Uploading!');
  }

  return <Button onClick={handleUploadClick}>Upload Image</Button>;
}
```

**Pattern:**
1. Parent defines the handler function
2. Parent passes it as a prop to child
3. Child passes it to the appropriate element

---

## Naming Event Handler Props

### Built-in Components

Built-in browser elements (like `<button>`, `<div>`) use standard browser event names:
- `onClick`
- `onChange`
- `onSubmit`
- `onMouseEnter`
- etc.

### Custom Components

For custom components, you can name event handler props anything you want. **Convention: Start with `on` followed by a capital letter.**

```jsx
function Button({ onSmash, children }) {
  return <button onClick={onSmash}>{children}</button>;
}

function App() {
  return (
    <Button onSmash={() => alert('Smashed!')}>
      Smash me
    </Button>
  );
}
```

**Application-specific names:**
- `onPlayMovie`
- `onUploadImage`
- `onAddToCart`
- `onDeleteItem`

These names should reflect what the action means in your application's domain, not just the underlying DOM event.

---

## Event Propagation

### What is Event Propagation?

Events **propagate** (or "bubble") up the component tree. When you click a button inside a div, the click event fires on the button first, then on the div, then on its parents.

```jsx
function Toolbar() {
  function handleToolbarClick() {
    alert('Toolbar clicked!');
  }

  function handleButtonClick() {
    alert('Button clicked!');
  }

  return (
    <div onClick={handleToolbarClick}>
      <button onClick={handleButtonClick}>Click me</button>
    </div>
  );
}
```

**What happens when you click the button:**
1. `handleButtonClick` runs (button's handler)
2. `handleToolbarClick` runs (parent div's handler)

**Bubbling behavior:** Events travel upward from the target element through its ancestors.

---

## Stopping Propagation

To prevent an event from reaching parent handlers, use **`e.stopPropagation()`**:

```jsx
function Toolbar() {
  function handleToolbarClick() {
    alert('Toolbar clicked!');
  }

  function handleButtonClick(e) {
    e.stopPropagation(); // Stop the event from bubbling up
    alert('Button clicked!');
  }

  return (
    <div onClick={handleToolbarClick}>
      <button onClick={handleButtonClick}>Click me</button>
    </div>
  );
}
```

**Now when you click the button:**
1. Only `handleButtonClick` runs
2. `handleToolbarClick` does NOT run (propagation stopped)

### The Event Object

Event handlers receive an **event object** as their first parameter:

```jsx
function handleClick(e) {
  e.stopPropagation(); // Stop bubbling
  console.log(e.target); // The element that was clicked
}
```

---

## Preventing Default Behavior

Some browser events have default behaviors (e.g., form submission reloads the page). Use **`e.preventDefault()`** to block these:

```jsx
function Signup() {
  function handleSubmit(e) {
    e.preventDefault(); // Prevent page reload
    alert('Submitting!');
  }

  return (
    <form onSubmit={handleSubmit}>
      <input />
      <button type="submit">Submit</button>
    </form>
  );
}
```

### Key Distinction

| Method | What It Does |
|--------|--------------|
| `e.stopPropagation()` | Prevents parent handlers from firing |
| `e.preventDefault()` | Blocks the browser's default action for that event |

**Example use cases:**
- `preventDefault()`: Stop form submission, prevent link navigation
- `stopPropagation()`: Handle click on child without triggering parent's click handler

---

## Capture Phase Events

Events propagate in three phases:

1. **Capture phase**: Travels down from root to target
2. **Target phase**: Reaches the target element
3. **Bubble phase**: Bubbles up from target to root

By default, React uses the **bubble phase**. To catch events during the **capture phase**, add `Capture` to the event name:

```jsx
<div onClickCapture={() => { /* runs during capture phase */ }}>
  <button onClick={() => { /* runs during bubble phase */ }}>
    Click me
  </button>
</div>
```

**Use case:** Capture phase handlers run even if child elements call `stopPropagation()`. Useful for analytics or routing logic that should always execute.

---

## Can Event Handlers Have Side Effects?

**Yes! Event handlers are the best place for side effects.**

Unlike render functions (which must be pure), event handlers don't need to be pure. They're perfect for:
- Changing state
- Making API calls
- Updating the DOM
- Triggering animations
- Logging analytics

```jsx
function UploadButton() {
  function handleUpload() {
    // ✅ Side effects are fine here!
    uploadFile();
    showNotification('Upload started');
    trackAnalytics('file_upload');
  }

  return <button onClick={handleUpload}>Upload</button>;
}
```

---

## Interview Questions to Consider

1. **Q: What's the difference between `onClick={handleClick}` and `onClick={handleClick()}`?**
   - A: The first passes the function reference to be called later when the event occurs. The second calls the function immediately during rendering.

2. **Q: Why can event handlers access props?**
   - A: Event handlers are defined inside components, so they have access to props through JavaScript closure.

3. **Q: What is event propagation (bubbling)?**
   - A: Events travel upward from the target element through its ancestor elements. A click on a button also triggers click handlers on parent elements.

4. **Q: What's the difference between `stopPropagation()` and `preventDefault()`?**
   - A: `stopPropagation()` stops the event from reaching parent handlers. `preventDefault()` blocks the browser's default action (like form submission or link navigation).

5. **Q: When should you use inline event handlers vs. separate functions?**
   - A: Use inline for short, simple logic or when you need to pass arguments. Use separate functions for complex logic, reusability, or better readability.

6. **Q: What naming convention should custom event handler props follow?**
   - A: Start with "on" followed by a capital letter (e.g., `onPlayMovie`, `onUploadImage`), using application-specific names that describe the action's meaning.

7. **Q: What are the three phases of event propagation?**
   - A: Capture (down from root), Target (at the element), and Bubble (up to root). React uses bubble phase by default.

8. **Q: Can event handlers have side effects?**
   - A: Yes! Unlike render functions, event handlers don't need to be pure. They're the ideal place for side effects like API calls, state updates, and DOM manipulation.

---

## Key Takeaways

- Event handlers are functions that respond to user interactions
- **Pass functions, don't call them**: `onClick={handleClick}`, not `onClick={handleClick()}`
- Name handlers with "handle" prefix: `handleClick`, `handleSubmit`
- Event handlers can access props through closure
- Parent components can pass handlers to children as props
- Custom event handler props should start with "on": `onPlayMovie`, `onUploadImage`
- Events **propagate (bubble) up** from child to parent elements
- Use `e.stopPropagation()` to prevent events from reaching parents
- Use `e.preventDefault()` to block browser default behaviors
- Event handlers receive an event object as the first parameter
- Capture phase handlers (add `Capture` suffix) run even if propagation is stopped
- **Event handlers are the right place for side effects** (unlike render functions)

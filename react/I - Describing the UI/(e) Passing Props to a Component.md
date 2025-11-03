# Passing Props to a Component

## Overview
React components use **props** (short for "properties") to communicate with each other. Props allow parent components to pass data to their children, making components reusable and configurable.

## What You Will Learn
- How to pass props to a component
- How to read props from a component
- How to specify default values for props
- How to pass some JSX to a component
- How props change over time

---

## What Are Props?

**Props** are arguments passed to React components, similar to function parameters. They work like HTML attributes but can accept **any JavaScript value**, including:
- Strings and numbers
- Booleans
- Objects and arrays
- Functions
- JSX elements

Props enable parent-to-child communication, allowing you to customize child components from the parent.

---

## Passing Props to a Component

Props are passed to components using JSX attributes:

```jsx
function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
```

Here, `person` and `size` are props passed to the `Avatar` component.

**Key Points:**
- Use curly braces `{}` for JavaScript values (objects, numbers, etc.)
- Use quotes `""` for string literals
- Multiple props can be passed to a single component

---

## Reading Props from a Component

### Method 1: Destructuring (Recommended)

Access props by destructuring them in the function parameters:

```jsx
function Avatar({ person, size }) {
  return (
    <img
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}
```

### Method 2: Props Object

Access props as properties of a `props` object:

```jsx
function Avatar(props) {
  return (
    <img
      src={getImageUrl(props.person)}
      alt={props.person.name}
      width={props.size}
      height={props.size}
    />
  );
}
```

> **⚠️ PITFALL**
>
> Don't forget the curly braces when destructuring:
> ```jsx
> function Avatar({ person, size }) // ✅ Correct
> function Avatar(person, size)    // ❌ Wrong - props aren't separate parameters
> ```

---

## Specifying Default Values for Props

You can provide default values during destructuring:

```jsx
function Avatar({ person, size = 100 }) {
  // size will be 100 if not provided
  return (
    <img
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}
```

### When Defaults Apply

**Defaults are used when:**
- The prop is not provided: `<Avatar person={...} />`
- The prop is explicitly `undefined`: `<Avatar person={...} size={undefined} />`

**Defaults are NOT used when:**
- The prop is `null`: `<Avatar person={...} size={null} />`
- The prop is `0`: `<Avatar person={...} size={0} />`
- The prop is an empty string: `<Avatar person={...} size="" />`

> **⚠️ PITFALL**
>
> Default values only apply when the prop is missing or `undefined`. Passing `null` or `0` will **not** trigger the default value.

---

## Forwarding Props with Spread Syntax

You can forward all props from a parent to a child using the **spread operator** (`...`):

```jsx
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```

This is equivalent to:
```jsx
<Avatar person={props.person} size={props.size} />
```

### When to Use Spread Syntax

**Use it sparingly!** Spread syntax should be used when:
- Creating wrapper components
- Forwarding many props would be repetitive

**Avoid it when:**
- It reduces code readability
- It's unclear which props are being passed
- You only need to forward a few specific props

> **⚠️ PITFALL**
>
> Overusing spread syntax can make code harder to understand and maintain. Explicit props are usually clearer.

---

## Passing JSX as Children

Components can receive nested JSX content through the special **`children` prop**:

```jsx
function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

function Profile() {
  return (
    <Card>
      <Avatar person={...} size={100} />
    </Card>
  );
}
```

**How it works:**
- Content between `<Card>` and `</Card>` becomes the `children` prop
- `children` can be any valid JSX: elements, text, multiple elements, etc.

### Think of Children as "Slots"

The `children` prop acts like a "slot" that the parent component fills with content. This pattern is powerful for creating reusable wrapper components like cards, modals, and layouts.

```jsx
function Panel({ children }) {
  return (
    <section className="panel">
      {children}
    </section>
  );
}

<Panel>
  <h1>Title</h1>
  <p>Some content here</p>
</Panel>
```

---

## How Props Change Over Time

### Props Are Immutable

**Props are read-only.** A component cannot change its own props.

```jsx
function Avatar({ size }) {
  size = size + 10; // ❌ Don't do this!
  // ...
}
```

> **⚠️ PITFALL**
>
> **Do not attempt to modify props directly.** Props are immutable and should never be changed by the receiving component. If you need to change data based on user interaction, use **state** instead.

### Props Reflect Data at Render Time

Props represent a component's data at a **specific point in time**. Each render receives a new "snapshot" of props.

When a parent component passes different prop values, React:
1. Re-renders the child component
2. Provides the new prop values
3. The child receives fresh, updated props

**Key principle:** To change props, the parent component must pass different values, triggering a re-render.

---

## Interview Questions to Consider

1. **Q: What are props in React?**
   - A: Props are arguments passed to React components, similar to function parameters. They allow parent components to pass data to children and can be any JavaScript value.

2. **Q: Are props mutable or immutable?**
   - A: Props are **immutable**. A component cannot modify its own props. To change data, the parent must pass different prop values.

3. **Q: What's the difference between these two syntaxes?**
   ```jsx
   function Avatar({ person, size }) { }
   function Avatar(props) { }
   ```
   - A: The first uses destructuring to extract specific props directly. The second receives all props as an object and requires accessing them via `props.person`, `props.size`, etc.

4. **Q: When does a default prop value get used?**
   - A: Default values are used when a prop is missing or explicitly set to `undefined`. They are NOT used for `null`, `0`, `false`, or empty strings.

5. **Q: What is the `children` prop?**
   - A: `children` is a special prop that contains the content nested between a component's opening and closing tags. It allows components to wrap arbitrary JSX content.

6. **Q: Should you always use spread syntax to forward props?**
   - A: No, spread syntax should be used sparingly. While convenient for forwarding many props, it can reduce code clarity. Explicit props are usually better for maintainability.

7. **Q: How do props change over time?**
   - A: Props themselves don't change—they're immutable. However, when a parent passes different values, React re-renders the child with new props, creating a new "snapshot" of data.

---

## Key Takeaways

- Props enable parent-to-child communication in React
- Pass props like HTML attributes, but they accept any JavaScript value
- Read props via destructuring or the `props` object
- Default values apply only when props are missing or `undefined`
- The `children` prop contains nested JSX content
- Props are **immutable**—components cannot modify them
- Use state (not props) when you need to change data based on user interaction
- Props reflect component data at a specific point in time
- Spread syntax exists but should be used judiciously

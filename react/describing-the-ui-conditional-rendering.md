# Conditional Rendering

## Overview
Your components often need to display different content based on conditions. In React, you can conditionally render JSX using JavaScript syntax like `if` statements, `&&`, and `?` (ternary) operators.

## What You Will Learn
- How to return different JSX depending on a condition
- How to conditionally include or exclude JSX pieces
- Common conditional syntax shortcuts encountered in React codebases

---

## Conditionally Returning JSX

Use `if`/`else` statements to return completely different JSX based on conditions:

```jsx
function Item({ name, isPacked }) {
  if (isPacked) {
    return <li className="item">{name} ✅</li>;
  }
  return <li className="item">{name}</li>;
}
```

This approach is straightforward and works well when rendering entirely different markup for each case.

---

## Returning Nothing with `null`

If you don't want to render anything, return `null`:

```jsx
function Item({ name, isPacked }) {
  if (isPacked) {
    return null;
  }
  return <li className="item">{name}</li>;
}
```

**When to use:** When a component should sometimes render nothing at all.

**Note:** Returning `null` is uncommon in practice. More often, you'll conditionally include or exclude a component in the parent's JSX.

---

## Conditional (Ternary) Operator

The **ternary operator** (`condition ? trueCase : falseCase`) provides a compact alternative to `if`/`else`:

```jsx
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {isPacked ? name + ' ✅' : name}
    </li>
  );
}
```

### When to Use Ternary Operators

**Good use cases:**
- Simple, short expressions
- Choosing between two pieces of text or elements
- When the condition is easy to understand

**Avoid when:**
- Nesting gets too deep (hard to read)
- The logic becomes complex
- You need more than two alternatives

### Ternary for Elements

You can also choose between entire JSX elements:

```jsx
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {isPacked ? (
        <del>{name} ✅</del>
      ) : (
        name
      )}
    </li>
  );
}
```

---

## Logical AND Operator (`&&`)

The **logical AND operator** renders content only when a condition is true:

```jsx
function Item({ name, isPacked }) {
  return (
    <li className="item">
      {name} {isPacked && '✅'}
    </li>
  );
}
```

**How it works:**
- If `isPacked` is `true`, React renders `'✅'`
- If `isPacked` is `false`, React skips it and renders nothing

**Read as:** "If `isPacked`, then render the checkmark"

### When to Use `&&`

Use `&&` when you want to:
- Conditionally show something
- Render nothing if the condition is false
- Keep the code concise

> **⚠️ PITFALL**
>
> **Do not put numbers on the left side of `&&`!**
>
> JavaScript automatically converts the left side to a boolean. However, if the left side is `0`, React will render `0` instead of rendering nothing.
>
> **❌ Common mistake:**
> ```jsx
> {messageCount && <p>New messages</p>}
> // If messageCount is 0, this renders "0" on the page!
> ```
>
> **✅ Correct approach:**
> ```jsx
> {messageCount > 0 && <p>New messages</p>}
> // Now it renders nothing when messageCount is 0
> ```

---

## Conditionally Assigning JSX to Variables

For more complex logic, assign JSX to variables using `if` statements:

```jsx
function Item({ name, isPacked }) {
  let itemContent = name;

  if (isPacked) {
    itemContent = name + ' ✅';
  }

  return (
    <li className="item">
      {itemContent}
    </li>
  );
}
```

### Advantages of This Approach

- **Maximum readability** for complex conditions
- **Easy to debug** with clear variable names
- **Works well** when combining multiple conditions
- **Flexible** for both simple values and complex JSX

### Example with Complex JSX

```jsx
function Item({ name, isPacked }) {
  let itemContent = name;

  if (isPacked) {
    itemContent = (
      <del>
        {name + ' ✅'}
      </del>
    );
  }

  return (
    <li className="item">
      {itemContent}
    </li>
  );
}
```

---

## Choosing the Right Approach

| Technique | Best For | Example |
|-----------|----------|---------|
| **`if`/`else`** | Returning completely different JSX | Different components for logged in vs. logged out |
| **`null`** | Rendering nothing | Hide component when condition is false |
| **Ternary `? :`** | Choosing between two short expressions | Show "Yes" or "No" |
| **Logical AND `&&`** | Conditionally showing one thing | Show notification badge if count > 0 |
| **Variable assignment** | Complex conditions or multi-step logic | Building JSX based on multiple checks |

---

## Practical Examples

### Example 1: User Status
```jsx
function UserGreeting({ isLoggedIn, username }) {
  if (isLoggedIn) {
    return <h1>Welcome back, {username}!</h1>;
  }
  return <h1>Please sign in.</h1>;
}
```

### Example 2: Notification Badge
```jsx
function NotificationBadge({ count }) {
  return (
    <div>
      <span>Messages</span>
      {count > 0 && <span className="badge">{count}</span>}
    </div>
  );
}
```

### Example 3: Loading State
```jsx
function Content({ isLoading, data }) {
  return (
    <div>
      {isLoading ? (
        <Spinner />
      ) : (
        <DataDisplay data={data} />
      )}
    </div>
  );
}
```

### Example 4: Complex Conditional Logic
```jsx
function PackingItem({ name, isPacked, importance }) {
  let itemContent = name;

  if (isPacked && importance === 'high') {
    itemContent = <strong>{name} ✅</strong>;
  } else if (isPacked) {
    itemContent = name + ' ✅';
  } else if (importance === 'high') {
    itemContent = <strong>{name}</strong>;
  }

  return <li className="item">{itemContent}</li>;
}
```

---

## Interview Questions to Consider

1. **Q: What are the different ways to conditionally render in React?**
   - A: (1) `if`/`else` statements, (2) returning `null`, (3) ternary operator `? :`, (4) logical AND `&&`, and (5) assigning JSX to variables with `if` statements.

2. **Q: When should you use `&&` vs. ternary operator for conditional rendering?**
   - A: Use `&&` when you want to render something or nothing. Use ternary `? :` when you want to choose between two alternatives.

3. **Q: What happens if you put `0` on the left side of `&&`?**
   - A: React will render the number `0` instead of rendering nothing, because `0` is a falsy value that JavaScript converts to `false`, but React still renders the `0`.

4. **Q: What does returning `null` from a component do?**
   - A: It tells React to render nothing for that component. The component is mounted but produces no DOM output.

5. **Q: Can you nest ternary operators? Should you?**
   - A: Yes, you can nest them, but it quickly becomes hard to read. If you need multiple conditions, use `if`/`else` statements or variable assignment instead.

6. **Q: Is it better to return early with `if` or use ternary operators?**
   - A: Both are valid. Early returns are clearer for completely different render paths. Ternaries work well for small differences within similar JSX structures.

---

## Key Takeaways

- React uses JavaScript control flow for conditional rendering
- Multiple techniques exist—choose based on readability and complexity
- `if`/`else` returns different JSX trees entirely
- Ternary operators `? :` choose between two expressions
- `&&` operator shows content or nothing
- **Never put numbers on the left of `&&`** (use explicit comparisons like `count > 0`)
- Variable assignment with `if` offers maximum flexibility
- Plain `if` statements are always acceptable—don't feel pressured to use shortcuts
- Readability should guide your choice of conditional rendering technique

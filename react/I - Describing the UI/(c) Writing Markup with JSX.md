# Writing Markup with JSX

## Overview
JSX is a syntax extension for JavaScript that allows you to write HTML-like markup inside JavaScript files. It's the standard way to define component structure in React.

## What You Will Learn
- Why React combines markup with rendering logic
- Distinctions between JSX and HTML syntax
- Methods for displaying dynamic information using JSX

---

## What is JSX?

**JSX (JavaScript XML)** is a syntax extension that lets you write HTML-like markup inside JavaScript files. It makes component code more readable and expressive by keeping rendering logic and markup together.

```jsx
function Greeting() {
  return <h1>Hello, World!</h1>;
}
```

---

## Why React Uses JSX

Traditional web development separated concerns by technology:
- **HTML** for content
- **CSS** for styling
- **JavaScript** for logic

React takes a different approach: **"In React, rendering logic and markup live together in the same place—components."**

### Benefits of Co-location
- Changes to markup and logic stay synchronized
- Each component encapsulates its own concerns
- Unrelated components remain isolated from each other

> **⚠️ IMPORTANT**
>
> JSX and React are **two separate things**. They're often used together, but you can use them independently. JSX is a syntax extension, while React is a JavaScript library.

---

## The Three Rules of JSX

### Rule 1: Return a Single Root Element

Components must return **one parent element**. You cannot return multiple sibling elements directly.

**❌ Incorrect:**
```jsx
function Profile() {
  return
    <img src="photo.jpg" />
    <h1>Name</h1>
}
```

**✅ Correct with div:**
```jsx
function Profile() {
  return (
    <div>
      <img src="photo.jpg" />
      <h1>Name</h1>
    </div>
  );
}
```

**✅ Correct with Fragment:**
```jsx
function Profile() {
  return (
    <>
      <img src="photo.jpg" />
      <h1>Name</h1>
    </>
  );
}
```

**Note:** Use Fragments (`<>...</>`) when you don't want to add extra nodes to the DOM.

---

### Rule 2: Close All Tags

In JSX, all tags must be explicitly closed.

**Self-closing tags:**
```jsx
<img src="photo.jpg" />
<br />
<input type="text" />
```

**Wrapping tags:**
```jsx
<li>Item</li>
<div>Content</div>
```

> **⚠️ PITFALL**
>
> HTML allows unclosed tags like `<img>` or `<br>`, but JSX requires explicit closing: `<img />` and `<br />`. Forgetting to close tags will cause syntax errors.

---

### Rule 3: Use camelCase for Attributes

Most HTML attributes convert to camelCase in JSX because they become JavaScript object properties.

**Common conversions:**
| HTML | JSX |
|------|-----|
| `class` | `className` |
| `for` | `htmlFor` |
| `stroke-width` | `strokeWidth` |
| `onclick` | `onClick` |
| `tabindex` | `tabIndex` |

**Example:**
```jsx
<div className="container">
  <label htmlFor="name">Name:</label>
  <input id="name" type="text" />
</div>
```

**Exceptions:**
- `aria-*` attributes keep their dashes: `aria-label`, `aria-hidden`
- `data-*` attributes keep their dashes: `data-testid`, `data-id`

> **⚠️ PITFALL**
>
> `class` is a reserved JavaScript keyword, so React uses `className` instead. Using `class` in JSX will not work as expected.

---

## Common HTML-to-JSX Conversion Errors

When converting HTML to JSX, watch out for:

1. **Multiple root elements** without a wrapper
2. **Unclosed self-closing tags** like `<img>` or `<input>`
3. **Reserved keywords** used as attributes (`class`, `for`)
4. **Dashed attribute names** that need camelCase conversion

**HTML:**
```html
<div class="card">
  <img src="photo.jpg">
  <label for="email">Email</label>
  <input id="email" type="email">
</div>
```

**JSX:**
```jsx
<div className="card">
  <img src="photo.jpg" />
  <label htmlFor="email">Email</label>
  <input id="email" type="email" />
</div>
```

---

## Interview Questions to Consider

1. **Q: What is JSX and why does React use it?**
   - A: JSX is a syntax extension that allows writing HTML-like markup in JavaScript. React uses it because rendering logic and markup are inherently coupled—keeping them together in components ensures consistency and better encapsulation.

2. **Q: What are the three main rules of JSX?**
   - A: (1) Return a single root element, (2) Close all tags explicitly, (3) Use camelCase for most attributes.

3. **Q: Why do we use `className` instead of `class` in JSX?**
   - A: Because `class` is a reserved keyword in JavaScript. JSX attributes become JavaScript object properties, so we need to use valid JavaScript identifiers.

4. **Q: What's the difference between `<div>` and `<>` as wrapper elements?**
   - A: `<div>` adds an actual DOM node, while `<>` (Fragment) is a wrapper that doesn't create any extra DOM elements. Fragments are useful when you need to group elements without affecting the DOM structure.

5. **Q: Are JSX and React the same thing?**
   - A: No, they are separate. JSX is a syntax extension for JavaScript, while React is a library. You can use JSX without React and React without JSX (though JSX is the conventional way).

6. **Q: Which HTML attributes keep their dash syntax in JSX?**
   - A: `aria-*` attributes (for accessibility) and `data-*` attributes (for custom data) retain their dashes in JSX.

---

## Key Takeaways

- JSX combines markup and logic in components for better co-location
- Always return a single root element (use Fragments to avoid extra DOM nodes)
- All tags must be explicitly closed in JSX
- Most attributes convert to camelCase, except `aria-*` and `data-*`
- For large HTML conversions, use converter tools, but understand the rules manually
- JSX is a syntax extension separate from React itself

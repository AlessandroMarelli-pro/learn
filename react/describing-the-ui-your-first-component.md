# Your First Component

## You Will Learn

- What a component is
- What role components play in a React application
- How to write your first React component

---

## What is a Component?

Components are one of React's core concepts. They are the foundation upon which you build user interfaces (UI), making them the perfect place to start your React journey.

In React, **components are reusable UI building blocks**. Instead of writing plain HTML, you create custom components that combine markup, CSS, and JavaScript logic into self-contained pieces. A component might be as small as a button, or as large as an entire page.

React applications are built by composing these components together. For example, instead of repeating table of contents markup across multiple pages, you could create a `<TableOfContents />` component and use it wherever needed.

---

## Defining a Component

React components are JavaScript functions that return markup. Here's how to create one:

### Step 1: Export the Component

Use the `export default` prefix to make the component available for import in other files:

```jsx
export default function Profile() {
  // component code
}
```

### Step 2: Define the Function

Create a JavaScript function. Component names **must start with a capital letter**:

```jsx
function Profile() {
  // component code
}
```

### Step 3: Add Markup (Return JSX)

Return the markup you want to render. JSX looks like HTML but is actually JavaScript:

```jsx
export default function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3Am.jpg"
      alt="Katherine Johnson"
    />
  );
}
```

> **⚠️ PITFALL:** React components are regular JavaScript functions, but **their names must start with a capital letter** or they won't work!

> **⚠️ PITFALL:** Without parentheses, any code on the lines after `return` will be ignored! If your JSX spans multiple lines, wrap it in parentheses:
> ```jsx
> return (
>   <div>
>     <img src="image.jpg" />
>   </div>
> );
> ```

---

## Using a Component

Once you've defined a component, you can use it inside other components:

```jsx
function Profile() {
  return (
    <img
      src="https://i.imgur.com/MK3eW3As.jpg"
      alt="Katherine Johnson"
    />
  );
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

### Key Points About Using Components:

- **Components can be reused**: `<Profile />` appears three times in the example above
- **Components can be nested**: `Gallery` renders multiple `Profile` components
- **React components must be capitalized**: `<Profile />` (component) vs `<section>` (HTML tag)
- **HTML tags are lowercase**: React knows `<section>` is an HTML tag because it's lowercase

---

## Nesting and Organizing Components

Components are regular JavaScript functions, so you can keep multiple components in the same file. This is convenient when components are relatively small or tightly related.

However, as your file grows, you can move components to separate files for better organization.

> **⚠️ PITFALL:** Components can render other components, but **you must never nest their definitions**:
> ```jsx
> // ❌ NEVER DO THIS
> export default function Gallery() {
>   function Profile() {  // Don't define components inside other components!
>     // ...
>   }
>   // ...
> }
> ```
>
> ```jsx
> // ✅ DO THIS INSTEAD
> export default function Gallery() {
>   // ...
> }
>
> function Profile() {  // Define components at the top level
>   // ...
> }
> ```
>
> **Why?** Nesting component definitions causes performance problems and bugs because React will recreate the child component from scratch on every render, losing its state.

---

## Components All the Way Down

Your React application begins at a "root" component. Usually, this is created automatically when you start a new project.

- **Most React apps use components all the way down**: You'll use components not just for reusable pieces like buttons, but also for larger pieces like sidebars, lists, and entire pages.
- **Frameworks like Next.js go further**: They can generate pages from your components, meaning even "the page" itself is a component.

Components are a handy way to organize UI code and markup, even if some are only used once.

---

## Interview Questions to Consider

### Conceptual Understanding
1. **What is a React component?**
   - A reusable UI building block that combines markup, styling, and logic into a JavaScript function that returns JSX.

2. **Why must React component names start with a capital letter?**
   - React uses capitalization to distinguish between HTML tags (lowercase) and custom components (capitalized). Without capitalization, React treats it as an HTML tag.

3. **What is JSX?**
   - JSX is a syntax extension that allows you to write HTML-like markup inside JavaScript. It gets transformed into regular JavaScript function calls.

### Practical Application
4. **What's wrong with this code?**
   ```jsx
   function myComponent() {
     return <div>Hello</div>;
   }
   ```
   - The component name should start with a capital letter: `MyComponent`

5. **Why is this considered bad practice?**
   ```jsx
   function Parent() {
     function Child() {
       return <div>Child</div>;
     }
     return <Child />;
   }
   ```
   - Never nest component definitions. The `Child` component will be recreated on every render, causing performance issues and state loss.

6. **What will happen if you forget parentheses in a multi-line return?**
   ```jsx
   return
     <div>
       <h1>Title</h1>
     </div>
   ```
   - JavaScript's automatic semicolon insertion will add a semicolon after `return`, causing the function to return `undefined` instead of the JSX.

### Architecture & Design
7. **How do components promote code reusability?**
   - Components can be used multiple times throughout an application, reducing code duplication and making updates easier (change once, reflect everywhere).

8. **What's the difference between `<Profile />` and `<section>` in React?**
   - `<Profile />` is a custom React component (capitalized), while `<section>` is a native HTML element (lowercase).

9. **When should you split a component into a separate file?**
   - When it grows large, when it's reused in multiple places, or when you want better code organization. Start simple and refactor as needed.

### Advanced Thinking
10. **Can a component return multiple root elements?**
    - Not directly. You need to wrap them in a parent element (like `<div>`) or use a React Fragment (`<>...</>`).

---

## Summary

Components are the fundamental building blocks of React applications. They are JavaScript functions that:
- Must have names starting with a capital letter
- Return JSX markup
- Can be reused and composed together
- Should be defined at the top level (never nested)

By mastering components, you unlock the power to build modular, maintainable, and scalable user interfaces.
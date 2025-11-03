# Importing and Exporting Components

## Overview
As your React application grows, organizing components across multiple files becomes essential. This guide covers how to split components into separate files and make them reusable through imports and exports.

## What You Will Learn
- What constitutes a root component file
- Methods for importing and exporting components
- Appropriate contexts for default versus named imports/exports
- Techniques for exporting multiple components from a single file
- Strategies for splitting components across multiple files

---

## Root Component File

Your main application component typically resides in a **root component file**, commonly named `App.js`. This is where your main component tree starts.

**Important:** Framework-based routing systems like Next.js may utilize different root components per page, depending on the route structure.

---

## Three-Step Component Migration Process

When you want to move a component to its own file:

1. **Create** a new JavaScript file for your component
2. **Export** the function component using either default or named export
3. **Import** it using the corresponding technique in the consuming file

---

## Default vs. Named Exports

### Default Exports

**Characteristics:**
- One file can have **only one** default export
- You can import with any name you choose
- Optimal for files that export a single component

**Syntax:**
```jsx
// Button.js
export default function Button() {
  return <button>Click me</button>;
}

// App.js
import Button from './Button.js';
import MyButton from './Button.js'; // You can use any name
```

### Named Exports

**Characteristics:**
- A file can have **multiple** named exports
- Import names must match export names exactly
- Preferred when exporting multiple items from one file

**Syntax:**
```jsx
// Gallery.js
export function Profile() {
  return <img src="profile.jpg" />;
}

export function Gallery() {
  return (
    <div>
      <Profile />
    </div>
  );
}

// App.js
import { Profile, Gallery } from './Gallery.js';
```

---

## Best Practices

### When to Use Each Type

| Export Type | Use Case |
|-------------|----------|
| **Default Export** | File exports only one component |
| **Named Export** | File exports multiple components or utilities |

> **⚠️ PITFALL**
>
> A file can have **no more than one default export**, but it can have **as many named exports as you like**.
>
> Components lacking descriptive names (like `export default () => {}`) are discouraged since they complicate debugging and make code harder to maintain.

### File Extension Convention

Both import syntaxes work in React:
- `import Gallery from './Gallery.js'` (explicit extension)
- `import Gallery from './Gallery'` (implicit extension)

**Note:** The explicit `.js` extension aligns more closely with native ES Modules behavior and is generally recommended for clarity.

---

## Interview Questions to Consider

1. **Q: What's the difference between default and named exports?**
   - A: Default exports allow one export per file and can be imported with any name. Named exports allow multiple exports per file but require exact name matching on import.

2. **Q: Can you mix default and named exports in the same file?**
   - A: Yes, but it's generally better to stick to one pattern per file for consistency.

3. **Q: Why might named exports be preferred in larger codebases?**
   - A: Named exports enforce consistent naming across the codebase, making it easier to search and refactor. They also enable better IDE autocomplete support.

4. **Q: What is a root component file?**
   - A: The root component file contains the main component where your application's component tree starts (commonly `App.js`).

5. **Q: How do frameworks like Next.js handle root components differently?**
   - A: Next.js uses file-based routing where each page can have its own root component, rather than a single application-wide root.

---

## Key Takeaways

- Splitting components into separate files improves code organization and reusability
- Use **default exports** for single-component files
- Use **named exports** for multiple exports from one file
- Always use descriptive names for your components
- The three-step process (create, export, import) is fundamental to component organization

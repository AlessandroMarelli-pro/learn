# Passing Data Deeply with Context

## You Will Learn

- The problem with passing props through many levels (prop drilling)
- How Context allows you to pass data deeply without prop drilling
- When Context is a good choice and when it's not
- Common patterns for using Context

---

## The Problem: Prop Drilling

Passing props is the primary way to share data between components in React. However, when you need to pass data through many component layers, it becomes cumbersome and hard to maintain. This is called **prop drilling**.

### Example: Prop Drilling Problem

```jsx
function App() {
  const theme = 'dark';
  return <Layout theme={theme} />;
}

function Layout({ theme }) {
  // Layout doesn't use theme, but must pass it down
  return <Header theme={theme} />;
}

function Header({ theme }) {
  // Header doesn't use theme, but must pass it down
  return <Navigation theme={theme} />;
}

function Navigation({ theme }) {
  // Navigation doesn't use theme, but must pass it down
  return <Button theme={theme} />;
}

function Button({ theme }) {
  // Finally, Button uses the theme
  return <button className={theme === 'dark' ? 'dark' : 'light'}>Click</button>;
}
```

**Problems with prop drilling:**

- **Verbose code**: Props passed through many unused components
- **Hard to maintain**: Adding new props requires updating many components
- **Tight coupling**: Intermediate components must know about props they don't use
- **Error-prone**: Easy to forget to pass props or make typos

**Solution**: Use Context to provide data directly to components that need it, bypassing intermediate components.

---

## Context: An Alternative to Passing Props

Context lets a parent component provide data to any component in the tree below it—no matter how deep—without passing it through props at every level.

**When Context is useful:**

- **Deep nesting**: Passing data through 3+ component layers
- **Many components need it**: Multiple components at different levels need the same data
- **Global-like data**: Theme, user preferences, authentication status

**When NOT to use Context:**

- **Simple parent-child**: Just pass props
- **Few components**: If only 1-2 levels deep, props are simpler
- **Frequently changing data**: Context causes re-renders of all consumers

---

## Step-by-Step: Using Context

### Step 1: Create the Context

Use `createContext` to create a new context. You can provide a default value:

```jsx
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

The default value (`1` in this case) is used when a component consumes the context but there's no Provider above it in the tree.

### Step 2: Provide the Context

Wrap the part of your component tree that needs access to the context with a Provider:

```jsx
import { LevelContext } from './LevelContext.js';

function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

**Key points:**

- The `value` prop is what gets passed to consuming components
- Any descendant component can access this value
- You can have multiple Providers in your tree (inner Providers override outer ones)

### Step 3: Consume the Context

Use the `useContext` Hook to read the context value in any child component:

```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

function Heading({ children }) {
  const level = useContext(LevelContext);
  const Tag = `h${level}`;
  return <Tag>{children}</Tag>;
}
```

**How it works:**

1. `useContext(LevelContext)` looks for the nearest `LevelContext.Provider` above it
2. It returns the `value` from that Provider
3. If no Provider is found, it returns the default value from `createContext`

---

## Complete Example: Nested Headings

Here's a complete example showing how Context eliminates prop drilling:

**Before (with prop drilling):**

```jsx
function App() {
  return (
    <Section level={1}>
      <Heading level={1}>Title</Heading>
      <Section level={2}>
        <Heading level={2}>Heading</Heading>
        <Section level={3}>
          <Heading level={3}>Sub-heading</Heading>
        </Section>
      </Section>
    </Section>
  );
}

function Section({ level, children }) {
  return <section>{children}</section>;
}

function Heading({ level, children }) {
  const Tag = `h${level}`;
  return <Tag>{children}</Tag>;
}
```

**After (with Context):**

```jsx
// LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(1);

// App.js
import { LevelContext } from './LevelContext.js';

function App() {
  return (
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
        </Section>
      </Section>
    </Section>
  );
}

function Section({ level, children }) {
  return (
    <section>
      <LevelContext.Provider value={level}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}

function Heading({ children }) {
  const level = useContext(LevelContext);
  const Tag = `h${level}`;
  return <Tag>{children}</Tag>;
}
```

**Benefits:**

- `Heading` doesn't need a `level` prop anymore
- `Section` doesn't need to pass `level` to `Heading`
- Each `Section` provides its own `level` value to its children
- Adding more nested sections automatically works

---

## Common Use Cases for Context

### 1. Theming

Provide theme information (colors, styles) to components throughout your app:

```jsx
const ThemeContext = createContext('light');

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function Button() {
  const { theme } = useContext(ThemeContext);
  return <button className={theme}>Click me</button>;
}
```

### 2. Authentication

Share current user and authentication status:

```jsx
const UserContext = createContext(null);

function App() {
  const [user, setUser] = useState(null);
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <NavBar />
      <MainContent />
    </UserContext.Provider>
  );
}

function NavBar() {
  const { user } = useContext(UserContext);
  return user ? <LogoutButton /> : <LoginButton />;
}
```

### 3. Localization (i18n)

Provide language and locale information:

```jsx
const LocaleContext = createContext('en');

function App() {
  const [locale, setLocale] = useState('en');
  return (
    <LocaleContext.Provider value={locale}>
      <Header />
      <Content />
    </LocaleContext.Provider>
  );
}

function Content() {
  const locale = useContext(LocaleContext);
  const messages = translations[locale];
  return <p>{messages.welcome}</p>;
}
```

---

## Alternatives to Context

Context isn't always the right solution. Consider these alternatives:

### 1. Passing Props (Simplest)

For simple parent-child relationships, props are often better:

```jsx
// ✅ Good: Just pass props for simple cases
function Parent() {
  const name = 'Alice';
  return <Child name={name} />;
}

function Child({ name }) {
  return <p>{name}</p>;
}
```

### 2. Component Composition

Sometimes restructuring components eliminates the need for Context:

```jsx
// Instead of passing many props
function App() {
  return <Layout user={user} theme={theme} locale={locale} />;
}

// Compose components to avoid prop drilling
function App() {
  return (
    <ThemeProvider>
      <UserProvider>
        <LocaleProvider>
          <Layout />
        </LocaleProvider>
      </UserProvider>
    </ThemeProvider>
  );
}
```

### 3. State Management Libraries

For complex state management, consider Redux, Zustand, or Jotai:

```jsx
// For very complex global state
import { create } from 'zustand';

const useStore = create((set) => ({
  user: null,
  setUser: (user) => set({ user }),
}));
```

**When to choose each:**

- **Props**: 1-2 levels, simple data
- **Context**: 3+ levels, many consumers, relatively static data
- **State library**: Complex state logic, many updates, performance critical

---

## Pitfalls

> **⚠️ PITFALL: Context Value Changes Cause All Consumers to Re-render**
>
> Every time the context value changes, all components consuming that context will re-render, even if they only use part of the value.
>
> ```jsx
> // ❌ PROBLEM: New object created every render
> function ThemeProvider({ children }) {
>   const [theme, setTheme] = useState('light');
>   const [user, setUser] = useState(null);
>   
>   return (
>     <ThemeContext.Provider value={{ theme, setTheme, user, setUser }}>
>       {children}
>     </ThemeContext.Provider>
>   );
> }
> ```
>
> Every render creates a new object `{ theme, setTheme, user, setUser }`, causing all consumers to re-render even if only `theme` changed.
>
> ```jsx
> // ✅ SOLUTION: Split contexts or memoize
> // Option 1: Split contexts
> <ThemeContext.Provider value={theme}>
>   <UserContext.Provider value={user}>
>     {children}
>   </UserContext.Provider>
> </ThemeContext.Provider>
>
> // Option 2: Memoize the value
> const value = useMemo(
>   () => ({ theme, setTheme, user, setUser }),
>   [theme, user]
> );
> <ThemeContext.Provider value={value}>
>   {children}
> </ThemeContext.Provider>
> ```
>
> **Why this matters**: Unnecessary re-renders hurt performance, especially with many components or expensive renders.

---

> **⚠️ PITFALL: Using Context for Frequently Changing Values**
>
> Context is optimized for values that don't change often. Using it for rapidly changing values (like scroll position or mouse coordinates) causes performance issues.
>
> ```jsx
> // ❌ BAD: Rapidly changing value
> function ScrollProvider({ children }) {
>   const [scrollY, setScrollY] = useState(0);
>   
>   useEffect(() => {
>     const handleScroll = () => setScrollY(window.scrollY);
>     window.addEventListener('scroll', handleScroll);
>     return () => window.removeEventListener('scroll', handleScroll);
>   }, []);
>   
>   return (
>     <ScrollContext.Provider value={scrollY}>
>       {children}
>     </ScrollContext.Provider>
>   );
> }
> ```
>
> Every scroll event causes all consumers to re-render, which is very expensive.
>
> ```jsx
> // ✅ BETTER: Use props, refs, or events for rapid changes
> // Option 1: Pass as props to specific components
> function App() {
>   const [scrollY, setScrollY] = useState(0);
>   return <Header scrollY={scrollY} />;
> }
>
> // Option 2: Use a custom event system or library
> // Option 3: Use refs for non-reactive values
> ```
>
> **Guideline**: Use Context for relatively static data (theme, user, locale). Use props/events/state libraries for frequently changing values.

---

> **⚠️ PITFALL: Creating Context Inside Components**
>
> Creating context inside a component function causes it to be recreated on every render, breaking React's ability to track context providers.
>
> ```jsx
> // ❌ NEVER DO THIS
> function App() {
>   const ThemeContext = createContext('light'); // Recreated every render!
>   return (
>     <ThemeContext.Provider value="dark">
>       <Child />
>     </ThemeContext.Provider>
>   );
> }
> ```
>
> React can't properly track this context because it's a new object every render.
>
> ```jsx
> // ✅ DO THIS: Create context outside component
> const ThemeContext = createContext('light');
>
> function App() {
>   return (
>     <ThemeContext.Provider value="dark">
>       <Child />
>     </ThemeContext.Provider>
>   );
> }
> ```
>
> **Rule**: Always create contexts at the module level (outside components) so they're stable across renders.

---

> **⚠️ PITFALL: Overusing Context**
>
> Not every piece of data needs to be in Context. Overusing Context makes components harder to understand and can hurt performance.
>
> ```jsx
> // ❌ OVERUSE: Context for everything
> const NameContext = createContext('');
> const EmailContext = createContext('');
> const PhoneContext = createContext('');
> const AddressContext = createContext('');
> // ... 20 more contexts
>
> function Form() {
>   return (
>     <NameContext.Provider value={name}>
>       <EmailContext.Provider value={email}>
>         <PhoneContext.Provider value={phone}>
>           {/* ... many nested providers */}
>         </PhoneContext.Provider>
>       </EmailContext.Provider>
>     </NameContext.Provider>
>   );
> }
> ```
>
> This is overcomplicated and hard to maintain.
>
> ```jsx
> // ✅ BETTER: Group related data, use props when simple
> const FormContext = createContext(null);
>
> function FormProvider({ children }) {
>   const formData = { name, email, phone, address };
>   return (
>     <FormContext.Provider value={formData}>
>       {children}
>     </FormContext.Provider>
>   );
> }
>
> // Or even simpler: just use props for form fields
> function FormField({ label, value, onChange }) {
>   return (
>     <label>
>       {label}
>       <input value={value} onChange={onChange} />
>     </label>
>   );
> }
> ```
>
> **Guideline**: Use Context sparingly. Group related data. Prefer props for simple cases.

---

## Pattern: Custom Hook for Context

A common pattern is to create a custom hook that consumes the context and throws a helpful error if used outside a Provider:

```jsx
import { createContext, useContext } from 'react';

const ThemeContext = createContext(null);

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (context === null) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}
```

**Benefits:**

- Clear error messages if used incorrectly
- Better developer experience
- Can add validation or transformation logic
- Hides implementation details

**Usage:**

```jsx
function Button() {
  const { theme, setTheme } = useTheme(); // Clean API
  return (
    <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
      Current theme: {theme}
    </button>
  );
}
```

---

## Interview Questions to Consider

### Conceptual Understanding

1. **What is prop drilling, and why is it a problem?**
   - Prop drilling is passing props through multiple component layers where intermediate components don't use the props. It makes code verbose, hard to maintain, and error-prone. Context solves this by allowing components to access data directly without prop passing.

2. **What is React Context, and when should you use it?**
   - Context is an API that allows passing data through the component tree without prop drilling. Use it when you need to pass data through 3+ levels, when many components need the same data, or for global-like data (theme, auth, locale). Don't use it for simple parent-child relationships or frequently changing values.

3. **How does Context work under the hood?**
   - Context creates a provider-consumer relationship. The Provider stores a value, and any descendant component can consume it using `useContext`. React maintains a tree of context providers and finds the nearest one when consuming.

4. **What is the difference between Context and props?**
   - Props are explicitly passed from parent to child. Context allows any descendant to access data without explicit prop passing. Props are better for simple cases; Context is better for deep nesting or global data.

### Practical Application

5. **Implement a Context for theme management:**
   ```jsx
   import { createContext, useContext, useState } from 'react';
   
   const ThemeContext = createContext(null);
   
   export function ThemeProvider({ children }) {
     const [theme, setTheme] = useState('light');
     return (
       <ThemeContext.Provider value={{ theme, setTheme }}>
         {children}
       </ThemeContext.Provider>
     );
   }
   
   export function useTheme() {
     const context = useContext(ThemeContext);
     if (!context) {
       throw new Error('useTheme must be used within ThemeProvider');
     }
     return context;
   }
   ```

6. **What's wrong with this Context implementation?**
   ```jsx
   function App() {
     const ThemeContext = createContext('light');
     const [theme, setTheme] = useState('light');
     return (
       <ThemeContext.Provider value={theme}>
         <Child />
       </ThemeContext.Provider>
     );
   }
   ```
   - The context is created inside the component, so it's recreated on every render. Move `createContext` outside the component to make it stable.

7. **How would you optimize this Context to prevent unnecessary re-renders?**
   ```jsx
   function App() {
     const [theme, setTheme] = useState('light');
     const [user, setUser] = useState(null);
     return (
       <AppContext.Provider value={{ theme, setTheme, user, setUser }}>
         <Child />
       </AppContext.Provider>
     );
   }
   ```
   - Split into separate contexts (`ThemeContext` and `UserContext`) or memoize the value object with `useMemo` to prevent creating a new object every render.

### Advanced Thinking

8. **Can you have multiple Context Providers in the same component tree?**
   - Yes, and inner Providers override outer ones. This is useful for nested sections that need different values:
   ```jsx
   <ThemeProvider value="dark">
     <Section>
       <ThemeProvider value="light">
         <Button /> {/* Uses "light" theme */}
       </ThemeProvider>
     </Section>
   </ThemeProvider>
   ```

9. **How does Context performance compare to props?**
   - Props are generally faster because React can optimize prop passing. Context can cause unnecessary re-renders if not used carefully (e.g., creating new objects in Provider value). However, for deeply nested data, Context avoids the overhead of passing props through unused components.

10. **When would you choose a state management library (Redux, Zustand) over Context?**
    - Choose a state management library when you need:
      - Complex state logic (middleware, time-travel debugging)
      - Better performance with fine-grained subscriptions
      - DevTools and debugging features
      - Very frequent updates or large state
    - Context is sufficient for simpler cases like theming, auth, or passing data a few levels deep.

---

## Summary

React Context provides a powerful way to share data across the component tree without prop drilling:

- **Solves prop drilling**: Components can access data without props passing through intermediate components
- **Simple API**: Three steps—create context, provide value, consume with `useContext`
- **Use wisely**: Best for 3+ levels deep, many consumers, or global-like data (theme, auth, locale)
- **Performance matters**: Avoid creating new objects in Provider values; split contexts if needed
- **Patterns**: Custom hooks for context consumption improve developer experience and error handling

Context is a great tool when used appropriately. For simple cases, props are better. For complex state management, consider state management libraries. The key is choosing the right tool for each situation.


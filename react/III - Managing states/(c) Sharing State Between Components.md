# Sharing State Between Components

## Overview
Sometimes, you want the state of two components to always change together. To do it, remove state from both of them, move it to their closest common parent, and then pass it down to them via props. This is known as "lifting state up."

## What You Will Learn
- Sharing state between components via lifting it up
- Understanding controlled vs. uncontrolled component patterns

---

## The Problem: Independent State

When two components need to stay in sync, keeping state in each component separately doesn't work.

### Example: Accordion Panels

```jsx
function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);

  return (
    <section>
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>Show</button>
      )}
    </section>
  );
}

function Accordion() {
  return (
    <>
      <Panel title="About">...</Panel>
      <Panel title="Etymology">...</Panel>
    </>
  );
}
```

**Problem:** Both panels can be open simultaneously. If you want only one open at a time, you can't coordinate between independent components.

---

## The Solution: Lifting State Up

**"Lifting state up"** means moving state from child components to their common parent.

### Three Steps to Lift State Up

#### Step 1: Remove State from Children

Remove the state variable from the child components:

```jsx
function Panel({ title, children }) {
  // ❌ Remove this:
  // const [isActive, setIsActive] = useState(false);

  return (
    <section>
      <h3>{title}</h3>
      {/* Need isActive from somewhere... */}
    </section>
  );
}
```

#### Step 2: Pass Hardcoded Data from Parent

Add props to receive data and callbacks from the parent:

```jsx
function Panel({ title, children, isActive, onShow }) {
  return (
    <section>
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>Show</button>
      )}
    </section>
  );
}

function Accordion() {
  return (
    <>
      <Panel title="About" isActive={true} onShow={() => {}}>
        ...
      </Panel>
      <Panel title="Etymology" isActive={false} onShow={() => {}}>
        ...
      </Panel>
    </>
  );
}
```

Start with hardcoded values to establish the structure.

#### Step 3: Add State to the Common Parent

Now add state to the parent and wire it up:

```jsx
function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);

  return (
    <>
      <Panel
        title="About"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        ...
      </Panel>
      <Panel
        title="Etymology"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        ...
      </Panel>
    </>
  );
}
```

**Now the parent controls which panel is active!**

---

## Before and After Comparison

### ❌ Before: Uncoordinated State

```jsx
function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  // Each panel manages its own state
  // Can't coordinate between panels
}
```

**Problems:**
- Both panels can be open
- Can't enforce "only one open" rule
- No coordination possible

### ✅ After: Lifted State

```jsx
function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  // Parent manages state
  // Passes down props to coordinate children
}

function Panel({ isActive, onShow }) {
  // Panel controlled by parent
  // No internal state needed
}
```

**Benefits:**
- Only one panel can be active
- Parent coordinates behavior
- Easy to enforce rules

---

## Controlled vs. Uncontrolled Components

### Uncontrolled Components

**Components with local state** are "uncontrolled."

```jsx
function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  // This component controls itself
}
```

**Characteristics:**
- Manage their own internal state
- Easier to use (less configuration needed)
- Less flexible (parent can't control them)
- Good when you don't need coordination

### Controlled Components

**Components driven by props** are "controlled."

```jsx
function Panel({ title, children, isActive, onShow }) {
  // Parent controls this component via props
  // No internal state for isActive
}
```

**Characteristics:**
- Behavior driven by props from parent
- Maximally flexible (parent has full control)
- Require parent configuration
- Good when you need coordination or complex logic

### Terminology Note

"Controlled" and "uncontrolled" are informal terms, not technical React terms. A component can have both controlled props (passed by parent) and uncontrolled state (managed internally).

---

## Single Source of Truth

For each unique piece of state, choose which component "owns" it.

**Key principle:** Each piece of state lives in **one specific component**.

Rather than duplicating shared state, **lift it up** to the common parent and pass it down.

### Example: Todo Selection

**❌ Bad: Duplicated State**
```jsx
// Both store the same information
const [todos, setTodos] = useState([...]);
const [selectedTodo, setSelectedTodo] = useState({ id: 1, ... });
```

**✅ Good: Single Source**
```jsx
const [todos, setTodos] = useState([...]);
const [selectedId, setSelectedId] = useState(1);

// Derive selected todo
const selectedTodo = todos.find(t => t.id === selectedId);
```

---

## Complete Example: Accordion

```jsx
import { useState } from 'react';

function Panel({ title, children, isActive, onShow }) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>Show</button>
      )}
    </section>
  );
}

function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);

  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel
        title="About"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        With a population of about 2 million, Almaty is Kazakhstan's largest city.
      </Panel>
      <Panel
        title="Etymology"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        The name comes from алма, the Kazakh word for "apple".
      </Panel>
    </>
  );
}

export default Accordion;
```

**How it works:**
1. Parent (`Accordion`) stores `activeIndex`
2. Parent passes `isActive` prop to each panel (computed from `activeIndex`)
3. Parent passes `onShow` callback to each panel
4. When user clicks "Show", callback updates parent's state
5. Parent re-renders with new `activeIndex`
6. Panels receive updated `isActive` props

---

## Data Flow

### One-Way Data Flow

Data flows **down** the component tree:

```
Accordion (state: activeIndex = 0)
    ↓ props
Panel (isActive: true, onShow: function)
    ↓ props
Panel (isActive: false, onShow: function)
```

**Parent controls children** through props.

### Events Flow Up

User events trigger callbacks that flow **up**:

```
Panel: User clicks button
    ↓ calls
onShow callback
    ↓ executes in
Accordion: setActiveIndex(1)
    ↓ triggers
Re-render with new state
```

---

## Practical Patterns

### Pattern 1: Radio Buttons / Tabs

```jsx
function TabPanel({ children, isActive }) {
  return isActive ? <div>{children}</div> : null;
}

function Tabs() {
  const [activeTab, setActiveTab] = useState('home');

  return (
    <>
      <button onClick={() => setActiveTab('home')}>Home</button>
      <button onClick={() => setActiveTab('profile')}>Profile</button>
      <button onClick={() => setActiveTab('settings')}>Settings</button>

      <TabPanel isActive={activeTab === 'home'}>Home content</TabPanel>
      <TabPanel isActive={activeTab === 'profile'}>Profile content</TabPanel>
      <TabPanel isActive={activeTab === 'settings'}>Settings content</TabPanel>
    </>
  );
}
```

### Pattern 2: Form Fields

```jsx
function Input({ value, onChange }) {
  return <input value={value} onChange={onChange} />;
}

function Form() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  return (
    <form>
      <Input value={email} onChange={e => setEmail(e.target.value)} />
      <Input value={password} onChange={e => setPassword(e.target.value)} />
    </form>
  );
}
```

### Pattern 3: Filtered List

```jsx
function SearchBar({ query, onQueryChange }) {
  return <input value={query} onChange={e => onQueryChange(e.target.value)} />;
}

function ProductList({ products, query }) {
  const filtered = products.filter(p =>
    p.name.toLowerCase().includes(query.toLowerCase())
  );

  return filtered.map(product => <div key={product.id}>{product.name}</div>);
}

function ShopPage() {
  const [query, setQuery] = useState('');
  const [products] = useState([...]);

  return (
    <>
      <SearchBar query={query} onQueryChange={setQuery} />
      <ProductList products={products} query={query} />
    </>
  );
}
```

---

## When to Lift State Up

Lift state when:
- **Two or more components need the same data** and should stay in sync
- **One component's action affects another component**
- **You need to enforce rules** across multiple components (like "only one active")
- **Parent needs to orchestrate** interactions between children

Keep state local when:
- **Only one component needs it**
- **No coordination required** with other components
- **Simpler mental model** if kept local

> **⚠️ PITFALL: STATE LOCATION CHANGES**
>
> **Don't worry about getting state location perfect initially.**
>
> As you build your app, you'll often:
> - Start with state in one component
> - Realize it needs to be shared
> - Lift it up to a parent
> - Later lift it even higher
> - Sometimes push it back down
>
> This refactoring is **normal and expected**. Start with what makes sense, then refactor as requirements become clearer.

---

## Interview Questions to Consider

1. **Q: What does "lifting state up" mean?**
   - A: Moving state from child components to their closest common parent, then passing it down as props. This allows the parent to coordinate behavior between children.

2. **Q: What are the three steps to lift state up?**
   - A: (1) Remove state from children, (2) Pass hardcoded data from parent, (3) Add state to parent and wire it up.

3. **Q: What's the difference between controlled and uncontrolled components?**
   - A: Uncontrolled components manage their own state internally. Controlled components are driven by props from their parent.

4. **Q: When should you lift state up?**
   - A: When two or more components need to share and synchronize the same data, or when a parent needs to coordinate behavior between children.

5. **Q: What is "single source of truth"?**
   - A: Each piece of state should live in exactly one component (usually the lowest common ancestor). Other components derive or receive that data through props.

6. **Q: How does data flow in React?**
   - A: Data flows down (parent → child via props). Events flow up (child → parent via callbacks).

7. **Q: Is it bad to change where state lives during development?**
   - A: No! It's normal and expected. Start with local state, then lift as needed when requirements become clear.

8. **Q: Can a component be both controlled and uncontrolled?**
   - A: Yes! A component can have some props controlled by parent and some internal state it manages itself.

9. **Q: Why not just duplicate state in multiple components?**
   - A: Duplicating state leads to synchronization issues. Components can get out of sync, causing bugs. Single source of truth prevents this.

---

## Key Takeaways

- **Lifting state up** = moving state to a common parent to coordinate children
- **Three steps:** Remove from children → Pass hardcoded props → Add state to parent
- **Controlled components** are driven by props (flexible, require configuration)
- **Uncontrolled components** manage their own state (easier to use, less flexible)
- **Single source of truth:** Each piece of state lives in one specific component
- **Data flows down** through props; **events flow up** through callbacks
- Parent passes both **data props** (`isActive`) and **callback props** (`onShow`)
- Refactoring state location during development is **normal and expected**
- When multiple components need the same data → lift to common ancestor
- When only one component needs data → keep it local
- Think of controlled components like form inputs: parent controls their value
- State location isn't set in stone—move it as requirements evolve


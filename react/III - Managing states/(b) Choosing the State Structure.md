# Choosing the State Structure

## Overview
Structuring state well can make a difference between a component that is pleasant to modify and debug, and one that is a constant source of bugs. Here are some principles you should consider when structuring state.

## What You Will Learn
- When to use a single vs multiple state variables
- What to avoid when organizing state
- How to fix common issues with the state structure

---

## Principles for Structuring State

The goal is to make state **"as simple as it can be—but no simpler."** Well-structured state:
- Makes updates easy without introducing bugs
- Prevents impossible or contradictory states
- Reduces duplication and redundancy
- Makes code easier to read and maintain

---

## Principle 1: Group Related State

If two or more state variables **always update together**, consider combining them into a single state variable.

### ❌ Bad: Separate Variables
```jsx
const [x, setX] = useState(0);
const [y, setY] = useState(0);

// Easy to forget to update both!
setX(100); // Oops, forgot to update y
```

### ✅ Good: Grouped Together
```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });

// Always update together
setPosition({ x: 100, y: 100 });
```

### When to Group

**Group when:**
- Variables always change together
- They represent parts of the same concept (like coordinates, dimensions, ranges)
- You find yourself updating them in the same event handlers

**Example use cases:**
- Mouse position: `{ x, y }`
- Form fields that are submitted together
- Selected range: `{ start, end }`
- Dimensions: `{ width, height }`

> **⚠️ PITFALL: REMEMBER TO COPY OTHER FIELDS**
>
> When updating an object in state, you must copy all other fields:
> ```jsx
> // ❌ Wrong: Loses the y coordinate
> setPosition({ x: 100 });
>
> // ✅ Correct: Preserves y
> setPosition({ ...position, x: 100 });
> ```

---

## Principle 2: Avoid Contradictions in State

When state is structured so that multiple pieces can contradict each other, you leave room for bugs.

### ❌ Bad: Contradictory Booleans
```jsx
const [isSending, setIsSending] = useState(false);
const [isSent, setIsSent] = useState(false);

// Can create impossible states:
// - Both true? Can't be sending AND sent
// - Both false? What state is that?
```

### ✅ Good: Single Status Variable
```jsx
const [status, setStatus] = useState('typing');
// Possible values: 'typing' | 'sending' | 'sent'
// Only one status at a time—no contradictions!
```

### Why This Matters

**With multiple booleans:**
- Can represent invalid states
- Hard to ensure consistency
- More state variables to manage

**With single status:**
- Impossible to be in invalid state
- Clear state transitions
- Easier to reason about

### Example: Form Status
```jsx
function Form() {
  const [status, setStatus] = useState('typing');

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('sending');

    try {
      await sendMessage();
      setStatus('sent');
    } catch (err) {
      setStatus('typing');
    }
  }

  const isSending = status === 'sending';
  const isSent = status === 'sent';

  if (isSent) {
    return <h1>Thanks for feedback!</h1>;
  }

  return (
    <form onSubmit={handleSubmit}>
      <textarea disabled={isSending} />
      <button disabled={isSending}>Send</button>
    </form>
  );
}
```

---

## Principle 3: Avoid Redundant State

If you can calculate some information from props or existing state during rendering, **don't put that information into state**.

### ❌ Bad: Storing Derived Values
```jsx
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState(''); // ❌ Redundant!

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName); // Must update manually
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value); // Must update manually
  }
}
```

### ✅ Good: Calculate During Render
```jsx
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Calculate, don't store
  const fullName = firstName + ' ' + lastName;

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    // fullName updates automatically!
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    // fullName updates automatically!
  }
}
```

### Benefits of Calculating

- **No sync issues:** Derived value always matches source
- **Less code:** No manual updates needed
- **Fewer bugs:** Can't forget to update derived value
- **Better performance:** React is already fast at re-rendering

### Don't Mirror Props in State

> **⚠️ PITFALL: MIRRORING PROPS**
>
> **Avoid "mirroring" props into state:**
> ```jsx
> function Message({ messageColor }) {
>   const [color, setColor] = useState(messageColor); // ❌ Mirrors prop
>   // Problem: If messageColor prop changes, color state doesn't update!
> }
> ```
>
> **Use the prop directly:**
> ```jsx
> function Message({ messageColor }) {
>   const color = messageColor; // ✅ Just use the prop
> }
> ```
>
> **Only mirror when you want to ignore updates:**
> ```jsx
> function Message({ initialColor }) {
>   // ✅ OK: "initial" prefix shows it's only used once
>   const [color, setColor] = useState(initialColor);
> }
> ```

---

## Principle 4: Avoid Duplication in State

Never store the same data in multiple state variables or nested objects.

### ❌ Bad: Duplicating Data
```jsx
const [items, setItems] = useState([
  { id: 0, title: 'Pretzels', description: 'Crunchy' },
  { id: 1, title: 'Seaweed', description: 'Salty' }
]);
const [selectedItem, setSelectedItem] = useState(items[0]); // ❌ Duplicate!

// Problem: If you edit items, selectedItem won't update
function handleEdit(id, newTitle) {
  setItems(items.map(item =>
    item.id === id ? { ...item, title: newTitle } : item
  ));
  // selectedItem still has old title!
}
```

### ✅ Good: Store ID, Not Full Object
```jsx
const [items, setItems] = useState([
  { id: 0, title: 'Pretzels', description: 'Crunchy' },
  { id: 1, title: 'Seaweed', description: 'Salty' }
]);
const [selectedId, setSelectedId] = useState(0); // ✅ Just the ID

// Derive the full object
const selectedItem = items.find(item => item.id === selectedId);

// Now edits automatically reflect in selectedItem
function handleEdit(id, newTitle) {
  setItems(items.map(item =>
    item.id === id ? { ...item, title: newTitle } : item
  ));
  // selectedItem updates automatically since we look it up fresh!
}
```

### Why This Matters

**With duplication:**
- Must update multiple places
- Easy to forget to sync
- Data can become inconsistent

**With single source of truth:**
- Update in one place
- Always consistent
- Simpler logic

---

## Principle 5: Avoid Deeply Nested State

Deeply hierarchical state is difficult to update. When possible, structure state in a flat way.

### ❌ Bad: Deeply Nested Structure
```jsx
const [places, setPlaces] = useState({
  0: {
    id: 0,
    title: 'Earth',
    childPlaces: [{
      id: 1,
      title: 'Africa',
      childPlaces: [{
        id: 2,
        title: 'Kenya',
        childPlaces: []
      }]
    }]
  }
});

// Updating deeply nested data is painful
function deletePlace(parentId, childId) {
  // Need to carefully copy every level... 😰
}
```

### ✅ Good: Normalized (Flat) Structure
```jsx
const [places, setPlaces] = useState({
  0: { id: 0, title: 'Earth', childIds: [1] },
  1: { id: 1, title: 'Africa', childIds: [2] },
  2: { id: 2, title: 'Kenya', childIds: [] }
});

// Much easier to update!
function deletePlace(parentId, childId) {
  setPlaces({
    ...places,
    [parentId]: {
      ...places[parentId],
      childIds: places[parentId].childIds.filter(id => id !== childId)
    }
  });
}
```

### Normalization Pattern

**Think like a database:**
- Each object has a unique ID
- Store objects in a lookup table (keyed by ID)
- Reference other objects by ID, not by nesting
- Use arrays of IDs for relationships

**Example structure:**
```jsx
{
  places: {
    0: { id: 0, title: 'Earth', childIds: [1, 2, 3] },
    1: { id: 1, title: 'Africa', childIds: [4, 5] },
    2: { id: 2, title: 'Asia', childIds: [] },
    // ...
  }
}
```

### Benefits of Flat Structure

- **Easier updates:** No deep nesting to navigate
- **Better performance:** Update only what changed
- **Simpler code:** Less spreading and copying
- **Reusable:** Items can be referenced from multiple places

---

## Putting It All Together

### Example: Todo App with Good State Structure

```jsx
function TodoApp() {
  // ✅ Group related state
  const [todos, setTodos] = useState({
    1: { id: 1, text: 'Buy milk', completed: false },
    2: { id: 2, text: 'Walk dog', completed: true }
  });

  // ✅ Single status variable (no contradictions)
  const [filter, setFilter] = useState('all'); // 'all' | 'active' | 'completed'

  // ✅ Store ID, not full object (no duplication)
  const [selectedId, setSelectedId] = useState(null);

  // ✅ Calculate derived values (no redundancy)
  const todoList = Object.values(todos);
  const filteredTodos = todoList.filter(todo => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true;
  });
  const selectedTodo = selectedId ? todos[selectedId] : null;

  function toggleTodo(id) {
    setTodos({
      ...todos,
      [id]: { ...todos[id], completed: !todos[id].completed }
    });
  }

  return (
    <>
      <select value={filter} onChange={e => setFilter(e.target.value)}>
        <option value="all">All</option>
        <option value="active">Active</option>
        <option value="completed">Completed</option>
      </select>

      {filteredTodos.map(todo => (
        <div key={todo.id} onClick={() => setSelectedId(todo.id)}>
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => toggleTodo(todo.id)}
          />
          {todo.text}
        </div>
      ))}

      {selectedTodo && <p>Selected: {selectedTodo.text}</p>}
    </>
  );
}
```

---

## Interview Questions to Consider

1. **Q: When should you group multiple state variables into one object?**
   - A: When they always update together or represent parts of the same concept (like x/y coordinates).

2. **Q: What's wrong with having separate `isSending` and `isSent` boolean states?**
   - A: They can create contradictory states (both true, or unclear what both false means). Use a single status variable instead.

3. **Q: Should you store calculated values in state?**
   - A: No, calculate them during render. Storing derived values creates redundancy and sync issues.

4. **Q: What does "don't mirror props in state" mean?**
   - A: Don't copy prop values into state unless you explicitly want to ignore future prop updates. Use props directly instead.

5. **Q: Why should you store an ID instead of a full object when selecting from a list?**
   - A: Storing the full object duplicates data. If the original changes, the copy won't update. Storing the ID and looking it up ensures consistency.

6. **Q: What is "normalized" or "flat" state structure?**
   - A: Like database normalization: store objects in a lookup table (keyed by ID) and reference by ID rather than nesting deeply.

7. **Q: What problems does deeply nested state cause?**
   - A: It's hard to update (requires deep copying), difficult to maintain, and hurts performance. Flat structures are much easier.

8. **Q: When is it OK to duplicate data in state?**
   - A: Generally avoid it. Only duplicate if the data truly needs to be independent and won't need to stay in sync.

---

## Key Takeaways

### The Five Principles

1. **Group related state** that always updates together
2. **Avoid contradictions** (use single status instead of multiple booleans)
3. **Avoid redundant state** (calculate derived values during render)
4. **Avoid duplication** (store IDs, not full objects; single source of truth)
5. **Avoid deeply nested state** (normalize like a database)

### Remember

- State should be **as simple as possible, but no simpler**
- When updating objects, **copy all other fields** with spread syntax
- **Don't mirror props** in state (use props directly)
- Only mirror props prefixed with `initial` or `default` when intentionally ignoring updates
- **Calculate, don't duplicate**—derive values during render
- Think of flat state like a **database**: lookup tables with IDs
- Good state structure makes bugs **impossible**, not just unlikely

### Quick Checklist

Before adding state, ask:
- ✅ Does this always update with another piece of state? → Group them
- ✅ Can this create contradictory states? → Use single status variable
- ✅ Can I calculate this from existing state/props? → Don't store it
- ✅ Am I duplicating this data? → Store ID instead
- ✅ Is this deeply nested? → Flatten the structure


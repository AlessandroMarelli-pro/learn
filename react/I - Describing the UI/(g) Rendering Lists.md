# Rendering Lists

## Overview
You often need to display multiple similar components from a collection of data. In React, you can use JavaScript array methods like `filter()` and `map()` to transform arrays of data into arrays of components.

## What You Will Learn
- How to render components from an array using JavaScript's `map()`
- How to render only specific components using JavaScript's `filter()`
- When and why to use React keys

---

## Rendering Data from Arrays

Instead of writing repetitive markup, move your data into an array and use `map()` to transform it into components.

### Basic Pattern

**Step 1: Store data in an array**
```jsx
const people = [
  { id: 0, name: 'Creola Katherine Johnson', profession: 'mathematician' },
  { id: 1, name: 'Mario José Molina-Pasquel Henríquez', profession: 'chemist' },
  { id: 2, name: 'Mohammad Abdus Salam', profession: 'physicist' },
];
```

**Step 2: Use `map()` to transform array items into JSX**
```jsx
const listItems = people.map(person =>
  <li key={person.id}>
    <p><b>{person.name}:</b> {person.profession}</p>
  </li>
);
```

**Step 3: Return the result**
```jsx
return <ul>{listItems}</ul>;
```

### Complete Example
```jsx
function PeopleList() {
  const people = [
    { id: 0, name: 'Creola Katherine Johnson', profession: 'mathematician' },
    { id: 1, name: 'Mario José Molina-Pasquel Henríquez', profession: 'chemist' },
    { id: 2, name: 'Mohammad Abdus Salam', profession: 'physicist' },
  ];

  const listItems = people.map(person =>
    <li key={person.id}>
      <p><b>{person.name}:</b> {person.profession}</p>
    </li>
  );

  return <ul>{listItems}</ul>;
}
```

---

## Filtering Arrays

Use JavaScript's `filter()` method to show only specific items based on a condition.

**`filter()` returns a new array of only those items that passed the test.**

### Example: Filter by Profession
```jsx
function ChemistList() {
  const people = [
    { id: 0, name: 'Creola Katherine Johnson', profession: 'mathematician' },
    { id: 1, name: 'Mario José Molina-Pasquel Henríquez', profession: 'chemist' },
    { id: 2, name: 'Mohammad Abdus Salam', profession: 'physicist' },
    { id: 3, name: 'Percy Lavon Julian', profession: 'chemist' },
  ];

  // Filter to get only chemists
  const chemists = people.filter(person =>
    person.profession === 'chemist'
  );

  // Map to JSX
  const listItems = chemists.map(person =>
    <li key={person.id}>
      <p><b>{person.name}:</b> {person.profession}</p>
    </li>
  );

  return <ul>{listItems}</ul>;
}
```

### Combining Filter and Map
```jsx
const chemistItems = people
  .filter(person => person.profession === 'chemist')
  .map(person =>
    <li key={person.id}>
      <p><b>{person.name}</b></p>
    </li>
  );
```

---

## Understanding Keys

### What Are Keys?

**Keys** are strings or numbers that uniquely identify an item among its siblings in an array. They tell React which array item each component corresponds to, enabling React to match them correctly across renders.

**Analogy:** Think of files on your desktop. Without filenames, you'd identify them by position: "the first file," "the second file." If you delete one, it becomes confusing. Filenames (like keys) provide stable identity regardless of position.

### Where to Get Keys

| Source | How to Get Keys | Example |
|--------|-----------------|---------|
| **Database data** | Use database IDs | `key={user.id}` |
| **Locally generated** | Use incrementing counter | `let nextId = 0; nextId++` |
| **Locally generated** | Use UUID library | `key={crypto.randomUUID()}` |
| **Locally generated** | Use `uuid` package | `key={uuid()}` |

**Recommendation:** Database IDs are ideal because they're "unique by nature."

---

## Rules for Keys

### Rule 1: Keys Must Be Unique Among Siblings

Keys need to be unique **within the same array**, but can repeat across different arrays.

```jsx
<ul>
  <li key="1">Item 1</li>
  <li key="2">Item 2</li>
  <li key="1">Item 1</li> {/* ❌ Duplicate key! */}
</ul>
```

### Rule 2: Keys Must Not Change

**Keys must be stable across renders.** Don't generate them during rendering.

```jsx
// ❌ Don't do this
{items.map(item => <li key={Math.random()}>{item}</li>)}

// ✅ Use stable identifiers
{items.map(item => <li key={item.id}>{item}</li>)}
```

> **⚠️ PITFALL**
>
> **Do not generate keys on the fly**, e.g., with `key={Math.random()}`. This causes keys to never match up between renders, leading to all components and DOM being recreated every time. This is slow and **will lose user input** inside list items.

### Rule 3: Don't Use Array Index as Key (Usually)

You might be tempted to use the array index:
```jsx
{items.map((item, index) => <li key={index}>{item}</li>)}
```

> **⚠️ PITFALL**
>
> Using array indices as keys **often leads to subtle and confusing bugs**, especially when:
> - Items can be reordered (sorting, drag-and-drop)
> - Items can be added or removed
> - The array can be filtered
>
> **Why?** If an item is deleted, the indices shift, causing React to match the wrong components to the wrong data.

**When index keys are okay:**
- The list never changes (static data)
- Items are never reordered
- Items are never added/removed
- The list doesn't have state or user input

React defaults to using indices if you don't specify keys, but this is risky for dynamic lists.

---

## Why Keys Matter

Keys enable React to:

1. **Uniquely identify items between siblings** in an array
2. **Track items throughout their lifetime**, even when positions change
3. **Preserve component state** correctly during reordering or deletion
4. **Optimize rendering** by matching components correctly across renders

### What Happens Without Proper Keys?

**Without keys or with bad keys (like `Math.random()`):**
- React recreates components unnecessarily
- Component state is lost on every render
- User input disappears
- Performance degrades
- Animations break
- Focus is lost

**With good keys:**
- React knows which components to keep, update, or remove
- Component state persists correctly
- Better performance through efficient reconciliation

---

## Keys Are Not Props

> **⚠️ PITFALL**
>
> Components **do not receive the `key` prop**. It's only used by React as a hint. If your component needs an ID, pass it as a separate prop:
>
> ```jsx
> <Profile key={user.id} userId={user.id} />
> ```

---

## Practical Examples

### Example 1: Rendering Product List
```jsx
function ProductList({ products }) {
  return (
    <div>
      {products.map(product => (
        <ProductCard
          key={product.id}
          name={product.name}
          price={product.price}
        />
      ))}
    </div>
  );
}
```

### Example 2: Filtering and Rendering
```jsx
function TodoList({ todos, filter }) {
  const filteredTodos = todos.filter(todo => {
    if (filter === 'active') return !todo.completed;
    if (filter === 'completed') return todo.completed;
    return true; // 'all'
  });

  return (
    <ul>
      {filteredTodos.map(todo => (
        <li key={todo.id}>
          <input type="checkbox" checked={todo.completed} />
          {todo.text}
        </li>
      ))}
    </ul>
  );
}
```

### Example 3: Using UUID for Dynamic Lists
```jsx
import { v4 as uuidv4 } from 'uuid';

function ShoppingList() {
  const [items, setItems] = useState([]);

  const addItem = (name) => {
    setItems([...items, { id: uuidv4(), name }]);
  };

  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}
```

---

## Interview Questions to Consider

1. **Q: Why do we need keys when rendering lists in React?**
   - A: Keys help React identify which items have changed, been added, or removed. They enable React to match components correctly across renders, preserve state, and optimize performance.

2. **Q: What makes a good key?**
   - A: A good key is stable (doesn't change between renders), unique among siblings, and comes from the data itself (like a database ID) rather than being generated or positional.

3. **Q: Why shouldn't you use array index as a key?**
   - A: Array indices can cause bugs when items are reordered, added, or removed because the index of an item changes, causing React to match components incorrectly and lose state.

4. **Q: When is it okay to use array index as a key?**
   - A: Only when the list is static (never changes), items are never reordered, and there's no component state or user input to preserve.

5. **Q: What happens if you use `Math.random()` as a key?**
   - A: React will recreate all components and DOM nodes on every render because keys never match. This loses user input, breaks state, and kills performance.

6. **Q: Can a component access its own `key` prop?**
   - A: No, React uses `key` internally but doesn't pass it to the component. If you need the ID in your component, pass it as a separate prop.

7. **Q: Do keys need to be globally unique?**
   - A: No, keys only need to be unique among siblings in the same array. Different arrays can have items with the same keys.

---

## Key Takeaways

- Use `map()` to transform arrays of data into arrays of components
- Use `filter()` to show only items that meet certain criteria
- **Always provide keys** when rendering lists
- Keys must be **unique among siblings** and **stable across renders**
- Use database IDs or UUIDs for dynamic lists
- **Avoid using array index as key** unless the list is truly static
- **Never generate keys during render** (no `Math.random()`)
- Keys help React identify items, preserve state, and optimize performance
- Keys are not props—pass IDs separately if components need them
- Think of keys like filenames: stable identifiers independent of position

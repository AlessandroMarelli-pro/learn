# Updating Arrays in State

## Overview
Arrays are mutable in JavaScript, but you should treat them as immutable when stored in React state. Just like with objects, when you want to update an array in state, create a new one (or make a copy), and then set state to use the new array.

## What You Will Learn
- How to add, remove, or change items in an array in React state
- How to update an object inside of an array
- How to make array copying less repetitive with Immer

---

## Updating Arrays Without Mutation

In JavaScript, arrays are mutable, but you should treat them as **immutable** in React state.

**Don't do this:**
```jsx
const [items, setItems] = useState([]);
items.push(newItem); // ❌ Mutates the array
```

**Do this instead:**
```jsx
const [items, setItems] = useState([]);
setItems([...items, newItem]); // ✅ Creates new array
```

---

## Array Methods Reference

### Avoid These (Mutating)

| Operation | Mutating Methods | Description |
|-----------|------------------|-------------|
| **Adding** | `push`, `unshift` | Modify array in place |
| **Removing** | `pop`, `shift`, `splice` | Modify array in place |
| **Replacing** | `splice`, `arr[i] = ...` | Modify array in place |
| **Sorting** | `reverse`, `sort` | Modify array in place |

### Prefer These (Non-mutating)

| Operation | Non-mutating Methods | Description |
|-----------|---------------------|-------------|
| **Adding** | `concat`, `[...arr]` | Return new array |
| **Removing** | `filter`, `slice` | Return new array |
| **Replacing** | `map` | Return new array |
| **Sorting** | Copy first, then sort | Create copy, then mutate copy |

---

## Adding to an Array

### Add to End
```jsx
function List() {
  const [artists, setArtists] = useState([]);

  function handleAdd() {
    const newArtist = { id: nextId++, name: 'New Artist' };
    setArtists([...artists, newArtist]); // ✅ Append
  }

  // Or using concat:
  function handleAddConcat() {
    setArtists(artists.concat(newArtist)); // ✅ Also creates new array
  }
}
```

### Add to Beginning
```jsx
function handleAddToStart() {
  const newArtist = { id: nextId++, name: 'New Artist' };
  setArtists([newArtist, ...artists]); // ✅ Prepend
}
```

### Insert at Specific Position
```jsx
function handleInsert(index) {
  const newArtist = { id: nextId++, name: 'New Artist' };
  setArtists([
    ...artists.slice(0, index),  // Everything before index
    newArtist,                    // New item
    ...artists.slice(index)       // Everything from index onward
  ]);
}
```

---

## Removing from an Array

Use `filter` to keep only items that pass a condition:

```jsx
function List() {
  const [artists, setArtists] = useState([
    { id: 0, name: 'Marta Colvin Andrade' },
    { id: 1, name: 'Lamidi Olonade Fakeye' },
    { id: 2, name: 'Louise Nevelson' }
  ]);

  function handleRemove(artistId) {
    setArtists(artists.filter(a => a.id !== artistId));
  }

  return (
    <ul>
      {artists.map(artist => (
        <li key={artist.id}>
          {artist.name}
          <button onClick={() => handleRemove(artist.id)}>Delete</button>
        </li>
      ))}
    </ul>
  );
}
```

**How `filter` works:**
- Creates new array containing only items where the function returns `true`
- `a.id !== artistId` keeps everything except the item being removed

---

## Transforming an Array

Use `map` to create a new array where some or all items are transformed:

```jsx
function ShapeEditor() {
  const [shapes, setShapes] = useState([
    { id: 0, type: 'circle', x: 50, y: 100 },
    { id: 1, type: 'square', x: 150, y: 100 },
    { id: 2, type: 'circle', x: 250, y: 100 }
  ]);

  function handleClick() {
    // Move all circles down by 50px
    setShapes(shapes.map(shape => {
      if (shape.type === 'circle') {
        return { ...shape, y: shape.y + 50 };
      }
      return shape;
    }));
  }
}
```

---

## Replacing Items in an Array

### Replace One Item by Index
```jsx
function Counter() {
  const [counters, setCounters] = useState([0, 0, 0]);

  function handleIncrement(index) {
    setCounters(counters.map((c, i) =>
      i === index ? c + 1 : c
    ));
  }

  return counters.map((counter, i) => (
    <div key={i}>
      {counter}
      <button onClick={() => handleIncrement(i)}>+1</button>
    </div>
  ));
}
```

### Replace One Item by ID
```jsx
function List() {
  const [items, setItems] = useState([
    { id: 1, text: 'Buy milk', done: false },
    { id: 2, text: 'Walk dog', done: false }
  ]);

  function handleToggle(itemId) {
    setItems(items.map(item =>
      item.id === itemId
        ? { ...item, done: !item.done }
        : item
    ));
  }
}
```

---

## Sorting and Reversing Arrays

Methods like `sort()` and `reverse()` mutate the original array. **Create a copy first!**

### ❌ Wrong: Direct Mutation
```jsx
function handleReverse() {
  list.reverse(); // ❌ Mutates original
  setList(list);  // React won't detect the change!
}
```

### ✅ Correct: Copy Then Mutate
```jsx
function handleReverse() {
  const copy = [...list];  // Create copy
  copy.reverse();          // Mutate the copy
  setList(copy);           // Set state with new array
}
```

### Sorting Example
```jsx
function handleSort() {
  const sorted = [...items].sort((a, b) => a.name.localeCompare(b.name));
  setItems(sorted);
}

// Or in one line:
setItems([...items].sort((a, b) => a.name.localeCompare(b.name)));
```

---

## Updating Objects Inside Arrays

> **⚠️ CRITICAL: SHALLOW COPYING LIMITATION**
>
> **Copying an array doesn't copy its contents!**
>
> When you copy an array with `[...arr]`, the array itself is new, but the objects inside are still the **same references**.
>
> ```jsx
> const original = [{ id: 1, name: 'Alice' }];
> const copy = [...original];
>
> copy[0].name = 'Bob'; // ❌ Mutates original object!
> console.log(original[0].name); // 'Bob' (changed!)
> ```
>
> **Solution:** Copy individual objects too:
> ```jsx
> const copy = original.map(item => ({ ...item }));
> copy[0].name = 'Bob'; // ✅ Only changes the copy
> ```

### Updating One Object in Array

```jsx
function TodoList() {
  const [todos, setTodos] = useState([
    { id: 0, title: 'Buy milk', done: false },
    { id: 1, title: 'Walk dog', done: false },
    { id: 2, title: 'Read book', done: true }
  ]);

  function handleToggle(todoId) {
    setTodos(todos.map(todo => {
      if (todo.id === todoId) {
        // Create NEW object with changed property
        return { ...todo, done: !todo.done };
      }
      // Return unchanged object
      return todo;
    }));
  }

  return todos.map(todo => (
    <li key={todo.id}>
      <input
        type="checkbox"
        checked={todo.done}
        onChange={() => handleToggle(todo.id)}
      />
      {todo.title}
    </li>
  ));
}
```

**Pattern:**
- Use `map` to create new array
- For the item to update: create new object with `{ ...item, property: newValue }`
- For other items: return unchanged

---

## Updating Nested Properties

For objects with nested properties inside arrays:

```jsx
function ArtworkList() {
  const [artworks, setArtworks] = useState([
    {
      id: 0,
      title: 'Big Bellies',
      artist: { name: 'Alina Szapocznikow', born: 1926 }
    },
    {
      id: 1,
      title: 'Lunar Landscape',
      artist: { name: 'Louise Nevelson', born: 1899 }
    }
  ]);

  function handleIncrementBirthYear(artworkId) {
    setArtworks(artworks.map(artwork => {
      if (artwork.id === artworkId) {
        return {
          ...artwork,                          // Copy artwork
          artist: {
            ...artwork.artist,                 // Copy artist
            born: artwork.artist.born + 1      // Update born
          }
        };
      }
      return artwork;
    }));
  }
}
```

**Multiple levels of copying:**
1. Copy array with `map`
2. Copy object with `{ ...artwork }`
3. Copy nested object with `{ ...artwork.artist }`

---

## Using Immer for Arrays

For complex nested updates, **Immer** makes code much simpler:

### Installation
```bash
npm install use-immer
```

### Basic Usage
```jsx
import { useImmer } from 'use-immer';

function TodoList() {
  const [todos, updateTodos] = useImmer([
    { id: 0, title: 'Buy milk', done: false },
    { id: 1, title: 'Walk dog', done: false }
  ]);

  function handleToggle(todoId) {
    updateTodos(draft => {
      const todo = draft.find(t => t.id === todoId);
      todo.done = !todo.done; // ✅ "Mutate" the draft!
    });
  }

  function handleAdd(title) {
    updateTodos(draft => {
      draft.push({ id: nextId++, title, done: false }); // ✅ Can use push!
    });
  }

  function handleRemove(todoId) {
    updateTodos(draft => {
      const index = draft.findIndex(t => t.id === todoId);
      draft.splice(index, 1); // ✅ Can use splice!
    });
  }
}
```

**Benefits:**
- Write natural mutation syntax
- Works with `push`, `splice`, `pop`, etc.
- Immer handles immutability behind the scenes
- Especially helpful for deeply nested data

---

## Complete Example: Todo App

```jsx
import { useState } from 'react';

function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [text, setText] = useState('');
  let nextId = 0;

  function handleAdd() {
    setTodos([
      ...todos,
      { id: nextId++, text, done: false }
    ]);
    setText('');
  }

  function handleToggle(todoId) {
    setTodos(todos.map(todo =>
      todo.id === todoId
        ? { ...todo, done: !todo.done }
        : todo
    ));
  }

  function handleRemove(todoId) {
    setTodos(todos.filter(todo => todo.id !== todoId));
  }

  function handleEdit(todoId, newText) {
    setTodos(todos.map(todo =>
      todo.id === todoId
        ? { ...todo, text: newText }
        : todo
    ));
  }

  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={handleAdd}>Add</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.done}
              onChange={() => handleToggle(todo.id)}
            />
            {todo.text}
            <button onClick={() => handleRemove(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </>
  );
}
```

---

## Interview Questions to Consider

1. **Q: Why can't you use `push()` to add items to state arrays?**
   - A: `push()` mutates the array in place. React won't detect the change and won't re-render. You need to create a new array.

2. **Q: How do you add an item to the end of a state array?**
   - A: Use spread syntax: `setArray([...array, newItem])` or `concat`: `setArray(array.concat(newItem))`.

3. **Q: How do you remove an item from a state array?**
   - A: Use `filter`: `setArray(array.filter(item => item.id !== targetId))`.

4. **Q: How do you update one item in a state array?**
   - A: Use `map` to create a new array, conditionally creating a new object for the item to update: `setArray(array.map(item => item.id === targetId ? { ...item, property: newValue } : item))`.

5. **Q: What's the problem with `const copy = [...array]` when the array contains objects?**
   - A: Spread creates a shallow copy. The array is new, but objects inside are still the same references. Mutating those objects affects the original.

6. **Q: How do you sort an array in state?**
   - A: Create a copy first, then sort: `setArray([...array].sort((a, b) => ...))`. Never call `sort()` directly on state.

7. **Q: What's the benefit of using Immer?**
   - A: Immer lets you write mutation-like syntax (push, splice, direct assignment) while maintaining immutability behind the scenes. It's especially helpful for deeply nested data.

8. **Q: Can you use array index as a key when items can be reordered or deleted?**
   - A: No! Indices change when items move or are removed, causing React to match components incorrectly. Always use stable IDs.

9. **Q: How do you update a nested object inside an array?**
   - A: Use `map` to copy the array, spread syntax to copy the object, and spread again for nested objects: `array.map(item => item.id === id ? { ...item, nested: { ...item.nested, prop: value } } : item)`.

---

## Key Takeaways

- **Never mutate arrays in state**—create new arrays instead
- **Avoid:** `push`, `pop`, `shift`, `unshift`, `splice`, `reverse`, `sort`, `arr[i] = x`
- **Prefer:** spread (`...`), `concat`, `filter`, `slice`, `map`
- **Adding:** `[...arr, item]` (end), `[item, ...arr]` (start)
- **Removing:** `arr.filter(item => item.id !== targetId)`
- **Updating:** `arr.map(item => item.id === id ? { ...item, prop: value } : item)`
- **Sorting/Reversing:** Copy first `[...arr].sort()`, then mutate the copy
- **Shallow copy warning:** `[...arr]` doesn't copy objects inside
- For nested objects: copy at each level with spread syntax
- **Immer** simplifies complex updates with mutation-like syntax
- Always use stable IDs as keys, not array indices (for dynamic lists)
- Pattern: `map` for updates, `filter` for removals, spread for additions

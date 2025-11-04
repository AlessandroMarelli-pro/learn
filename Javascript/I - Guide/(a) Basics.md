### JavaScript Basics — Teacher’s Synthesis

Sources: [Introduction](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Introduction), [Grammar & types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types), [Loops & iteration](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration)

### 1) What JavaScript Is (and Where It Runs)

- **Language**: Cross‑platform, prototype‑based, dynamically typed scripting language.
- **Where it runs**: In browsers (DOM, events) and on servers (Node.js—files, network, databases).
- **Composition**: Core language + host‑provided APIs (e.g., DOM in browsers, `fs` in Node). Built‑ins include `Array`, `Map`, `Set`, `Math`, `Date`, etc.  
  See MDN: [Introduction](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Introduction)

### 2) Grammar & Types — The Essentials

- **Identifiers & statements**: Case‑sensitive; semicolons are optional (ASI) but recommended for clarity.
- **Comments**: `//` single‑line, `/* ... */` block.
- **Declarations**:
  - `const` (block‑scoped, no reassignment), `let` (block‑scoped, reassign), avoid `var` (function‑scoped, hoisted).
- **Primitive types**: `string`, `number`, `bigint`, `boolean`, `undefined`, `symbol`, `null`.
- **Objects**: Arrays, functions, plain objects, maps/sets, etc. Everything that’s not a primitive is an object.
- **Type checks**:
  - `typeof x` (note: `typeof null === 'object'` quirk)
  - `Array.isArray(x)` for arrays, `x === null` for `null`
- **Operators to know**: `?.` (optional chaining), `??` (nullish coalescing), strict equality `===`.

```js
const userName = 'Ava'; // string
let attempts = 0; // number
const id = Symbol('id'); // symbol
const maybe = null; // null

typeof maybe; // 'object' (historical quirk)
Array.isArray([1]); // true
```

See MDN: [Grammar & types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types)

### 3) Control Flow & Functions

- **Flow**: `if/else`, `switch`, `try/catch/finally`, `throw`.
- **Functions**: declarations, expressions, arrow functions; default and rest parameters. Arrow functions do not bind their own `this`.
- **Modern defensive patterns**: optional chaining and nullish coalescing.

```js
function add(a = 0, b = 0) {
  return a + b;
}
const incAll = (...vals) => vals.map((v) => v + 1);

const mode = config?.settings?.mode ?? 'default';
```

### 4) Loops & Iteration — Choosing the Right Tool

- `for (init; condition; step)`: index‑based; great for numeric loops.
- `for...of`: values of an iterable (arrays, strings, maps, sets, generators). Use this for array items.
- `for...in`: enumerable keys in an object (including inherited). Avoid on arrays; guard with `hasOwnProperty` if needed.
- `while` / `do...while`: condition‑controlled loops; `do...while` runs at least once.
- `break` and `continue`: control loop progression; labels exist but use sparingly.

```js
// for...of over values
for (const ch of 'abc') {
  console.log(ch);
}

// for...in over object keys (guard own props)
const obj = { a: 1, b: 2 };
for (const k in obj) {
  if (Object.prototype.hasOwnProperty.call(obj, k)) {
    console.log(k, obj[k]);
  }
}
```

See MDN: [Loops & iteration](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration)

> ⚠️ **Pitfalls (Read Carefully)**
>
> - ASI (automatic semicolon insertion) can misparse lines starting with `[` or `(`. Prefer explicit semicolons.
> - `typeof null === 'object'`: check `x === null` for null, not `typeof`.
> - `for...in` iterates keys (including inherited) and is not for arrays. Use `for...of` for values.
> - Loose equality `==` coerces types; prefer `===`.
> - Arrow functions don’t bind `this`; don’t use them for methods needing dynamic `this`.
> - Reassigning a `const` binding throws, but mutating a const object is allowed.
> - Omitting `default` values or nullish checks may cause `Cannot read properties of undefined`—use `?.` and `??`.

### 5) Heuristic: Loops vs Array Methods

- If you’re transforming to a new array, prefer `map`/`filter`/`reduce` for clarity.
- If you need early exit or performance‑critical hot paths, a classic `for` can be clearer and faster.

```js
// Clear intent via array methods
const activeUserNames = users.filter((u) => u.active).map((u) => u.name);
```

### Interview‑Style Questions (Practice)

- What are JavaScript’s primitive types? How do you reliably check for arrays and null?
- Explain differences among `var`, `let`, and `const` (scope, hoisting, reassignment).
- When would you choose `for...of` vs `for...in` vs classic `for`?
- What problems do optional chaining `?.` and nullish coalescing `??` solve?
- Why prefer `===` over `==`? Give coercion examples.
- How does `this` differ between arrow functions and regular functions?
- Describe a bug caused by ASI and how to avoid it.
- Why is `typeof null === 'object'` and how do you check for `null`?
- What are pros/cons of array methods vs loops in readability and performance?

### References

- MDN — Introduction: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Introduction
- MDN — Grammar & types: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Grammar_and_types
- MDN — Loops & iteration: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Loops_and_iteration

### Arrow Functions — Teacher’s Synthesis

Source: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions

### 1) What arrow functions are

- Concise function expressions with different semantics: lexical `this`, no own `arguments/super/new.target`, not constructible, and cannot be generators.

```js
const inc = (x) => x + 1; // expression body (implicit return)
const add = (a, b) => {
  return a + b;
}; // block body (explicit return)
```

### 2) Parameters and syntax notes

- Parentheses required for zero/multiple/default/rest/destructured params; optional for a single simple param.
- Return object literals by wrapping in parentheses.

```js
const id = (x) => x;
const sum = (a, b = 0) => a + b;
const pickX = ({ x }) => ({ x });
```

### 3) Lexical `this` (and no own `arguments`)

- Arrow functions capture `this` from the enclosing scope at definition time; `call/apply/bind` cannot change it.
- Use rest parameters instead of `arguments`.

```js
const obj = {
  x: 1,
  method() {
    const f = () => this.x; // `this` is obj
    return f(); // 1
  },
};

const sumAll = (...nums) => nums.reduce((t, n) => t + n, 0);
```

### 4) Not methods, not constructors, not generators

- As object/class methods that rely on dynamic `this`, prefer regular functions. `new (() => {})` throws `TypeError`.

```js
const obj2 = { x: 2, bad: () => this.x }; // likely undefined
function Good() {}
// new (() => {}) // TypeError
```

### 5) Async arrows

- Prefix with `async` for promise‑returning arrows.

```js
const fetchJson = async (url) => (await fetch(url)).json();
```

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Using arrows as methods expecting dynamic `this` results in wrong `this` (it’s lexical).
> - Forgetting parentheses around returned object literals returns `undefined`.
> - Expecting `call/apply/bind` to change an arrow’s `this` — they don’t.
> - Using `arguments` in arrows fails — use rest params.
> - Attempting to construct (`new`) or yield in arrows throws; they aren’t constructors or generators.

### Interview‑Style Questions (Practice)

- Contrast arrow functions with traditional function expressions (this, arguments, constructibility).
- Why do arrows help in callbacks/React handlers? Give an example avoiding `.bind` in a class via arrow field.
- Demonstrate returning an object literal from an arrow; explain why parentheses are required.
- Convert a function using `arguments` to use rest parameters.
- When should you NOT use an arrow? Provide a method example requiring dynamic `this`.

### References

- MDN — Arrow function expressions: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions

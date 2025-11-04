### Functions & Expressions — Teacher’s Synthesis

Sources: [Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions), [Expressions & operators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_operators)

### 1) Functions: the essentials

- **What**: Reusable blocks that take input (parameters) and return output. Defined via declarations, expressions, or arrows.  
  See MDN: [Functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions)
- **Ways to define**:
  - Declaration: `function add(a, b) { return a + b; }` (hoisted)
  - Expression: `const add = function (a, b) { return a + b; };`
  - Arrow: `const add = (a, b) => a + b;` (lexical `this`, no `arguments`)
- **Parameters**: default values and rest parameters.

```js
function greet(name = 'world') {
  return `Hello, ${name}`;
}
const sum = (...nums) => nums.reduce((t, n) => t + n, 0);
```

- **Arguments & spread**: The legacy `arguments` object exists in non‑arrow functions; prefer rest `(...args)` and call sites with spread `fn(...arr)`.
- **Methods & `this`**: When a function is called as an object property, `this` points to that object. Arrow functions don’t bind `this`; they capture it lexically.
- **Closures**: Inner functions capture variables from the outer scope even after the outer function returns.

```js
function makeCounter() {
  let n = 0;
  return () => ++n; // closure over n
}
const next = makeCounter();
next(); // 1, then 2, 3...
```

- **Binding & invocation control**: `fn.call(ctx, ...args)`, `fn.apply(ctx, argsArray)`, `fn.bind(ctx)` returns a new function with fixed `this` (and optional prefilled args).

### 2) Expressions & operators: building blocks of logic

- **Expression kinds**: primary (`1`, `'x'`, `true`, `null`, `this`), array/object literals, function/arrow expressions, class expressions, template literals.
- **Operators**:
  - Arithmetic: `+ - * / % **` (exponentiation is right‑associative)
  - Assignment: `=`, compound `+=`, `-=`, `&&=`, `||=`, `??=`
  - Comparison: `< <= > >=`, equality `===` (strict), `==` (loose/coercive)
  - Logical: `&&`, `||`, `!` with short‑circuiting; `??` for nullish coalescing
  - Optional chaining: `?.` safe property/access/call
  - Spread/Rest: `...` in literals/calls/params
  - Conditional: `cond ? a : b`
  - Comma operator: evaluates left to right, returns last (rare)

```js
const title = config?.ui?.title ?? 'Untitled';
user ??= { role: 'guest' }; // only if user is null/undefined
const copy = { ...defaults, ...overrides };
```

- **Precedence & associativity** matter: `a + b * c` multiplies before addition; parentheses clarify intent.
- **Numbers vs BigInt**: Can’t mix `number` and `bigint` in arithmetic; convert deliberately.

```js
10n + 2n; // 12n
// 10n + 2  -> TypeError (cannot mix BigInt and Number)
```

> ⚠️ **Pitfalls (Read Carefully)**
>
> - `==` performs type coercion; prefer `===` to avoid surprises.
> - Arrow functions lack their own `this`, `arguments`, and `prototype`; don’t use them as constructors or methods that need dynamic `this`.
> - Default parameters run at call time; `function f(x = expensive()) {}` calls `expensive()` when `x` is `undefined`.
> - Mixing `number` and `bigint` in operations throws; convert explicitly.
> - `||` treats many values (like `0`, `''`, `NaN`) as falsy; use `??` when you only want to default on `null`/`undefined`.
> - `for...in` iterates enumerable keys (including inherited); not for arrays—use `for...of` or array methods.
> - `bind` creates a new function; repeated binding and wrapping can complicate `this` and debugging.

### 3) Patterns worth knowing

- **Higher‑order functions**: functions that take or return functions (e.g., `map`, `filter`, decorators).
- **Partial application via `bind`**:

```js
function multiply(a, b) {
  return a * b;
}
const double = multiply.bind(null, 2);
double(10); // 20
```

- **Safe access with optional chaining**:

```js
const postal = user?.address?.postalCode ?? 'N/A';
```

### Interview‑Style Questions (Practice)

- Compare function declarations, function expressions, and arrow functions (hoisting, `this`, `arguments`).
- Explain closures with a minimal example and a real‑world use case.
- When do you use `call`, `apply`, and `bind`? Differences and examples.
- Difference between `||` and `??`. When does `||` backfire? Give examples.
- Why prefer `===` over `==`? Provide a coercion gotcha example.
- How do rest parameters differ from the `arguments` object? Why prefer rest?
- What breaks if you use an arrow function as an object method or constructor?
- Why can’t `BigInt` and `Number` be mixed in arithmetic? How to handle conversions?
- What’s operator precedence and how can it bite you? Show a fix with parentheses.

### References

- MDN — Functions: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions
- MDN — Expressions & operators: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Expressions_and_operators

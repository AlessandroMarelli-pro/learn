### Closures — Teacher’s Synthesis

Source: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures

### 1) What is a closure?

- A closure is a function bundled with its lexical environment (all variables from outer scopes that it references). Every function creation forms a closure.

```js
function makeAdder(x) {
  return function add(y) {
    return x + y; // x is captured from outer scope
  };
}

const add5 = makeAdder(5);
add5(2); // 7
```

### 2) Lexical scoping basics

- Variable resolution is based on where functions are defined, not where they are called.
- Closures can capture from block scope (`let`, `const`), function scope, and module scope.

```js
function outer() {
  const secret = 's';
  function inner() { return secret; }
  return inner;
}
outer()(); // 's'
```

### 3) Practical uses

- **Private state**: encapsulate data without exposing it directly.
- **Function factories**: produce specialized functions by capturing configuration.
- **Callbacks/handlers**: carry context into async operations.

```js
function createCounter() {
  let n = 0;
  return {
    inc() { return ++n; },
    value() { return n; },
  };
}
const c = createCounter();
c.inc(); c.value(); // 1
```

### 4) Loops and closures — the classic gotcha

- Using `var` in loops creates one function-scoped binding; all closures see the final value. Use `let` to get a per-iteration binding.

```js
// Wrong (var): logs 3, 3, 3
const fns1 = [];
for (var i = 0; i < 3; i++) {
  fns1.push(() => console.log(i));
}
fns1.forEach((f) => f());

// Correct (let): logs 0, 1, 2
const fns2 = [];
for (let j = 0; j < 3; j++) {
  fns2.push(() => console.log(j));
}
fns2.forEach((f) => f());
```

### 5) Memory and performance considerations

- Captured variables stay alive as long as any closure referencing them remains reachable.
- Avoid capturing large objects unnecessarily; extract needed fields or pass parameters directly.
- Be mindful in long-lived listeners or caches; unregister/clear when done.

### 6) Patterns and tips

- Prefer `let`/`const` to avoid function-scope surprises with `var`.
- Use closures to model modules or services with private internals and a small API surface.
- For event handlers, ensure you remove them to allow garbage collection of captured environments.

> ⚠️ **Pitfalls (Read Carefully)**
>
> - `var` in loops creates one binding; closures see the last value. Use `let` for per-iteration capture.  
> - Capturing mutable objects leads to shared state; subsequent mutations are visible to all closures.  
> - Accidentally retaining closures (e.g., lingering event listeners, timers) leaks memory and keeps captured data alive.  
> - Relying on `this` inside arrow functions for closure state can backfire; arrows capture `this` lexically, not dynamically.  
> - Heavy work in getters within closed-over objects runs on every access; closures don’t cache results unless you implement it.

### Interview‑Style Questions (Practice)

- Define a closure and explain lexical scoping with a minimal example.  
- Why does `var` inside a `for` loop cause the “3,3,3” bug? Fix it.  
- Show how to implement private state using closures without classes.  
- How can closures cause memory leaks? Give two mitigation strategies.  
- Contrast capturing a primitive vs capturing an object reference; what changes when the object mutates?  
- When would you use a function factory pattern in real projects?

### References

- MDN — Closures: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Closures



### Objects, Classes & Promises — Teacher’s Synthesis

Sources: [Working with objects](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects), [Using classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_classes), [Using promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)

### 1) Working with Objects — the essentials

- **What**: Objects are key–value maps; values may be data or functions (methods). Plain objects come from literals `{}` or `Object.create`.
- **Create**:
  - Literal: `const user = { name: 'Ava', age: 30 }`
  - Factory: `const make = (n) => ({ name: n })`
  - Prototype: `const o = Object.create(proto)`
- **Access & mutate**: dot `obj.x`, bracket `obj['x']` (computed keys). Delete with `delete obj.x`.
- **Descriptors**: `Object.defineProperty(obj, 'x', { value, writable, enumerable, configurable })`
- **Getters/Setters**: computed access with side effects or validation.

```js
const account = {
  _balance: 0,
  get balance() {
    return this._balance;
  },
  set balance(v) {
    if (v < 0) throw new Error('neg');
    this._balance = v;
  },
};
```

- **Introspection**: `Object.keys`, `values`, `entries`, `hasOwn` (or `Object.prototype.hasOwnProperty.call`).
- **Copy/merge**: shallow via spread `{ ...a, ...b }` or `Object.assign`.
- **Identity**: Objects compare by reference; deep equality requires custom logic.
- **Inheritance**: Each object has a prototype; property lookup walks the chain. Use `Object.create(proto)` or classes for structured inheritance.

### 2) Using Classes — syntax over prototypes

- **Definition**: Syntactic sugar over prototypes.

```js
class Person {
  #id; // private field
  static species = 'H. sapiens';
  constructor(name) {
    this.name = name;
    this.#id = crypto.randomUUID?.() ?? 'n/a';
  }
  greet() {
    return `Hi, I'm ${this.name}`;
  }
  get id() {
    return this.#id;
  }
}

class Student extends Person {
  constructor(name, cohort) {
    super(name);
    this.cohort = cohort;
  }
  greet() {
    return `${super.greet()} from cohort ${this.cohort}`;
  }
}
```

- **Key points**:
  - `constructor` runs on `new`. Methods are on the prototype (shared per instance).
  - `extends` sets up the prototype chain; use `super()` before accessing `this` in derived constructors.
  - `static` members belong to the class, not instances.
  - `#private` fields/methods are enforced by the language and inaccessible outside the class.
  - `public` fields/methods are writable, enumerable, and configurable properties defined on each class instance or class constructor.

### 3) Using Promises — async fundamentals

- **States**: pending → fulfilled or rejected. A promise settles once.
- **Consume**: `.then(onFulfilled)`, `.catch(onRejected)`, `.finally(handler)`; chaining flattens returned promises.
- **Create**:

```js
const fetchJson = (url) =>
  new Promise((resolve, reject) => {
    fetch(url)
      .then((r) => (r.ok ? r.json() : Promise.reject(new Error(r.status))))
      .then(resolve, reject);
  });
```

- **Combinators**: `Promise.all` (fail‑fast), `allSettled` (always waits), `race` (first settles), `any` (first fulfill).
- **Async/await**: sugar over promises; use `try/catch` around `await`.

```js
async function loadAll(urls) {
  const responses = await Promise.all(urls.map((u) => fetch(u)));
  return Promise.all(responses.map((r) => r.json()));
}
```

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Objects are compared by reference; copying with spread is shallow — nested objects remain shared.
> - Getters/setters run on access/assignment; avoid heavy work or hidden side effects.
> - In classes, using `this` before `super()` throws; private `#` fields are not accessible outside (no reflection).
> - Arrow methods as class fields capture `this` lexically, which helps with callbacks but creates a per‑instance function.
> - Promise chains: forgetting to `return` in `.then` breaks sequencing; missing `.catch` leads to unhandled rejections.
> - `await` inside `Array.prototype.forEach` won’t await; use `for...of` or `Promise.all` for parallelism.
> - `Promise.all` fails fast — one rejection rejects all; prefer `allSettled` when you need all results.
> - Mixing async patterns (callbacks + promises) can cause double resolution; wrap carefully.

### Interview‑Style Questions (Practice)

- How do `Object.keys`/`values`/`entries` differ from `for...in`? Why prefer `hasOwn` checks?
- Explain prototype lookup and how `Object.create(proto)` differs from a class instance.
- What are the trade‑offs of using getters/setters vs plain methods?
- In classes, when do you need `super()` and what does it do? How do `static` and `#private` members behave?
- Show two safe patterns to keep method `this` when passing as a callback.
- Compare `Promise.all`, `allSettled`, `race`, and `any` with use cases.
- Why does `await` not work as expected inside `forEach`? Provide a correct alternative.
- How does error propagation work across chained promises and `async/await`?

### References

- MDN — Working with objects: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Working_with_objects
- MDN — Using classes: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_classes
- MDN — Using promises: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises

### Promises in Node.js — Synthesis

Source: [Discover Promises in Node.js](https://nodejs.org/en/learn/asynchronous-work/discover-promises-in-nodejs)

---

### Big idea

- A Promise represents the eventual result of an async operation: pending → fulfilled or rejected, then settled.
- Use `.then/.catch/.finally` to handle outcomes; `async/await` builds on Promises for linear, readable flow.
- Many Node core APIs expose Promise variants (e.g., `fs/promises`).

---

### States and basics

- States: pending, fulfilled (value), rejected (reason/error).
- Create with `new Promise((resolve, reject) => { ... })`.

```js
const fetchValue = new Promise((resolve, reject) => {
  try {
    resolve('ok');
  } catch (err) {
    reject(err);
  }
});

fetchValue
  .then((v) => console.log(v))
  .catch((err) => console.error(err))
  .finally(() => console.log('done'));
```

---

### Chaining

- Each `.then` returns a new Promise, enabling sequencing.

```js
const { setTimeout: delay } = require('node:timers/promises');

delay(500)
  .then(() => 'first')
  .then((v) => delay(500).then(() => v + ' → second'))
  .then(console.log)
  .catch(console.error);
```

---

### Async/await

- `async` functions return a Promise; `await` pauses until the Promise settles.

```js
const fs = require('node:fs/promises');

async function readText(path) {
  try {
    const data = await fs.readFile(path, 'utf8');
    return data.trim();
  } catch (err) {
    // handle or rethrow
    throw err;
  }
}
```

Top-level await (ESM only):

```js
import { setTimeout as delay } from 'node:timers/promises';
await delay(100);
```

---

### Promise-based Node APIs

- Use `fs.promises` or `require('node:fs/promises')` for Promise-returning methods.

```js
const fs = require('node:fs/promises');
const text = await fs.readFile('example.txt', 'utf8');
```

---

### Advanced methods

- `Promise.all(promises)`: fail-fast; resolves to array of values.
- `Promise.allSettled(promises)`: never fail-fast; returns statuses for each.
- `Promise.race(promises)`: settles with the first to settle.
- `Promise.any(promises)`: resolves with the first fulfillment; rejects with `AggregateError` if all reject.
- `Promise.resolve(value)` / `Promise.reject(reason)`: create settled Promises.
- `Promise.withResolvers()`: create `{ promise, resolve, reject }` pair (useful for external resolution control).

```js
const { setTimeout: delay } = require('node:timers/promises');

const a = delay(200).then(() => 'A');
const b = delay(100).then(() => 'B');

const both = await Promise.all([a, b]); // ['A','B'] after both complete
const first = await Promise.race([a, b]); // 'B'
```

---

### Event loop scheduling (microtasks vs tasks)

- Microtasks: Promise reactions, `queueMicrotask` — run after current turn, before I/O/timers.
- `process.nextTick` runs even before microtasks in Node’s semantics; use sparingly.
- `setImmediate` runs in the check phase (after poll), useful to yield after I/O.

```js
queueMicrotask(() => console.log('microtask'));
process.nextTick(() => console.log('nextTick'));
setImmediate(() => console.log('immediate'));
console.log('sync');
```

---

> ⚠️ **Pitfalls**
>
> - **Unhandled rejections**: Always attach `.catch` or wrap `await` in `try/catch` to avoid process warnings/crashes.
> - **Forgotten `return` in chains**: Returning inside `.then` is required to pass values or Promises along.
> - **Sequential vs concurrent awaits**: Avoid `await` in loops when tasks can run in parallel; batch with `Promise.all`.
> - **Mixing styles**: Don’t mix `then` chains and `await` in the same flow unless you’re deliberate.
> - **Swallowed errors**: Empty `.catch` or broad `try/catch` can hide failures; log and propagate appropriately.
> - **Event loop ordering confusion**: Know when to use `nextTick`, microtasks, or `setImmediate`.
> - **Leaking promises**: Never leave Promises pending without cancellation/timeouts where appropriate.
> - **Top-level await latency**: In ESM, top-level `await` can delay module load; design for it.

---

### Interview-ready talking points

- Explain Promise states and how chaining works (each `.then` returns a new Promise).
- Compare `Promise.all`, `allSettled`, `race`, and `any` and when you’d use each.
- Show sequential vs concurrent patterns with `await` and `Promise.all`.
- Discuss unhandled rejection handling and error propagation strategies.
- Clarify event loop ordering: microtasks, `nextTick`, `setImmediate`.

---

### Quick checklist

- [ ] Did I handle rejections for every Promise path?
- [ ] Should steps run concurrently (`Promise.all`) instead of sequentially?
- [ ] Am I returning from `.then` consistently?
- [ ] Are timeouts/cancellation applied to long-running Promises?
- [ ] Is module startup impacted by top-level `await`?

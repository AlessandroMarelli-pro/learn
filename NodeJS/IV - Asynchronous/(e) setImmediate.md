### Understanding setImmediate — Synthesis

Source: [Understanding setImmediate()](https://nodejs.org/en/learn/asynchronous-work/understanding-setimmediate)

---

### What it does

- `setImmediate(fn)` schedules `fn` to run on the next iteration of the event loop (check phase).
- It’s different from:
  - `process.nextTick(fn)`: runs before the event loop continues (sooner than microtasks in Node semantics).
  - `Promise.resolve().then(fn)` / `queueMicrotask(fn)`: microtasks run after current stack, before macrotasks.
  - `setTimeout(fn, 0)`: a macrotask scheduled for the timers phase; ordering vs `setImmediate` depends on context.

---

### Ordering example (CommonJS)

```js
const baz = () => console.log('baz'); // setImmediate
const foo = () => console.log('foo'); // nextTick
const zoo = () => console.log('zoo'); // nextTick inside promise handler

console.log('start');
setImmediate(baz);
Promise.resolve('bar').then((val) => {
  console.log(val);
  process.nextTick(zoo);
});
process.nextTick(foo);

// Typical CJS order: start → foo → bar → zoo → baz
```

Note: In ESM, the entire module load is async, so the observed order can differ (e.g., `start → bar → foo → zoo → baz`).

---

### When to use

- Defer work to the next loop iteration without blocking current microtasks.
- Yield after heavy callbacks to allow I/O to proceed.
- Schedule tasks that should run after the poll phase (e.g., after pending I/O callbacks).

---

> ⚠️ **Pitfalls**
>
> - **Confusing with nextTick**: `process.nextTick` runs sooner and can starve the event loop if overused.
> - **Assuming fixed order vs setTimeout(0)**: Their relative order depends on timing and context.
> - **Microtask priority**: Promise handlers run before `setImmediate`; don’t expect `setImmediate` to beat `.then()`.
> - **Overuse in hot paths**: Excess scheduling introduces overhead; batch or restructure when possible.

---

### Interview-ready talking points

- Explain where `setImmediate` runs in the event loop (check phase) vs `setTimeout(0)` (timers phase).
- Contrast `process.nextTick` (pre-phase, starvation risk) and microtasks (Promises) with `setImmediate`.
- Describe scenarios to choose `setImmediate` to yield between expensive work chunks.
- Discuss ESM vs CommonJS differences in observed ordering.

---

### Quick checklist

- [ ] Do I need to defer to the next iteration (use `setImmediate`) vs immediately after current stack (`nextTick`/microtask)?
- [ ] Will microtasks run first and is that acceptable?
- [ ] Could repeated scheduling starve throughput or add overhead?

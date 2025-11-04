### Node.js Event Loop, Blocking vs Non-Blocking — Synthesis

Sources: [Overview of Blocking vs Non-Blocking](https://nodejs.org/en/learn/asynchronous-work/overview-of-blocking-vs-non-blocking) · [The Node.js Event Loop](https://nodejs.org/en/learn/asynchronous-work/event-loop-timers-and-nexttick) · [Don’t Block the Event Loop](https://nodejs.org/en/learn/asynchronous-work/dont-block-the-event-loop) · [Understanding process.nextTick](https://nodejs.org/en/learn/asynchronous-work/understanding-processnexttick)

---

### Big idea

- Node.js runs JavaScript on a single thread but achieves high throughput via an **event loop** and **non-blocking I/O** (libuv).
- Prefer async, non-blocking APIs; avoid CPU-heavy synchronous work on the main thread.

---

### Blocking vs non-blocking

- **Blocking/sync** halts JS execution until completion (e.g., `fs.readFileSync`).
- **Non-blocking/async** returns immediately and later invokes a callback or resolves a Promise (e.g., `fs.readFile`).

```js
const fs = require('node:fs');

// Blocking: next line waits for disk I/O
const data = fs.readFileSync('file.md');

// Non-blocking: moreWork() proceeds while I/O happens
fs.readFile('file.md', (err, data) => {
  /* use data */
});
moreWork();
```

Throughput increases when I/O waits don’t block the event loop.

---

### Event loop phases (simplified)

- **timers**: `setTimeout`/`setInterval` callbacks
- **pending callbacks**: system/low-level callbacks
- **idle, prepare**: internal
- **poll**: retrieve new I/O events; execute related callbacks
- **check**: `setImmediate` callbacks
- **close callbacks**: e.g., `'close'` events

Callbacks are executed FIFO within each phase; operations may schedule more tasks.

---

### Timers, immediates, microtasks

- `setTimeout(fn, 0)`: queue after current stack (timers phase)
- `setImmediate(fn)`: queue for the check phase (after poll)
- Microtasks: Promise reactions, `queueMicrotask`, and `process.nextTick` (Node-specific) run before moving to the next event loop phase

```js
setTimeout(() => console.log('timeout'));
setImmediate(() => console.log('immediate'));
Promise.resolve().then(() => console.log('microtask'));
process.nextTick(() => console.log('nextTick'));
console.log('sync');
```

Typical order in Node.js: `sync` → `nextTick` → `microtask` → timers/check (ordering between timeout/immediate depends on context).

---

### process.nextTick vs setImmediate

- `process.nextTick(fn)`: runs before the event loop continues (even before Promise microtasks in Node semantics). Use sparingly to avoid starving the loop.
- `setImmediate(fn)`: runs on the next iteration (check phase). Prefer this for deferring to the next tick.

```js
process.nextTick(() => console.log('nextTick')); // very soon
setImmediate(() => console.log('immediate')); // next iteration
```

Use `nextTick` to finish setup (e.g., emit after listeners attach), not as a general scheduler.

---

### Don’t block the event loop (CPU-bound work)

- Expensive CPU tasks (compression, crypto, CPU parsing) block I/O. Strategies:
  - Chunk work with timers or `setImmediate` to yield between chunks
  - Use worker threads or native addons for CPU-bound tasks
  - Stream data instead of buffering entire payloads

```js
function heavyWork(items) {
  let i = 0;
  function chunk() {
    const end = Math.min(i + 1000, items.length);
    while (i < end) {
      /* process items[i++] */
    }
    if (i < items.length) setImmediate(chunk); // yield to event loop
  }
  chunk();
}
```

---

> ⚠️ **Pitfalls**
>
> - **Mixing sync and async I/O**: A stray `*Sync` call can reorder logic or stall throughput.
> - **Starving the loop**: Excessive `process.nextTick` or long callbacks prevent I/O progress.
> - **Assuming timer precision**: Timers are not real-time; load and phase ordering affect timing.
> - **CPU-bound work on main thread**: Blocks all requests; offload or chunk work.
> - **Unhandled async ordering**: `setTimeout(0)` vs `setImmediate` behavior differs by context.
> - **Unhandled errors**: Async exceptions aren’t caught by surrounding `try/catch`; attach handlers.

---

### Interview-ready talking points

- Define blocking vs non-blocking with concrete `fs` examples and throughput implications.
- Describe event loop phases and where timers, poll, and check occur.
- Explain `process.nextTick` vs microtasks vs `setImmediate`, and proper use cases.
- Strategies to avoid blocking: streaming, chunking, worker threads.
- Discuss how mixing sync and async causes subtle bugs in ordering.

---

### Quick checklist

- [ ] Am I avoiding `*Sync` in hot paths?
- [ ] Are long operations chunked or offloaded to workers?
- [ ] Do I understand when to use `nextTick` vs `setImmediate`?
- [ ] Are errors handled in async flows (Promise `.catch`, callbacks)?

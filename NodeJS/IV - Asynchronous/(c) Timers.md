### JavaScript Timers — setTimeout, setInterval, and Recursive Scheduling

Source: [Discover JavaScript Timers (recursive setTimeout)](https://nodejs.org/en/learn/asynchronous-work/discover-javascript-timers#recursive-settimeout)

---

### Overview

- **setTimeout(fn, ms)**: run `fn` once after `ms` milliseconds; returns a handle to cancel.
- **setInterval(fn, ms)**: run `fn` repeatedly every `ms`; returns a handle to cancel.
- **clearTimeout/clearInterval(handle)**: cancel scheduled callbacks.
- In Node.js, timers return `Timeout` objects; in browsers, IDs.

---

### Zero delay and immediates

- `setTimeout(fn, 0)` queues `fn` after the current call stack completes.
- Node.js also has `setImmediate(fn)` to queue work after the poll phase of the event loop.

---

### Why recursive setTimeout

`setInterval` does not wait for the previous run to finish. If work duration varies, calls can overlap. Use a recursive `setTimeout` to run the next tick only after the current work completes.

```js
function tick() {
  // do work (may take variable time)
  setTimeout(tick, 1000); // schedule next run after finishing
}

setTimeout(tick, 1000);
```

This ensures no overlap: each run schedules the next when done.

---

### Drift-aware scheduling

For more regular cadence, compute the next target time to reduce drift:

```js
const intervalMs = 1000;
let next = Date.now() + intervalMs;

function loop() {
  // do work
  next += intervalMs;
  const delay = Math.max(0, next - Date.now());
  setTimeout(loop, delay);
}

setTimeout(loop, intervalMs);
```

---

### Cancel timers

```js
const handle = setTimeout(() => {
  /* work */
}, 2000);
clearTimeout(handle);

const interval = setInterval(() => {
  /* work */
}, 500);
clearInterval(interval);
```

---

> ⚠️ **Pitfalls**
>
> - **Overlapping work with setInterval**: If `fn` takes longer than the interval, runs can overlap. Prefer recursive `setTimeout`.
> - **Assuming exact timing**: Timers are not real-time; long tasks and event loop phases can delay callbacks.
> - **Zero-delay misconceptions**: `setTimeout(fn, 0)` runs after the current stack, not immediately.
> - **Leaking timer handles**: Un-cleared intervals/timeouts can keep the Node.js process alive.
> - **Clock drift**: Using fixed `setTimeout(interval)` accumulates drift; compute next delay from a target time.
> - **Heavy CPU work**: Timer callbacks that do CPU-bound work block I/O; offload or chunk work.

---

### Interview-ready talking points

- Compare `setInterval` vs recursive `setTimeout` and explain overlap/drift.
- Explain zero-delay semantics and when to use `setImmediate` in Node.
- Show how to cancel timers and why it matters for process lifecycle.
- Discuss timer accuracy limits due to the event loop and long tasks.

---

### Quick checklist

- [ ] Could runs overlap? Use recursive `setTimeout` instead of `setInterval`.
- [ ] Do I need steady cadence? Use drift-aware scheduling.
- [ ] Do I cancel timers when no longer needed?
- [ ] Is the work chunked to avoid blocking the event loop?

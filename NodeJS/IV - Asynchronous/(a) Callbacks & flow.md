### Callbacks & Asynchronous Flow Control — Synthesis

Sources: [JavaScript Asynchronous Programming and Callbacks](https://nodejs.org/en/learn/asynchronous-work/javascript-asynchronous-programming-and-callbacks) · [Asynchronous flow control](https://nodejs.org/en/learn/asynchronous-work/asynchronous-flow-control)

---

### Big idea

- JavaScript is single-threaded and synchronous by default, but its environments (browsers, Node.js) provide asynchronous, non-blocking APIs.
- Asynchrony in JS is often coordinated with **callbacks**; Node.js popularized the **error-first callback** pattern.
- Complex async work benefits from clear flow-control patterns (series, limited concurrency, full parallel) and modular functions.

---

### Callbacks 101 (error-first)

- A callback is a function passed to another function and invoked when work completes.
- Node-style: first argument is `err` (or `null` if no error), followed by result(s).

```js
const fs = require('node:fs');

fs.readFile('file.json', (err, data) => {
  if (err) {
    console.error(err);
    return; // always handle and return early
  }
  console.log('bytes:', data.length);
});
```

Why callbacks? They defer execution until an event (I/O, timer, user input) completes without blocking the main thread.

---

### The problem with callbacks

- Nested callbacks become hard to read and reason about ("callback hell").
- State and error propagation get messy with deep nesting.
- Modern alternatives: **Promises** and **async/await** reduce nesting and improve composability (covered elsewhere).

---

### Flow-control patterns with callbacks

1. In series (sequential)

```js
function runSeries(tasks, done) {
  const next = (i) => {
    if (i === tasks.length) return done();
    tasks[i]((err) => (err ? done(err) : next(i + 1)));
  };
  next(0);
}
```

2. Limited concurrency (cap parallelism)

```js
function runWithLimit(tasks, limit, done) {
  let i = 0,
    running = 0,
    finished = 0,
    error;
  const launch = () => {
    while (running < limit && i < tasks.length) {
      running += 1;
      const idx = i++;
      tasks[idx]((err) => {
        running -= 1;
        finished += 1;
        if (!error && err) error = err;
        if (finished === tasks.length || error) return done(error);
        launch();
      });
    }
  };
  launch();
}
```

3. Full parallel (fire all; join when last completes)

```js
function runParallel(tasks, done) {
  let completed = 0,
    error;
  tasks.forEach((task) => {
    task((err) => {
      if (!error && err) error = err;
      completed += 1;
      if (completed === tasks.length) done(error);
    });
  });
}
```

These patterns mirror common needs: strict order, throughput with bounds, or maximum concurrency.

---

> ⚠️ **Pitfalls**
>
> - **Forgetting error-first conventions**: Always handle `err` first and return early to avoid double-callbacks.
> - **Callback hell**: Deep nesting reduces readability—modularize, name functions, or use Promises/async.
> - **Double invocation**: Guard against calling a callback more than once (e.g., multiple event handlers firing).
> - **Uncaught async errors**: Exceptions thrown inside async callbacks won’t be caught by surrounding `try/catch`.
> - **Blocking the event loop**: CPU-heavy work freezes I/O; offload or stream instead of buffering huge data.
> - **Resource leaks**: Ensure timers, streams, and subscriptions are cleaned up on error and on success.

---

### Interview-ready talking points

- Explain error-first callbacks and why they exist in Node.js.
- Compare series vs limited concurrency vs full parallel; when to choose each.
- Strategies to avoid callback hell (named functions, composition, Promises/async).
- Why `try/catch` doesn’t capture async errors; where to attach error handlers.
- Event loop implications: non-blocking I/O, impact of CPU-bound tasks.

---

### Quick checklist

- [ ] Are callbacks error-first and invoked exactly once?
- [ ] Is concurrency controlled appropriately for the workload?
- [ ] Are errors handled and propagated without nesting?
- [ ] Are long-running/CPU-heavy tasks off the main event loop?
- [ ] Are resources (timers/streams) cleaned up on both success and failure?

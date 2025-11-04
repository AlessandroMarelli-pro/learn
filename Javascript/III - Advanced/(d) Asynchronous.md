### Asynchronous JavaScript — Teacher’s Synthesis

Source: https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS

### 1) Why async?

- JS runs on a single thread; blocking operations (network, disk, heavy compute) freeze the UI or event loop. Asynchrony lets long tasks complete without blocking responsiveness.

### 2) Async building blocks

- **Callbacks**: pass a function to be invoked later. Simple but can lead to nested “callback pyramids” and error handling complexity.
- **Promises**: represent eventual completion (fulfilled/rejected). Composable via chaining and combinators.
- **async/await**: syntax sugar on promises; write sequential style while remaining non‑blocking.
- **Workers**: background threads for CPU‑heavy work to keep main thread responsive (Web Workers/Service Workers).

```js
// Promise + async/await
function getJson(url) {
  return fetch(url).then((r) => (r.ok ? r.json() : Promise.reject(r.status)));
}

async function load() {
  try {
    const data = await getJson('/api/data');
    console.log(data);
  } catch (e) {
    console.error('failed', e);
  }
}
```

### 3) Promise patterns you should know

- `Promise.all([p1, p2])`: run in parallel, fail‑fast on first rejection.
- `Promise.allSettled([p1, p2])`: wait for all; inspect status of each.
- `Promise.race([p1, p2])`: settle with the first (fulfill or reject).
- `Promise.any([p1, p2])`: first fulfillment; rejects with AggregateError if all reject.

```js
const results = await Promise.allSettled(urls.map((u) => fetch(u)));
```

### 4) Workers for CPU‑bound tasks

- Offload long computations to a worker to avoid jank.

```js
// main.js
const w = new Worker('worker.js');
w.postMessage({ n: 5_000_000 });
w.onmessage = (e) => console.log('sum', e.data);

// worker.js
self.onmessage = (e) => {
  const { n } = e.data;
  let s = 0;
  for (let i = 0; i < n; i++) s += i;
  self.postMessage(s);
};
```

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Forgetting to return inside `.then()` chains breaks sequencing; always return promises you want to await next.
> - Swallowing rejections (missing `.catch`) leads to unhandled rejections; always terminate chains with a handler.
> - Using `await` inside `Array.prototype.forEach` doesn’t wait; use `for...of` or `Promise.all` for parallelism.
> - Blocking the main thread with CPU work still freezes the app; move heavy compute to workers.
> - Racing requests without timeouts can hang; combine with `AbortController` or `Promise.race` to enforce deadlines.
> - Mixing callbacks and promises can cause double invocation; wrap callback APIs carefully into a single promise.

### 5) Practice ideas

- Sequence animations or requests using promises (chain) versus `async/await` with `for...of`.
- Wrap a callback‑style API (e.g., `setTimeout`) into a promise.
- Add timeouts to fetch with `AbortController`.

```js
function delay(ms) {
  return new Promise((res) => setTimeout(res, ms));
}
await delay(200);
```

### Interview‑Style Questions (Practice)

- Compare callbacks, promises, and `async/await` in error handling and composition.
- When do you choose `Promise.all` vs `allSettled` vs `race` vs `any`? Give examples.
- Why doesn’t `await` inside `forEach` behave as expected? Provide two correct alternatives.
- How would you implement a timeout for `fetch`? Show an `AbortController` example.
- When do you reach for Web Workers, and what are their limitations (no DOM access, message passing)?

### References

- MDN — Asynchronous JavaScript: https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Async_JS

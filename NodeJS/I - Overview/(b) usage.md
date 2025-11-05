### Node.js Usage — Debugging, Fetch/WebSocket, Dev vs Prod, Profiling, WebAssembly

Sources: [Debugging](https://nodejs.org/en/learn/getting-started/debugging), [Fetch](https://nodejs.org/en/learn/getting-started/fetch), [WebSocket](https://nodejs.org/en/learn/getting-started/websocket), [Dev vs Prod](https://nodejs.org/en/learn/getting-started/nodejs-the-difference-between-development-and-production), [Profiling](https://nodejs.org/en/learn/getting-started/profiling), [Node.js with WebAssembly](https://nodejs.org/en/learn/getting-started/nodejs-with-webassembly)

### 1) Debugging with the Inspector

- Start inspector: `node --inspect app.js` (listens on 127.0.0.1:9229). Pause on start: `--inspect-brk`. Wait for client: `--inspect-wait`.
- Attach from Chrome/Edge: open `chrome://inspect` / `edge://inspect`, or use VS Code/JetBrains debug configs.
- Remote: keep inspector bound to localhost; use SSH tunnel `ssh -L 9221:localhost:9229 user@remote` and attach to `localhost:9221`.

```bash
node --inspect-brk=127.0.0.1:9229 server.js
```

### 2) Fetch API (Undici) — HTTP client

- Node ships `fetch` powered by Undici. Use like the browser; parse with `.json()`; for advanced control, use Undici APIs.

```js
const res = await fetch('https://api.example.com/items');
if (!res.ok) throw new Error(res.status);
const items = await res.json();
```

### 3) WebSockets — client basics

- Use `WebSocket` (whatwg) or libraries. Example with the global `WebSocket` in modern Node:

```js
const ws = new WebSocket('wss://echo.websocket.events');
ws.onopen = () => ws.send('hello');
ws.onmessage = (e) => console.log('msg:', e.data);
ws.onerror = (err) => console.error(err);
```

### 4) Development vs Production

- Dev: detailed logs, hot reload, source maps, strict validations. Prod: enable metrics, graceful shutdown, correct env (`NODE_ENV=production`), secure configs (no debug ports), process managers (PM2/systemd), proper timeouts.

```bash
NODE_ENV=production node server.js
```

### 5) Profiling and diagnostics

- CPU profile: `node --cpu-prof app.js` → analyze `.cpuprofile` in DevTools.
- Heap snapshot: `node --heapsnapshot-signal=SIGUSR2 app.js` then `kill -USR2 <pid>`.
- Inspect live: `node --inspect` + Performance tab (record) or use clinic/flamegraph tools.

### 6) WebAssembly in Node

- Load `.wasm` with `WebAssembly.instantiate`/`instantiateStreaming` and call exported functions from JS.

```js
import { readFile } from 'node:fs/promises';
const buf = await readFile('add.wasm');
const { instance } = await WebAssembly.instantiate(buf);
console.log(instance.exports.add(2, 3)); // 5
```

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Exposing `--inspect` on public interfaces (0.0.0.0) is unsafe; always bind to localhost and tunnel if needed.
> - Forgetting to check `res.ok` with `fetch` hides HTTP errors; handle non‑2xx explicitly.
> - WebSocket error handling: add `onerror`/`onclose` and retries; beware unbounded reconnect loops.
> - Running dev settings in prod (verbose logs, hot reload, open debug ports) hurts performance/security; set `NODE_ENV=production`.
> - Profiling changes timings; profile under realistic load and isolate warm‑up; don’t keep profiling flags on in prod.
> - WASM I/O boundaries: moving large data between JS and WASM can dominate cost; prefer typed arrays, avoid frequent crossings.

### Interview‑Style Questions (Practice)

- How do you attach a debugger to a remote Node process safely? Why are public inspector ports dangerous?
- Show robust error handling around `fetch` including timeouts (AbortController).
- Compare polling, Server‑Sent Events, and WebSockets for real‑time updates in Node.
- What changes between development and production for a Node service (env, logging, security, process management)?
- How do you capture and analyze a CPU profile and a heap snapshot?
- When would you choose WebAssembly in Node, and what are common performance traps?

### References

- Node.js — Debugging: https://nodejs.org/en/learn/getting-started/debugging
- Node.js — Fetch: https://nodejs.org/en/learn/getting-started/fetch
- Node.js — WebSocket: https://nodejs.org/en/learn/getting-started/websocket
- Node.js — Dev vs Prod: https://nodejs.org/en/learn/getting-started/nodejs-the-difference-between-development-and-production
- Node.js — Profiling: https://nodejs.org/en/learn/getting-started/profiling
- Node.js — WebAssembly: https://nodejs.org/en/learn/getting-started/nodejs-with-webassembly



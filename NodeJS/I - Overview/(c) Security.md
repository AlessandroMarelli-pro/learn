### Node.js Security — Teacher’s Synthesis

Source: https://nodejs.org/en/learn/getting-started/security-best-practices

### 1) Threats to keep in mind

- DoS on HTTP servers (slowloris, bad clients, missing error handlers)
- DNS rebinding against `--inspect` (local inspector exposed indirectly)
- Sensitive info exposure during publish (`.npmignore`, files allowlist)
- HTTP request smuggling (front proxy vs app parsing differences)
- Timing attacks (leaking info via response time)
- Malicious third‑party modules and supply chain risks
- Memory access issues; secure heap
- Monkey patching built‑ins at runtime
- Prototype pollution via `__proto__`, `constructor`, `prototype`
- Uncontrolled search path (module resolution surprises)
- Experimental features in prod

### 2) Core mitigations (actionable)

- Front with a reverse proxy (timeouts, rate limits, caching, blacklists).
- Set server timeouts: `headersTimeout`, `requestTimeout`, `timeout`, `keepAliveTimeout`. Limit sockets: `agent.maxSockets`, `agent.maxTotalSockets`, `server.maxRequestsPerSocket`.
- Always add error handlers on sockets/servers to avoid crashes.
- Keep inspector local only; never bind to 0.0.0.0. Use SSH tunnels for remote debugging.
- Control publish contents: use `npm publish --dry-run`, maintain `.npmignore`/`files` allowlist, unpublish if leaked.
- Avoid `insecureHTTPParser`; normalize ambiguous requests at proxy; prefer HTTP/2 end‑to‑end.
- Use constant‑time comparisons for secrets; avoid branching on secret length/content.
- Harden supply chain: `npm ci`, lockfiles, pin exact versions, `--ignore-scripts`, CI `npm audit` (plus tools like Socket), review typos.
- Use `--secure-heap=<bytes>` on non‑Windows as needed; avoid shared machines for prod.
- Freeze intrinsics: run with `--frozen-intrinsics`; optionally `Object.freeze(globalThis)` to prevent global replacement.
- Prevent prototype pollution: validate inputs (JSON Schema), `Object.create(null)`, `Object.freeze(MyObj.prototype)`, `--disable-proto`, prefer `Object.hasOwn(obj, k)` and avoid `Object.prototype` methods.
- Enforce module policy/integrity (experimental): `--experimental-policy=policy.json` plus `--policy-integrity`.

```js
// DoS prevention: add error handler on raw sockets
const net = require('node:net');
const server = net.createServer((socket) => {
  socket.on('error', console.error); // prevents crash
  socket.write('Echo\r\n');
  socket.pipe(socket);
});
server.listen(5000, '0.0.0.0');
```

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Exposing `--inspect` publicly or relying on SIGUSR1 to toggle inspector in prod increases attack surface. Keep localhost + tunnels only.
> - Publishing secrets due to missing ignore rules; always dry‑run and maintain allowlists.
> - Enabling `insecureHTTPParser` or accepting ambiguous requests can enable request smuggling.
> - Trusting dependency ranges: transitive deps can update silently; use lockfiles/pinning and `npm ci`.
> - Assuming constant‑time comparisons with `===`; use timing‑safe utilities when comparing secrets.
> - Prototype pollution via deep merges; avoid unsafe merges, disable `__proto__`, create null‑prototype objects for untrusted data.
> - Relying on experimental flags in production; they may change or break.

### Interview‑Style Questions (Practice)

- Describe slowloris and how you’d harden a Node HTTP server against it.
- How do DNS rebinding attacks target the Node inspector? List mitigations.
- What steps prevent leaking secrets when publishing to npm?
- Explain HTTP request smuggling and how Node + proxy mismatches cause it.
- How do you mitigate supply chain risks across direct and transitive dependencies?
- Show two ways to reduce prototype pollution risk in your codebase.

### References

- Node.js — Security Best Practices: https://nodejs.org/en/learn/getting-started/security-best-practices

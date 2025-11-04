### Node.js — Teacher’s Synthesis

Sources: [Introduction to Node.js](https://nodejs.org/en/learn/getting-started/introduction-to-nodejs), [Differences: Node.js vs Browser](https://nodejs.org/en/learn/getting-started/differences-between-nodejs-and-the-browser), [V8 engine](https://nodejs.org/en/learn/getting-started/the-v8-javascript-engine), [npm package manager](https://nodejs.org/en/learn/getting-started/an-introduction-to-the-npm-package-manager), [ES2015+ and beyond](https://nodejs.org/en/learn/getting-started/ecmascript-2015-es6-and-beyond)

### 1) What Node.js is

- Open‑source, cross‑platform JavaScript runtime built on the V8 engine. Runs JS outside the browser, optimized for non‑blocking I/O and high concurrency (single process, event loop, async APIs).  
  See: Introduction to Node.js

```js
// Minimal HTTP server (CommonJS)
const { createServer } = require('node:http');
const server = createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World');
});
server.listen(3000, '127.0.0.1', () => {
  console.log('Server running at http://127.0.0.1:3000/');
});
```

### 2) V8: the engine under the hood

- V8 parses and executes JS with just‑in‑time (JIT) compilation for performance; it’s independent of the browser, enabling Node.js outside the DOM.  
  See: The V8 JavaScript Engine

### 3) Node.js vs the browser — different runtimes

- Browser: has `window`, `document`, Web APIs (DOM, Cookies). No filesystem/network sockets by default.
- Node.js: has core modules (`fs`, `http`, `net`, `crypto`), no DOM. You control the runtime version (use modern ES features as supported by your Node version).
- Module systems: Node supports CommonJS (`require/module.exports`) and ES Modules (`import/export`). Browsers use ES Modules.  
  See: Differences between Node.js and the Browser

### 4) npm: packages, scripts, and semver

- npm is the default package manager and registry.
- Key commands: `npm init`, `npm install <pkg>`, `npm update`, `npm run <script>`.
- `package.json` defines dependencies and scripts; caret/tilde ranges manage updates.  
  See: An introduction to the npm package manager

```json
{
  "name": "my-app",
  "scripts": {
    "dev": "node server.js",
    "test": "node --test"
  },
  "dependencies": {
    "express": "^4.19.0"
  }
}
```

### 5) ES2015+ and language features in Node

- Modern JS features (classes, modules, async/await, destructuring, etc.) are available per Node version support; you can choose features by choosing Node LTS/current releases.  
  See: ECMAScript 2015 (ES6) and beyond

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Blocking the event loop (e.g., CPU‑heavy sync work) degrades concurrency; offload to workers or native add‑ons.
> - Assuming browser globals (`window`, `document`) exist in Node — they don’t. Use appropriate Node APIs or libraries.
> - Mixing CommonJS and ESM incorrectly (e.g., `require` in ESM without dynamic import) causes module resolution errors; pick a module type and configure properly.
> - Relying on unspecified Node versions can break features; pin engines in `package.json` or use an LTS with an `.nvmrc`.
> - Ignoring semver ranges/lockfiles leads to drift; use lockfiles, respect breaking changes (`major`), and test updates.
> - Treating async I/O as synchronous (missing `await`/`then`, unhandled rejections) leads to subtle bugs; always handle errors.

### Interview‑Style Questions (Practice)

- Explain how Node.js achieves high concurrency with a single process and event loop. When would you add worker threads or processes?
- Contrast Node.js and the browser environments (APIs, security model, module systems).
- When do you choose CommonJS vs ES Modules in Node, and what configuration changes?
- What does V8’s JIT do for performance, and how does it relate to Node’s runtime?
- Show how you’d manage dependencies and scripts with npm; explain caret (`^`) vs tilde (`~`) ranges and lockfiles.
- How do you prevent blocking the event loop in a file processing service? Provide two approaches.

### References

- Node.js — Introduction: https://nodejs.org/en/learn/getting-started/introduction-to-nodejs
- Node.js — Differences vs Browser: https://nodejs.org/en/learn/getting-started/differences-between-nodejs-and-the-browser
- Node.js — V8 engine: https://nodejs.org/en/learn/getting-started/the-v8-javascript-engine
- Node.js — npm package manager: https://nodejs.org/en/learn/getting-started/an-introduction-to-the-npm-package-manager
- Node.js — ES2015 and beyond: https://nodejs.org/en/learn/getting-started/ecmascript-2015-es6-and-beyond

### JavaScript Modules — Teacher’s Synthesis

Source: [MDN — JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)

### 1) Why modules

- **Purpose**: Split code into reusable files with explicit dependencies; enable tree shaking, lazy loading, clearer boundaries.
- **Support**: Natively in modern browsers (via `<script type="module">`) and in Node.js (ESM), alongside legacy systems (CJS/AMD).

### 2) Exporting & importing

- **Named exports**: A module can export many bindings by name; import must match those names (can rename).

```js
// math.js
export const PI = 3.14159;
export function area(r) { return PI * r * r; }

// consumer
import { PI, area as circleArea } from './math.js';
```

- **Default export**: One default export per module; importers choose its local name.

```js
// logger.js
export default function log(msg) { console.log(msg); }

// consumer
import log from './logger.js';
```

- **Export lists and re‑exports**:

```js
export { PI as Tau } from './math.js'; // re‑export with rename
export * from './shapes.js';           // re‑export all named exports
```

### 3) Using modules in HTML and Node

- **Browser**:

```html
<script type="module" src="/main.js"></script>
```

- **Node.js**: Use `.mjs` or set `"type": "module"` in `package.json`. Prefer ESM `import`/`export` in new code.

### 4) Import maps (browser)

- Map bare specifiers to URLs without a bundler.

```html
<script type="importmap">
{
  "imports": {
    "lodash": "/vendor/lodash-es/lodash.js"
  }
}
</script>
<script type="module">
  import _ from 'lodash';
  // ...
  
</script>
```

### 5) Dynamic import and module objects

- **Dynamic import** returns a promise; useful for code‑splitting and conditional loading.

```js
const { heavyFn } = await import('./heavy.js');
heavyFn();
```

- **Module namespace objects** gather all exports:

```js
import * as math from './math.js';
math.area(2);
```

### 6) Other ES module semantics

- **Hoisting**: Import declarations are hoisted; bindings are live (reflect updates from exporters).
- **Top‑level await**: Allowed in modules; importing module waits for awaited completion (can cascade delays).
- **Cyclic imports**: Supported; modules execute once; exports may be temporarily undefined during initialization order.
- **Modules vs classic scripts**: Modules are strict mode by default, deferred, and use separate scope per file.

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Default vs named export confusion: `import foo from` is for default; named require braces.  
> - Circular deps can observe partially initialized bindings — design cycles carefully or refactor shared state.  
> - Top‑level `await` delays importers; avoid in widely shared libraries or gate with dynamic import.  
> - Mixed systems (ESM vs CommonJS): interop requires care; Node’s `require` cannot import ESM directly without dynamic `import()`.  
> - Bare specifiers in browsers need import maps or bundlers; otherwise use relative/absolute URLs.  
> - Cross‑origin module loads require correct CORS/MIME types; `.mjs` may need server config.

### Interview‑Style Questions (Practice)

- Contrast default and named exports; how do you re‑export and rename?  
- Explain live bindings and how they differ from copying values.  
- When would you use dynamic `import()`? Show a code‑splitting example.  
- What changes with `<script type="module">` vs classic scripts (scope, defer, strict mode)?  
- How do import maps work and when would you prefer them over a bundler?  
- What are safe patterns to handle cyclic dependencies between modules?  
- How does top‑level `await` impact module graphs and startup time?  
- Describe ESM/CommonJS interop issues in Node.js and migration strategies.

### References

- MDN — JavaScript modules: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules


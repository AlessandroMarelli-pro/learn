### Publishing a Node.js Package — Synthesis

Source: [Publishing a package](https://nodejs.org/en/learn/modules/publishing-a-package)

---

### Big idea

- Choose one distribution format where possible: **CJS** or **ESM**. Avoid dual dist unless necessary (dual-package hazard).
- Use `package.json` fields precisely: `type`, `main`, `exports`, `engines`.

---

### Pick your approach

- CJS source → CJS dist (simple, broadly consumable)

```json
{
  "name": "cjs-source-and-distribution"
  // "main": "./index.js"
}
```

- ESM source → ESM dist (set module type)

```json
{
  "name": "esm-source-and-distribution",
  "type": "module"
  // "main": "./index.js"
}
```

- CJS source → ESM-only dist (e.g., legacy code, modern target)

```json
{
  "name": "cjs-source-with-esm-distribution",
  "main": "./dist/index.mjs"
}
```

- Dual dist (only if required). Options include property-based named exports in CJS, ESM wrappers, or two full distributions. Configure `exports` carefully.

---

### Key `package.json` fields

- `type`: toggles default `.js` interpretation (`commonjs` default; `module` for ESM).
- `main`: legacy entry point; simple but exposes internals if used alone.
- `exports`: define public entry points per loader; restricts deep imports, improves encapsulation.
- `engines`: communicate supported Node versions; some PMs enforce it at install.

Notes:

- `exports.require` ≠ CJS and `exports.import` ≠ ESM; they select files for require/import, not the file format.
- `.mjs` is always treated as ESM (overrides `type`).

---

### Making CJS named exports detectable

```js
// Avoid reassigning module.exports to a single object
// Prefer adding properties so static analysis can detect named exports
module.exports.foo = function foo() {};
module.exports.bar = function bar() {};
```

---

### Minimal `exports` examples

```json
{
  "name": "pkg",
  "exports": {
    ".": "./index.js"
  }
}
```

Dual mapping (illustrative; ensure file formats align with `type`/extensions):

```json
{
  "name": "pkg",
  "exports": {
    ".": {
      "import": "./dist/esm/index.js",
      "require": "./dist/cjs/index.js",
      "default": "./dist/cjs/index.js"
    }
  }
}
```

---

> ⚠️ **Pitfalls**
>
> - **Dual-package hazard**: Shipping both CJS and ESM can load two instances in one app (state split, `instanceof` fails).
> - **Misusing `type`**: `"type": "module"` makes `.js` ESM; don’t point CJS files via `main`/`exports.require` without proper extensions.
> - **Assuming `import`/`require` fields set format**: They choose entrypoints, not module kind; use `.mjs` or `type` to control format.
> - **Breaking deep imports**: Adding `exports` hides internals unless subpaths are explicitly exported (can be breaking).
> - **Overexposing internals**: Using only `main` can let consumers deep-reach; `exports` caps surface area.
> - **Unspecified engines**: Omit `engines` and users on old Node may get cryptic failures.

---

### Interview-ready talking points

- CJS vs ESM distribution: pros/cons, why choose one format.
- Role of `type`, `main`, and `exports`; how file extensions affect module kind.
- Dual-package hazard explanation and how to avoid it.
- Why `exports` is preferable to `main` for API stability.
- Strategies to make CJS named exports discoverable by ESM importers.

---

### Quick checklist

- [ ] Single-format dist unless dual is absolutely needed.
- [ ] `type`/extensions align with actual file formats (`.mjs` for ESM when needed).
- [ ] Use `exports` to define public surface; avoid deep import reliance.
- [ ] Provide `engines` for Node compatibility.
- [ ] If dual dist, verify no duplicate instances in consumer graphs.

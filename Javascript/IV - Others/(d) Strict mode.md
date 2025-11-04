### Strict Mode — Teacher’s Synthesis

Source: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode

### 1) What strict mode is

- Opt‑in to a safer subset of JS that turns silent errors into exceptions, simplifies `this`/name resolution, and blocks hazardous syntax. Modules and classes are strict by default.

### 2) How to enable

- Whole script: place the directive at top of the file.

```js
'use strict';
```

- Per function: first statement in the function body (only for simple parameter lists).

```js
function f() {
  'use strict';
  return 1;
}
```

- Implicit: ES modules and class bodies are always strict.

### 3) Key behavior changes (high‑impact)

- Assigning to undeclared vars throws (`ReferenceError`) instead of creating globals.
- Writing to non‑writable props, getter‑only props, or non‑extensible objects throws.
- Deleting non‑configurable props throws.
- Duplicate parameter names are syntax errors.
- Octal literals/escapes (legacy forms) are disallowed.
- `this` in standalone functions is `undefined` (no substitution to global object).
- `eval`/`arguments` are safer: cannot be re‑bound or used as identifiers; `eval` has its own scope.

```js
'use strict';
mistyped = 1; // ReferenceError (no implicit global)

const o = {};
Object.defineProperty(o, 'x', { value: 1, writable: false });
o.x = 2; // TypeError
```

### 4) Modules/classes are strict automatically

- No directive needed; treat top‑level `this` as `undefined` in modules; class bodies are strict code regions.

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Adding `"use strict"` to functions with default/rest/destructured params is a syntax error.
> - Libraries relying on sloppy features (e.g., implicit globals, octal literals) will break under strict mode.
> - In strict mode, `this` is `undefined` in simple calls — code expecting `globalThis` will fail.
> - Reassignments to `NaN`, `Infinity`, `undefined`, or non‑writable props throw instead of silently failing.
> - `delete` on non‑configurable props throws; ensure property descriptors allow it before deleting.

### Interview‑Style Questions (Practice)

- How does strict mode change the value of `this` in a standalone function?
- Show an example where strict mode turns a silent failure into a thrown error.
- Why are modules/classes strict by default, and what implications does that have for library code?
- Which parameter lists allow function‑level `"use strict"`?
- How do `eval` and `arguments` semantics differ under strict mode?

### References

- MDN — Strict mode: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode

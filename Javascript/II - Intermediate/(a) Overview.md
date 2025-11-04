### Intermediate Overview — Language, Data Structures, Equality, Enumerability

Sources: [Language overview](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Language_overview), [Data structures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Data_structures), [Equality comparisons & sameness](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Equality_comparisons_and_sameness), [Enumerability & ownership](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Enumerability_and_ownership_of_properties)

### 1) Language overview — quick orientation

- **Paradigms**: Multi‑paradigm (functional, OO via prototypes/classes, imperative). Dynamic typing; first‑class functions; modules.
- **Numbers**: `number` is IEEE‑754 double; `bigint` for arbitrary‑precision ints.
- **Strings**: UTF‑16; template literals with interpolation and multiline.
- **Objects**: Dictionaries with prototype chains; arrays are objects with numeric keys and special behaviors.
- **Async**: Promises and `async`/`await`; event loop concurrency model.

```js
const n = 0.1 + 0.2; // 0.30000000000000004 (IEEE‑754)
const big = 123n * 2n; // 246n
```

### 2) Data structures — choosing the right container

- **Primitives**: `string`, `number`, `bigint`, `boolean`, `undefined`, `symbol`, `null`.
- **Objects**: Plain objects `{}`, arrays `[]`, `Map`/`Set`, `WeakMap`/`WeakSet`, typed arrays, `Date`, `RegExp`, `Error`.
- **Guidance**:
  - Use plain objects for fixed schemas; `Map` for dynamic keys or frequent add/remove with any key type.
  - Use arrays for ordered lists; `Set` for uniqueness/membership tests.
  - Use typed arrays for raw numeric buffers; `DataView` for endianness control.

```js
const m = new Map([[{ id: 1 }, 'meta']]); // object keys allowed
const s = new Set([1, 1, 2]); // {1,2}
```

### 3) Equality, sameness, and comparisons

- **Strict equality `===`**: No coercion; safest default.
- **Loose equality `==`**: Coerces; many special cases (e.g., `'' == 0`, `null == undefined`).
- **`Object.is(a, b)`**: SameValue semantics; distinguishes `+0` and `-0`, treats `NaN` equal to itself.
- **SameValueZero**: Used by `Set`, `Map`, `Array.prototype.includes` — like `Object.is` but `+0` and `-0` are equal.

```js
NaN === NaN; // false
Object.is(NaN, NaN); // true
0 === -0; // true
Object.is(0, -0); // false
['a'].includes('a'); // true (SameValueZero)
```

### 4) Enumerability & ownership of properties

- **Own vs inherited**: Own properties live directly on the object; inherited come from the prototype chain.
- **Enumerable**: Controls whether a property appears in `for...in` and is listed by `Object.keys` (own + enumerable).
- **Common APIs**:
  - `Object.keys(obj)`: own enumerable keys
  - `Object.getOwnPropertyNames(obj)`: own keys incl. non‑enumerable (string keys)
  - `Object.getOwnPropertySymbols(obj)`: own symbol keys
  - `Object.getOwnPropertyDescriptor(obj, key)`: descriptor (writable, enumerable, configurable, get/set)
  - `Object.hasOwn(obj, key)` or `Object.prototype.hasOwnProperty.call(obj, key)`

```js
const base = { a: 1 };
const o = Object.create(base, {
  x: { value: 42, enumerable: false },
  y: { value: 7, enumerable: true },
});
Object.keys(o); // ['y']
'a' in o; // true (inherited)
```

> ⚠️ **Pitfalls (Read Carefully)**
>
> - IEEE‑754 surprises (`0.1 + 0.2 !== 0.3`); avoid equality on floats; prefer tolerances.
> - Mixing `number` and `bigint` in arithmetic throws; convert explicitly.
> - `==` coercion edge cases (`''==0`, `false==0`, `null==undefined`); prefer `===`.
> - `NaN` never equals itself with `===`; use `Number.isNaN` or `Object.is`.
> - `for...in` iterates enumerable keys including inherited; not for arrays — use `for...of` or array methods.
> - Non‑enumerable properties won’t appear in `Object.keys`; use descriptors or `getOwnPropertyNames` when needed.
> - Symbols are skipped by most key collectors; use `getOwnPropertySymbols` when you rely on symbol keys.

### Interview‑Style Questions (Practice)

- Compare `===`, `==`, `Object.is`, and SameValueZero; give examples.
- When should you choose `Map` over a plain object? `Set` over an array?
- How do you iterate own enumerable keys only? How to include non‑enumerable and symbols?
- Why is `for...in` risky on arrays and instances with prototypes? Alternatives?
- Show a safe float comparison helper.
- Explain how prototype inheritance affects property lookup and shadowing.

### References

- MDN — Language overview: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Language_overview
- MDN — Data structures: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Data_structures
- MDN — Equality comparisons & sameness: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Equality_comparisons_and_sameness
- MDN — Enumerability & ownership: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Enumerability_and_ownership_of_properties

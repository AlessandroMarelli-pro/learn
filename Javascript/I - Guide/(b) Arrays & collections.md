### Arrays & Collections — Teacher’s Synthesis

Sources: [Indexed collections — sparse arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Indexed_collections#sparse_arrays), [Keyed collections](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Keyed_collections)

### 1) Indexed collections (Arrays) — focus on sparse arrays

- **Array basics**: Ordered, zero‑based indices; `length` is highest index + 1 (not element count). Creation via `[]`, `Array()`, `Array.of()`. See MDN Indexed collections.
- **Sparse arrays**: Arrays with “holes” (missing elements). Holes are not the same as `undefined` values — the index doesn’t exist.

```js
const dense = [1, 2, 3];
const sparse = [];
sparse[2] = 'x'; // indices 0 and 1 are holes
0 in sparse; // false (hole)
2 in sparse; // true
```

- **Length & deletion**:
  - Setting `arr.length` can truncate or create holes.
  - `delete arr[i]` removes the element but leaves a hole (index missing).

```js
const a = [10, 20, 30];
delete a[1]; // a = [10, <hole>, 30]
a.length; // 3 (unchanged)
```

- **Iteration behavior** (key differences):
  - `for...of`, `Array.prototype.forEach`, `map`, `filter` skip holes.
  - `for...in` iterates existing enumerable indices (still skips holes but is not for arrays).
  - `Object.keys(a)` lists existing indices only.

```js
[, 1, 2].forEach((v) => console.log(v)); // logs 1, 2 (hole skipped)
for (const v of [, 1, 2]) console.log(v); // 1, 2
```

- **Serialization**:
  - `JSON.stringify` converts holes to `null` to preserve positions: `JSON.stringify([ , 1]) // "[null,1]"`.

> ⚙️ Practical: Prefer creating dense arrays via `Array.from({ length: n }, (_, i) => i)` or `new Array(n).fill(0)` over assigning far indexes or using `delete`.

### 2) Keyed collections — Map, Set, WeakMap, WeakSet

- **Map**: Key–value pairs with keys of any type, ordered by insertion. Reliable `size`, iteration via `map.keys()`, `map.values()`, `map.entries()`.

```js
const m = new Map();
const objKey = { id: 1 };
m.set(objKey, 'data');
m.get(objKey); // 'data'
m.has(objKey); // true
for (const [k, v] of m) {
  /* ... */
}
```

- **Set**: Unique values (any type), insertion‑ordered.

```js
const s = new Set([1, 2, 2, 3]);
s.size; // 3
s.has(2); // true
```

- **WeakMap**: Keys must be objects; held weakly (no strong GC roots). Not iterable; no `size`. Useful for private per‑object data.

```js
const wm = new WeakMap();
const key = {};
wm.set(key, { hidden: true });
// if `key` is GC’d elsewhere, its entry can disappear
```

- **WeakSet**: Stores objects (no primitives), weakly held; not iterable, no `size`.

```js
const ws = new WeakSet();
const o = {};
ws.add(o);
ws.has(o); // true
```

> When to choose:
>
> - Use **Map** instead of object literals when keys aren’t strings/symbols, you need ordered iteration, or frequent add/removes.
> - Use **Set** to deduplicate or test membership.
> - Use **WeakMap/WeakSet** to attach metadata without preventing garbage collection.

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Sparse arrays cause surprising iteration and performance characteristics; avoid creating holes with `delete` or by jumping indices.
> - Don’t use `for...in` on arrays; it’s for object keys and can traverse inherited props. Prefer `for...of` or array methods.
> - `JSON.stringify` of sparse arrays yields `null` placeholders; data shape may change unexpectedly.
> - `Map` and `Set` are not JSON‑serializable by default; convert via `Array.from(map.entries())` or spread.
> - `WeakMap`/`WeakSet` are non‑iterable and have no `size` — you cannot inspect contents for debugging or enumeration.
> - Using objects as keys in plain objects coerces keys to `"[object Object]"`; use `Map` for object keys.
> - Typed arrays are not normal arrays: fixed length, no `push`/`pop`, and `Array.isArray(typed)` is `false`.
> - Endianness matters when using `DataView`; the same bytes can read differently on little vs big endian operations.

### 3) Typed arrays — binary data at speed

- **What**: Array‑like views over raw binary buffers for numeric data. Ideal for media processing, networking, and performant numeric operations. See MDN: [Typed arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Typed_arrays)
- **Buffers**:
  - `ArrayBuffer`: owns a chunk of memory; can be transferred/resized in modern runtimes.
  - `SharedArrayBuffer`: shared across threads; coordinate with `Atomics`.
- **Views**:
  - TypedArray views include `Int8Array`, `Uint8Array`, `Uint8ClampedArray`, `Int16Array`, `Uint16Array`, `Int32Array`, `Uint32Array`, `BigInt64Array`, `BigUint64Array`, `Float32Array`, `Float64Array`.
  - `DataView`: low‑level reads/writes with explicit endianness.

```js
// One buffer, multiple typed views
const buf = new ArrayBuffer(8);
const u32 = new Uint32Array(buf); // 2 elements of 4 bytes
const f64 = new Float64Array(buf); // 1 element of 8 bytes
u32[0] = 0xffffffff;
f64[0]; // observes the same 8 bytes as a float64

// DataView gives endianness control
const dv = new DataView(buf);
dv.setUint16(0, 0x1234, true); // little‑endian write
dv.getUint16(0, false); // big‑endian read => different value
```

- **Key differences vs Arrays**:
  - Fixed length; no structural methods like `push`/`pop`/`splice` that change size.
  - Indices are always contiguous (no holes) and elements are clamped/wrapped per type.
  - `Array.isArray(new Uint8Array())` is `false`; use `ArrayBuffer.isView(view)` to detect views.

### Interview‑Style Questions (Practice)

- How do holes in a sparse array differ from `undefined` values? How do common iterators treat them?
- What are safe ways to initialize an array of length N without holes?
- Why prefer `Map` over a plain object for dynamic keys or frequent insertions/deletions?
- Explain when to choose `Set` vs an array plus `includes`. Complexity and semantics?
- Why can’t you iterate a `WeakMap`? What problem does it solve despite that limitation?
- How would you serialize a `Map` to JSON and reconstruct it later?
- Why isn’t `Array.isArray(new Uint8Array())` true? How do typed arrays differ from arrays?
- When do you choose `TypedArray` vs `DataView`? Show an endianness example.
- How do `ArrayBuffer` and `SharedArrayBuffer` differ, and when do `Atomics` matter?

### References

- MDN — Indexed collections (sparse arrays): https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Indexed_collections#sparse_arrays
- MDN — Keyed collections: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Keyed_collections
- MDN — Typed arrays: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Typed_arrays

### Meta Programming — Teacher’s Synthesis

Source: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Meta_programming

### 1) What is meta programming?

- Program at the language’s meta level by intercepting core operations (get/set, has, call, construct, enumerate, etc.).
- JavaScript exposes this via `Proxy` (interception) and `Reflect` (standardized, low‑level methods mirroring language ops).

### 2) Proxies: targets, handlers, and traps

- A `Proxy` wraps a `target` and uses a `handler` object with trap methods to override behavior.

```js
const handler = {
  get(target, key, receiver) {
    if (!(key in target)) return 42; // default for missing
    return Reflect.get(target, key, receiver); // delegate
  },
  set(target, key, value, receiver) {
    if (key === 'id' && typeof value !== 'number')
      throw new TypeError('id must be number');
    return Reflect.set(target, key, value, receiver);
  },
};

const user = new Proxy({}, handler);
user.name = 'Ava';
user.missing; // 42
```

- Common traps: `get`, `set`, `has`, `deleteProperty`, `ownKeys`, `getOwnPropertyDescriptor`, `defineProperty`,  
  `getPrototypeOf`/`setPrototypeOf`, `isExtensible`/`preventExtensions`, `apply` (call functions), `construct` (with `new`).

### 3) Reflect: the companion API

- `Reflect.*` provides standardized counterparts for operations and returns booleans instead of throwing where appropriate.

```js
const obj = {};
Reflect.defineProperty(obj, 'x', { value: 1, configurable: true });
Reflect.has(obj, 'x'); // true
Reflect.ownKeys(obj); // ['x'] (incl. symbols, non‑enumerables)
```

### 4) Revocable proxies

- `Proxy.revocable(target, handler)` returns `{ proxy, revoke }`; calling `revoke()` disables the proxy.

```js
const { proxy, revoke } = Proxy.revocable(
  { secret: 1 },
  { get: () => 'redacted' },
);
proxy.secret; // 'redacted'
revoke();
// Further operations now throw TypeError
```

### 5) Design uses

- Validation and invariants, access control, lazy initialization, caching/memoization, virtualized collections, DOM or API façades, deprecations/logging.

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Invariants: Traps must honor language invariants (e.g., non‑configurable props can’t be reported as absent); violations throw `TypeError`.
> - Performance: Proxies add overhead and can disable engine optimizations; avoid wrapping hot paths.
> - Identity and leaks: Returning wrapped objects inconsistently breaks `===` expectations; decide a stable wrapping strategy.
> - `this` forwarding: Use `Reflect.get`/`set` with the `receiver` to preserve correct `this` in accessors.
> - Enumerations: Implement `ownKeys` + `getOwnPropertyDescriptor` coherently; mismatches cause errors.
> - Security: Proxies don’t sandbox by themselves; be explicit about what operations are allowed.

### Interview‑Style Questions (Practice)

- Explain `Proxy` parts: target, handler, traps. Show a `get` trap that delegates using `Reflect`.
- What invariants must traps respect? Give an example that would throw.
- When would you use `apply` and `construct` traps? Show minimal examples.
- Contrast `Reflect.defineProperty` with `Object.defineProperty`. Why prefer `Reflect` in trap implementations?
- What’s a revocable proxy and when is it useful?
- How can proxies impact performance and equality semantics? Mitigations?

### References

- MDN — Meta programming: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Meta_programming

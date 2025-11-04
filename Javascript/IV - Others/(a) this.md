### `this` — Teacher’s Synthesis

Source: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this#specifications

### 1) What `this` means

- `this` is the call‑site bound receiver for a function. Its value depends on HOW a function is invoked, not where it’s defined. In strict mode, `this` can be any value; in non‑strict, it’s always an object due to substitution rules.

### 2) Binding rules by call form

- **Method call (`obj.fn()`)**: `this` → `obj` (the value before the dot at call time).
- **Standalone call (`fn()`)**: strict → `undefined`; non‑strict → `globalThis` (after substitution).
- **call/apply/bind**: `fn.call(ctx, ...)`, `fn.apply(ctx, args)` set `this` for that call; `bind(ctx)` returns a permanently bound function.
- **Arrow functions**: capture `this` lexically from the enclosing scope at definition time; cannot be changed by `call/apply/bind`.
- **Class methods**: behave like regular functions (not auto‑bound). Static methods receive the class as `this` when called as `C.m()`.

```js
function getThis() {
  return this;
}
const obj = { getThis };
obj.getThis() === obj; // true (method)
getThis() === undefined; // true in strict mode

const bound = getThis.bind({ x: 1 });
bound(); // { x: 1 }

const o = { x: 1, arrow: () => this };
o.arrow() === this; // true: arrow captured outer `this`, not `o`
```

### 3) Prototype chain and `this`

- Method lookup uses the prototype chain, but `this` is the receiver the method was called on, not where it was found.

```js
const base = {
  who() {
    return this.tag;
  },
};
const a = Object.create(base);
a.tag = 'A';
const b = Object.create(base);
b.tag = 'B';
base.who.call(a); // 'A'
b.who(); // 'B' (method found on base, `this` is b)
```

### 4) Special cases

- Primitives as receivers: in non‑strict, boxed (objectified); in strict, remain primitives.
- Class field arrows: `handler = () => { ... }` auto‑capture `this` per instance, useful for callbacks.
- DOM/event listeners: `this` often set to the event target element in classic listeners; in modules/strict contexts prefer explicit `event.currentTarget`.

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Assuming `this` equals the object that defines the method; it equals the object used to CALL the method.
> - Losing `this` when extracting methods (`const f = obj.m; f()`); fix with `bind` or arrow class fields.
> - Expecting `call/apply/bind` to affect arrow functions — they do not.
> - In non‑strict functions, `this` may become `globalThis` unintentionally; enable strict mode/modules.
> - Inside prototypes/getters/setters, using `Reflect.get/Set` without `receiver` can break accessor `this` behavior.

### Interview‑Style Questions (Practice)

- Explain how `this` is determined for `obj.fn()`, `fn()`, `new Fn()`, and arrow functions.
- Why does `({ m: obj.m }).m()` produce a different `this` than `obj.m()`?
- Show two ways to preserve `this` when passing a method as a callback.
- Why doesn’t `bind` change `this` for arrow functions? When are arrow class fields beneficial?
- Demonstrate `this` along a prototype chain and with `call/apply`.

### References

- MDN — `this`: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this#specifications

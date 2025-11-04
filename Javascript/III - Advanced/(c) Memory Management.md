### Memory Management — Teacher’s Synthesis

Source: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Memory_management

### 1) Memory lifecycle in JS

- **Allocate**: Happens implicitly when you create values (literals, objects, arrays, functions) or call APIs that allocate (e.g., `new Date()`).
- **Use**: Read/write variables and object properties; pass values to functions.
- **Release**: Garbage collector (GC) reclaims memory that’s no longer reachable.

```js
const o = { a: 1 };       // allocation
let r = o;                 // reference
r = null;                  // last reference dropped → eligible for GC
```

### 2) Garbage collection basics

- Modern engines use reachability analysis (mark-and-sweep variants, generational, incremental, concurrent).
- A value is collectible when it’s no longer reachable from GC roots (e.g., global scope, currently executing stack, closures, DOM roots).
- Reference counting alone fails on cycles; reachability handles cycles.

### 3) What creates references (keeps memory alive)

- Variables and closures capturing objects.
- Object graphs: properties referencing other objects; prototypes; DOM references.
- Event listeners, timers, caches, singletons, global registries.

```js
function makeCache() {
  const map = new Map();
  return {
    set(k, v) { map.set(k, v); },
    get(k) { return map.get(k); },
  };
} // map stays alive while the returned object is reachable
```

### 4) Avoiding leaks in practice

- Unsubscribe listeners (`removeEventListener`), clear timers/intervals, close observers/workers.
- Null out large references when done, or scope them tightly.
- Use `WeakMap`/`WeakSet` for metadata keyed by objects so entries can be GC’d.
- Prefer local variables over globals/singletons for temporary structures.

```js
const wm = new WeakMap();
function attach(o, data) { wm.set(o, data); } // data freed when o is GC’d
```

### 5) Measuring and tuning

- Use browser devtools memory profiles: heap snapshots, allocation timelines, detached DOM nodes.
- Watch long-lived closures and component lifecycles; ensure cleanup paths run on errors/unmounts.
- In Node, monitor RSS/heap usage; use heap snapshots and `--inspect`.

### 6) Data structures aiding memory

- Typed arrays minimize overhead for numeric buffers.
- Strings are immutable; repeated concatenations may create intermediates—builders or arrays + `join` can help in hot paths.

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Forgotten listeners/timers keep entire object graphs alive.  
> - Accidental globals or long-lived singletons become implicit roots.  
> - Capturing large objects in closures unnecessarily prevents collection.  
> - Maps/Sets with object keys/values grow unbounded; use `WeakMap/WeakSet` for ephemeral associations.  
> - DOM–JS reference cycles still collect if unreachable, but detached DOM with references in JS will leak until references are cleared.  
> - Relying on finalizers for critical cleanup is unsafe; they may not run promptly or at all.

### Interview‑Style Questions (Practice)

- Explain reachability-based GC and why reference counting alone is insufficient.  
- Give three common sources of leaks in web apps and how to mitigate them.  
- When should you choose `WeakMap`/`WeakSet` over `Map`/`Set`?  
- How do closures cause memory to stick around? Provide a safe pattern.  
- Describe how you’d detect and fix a leak involving detached DOM nodes.  
- What GC implications do long-lived caches and event buses have, and how would you design them?

### References

- MDN — Memory management: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Memory_management



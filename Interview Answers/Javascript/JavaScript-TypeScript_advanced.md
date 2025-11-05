# JavaScript/TypeScript - Advanced Level Answers

---

## 1. Explain the JavaScript memory model and how it affects performance optimization

**Memory structure:**
- **Stack:** Primitives, function calls (LIFO, fast, limited size)
- **Heap:** Objects, arrays, functions (larger, slower, garbage collected)

**Performance optimization strategies:**

**1. Object pooling:** Reuse objects instead of creating new ones (game particles, buffers)

**2. Avoid memory leaks:** Clear timers, remove listeners, nullify references

**3. WeakMap/WeakSet:** Garbage-collectible caches

**4. Typed arrays:** More memory efficient for numeric data

**5. String optimization:** Array join vs concatenation for many strings

**6. Hidden classes (V8):** Consistent object shapes for optimization

**7. Avoid `delete`:** Deoptimizes, set to `undefined` instead

---

## 2. How would you implement a custom event emitter from scratch?

**Core functionality:**
- `on()`: Register listener
- `off()`: Remove listener
- `emit()`: Trigger event
- `once()`: One-time listener

**Advanced features:**
- Max listeners warning
- Error handling
- Async emit
- Prepend listener
- Wildcard events

**Implementation considerations:**
- Store listeners in Map or object
- Handle listener removal during emit
- Support method chaining
- Memory leak prevention

---

## 3. Explain WeakMap and WeakSet and their practical use cases in preventing memory leaks

**WeakMap:**
- Keys must be objects (weakly referenced)
- Keys can be garbage collected
- Not enumerable (no size, no iteration)

**WeakSet:**
- Contains objects only
- Objects can be garbage collected
- Not enumerable

**vs Map/Set:** Regular Map/Set prevent garbage collection of entries.

**Use cases:**
1. **Private data:** Store private properties without preventing GC
2. **DOM metadata:** Attach data to DOM elements safely
3. **Caching:** Cache computed values without memory leaks
4. **Object tracking:** Track processed objects temporarily
5. **Circular reference detection**

---

## 4. What are the performance implications of different data structures in V8?

**Arrays:**
- Fast for indexed access
- Holey arrays (with gaps) slower than packed arrays
- Mixed types slower than single type

**Objects:**
- Fast property access if shape consistent
- Slower when adding properties dynamically
- Dictionary mode when too many properties

**Maps:**
- Better for frequent additions/deletions
- Preserves insertion order
- Slower access than objects for string keys

**Sets:**
- Fast membership checking
- No duplicates
- Better than array for uniqueness

**Typed arrays:**
- Fastest for numeric data
- Fixed type, contiguous memory
- Direct buffer access

**Optimization tips:** Consistent shapes, avoid holey arrays, use appropriate structure.

---

## 5. Explain how you would implement a custom Promise implementation

**Core components:**

**States:** Pending, Fulfilled, Rejected (immutable once settled)

**Methods to implement:**
- Constructor with executor function
- `.then(onFulfilled, onRejected)`
- `.catch(onRejected)`
- `.finally(onFinally)`

**Key behaviors:**
- Async resolution/rejection
- Chaining support
- Error propagation
- Microtask scheduling

**Challenges:**
- Handle synchronous resolution
- Queue callbacks if already settled
- Return new Promise from `.then()`
- Handle errors in callbacks

---

## 6. What are the trade-offs between using TypeScript's strict mode features?

**Strict mode includes:**
- `strictNullChecks`
- `strictFunctionTypes`
- `strictBindCallApply`
- `strictPropertyInitialization`
- `noImplicitAny`
- `noImplicitThis`

**Benefits:**
- Catches more bugs at compile time
- Better code quality
- Safer refactoring
- Self-documenting code

**Drawbacks:**
- More upfront effort
- Longer initial development
- Harder with third-party libs
- Migration cost for existing code

**Recommendation:** Enable from start, worth the investment for quality.

---

## 7. How would you design a type-safe event system in TypeScript?

**Requirements:**
- Type-safe event names
- Type-safe payloads per event
- Autocomplete support
- Compile-time errors for wrong types

**Approach:**
- Define event map interface
- Generic EventEmitter with constraint
- Mapped types for type safety
- Strict typing on emit/on methods

**Benefits:**
- Cannot emit undefined events
- Payload types enforced
- IDE autocomplete
- Refactoring safe

---

## 8. Explain conditional types and mapped types in TypeScript with advanced examples

**Conditional types:** `T extends U ? X : Y`
- Type-level if-else
- Used in utility types (`Extract`, `Exclude`, `NonNullable`)
- Can infer types with `infer` keyword

**Mapped types:** Transform properties of existing type
- Iterate over keys
- Change modifiers (readonly, optional)
- Built-in: `Partial`, `Required`, `Readonly`, `Pick`, `Omit`

**Advanced patterns:**
- Template literal types
- Recursive conditional types
- Distributive conditional types
- Key remapping in mapped types

---

## 9. What are the internals of `async/await` (microtasks, macrotasks, event loop)?

**How async/await works:**
- `async` function returns Promise
- `await` pauses execution, yields to event loop
- Resumes when Promise settles
- Uses microtask queue

**Execution order:**
1. Synchronous code (call stack)
2. Microtasks (Promises, await continuations)
3. Macrotasks (setTimeout, setInterval)
4. Repeat

**Key insight:** `await` creates implicit `.then()` which schedules microtask.

**Implication:** Awaited code runs before setTimeout callbacks even with 0 delay.

---

## 10. How would you implement a cancellable Promise pattern?

**Challenges:**
- Promises are not cancellable by design
- Need external cancellation mechanism

**Approaches:**

**1. AbortController + signal:**
- Web standard API
- Pass signal to async operations
- Check signal.aborted periodically

**2. Cancellation token:**
- Custom token object
- Pass to Promise chain
- Check token before each step

**3. Race with reject:**
- Race Promise with cancellation Promise
- Resolve cancellation to reject main Promise

**Considerations:**
- Clean up resources
- Handle partial completion
- Clear timers/intervals
- Notify about cancellation

---

## 11. Explain the concept of structural typing in TypeScript vs nominal typing

**Structural typing (TypeScript):**
- Type compatibility based on structure/shape
- If it walks like a duck, it's a duck
- More flexible

**Nominal typing (Java, C#):**
- Type compatibility based on name/declaration
- Explicit relationships required
- More strict

**TypeScript implications:**
- Can use any object with matching shape
- No need to explicitly implement interfaces
- Can accidentally satisfy interface
- Harder to prevent misuse

**Workaround for nominal:** Use branded types with unique symbols.

---

## 12. What are TypeScript decorators and how would you implement a custom one?

**Decorators:** Functions that modify classes, methods, properties, or parameters.

**Types:**
- Class decorators
- Method decorators
- Property decorators
- Parameter decorators

**Uses:**
- Logging/tracing
- Validation
- Dependency injection
- Memoization
- Access control

**Note:** Stage 2 proposal, experimental feature, syntax may change.

**Enable:** `"experimentalDecorators": true`

---

## 13. How would you optimize JavaScript code for JIT compilation?

**V8 optimization tips:**

**1. Monomorphic code:** Same function receives same types
**2. Avoid polymorphism:** Don't pass different types to same function
**3. Hidden classes:** Initialize object properties in same order
**4. Avoid holes in arrays:** Don't skip indices
**5. Use typed arrays:** For numeric data
**6. Small functions:** More likely to be inlined
**7. Avoid `arguments`:** Use rest parameters
**8. Avoid `eval` and `with`**
**9. Predictable control flow**

**Deoptimization triggers:** `delete`, changing object shape, holes, mixed types.

---

## 14. Explain the concept of Tail Call Optimization and its support in JavaScript

**Tail call:** Function call that's the last operation before returning.

**Tail call optimization (TCO):**
- Reuse current stack frame
- Prevent stack overflow
- Enable infinite recursion for certain patterns

**JavaScript support:**
- ES6 spec includes TCO
- Only in strict mode
- **Almost no browser support** (Safari only)
- V8/Node.js doesn't implement it

**Alternative:** Use loops or trampolining instead of relying on TCO.

---

## 15. How would you implement a reactive programming system from scratch?

**Core concepts:**

**Observables:** Data sources that emit values over time

**Observers:** Subscribe to observables, react to emissions

**Operators:** Transform, filter, combine observables

**Subscriptions:** Manage lifecycle, enable unsubscribe

**Key components to implement:**
1. Observable class with subscribe method
2. Subject (both observable and observer)
3. Operators (map, filter, merge, etc.)
4. Scheduler for async operations
5. Subscription disposal

**Challenges:**
- Memory management (unsubscribe)
- Error handling
- Completion semantics
- Backpressure
- Hot vs cold observables

**Real libraries:** RxJS, most.js

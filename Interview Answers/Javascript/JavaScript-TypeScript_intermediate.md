# JavaScript/TypeScript - Intermediate Level Answers

---

## 1. Explain prototypal inheritance in JavaScript and how it differs from classical inheritance

**Prototypal inheritance:** Objects inherit directly from other objects through the prototype chain.

**How it works:**
- Every object has an internal `[[Prototype]]` link
- Properties/methods lookup walks up the chain
- `Object.create()` creates object with specified prototype
- Constructor functions use `prototype` property

**vs Classical inheritance:**
- **Prototypal:** Objects inherit from objects, more flexible, dynamic
- **Classical:** Classes inherit from classes (blueprints), static structure

**ES6 classes:** Syntactic sugar over prototypal inheritance, not true classical inheritance.

---

## 2. What are JavaScript Symbols and when would you use them?

**Symbol:** Unique, immutable primitive value used as object property key.

**Characteristics:**
- Always unique (even with same description)
- Not enumerable in `for...in` or `Object.keys()`
- Accessed with `Object.getOwnPropertySymbols()`

**Use cases:**
1. **Unique property keys:** Avoid naming collisions
2. **Well-known Symbols:** `Symbol.iterator`, `Symbol.toStringTag`
3. **Private-like properties:** Not truly private but hidden from enumeration
4. **Protocol/interface definition:** Custom behaviors

---

## 3. Explain the Module pattern and its benefits

**Module pattern:** Uses closures to create private and public members, encapsulating functionality.

**Implementation:**
- IIFE (Immediately Invoked Function Expression) creates scope
- Returns object with public API
- Private variables/functions in closure

**Benefits:**
- Encapsulation (private state)
- Namespace pollution prevention
- Organized code structure
- Dependency management

**Modern alternative:** ES6 modules with `import`/`export`.

---

## 4. What is memoization and how would you implement it?

**Memoization:** Optimization technique caching function results based on input arguments.

**How it works:**
- Store results in cache (usually object or Map)
- Check cache before computing
- Return cached result if exists
- Compute and cache if not

**Best for:**
- Pure functions (same input = same output)
- Expensive computations
- Recursive functions (fibonacci, factorial)

**Trade-off:** Memory usage vs computation time.

---

## 5. Explain the difference between debouncing and throttling with use cases

**Debouncing:** Delays function execution until after a wait period since last call.
- **Use case:** Search input, window resize, form validation
- **Effect:** Function runs once after user stops action

**Throttling:** Ensures function executes at most once per time interval.
- **Use case:** Scroll events, mouse movement, API rate limiting
- **Effect:** Function runs at regular intervals during continuous action

**Key difference:** Debouncing waits for pause, throttling limits frequency.

---

## 6. What are JavaScript Proxies and Reflect API?

**Proxy:** Object that wraps another object and intercepts operations (get, set, delete, etc.).

**Trap methods:**
- `get`, `set`: Property access
- `has`: `in` operator
- `deleteProperty`: `delete` operator
- `apply`: Function calls
- And more...

**Reflect API:** Companion to Proxy providing default operation implementations.

**Use cases:**
- Validation
- Property access logging
- Data binding
- Computed properties
- Negative array indices
- API wrappers

---

## 7. How does garbage collection work in JavaScript?

**Garbage collection:** Automatic memory management that reclaims memory from unreachable objects.

**Algorithm: Mark-and-sweep**
1. Mark all reachable objects from roots (global, stack)
2. Sweep unmarked objects (unreachable = garbage)
3. Compact memory

**Roots:** Global variables, currently executing functions, closures.

**Common memory leaks:**
- Global variables
- Forgotten timers/intervals
- Closures holding large objects
- Detached DOM nodes
- Event listeners not removed

**Prevention:** Clear timers, remove listeners, nullify large objects, use WeakMap/WeakSet.

---

## 8. Explain the concept of currying and provide an example

**Currying:** Transforming a function with multiple arguments into a sequence of functions each taking a single argument.

**Purpose:**
- Partial application
- Function composition
- Configuration/specialization
- Point-free programming

**Pattern:** `f(a, b, c)` becomes `f(a)(b)(c)`

**Benefits:**
- Reusable partially applied functions
- Better function composition
- Flexible configuration

---

## 9. What are Generators and Iterators in JavaScript?

**Iterator:** Object with `next()` method returning `{value, done}`.

**Generator:** Function that can pause and resume, returning an iterator.
- Declared with `function*`
- Uses `yield` keyword to pause
- Can `yield*` to delegate

**Use cases:**
- Lazy evaluation
- Infinite sequences
- Async data fetching
- Custom iterables
- Pausable operations

---

## 10. How would you implement deep cloning of an object?

**Approaches:**

**1. JSON method (limitations):**
- Loses functions, undefined, Symbols, Dates
- Breaks on circular references

**2. Recursive deep clone:**
- Handle all types
- Use WeakMap for circular references
- Clone Date, RegExp, Arrays separately

**3. `structuredClone()` (modern):**
- Native browser/Node 17+ API
- Handles most types and circular refs
- Cannot clone functions

**Consider:** Use library (lodash `_.cloneDeep`) for production.

---

## 11. Explain the differences between TypeScript interfaces and types

**Similarities:** Both define object shapes and contracts.

**Key differences:**

**Interfaces:**
- Can be extended/implemented
- Support declaration merging
- Better for OOP patterns
- Clearer error messages

**Types:**
- Support unions, intersections, primitives
- Can use mapped/conditional types
- More flexible and powerful
- No declaration merging

**Best practice:** Use interfaces for objects/classes, types for unions/complex types.

---

## 12. What are TypeScript generics and why are they useful?

**Generics:** Parameterized types allowing reusable components that work with multiple types while maintaining type safety.

**Syntax:** `<T>` type parameter

**Benefits:**
- Type safety without losing flexibility
- Code reuse
- Better IDE support
- Compile-time error catching
- No runtime cost

**Features:**
- Generic constraints (`extends`)
- Multiple type parameters
- Default types
- Generic classes/interfaces/functions

---

## 13. Explain covariance and contravariance in TypeScript

**Variance:** How subtyping relationships work with generic types.

**Covariance (preserves direction):**
- If `Cat extends Animal`, then `Array<Cat> extends Array<Animal>`
- Common in return types, readonly properties
- Allows substituting more specific type

**Contravariance (reverses direction):**
- If `Cat extends Animal`, then `Handler<Animal> extends Handler<Cat>`
- Common in function parameters
- Allows substituting more general type

**TypeScript:** Covariant in return types, contravariant in parameters (with `strictFunctionTypes`).

---

## 14. What is the purpose of `unknown` vs `any` in TypeScript?

**`any`:**
- Disables type checking completely
- Can do anything without errors
- Unsafe, should be avoided

**`unknown`:**
- Type-safe alternative to `any`
- Must narrow type before use
- Forces type checking

**Use `unknown` when:**
- Type is truly unknown (API responses)
- Need type safety
- Validating external data

**Use `any` when:**
- Gradual migration from JS
- Third-party libs without types
- Truly dynamic behavior (avoid if possible)

---

## 15. How would you handle error boundaries in asynchronous code?

**Strategies:**

**1. Try-catch with async/await:**
- Clearest syntax
- Handle errors where they occur
- Can rethrow or return default

**2. Promise .catch():**
- Chain error handlers
- Centralized error handling
- Can recover or propagate

**3. Custom error classes:**
- Distinguish error types
- Better error handling logic
- Include context (status codes, etc.)

**4. Result type pattern:**
- Functional approach
- Explicit success/failure
- No throwing errors

**5. Global handlers:**
- `process.on('unhandledRejection')`
- Last resort, log and exit
- Should not rely on these

**Best practice:** Handle errors close to source, use typed errors, log appropriately.

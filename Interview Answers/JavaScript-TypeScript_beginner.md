# JavaScript/TypeScript - Beginner Level Answers

---

## 1. What are the differences between `var`, `let`, and `const`?

**`var`:**
- Function-scoped
- Hoisted to top of function
- Can be redeclared and updated
- Creates global property when declared globally

**`let`:**
- Block-scoped
- Hoisted but not initialized (temporal dead zone)
- Can be updated but not redeclared in same scope

**`const`:**
- Block-scoped
- Hoisted but not initialized
- Cannot be updated or redeclared
- Immutable binding (not value - objects can still be modified)

**Best practice:** Use `const` by default, `let` when reassignment needed, avoid `var`.

---

## 2. Explain the concept of hoisting in JavaScript

**Hoisting** is JavaScript's behavior of moving declarations to the top of their scope during compilation, before code execution.

**What gets hoisted:**
- Variable declarations with `var` (initialized to `undefined`)
- Function declarations (fully hoisted with definition)
- `let` and `const` declarations (hoisted but in temporal dead zone)

**Result:**
- Can call function declarations before they appear in code
- `var` variables return `undefined` before initialization
- `let`/`const` throw ReferenceError before initialization

---

## 3. What is the difference between `==` and `===`?

**`==` (Loose equality):**
- Performs type coercion before comparison
- Converts operands to same type, then compares
- Can produce unexpected results

**`===` (Strict equality):**
- No type coercion
- Compares both type and value
- More predictable and safer

**Best practice:** Always use `===` unless you specifically need type coercion.

---

## 4. What are arrow functions and how do they differ from regular functions?

**Arrow functions** are a concise syntax for writing functions with different `this` binding behavior.

**Key differences:**
1. **Syntax:** Shorter, implicit return for single expressions
2. **`this` binding:** Lexical (inherits from parent scope), not dynamic
3. **No `arguments` object:** Use rest parameters instead
4. **Cannot be constructors:** No `new` keyword
5. **No `prototype` property**
6. **Cannot be generators**

**When to use:**
- Arrow: Callbacks, array methods, when you need lexical `this`
- Regular: Object methods, constructors, when you need `arguments`

---

## 5. Explain the concept of closures with a simple example

**Closure:** A function that retains access to variables from its outer (enclosing) scope, even after the outer function has finished executing.

**How it works:**
- Inner functions have access to outer function's variables
- Creates private variables/state
- Formed every time a function is created

**Use cases:**
- Data privacy (private variables)
- Factory functions
- Module pattern
- Event handlers with specific data
- Memoization

---

## 6. What are Promises and how do they help with asynchronous code?

**Promise:** An object representing the eventual completion or failure of an asynchronous operation.

**Three states:**
1. **Pending:** Initial state
2. **Fulfilled:** Operation successful
3. **Rejected:** Operation failed

**Benefits:**
- Avoid callback hell (cleaner than nested callbacks)
- Better error handling with `.catch()`
- Chainable with `.then()`
- Composable with `Promise.all()`, `Promise.race()`, etc.
- Foundation for `async/await` syntax

---

## 7. What is the event loop in JavaScript?

**Event loop:** Mechanism that handles asynchronous operations by managing execution of code, events, and callbacks.

**How it works:**
1. Execute synchronous code (call stack)
2. When call stack is empty, check microtask queue (process all)
3. Then check callback queue (process one macrotask)
4. Repeat

**Queues:**
- **Microtasks:** Promises, `queueMicrotask` (higher priority)
- **Macrotasks:** `setTimeout`, `setInterval`, I/O operations

**Result:** JavaScript is single-threaded but can handle async operations without blocking.

---

## 8. Explain the difference between `null` and `undefined`

**`undefined`:**
- Default value when variable declared but not assigned
- Function returns this when no return statement
- Accessing non-existent object property
- Automatically assigned by JavaScript

**`null`:**
- Intentional absence of value
- Must be explicitly assigned
- Represents "no object" or deliberate emptiness

**Type quirk:** `typeof null` returns `"object"` (historical bug), `typeof undefined` returns `"undefined"`.

**Best practice:** Use `null` for intentional "no value", let JavaScript use `undefined` for uninitialized.

---

## 9. What are template literals and when would you use them?

**Template literals:** String literals using backticks (`` ` ``) that support interpolation and multi-line strings.

**Features:**
- **String interpolation:** Embed expressions with `${}`
- **Multi-line strings:** Preserve line breaks and formatting
- **Tagged templates:** Custom processing with functions

**Use cases:**
- Dynamic strings with variables
- HTML/SQL generation
- Multi-line text
- Localization
- Styled components (React)

**Note:** Always escape user input to prevent XSS attacks.

---

## 10. What is destructuring in JavaScript?

**Destructuring:** Syntax for unpacking values from arrays or properties from objects into distinct variables.

**Types:**
- **Array destructuring:** Extract values by position
- **Object destructuring:** Extract values by property name

**Features:**
- Default values
- Renaming variables (objects)
- Rest operator (`...`)
- Nested destructuring
- Function parameter destructuring

**Benefits:**
- More readable code
- Less repetitive code
- Easy variable swapping
- Clean function signatures

---

## 11. Explain the spread operator and rest parameters

**Spread operator (`...`):**
- Expands iterables into individual elements
- Used in array/object literals or function calls
- Creates shallow copies

**Rest parameters (`...`):**
- Collects multiple elements into an array
- Used in function parameters or destructuring
- Must be last parameter

**Both use same syntax but opposite purposes:** Spread expands, rest collects.

---

## 12. What are the primitive data types in JavaScript?

**Seven primitive types:**
1. **String:** Text data
2. **Number:** Integers and floats (64-bit floating point)
3. **BigInt:** Arbitrarily large integers
4. **Boolean:** `true` or `false`
5. **Undefined:** Uninitialized variable
6. **Null:** Intentional absence of value
7. **Symbol:** Unique identifier

**Characteristics:**
- Immutable
- Passed by value
- Have wrapper objects (String, Number, Boolean)
- Everything else is an object (arrays, functions, dates)

---

## 13. What is the purpose of `async/await`?

**`async/await`:** Syntactic sugar over Promises for writing asynchronous code that looks synchronous.

**`async` keyword:**
- Makes function return a Promise
- Allows use of `await` inside

**`await` keyword:**
- Pauses execution until Promise resolves
- Can only be used in async functions
- Returns resolved value or throws on rejection

**Benefits:**
- More readable than Promise chains
- Easier error handling with try/catch
- Better debugging (clear stack traces)
- Feels like synchronous code

---

## 14. How does `this` keyword work in JavaScript?

**`this`:** References the object executing the current function. Value depends on how function is called.

**Four binding rules:**
1. **Method call:** `this` = object owning the method
2. **Function call:** `this` = global object (or `undefined` in strict mode)
3. **Constructor:** `this` = newly created instance
4. **Explicit binding:** Set with `call()`, `apply()`, `bind()`

**Arrow functions:** Inherit `this` from parent scope (lexical binding).

**Common pitfall:** Losing `this` context when passing methods as callbacks.

---

## 15. What is the difference between `.call()`, `.apply()`, and `.bind()`?

All three methods set the `this` value for a function.

**`.call(thisArg, arg1, arg2, ...)`:**
- Invokes function immediately
- Arguments passed individually

**`.apply(thisArg, [args])`:**
- Invokes function immediately
- Arguments passed as array

**`.bind(thisArg, arg1, arg2, ...)`:**
- Returns new function with bound `this`
- Does NOT invoke immediately
- Can pre-set arguments (partial application)

**Use cases:**
- `call`/`apply`: Borrow methods, invoke with specific context
- `bind`: Event handlers, React class methods, callbacks

# JavaScript Comprehensive Interview Guide

## Table of Contents

1. [Variables and Data Types](#variables-and-data-types)
2. [Functions](#functions)
3. [Scope and Closures](#scope-and-closures)
4. [this Keyword](#this-keyword)
5. [Prototypes and Inheritance](#prototypes-and-inheritance)
6. [Asynchronous JavaScript](#asynchronous-javascript)
7. [Event Loop](#event-loop)
8. [ES6+ Features](#es6-features)
9. [Array Methods](#array-methods)
10. [Object Manipulation](#object-manipulation)
11. [Error Handling](#error-handling)
12. [Modules](#modules)
13. [Design Patterns](#design-patterns)
14. [Performance Optimization](#performance-optimization)
15. [Interview Questions](#interview-questions)

---

## Variables and Data Types

### Simple Explanation

Think of variables as labeled boxes where you store things. JavaScript has three ways to create these boxes: `var` (the old way), `let` (the modern way), and `const` (for things that shouldn't change).

Imagine you're organizing your closet:

- **var** is like putting clothes anywhere in your house - you might forget where they are
- **let** is like keeping clothes in specific rooms - you always know where to find them
- **const** is like a picture frame on the wall - it stays in one place

### Technical Explanation

JavaScript has **primitive types** (immutable values) and **reference types** (mutable objects).

**Primitive Types:**

- `string`: text data
- `number`: numeric values (includes integers and floats)
- `boolean`: true/false
- `undefined`: declared but not assigned
- `null`: intentional absence of value
- `symbol`: unique identifiers
- `bigint`: large integers

**Reference Types:**

- `object`: collections of key-value pairs
- `array`: ordered lists
- `function`: executable code blocks

```typescript
// Primitive types
let name: string = 'Ale';
let age: number = 30;
let isDeveloper: boolean = true;
let notAssigned: undefined;
let empty: null = null;
let uniqueId: symbol = Symbol('id');
let bigNumber: bigint = 9007199254740991n;

// Reference types
let person: object = { name: 'Ale', age: 30 };
let skills: string[] = ['JavaScript', 'TypeScript', 'React'];
let greet: Function = function () {
  console.log('Hello');
};
```

### Merged Explanation

Variables are containers for storing data. JavaScript offers three declaration keywords with different behaviors:

**var (function-scoped, hoisted, can be redeclared)**

```typescript
// var is hoisted and function-scoped
function varExample() {
  console.log(x); // undefined (hoisted but not initialized)
  var x = 5;

  if (true) {
    var x = 10; // Same variable!
    console.log(x); // 10
  }
  console.log(x); // 10 - var ignores block scope
}
```

**let (block-scoped, not hoisted in TDZ, cannot be redeclared)**

```typescript
// let is block-scoped
function letExample() {
  // console.log(y); // ReferenceError: Cannot access before initialization
  let y = 5;

  if (true) {
    let y = 10; // Different variable!
    console.log(y); // 10
  }
  console.log(y); // 5
}
```

**const (block-scoped, must be initialized, cannot be reassigned)**

```typescript
// const prevents reassignment, but not mutation
const PI = 3.14159;
// PI = 3.14; // Error: Assignment to constant variable

const user = { name: 'Ale' };
user.name = 'Alexander'; // OK - mutating object properties
// user = {}; // Error: Assignment to constant variable

// Use Object.freeze() for true immutability
const frozenUser = Object.freeze({ name: 'Ale' });
// frozenUser.name = "Alexander"; // Fails silently in normal mode, throws in strict mode
```

### Common Pitfalls

1. **Type coercion surprises**

```typescript
console.log('5' + 3); // "53" (string concatenation)
console.log('5' - 3); // 2 (numeric subtraction)
console.log([] + []); // "" (empty string)
console.log([] + {}); // "[object Object]"
console.log({} + []); // 0 (in some contexts)
```

2. **Reference vs Value**

```typescript
// Primitives are copied by value
let a = 5;
let b = a;
b = 10;
console.log(a); // 5 (unchanged)

// Objects are copied by reference
let obj1 = { value: 5 };
let obj2 = obj1;
obj2.value = 10;
console.log(obj1.value); // 10 (changed!)

// Deep clone to avoid this
let obj3 = structuredClone(obj1); // or JSON.parse(JSON.stringify(obj1))
```

### Best Practices

1. **Use `const` by default**, `let` when reassignment is needed, avoid `var`
2. **Be explicit with types** (use TypeScript)
3. **Avoid global variables**
4. **Use meaningful variable names**

```typescript
// Bad
let d = new Date();
let x = u.filter((i) => i.a);

// Good
const currentDate = new Date();
const activeUsers = users.filter((user) => user.isActive);
```

---

## Functions

### Simple Explanation

Functions are like recipes. You give them ingredients (parameters), they follow steps, and give you a dish (return value). You can write the recipe once and use it many times.

Think of a coffee machine:

- Input: coffee beans, water (parameters)
- Process: grinding, brewing (function body)
- Output: coffee (return value)

### Technical Explanation

JavaScript functions are first-class objects, meaning they can be:

- Assigned to variables
- Passed as arguments
- Returned from other functions
- Stored in data structures

```typescript
// Function Declaration (hoisted)
function add(a: number, b: number): number {
  return a + b;
}

// Function Expression (not hoisted)
const subtract = function (a: number, b: number): number {
  return a - b;
};

// Arrow Function (lexical this)
const multiply = (a: number, b: number): number => a * b;

// Arrow function with body
const divide = (a: number, b: number): number => {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
};
```

### Merged Explanation

Functions encapsulate reusable logic. JavaScript provides multiple syntaxes, each with specific characteristics:

**Function Declarations vs Expressions**

```typescript
// Declaration - hoisted, can be called before definition
console.log(greet('Ale')); // Works!

function greet(name: string): string {
  return `Hello, ${name}!`;
}

// Expression - not hoisted
// console.log(farewell("Ale")); // Error!

const farewell = function (name: string): string {
  return `Goodbye, ${name}!`;
};
```

**Arrow Functions and `this` Binding**

```typescript
// Traditional function - dynamic 'this'
const person = {
  name: 'Ale',
  traditional: function () {
    console.log(this.name); // "Ale"

    setTimeout(function () {
      console.log(this.name); // undefined (this refers to global/undefined)
    }, 1000);
  },

  // Arrow function - lexical 'this'
  arrow: function () {
    console.log(this.name); // "Ale"

    setTimeout(() => {
      console.log(this.name); // "Ale" (inherits this from parent)
    }, 1000);
  },
};
```

**Parameters and Arguments**

```typescript
// Default parameters
function createUser(name: string, role: string = 'user'): object {
  return { name, role };
}

console.log(createUser('Ale')); // { name: "Ale", role: "user" }
console.log(createUser('Ale', 'admin')); // { name: "Ale", role: "admin" }

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3, 4)); // 10

// Destructuring parameters
interface User {
  name: string;
  age: number;
  email?: string;
}

function displayUser({ name, age, email = 'N/A' }: User): void {
  console.log(`${name}, ${age}, ${email}`);
}

displayUser({ name: 'Ale', age: 30 }); // "Ale, 30, N/A"
```

**Higher-Order Functions**

```typescript
// Function that returns a function
function createMultiplier(multiplier: number) {
  return function (value: number): number {
    return value * multiplier;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15

// Function that takes a function as argument
function processArray(arr: number[], fn: (n: number) => number): number[] {
  return arr.map(fn);
}

const numbers = [1, 2, 3, 4];
console.log(processArray(numbers, (x) => x * 2)); // [2, 4, 6, 8]
```

**Immediately Invoked Function Expressions (IIFE)**

```typescript
// Used for creating isolated scope
(function () {
  const secret = 'This is private';
  console.log(secret);
})();

// console.log(secret); // Error: secret is not defined

// Modern alternative: block scope
{
  const secret = 'This is also private';
  console.log(secret);
}
```

### Common Pitfalls

1. **Forgetting return statement**

```typescript
// Implicit return in arrow functions
const square = (n: number) => n * n; // OK

// But curly braces require explicit return
const squareWithLog = (n: number) => {
  console.log(`Squaring ${n}`);
  n * n; // Oops! Returns undefined
};

const squareWithLogFixed = (n: number) => {
  console.log(`Squaring ${n}`);
  return n * n; // Fixed!
};
```

2. **Arrow functions as methods**

```typescript
const counter = {
  count: 0,
  // Don't use arrow functions as object methods
  incrementWrong: () => {
    this.count++; // 'this' doesn't refer to counter object
  },
  // Use regular functions or method shorthand
  increment() {
    this.count++;
  },
};
```

### Best Practices

1. **Keep functions small and focused** (single responsibility)
2. **Use descriptive names** (verbs for actions)
3. **Limit parameters** (max 3-4, use object for more)
4. **Avoid side effects** when possible (pure functions)

```typescript
// Bad - function does too much
function processUserData(user: any) {
  validateUser(user);
  saveToDatabase(user);
  sendEmail(user);
  logActivity(user);
  updateCache(user);
}

// Good - separate concerns
function validateUser(user: User): boolean {
  /* ... */
}
function saveUser(user: User): Promise<User> {
  /* ... */
}
function notifyUser(user: User): Promise<void> {
  /* ... */
}

// Use in sequence
async function processUser(user: User) {
  if (validateUser(user)) {
    const savedUser = await saveUser(user);
    await notifyUser(savedUser);
  }
}

// Pure function example
function calculateTotal(items: number[]): number {
  return items.reduce((sum, item) => sum + item, 0);
}
```

---

## Scope and Closures

### Simple Explanation

**Scope** is like the visibility range of a person. If you're in a room (function), you can see things in that room (local variables) and things in the house (outer scopes), but people outside the house can't see what's in your room.

**Closures** are like a backpack. When a function is created, it packs up all the variables it needs from its surrounding environment into a backpack. Even when the function travels far away (is called elsewhere), it still has access to those packed variables.

Imagine a lunchbox your parent prepared:

- Even though you're at school (different scope)
- You still have access to the lunch (variables) from home (parent scope)
- This is a closure!

### Technical Explanation

**Scope** determines variable accessibility and lifetime. JavaScript has:

- Global scope
- Function scope (created by functions)
- Block scope (created by {}, introduced with let/const)
- Module scope (in ES6 modules)

**Closures** occur when an inner function has access to variables in its outer function's scope, even after the outer function has returned.

```typescript
// Lexical scope example
const globalVar = "I'm global";

function outerFunction() {
  const outerVar = "I'm from outer";

  function innerFunction() {
    const innerVar = "I'm from inner";
    console.log(globalVar); // Accessible
    console.log(outerVar); // Accessible
    console.log(innerVar); // Accessible
  }

  innerFunction();
  // console.log(innerVar); // Error: not accessible here
}

outerFunction();
```

### Merged Explanation

**Understanding Scope Chain**

```typescript
let globalCount = 0;

function outer() {
  let outerCount = 0;

  function middle() {
    let middleCount = 0;

    function inner() {
      let innerCount = 0;

      // Scope chain: inner -> middle -> outer -> global
      console.log(innerCount); // 0 (own scope)
      console.log(middleCount); // 0 (parent scope)
      console.log(outerCount); // 0 (grandparent scope)
      console.log(globalCount); // 0 (global scope)
    }

    inner();
    // console.log(innerCount); // Error: not in scope
  }

  middle();
}

outer();
```

**Block Scope vs Function Scope**

```typescript
// Function scope with var
function functionScopeExample() {
  if (true) {
    var functionScoped = "I'm function scoped";
  }
  console.log(functionScoped); // Accessible - var ignores block scope
}

// Block scope with let/const
function blockScopeExample() {
  if (true) {
    let blockScoped = "I'm block scoped";
    const alsoBlockScoped = 'Me too';
    console.log(blockScoped); // Accessible here
  }
  // console.log(blockScoped); // Error: not accessible outside block
}

// Practical example: loop with var vs let
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log('var:', i), 100); // Prints "var: 3" three times
}

for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log('let:', j), 100); // Prints "let: 0", "let: 1", "let: 2"
}
```

**Closures in Action**

```typescript
// Basic closure
function createCounter() {
  let count = 0; // Private variable

  return {
    increment() {
      count++;
      return count;
    },
    decrement() {
      count--;
      return count;
    },
    getCount() {
      return count;
    },
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.getCount()); // 2
// console.log(counter.count);    // undefined - truly private!

// Closure in callbacks
function setupEventHandler(buttonId: string) {
  const clickCount = 0; // This would normally be lost after function returns

  document.getElementById(buttonId)?.addEventListener('click', function () {
    // But the callback "closes over" clickCount
    console.log(`Button clicked ${clickCount} times`);
  });
}
```

**Practical Closure Examples**

```typescript
// 1. Data Privacy
function createBankAccount(initialBalance: number) {
  let balance = initialBalance; // Private

  return {
    deposit(amount: number) {
      if (amount > 0) {
        balance += amount;
        return balance;
      }
      throw new Error('Invalid deposit amount');
    },
    withdraw(amount: number) {
      if (amount > 0 && amount <= balance) {
        balance -= amount;
        return balance;
      }
      throw new Error('Invalid withdrawal amount');
    },
    getBalance() {
      return balance;
    },
  };
}

const account = createBankAccount(1000);
account.deposit(500); // 1500
account.withdraw(200); // 1300
console.log(account.getBalance()); // 1300
// account.balance = 10000; // Impossible - balance is truly private

// 2. Function Factories
function createGreeter(greeting: string) {
  return function (name: string) {
    return `${greeting}, ${name}!`;
  };
}

const sayHello = createGreeter('Hello');
const sayHola = createGreeter('Hola');
const sayBonjour = createGreeter('Bonjour');

console.log(sayHello('Ale')); // "Hello, Ale!"
console.log(sayHola('Ale')); // "Hola, Ale!"
console.log(sayBonjour('Ale')); // "Bonjour, Ale!"

// 3. Memoization with Closure
function memoize<T extends (...args: any[]) => any>(fn: T): T {
  const cache = new Map();

  return ((...args: Parameters<T>): ReturnType<T> => {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      console.log('From cache');
      return cache.get(key);
    }

    console.log('Computing');
    const result = fn(...args);
    cache.set(key, result);
    return result;
  }) as T;
}

const expensiveCalculation = memoize((n: number): number => {
  let sum = 0;
  for (let i = 0; i < n; i++) {
    sum += i;
  }
  return sum;
});

console.log(expensiveCalculation(1000000)); // Computing
console.log(expensiveCalculation(1000000)); // From cache

// 4. Partial Application
function multiply(a: number, b: number, c: number): number {
  return a * b * c;
}

function partial(fn: Function, ...fixedArgs: any[]) {
  return function (...remainingArgs: any[]) {
    return fn(...fixedArgs, ...remainingArgs);
  };
}

const multiplyBy2 = partial(multiply, 2);
console.log(multiplyBy2(3, 4)); // 2 * 3 * 4 = 24

const multiplyBy2And3 = partial(multiply, 2, 3);
console.log(multiplyBy2And3(4)); // 2 * 3 * 4 = 24
```

**Module Pattern with Closures**

```typescript
const UserModule = (function () {
  // Private variables and functions
  let users: string[] = [];

  function validateUser(user: string): boolean {
    return user.length >= 3;
  }

  // Public API
  return {
    addUser(user: string): boolean {
      if (validateUser(user)) {
        users.push(user);
        return true;
      }
      return false;
    },

    getUsers(): string[] {
      return [...users]; // Return copy, not reference
    },

    getUserCount(): number {
      return users.length;
    },
  };
})();

UserModule.addUser('Ale');
UserModule.addUser('Jo');
console.log(UserModule.getUserCount()); // 2
// UserModule.users; // undefined - private!
// UserModule.validateUser("test"); // undefined - private!
```

### Common Pitfalls

1. **Memory leaks with closures**

```typescript
// Bad - creates memory leak
function addButton() {
  const button = document.getElementById('myButton');
  const largeData = new Array(1000000);

  button?.addEventListener('click', function () {
    console.log('Clicked');
    // This closure captures largeData unnecessarily
  });
}

// Good - don't capture unnecessary data
function addButtonFixed() {
  const button = document.getElementById('myButton');

  button?.addEventListener('click', function () {
    console.log('Clicked');
  });
  // largeData can be garbage collected
}
```

2. **Closure in loops (classic var problem)**

```typescript
// Bad - all callbacks reference same variable
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // Prints 3, 3, 3
}

// Good - use let (creates new binding per iteration)
for (let j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 100); // Prints 0, 1, 2
}

// Alternative - use IIFE to create new scope
for (var k = 0; k < 3; k++) {
  (function (num) {
    setTimeout(() => console.log(num), 100); // Prints 0, 1, 2
  })(k);
}
```

### Best Practices

1. **Use closures for data privacy** (private variables/methods)
2. **Be mindful of memory** (don't capture large objects unnecessarily)
3. **Prefer `let`/`const` over `var`** (proper block scoping)
4. **Use closures for configuration** (function factories)

```typescript
// Good: Closure for configuration
function createApiClient(baseUrl: string, apiKey: string) {
  // Configuration is captured in closure
  return {
    async get(endpoint: string) {
      const response = await fetch(`${baseUrl}${endpoint}`, {
        headers: { Authorization: `Bearer ${apiKey}` },
      });
      return response.json();
    },
    async post(endpoint: string, data: any) {
      const response = await fetch(`${baseUrl}${endpoint}`, {
        method: 'POST',
        headers: {
          Authorization: `Bearer ${apiKey}`,
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(data),
      });
      return response.json();
    },
  };
}

const api = createApiClient('https://api.example.com', 'secret-key');
api.get('/users');
```

---

## this Keyword

### Simple Explanation

`this` is like a pronoun in a sentence. Its meaning depends on context:

- "I love my car" - "my" refers to the speaker
- In JavaScript, `this` refers to different objects depending on how a function is called

Think of `this` as a spotlight in a theater:

- Regular function call: spotlight on the stage (global/window)
- Method call: spotlight on the actor (object)
- Arrow function: spotlight stays where it was defined (lexical)

### Technical Explanation

`this` is a special keyword that refers to the context in which a function is executed. Its value is determined by how the function is called (runtime binding), not where it's defined.

```typescript
// Four ways to bind 'this':
// 1. Default binding (global object or undefined in strict mode)
// 2. Implicit binding (method call)
// 3. Explicit binding (call, apply, bind)
// 4. new binding (constructor)
```

### Merged Explanation

**1. Default Binding (Function Invocation)**

```typescript
function showThis() {
  console.log(this);
}

showThis(); // Window (browser) or global (Node.js) in non-strict mode
// undefined in strict mode

('use strict');
function showThisStrict() {
  console.log(this);
}

showThisStrict(); // undefined
```

**2. Implicit Binding (Method Invocation)**

```typescript
const person = {
  name: 'Ale',
  greet: function () {
    console.log(`Hello, I'm ${this.name}`);
  },
};

person.greet(); // "Hello, I'm Ale" - this = person

// But 'this' can be lost!
const greetFunc = person.greet;
greetFunc(); // "Hello, I'm undefined" - this = global/undefined

// Also lost in callbacks
setTimeout(person.greet, 1000); // "Hello, I'm undefined"
```

**3. Explicit Binding (call, apply, bind)**

```typescript
function introduce(greeting: string, punctuation: string) {
  console.log(`${greeting}, I'm ${this.name}${punctuation}`);
}

const user = { name: 'Ale' };

// call - arguments listed individually
introduce.call(user, 'Hi', '!'); // "Hi, I'm Ale!"

// apply - arguments as array
introduce.apply(user, ['Hello', '.']); // "Hello, I'm Ale."

// bind - returns new function with bound this
const boundIntroduce = introduce.bind(user, 'Hey');
boundIntroduce('!!!'); // "Hey, I'm Ale!!!"

// Fixing the setTimeout problem
setTimeout(person.greet.bind(person), 1000); // Now works correctly
```

**4. new Binding (Constructor)**

```typescript
function User(name: string, email: string) {
  // When called with 'new', 'this' refers to the new object being created
  this.name = name;
  this.email = email;
}

const user1 = new User('Ale', 'ale@example.com');
console.log(user1.name); // "Ale"

// Without 'new', 'this' might be global object
// const user2 = User("Bob", "bob@example.com"); // Bad! Pollutes global scope
```

**5. Arrow Functions (Lexical this)**

```typescript
const person = {
  name: 'Ale',
  hobbies: ['coding', 'music'],

  // Traditional function
  showHobbiesTraditional: function () {
    this.hobbies.forEach(function (hobby) {
      // 'this' is undefined or global here!
      console.log(`${this.name} likes ${hobby}`); // undefined likes coding
    });
  },

  // Arrow function solution
  showHobbiesArrow: function () {
    this.hobbies.forEach((hobby) => {
      // 'this' is inherited from parent scope
      console.log(`${this.name} likes ${hobby}`); // Ale likes coding
    });
  },

  // Or use the 'thisArg' parameter
  showHobbiesBound: function () {
    this.hobbies.forEach(function (hobby) {
      console.log(`${this.name} likes ${hobby}`);
    }, this); // Pass 'this' as second argument
  },
};

person.showHobbiesArrow();
```

**Binding Precedence**

```typescript
// Priority (highest to lowest):
// 1. new binding
// 2. Explicit binding (call, apply, bind)
// 3. Implicit binding (method call)
// 4. Default binding

function Person(name: string) {
  this.name = name;
}

const obj = {};

// Explicit binding
const BoundPerson = Person.bind(obj);
BoundPerson('Ale');
console.log(obj.name); // undefined? No, "Ale" because bind was called

// But 'new' overrides bind!
const newPerson = new BoundPerson('Bob');
console.log(newPerson.name); // "Bob" - new wins!
console.log(obj.name); // "Ale" - unchanged
```

**Class Methods and this**

```typescript
class Counter {
  count: number = 0;

  // Method - 'this' can be lost
  increment() {
    this.count++;
  }

  // Arrow function property - 'this' is always bound
  incrementArrow = () => {
    this.count++;
  };

  // Bound in constructor
  constructor() {
    this.increment = this.increment.bind(this);
  }
}

const counter = new Counter();

// Loses 'this' with regular method
const incrementFn = counter.increment;
// incrementFn(); // Error: Cannot read property 'count' of undefined

// But arrow function property keeps 'this'
const incrementArrowFn = counter.incrementArrow;
incrementArrowFn(); // Works!
console.log(counter.count); // 1
```

**Practical Examples**

```typescript
// React component (common use case)
class Button extends React.Component {
  state = { clicks: 0 };

  // Problem: 'this' is undefined in event handler
  handleClickBad() {
    this.setState({ clicks: this.state.clicks + 1 }); // Error!
  }

  // Solution 1: Arrow function property
  handleClickArrow = () => {
    this.setState({ clicks: this.state.clicks + 1 }); // Works!
  };

  // Solution 2: Bind in constructor
  constructor(props) {
    super(props);
    this.handleClickBound = this.handleClickBound.bind(this);
  }

  handleClickBound() {
    this.setState({ clicks: this.state.clicks + 1 }); // Works!
  }

  render() {
    return (
      <div>
        {/* Solution 3: Arrow function in JSX (creates new function each render) */}
        <button onClick={() => this.handleClickArrow()}>Click me</button>
        <button onClick={this.handleClickBound}>Or me</button>
      </div>
    );
  }
}

// API client example
class ApiClient {
  baseUrl: string;
  token: string;

  constructor(baseUrl: string, token: string) {
    this.baseUrl = baseUrl;
    this.token = token;
  }

  // Must bind 'this' or use arrow function
  get = async (endpoint: string) => {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      headers: { Authorization: `Bearer ${this.token}` },
    });
    return response.json();
  };
}

const api = new ApiClient('https://api.example.com', 'token');
const fetchUsers = api.get; // 'this' is preserved
fetchUsers('/users'); // Works because get is an arrow function
```

### Common Pitfalls

1. **Losing this in callbacks**

```typescript
const obj = {
  value: 42,
  getValue: function () {
    return this.value;
  },
};

// Direct call works
console.log(obj.getValue()); // 42

// But passing as callback loses 'this'
const getValueRef = obj.getValue;
console.log(getValueRef()); // undefined

// Solutions:
// 1. Arrow function wrapper
setTimeout(() => obj.getValue(), 1000);

// 2. Bind
setTimeout(obj.getValue.bind(obj), 1000);

// 3. Make getValue an arrow function
const obj2 = {
  value: 42,
  getValue: () => this.value, // But this won't work either in object literal!
};

// Correct: use arrow function in class or assign separately
class MyClass {
  value = 42;
  getValue = () => this.value; // This works!
}
```

2. **Arrow functions as object methods**

```typescript
// Bad - arrow function doesn't have its own 'this'
const calculator = {
  value: 0,
  // This won't work as expected!
  add: (n: number) => {
    this.value += n; // 'this' doesn't refer to calculator
    return this.value;
  },
};

// Good - use regular function or method shorthand
const calculatorFixed = {
  value: 0,
  add(n: number) {
    this.value += n;
    return this.value;
  },
};
```

### Best Practices

1. **Use arrow functions for callbacks** to preserve `this`
2. **Use regular functions for object methods**
3. **Bind in constructor** for class methods passed as callbacks
4. **Use arrow function properties in classes** for React event handlers
5. **Be explicit** - use `self` or `that` if needed (though modern alternatives are better)

```typescript
// Good patterns
class Component {
  // Arrow function for event handlers
  handleClick = () => {
    console.log(this); // Always the component instance
  };

  // Regular method for internal logic
  internalMethod() {
    console.log(this); // Component instance when called correctly
  }

  // Bind in constructor if passing method reference
  constructor() {
    this.internalMethod = this.internalMethod.bind(this);
  }
}

// Functional approach (no 'this' problems!)
function createCounter() {
  let count = 0;
  return {
    increment: () => ++count,
    decrement: () => --count,
    getCount: () => count,
  };
}
```

---

## Prototypes and Inheritance

### Simple Explanation

Imagine a family tree:

- You inherit traits from your parents (eye color, height)
- Your parents inherited from their parents
- JavaScript objects work the same way - they inherit properties from their "parents" (prototypes)

Think of prototypes like a shared family cookbook:

- Every family member (object) can access recipes (methods) from the family cookbook (prototype)
- If you don't have a recipe, you check the family cookbook
- If it's not there, you check grandma's cookbook (prototype chain)
- This saves memory - one cookbook instead of copies for everyone

### Technical Explanation

In JavaScript, nearly every object has an internal property `[[Prototype]]` (accessed via `__proto__` or `Object.getPrototypeOf()`) that references another object. When you access a property, JavaScript first looks at the object itself, then up the prototype chain until it finds the property or reaches `null`.

**Prototype Chain:**

```
instance → Constructor.prototype → Object.prototype → null
```

### Merged Explanation

**Understanding Prototypes**

```typescript
// Every function has a 'prototype' property
function Person(name: string) {
  this.name = name;
}

// Add methods to the prototype, not the instance
Person.prototype.greet = function () {
  return `Hello, I'm ${this.name}`;
};

// Create instances
const ale = new Person('Ale');
const bob = new Person('Bob');

// Both share the same greet method (memory efficient)
console.log(ale.greet === bob.greet); // true
console.log(ale.greet()); // "Hello, I'm Ale"

// Check prototype
console.log(Object.getPrototypeOf(ale) === Person.prototype); // true
console.log(ale.__proto__ === Person.prototype); // true (legacy, avoid)
```

**Prototype Chain**

```typescript
// Chain: ale → Person.prototype → Object.prototype → null

console.log(ale.name); // "Ale" - found on instance
console.log(ale.greet()); // "Hello, I'm Ale" - found on Person.prototype
console.log(ale.toString()); // "[object Object]" - found on Object.prototype
console.log(ale.nonExistent); // undefined - not found anywhere

// Checking the chain
console.log(ale.hasOwnProperty('name')); // true - own property
console.log(ale.hasOwnProperty('greet')); // false - inherited
console.log('greet' in ale); // true - in instance or prototype

// Walking up the chain
let current = ale;
while (current !== null) {
  console.log(Object.getPrototypeOf(current));
  current = Object.getPrototypeOf(current);
}
// Person.prototype
// Object.prototype
// null
```

**Constructor Functions vs Classes**

```typescript
// ES5 Constructor Function
function Animal(name: string) {
  this.name = name;
}

Animal.prototype.speak = function () {
  console.log(`${this.name} makes a sound`);
};

// ES6 Class (syntactic sugar over prototypes)
class AnimalClass {
  constructor(public name: string) {}

  speak() {
    console.log(`${this.name} makes a sound`);
  }
}

// Both create the same structure!
const dog1 = new Animal('Rex');
const dog2 = new AnimalClass('Max');

console.log(typeof Animal); // "function"
console.log(typeof AnimalClass); // "function"
console.log(dog1.speak === dog2.speak); // false (different prototypes)
```

**Inheritance with Prototypes**

```typescript
// ES5 Prototypal Inheritance
function Animal(name: string) {
  this.name = name;
}

Animal.prototype.eat = function () {
  console.log(`${this.name} is eating`);
};

function Dog(name: string, breed: string) {
  Animal.call(this, name); // Call parent constructor
  this.breed = breed;
}

// Set up prototype chain
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog; // Fix constructor reference

Dog.prototype.bark = function () {
  console.log(`${this.name} says woof!`);
};

const rex = new Dog('Rex', 'Labrador');
rex.eat(); // "Rex is eating" - inherited from Animal
rex.bark(); // "Rex says woof!" - defined on Dog

// Chain: rex → Dog.prototype → Animal.prototype → Object.prototype → null
```

**Modern Class Inheritance**

```typescript
// ES6 Class Inheritance
class Animal {
  constructor(public name: string) {}

  eat() {
    console.log(`${this.name} is eating`);
  }

  toString() {
    return `Animal: ${this.name}`;
  }
}

class Dog extends Animal {
  constructor(name: string, public breed: string) {
    super(name); // Must call parent constructor
  }

  bark() {
    console.log(`${this.name} says woof!`);
  }

  // Override parent method
  eat() {
    super.eat(); // Call parent implementation
    console.log('Dog specific eating behavior');
  }
}

class GuideDog extends Dog {
  constructor(name: string, breed: string, public owner: string) {
    super(name, breed);
  }

  guide() {
    console.log(`${this.name} is guiding ${this.owner}`);
  }
}

const buddy = new GuideDog('Buddy', 'Golden Retriever', 'Ale');
buddy.eat(); // Animal + Dog eating
buddy.bark(); // Dog bark
buddy.guide(); // GuideDog guide

// instanceof checks the prototype chain
console.log(buddy instanceof GuideDog); // true
console.log(buddy instanceof Dog); // true
console.log(buddy instanceof Animal); // true
console.log(buddy instanceof Object); // true
```

**Object.create() for Prototypal Inheritance**

```typescript
// Pure prototypal inheritance (no constructors)
const animalMethods = {
  eat() {
    console.log(`${this.name} is eating`);
  },
  sleep() {
    console.log(`${this.name} is sleeping`);
  },
};

const dogMethods = Object.create(animalMethods);
dogMethods.bark = function () {
  console.log(`${this.name} says woof!`);
};

function createDog(name: string, breed: string) {
  const dog = Object.create(dogMethods);
  dog.name = name;
  dog.breed = breed;
  return dog;
}

const max = createDog('Max', 'Beagle');
max.eat(); // inherited from animalMethods
max.bark(); // from dogMethods

// Factory function pattern
const person = {
  init(name: string, age: number) {
    this.name = name;
    this.age = age;
    return this;
  },
  greet() {
    return `Hi, I'm ${this.name}`;
  },
};

const ale = Object.create(person).init('Ale', 30);
console.log(ale.greet()); // "Hi, I'm Ale"
```

**Mixins - Multiple Inheritance**

```typescript
// JavaScript doesn't support multiple inheritance directly
// But we can use mixins

const CanSwim = {
  swim() {
    console.log(`${this.name} is swimming`);
  },
};

const CanFly = {
  fly() {
    console.log(`${this.name} is flying`);
  },
};

class Bird {
  constructor(public name: string) {}
}

// Add mixin methods to prototype
Object.assign(Bird.prototype, CanFly);

class Duck extends Bird {}

// Add multiple mixins
Object.assign(Duck.prototype, CanSwim, CanFly);

const donald = new Duck('Donald');
donald.fly(); // "Donald is flying"
donald.swim(); // "Donald is swimming"

// Better mixin pattern with TypeScript
type Constructor<T = {}> = new (...args: any[]) => T;

function Flying<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    fly() {
      console.log('Flying!');
    }
  };
}

function Swimming<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    swim() {
      console.log('Swimming!');
    }
  };
}

class Animal {
  constructor(public name: string) {}
}

const FlyingSwimmingAnimal = Swimming(Flying(Animal));
const penguin = new FlyingSwimmingAnimal('Penguin');
penguin.swim();
```

**Prototype Manipulation**

```typescript
// Adding methods to existing prototypes (be careful!)
Array.prototype.first = function () {
  return this[0];
};

Array.prototype.last = function () {
  return this[this.length - 1];
};

console.log([1, 2, 3].first()); // 1
console.log([1, 2, 3].last()); // 3

// But this is dangerous - can break code that iterates over arrays!
const arr = [1, 2, 3];
for (let key in arr) {
  console.log(key); // 0, 1, 2, "first", "last" - unwanted properties!
}

// Use Object.defineProperty for safer additions
Object.defineProperty(Array.prototype, 'firstSafe', {
  value: function () {
    return this[0];
  },
  enumerable: false, // Won't show up in for-in loops
  configurable: true,
  writable: true,
});

for (let key in arr) {
  console.log(key); // Only 0, 1, 2 now
}

// Modern approach: don't modify native prototypes, use utility functions
function first<T>(arr: T[]): T | undefined {
  return arr[0];
}
```

**Property Shadowing**

```typescript
function Parent() {
  this.parentProp = 'parent';
}

Parent.prototype.sharedProp = 'shared';

const child = new Parent();
console.log(child.sharedProp); // "shared"

// Shadow the prototype property
child.sharedProp = 'child';
console.log(child.sharedProp); // "child" - own property shadows prototype

// Original prototype unchanged
console.log(Parent.prototype.sharedProp); // "shared"

// Delete own property to reveal prototype property
delete child.sharedProp;
console.log(child.sharedProp); // "shared" - from prototype again
```

### Common Pitfalls

1. **Forgetting to set constructor**

```typescript
function Parent() {}
function Child() {}

Child.prototype = Object.create(Parent.prototype);
// Forgot to set constructor!

const child = new Child();
console.log(child.constructor === Child); // false!
console.log(child.constructor === Parent); // true - wrong!

// Fix:
Child.prototype.constructor = Child;
```

2. **Modifying prototype after creating instances**

```typescript
function Dog(name: string) {
  this.name = name;
}

const rex = new Dog('Rex');

// Add method to prototype
Dog.prototype.bark = function () {
  console.log('Woof!');
};

rex.bark(); // Works! Existing instances get new methods

// But reassigning prototype doesn't affect existing instances
Dog.prototype = {
  constructor: Dog,
  newMethod() {
    console.log('New!');
  },
};

rex.newMethod(); // Error! rex still points to old prototype

const max = new Dog('Max');
max.newMethod(); // Works - uses new prototype
```

3. **Arrow functions as prototype methods**

```typescript
function Counter() {
  this.count = 0;
}

// Bad - arrow function doesn't use the right 'this'
Counter.prototype.increment = () => {
  this.count++; // 'this' is not the instance!
};

// Good - use regular function
Counter.prototype.increment = function () {
  this.count++;
};
```

### Best Practices

1. **Use classes instead of constructor functions** (cleaner syntax)
2. **Don't modify native prototypes** (breaks code, use utility functions)
3. **Prefer composition over inheritance** (more flexible)
4. **Use Object.create() for pure prototypal patterns**

```typescript
// Modern approach: Composition over Inheritance
class Animal {
  constructor(public name: string) {}
}

// Instead of inheritance, use composition
interface Swimmer {
  swim(): void;
}

interface Flyer {
  fly(): void;
}

class Duck extends Animal implements Swimmer, Flyer {
  private swimmer = {
    swim: () => console.log(`${this.name} swims`),
  };

  private flyer = {
    fly: () => console.log(`${this.name} flies`),
  };

  swim() {
    this.swimmer.swim();
  }

  fly() {
    this.flyer.fly();
  }
}

// Or use function composition
const createAnimal = (name: string) => ({
  name,
  eat: () => console.log(`${name} eats`),
});

const canSwim = (animal: { name: string }) => ({
  swim: () => console.log(`${animal.name} swims`),
});

const canFly = (animal: { name: string }) => ({
  fly: () => console.log(`${animal.name} flies`),
});

const createDuck = (name: string) => ({
  ...createAnimal(name),
  ...canSwim({ name }),
  ...canFly({ name }),
});

const duck = createDuck('Donald');
duck.eat();
duck.swim();
duck.fly();
```

---

## Asynchronous JavaScript

### Simple Explanation

Imagine you're at a restaurant:

- **Synchronous**: You order food, stand at the counter waiting until it's ready, blocking everyone behind you
- **Asynchronous**: You order food, get a number, sit down, and do other things while waiting. When ready, you're called

JavaScript is single-threaded (one waiter) but can handle multiple tasks by not waiting for slow operations:

- Making API calls (waiting for server)
- Reading files (waiting for disk)
- Timers (waiting for time to pass)

### Technical Explanation

JavaScript uses an event loop and callback queue to handle asynchronous operations. When an async operation completes, its callback is added to the queue and executed when the call stack is clear.

**Evolution of async patterns:**

1. Callbacks (2009-)
2. Promises (ES6/2015)
3. Async/Await (ES8/2017)

### Merged Explanation

**Callbacks (The Old Way)**

```typescript
// Callback pattern
function fetchUser(
  id: number,
  callback: (error: Error | null, user?: any) => void,
) {
  setTimeout(() => {
    if (id <= 0) {
      callback(new Error('Invalid ID'));
    } else {
      callback(null, { id, name: 'Ale' });
    }
  }, 1000);
}

// Usage
fetchUser(1, (error, user) => {
  if (error) {
    console.error(error);
    return;
  }
  console.log(user);
});

// Callback Hell (Pyramid of Doom)
fetchUser(1, (error, user) => {
  if (error) return console.error(error);

  fetchUserPosts(user.id, (error, posts) => {
    if (error) return console.error(error);

    fetchPostComments(posts[0].id, (error, comments) => {
      if (error) return console.error(error);

      fetchCommentAuthor(comments[0].authorId, (error, author) => {
        if (error) return console.error(error);
        console.log(author); // Finally!
      });
    });
  });
});
```

**Promises**

```typescript
// Promise creation
function fetchUserPromise(id: number): Promise<any> {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (id <= 0) {
        reject(new Error('Invalid ID'));
      } else {
        resolve({ id, name: 'Ale' });
      }
    }, 1000);
  });
}

// Promise usage
fetchUserPromise(1)
  .then((user) => {
    console.log(user);
    return fetchUserPosts(user.id); // Return promise for chaining
  })
  .then((posts) => {
    console.log(posts);
    return fetchPostComments(posts[0].id);
  })
  .then((comments) => {
    console.log(comments);
    return fetchCommentAuthor(comments[0].authorId);
  })
  .then((author) => {
    console.log(author);
  })
  .catch((error) => {
    console.error('Error:', error); // Single error handler!
  })
  .finally(() => {
    console.log('Cleanup'); // Always runs
  });

// Promise states
// 1. Pending: initial state
// 2. Fulfilled: operation completed successfully
// 3. Rejected: operation failed

// Creating immediate promises
const resolvedPromise = Promise.resolve(42);
const rejectedPromise = Promise.reject(new Error('Failed'));
```

**Async/Await (Modern Way)**

```typescript
// Async function returns a Promise
async function fetchAndDisplayUser(id: number): Promise<void> {
  try {
    const user = await fetchUserPromise(id); // Wait for promise
    console.log(user);

    const posts = await fetchUserPosts(user.id);
    console.log(posts);

    const comments = await fetchPostComments(posts[0].id);
    console.log(comments);

    const author = await fetchCommentAuthor(comments[0].authorId);
    console.log(author);
  } catch (error) {
    console.error('Error:', error);
  } finally {
    console.log('Cleanup');
  }
}

// Call async function
fetchAndDisplayUser(1);

// Async functions always return Promises
async function getValue(): Promise<number> {
  return 42; // Automatically wrapped in Promise.resolve(42)
}

getValue().then((value) => console.log(value)); // 42

// await can only be used in async functions
function normalFunction() {
  // await fetchUser(1); // Error!
}

// But top-level await is allowed in modules (ES2022)
// const user = await fetchUser(1); // Works in modules
```

**Promise Combinators**

```typescript
// Promise.all - wait for all promises (fails if any fails)
async function fetchMultipleUsers() {
  const promises = [
    fetchUserPromise(1),
    fetchUserPromise(2),
    fetchUserPromise(3),
  ];

  try {
    const users = await Promise.all(promises);
    console.log(users); // [user1, user2, user3]
  } catch (error) {
    // If any promise rejects, catch the first rejection
    console.error('One of the requests failed:', error);
  }
}

// Promise.allSettled - wait for all, never fails
async function fetchUsersAllSettled() {
  const promises = [
    fetchUserPromise(1),
    fetchUserPromise(2),
    fetchUserPromise(-1), // This will fail
  ];

  const results = await Promise.allSettled(promises);
  console.log(results);
  // [
  //   { status: 'fulfilled', value: user1 },
  //   { status: 'fulfilled', value: user2 },
  //   { status: 'rejected', reason: Error }
  // ]

  results.forEach((result, index) => {
    if (result.status === 'fulfilled') {
      console.log(`User ${index}:`, result.value);
    } else {
      console.error(`User ${index} failed:`, result.reason);
    }
  });
}

// Promise.race - returns first settled promise
async function fetchWithTimeout(url: string, timeout: number) {
  const fetchPromise = fetch(url);
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => reject(new Error('Timeout')), timeout);
  });

  return Promise.race([fetchPromise, timeoutPromise]);
}

// Usage
try {
  const response = await fetchWithTimeout('https://api.example.com/data', 5000);
  console.log(await response.json());
} catch (error) {
  console.error('Request failed or timed out:', error);
}

// Promise.any - returns first fulfilled promise (ignores rejections)
async function fetchFromMultipleSources() {
  const promises = [
    fetch('https://api1.example.com/user'),
    fetch('https://api2.example.com/user'),
    fetch('https://api3.example.com/user'),
  ];

  try {
    // First successful response wins
    const response = await Promise.any(promises);
    return await response.json();
  } catch (error) {
    // All promises rejected
    console.error('All APIs failed:', error);
  }
}
```

**Practical Async Patterns**

```typescript
// 1. Sequential vs Parallel execution
async function sequentialFetch() {
  console.time('sequential');

  const user = await fetchUserPromise(1); // Wait 1s
  const posts = await fetchUserPosts(1); // Wait 1s
  const comments = await fetchPostComments(1); // Wait 1s

  console.timeEnd('sequential'); // ~3 seconds
  return { user, posts, comments };
}

async function parallelFetch() {
  console.time('parallel');

  // Start all at once
  const [user, posts, comments] = await Promise.all([
    fetchUserPromise(1),
    fetchUserPosts(1),
    fetchPostComments(1),
  ]);

  console.timeEnd('parallel'); // ~1 second (all run simultaneously)
  return { user, posts, comments };
}

// 2. Retry logic
async function fetchWithRetry<T>(
  fn: () => Promise<T>,
  retries: number = 3,
  delay: number = 1000,
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    if (retries <= 0) throw error;

    console.log(`Retrying... (${retries} attempts left)`);
    await new Promise((resolve) => setTimeout(resolve, delay));
    return fetchWithRetry(fn, retries - 1, delay);
  }
}

// Usage
const user = await fetchWithRetry(() => fetchUserPromise(1), 3);

// 3. Timeout wrapper
function withTimeout<T>(promise: Promise<T>, ms: number): Promise<T> {
  return Promise.race([
    promise,
    new Promise<T>((_, reject) =>
      setTimeout(() => reject(new Error('Timeout')), ms),
    ),
  ]);
}

// Usage
try {
  const user = await withTimeout(fetchUserPromise(1), 5000);
} catch (error) {
  console.error('Request timed out');
}

// 4. Queue with concurrency limit
class AsyncQueue {
  private queue: Array<() => Promise<any>> = [];
  private running: number = 0;

  constructor(private concurrency: number) {}

  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      this.run();
    });
  }

  private async run() {
    while (this.running < this.concurrency && this.queue.length > 0) {
      const task = this.queue.shift();
      if (task) {
        this.running++;
        task().finally(() => {
          this.running--;
          this.run();
        });
      }
    }
  }
}

// Usage
const queue = new AsyncQueue(2); // Max 2 concurrent requests

const promises = [1, 2, 3, 4, 5].map((id) =>
  queue.add(() => fetchUserPromise(id)),
);

const users = await Promise.all(promises);

// 5. Debounced async function
function debounceAsync<T extends (...args: any[]) => Promise<any>>(
  fn: T,
  delay: number,
): (...args: Parameters<T>) => Promise<ReturnType<T>> {
  let timeoutId: NodeJS.Timeout | null = null;

  return (...args: Parameters<T>): Promise<ReturnType<T>> => {
    return new Promise((resolve, reject) => {
      if (timeoutId) clearTimeout(timeoutId);

      timeoutId = setTimeout(async () => {
        try {
          const result = await fn(...args);
          resolve(result);
        } catch (error) {
          reject(error);
        }
      }, delay);
    });
  };
}

// Usage (e.g., search as user types)
const searchDebounced = debounceAsync(async (query: string) => {
  const response = await fetch(`/api/search?q=${query}`);
  return response.json();
}, 300);
```

**Error Handling Patterns**

```typescript
// 1. Try-catch with async/await
async function handleErrors() {
  try {
    const user = await fetchUserPromise(1);
    const posts = await fetchUserPosts(user.id);
    return posts;
  } catch (error) {
    if (error instanceof TypeError) {
      console.error('Network error:', error);
    } else if (error instanceof SyntaxError) {
      console.error('Parse error:', error);
    } else {
      console.error('Unknown error:', error);
    }
    throw error; // Re-throw if needed
  }
}

// 2. Multiple try-catch blocks
async function partialErrorHandling() {
  let user;

  try {
    user = await fetchUserPromise(1);
  } catch (error) {
    console.error('Failed to fetch user:', error);
    return; // Exit early
  }

  try {
    const posts = await fetchUserPosts(user.id);
    return posts;
  } catch (error) {
    console.error('Failed to fetch posts:', error);
    return []; // Return default value
  }
}

// 3. Error-first callback style with promises
function safeAwait<T>(promise: Promise<T>): Promise<[Error | null, T | null]> {
  return promise
    .then((data) => [null, data] as [null, T])
    .catch((error) => [error, null] as [Error, null]);
}

// Usage (no try-catch needed)
async function useResult() {
  const [error, user] = await safeAwait(fetchUserPromise(1));

  if (error) {
    console.error('Error:', error);
    return;
  }

  console.log('User:', user);
}
```

### Common Pitfalls

1. **Forgetting await**

```typescript
// Bad - returns Promise, not user
async function getUser() {
  const user = fetchUserPromise(1); // Forgot await!
  console.log(user); // Promise { <pending> }
  return user;
}

// Good
async function getUserFixed() {
  const user = await fetchUserPromise(1);
  console.log(user); // { id: 1, name: "Ale" }
  return user;
}
```

2. **Using await in loops unnecessarily**

```typescript
// Bad - sequential (slow)
async function fetchAllUsers(ids: number[]) {
  const users = [];
  for (const id of ids) {
    const user = await fetchUserPromise(id); // Waits for each
    users.push(user);
  }
  return users; // Takes n * 1 second
}

// Good - parallel (fast)
async function fetchAllUsersParallel(ids: number[]) {
  const promises = ids.map((id) => fetchUserPromise(id));
  return Promise.all(promises); // Takes 1 second
}

// When sequential is actually needed
async function processInOrder(ids: number[]) {
  for (const id of ids) {
    const user = await fetchUserPromise(id);
    await processUser(user); // Must wait for processing before next
  }
}
```

3. **Not handling promise rejections**

```typescript
// Bad - unhandled promise rejection
async function riskyFunction() {
  fetchUserPromise(-1); // Returns rejected promise but not awaited
  console.log('This runs before the error is thrown');
}

// Good - handle the promise
async function safeFunction() {
  try {
    await fetchUserPromise(-1);
  } catch (error) {
    console.error('Handled:', error);
  }
}

// Or at least catch it
fetchUserPromise(-1).catch((error) => console.error('Handled:', error));
```

4. **Returning in finally**

```typescript
// Bad - finally return overrides try/catch return
async function badFinally() {
  try {
    return await fetchUserPromise(1); // This is ignored!
  } catch (error) {
    return null;
  } finally {
    return 'always this'; // Overrides everything
  }
}

// Good - don't return in finally
async function goodFinally() {
  try {
    return await fetchUserPromise(1);
  } catch (error) {
    return null;
  } finally {
    console.log('Cleanup'); // Side effects only
  }
}
```

### Best Practices

1. **Use async/await over raw promises** (more readable)
2. **Handle errors with try-catch**
3. **Run independent operations in parallel**
4. **Add timeout to prevent hanging**
5. **Use Promise.allSettled when some failures are acceptable**

```typescript
// Comprehensive example
interface User {
  id: number;
  name: string;
}

interface Post {
  id: number;
  userId: number;
  title: string;
}

class UserService {
  private readonly timeout = 5000;

  async getUserWithPosts(
    userId: number,
  ): Promise<{ user: User; posts: Post[] }> {
    try {
      // Parallel fetch with timeout
      const [user, posts] = await withTimeout(
        Promise.all([this.fetchUser(userId), this.fetchPosts(userId)]),
        this.timeout,
      );

      return { user, posts };
    } catch (error) {
      console.error(`Failed to fetch user ${userId}:`, error);
      throw new Error(`User data unavailable: ${error.message}`);
    }
  }

  private async fetchUser(id: number): Promise<User> {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  }

  private async fetchPosts(userId: number): Promise<Post[]> {
    const response = await fetch(`/api/users/${userId}/posts`);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  }
}
```

---

## Event Loop

### Simple Explanation

The event loop is like a restaurant kitchen:

- **Call Stack**: The chef's hands (can only do one thing at a time)
- **Web APIs**: The oven, dishwasher (run in background)
- **Callback Queue**: Finished dishes waiting to be served
- **Event Loop**: The manager checking if chef is free to handle next dish

When you order (make async call):

1. Chef sends it to oven (Web API)
2. Chef continues with other orders (call stack keeps running)
3. Oven finishes (callback added to queue)
4. When chef is free (stack empty), manager serves the dish (event loop moves callback to stack)

### Technical Explanation

JavaScript is single-threaded but can handle async operations through:

1. **Call Stack**: Tracks function execution (LIFO)
2. **Heap**: Memory allocation for objects
3. **Web APIs**: Browser-provided async features (setTimeout, fetch, DOM events)
4. **Callback Queue** (Task Queue): Callbacks waiting to execute
5. **Microtask Queue**: Promise callbacks (higher priority)
6. **Event Loop**: Monitors stack and queues, moves callbacks when stack is empty

**Order of execution:**

1. Synchronous code (call stack)
2. Microtasks (promises, queueMicrotask)
3. Macrotasks (setTimeout, setInterval, I/O)

### Merged Explanation

**Understanding the Call Stack**

```typescript
function third() {
  console.log('Third');
}

function second() {
  third();
  console.log('Second');
}

function first() {
  second();
  console.log('First');
}

first();

// Call stack visualization:
// 1. first() pushed
// 2. second() pushed
// 3. third() pushed
// 3. third() popped (logs "Third")
// 2. second() popped (logs "Second")
// 1. first() popped (logs "First")

// Output:
// Third
// Second
// First
```

**Async Operations and Web APIs**

```typescript
console.log('Start');

setTimeout(() => {
  console.log('Timeout 1');
}, 0);

setTimeout(() => {
  console.log('Timeout 2');
}, 0);

console.log('End');

// Execution:
// 1. "Start" - call stack
// 2. setTimeout - sent to Web API
// 3. setTimeout - sent to Web API
// 4. "End" - call stack
// 5. Stack empty, check queues
// 6. "Timeout 1" - from callback queue
// 7. "Timeout 2" - from callback queue

// Output:
// Start
// End
// Timeout 1
// Timeout 2
```

**Microtasks vs Macrotasks**

```typescript
console.log('Script start');

setTimeout(() => {
  console.log('setTimeout'); // Macrotask
}, 0);

Promise.resolve()
  .then(() => {
    console.log('Promise 1'); // Microtask
  })
  .then(() => {
    console.log('Promise 2'); // Microtask
  });

console.log('Script end');

// Execution order:
// 1. Synchronous code first
// 2. Then microtasks (promises)
// 3. Then macrotasks (setTimeout)

// Output:
// Script start
// Script end
// Promise 1
// Promise 2
// setTimeout

// Microtasks have higher priority!
```

**Complex Event Loop Example**

```typescript
console.log('1');

setTimeout(() => {
  console.log('2');
  Promise.resolve().then(() => {
    console.log('3');
  });
}, 0);

new Promise((resolve) => {
  console.log('4'); // Promise executor runs synchronously!
  resolve('5');
}).then((val) => {
  console.log(val);

  setTimeout(() => {
    console.log('6');
  }, 0);
});

setTimeout(() => {
  console.log('7');
  Promise.resolve().then(() => {
    console.log('8');
  });
}, 0);

console.log('9');

// Output:
// 1 (sync)
// 4 (sync - promise executor)
// 9 (sync)
// 5 (microtask - promise then)
// 2 (macrotask - first setTimeout)
// 3 (microtask - promise inside setTimeout)
// 7 (macrotask - second setTimeout)
// 8 (microtask - promise inside setTimeout)
// 6 (macrotask - setTimeout from promise then)
```

**Execution Order Visualization**

```typescript
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end'); // Microtask
}

async function async2() {
  console.log('async2');
}

console.log('script start');

setTimeout(() => {
  console.log('setTimeout');
}, 0);

async1();

new Promise((resolve) => {
  console.log('promise1');
  resolve();
}).then(() => {
  console.log('promise2');
});

console.log('script end');

// Output:
// script start (1)
// async1 start (2)
// async2 (3)
// promise1 (4)
// script end (5)
// async1 end (6 - microtask)
// promise2 (7 - microtask)
// setTimeout (8 - macrotask)

// Why?
// 1-5: Synchronous code
// 6-7: Microtask queue (promises, async/await)
// 8: Macrotask queue (setTimeout)
```

**Visual Model of Event Loop**

```typescript
// Simplified event loop pseudocode
while (true) {
  // 1. Execute all synchronous code (call stack)
  while (callStack.isNotEmpty()) {
    execute(callStack.pop());
  }

  // 2. Process all microtasks
  while (microtaskQueue.isNotEmpty()) {
    const microtask = microtaskQueue.shift();
    callStack.push(microtask);
    execute(microtask);

    // New microtasks can be added while processing
    // and will be executed in the same cycle
  }

  // 3. Process one macrotask
  if (macrotaskQueue.isNotEmpty()) {
    const macrotask = macrotaskQueue.shift();
    callStack.push(macrotask);
    execute(macrotask);
  }

  // 4. Render if needed (in browsers)
  if (needsRender()) {
    render();
  }

  // 5. Repeat
}
```

**Microtask vs Macrotask Examples**

```typescript
// Microtasks (higher priority):
// - Promise.then/catch/finally
// - queueMicrotask()
// - MutationObserver
// - process.nextTick (Node.js)

// Macrotasks (lower priority):
// - setTimeout
// - setInterval
// - setImmediate (Node.js)
// - requestAnimationFrame (browser)
// - I/O operations
// - UI rendering

// Example: Starving macrotasks
function recursiveMicrotask() {
  console.log('Microtask');
  Promise.resolve().then(recursiveMicrotask);
}

setTimeout(() => {
  console.log('This might never run!'); // Macrotask starved
}, 0);

// recursiveMicrotask(); // Uncomment to block setTimeout
```

**Practical Event Loop Patterns**

```typescript
// 1. Breaking up long tasks
function processLargeArray(array: number[]) {
  const batchSize = 1000;
  let index = 0;

  function processBatch() {
    const endIndex = Math.min(index + batchSize, array.length);

    for (; index < endIndex; index++) {
      // Process item
      array[index] *= 2;
    }

    if (index < array.length) {
      // Schedule next batch as macrotask (allows UI to update)
      setTimeout(processBatch, 0);
    }
  }

  processBatch();
}

// 2. Ensuring order with microtasks
async function ensureOrder() {
  console.log(1);

  // These run in order because they're microtasks
  await Promise.resolve();
  console.log(2);

  await Promise.resolve();
  console.log(3);

  // But setTimeout breaks the order
  setTimeout(() => console.log(4), 0);

  await Promise.resolve();
  console.log(5);

  // Output: 1, 2, 3, 5, 4
}

// 3. Debouncing with microtasks
let microtaskQueued = false;
let data: any[] = [];

function queueDataUpdate(newData: any) {
  data.push(newData);

  if (!microtaskQueued) {
    microtaskQueued = true;
    queueMicrotask(() => {
      processData(data);
      data = [];
      microtaskQueued = false;
    });
  }
}

// All updates in same tick are batched
queueDataUpdate({ id: 1 });
queueDataUpdate({ id: 2 });
queueDataUpdate({ id: 3 });
// processData called once with all three
```

**Node.js Event Loop (More Complex)**

```typescript
// Node.js has additional phases:
// 1. Timers (setTimeout, setInterval)
// 2. Pending callbacks (I/O)
// 3. Idle, prepare (internal)
// 4. Poll (retrieve new I/O events)
// 5. Check (setImmediate)
// 6. Close callbacks

// process.nextTick runs before all other phases
process.nextTick(() => {
  console.log('nextTick'); // Runs first
});

setImmediate(() => {
  console.log('setImmediate'); // Runs in check phase
});

setTimeout(() => {
  console.log('setTimeout'); // Runs in timers phase
}, 0);

Promise.resolve().then(() => {
  console.log('Promise'); // Runs after nextTick
});

// Output:
// nextTick
// Promise
// setTimeout
// setImmediate
```

### Common Pitfalls

1. **Blocking the event loop**

```typescript
// Bad - blocks everything
function blockingOperation() {
  const start = Date.now();
  while (Date.now() - start < 5000) {
    // Busy wait - blocks event loop!
  }
  console.log('Done');
}

// Good - allows event loop to continue
function nonBlockingOperation() {
  return new Promise((resolve) => {
    setTimeout(() => {
      console.log('Done');
      resolve();
    }, 5000);
  });
}
```

2. **Assuming setTimeout(0) runs immediately**

```typescript
console.log('A');

setTimeout(() => {
  console.log('B'); // Not immediate!
}, 0);

Promise.resolve().then(() => {
  console.log('C'); // Runs before B
});

console.log('D');

// Output: A, D, C, B
// setTimeout is a macrotask, promise is a microtask
```

3. **Memory leaks with uncleared timers**

```typescript
// Bad - timer keeps running
function startTimer() {
  setInterval(() => {
    console.log('Tick');
  }, 1000);
}

// Good - store reference and clear
function startTimerProper() {
  const timerId = setInterval(() => {
    console.log('Tick');
  }, 1000);

  return () => clearInterval(timerId); // Cleanup function
}

const cleanup = startTimerProper();
// Later...
cleanup();
```

### Best Practices

1. **Don't block the event loop** (break up long tasks)
2. **Understand microtask vs macrotask priority**
3. **Use async/await for cleaner async code**
4. **Clear timers and intervals when done**
5. **Use requestAnimationFrame for animations** (browser)

```typescript
// Good pattern: Non-blocking CPU-intensive work
async function processHugeDataset(data: any[]) {
  const chunks = chunkArray(data, 1000);

  for (const chunk of chunks) {
    await new Promise((resolve) => {
      // Process chunk
      processChunk(chunk);

      // Yield to event loop
      setTimeout(resolve, 0);
    });
  }
}

// Worker Threads for truly parallel work (Node.js)
// Or Web Workers (browser)
```

---

_Due to length constraints, I'll continue with the remaining sections in the file. Let me proceed with the creation._

## ES6+ Features

### Simple Explanation

ES6 (ECMAScript 2015) and later versions brought major improvements to JavaScript, like upgrading from a flip phone to a smartphone. These features make code cleaner, more readable, and less error-prone.

Think of it as learning new vocabulary words that express ideas more clearly than before - you could explain things the old way, but the new way is much clearer.

### Technical Explanation

ES6+ introduced:

- Better variable declarations (let, const)
- Arrow functions
- Classes
- Template literals
- Destructuring
- Spread/rest operators
- Modules
- Promises
- And many more...

### Merged Explanation

**1. Template Literals**

```typescript
// Old way
const name = 'Ale';
const age = 30;
const message = 'Hello, my name is ' + name + " and I'm " + age + ' years old.';

// New way
const messageBetter = `Hello, my name is ${name} and I'm ${age} years old.`;

// Multi-line strings
const html = `
  <div>
    <h1>${name}</h1>
    <p>Age: ${age}</p>
  </div>
`;

// Tagged templates
function highlight(strings: TemplateStringsArray, ...values: any[]) {
  return strings.reduce((result, str, i) => {
    return `${result}${str}<strong>${values[i] || ''}</strong>`;
  }, '');
}

const highlighted = highlight`Name: ${name}, Age: ${age}`;
// "Name: <strong>Ale</strong>, Age: <strong>30</strong>"
```

**2. Destructuring**

```typescript
// Array destructuring
const numbers = [1, 2, 3, 4, 5];

const [first, second] = numbers; // 1, 2
const [, , third] = numbers; // 3
const [head, ...tail] = numbers; // 1, [2, 3, 4, 5]

// Object destructuring
const user = {
  name: 'Ale',
  age: 30,
  email: 'ale@example.com',
  address: {
    city: 'Paris',
    country: 'France',
  },
};

const { name, age } = user;
const { email: userEmail } = user; // Rename
const { phone = 'N/A' } = user; // Default value
const {
  address: { city },
} = user; // Nested

// Function parameters
function greet({ name, age }: { name: string; age: number }) {
  return `${name} is ${age} years old`;
}

greet(user);

// Swap variables
let a = 1,
  b = 2;
[a, b] = [b, a]; // a = 2, b = 1
```

**3. Spread and Rest Operators**

```typescript
// Spread in arrays
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2]; // [1, 2, 3, 4, 5, 6]

// Clone array
const clone = [...arr1];

// Spread in objects
const user = { name: 'Ale', age: 30 };
const updatedUser = { ...user, age: 31, city: 'Paris' };
// { name: "Ale", age: 31, city: "Paris" }

// Rest in function parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((total, n) => total + n, 0);
}

sum(1, 2, 3, 4, 5); // 15

// Rest in destructuring
const { name, ...rest } = user;
// name = "Ale", rest = { age: 30 }
```

**4. Default Parameters**

```typescript
// Old way
function createUser(name, role) {
  role = role || 'user';
  return { name, role };
}

// New way
function createUserBetter(name: string, role: string = 'user') {
  return { name, role };
}

// With destructuring
function setupConfig({ timeout = 3000, retries = 3, verbose = false } = {}) {
  return { timeout, retries, verbose };
}

setupConfig(); // Uses all defaults
setupConfig({ timeout: 5000 }); // { timeout: 5000, retries: 3, verbose: false }
```

**5. Enhanced Object Literals**

```typescript
const name = 'Ale';
const age = 30;

// Old way
const user1 = {
  name: name,
  age: age,
  greet: function () {
    return 'Hello';
  },
};

// New way
const user2 = {
  name, // Shorthand property
  age,
  greet() {
    // Shorthand method
    return 'Hello';
  },
  // Computed property names
  [`role_${name.toLowerCase()}`]: 'admin',
};

// Dynamic property names
function createObject(key: string, value: any) {
  return {
    [key]: value,
    [`${key}_timestamp`]: Date.now(),
  };
}
```

**6. Optional Chaining and Nullish Coalescing**

```typescript
interface User {
  name: string;
  address?: {
    street?: string;
    city?: string;
  };
  contacts?: string[];
}

const user: User = { name: 'Ale' };

// Optional chaining (?.)
const city = user.address?.city; // undefined (no error)
const first = user.contacts?.[0]; // undefined

// Without optional chaining
const cityOld = user.address && user.address.city;

// Nullish coalescing (??)
const city2 = user.address?.city ?? 'Unknown'; // "Unknown"

// Difference from ||
const value1 = 0 || 10; // 10 (0 is falsy)
const value2 = 0 ?? 10; // 0 (only null/undefined)

// Combined
const userCity = user?.address?.city ?? 'Unknown';
```

**7. For...of Loop**

```typescript
const numbers = [1, 2, 3, 4, 5];

// for...of (iterates values)
for (const num of numbers) {
  console.log(num); // 1, 2, 3, 4, 5
}

// for...in (iterates keys) - avoid for arrays!
for (const key in numbers) {
  console.log(key); // "0", "1", "2", "3", "4"
}

// With entries
for (const [index, value] of numbers.entries()) {
  console.log(index, value);
}

// Objects are not iterable by default
const user = { name: 'Ale', age: 30 };
// for (const prop of user) {} // Error!

// Make iterable
for (const [key, value] of Object.entries(user)) {
  console.log(key, value);
}
```

**8. Symbols**

```typescript
// Unique identifiers
const id1 = Symbol('id');
const id2 = Symbol('id');
console.log(id1 === id2); // false - always unique

// Hidden object properties
const user = {
  name: 'Ale',
  [id1]: 'secret_id_123',
};

console.log(Object.keys(user)); // ["name"] - symbol not included
console.log(user[id1]); // "secret_id_123"

// Well-known symbols
class Collection {
  private items: any[] = [];

  add(item: any) {
    this.items.push(item);
  }

  // Make class iterable
  *[Symbol.iterator]() {
    for (const item of this.items) {
      yield item;
    }
  }
}

const collection = new Collection();
collection.add(1);
collection.add(2);

for (const item of collection) {
  console.log(item); // 1, 2
}
```

**9. Generators and Iterators**

```typescript
// Generator function
function* numberGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = numberGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }

// Infinite generator
function* infiniteSequence() {
  let i = 0;
  while (true) {
    yield i++;
  }
}

// Practical use: pagination
async function* fetchPages(url: string) {
  let page = 1;
  let hasMore = true;

  while (hasMore) {
    const response = await fetch(`${url}?page=${page}`);
    const data = await response.json();

    yield data.items;

    hasMore = data.hasMore;
    page++;
  }
}

// Usage
for await (const items of fetchPages('/api/users')) {
  console.log(items);
}

// Generator for range
function* range(start: number, end: number) {
  for (let i = start; i <= end; i++) {
    yield i;
  }
}

const numbers = [...range(1, 5)]; // [1, 2, 3, 4, 5]
```

**10. Map and Set**

```typescript
// Map - key-value pairs with any type as key
const map = new Map();

map.set('name', 'Ale');
map.set(1, 'one');
map.set(true, 'yes');

console.log(map.get('name')); // "Ale"
console.log(map.has(1)); // true
console.log(map.size); // 3

// Object keys
const obj = { id: 1 };
map.set(obj, 'object key');
console.log(map.get(obj)); // "object key"

// Iteration
for (const [key, value] of map) {
  console.log(key, value);
}

// Set - unique values
const set = new Set([1, 2, 3, 3, 4, 4, 5]);
console.log(set); // Set {1, 2, 3, 4, 5}

set.add(6);
set.delete(1);
console.log(set.has(2)); // true

// Convert to array
const uniqueNumbers = [...set];

// Practical use: remove duplicates
const withDuplicates = [1, 2, 2, 3, 3, 3, 4];
const unique = [...new Set(withDuplicates)];

// WeakMap and WeakSet (keys can be garbage collected)
const weakMap = new WeakMap();
let obj1 = { name: 'Ale' };
weakMap.set(obj1, 'metadata');
obj1 = null; // Object can now be garbage collected
```

**11. Proxy and Reflect**

```typescript
// Proxy - intercept object operations
const handler = {
  get(target: any, property: string) {
    console.log(`Getting ${property}`);
    return target[property];
  },
  set(target: any, property: string, value: any) {
    console.log(`Setting ${property} to ${value}`);
    if (property === 'age' && typeof value !== 'number') {
      throw new TypeError('Age must be a number');
    }
    target[property] = value;
    return true;
  },
};

const user = new Proxy({ name: 'Ale', age: 30 }, handler);

console.log(user.name); // Logs: "Getting name", returns "Ale"
user.age = 31; // Logs: "Setting age to 31"
// user.age = "invalid"; // Throws TypeError

// Validation proxy
function createValidator(target: any, validator: any) {
  return new Proxy(target, {
    set(obj, prop, value) {
      if (validator[prop]) {
        validator[prop](value);
      }
      obj[prop] = value;
      return true;
    },
  });
}

const userValidator = {
  age: (val: any) => {
    if (typeof val !== 'number' || val < 0 || val > 150) {
      throw new Error('Invalid age');
    }
  },
};

const validatedUser = createValidator({}, userValidator);
validatedUser.age = 30; // OK
// validatedUser.age = -5; // Error
```

**12. BigInt (ES2020)**

```typescript
// Numbers larger than Number.MAX_SAFE_INTEGER
const bigNumber = 9007199254740991n;
const alsoHuge = BigInt(9007199254740991);

console.log(bigNumber + 1n); // 9007199254740992n

// Cannot mix BigInt and Number
// bigNumber + 1; // Error
bigNumber + BigInt(1); // OK

// Use case: precise calculations
const price = 999999999999999999n;
const quantity = 3n;
const total = price * quantity;
```

### Common Pitfalls

1. **Destructuring undefined/null**

```typescript
const user = null;
// const { name } = user; // Error!

// Solution: provide default
const { name } = user || {};
```

2. **Spread creating shallow copies**

```typescript
const original = { data: { value: 1 } };
const copy = { ...original };

copy.data.value = 2;
console.log(original.data.value); // 2 - mutated!

// Deep clone needed
const deepCopy = JSON.parse(JSON.stringify(original));
```

3. **Template literals not calling toString**

```typescript
const obj = { name: 'Ale' };
console.log(`Object: ${obj}`); // "Object: [object Object]"

// Need explicit conversion
console.log(`Object: ${JSON.stringify(obj)}`);
```

### Best Practices

1. **Use const by default, let when needed**
2. **Destructure to extract only needed properties**
3. **Use template literals for string interpolation**
4. **Prefer spread over Object.assign/concat**
5. **Use Map for frequent additions/deletions**
6. **Use Set for uniqueness**

---

## Array Methods

### Simple Explanation

Array methods are like Swiss Army knives for working with lists. Instead of writing loops to find, filter, or transform items, you use built-in methods that do the work for you.

Think of array methods as assembly line workers:

- **map**: Transforms each item (paint each car)
- **filter**: Selects certain items (quality control)
- **reduce**: Combines items into one result (assemble parts into final product)
- **find**: Locates first match (find a specific order)
- **forEach**: Does something with each item (label each box)

### Technical Explanation

Modern JavaScript provides functional array methods that:

- Don't mutate the original array (immutable)
- Return new arrays or values
- Accept callback functions
- Enable method chaining
- Make code more declarative and readable

### Merged Explanation

**1. map() - Transform Elements**

```typescript
const numbers = [1, 2, 3, 4, 5];

// Transform each element
const doubled = numbers.map((num) => num * 2);
// [2, 4, 6, 8, 10]

const users = [
  { id: 1, name: 'Ale', age: 30 },
  { id: 2, name: 'Bob', age: 25 },
];

// Extract specific properties
const names = users.map((user) => user.name);
// ["Ale", "Bob"]

// Transform objects
const withEmail = users.map((user) => ({
  ...user,
  email: `${user.name.toLowerCase()}@example.com`,
}));

// With index parameter
const withIndex = numbers.map((num, index) => `${index}: ${num}`);
// ["0: 1", "1: 2", "2: 3", "3: 4", "4: 5"]
```

**2. filter() - Select Elements**

```typescript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Filter by condition
const evens = numbers.filter((num) => num % 2 === 0);
// [2, 4, 6, 8, 10]

const users = [
  { name: 'Ale', age: 30, active: true },
  { name: 'Bob', age: 25, active: false },
  { name: 'Charlie', age: 35, active: true },
];

// Filter active users
const activeUsers = users.filter((user) => user.active);

// Multiple conditions
const youngActiveUsers = users.filter((user) => user.active && user.age < 30);

// Remove falsy values
const mixed = [0, 1, false, 2, '', 3, null, undefined, 4];
const truthy = mixed.filter(Boolean);
// [1, 2, 3, 4]

// Remove duplicates (with indexOf)
const withDuplicates = [1, 2, 2, 3, 3, 3, 4];
const unique = withDuplicates.filter(
  (item, index, arr) => arr.indexOf(item) === index,
);
```

**3. reduce() - Aggregate Elements**

```typescript
const numbers = [1, 2, 3, 4, 5];

// Sum
const sum = numbers.reduce((total, num) => total + num, 0);
// 15

// Product
const product = numbers.reduce((acc, num) => acc * num, 1);
// 120

// Group by property
interface User {
  name: string;
  age: number;
  role: string;
}

const users: User[] = [
  { name: 'Ale', age: 30, role: 'developer' },
  { name: 'Bob', age: 25, role: 'designer' },
  { name: 'Charlie', age: 35, role: 'developer' },
];

const byRole = users.reduce((acc, user) => {
  const role = user.role;
  if (!acc[role]) acc[role] = [];
  acc[role].push(user);
  return acc;
}, {} as Record<string, User[]>);
// {
//   developer: [{Ale}, {Charlie}],
//   designer: [{Bob}]
// }

// Count occurrences
const fruits = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple'];
const count = fruits.reduce((acc, fruit) => {
  acc[fruit] = (acc[fruit] || 0) + 1;
  return acc;
}, {} as Record<string, number>);
// { apple: 3, banana: 2, orange: 1 }

// Flatten nested arrays
const nested = [
  [1, 2],
  [3, 4],
  [5, 6],
];
const flattened = nested.reduce((acc, arr) => acc.concat(arr), []);
// [1, 2, 3, 4, 5, 6]

// Better: use flat()
const flattenedBetter = nested.flat();

// Create lookup object
const usersLookup = users.reduce((acc, user) => {
  acc[user.name] = user;
  return acc;
}, {} as Record<string, User>);

// Pipeline of operations
const result = users
  .filter((u) => u.age > 25)
  .map((u) => u.name)
  .reduce((acc, name) => acc + name + ', ', 'Users: ');
// "Users: Ale, Charlie, "
```

**4. find() and findIndex()**

```typescript
const users = [
  { id: 1, name: 'Ale', age: 30 },
  { id: 2, name: 'Bob', age: 25 },
  { id: 3, name: 'Charlie', age: 35 },
];

// Find first match
const user = users.find((u) => u.age > 25);
// { id: 1, name: "Ale", age: 30 }

const notFound = users.find((u) => u.age > 100);
// undefined

// Find index
const index = users.findIndex((u) => u.name === 'Bob');
// 1

// findLast and findLastIndex (ES2023)
const lastOld = users.findLast((u) => u.age > 25);
// { id: 3, name: "Charlie", age: 35 }
```

**5. every() and some()**

```typescript
const numbers = [2, 4, 6, 8, 10];

// Check if all match
const allEven = numbers.every((num) => num % 2 === 0);
// true

const allPositive = numbers.every((num) => num > 0);
// true

// Check if any match
const hasLarge = numbers.some((num) => num > 5);
// true

const hasNegative = numbers.some((num) => num < 0);
// false

// Practical use: validation
interface FormData {
  name: string;
  email: string;
  age: number;
}

const forms: FormData[] = [
  { name: 'Ale', email: 'ale@example.com', age: 30 },
  { name: 'Bob', email: 'bob@example.com', age: 25 },
];

const allValid = forms.every((form) => form.name && form.email && form.age > 0);
```

**6. forEach() - Iterate**

```typescript
const numbers = [1, 2, 3, 4, 5];

// Simple iteration
numbers.forEach((num) => console.log(num));

// With index and array
numbers.forEach((num, index, arr) => {
  console.log(`${index}: ${num} of ${arr.length}`);
});

// Note: cannot break or return early
numbers.forEach((num) => {
  if (num === 3) return; // Only skips current iteration
  console.log(num);
});
// Logs: 1, 2, 4, 5

// Use for...of if you need to break
for (const num of numbers) {
  if (num === 3) break; // Exits loop
  console.log(num);
}
// Logs: 1, 2
```

**7. sort() - MUTATES ORIGINAL**

```typescript
const numbers = [3, 1, 4, 1, 5, 9, 2, 6];

// Default sort (converts to strings!)
numbers.sort();
// [1, 1, 2, 3, 4, 5, 6, 9] - works for single digits

const numbers2 = [10, 5, 40, 25, 1000, 1];
numbers2.sort();
// [1, 10, 1000, 25, 40, 5] - wrong! Sorted as strings

// Numeric sort
numbers2.sort((a, b) => a - b);
// [1, 5, 10, 25, 40, 1000] - correct

// Descending
numbers2.sort((a, b) => b - a);

// Sort objects
interface User {
  name: string;
  age: number;
}

const users: User[] = [
  { name: 'Charlie', age: 35 },
  { name: 'Ale', age: 30 },
  { name: 'Bob', age: 25 },
];

// Sort by age
users.sort((a, b) => a.age - b.age);

// Sort by name
users.sort((a, b) => a.name.localeCompare(b.name));

// Multi-level sort
users.sort((a, b) => {
  // First by age
  const ageDiff = a.age - b.age;
  if (ageDiff !== 0) return ageDiff;

  // Then by name
  return a.name.localeCompare(b.name);
});

// Non-mutating sort (ES2023)
const sorted = users.toSorted((a, b) => a.age - b.age);
// Original users array unchanged
```

**8. slice() vs splice()**

```typescript
const numbers = [1, 2, 3, 4, 5];

// slice - returns copy, doesn't mutate
const sliced = numbers.slice(1, 3);
// [2, 3]
console.log(numbers); // [1, 2, 3, 4, 5] - unchanged

// splice - mutates original
const removed = numbers.splice(1, 2);
// [2, 3] - removed elements
console.log(numbers); // [1, 4, 5] - mutated!

// splice to insert
const arr = [1, 2, 5];
arr.splice(2, 0, 3, 4); // At index 2, remove 0, insert 3, 4
console.log(arr); // [1, 2, 3, 4, 5]

// splice to replace
arr.splice(0, 1, 10); // At index 0, remove 1, insert 10
console.log(arr); // [10, 2, 3, 4, 5]
```

**9. includes() and indexOf()**

```typescript
const numbers = [1, 2, 3, 4, 5];

// Check existence
console.log(numbers.includes(3)); // true
console.log(numbers.includes(10)); // false

// Works with NaN
const withNaN = [1, NaN, 3];
console.log(withNaN.includes(NaN)); // true
console.log(withNaN.indexOf(NaN)); // -1 (indexOf can't find NaN)

// indexOf returns index or -1
const index = numbers.indexOf(3); // 2
const notFound = numbers.indexOf(10); // -1

// lastIndexOf
const duplicates = [1, 2, 3, 2, 4];
console.log(duplicates.indexOf(2)); // 1
console.log(duplicates.lastIndexOf(2)); // 3
```

**10. flat() and flatMap()**

```typescript
// Flatten nested arrays
const nested = [1, [2, 3], [4, [5, 6]]];

console.log(nested.flat()); // [1, 2, 3, 4, [5, 6]] - default depth 1
console.log(nested.flat(2)); // [1, 2, 3, 4, 5, 6] - depth 2
console.log(nested.flat(Infinity)); // [1, 2, 3, 4, 5, 6] - all depths

// flatMap - map then flat(1)
const numbers = [1, 2, 3];

const doubled = numbers.flatMap((n) => [n, n * 2]);
// [1, 2, 2, 4, 3, 6]

// Equivalent to:
const doubledManual = numbers.map((n) => [n, n * 2]).flat();

// Practical use: expanding items
const sentences = ['Hello world', 'How are you'];
const words = sentences.flatMap((s) => s.split(' '));
// ["Hello", "world", "How", "are", "you"]
```

**11. at() - Access by Index**

```typescript
const numbers = [1, 2, 3, 4, 5];

// Positive index
console.log(numbers.at(0)); // 1
console.log(numbers[0]); // 1 - same

// Negative index (from end)
console.log(numbers.at(-1)); // 5 (last)
console.log(numbers.at(-2)); // 4 (second to last)

// Old way
console.log(numbers[numbers.length - 1]); // 5

// More readable with at()
const last = numbers.at(-1);
```

**12. join() and reverse()**

```typescript
const words = ['Hello', 'World', 'JavaScript'];

// Join into string
console.log(words.join(' ')); // "Hello World JavaScript"
console.log(words.join('-')); // "Hello-World-JavaScript"
console.log(words.join('')); // "HelloWorldJavaScript"

// reverse() - MUTATES!
const numbers = [1, 2, 3, 4, 5];
numbers.reverse();
console.log(numbers); // [5, 4, 3, 2, 1]

// Non-mutating reverse
const reversed = [...numbers].reverse();
// or
const reversed2 = numbers.toReversed(); // ES2023
```

**13. fill() and copyWithin()**

```typescript
// fill - MUTATES
const arr = [1, 2, 3, 4, 5];
arr.fill(0);
console.log(arr); // [0, 0, 0, 0, 0]

// fill with start and end
const arr2 = [1, 2, 3, 4, 5];
arr2.fill(0, 2, 4);
console.log(arr2); // [1, 2, 0, 0, 5]

// Create array with default value
const defaults = new Array(5).fill(0);
// [0, 0, 0, 0, 0]

// copyWithin - MUTATES
const arr3 = [1, 2, 3, 4, 5];
arr3.copyWithin(0, 3); // Copy from index 3 to index 0
console.log(arr3); // [4, 5, 3, 4, 5]
```

**14. Method Chaining**

```typescript
interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
  inStock: boolean;
}

const products: Product[] = [
  {
    id: 1,
    name: 'Laptop',
    price: 1200,
    category: 'electronics',
    inStock: true,
  },
  { id: 2, name: 'Mouse', price: 25, category: 'electronics', inStock: true },
  {
    id: 3,
    name: 'Keyboard',
    price: 75,
    category: 'electronics',
    inStock: false,
  },
  { id: 4, name: 'Desk', price: 300, category: 'furniture', inStock: true },
];

// Chain multiple operations
const result = products
  .filter((p) => p.inStock) // Only in stock
  .filter((p) => p.category === 'electronics') // Electronics only
  .map((p) => ({
    // Transform
    name: p.name,
    discount: p.price * 0.1,
  }))
  .sort((a, b) => b.discount - a.discount); // Sort by discount desc

console.log(result);

// Performance note: each method creates new array
// For large arrays, consider reduce for single pass
const resultOptimized = products
  .reduce((acc, product) => {
    if (product.inStock && product.category === 'electronics') {
      acc.push({
        name: product.name,
        discount: product.price * 0.1,
      });
    }
    return acc;
  }, [] as Array<{ name: string; discount: number }>)
  .sort((a, b) => b.discount - a.discount);
```

### Common Pitfalls

1. **Mutating methods**

```typescript
// These mutate: sort, reverse, splice, fill, copyWithin, push, pop, shift, unshift
const arr = [3, 1, 2];
arr.sort(); // Mutates arr!

// Clone first if needed
const sorted = [...arr].sort();
```

2. **Forgetting return in callbacks**

```typescript
// Bad - returns undefined for each element
const doubled = numbers.map((num) => {
  num * 2; // No return!
});

// Good
const doubled = numbers.map((num) => num * 2);
```

3. **Default string sort**

```typescript
const numbers = [10, 5, 40, 25];
numbers.sort(); // [10, 25, 40, 5] - wrong!

// Always provide comparator for numbers
numbers.sort((a, b) => a - b); // [5, 10, 25, 40]
```

### Best Practices

1. **Prefer immutable methods** (map, filter, reduce)
2. **Clone before using mutating methods**
3. **Use method chaining** for readability
4. **Consider performance** for large arrays (reduce over multiple passes)
5. **Use early returns** in callbacks when possible

```typescript
// Good pattern: data transformation pipeline
const processUsers = (users: User[]) =>
  users
    .filter((u) => u.active)
    .map((u) => ({ ...u, fullName: `${u.firstName} ${u.lastName}` }))
    .sort((a, b) => a.fullName.localeCompare(b.fullName));
```

---

## Object Manipulation

### Simple Explanation

Objects in JavaScript are like filing cabinets - they store related information together. Object manipulation is about efficiently accessing, modifying, and transforming these cabinets and their contents.

Think of object methods as tools in a workshop:

- **Object.keys()**: Get labels of all drawers
- **Object.values()**: Get contents of all drawers
- **Object.entries()**: Get both labels and contents
- **Object.assign()**: Merge filing cabinets
- **Object.freeze()**: Lock the cabinet

### Technical Explanation

JavaScript provides static methods on the Object constructor for working with objects. These methods enable:

- Property enumeration
- Object composition
- Immutability control
- Property descriptor manipulation
- Prototype operations

### Merged Explanation

**1. Object.keys(), values(), entries()**

```typescript
const user = {
  name: 'Ale',
  age: 30,
  email: 'ale@example.com',
};

// Get all keys
const keys = Object.keys(user);
// ["name", "age", "email"]

// Get all values
const values = Object.values(user);
// ["Ale", 30, "ale@example.com"]

// Get key-value pairs
const entries = Object.entries(user);
// [["name", "Ale"], ["age", 30], ["email", "ale@example.com"]]

// Iterate over object
Object.entries(user).forEach(([key, value]) => {
  console.log(`${key}: ${value}`);
});

// Transform object
const upperUser = Object.fromEntries(
  Object.entries(user).map(([key, value]) => [
    key,
    typeof value === 'string' ? value.toUpperCase() : value,
  ]),
);
// { name: "ALE", age: 30, email: "ALE@EXAMPLE.COM" }
```

**2. Object.assign() - Shallow Copy/Merge**

```typescript
const target = { a: 1, b: 2 };
const source1 = { b: 3, c: 4 };
const source2 = { c: 5, d: 6 };

// Merge objects (mutates target)
Object.assign(target, source1, source2);
console.log(target);
// { a: 1, b: 3, c: 5, d: 6 }

// Clone object (empty target)
const user = { name: 'Ale', age: 30 };
const clone = Object.assign({}, user);

// Modern alternative: spread operator
const cloneBetter = { ...user };

// Merge with spread
const merged = { ...source1, ...source2 };

// Note: shallow copy only!
const original = {
  name: 'Ale',
  address: { city: 'Paris' },
};

const copy = { ...original };
copy.address.city = 'London';
console.log(original.address.city); // "London" - mutated!

// Deep clone (simple objects only)
const deepClone = JSON.parse(JSON.stringify(original));

// Or use structuredClone (modern)
const deepClone2 = structuredClone(original);
```

**3. Object.freeze(), seal(), preventExtensions()**

```typescript
const user = { name: 'Ale', age: 30 };

// freeze - cannot add, delete, or modify
Object.freeze(user);
user.name = 'Bob'; // Fails silently
user.email = 'ale@example.com'; // Fails silently
delete user.age; // Fails silently
console.log(user); // { name: "Ale", age: 30 }

// Check if frozen
console.log(Object.isFrozen(user)); // true

// seal - cannot add or delete, but can modify
const sealed = { name: 'Ale', age: 30 };
Object.seal(sealed);
sealed.name = 'Bob'; // OK
sealed.email = 'ale@example.com'; // Fails
delete sealed.age; // Fails
console.log(sealed); // { name: "Bob", age: 30 }

// preventExtensions - cannot add, but can modify and delete
const limited = { name: 'Ale', age: 30 };
Object.preventExtensions(limited);
limited.name = 'Bob'; // OK
delete limited.age; // OK
limited.email = 'ale@example.com'; // Fails
console.log(limited); // { name: "Bob" }

// Note: all are shallow
const nested = { data: { value: 1 } };
Object.freeze(nested);
nested.data.value = 2; // Works! Nested object not frozen
console.log(nested.data.value); // 2

// Deep freeze function
function deepFreeze<T extends object>(obj: T): T {
  Object.freeze(obj);
  Object.values(obj).forEach((value) => {
    if (typeof value === 'object' && value !== null) {
      deepFreeze(value);
    }
  });
  return obj;
}
```

**4. Object.create() - Prototype Inheritance**

```typescript
const personPrototype = {
  greet() {
    return `Hello, I'm ${this.name}`;
  },
};

// Create object with specific prototype
const ale = Object.create(personPrototype);
ale.name = 'Ale';
ale.age = 30;

console.log(ale.greet()); // "Hello, I'm Ale"
console.log(Object.getPrototypeOf(ale) === personPrototype); // true

// Create with properties
const bob = Object.create(personPrototype, {
  name: { value: 'Bob', writable: true, enumerable: true },
  age: { value: 25, writable: true, enumerable: true },
});

// Create plain object (no prototype)
const plain = Object.create(null);
plain.name = 'test';
// console.log(plain.toString()); // Error! No inherited methods
```

**5. Object.defineProperty() - Property Descriptors**

```typescript
const user = {};

// Define property with descriptor
Object.defineProperty(user, 'name', {
  value: 'Ale',
  writable: false, // Cannot change value
  enumerable: true, // Shows in for...in and Object.keys()
  configurable: false, // Cannot delete or change descriptor
});

// user.name = "Bob"; // Fails
// delete user.name; // Fails

// Define multiple properties
Object.defineProperties(user, {
  age: {
    value: 30,
    writable: true,
    enumerable: true,
    configurable: true,
  },
  email: {
    value: 'ale@example.com',
    writable: true,
    enumerable: false, // Hidden from enumeration
    configurable: true,
  },
});

console.log(Object.keys(user)); // ["name", "age"] - email is hidden

// Getters and setters
const person = {
  firstName: 'Ale',
  lastName: 'Smith',
};

Object.defineProperty(person, 'fullName', {
  get() {
    return `${this.firstName} ${this.lastName}`;
  },
  set(value: string) {
    [this.firstName, this.lastName] = value.split(' ');
  },
  enumerable: true,
  configurable: true,
});

console.log(person.fullName); // "Ale Smith"
person.fullName = 'Bob Johnson';
console.log(person.firstName); // "Bob"

// Get property descriptor
const descriptor = Object.getOwnPropertyDescriptor(user, 'name');
console.log(descriptor);
// {
//   value: "Ale",
//   writable: false,
//   enumerable: true,
//   configurable: false
// }
```

**6. Object.hasOwn() and hasOwnProperty()**

```typescript
const user = { name: 'Ale', age: 30 };

// Check own property (not inherited)
console.log(Object.hasOwn(user, 'name')); // true
console.log(Object.hasOwn(user, 'toString')); // false (inherited)

// Old way (less safe)
console.log(user.hasOwnProperty('name')); // true

// Why Object.hasOwn is better:
const obj = Object.create(null);
obj.name = 'test';
// obj.hasOwnProperty("name"); // Error! No hasOwnProperty on null prototype
console.log(Object.hasOwn(obj, 'name')); // true - works!
```

**7. Object Comparison and Equality**

```typescript
// Objects are compared by reference, not value
const obj1 = { name: 'Ale' };
const obj2 = { name: 'Ale' };
const obj3 = obj1;

console.log(obj1 === obj2); // false - different references
console.log(obj1 === obj3); // true - same reference

// Deep equality check
function deepEqual(obj1: any, obj2: any): boolean {
  if (obj1 === obj2) return true;

  if (
    typeof obj1 !== 'object' ||
    typeof obj2 !== 'object' ||
    obj1 === null ||
    obj2 === null
  ) {
    return false;
  }

  const keys1 = Object.keys(obj1);
  const keys2 = Object.keys(obj2);

  if (keys1.length !== keys2.length) return false;

  return keys1.every((key) => deepEqual(obj1[key], obj2[key]));
}

console.log(deepEqual(obj1, obj2)); // true - same structure and values
```

**8. Property Iteration**

```typescript
const parent = { inherited: 'value' };
const child = Object.create(parent);
child.own = 'property';

// for...in - includes inherited
for (const key in child) {
  console.log(key); // "own", "inherited"
}

// Only own properties
for (const key in child) {
  if (Object.hasOwn(child, key)) {
    console.log(key); // "own"
  }
}

// Object.keys - only own and enumerable
console.log(Object.keys(child)); // ["own"]

// getOwnPropertyNames - own including non-enumerable
const obj = {};
Object.defineProperty(obj, 'hidden', {
  value: 'secret',
  enumerable: false,
});

console.log(Object.keys(obj)); // []
console.log(Object.getOwnPropertyNames(obj)); // ["hidden"]

// getOwnPropertySymbols
const sym = Symbol('id');
obj[sym] = 123;
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(id)]
```

**9. Optional Chaining and Nullish Coalescing**

```typescript
interface User {
  name: string;
  address?: {
    street?: string;
    city?: string;
  };
  getEmail?: () => string;
}

const user: User = { name: 'Ale' };

// Safe property access
const city = user.address?.city; // undefined (no error)
const street = user.address?.street ?? 'Unknown'; // "Unknown"

// Safe method call
const email = user.getEmail?.(); // undefined (no error)

// Safe array access
const users: User[] | undefined = undefined;
const firstCity = users?.[0]?.address?.city;

// Short-circuit evaluation
const value = user.address?.city ?? getDefaultCity(); // Only calls if null/undefined
```

**10. Object Destructuring and Rest**

```typescript
const user = {
  id: 1,
  name: 'Ale',
  age: 30,
  email: 'ale@example.com',
  address: {
    city: 'Paris',
    country: 'France',
  },
};

// Basic destructuring
const { name, age } = user;

// Rename
const { name: userName, age: userAge } = user;

// Default values
const { phone = 'N/A' } = user;

// Nested
const {
  address: { city },
} = user;

// Rest operator
const { id, ...rest } = user;
// rest = { name, age, email, address }

// In function parameters
function displayUser({ name, age }: { name: string; age: number }) {
  console.log(`${name} is ${age}`);
}

// Dynamic property names
const key = 'name';
const { [key]: value } = user;
// value = "Ale"
```

### Common Pitfalls

1. **Shallow copying nested objects**

```typescript
const original = { data: { value: 1 } };
const copy = { ...original };
copy.data.value = 2;
console.log(original.data.value); // 2 - mutated!
```

2. **Freeze only freezes first level**

```typescript
const obj = { nested: { value: 1 } };
Object.freeze(obj);
obj.nested.value = 2; // Works!
```

3. **Modifying during iteration**

```typescript
// Bad
const obj = { a: 1, b: 2, c: 3 };
for (const key in obj) {
  delete obj[key]; // Unpredictable behavior!
}

// Good
const keys = Object.keys(obj);
keys.forEach((key) => delete obj[key]);
```

### Best Practices

1. **Use spread for shallow clones**
2. **Use structuredClone for deep clones**
3. **Prefer Object.hasOwn over hasOwnProperty**
4. **Use optional chaining for safe access**
5. **Freeze objects when immutability needed**

---

## Error Handling

### Simple Explanation

Error handling is like having safety nets and backup plans. When something goes wrong (and it will), you want to:

1. Catch the problem before it crashes everything
2. Understand what went wrong
3. Recover gracefully or inform the user properly

Think of try-catch like insurance:

- **try**: Normal operations
- **catch**: What to do if something goes wrong
- **finally**: Cleanup that happens regardless (like closing doors)

### Technical Explanation

JavaScript provides several mechanisms for error handling:

- try-catch-finally blocks
- throw statements
- Error objects and custom errors
- Async error handling with promises
- Global error handlers

### Merged Explanation

**1. Basic Try-Catch-Finally**

```typescript
try {
  // Code that might throw an error
  const data = JSON.parse('invalid json');
} catch (error) {
  // Handle the error
  console.error('Parsing failed:', error);
} finally {
  // Always runs, even if there's a return statement
  console.log('Cleanup');
}

// Catching specific errors
try {
  const user = null;
  console.log(user.name);
} catch (error) {
  if (error instanceof TypeError) {
    console.error('Type error:', error.message);
  } else if (error instanceof ReferenceError) {
    console.error('Reference error:', error.message);
  } else {
    console.error('Unknown error:', error);
  }
}

// Re-throwing errors
try {
  processData();
} catch (error) {
  console.error('Failed to process:', error);
  throw error; // Re-throw to caller
}
```

**2. Error Types**

```typescript
// Built-in error types:
// - Error: Generic error
// - TypeError: Wrong type
// - ReferenceError: Invalid reference
// - SyntaxError: Syntax error
// - RangeError: Value out of range
// - URIError: Invalid URI handling

// Creating errors
const error1 = new Error('Something went wrong');
const error2 = new TypeError('Expected number, got string');

// Error properties
console.log(error1.name); // "Error"
console.log(error1.message); // "Something went wrong"
console.log(error1.stack); // Stack trace
```

**3. Custom Errors**

```typescript
// Custom error class
class ValidationError extends Error {
  constructor(message: string, public field?: string) {
    super(message);
    this.name = 'ValidationError';

    // Maintains proper stack trace (V8 only)
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, ValidationError);
    }
  }
}

class NetworkError extends Error {
  constructor(
    message: string,
    public statusCode: number,
    public endpoint: string,
  ) {
    super(message);
    this.name = 'NetworkError';
  }
}

// Usage
function validateUser(user: any) {
  if (!user.email) {
    throw new ValidationError('Email is required', 'email');
  }
  if (!user.email.includes('@')) {
    throw new ValidationError('Invalid email format', 'email');
  }
}

try {
  validateUser({ name: 'Ale' });
} catch (error) {
  if (error instanceof ValidationError) {
    console.error(`Validation error in ${error.field}: ${error.message}`);
  } else {
    console.error('Unknown error:', error);
  }
}

// API error class
class ApiError extends Error {
  constructor(
    message: string,
    public status: number,
    public code?: string,
    public details?: any,
  ) {
    super(message);
    this.name = 'ApiError';
  }

  static notFound(resource: string) {
    return new ApiError(`${resource} not found`, 404, 'NOT_FOUND');
  }

  static unauthorized(message = 'Unauthorized') {
    return new ApiError(message, 401, 'UNAUTHORIZED');
  }

  static badRequest(message: string, details?: any) {
    return new ApiError(message, 400, 'BAD_REQUEST', details);
  }
}

// Usage
throw ApiError.notFound('User');
throw ApiError.badRequest('Invalid input', { field: 'email' });
```

**4. Async Error Handling**

```typescript
// Promises with catch
fetchUser(1)
  .then((user) => console.log(user))
  .catch((error) => console.error('Failed:', error))
  .finally(() => console.log('Done'));

// Async/await with try-catch
async function loadUser(id: number) {
  try {
    const user = await fetchUser(id);
    const posts = await fetchPosts(user.id);
    return { user, posts };
  } catch (error) {
    console.error('Failed to load user:', error);
    throw error; // Re-throw if needed
  } finally {
    console.log('Cleanup');
  }
}

// Multiple try-catch blocks
async function processUser(id: number) {
  let user;

  try {
    user = await fetchUser(id);
  } catch (error) {
    console.error('User fetch failed:', error);
    return null; // Early return
  }

  try {
    const posts = await fetchPosts(user.id);
    return { user, posts };
  } catch (error) {
    console.error('Posts fetch failed:', error);
    return { user, posts: [] }; // Return partial data
  }
}

// Handling multiple promises
async function fetchAllData() {
  try {
    // If any fails, catch block runs
    const [users, posts, comments] = await Promise.all([
      fetchUsers(),
      fetchPosts(),
      fetchComments(),
    ]);
    return { users, posts, comments };
  } catch (error) {
    console.error('Failed to fetch all data:', error);
    throw error;
  }
}

// Handling with allSettled (doesn't throw)
async function fetchAllDataSafe() {
  const results = await Promise.allSettled([
    fetchUsers(),
    fetchPosts(),
    fetchComments(),
  ]);

  const users = results[0].status === 'fulfilled' ? results[0].value : [];
  const posts = results[1].status === 'fulfilled' ? results[1].value : [];
  const comments = results[2].status === 'fulfilled' ? results[2].value : [];

  return { users, posts, comments };
}
```

**5. Error Boundaries (React)**

```typescript
class ErrorBoundary extends React.Component<
  { children: React.ReactNode },
  { hasError: boolean; error?: Error }
> {
  constructor(props: any) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error boundary caught:', error, errorInfo);
    // Log to error reporting service
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h1>Something went wrong</h1>
          <p>{this.state.error?.message}</p>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>;
```

**6. Error Recovery Patterns**

```typescript
// Retry with exponential backoff
async function fetchWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  baseDelay: number = 1000,
): Promise<T> {
  let lastError: Error;

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;

      if (attempt < maxRetries) {
        const delay = baseDelay * Math.pow(2, attempt);
        console.log(`Retry ${attempt + 1}/${maxRetries} after ${delay}ms`);
        await new Promise((resolve) => setTimeout(resolve, delay));
      }
    }
  }

  throw lastError!;
}

// Usage
const user = await fetchWithRetry(() => fetchUser(1));

// Fallback pattern
async function getUserWithFallback(id: number) {
  try {
    return await fetchUserFromPrimary(id);
  } catch (error) {
    console.warn('Primary source failed, trying backup');
    try {
      return await fetchUserFromBackup(id);
    } catch (backupError) {
      console.warn('Backup also failed, using cache');
      return getCachedUser(id);
    }
  }
}

// Circuit breaker pattern
class CircuitBreaker {
  private failures: number = 0;
  private lastFailureTime?: number;
  private state: 'closed' | 'open' | 'half-open' = 'closed';

  constructor(private threshold: number = 5, private timeout: number = 60000) {}

  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'open') {
      if (Date.now() - this.lastFailureTime! > this.timeout) {
        this.state = 'half-open';
      } else {
        throw new Error('Circuit breaker is open');
      }
    }

    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  private onSuccess() {
    this.failures = 0;
    this.state = 'closed';
  }

  private onFailure() {
    this.failures++;
    this.lastFailureTime = Date.now();

    if (this.failures >= this.threshold) {
      this.state = 'open';
    }
  }
}

// Usage
const breaker = new CircuitBreaker();
const data = await breaker.execute(() => fetchData());
```

**7. Error Logging and Monitoring**

```typescript
// Structured error logging
interface ErrorLog {
  timestamp: Date;
  level: 'error' | 'warn' | 'info';
  message: string;
  stack?: string;
  context?: any;
}

class ErrorLogger {
  private logs: ErrorLog[] = [];

  log(error: Error, context?: any) {
    const log: ErrorLog = {
      timestamp: new Date(),
      level: 'error',
      message: error.message,
      stack: error.stack,
      context,
    };

    this.logs.push(log);

    // Send to monitoring service
    this.sendToMonitoring(log);

    // Console log in development
    if (process.env.NODE_ENV === 'development') {
      console.error(log);
    }
  }

  private sendToMonitoring(log: ErrorLog) {
    // Send to service like Sentry, LogRocket, etc.
  }
}

// Global error handler (browser)
window.addEventListener('error', (event) => {
  console.error('Global error:', event.error);
  errorLogger.log(event.error, {
    filename: event.filename,
    lineno: event.lineno,
    colno: event.colno,
  });
});

// Unhandled promise rejection (browser)
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled rejection:', event.reason);
  errorLogger.log(
    event.reason instanceof Error
      ? event.reason
      : new Error(String(event.reason)),
    { type: 'unhandledRejection' },
  );
});

// Node.js global handlers
process.on('uncaughtException', (error) => {
  console.error('Uncaught exception:', error);
  errorLogger.log(error);
  process.exit(1);
});

process.on('unhandledRejection', (reason) => {
  console.error('Unhandled rejection:', reason);
  errorLogger.log(reason instanceof Error ? reason : new Error(String(reason)));
});
```

### Common Pitfalls

1. **Empty catch blocks**

```typescript
// Bad - silently swallows errors
try {
  riskyOperation();
} catch (error) {
  // Nothing
}

// Good - at least log it
try {
  riskyOperation();
} catch (error) {
  console.error('Operation failed:', error);
}
```

2. **Not handling async errors**

```typescript
// Bad - unhandled promise rejection
async function badFunction() {
  throw new Error('Oops');
}
badFunction(); // Unhandled!

// Good
badFunction().catch((error) => console.error(error));

// Or with await
try {
  await badFunction();
} catch (error) {
  console.error(error);
}
```

3. **Throwing non-Error objects**

```typescript
// Bad
throw 'Something went wrong'; // String
throw { message: 'Error' }; // Plain object

// Good
throw new Error('Something went wrong');
```

### Best Practices

1. **Always use Error objects** (proper stack traces)
2. **Create custom error classes** for different error types
3. **Handle async errors** with try-catch or .catch()
4. **Log errors with context** (user ID, action, etc.)
5. **Fail fast** (throw early, handle at boundaries)
6. **Clean up in finally** blocks
7. **Don't catch errors you can't handle** (let them propagate)

```typescript
// Good error handling pattern
class UserService {
  async getUser(id: number): Promise<User> {
    try {
      const user = await this.repository.findById(id);

      if (!user) {
        throw ApiError.notFound('User');
      }

      return user;
    } catch (error) {
      // Log error with context
      logger.error('Failed to get user', {
        userId: id,
        error: error instanceof Error ? error.message : String(error),
        stack: error instanceof Error ? error.stack : undefined,
      });

      // Transform database errors to API errors
      if (error instanceof DatabaseError) {
        throw new ApiError('Failed to fetch user', 500, 'DATABASE_ERROR');
      }

      // Re-throw if already API error
      if (error instanceof ApiError) {
        throw error;
      }

      // Wrap unknown errors
      throw new ApiError('An unexpected error occurred', 500, 'INTERNAL_ERROR');
    }
  }
}
```

---

## Modules

### Simple Explanation

Modules are like LEGO building blocks - each module is a self-contained piece that:

- Keeps its code private unless explicitly shared
- Can use code from other modules
- Makes code organized and reusable

Think of modules like rooms in a house:

- Each room (module) has a specific purpose
- Rooms have doors (exports) to access specific things
- You can enter through doors (imports) to use what's inside
- Private things stay inside the room

### Technical Explanation

JavaScript modules (ES6 modules) provide:

- File-based scope (variables are private by default)
- Explicit import/export syntax
- Static structure (analyzed before execution)
- Automatic strict mode
- Singleton pattern (module loaded once, cached)

### Merged Explanation

**1. Export Syntax**

```typescript
// Named exports (multiple per module)
export const API_KEY = 'abc123';
export const API_URL = 'https://api.example.com';

export function fetchUser(id: number) {
  return fetch(`${API_URL}/users/${id}`);
}

export class UserService {
  getUser(id: number) {
    return fetchUser(id);
  }
}

// Export separately
const CONFIG = { timeout: 5000 };
function helper() {
  /* ... */
}
export { CONFIG, helper };

// Export with rename
export { CONFIG as config, helper as utilityHelper };

// Default export (one per module)
export default class User {
  constructor(public name: string) {}
}

// Or
class User {
  constructor(public name: string) {}
}
export default User;

// Named + default in same file
export const VERSION = '1.0.0';
export default function main() {
  console.log('Main function');
}
```

**2. Import Syntax**

```typescript
// Named imports
import { fetchUser, UserService } from './api.js';

// Import with rename
import { fetchUser as getUser } from './api.js';

// Import all as namespace
import * as API from './api.js';
API.fetchUser(1);

// Default import (can use any name)
import User from './User.js';
import MyUser from './User.js'; // Same thing

// Mixed import
import User, { VERSION, fetchUser } from './api.js';

// Import for side effects only (runs module code)
import './polyfills.js';

// Dynamic import (returns Promise)
const module = await import('./heavy-module.js');
module.doSomething();

// Or with then
import('./module.js').then((module) => {
  module.doSomething();
});
```

**3. Module Patterns**

```typescript
// user.ts - Model
export interface User {
  id: number;
  name: string;
  email: string;
}

export function createUser(data: Partial<User>): User {
  return {
    id: Date.now(),
    name: data.name || '',
    email: data.email || '',
  };
}

// userService.ts - Service layer
import { User, createUser } from './user.js';

export class UserService {
  private users: User[] = [];

  addUser(data: Partial<User>): User {
    const user = createUser(data);
    this.users.push(user);
    return user;
  }

  getUser(id: number): User | undefined {
    return this.users.find((u) => u.id === id);
  }

  getAllUsers(): User[] {
    return [...this.users];
  }
}

// Singleton pattern
let instance: UserService | null = null;

export function getUserService(): UserService {
  if (!instance) {
    instance = new UserService();
  }
  return instance;
}

// main.ts - Main application
import { getUserService } from './userService.js';

const service = getUserService();
service.addUser({ name: 'Ale', email: 'ale@example.com' });
```

**4. Re-exporting (Barrel Pattern)**

```typescript
// models/index.ts - Barrel file
export { User, createUser } from './user.js';
export { Post, createPost } from './post.js';
export { Comment, createComment } from './comment.js';

// Now import from single file
import { User, Post, Comment } from './models/index.js';

// Re-export with rename
export { User as UserModel } from './user.js';

// Re-export all
export * from './user.js';
export * from './post.js';

// Re-export default
export { default as User } from './user.js';
```

**5. Dynamic Imports (Code Splitting)**

```typescript
// Lazy load heavy module
async function loadChart() {
  const { Chart } = await import('./chart.js');
  return new Chart();
}

// Conditional loading
async function loadFeature(featureName: string) {
  if (featureName === 'analytics') {
    const analytics = await import('./analytics.js');
    analytics.init();
  } else if (featureName === 'reporting') {
    const reporting = await import('./reporting.js');
    reporting.init();
  }
}

// React lazy loading
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}

// Webpack magic comments
const module = await import(
  /* webpackChunkName: "my-chunk" */
  /* webpackPrefetch: true */
  './module.js'
);
```

**6. Module Resolution**

```typescript
// Relative imports
import { helper } from './utils.js'; // Same directory
import { User } from '../models/user.js'; // Parent directory
import { API } from '../../services/api.js'; // Two levels up

// Absolute imports (with baseUrl in tsconfig.json)
import { helper } from 'utils/helper.js';
import { User } from 'models/user.js';

// Package imports (node_modules)
import React from 'react';
import { format } from 'date-fns';

// Type-only imports (TypeScript)
import type { User } from './user.js';
```

**7. CommonJS vs ES Modules**

```typescript
// CommonJS (Node.js traditional)
// Synchronous, dynamic
const fs = require('fs');
const helper = require('./helper');

module.exports = {
  doSomething() {},
};

// Or
module.exports.doSomething = function () {};

// ES Modules
// Asynchronous, static
import fs from 'fs';
import { helper } from './helper.js';

export function doSomething() {}

// Interop: Using CommonJS in ESM
import pkg from 'commonjs-package'; // Default export
const { method } = pkg;

// ESM in CommonJS (dynamic import)
(async () => {
  const { helper } = await import('./esm-module.mjs');
})();
```

**8. Module Scope and Caching**

```typescript
// counter.ts
let count = 0;

export function increment() {
  count++;
  console.log(count);
}

export function getCount() {
  return count;
}

// main.ts
import { increment, getCount } from './counter.js';
import { increment as inc2 } from './counter.js';

increment(); // 1
inc2(); // 2 - Same instance!
console.log(getCount()); // 2

// Module runs once, shared state
```

**9. Circular Dependencies**

```typescript
// a.ts
import { b } from './b.js';

export const a = 'A';

export function showB() {
  console.log(b); // This works
}

// b.ts
import { a } from './a.js';

export const b = 'B';

console.log(a); // undefined! (a not yet initialized)

// Solution: Use functions to defer access
// a.ts
import { getB } from './b.js';

export const a = 'A';

export function showB() {
  console.log(getB()); // Now works
}

// b.ts
import { a } from './a.js';

export const b = 'B';

export function getA() {
  return a; // Accessed when function is called
}
```

**10. Practical Module Organization**

```typescript
// Feature-based structure
src / features / auth / auth.service.ts;
auth.types.ts;
auth.utils.ts;
index.ts; // Barrel export
users / users.service.ts;
users.types.ts;
index.ts;
shared / utils / date.utils.ts;
string.utils.ts;
index.ts;
types / common.types.ts;
constants / api.constants.ts;
main.ts;

// features/auth/index.ts
export * from './auth.service.js';
export * from './auth.types.js';
export { hashPassword, verifyPassword } from './auth.utils.js';

// main.ts
import { AuthService } from './features/auth/index.js';
import { UserService } from './features/users/index.js';
```

### Common Pitfalls

1. **Forgetting file extensions**

```typescript
// In Node.js ESM, extensions are required!
import { helper } from './utils'; // Error!
import { helper } from './utils.js'; // OK
```

2. **Circular dependencies**

```typescript
// Can cause undefined exports
// Use functions to defer access or restructure
```

3. **Mixing default and named exports**

```typescript
// Confusing for consumers
export default function () {}
export const OTHER = 'value';

// Better: be consistent
export function main() {}
export const OTHER = 'value';
```

### Best Practices

1. **Use named exports** (better for tree-shaking and refactoring)
2. **One module, one purpose** (Single Responsibility)
3. **Avoid circular dependencies** (indicates design issue)
4. **Use barrel files** for convenient imports
5. **Lazy load heavy modules** (code splitting)
6. **Keep modules focused and small**

---

## Design Patterns

### Simple Explanation

Design patterns are like recipes for solving common programming problems. Instead of reinventing solutions, you use proven patterns that other developers recognize.

Think of patterns like furniture assembly instructions:

- **Singleton**: Only one instance (one king in a kingdom)
- **Factory**: Manufacturing objects (car factory makes different models)
- **Observer**: Newsletter subscription (notify when new content)
- **Module**: Organized toolbox (tools grouped by type)

### Technical Explanation

Design patterns are reusable solutions to common software design problems. They provide:

- Proven architecture
- Common vocabulary for developers
- Flexible and maintainable code
- Best practices encapsulation

### Merged Explanation

**1. Singleton Pattern**

```typescript
// Ensures only one instance exists

class Database {
  private static instance: Database;
  private connection: any;

  private constructor() {
    // Private constructor prevents direct instantiation
    this.connection = this.connect();
  }

  static getInstance(): Database {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }

  private connect() {
    console.log('Connecting to database...');
    return {
      /* connection object */
    };
  }

  query(sql: string) {
    console.log('Executing:', sql);
    return this.connection;
  }
}

// Usage
const db1 = Database.getInstance();
const db2 = Database.getInstance();
console.log(db1 === db2); // true - same instance

// Modern approach: ES6 module singleton
// database.ts
const connection = createConnection();

export function query(sql: string) {
  return connection.execute(sql);
}

// Automatically singleton (module loaded once)
```

**2. Factory Pattern**

```typescript
// Creates objects without specifying exact class

interface Vehicle {
  type: string;
  wheels: number;
  drive(): void;
}

class Car implements Vehicle {
  type = 'car';
  wheels = 4;
  drive() {
    console.log('Driving a car');
  }
}

class Motorcycle implements Vehicle {
  type = 'motorcycle';
  wheels = 2;
  drive() {
    console.log('Riding a motorcycle');
  }
}

class Truck implements Vehicle {
  type = 'truck';
  wheels = 6;
  drive() {
    console.log('Driving a truck');
  }
}

// Factory
class VehicleFactory {
  createVehicle(type: string): Vehicle {
    switch (type) {
      case 'car':
        return new Car();
      case 'motorcycle':
        return new Motorcycle();
      case 'truck':
        return new Truck();
      default:
        throw new Error(`Unknown vehicle type: ${type}`);
    }
  }
}

// Usage
const factory = new VehicleFactory();
const car = factory.createVehicle('car');
const motorcycle = factory.createVehicle('motorcycle');

car.drive(); // "Driving a car"
motorcycle.drive(); // "Riding a motorcycle"

// Functional factory
type VehicleType = 'car' | 'motorcycle' | 'truck';

function createVehicle(type: VehicleType): Vehicle {
  const vehicles = {
    car: () => new Car(),
    motorcycle: () => new Motorcycle(),
    truck: () => new Truck(),
  };

  return vehicles[type]();
}
```

**3. Observer Pattern (Pub/Sub)**

```typescript
// Objects subscribe to events and get notified

type Listener<T> = (data: T) => void;

class EventEmitter<T = any> {
  private listeners: Map<string, Listener<T>[]> = new Map();

  on(event: string, listener: Listener<T>): () => void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event)!.push(listener);

    // Return unsubscribe function
    return () => this.off(event, listener);
  }

  off(event: string, listener: Listener<T>) {
    const eventListeners = this.listeners.get(event);
    if (eventListeners) {
      const index = eventListeners.indexOf(listener);
      if (index > -1) {
        eventListeners.splice(index, 1);
      }
    }
  }

  emit(event: string, data: T) {
    const eventListeners = this.listeners.get(event);
    if (eventListeners) {
      eventListeners.forEach((listener) => listener(data));
    }
  }

  once(event: string, listener: Listener<T>) {
    const onceWrapper: Listener<T> = (data) => {
      listener(data);
      this.off(event, onceWrapper);
    };
    this.on(event, onceWrapper);
  }
}

// Usage
interface UserEvent {
  userId: number;
  action: string;
}

const events = new EventEmitter<UserEvent>();

// Subscribe
const unsubscribe = events.on('user:login', (data) => {
  console.log(`User ${data.userId} logged in`);
});

events.on('user:logout', (data) => {
  console.log(`User ${data.userId} logged out`);
});

// Emit
events.emit('user:login', { userId: 1, action: 'login' });
events.emit('user:logout', { userId: 1, action: 'logout' });

// Unsubscribe
unsubscribe();
```

**4. Module Pattern**

```typescript
// Encapsulates private data and exposes public API

const UserModule = (function () {
  // Private variables and functions
  let users: any[] = [];
  let idCounter = 1;

  function generateId() {
    return idCounter++;
  }

  function validate(user: any) {
    return user.name && user.email;
  }

  // Public API
  return {
    add(user: any) {
      if (!validate(user)) {
        throw new Error('Invalid user');
      }
      const newUser = {
        id: generateId(),
        ...user,
      };
      users.push(newUser);
      return newUser;
    },

    get(id: number) {
      return users.find((u) => u.id === id);
    },

    getAll() {
      return [...users]; // Return copy
    },

    remove(id: number) {
      const index = users.findIndex((u) => u.id === id);
      if (index > -1) {
        users.splice(index, 1);
        return true;
      }
      return false;
    },
  };
})();

// Usage
UserModule.add({ name: 'Ale', email: 'ale@example.com' });
// UserModule.users; // undefined - private!
```

**5. Strategy Pattern**

```typescript
// Defines family of algorithms, makes them interchangeable

interface PaymentStrategy {
  pay(amount: number): void;
}

class CreditCardPayment implements PaymentStrategy {
  constructor(private cardNumber: string) {}

  pay(amount: number) {
    console.log(`Paying $${amount} with credit card ${this.cardNumber}`);
  }
}

class PayPalPayment implements PaymentStrategy {
  constructor(private email: string) {}

  pay(amount: number) {
    console.log(`Paying $${amount} via PayPal (${this.email})`);
  }
}

class CryptoPayment implements PaymentStrategy {
  constructor(private wallet: string) {}

  pay(amount: number) {
    console.log(`Paying $${amount} via crypto to ${this.wallet}`);
  }
}

class ShoppingCart {
  private items: any[] = [];

  addItem(item: any) {
    this.items.push(item);
  }

  checkout(strategy: PaymentStrategy) {
    const total = this.items.reduce((sum, item) => sum + item.price, 0);
    strategy.pay(total);
  }
}

// Usage
const cart = new ShoppingCart();
cart.addItem({ name: 'Book', price: 20 });
cart.addItem({ name: 'Pen', price: 5 });

// Pay with different strategies
cart.checkout(new CreditCardPayment('1234-5678'));
cart.checkout(new PayPalPayment('ale@example.com'));
cart.checkout(new CryptoPayment('0x123...'));
```

**6. Decorator Pattern**

```typescript
// Adds behavior to objects dynamically

interface Coffee {
  cost(): number;
  description(): string;
}

class SimpleCoffee implements Coffee {
  cost() {
    return 5;
  }

  description() {
    return 'Simple coffee';
  }
}

// Decorators
class MilkDecorator implements Coffee {
  constructor(private coffee: Coffee) {}

  cost() {
    return this.coffee.cost() + 2;
  }

  description() {
    return this.coffee.description() + ', milk';
  }
}

class SugarDecorator implements Coffee {
  constructor(private coffee: Coffee) {}

  cost() {
    return this.coffee.cost() + 1;
  }

  description() {
    return this.coffee.description() + ', sugar';
  }
}

class WhipDecorator implements Coffee {
  constructor(private coffee: Coffee) {}

  cost() {
    return this.coffee.cost() + 3;
  }

  description() {
    return this.coffee.description() + ', whip';
  }
}

// Usage
let coffee: Coffee = new SimpleCoffee();
console.log(coffee.description(), '-', coffee.cost()); // "Simple coffee - 5"

coffee = new MilkDecorator(coffee);
console.log(coffee.description(), '-', coffee.cost()); // "Simple coffee, milk - 7"

coffee = new SugarDecorator(coffee);
console.log(coffee.description(), '-', coffee.cost()); // "Simple coffee, milk, sugar - 8"

coffee = new WhipDecorator(coffee);
console.log(coffee.description(), '-', coffee.cost()); // "Simple coffee, milk, sugar, whip - 11"

// TypeScript decorator syntax (experimental)
function log(target: any, key: string, descriptor: PropertyDescriptor) {
  const original = descriptor.value;
  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${key} with args:`, args);
    return original.apply(this, args);
  };
  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number) {
    return a + b;
  }
}
```

**7. Adapter Pattern**

```typescript
// Makes incompatible interfaces work together

// Old API
class OldApiService {
  getUser(userId: number) {
    return {
      user_id: userId,
      user_name: 'Ale',
      user_email: 'ale@example.com',
    };
  }
}

// New interface our app expects
interface User {
  id: number;
  name: string;
  email: string;
}

// Adapter
class UserApiAdapter {
  constructor(private oldApi: OldApiService) {}

  getUser(id: number): User {
    const oldUser = this.oldApi.getUser(id);

    // Adapt old format to new format
    return {
      id: oldUser.user_id,
      name: oldUser.user_name,
      email: oldUser.user_email,
    };
  }
}

// Usage
const oldApi = new OldApiService();
const adapter = new UserApiAdapter(oldApi);
const user = adapter.getUser(1); // Returns User interface
```

**8. Facade Pattern**

```typescript
// Provides simplified interface to complex subsystem

class CPU {
  freeze() {
    console.log('CPU: Freezing');
  }
  jump(position: number) {
    console.log(`CPU: Jumping to ${position}`);
  }
  execute() {
    console.log('CPU: Executing');
  }
}

class Memory {
  load(position: number, data: string) {
    console.log(`Memory: Loading ${data} at ${position}`);
  }
}

class HardDrive {
  read(lba: number, size: number) {
    console.log(`HardDrive: Reading ${size} bytes from ${lba}`);
    return 'boot data';
  }
}

// Facade
class ComputerFacade {
  private cpu: CPU;
  private memory: Memory;
  private hardDrive: HardDrive;

  constructor() {
    this.cpu = new CPU();
    this.memory = new Memory();
    this.hardDrive = new HardDrive();
  }

  start() {
    console.log('Starting computer...');
    this.cpu.freeze();
    this.memory.load(0, this.hardDrive.read(0, 1024));
    this.cpu.jump(0);
    this.cpu.execute();
    console.log('Computer started!');
  }
}

// Usage - Simple interface to complex process
const computer = new ComputerFacade();
computer.start(); // Single method hides complexity
```

**9. Command Pattern**

```typescript
// Encapsulates requests as objects

interface Command {
  execute(): void;
  undo(): void;
}

class Light {
  on() {
    console.log('Light is ON');
  }
  off() {
    console.log('Light is OFF');
  }
}

class LightOnCommand implements Command {
  constructor(private light: Light) {}

  execute() {
    this.light.on();
  }

  undo() {
    this.light.off();
  }
}

class LightOffCommand implements Command {
  constructor(private light: Light) {}

  execute() {
    this.light.off();
  }

  undo() {
    this.light.on();
  }
}

// Invoker
class RemoteControl {
  private history: Command[] = [];

  execute(command: Command) {
    command.execute();
    this.history.push(command);
  }

  undo() {
    const command = this.history.pop();
    if (command) {
      command.undo();
    }
  }
}

// Usage
const light = new Light();
const remote = new RemoteControl();

remote.execute(new LightOnCommand(light)); // Light is ON
remote.execute(new LightOffCommand(light)); // Light is OFF
remote.undo(); // Light is ON (undo last command)
```

**10. Builder Pattern**

```typescript
// Constructs complex objects step by step

class User {
  constructor(
    public name: string,
    public email: string,
    public age?: number,
    public address?: string,
    public phone?: string,
  ) {}
}

class UserBuilder {
  private name!: string;
  private email!: string;
  private age?: number;
  private address?: string;
  private phone?: string;

  setName(name: string): this {
    this.name = name;
    return this;
  }

  setEmail(email: string): this {
    this.email = email;
    return this;
  }

  setAge(age: number): this {
    this.age = age;
    return this;
  }

  setAddress(address: string): this {
    this.address = address;
    return this;
  }

  setPhone(phone: string): this {
    this.phone = phone;
    return this;
  }

  build(): User {
    if (!this.name || !this.email) {
      throw new Error('Name and email are required');
    }
    return new User(this.name, this.email, this.age, this.address, this.phone);
  }
}

// Usage
const user = new UserBuilder()
  .setName('Ale')
  .setEmail('ale@example.com')
  .setAge(30)
  .setPhone('+33123456789')
  .build();
```

### Common Pitfalls

1. **Overusing patterns** (adds unnecessary complexity)
2. **Using wrong pattern** (understand the problem first)
3. **Forcing patterns** (sometimes simple code is better)

### Best Practices

1. **Learn when to use each pattern**
2. **Keep it simple** (don't over-engineer)
3. **Prefer composition over inheritance**
4. **Use patterns as guidelines**, not strict rules

---

## Performance Optimization

### Simple Explanation

Performance optimization is like tuning a car engine - you make it run faster and more efficiently. In JavaScript, this means:

- Writing code that executes quickly
- Using less memory
- Loading resources efficiently
- Reducing unnecessary work

Think of optimization as organizing your workspace:

- Keep frequently used tools nearby (caching)
- Clean up clutter (memory management)
- Use efficient methods (algorithms)
- Don't do unnecessary work (lazy loading)

### Technical Explanation

Performance optimization involves:

- Algorithm efficiency (time complexity)
- Memory management
- Bundle optimization
- Code splitting
- Caching strategies
- Debouncing and throttling
- Virtual DOM and reconciliation

### Merged Explanation

**1. Debouncing and Throttling**

```typescript
// Debounce - executes after wait period of inactivity
function debounce<T extends (...args: any[]) => any>(
  func: T,
  wait: number,
): (...args: Parameters<T>) => void {
  let timeout: NodeJS.Timeout;

  return function (...args: Parameters<T>) {
    clearTimeout(timeout);
    timeout = setTimeout(() => func(...args), wait);
  };
}

// Usage: search as user types
const searchAPI = debounce((query: string) => {
  console.log('Searching for:', query);
  fetch(`/api/search?q=${query}`);
}, 300);

input.addEventListener('input', (e) => {
  searchAPI(e.target.value);
});

// Throttle - executes at most once per interval
function throttle<T extends (...args: any[]) => any>(
  func: T,
  limit: number,
): (...args: Parameters<T>) => void {
  let inThrottle: boolean;

  return function (...args: Parameters<T>) {
    if (!inThrottle) {
      func(...args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}

// Usage: scroll event
const handleScroll = throttle(() => {
  console.log('Scroll position:', window.scrollY);
}, 100);

window.addEventListener('scroll', handleScroll);
```

**2. Memoization**

```typescript
// Cache function results
function memoize<T extends (...args: any[]) => any>(fn: T): T {
  const cache = new Map<string, ReturnType<T>>();

  return ((...args: Parameters<T>) => {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      return cache.get(key)!;
    }

    const result = fn(...args);
    cache.set(key, result);
    return result;
  }) as T;
}

// Expensive calculation
const fibonacci = memoize((n: number): number => {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.time('First call');
console.log(fibonacci(40)); // Slow
console.timeEnd('First call');

console.time('Second call');
console.log(fibonacci(40)); // Instant (cached)
console.timeEnd('Second call');

// React useMemo
function ExpensiveComponent({ data }: { data: number[] }) {
  const processed = useMemo(() => {
    console.log('Processing data');
    return data.map((n) => n * 2).sort((a, b) => b - a);
  }, [data]); // Only recompute when data changes

  return <div>{processed.join(', ')}</div>;
}
```

**3. Lazy Loading**

```typescript
// Dynamic imports
async function loadFeature() {
  const module = await import('./heavy-feature.js');
  module.initialize();
}

// Load on demand
button.addEventListener('click', loadFeature);

// React lazy loading
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <HeavyComponent />
    </Suspense>
  );
}

// Image lazy loading
<img
  src="placeholder.jpg"
  data-src="actual-image.jpg"
  loading="lazy"
  alt="Description"
/>;

// Intersection Observer for custom lazy loading
const observer = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      const img = entry.target as HTMLImageElement;
      img.src = img.dataset.src!;
      observer.unobserve(img);
    }
  });
});

document.querySelectorAll('img[data-src]').forEach((img) => {
  observer.observe(img);
});
```

**4. Virtual Scrolling**

```typescript
// Only render visible items
interface VirtualListProps {
  items: any[];
  itemHeight: number;
  containerHeight: number;
}

function VirtualList({ items, itemHeight, containerHeight }: VirtualListProps) {
  const [scrollTop, setScrollTop] = useState(0);

  const visibleStart = Math.floor(scrollTop / itemHeight);
  const visibleEnd = Math.ceil((scrollTop + containerHeight) / itemHeight);

  const visibleItems = items.slice(visibleStart, visibleEnd);

  const offsetY = visibleStart * itemHeight;
  const totalHeight = items.length * itemHeight;

  return (
    <div
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={(e) => setScrollTop(e.currentTarget.scrollTop)}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>
          {visibleItems.map((item, index) => (
            <div key={visibleStart + index} style={{ height: itemHeight }}>
              {item}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}
```

**5. Web Workers**

```typescript
// Heavy computation in background thread
// worker.ts
self.addEventListener('message', (e) => {
  const result = heavyComputation(e.data);
  self.postMessage(result);
});

function heavyComputation(data: any) {
  // CPU-intensive task
  let sum = 0;
  for (let i = 0; i < 1000000000; i++) {
    sum += i;
  }
  return sum;
}

// main.ts
const worker = new Worker('worker.js');

worker.postMessage({ data: 'some data' });

worker.addEventListener('message', (e) => {
  console.log('Result from worker:', e.data);
});

// Terminate when done
worker.terminate();
```

**6. Code Splitting**

```typescript
// Webpack
const routes = [
  {
    path: '/home',
    component: () => import('./pages/Home'),
  },
  {
    path: '/dashboard',
    component: () => import('./pages/Dashboard'),
  },
  {
    path: '/profile',
    component: () => import('./pages/Profile'),
  },
];

// React Router
const Home = lazy(() => import('./pages/Home'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
  return (
    <Router>
      <Suspense fallback={<Loading />}>
        <Routes>
          <Route path="/home" element={<Home />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Suspense>
    </Router>
  );
}
```

**7. Efficient DOM Operations**

```typescript
// Bad - multiple reflows
const container = document.getElementById('container');
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  div.textContent = `Item ${i}`;
  container.appendChild(div); // Reflow on each append!
}

// Good - single reflow
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
  const div = document.createElement('div');
  div.textContent = `Item ${i}`;
  fragment.appendChild(div);
}
container.appendChild(fragment); // Single reflow

// Best - template string (even faster)
const html = Array.from(
  { length: 1000 },
  (_, i) => `<div>Item ${i}</div>`,
).join('');
container.innerHTML = html;

// Batch style changes
const element = document.getElementById('myElement');

// Bad - multiple reflows
element.style.width = '100px';
element.style.height = '100px';
element.style.backgroundColor = 'red';

// Good - single reflow
element.style.cssText = 'width: 100px; height: 100px; background-color: red;';

// Or use classes
element.className = 'styled-element';
```

**8. Event Delegation**

```typescript
// Bad - attach listener to each item
document.querySelectorAll('.item').forEach((item) => {
  item.addEventListener('click', handleClick);
});

// Good - single listener on parent
document.getElementById('container').addEventListener('click', (e) => {
  const target = e.target as HTMLElement;
  if (target.classList.contains('item')) {
    handleClick(e);
  }
});
```

**9. Optimize Loops**

```typescript
// Cache length
const items = [
  /* large array */
];

// Bad
for (let i = 0; i < items.length; i++) {
  process(items[i]);
}

// Good
const length = items.length;
for (let i = 0; i < length; i++) {
  process(items[i]);
}

// Best - use forEach/map when appropriate
items.forEach(process);

// Avoid nested loops when possible
// Bad - O(n²)
for (let i = 0; i < arr1.length; i++) {
  for (let j = 0; j < arr2.length; j++) {
    if (arr1[i] === arr2[j]) {
      // match
    }
  }
}

// Good - O(n) with Set
const set = new Set(arr2);
for (const item of arr1) {
  if (set.has(item)) {
    // match
  }
}
```

**10. Avoid Memory Leaks**

```typescript
// Memory leak: event listeners not removed
class Component {
  constructor() {
    window.addEventListener('resize', this.handleResize);
  }

  handleResize = () => {
    // ...
  };

  // Missing cleanup!
}

// Fixed
class ComponentFixed {
  constructor() {
    window.addEventListener('resize', this.handleResize);
  }

  handleResize = () => {
    // ...
  };

  destroy() {
    window.removeEventListener('resize', this.handleResize);
  }
}

// Memory leak: timers not cleared
const timers = [];
function startTimer() {
  timers.push(setInterval(() => {}, 1000)); // Accumulates!
}

// Fixed
const timers = [];
function startTimerFixed() {
  const id = setInterval(() => {}, 1000);
  timers.push(id);
  return () => clearInterval(id);
}

// Memory leak: closures holding references
function createButtons() {
  const buttons = [];
  const largeData = new Array(1000000).fill('data');

  for (let i = 0; i < 100; i++) {
    buttons.push(() => {
      console.log(i);
      console.log(largeData); // Holds reference to largeData!
    });
  }

  return buttons;
}

// Fixed - don't capture unnecessary data
function createButtonsFixed() {
  const buttons = [];

  for (let i = 0; i < 100; i++) {
    buttons.push(() => {
      console.log(i);
    });
  }

  return buttons;
}
```

### Common Pitfalls

1. **Premature optimization** (profile first!)
2. **Optimizing wrong things** (focus on bottlenecks)
3. **Making code unreadable** (balance performance and maintainability)

### Best Practices

1. **Measure before optimizing** (use profiling tools)
2. **Focus on hot paths** (code that runs frequently)
3. **Lazy load heavy resources**
4. **Cache expensive computations**
5. **Batch DOM updates**
6. **Use efficient algorithms** (understand Big O)
7. **Clean up resources** (prevent memory leaks)

---

## Interview Questions

### Basic Level

**Q1: What is the difference between `let`, `const`, and `var`?**

**Answer:**

- `var`: Function-scoped, hoisted, can be redeclared
- `let`: Block-scoped, not hoisted (TDZ), cannot be redeclared
- `const`: Block-scoped, not hoisted (TDZ), cannot be reassigned or redeclared (but objects are mutable)

```typescript
// var example
function varTest() {
  var x = 1;
  if (true) {
    var x = 2; // Same variable!
    console.log(x); // 2
  }
  console.log(x); // 2
}

// let example
function letTest() {
  let x = 1;
  if (true) {
    let x = 2; // Different variable
    console.log(x); // 2
  }
  console.log(x); // 1
}
```

**Q2: Explain closures with an example.**

**Answer:** A closure is when a function remembers variables from its outer scope even after the outer function has returned.

```typescript
function createCounter() {
  let count = 0; // Private variable

  return {
    increment() {
      count++;
      return count;
    },
    decrement() {
      count--;
      return count;
    },
    getCount() {
      return count;
    },
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.getCount()); // 2
```

**Q3: What is event delegation?**

**Answer:** Event delegation is attaching a single event listener to a parent element instead of multiple listeners to child elements. Uses event bubbling.

```typescript
// Bad - listener on each item
items.forEach((item) => {
  item.addEventListener('click', handleClick);
});

// Good - single listener on parent
parent.addEventListener('click', (e) => {
  if (e.target.matches('.item')) {
    handleClick(e);
  }
});
```

**Q4: What is the difference between `==` and `===`?**

**Answer:**

- `==`: Loose equality, performs type coercion
- `===`: Strict equality, no type coercion

```typescript
console.log(5 == '5'); // true (coercion)
console.log(5 === '5'); // false (different types)
console.log(null == undefined); // true
console.log(null === undefined); // false
```

**Q5: Explain hoisting.**

**Answer:** Hoisting is JavaScript's behavior of moving declarations to the top of their scope during compilation.

```typescript
console.log(x); // undefined (not error!)
var x = 5;

// Interpreted as:
var x;
console.log(x); // undefined
x = 5;

// Functions are fully hoisted
greet(); // Works!
function greet() {
  console.log('Hello');
}

// But let/const have Temporal Dead Zone
// console.log(y); // ReferenceError
let y = 5;
```

### Intermediate Level

**Q6: How does `this` work in JavaScript?**

**Answer:** `this` refers to the context in which a function is called:

1. Default: global object or undefined (strict mode)
2. Method call: the object before the dot
3. call/apply/bind: explicitly set
4. Arrow functions: lexical (from enclosing scope)
5. new: the new object being created

```typescript
const obj = {
  name: 'Ale',
  greet: function () {
    console.log(this.name); // "Ale"
  },
  greetArrow: () => {
    console.log(this.name); // undefined (arrow function)
  },
};

obj.greet(); // Method call - this = obj
const fn = obj.greet;
fn(); // Function call - this = undefined/global
```

**Q7: Explain prototypal inheritance.**

**Answer:** Every object has a prototype (another object) from which it inherits properties and methods.

```typescript
function Animal(name: string) {
  this.name = name;
}

Animal.prototype.speak = function () {
  console.log(`${this.name} makes a sound`);
};

function Dog(name: string) {
  Animal.call(this, name);
}

Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function () {
  console.log(`${this.name} barks`);
};

const dog = new Dog('Rex');
dog.speak(); // Inherited from Animal
dog.bark(); // From Dog
```

**Q8: What is the event loop?**

**Answer:** The event loop is JavaScript's concurrency model. It handles:

1. Call stack (synchronous code)
2. Microtask queue (promises, queueMicrotask)
3. Macrotask queue (setTimeout, setInterval)

```typescript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

console.log('4');

// Output: 1, 4, 3, 2
// Sync first, then microtasks, then macrotasks
```

**Q9: How do you handle errors in async functions?**

**Answer:**

```typescript
// Try-catch
async function fetchData() {
  try {
    const data = await fetch('/api/data');
    return await data.json();
  } catch (error) {
    console.error('Failed:', error);
    throw error;
  }
}

// .catch()
fetchData()
  .then((data) => console.log(data))
  .catch((error) => console.error(error));

// Multiple try-catch
async function process() {
  let data;
  try {
    data = await fetchData();
  } catch (error) {
    console.error('Fetch failed');
    return null;
  }

  try {
    return processData(data);
  } catch (error) {
    console.error('Process failed');
    return data; // Return partial result
  }
}
```

**Q10: What's the difference between `map()`, `forEach()`, `filter()`, and `reduce()`?**

**Answer:**

```typescript
const numbers = [1, 2, 3, 4, 5];

// map - transforms, returns new array
const doubled = numbers.map((n) => n * 2);
// [2, 4, 6, 8, 10]

// forEach - side effects, returns undefined
numbers.forEach((n) => console.log(n));

// filter - selects, returns new array
const evens = numbers.filter((n) => n % 2 === 0);
// [2, 4]

// reduce - aggregates, returns single value
const sum = numbers.reduce((total, n) => total + n, 0);
// 15
```

### Advanced Level

**Q11: Implement debounce and throttle.**

**Answer:**

```typescript
function debounce<T extends (...args: any[]) => any>(
  func: T,
  wait: number,
): (...args: Parameters<T>) => void {
  let timeout: NodeJS.Timeout;

  return function (...args: Parameters<T>) {
    clearTimeout(timeout);
    timeout = setTimeout(() => func(...args), wait);
  };
}

function throttle<T extends (...args: any[]) => any>(
  func: T,
  limit: number,
): (...args: Parameters<T>) => void {
  let inThrottle: boolean;

  return function (...args: Parameters<T>) {
    if (!inThrottle) {
      func(...args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}
```

**Q12: Implement Promise.all()**

**Answer:**

```typescript
function promiseAll<T>(promises: Promise<T>[]): Promise<T[]> {
  return new Promise((resolve, reject) => {
    if (promises.length === 0) {
      resolve([]);
      return;
    }

    const results: T[] = [];
    let completed = 0;

    promises.forEach((promise, index) => {
      Promise.resolve(promise)
        .then((value) => {
          results[index] = value;
          completed++;

          if (completed === promises.length) {
            resolve(results);
          }
        })
        .catch(reject);
    });
  });
}
```

**Q13: Implement deep clone.**

**Answer:**

```typescript
function deepClone<T>(obj: T, hash = new WeakMap()): T {
  // Handle primitives and functions
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }

  // Handle circular references
  if (hash.has(obj)) {
    return hash.get(obj);
  }

  // Handle Date
  if (obj instanceof Date) {
    return new Date(obj.getTime()) as any;
  }

  // Handle RegExp
  if (obj instanceof RegExp) {
    return new RegExp(obj) as any;
  }

  // Handle Array
  if (Array.isArray(obj)) {
    const arrCopy: any[] = [];
    hash.set(obj, arrCopy);
    obj.forEach((item, index) => {
      arrCopy[index] = deepClone(item, hash);
    });
    return arrCopy as any;
  }

  // Handle Object
  const objCopy: any = Object.create(Object.getPrototypeOf(obj));
  hash.set(obj, objCopy);
  Object.keys(obj).forEach((key) => {
    objCopy[key] = deepClone((obj as any)[key], hash);
  });

  return objCopy;
}
```

**Q14: Explain and implement a simple event emitter.**

**Answer:**

```typescript
class EventEmitter {
  private events: Map<string, Function[]> = new Map();

  on(event: string, listener: Function): () => void {
    if (!this.events.has(event)) {
      this.events.set(event, []);
    }
    this.events.get(event)!.push(listener);

    // Return unsubscribe function
    return () => this.off(event, listener);
  }

  off(event: string, listener: Function) {
    const listeners = this.events.get(event);
    if (listeners) {
      const index = listeners.indexOf(listener);
      if (index > -1) {
        listeners.splice(index, 1);
      }
    }
  }

  emit(event: string, ...args: any[]) {
    const listeners = this.events.get(event);
    if (listeners) {
      listeners.forEach((listener) => listener(...args));
    }
  }

  once(event: string, listener: Function) {
    const onceWrapper = (...args: any[]) => {
      listener(...args);
      this.off(event, onceWrapper);
    };
    this.on(event, onceWrapper);
  }
}
```

**Q15: How would you optimize a React application?**

**Answer:**

1. Use React.memo for expensive components
2. Use useMemo for expensive calculations
3. Use useCallback for stable function references
4. Lazy load components with React.lazy
5. Virtual scrolling for long lists
6. Code splitting by route
7. Avoid inline object/array creation in render
8. Use production build
9. Analyze bundle size

```typescript
// Memoize component
const ExpensiveComponent = React.memo(({ data }) => {
  return <div>{/* Expensive render */}</div>;
});

// Memoize value
function Component({ data }) {
  const processed = useMemo(() => {
    return expensiveProcessing(data);
  }, [data]);

  return <div>{processed}</div>;
}

// Memoize callback
function Parent() {
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);

  return <Child onClick={handleClick} />;
}

// Lazy load
const Heavy = React.lazy(() => import('./Heavy'));

function App() {
  return (
    <Suspense fallback={<Loading />}>
      <Heavy />
    </Suspense>
  );
}
```

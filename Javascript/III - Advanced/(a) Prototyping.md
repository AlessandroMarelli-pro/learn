### Prototyping & Inheritance — Teacher’s Synthesis

Source: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain

### 1) Prototype chain: how lookup works

- Every object has an internal `[[Prototype]]` that points to another object (or `null`).
- Property access walks own properties, then the prototype, then up the chain until found or `null`.
- Methods found on prototypes are shared across instances.

```js
const base = { b: 2 };
const obj = Object.create(base); // set [[Prototype]] to base
obj.a = 1;
obj.b; // 2 (inherited)
'toString' in obj; // true (from Object.prototype)
```

### 2) Constructors and `prototype`

- Functions used with `new` act as constructors. Their `.prototype` object becomes the `[[Prototype]]` of created instances.
- Instance → constructor.prototype → Object.prototype → null.

```js
function Person(name) {
  this.name = name;
}
Person.prototype.greet = function () {
  return `Hi, ${this.name}`;
};
const p = new Person('Ava');
p.greet(); // method from Person.prototype
```

### 3) Creating/mutating prototype chains

- `Object.create(proto, descriptors?)` — preferred way to create with a given prototype.
- `Object.setPrototypeOf(obj, proto)` — changes `[[Prototype]]` at runtime (slow; avoid on hot paths).
- In literals, `{ __proto__: other }` sets the prototype of the new object.

```js
const proto = { kind: 'shape' };
const circle = Object.create(proto, { r: { value: 10, enumerable: true } });
```

### 4) Shadowing, descriptors, and accessors

- An own property with the same name hides (shadows) a prototype property.
- Descriptors control writability, enumerability, configurability; getters/setters intercept access.

```js
const proto2 = { x: 1 };
const o = Object.create(proto2);
o.x = 2; // shadows proto2.x
Object.getPrototypeOf(o).x; // 1
```

### 5) ES classes: sugar over prototypes

- `class`/`extends` define the same prototype relationships with nicer syntax; not a new inheritance model.
- `super` calls access parent methods; methods live on the prototype.

```js
class A {
  m() {
    return 'A';
  }
}
class B extends A {
  m() {
    return super.m() + 'B';
  }
}
new B().m(); // 'AB'
```

> ⚠️ **Pitfalls (Read Carefully)**
>
> - Confusing `__proto__` (object's prototype) with `Function.prototype` (prototype assigned to instances).
> - Mutating prototypes at runtime (`setPrototypeOf`/`__proto__`) deoptimizes performance; prefer `Object.create` at construction time.
> - Shadowing can mask fixes on prototypes; reading hits own props first.
> - Writing to an inherited accessor may invoke a setter; writing to an inherited data prop creates an own prop instead.
> - Extending `Object.prototype` or built‑ins can break libraries/enumeration.
> - `this` depends on call site; extracting methods from prototypes without binding can change `this`.

### Interview‑Style Questions (Practice)

- Explain the prototype chain for an instance created by `new C()`; where are methods stored?
- Difference between `obj.__proto__`, `Object.getPrototypeOf(obj)`, and `Func.prototype`.
- Why is `Object.create` preferred over `setPrototypeOf`? Performance and safety.
- Show property shadowing and how to read the parent’s value.
- How do getters/setters on prototypes affect reads/writes on instances?
- In classes, what does `super.m()` do under the hood with prototypes?

### References

- MDN — Inheritance and the prototype chain: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Inheritance_and_the_prototype_chain

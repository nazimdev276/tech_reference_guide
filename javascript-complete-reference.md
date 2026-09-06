# JavaScript: The Complete Study & Reference Guide

*A single-file reference covering basics, advanced concepts, built-in functions by use case, and effective npm usage.*

> **How to use this file:** It's organized to be read top-to-bottom as a course, or jumped into as a reference via the Table of Contents. Every section has runnable code examples. Search (Ctrl/Cmd+F) for a method name if you just need a quick lookup.

---

## Table of Contents

1. [Getting Started](#1-getting-started)
2. [Variables & Data Types](#2-variables--data-types)
3. [Operators](#3-operators)
4. [Control Flow](#4-control-flow)
5. [Functions](#5-functions)
6. [Scope, Hoisting & Closures](#6-scope-hoisting--closures)
7. [Objects](#7-objects)
8. [Arrays](#8-arrays)
9. [Destructuring, Spread & Rest](#9-destructuring-spread--rest)
10. [`this`, `call`/`apply`/`bind`](#10-this-callapplybind)
11. [Prototypes & Classes](#11-prototypes--classes)
12. [Asynchronous JavaScript](#12-asynchronous-javascript)
13. [Error Handling](#13-error-handling)
14. [Iterators & Generators](#14-iterators--generators)
15. [Modules](#15-modules)
16. [ES6+ Collections: Map, Set, WeakMap, WeakSet](#16-es6-collections-map-set-weakmap-weakset)
17. [Symbols, Proxy & Reflect](#17-symbols-proxy--reflect)
18. [Regular Expressions](#18-regular-expressions)
19. [Built-in Functions Grouped by Use Case](#19-built-in-functions-grouped-by-use-case)
20. [Browser & Runtime APIs (Quick Tour)](#20-browser--runtime-apis-quick-tour)
21. [Using npm Effectively](#21-using-npm-effectively)
22. [Best Practices & Common Pitfalls](#22-best-practices--common-pitfalls)
23. [Cheat Sheet Appendix](#23-cheat-sheet-appendix)

---

## 1. Getting Started

JavaScript (JS) is a dynamically-typed, prototype-based, multi-paradigm language. It runs in browsers (via engines like V8, SpiderMonkey) and outside browsers via **Node.js** (or Deno/Bun) using the same V8-family engines.

### Running JavaScript

- **Browser console**: DevTools → Console tab.
- **Node.js**: install from nodejs.org, then run `node file.js` or `node` for a REPL.
- **`<script>` tag** in HTML:

```html
<script src="app.js" defer></script>
```

`defer` waits for HTML parsing to finish before running the script, and preserves order across multiple scripts. `async` runs as soon as the script is downloaded, without waiting — order isn't guaranteed.

### Strict Mode

```js
"use strict";
```

Enables stricter parsing/error handling (e.g., prevents accidental globals, disallows duplicate params). ES modules and classes are strict mode by default automatically.


---

## 2. Variables & Data Types

### Declaring Variables

```js
var x = 1;    // function-scoped, hoisted, can be redeclared — avoid in modern code
let y = 2;    // block-scoped, can be reassigned, cannot be redeclared in same scope
const z = 3;  // block-scoped, cannot be reassigned (but object/array CONTENTS can mutate)
```

**Rule of thumb:** default to `const`. Use `let` when you need to reassign. Avoid `var`.

```js
const arr = [1, 2, 3];
arr.push(4);       // OK — mutating contents, not reassigning the binding
arr = [5, 6];       // TypeError: Assignment to constant variable.
```

### Primitive Types

JavaScript has 7 primitive types (immutable, compared by value):

| Type        | Example                     | `typeof` result |
|-------------|------------------------------|------------------|
| Number      | `42`, `3.14`, `NaN`, `Infinity` | `"number"` |
| BigInt      | `123n`                       | `"bigint"` |
| String      | `"hi"`, `'hi'`, `` `hi` ``   | `"string"` |
| Boolean     | `true`, `false`              | `"boolean"` |
| undefined   | `undefined`                  | `"undefined"` |
| null        | `null`                       | `"object"` (a famous historical bug) |
| Symbol      | `Symbol('id')`                | `"symbol"` |

### Reference Type

Everything else — `Object`, `Array`, `Function`, `Date`, `Map`, `Set`, etc. — is an **object**, compared by reference, and mutable.

```js
typeof {};          // "object"
typeof [];           // "object"  (arrays are objects!)
typeof function(){}; // "function"
typeof null;         // "object"  (quirk, not a bug you can rely on)
```

### Type Coercion

JS is loosely typed and coerces types implicitly in many operations.

```js
"5" + 1;      // "51"  (string concatenation wins)
"5" - 1;      // 4     (numeric coercion, - has no string meaning)
"5" * "2";    // 10
true + 1;     // 2     (true -> 1)
[] + [];      // ""    (both -> "")
[] + {};      // "[object Object]"
null == undefined;   // true  (loose equality special case)
null === undefined;  // false (different types)
NaN === NaN;         // false (!) — use Number.isNaN() or Object.is()
```

**Always prefer `===` and `!==`** (strict equality) over `==`/`!=` unless you specifically want type coercion (e.g., checking for both `null` and `undefined` with `== null`).

### Checking Types Reliably

```js
Array.isArray([1,2,3]);              // true
Object.prototype.toString.call([]);   // "[object Array]"
Object.prototype.toString.call(null); // "[object Null]"
Number.isInteger(4);                  // true
Number.isNaN(NaN);                    // true (safer than global isNaN)
```

### Type Conversion (Explicit)

```js
String(123);        // "123"
Number("123");       // 123
Number("abc");        // NaN
Boolean(0);           // false
Boolean("");          // false
Boolean("0");         // true  (non-empty string is truthy!)
parseInt("42px");     // 42
parseFloat("3.14m");  // 3.14
+"42";                // 42  (unary plus converts to number)
!!"hello";            // true (double negation converts to boolean)
```

### Falsy Values (memorize these — everything else is truthy)

```
false, 0, -0, 0n, "", null, undefined, NaN
```

Everything else — including `"0"`, `"false"`, `[]`, `{}` — is **truthy**.


---

## 3. Operators

### Arithmetic
```js
+  -  *  /  %  **      // % = remainder, ** = exponent (2**3 === 8)
x++  x--  ++x  --x     // increment/decrement (post vs pre)
```

### Assignment
```js
=  +=  -=  *=  /=  %=  **=
&&=  ||=  ??=           // logical assignment (ES2021)

let a = null;
a ??= 5;   // a = 5 (only assigns if a is null/undefined)

let count = 0;
count ||= 10; // count = 10 (0 is falsy)
```

### Comparison
```js
===  !==   // strict (no coercion) — PREFER THESE
==   !=    // loose (with coercion) — avoid unless intentional
>  <  >=  <=
```

### Logical
```js
&&   // AND — returns first falsy value or last value
||   // OR  — returns first truthy value or last value
!    // NOT
??   // Nullish coalescing — returns right side ONLY if left is null/undefined

0 || "default";   // "default"  (0 is falsy)
0 ?? "default";    // 0          (0 is not null/undefined)
```

### Optional Chaining
```js
const user = { profile: { name: "Ana" } };
user.profile?.name;        // "Ana"
user.address?.city;        // undefined (no error thrown)
user.getAge?.();           // undefined if getAge doesn't exist
user.list?.[0];            // safe array-style access
```

### Ternary
```js
const status = age >= 18 ? "adult" : "minor";
```

### Comma Operator
```js
for (let i = 0, j = 10; i < j; i++, j--) { /* ... */ }
```

### `typeof`, `instanceof`, `in`
```js
typeof "hi";                 // "string"
[] instanceof Array;          // true
"name" in { name: "x" };      // true (checks own AND inherited properties)
```

### Bitwise (less common, but used for flags/perf tricks)
```js
&  |  ^  ~  <<  >>  >>>
5 & 1;    // 1
~5;       // -6
1 << 3;   // 8
```


---

## 4. Control Flow

### if / else if / else
```js
if (score >= 90) {
  grade = "A";
} else if (score >= 80) {
  grade = "B";
} else {
  grade = "F";
}
```

### switch
```js
switch (day) {
  case "Mon":
  case "Tue":
    console.log("Early week");
    break;
  case "Fri":
    console.log("Almost weekend");
    break;
  default:
    console.log("Unknown");
}
```
Forgetting `break` causes **fall-through** into the next case (sometimes intentional, usually a bug).

### Loops

```js
// Classic for
for (let i = 0; i < 5; i++) { console.log(i); }

// while
let i = 0;
while (i < 5) { console.log(i); i++; }

// do...while (runs body at least once)
let j = 0;
do { console.log(j); j++; } while (j < 5);

// for...of  → iterates VALUES of an iterable (arrays, strings, Maps, Sets)
for (const value of [10, 20, 30]) { console.log(value); }

// for...in  → iterates KEYS of an object (or indices of an array — avoid for arrays!)
for (const key in { a: 1, b: 2 }) { console.log(key); }
```

**for...of vs for...in:** use `for...of` for arrays/iterables (values); use `for...in` only for plain object keys. Using `for...in` on arrays can pick up inherited enumerable properties and doesn't guarantee order — prefer `for...of`, `.forEach()`, or `Object.entries()`.

### Loop Control
```js
for (let i = 0; i < 10; i++) {
  if (i === 3) continue;  // skip this iteration
  if (i === 6) break;     // exit loop entirely
  console.log(i);
}
```

### Labeled statements (rare, useful for nested loop breaks)
```js
outer: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 1) continue outer;
    console.log(i, j);
  }
}
```


---

## 5. Functions

### Declaration vs Expression vs Arrow

```js
// Function declaration — hoisted (can be called before defined in source)
function add(a, b) { return a + b; }

// Function expression — NOT hoisted (the variable is, but not the assignment)
const subtract = function (a, b) { return a - b; };

// Named function expression (name helps in stack traces & recursion)
const factorial = function fact(n) { return n <= 1 ? 1 : n * fact(n - 1); };

// Arrow function — shorter syntax, NO own `this`, `arguments`, or `super`
const multiply = (a, b) => a * b;
const square = x => x * x;              // single param: parens optional
const noop = () => {};                    // empty body
const makeObj = () => ({ key: "val" });  // wrap object literal in parens!
```

### Default, Rest Parameters

```js
function greet(name = "friend") { return `Hello, ${name}`; }

function sum(...numbers) {           // rest param — gathers remaining args into array
  return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4); // 10
```

### `arguments` object (only in regular functions, not arrows)
```js
function old() {
  console.log(arguments.length);
  console.log(Array.from(arguments)); // convert to real array
}
```

### Immediately Invoked Function Expression (IIFE)
```js
(function () {
  console.log("Runs immediately, own scope");
})();

(() => { console.log("arrow IIFE"); })();
```

### Higher-Order Functions
A function that takes and/or returns another function.
```js
function withLogging(fn) {
  return function (...args) {
    console.log("calling with", args);
    return fn(...args);
  };
}
const loggedAdd = withLogging(add);
loggedAdd(2, 3); // logs then returns 5
```

### Pure Functions
A pure function: (1) given the same input always returns the same output, (2) has no side effects (doesn't mutate external state, doesn't do I/O). Pure functions are easier to test, reason about, and memoize.

### Recursion
```js
function fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}
```
Tail-call optimization is specified in ES6 but not implemented in most engines (V8/Node included) — deep recursion can still overflow the stack.

### Currying & Partial Application
```js
const curriedAdd = a => b => c => a + b + c;
curriedAdd(1)(2)(3); // 6

function partial(fn, ...presetArgs) {
  return (...laterArgs) => fn(...presetArgs, ...laterArgs);
}
const add10 = partial(add, 10);
add10(5); // 15
```

### Arrow Functions vs Regular Functions — Key Differences

| Feature | Regular function | Arrow function |
|---|---|---|
| Own `this` | Yes (depends on how it's called) | No — inherits `this` from enclosing scope |
| `arguments` object | Yes | No (use rest params) |
| Can be used as constructor (`new`) | Yes | No — throws TypeError |
| `prototype` property | Yes | No |
| Good for | Methods, constructors | Callbacks, short utilities, preserving outer `this` |

```js
const obj = {
  name: "Widget",
  regularMethod: function () { console.log(this.name); },  // "Widget"
  arrowMethod: () => { console.log(this.name); },            // undefined (this = outer scope)
};
```


---

## 6. Scope, Hoisting & Closures

### Scope Types
- **Global scope**: accessible everywhere.
- **Function scope**: `var` is scoped to the nearest enclosing function.
- **Block scope**: `let`/`const` are scoped to the nearest `{ }` block (if, for, while, or bare blocks).

```js
if (true) {
  var a = 1;   // leaks outside the block (function-scoped)
  let b = 2;   // stays inside this block
}
console.log(a); // 1
console.log(b); // ReferenceError
```

### Hoisting
Declarations are processed before code executes.

```js
console.log(x); // undefined (var is hoisted, initialized as undefined)
var x = 5;

console.log(y); // ReferenceError: Cannot access 'y' before initialization
let y = 5;       // let/const are hoisted too, but land in the "Temporal Dead Zone" (TDZ)

sayHi(); // works! Function declarations are fully hoisted (including body)
function sayHi() { console.log("hi"); }

sayBye(); // TypeError: sayBye is not a function
var sayBye = function () { console.log("bye"); }; // only the var declaration hoists
```

### Closures
A closure is a function that "remembers" variables from the scope it was created in, even after that outer scope has returned.

```js
function makeCounter() {
  let count = 0;               // private state
  return {
    increment: () => ++count,
    decrement: () => --count,
    value: () => count,
  };
}
const counter = makeCounter();
counter.increment(); // 1
counter.increment(); // 2
counter.value();     // 2
// `count` is not accessible directly from outside — this is how JS does "privacy"
```

**Classic closure-in-loop pitfall:**
```js
// BUGGY with var — logs 3, 3, 3 (all callbacks share the same `i`)
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}

// FIXED with let — logs 0, 1, 2 (each iteration gets its own binding)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
```

### Module Pattern (pre-ES6 way of using closures for encapsulation)
```js
const BankAccount = (function () {
  let balance = 0; // private
  return {
    deposit(amount) { balance += amount; return balance; },
    getBalance() { return balance; },
  };
})();
```


---

## 7. Objects

### Creating Objects
```js
const obj1 = { key: "value", num: 42 };     // object literal
const obj2 = new Object();                    // constructor (rarely used)
const obj3 = Object.create(protoObj);         // with explicit prototype
```

### Accessing / Modifying Properties
```js
const person = { name: "Sam", age: 30 };
person.name;          // dot notation
person["age"];         // bracket notation (needed for dynamic/invalid identifier keys)
person.email = "sam@example.com"; // add
delete person.age;                  // remove
"name" in person;                   // true — property existence check
person.hasOwnProperty("name");      // true — own (not inherited) property check
```

### Computed Property Names
```js
const key = "score";
const result = { [key]: 100 };   // { score: 100 }
```

### Shorthand Properties & Methods
```js
const x = 1, y = 2;
const point = { x, y };                  // shorthand for { x: x, y: y }
const obj = {
  greet() { return "hi"; },               // method shorthand
};
```

### Object Methods (grouped)

**Copying / merging**
```js
Object.assign({}, obj1, obj2);   // shallow merge (later sources win on conflict)
{ ...obj1, ...obj2 };              // spread — same effect, more common today
structuredClone(obj1);              // deep clone (modern browsers & Node 17+)
```

**Inspecting**
```js
Object.keys(obj);      // ["key", "num"]  — array of own enumerable keys
Object.values(obj);     // ["value", 42]
Object.entries(obj);     // [["key","value"], ["num",42]]
Object.fromEntries([["a",1],["b",2]]);  // { a: 1, b: 2 } — inverse of entries()
```

**Freezing / sealing (immutability)**
```js
Object.freeze(obj);       // prevents add/remove/modify of properties (shallow!)
Object.isFrozen(obj);      // true
Object.seal(obj);           // prevents add/remove, but existing props still writable
Object.preventExtensions(obj); // prevents adding new properties only
```

**Prototype-related**
```js
Object.getPrototypeOf(obj);
Object.setPrototypeOf(obj, newProto);
Object.create(proto, propsObj);
```

**Property descriptors (fine-grained control)**
```js
Object.defineProperty(obj, "id", {
  value: 1,
  writable: false,
  enumerable: true,
  configurable: false,
});
Object.getOwnPropertyDescriptor(obj, "id");
Object.getOwnPropertyNames(obj); // includes non-enumerable own keys too
```

### Optional Chaining + Nullish Coalescing with Objects
```js
const city = user?.address?.city ?? "Unknown";
```

### JSON Conversion
```js
JSON.stringify(obj);                       // object -> JSON string
JSON.stringify(obj, null, 2);              // pretty-printed, 2-space indent
JSON.stringify(obj, ["name", "age"]);       // only include these keys
JSON.parse(jsonString);                       // JSON string -> object
JSON.parse(jsonString, (key, val) => ...);   // with a reviver function
```


---

## 8. Arrays

Arrays are ordered, index-based, resizable collections (technically objects with numeric keys + a `length` property).

```js
const arr = [1, 2, 3];
const arr2 = new Array(1, 2, 3);   // same result, avoid new Array(n) ambiguity
const empty5 = new Array(5);        // array with length 5, no elements (holes!) — prefer Array(5).fill(0)
```

> Full method-by-method breakdown of array built-ins lives in **Section 19** (grouped by use case: iterate, transform, search, mutate). This section covers array *concepts*.

### Arrays Are Objects
```js
typeof [];            // "object"
Array.isArray([]);    // true — the correct way to check
```

### Sparse Arrays & Holes
```js
const sparse = [1, , 3]; // index 1 is a "hole", not `undefined`
sparse.length; // 3
```

### Multi-dimensional Arrays
```js
const grid = [[1,2,3],[4,5,6],[7,8,9]];
grid[1][2]; // 6
```

### Common Patterns

**Cloning (shallow)**
```js
const copy1 = [...original];
const copy2 = Array.from(original);
const copy3 = original.slice();
```

**Removing duplicates**
```js
const unique = [...new Set([1, 2, 2, 3, 3, 3])]; // [1, 2, 3]
```

**Flattening**
```js
[1, [2, [3, [4]]]].flat(Infinity); // [1, 2, 3, 4]
```

**Checking emptiness**
```js
arr.length === 0;
```

**Array-like objects → real arrays**
```js
Array.from(document.querySelectorAll("div"));
Array.from(arguments);
Array.from({ length: 5 }, (_, i) => i * 2); // [0,2,4,6,8]
```


---

## 9. Destructuring, Spread & Rest

### Array Destructuring
```js
const [a, b, c] = [1, 2, 3];
const [first, , third] = [1, 2, 3];         // skip elements
const [head, ...tail] = [1, 2, 3, 4];        // head=1, tail=[2,3,4]
const [x = 10, y = 20] = [undefined];         // defaults: x=10, y=20
[a, b] = [b, a];                                // swap without temp variable
```

### Object Destructuring
```js
const { name, age } = { name: "Ana", age: 25 };
const { name: fullName } = person;                     // rename while destructuring
const { age = 18 } = {};                                // default value
const { address: { city } = {} } = person;              // nested + default fallback
function greet({ name, greeting = "Hello" }) {           // destructure in params
  return `${greeting}, ${name}`;
}
```

### Spread Operator (`...`)
Expands an iterable/object into individual elements.
```js
const arr = [1, 2, 3];
const arr2 = [0, ...arr, 4];       // [0,1,2,3,4]
Math.max(...arr);                    // spread into function args

const obj = { a: 1 };
const obj2 = { ...obj, b: 2 };      // merge/clone objects
```

### Rest Operator (`...`)
Collects remaining items — opposite direction of spread, used in destructuring/params.
```js
const [first, ...rest] = [1, 2, 3, 4]; // rest = [2, 3, 4]
const { id, ...others } = { id: 1, name: "x", age: 2 }; // others = { name, age }
function log(first, ...rest) { console.log(first, rest); }
```


---

## 10. `this`, `call`/`apply`/`bind`

`this` is determined by **how a function is called**, not where it's defined (except for arrow functions).

### The Four Binding Rules

**1. Default binding** — plain function call
```js
function show() { console.log(this); }
show(); // `undefined` in strict mode, global object otherwise
```

**2. Implicit binding** — called as a method
```js
const obj = { name: "A", show() { console.log(this.name); } };
obj.show(); // "A" — `this` = obj

const detached = obj.show;
detached(); // undefined/error — `this` is lost when detached from obj!
```

**3. Explicit binding** — `call`, `apply`, `bind`
```js
function greet(greeting) { return `${greeting}, ${this.name}`; }
const user = { name: "Sam" };

greet.call(user, "Hi");        // "Hi, Sam" — args passed individually
greet.apply(user, ["Hi"]);      // "Hi, Sam" — args passed as an array
const bound = greet.bind(user); // returns a NEW function permanently bound to user
bound("Hi");                     // "Hi, Sam"
```

**4. `new` binding** — constructor calls
```js
function Person(name) { this.name = name; }
const p = new Person("Ana"); // `this` = the newly created object
```

### Arrow Functions and `this`
Arrow functions don't have their own `this` — they lexically inherit it from where they were *defined*.
```js
const obj = {
  name: "Widget",
  delayedGreet() {
    setTimeout(() => {
      console.log(this.name); // "Widget" — arrow inherits `this` from delayedGreet
    }, 100);
  },
};
```
This is the #1 reason arrow functions are preferred for callbacks inside methods.


---

## 11. Prototypes & Classes

### Prototype Chain
Every object has an internal link (`[[Prototype]]`, exposed as `__proto__`) to another object. Property lookups walk up this chain until found or the chain ends at `null`.

```js
const animal = { eats: true };
const rabbit = Object.create(animal); // rabbit's prototype is `animal`
rabbit.hops = true;
rabbit.eats; // true — found on the prototype chain
```

### Constructor Functions (pre-ES6 way of doing "classes")
```js
function Person(name) {
  this.name = name;
}
Person.prototype.greet = function () {   // shared across all instances (memory-efficient)
  return `Hi, I'm ${this.name}`;
};
const p = new Person("Ana");
p.greet(); // "Hi, I'm Ana"
p instanceof Person; // true
```

### ES6 Classes (syntactic sugar over prototypes)
```js
class Animal {
  static kingdom = "Animalia";        // static property (on the class, not instances)
  #secret = "shh";                      // private field (# prefix, ES2022)

  constructor(name, sound) {
    this.name = name;
    this.sound = sound;
  }

  speak() {                             // instance method (goes on Animal.prototype)
    return `${this.name} says ${this.sound}`;
  }

  static create(name) {                // static method — called on the class itself
    return new Animal(name, "...");
  }

  get description() {                    // getter
    return `A ${this.name}`;
  }
  set nickname(value) {                    // setter
    this._nickname = value;
  }

  #privateMethod() {                        // private method
    return this.#secret;
  }
}

class Dog extends Animal {                  // inheritance
  constructor(name) {
    super(name, "Woof");                    // must call super() before using `this`
  }
  speak() {
    return `${super.speak()} (a dog)`;      // call parent method
  }
}

const rex = new Dog("Rex");
rex.speak();           // "Rex says Woof (a dog)"
rex instanceof Animal;  // true
Animal.kingdom;          // "Animalia"
```

### Key Class Notes
- Classes are **not hoisted** the way function declarations are (they're in the TDZ until evaluated).
- Class methods are **not enumerable** (won't show up in `for...in` or `Object.keys`).
- `class` bodies run in strict mode automatically.
- Private fields (`#field`) are truly private — inaccessible even via bracket notation or reflection.

### Mixins (composing behavior without single-inheritance limits)
```js
const Serializable = Base => class extends Base {
  serialize() { return JSON.stringify(this); }
};
class Model {}
class User extends Serializable(Model) {}
```


---

## 12. Asynchronous JavaScript

JS is single-threaded but handles async work via the **event loop**, **callback queue** (macrotasks), and **microtask queue**.

### The Event Loop (mental model)
1. Run all synchronous code (the "call stack") to completion.
2. Drain the **microtask queue** completely (Promise `.then`/`.catch`/`.finally`, `queueMicrotask`).
3. Run **one** macrotask (`setTimeout`, `setInterval`, I/O callbacks, UI events).
4. Repeat from step 2.

```js
console.log("1");
setTimeout(() => console.log("2 (macrotask)"), 0);
Promise.resolve().then(() => console.log("3 (microtask)"));
console.log("4");
// Output: 1, 4, 3 (microtask), 2 (macrotask)
```
**Takeaway: microtasks (Promises) always run before the next macrotask (setTimeout), even with a 0ms delay.**

### Callbacks (the original pattern)
```js
function fetchData(callback) {
  setTimeout(() => callback(null, { data: "result" }), 1000);
}
fetchData((err, result) => {
  if (err) return console.error(err);
  console.log(result);
});
```
Nesting many callbacks leads to **"callback hell"** — hard to read/maintain. Promises and async/await solve this.

### Promises
A Promise represents a value that will be available now, later, or never (rejected).

```js
const promise = new Promise((resolve, reject) => {
  const success = true;
  setTimeout(() => {
    success ? resolve("Done!") : reject(new Error("Failed"));
  }, 1000);
});

promise
  .then(result => console.log(result))
  .catch(err => console.error(err))
  .finally(() => console.log("cleanup, always runs"));
```

**Promise states:** `pending` → `fulfilled` (resolved) or `rejected`. Once settled, a Promise's state and value never change.

**Chaining:**
```js
fetchUser(id)
  .then(user => fetchPosts(user.id))    // return a new promise -> chained
  .then(posts => console.log(posts))
  .catch(err => console.error("Something failed:", err)); // catches ANY error in the chain
```

**Combinators:**
```js
Promise.all([p1, p2, p3]);        // resolves when ALL resolve; rejects fast on first rejection
Promise.allSettled([p1, p2, p3]);  // waits for all, never short-circuits; gives status per promise
Promise.race([p1, p2, p3]);         // settles as soon as the FIRST settles (resolve or reject)
Promise.any([p1, p2, p3]);           // resolves on first FULFILLED; rejects only if ALL reject
```

```js
Promise.allSettled([p1, p2]).then(results => {
  results.forEach(r => {
    if (r.status === "fulfilled") console.log(r.value);
    else console.log(r.reason);
  });
});
```

### async / await
Syntactic sugar over Promises — makes async code look synchronous.

```js
async function getUser(id) {
  try {
    const res = await fetch(`/api/users/${id}`);      // pauses here without blocking the thread
    if (!res.ok) throw new Error("Request failed");
    const user = await res.json();
    return user;
  } catch (err) {
    console.error("Failed to fetch user:", err);
    throw err; // re-throw if the caller needs to know
  }
}

// An `async function` ALWAYS returns a Promise, even if you `return` a plain value.
getUser(1).then(user => console.log(user));
```

**Running async operations in parallel (don't await sequentially unless order matters!):**
```js
// SLOW — sequential, each waits for the previous
const a = await fetchA();
const b = await fetchB();

// FAST — parallel, both start immediately
const [a, b] = await Promise.all([fetchA(), fetchB()]);
```

**Async iteration:**
```js
for await (const chunk of readableStream) {
  console.log(chunk);
}
```

### Timers
```js
const id = setTimeout(() => console.log("once"), 1000);
clearTimeout(id);

const intervalId = setInterval(() => console.log("repeats"), 1000);
clearInterval(intervalId);

queueMicrotask(() => console.log("runs before next render/macrotask"));
```


---

## 13. Error Handling

### try / catch / finally
```js
try {
  riskyOperation();
} catch (error) {
  console.error("Caught:", error.message);
} finally {
  console.log("Runs no matter what (cleanup)");
}

// Optional catch binding (ES2019) — if you don't need the error object
try { doSomething(); } catch { console.log("failed"); }
```

### Throwing Errors
```js
throw new Error("Something broke");
throw new TypeError("Expected a number");
throw new RangeError("Value out of range");
```

### Built-in Error Types
`Error`, `TypeError`, `RangeError`, `SyntaxError`, `ReferenceError`, `EvalError`, `URIError`.

### Custom Errors
```js
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = "ValidationError";
    this.field = field;
    Error.captureStackTrace?.(this, ValidationError); // V8-specific, optional
  }
}

try {
  throw new ValidationError("Invalid email", "email");
} catch (e) {
  if (e instanceof ValidationError) {
    console.log(`${e.field}: ${e.message}`);
  } else {
    throw e; // re-throw unexpected errors
  }
}
```

### Async Error Handling
```js
// Promises
promise.catch(err => console.error(err));

// async/await
try {
  await riskyAsyncOp();
} catch (err) {
  console.error(err);
}

// Unhandled rejection safety nets (Node.js / browser)
process.on("unhandledRejection", (reason) => console.error(reason)); // Node
window.addEventListener("unhandledrejection", e => console.error(e.reason)); // Browser
```

### Error Object Properties
```js
try { null.foo(); } catch (e) {
  e.message;    // "Cannot read properties of null (reading 'foo')"
  e.name;       // "TypeError"
  e.stack;      // full stack trace (non-standard but universally supported)
}
```


---

## 14. Iterators & Generators

### The Iterator Protocol
An object is iterable if it implements `Symbol.iterator`, returning an object with a `.next()` method that yields `{ value, done }`.

```js
function makeRangeIterator(start, end) {
  let current = start;
  return {
    next() {
      if (current < end) return { value: current++, done: false };
      return { value: undefined, done: true };
    },
    [Symbol.iterator]() { return this; }, // makes it usable in for...of
  };
}
for (const num of makeRangeIterator(1, 4)) console.log(num); // 1, 2, 3
```

### Generators
Functions that can pause (`yield`) and resume — an easier way to build iterators.

```js
function* range(start, end) {
  for (let i = start; i < end; i++) {
    yield i;
  }
}
for (const n of range(1, 4)) console.log(n); // 1, 2, 3
[...range(1, 4)]; // [1, 2, 3] — generators are spreadable too

function* idGenerator() {
  let id = 1;
  while (true) { yield id++; }   // infinite generator — pulled lazily
}
const gen = idGenerator();
gen.next().value; // 1
gen.next().value; // 2
```

**Two-way communication with generators:**
```js
function* echo() {
  const x = yield "first";
  const y = yield `got ${x}`;
  return `final with ${y}`;
}
const g = echo();
g.next();          // { value: "first", done: false }
g.next("hello");    // { value: "got hello", done: false }
g.next("world");     // { value: "final with world", done: true }
```

**Async generators (combine `async` + `function*`):**
```js
async function* asyncRange(start, end) {
  for (let i = start; i < end; i++) {
    await new Promise(r => setTimeout(r, 100));
    yield i;
  }
}
for await (const n of asyncRange(1, 4)) console.log(n);
```

### Making Your Own Classes Iterable
```js
class Collection {
  constructor(items) { this.items = items; }
  [Symbol.iterator]() {
    let i = 0;
    const items = this.items;
    return {
      next: () => i < items.length
        ? { value: items[i++], done: false }
        : { value: undefined, done: true },
    };
  }
}
for (const item of new Collection(["a", "b", "c"])) console.log(item);
```


---

## 15. Modules

Modern JS supports **ES Modules (ESM)** natively; Node.js also historically uses **CommonJS (CJS)**. Know both — you'll encounter both in the wild.

### ES Modules (ESM) — `import` / `export`

```js
// math.js
export const PI = 3.14159;
export function add(a, b) { return a + b; }
export default function multiply(a, b) { return a * b; } // one default export per file

// main.js
import multiply, { PI, add } from "./math.js";
import * as MathUtils from "./math.js";     // namespace import
export { add as sum };                        // re-export with a new name
export * from "./math.js";                    // re-export everything
```

In the browser:
```html
<script type="module" src="main.js"></script>
```
In Node.js: use `.mjs` extension, or set `"type": "module"` in `package.json`.

**Dynamic import (lazy-loading, returns a Promise):**
```js
const module = await import("./heavy-module.js");
```

### CommonJS (CJS) — Node.js's original module system

```js
// math.js
function add(a, b) { return a + b; }
module.exports = { add, PI: 3.14159 };
// or: exports.add = add;

// main.js
const { add, PI } = require("./math.js");
```

### Key ESM vs CJS Differences

| | ESM | CommonJS |
|---|---|---|
| Syntax | `import`/`export` | `require`/`module.exports` |
| Loading | Static, analyzed at parse time (enables tree-shaking) | Dynamic, runtime |
| Top-level `await` | Supported | Not supported |
| File extension (Node) | `.mjs` or `"type": "module"` | `.cjs` or default |
| `this` at module top level | `undefined` | `module.exports` |
| Browser support | Native | None (needs a bundler) |

Most modern bundlers (Vite, webpack, esbuild) let you write ESM syntax regardless of target.


---

## 16. ES6+ Collections: Map, Set, WeakMap, WeakSet

### Map — key-value pairs, keys can be ANY type (unlike plain objects, whose keys become strings)
```js
const map = new Map();
map.set("name", "Ana");
map.set(42, "numeric key");
map.set({ id: 1 }, "object key");

map.get("name");     // "Ana"
map.has("name");      // true
map.delete("name");    // true
map.size;               // number of entries

for (const [key, value] of map) console.log(key, value);
[...map.keys()];
[...map.values()];
[...map.entries()];

// Init from an array of pairs
const map2 = new Map([["a", 1], ["b", 2]]);
```
**Map vs Object:** use Map when keys aren't known ahead of time / aren't strings, when you need guaranteed insertion order + easy size, or when you're adding/removing keys frequently (Maps are optimized for this).

### Set — collection of unique values
```js
const set = new Set([1, 2, 2, 3]); // { 1, 2, 3 }
set.add(4);
set.has(2);       // true
set.delete(2);
set.size;           // 3

[...set];                    // convert to array
const unique = [...new Set(array)];  // dedupe an array — very common idiom
```

### WeakMap / WeakSet
Like Map/Set, but keys (WeakMap) or values (WeakSet) **must be objects**, and are held **weakly** — meaning they don't prevent garbage collection. Not iterable, no `.size`, no `.clear()`. Used for attaching private metadata to objects without causing memory leaks.

```js
const cache = new WeakMap();
function process(obj) {
  if (cache.has(obj)) return cache.get(obj);
  const result = expensiveComputation(obj);
  cache.set(obj, result);
  return result;
}
```


---

## 17. Symbols, Proxy & Reflect

### Symbols
A primitive type representing a **unique, immutable identifier**. Useful for avoiding property name collisions and defining "hidden" object protocols (like `Symbol.iterator`).

```js
const sym1 = Symbol("description");
const sym2 = Symbol("description");
sym1 === sym2; // false — always unique, even with the same description

const obj = {
  [sym1]: "hidden value", // won't show up in Object.keys / JSON.stringify / for...in
};

Symbol.for("shared");         // global symbol registry — SAME symbol if called again with same key
Symbol.for("shared") === Symbol.for("shared"); // true

// Well-known symbols customize built-in behavior:
Symbol.iterator;    // makes an object usable in for...of / spread
Symbol.asyncIterator;
Symbol.toPrimitive;  // customize how an object converts to a primitive
```

### Proxy
Wraps an object to intercept and customize fundamental operations (get, set, has, delete, etc.) — the basis of reactive libraries like Vue 3.

```js
const target = { message: "hello" };
const handler = {
  get(obj, prop) {
    console.log(`Getting ${prop}`);
    return prop in obj ? obj[prop] : `no such property: ${prop}`;
  },
  set(obj, prop, value) {
    console.log(`Setting ${prop} = ${value}`);
    obj[prop] = value;
    return true; // must return true to indicate success
  },
};
const proxy = new Proxy(target, handler);
proxy.message;         // logs "Getting message", returns "hello"
proxy.missing;          // returns "no such property: missing"
proxy.newProp = "x";     // logs "Setting newProp = x"
```

Common traps: `get`, `set`, `has`, `deleteProperty`, `ownKeys`, `apply` (for functions), `construct` (for `new`).

### Reflect
A built-in object providing methods that mirror the internal operations a Proxy can trap — usually used *inside* Proxy handlers to forward the default behavior.

```js
const handler = {
  get(target, prop, receiver) {
    console.log(`Accessed: ${prop}`);
    return Reflect.get(target, prop, receiver); // forward to default behavior
  },
};

Reflect.has(obj, "key");      // like `"key" in obj`
Reflect.ownKeys(obj);           // like Object.getOwnPropertyNames + symbols
Reflect.deleteProperty(obj, "key");
```


---

## 18. Regular Expressions

```js
const re1 = /pattern/flags;
const re2 = new RegExp("pattern", "flags"); // useful when the pattern is dynamic

// Flags
// g - global (find all matches)     i - case-insensitive
// m - multiline (^ and $ match per line)   s - dotAll (. matches newlines)
// u - unicode                          y - sticky
```

### Common Methods

```js
/hello/i.test("Hello world");         // true — does the pattern match?
"Hello world".match(/o/g);            // ["o", "o"] — all matches (with g flag)
"Hello world".matchAll(/o/g);          // iterator of match objects with capture groups + index
"Hello world".search(/world/);          // 6 — index of first match, or -1
"a,b;c".split(/[,;]/);                    // ["a", "b", "c"]
"2024-01-15".replace(/(\d+)-(\d+)-(\d+)/, "$3/$2/$1"); // "15/01/2024"
"2024-01-15".replace(/(\d+)-(\d+)-(\d+)/, (_, y, m, d) => `${d}/${m}/${y}`); // function replacer
```

### Common Patterns Cheat Sheet

| Pattern | Meaning |
|---|---|
| `.` | any character except newline |
| `\d` `\D` | digit / non-digit |
| `\w` `\W` | word char (`[A-Za-z0-9_]`) / non-word |
| `\s` `\S` | whitespace / non-whitespace |
| `^` `$` | start / end of string (or line, with `m` flag) |
| `*` `+` `?` | 0+, 1+, 0-or-1 of preceding token |
| `{n,m}` | between n and m repetitions |
| `[abc]` `[^abc]` | character class / negated class |
| `(...)` | capture group |
| `(?:...)` | non-capturing group |
| `(?<name>...)` | named capture group |
| `\|` | alternation (OR) |
| `(?=...)` `(?!...)` | lookahead / negative lookahead |
| `(?<=...)` `(?<!...)` | lookbehind / negative lookbehind |

```js
// Named capture groups
const match = "2024-01-15".match(/(?<year>\d+)-(?<month>\d+)-(?<day>\d+)/);
match.groups.year; // "2024"

// Practical examples
const emailRe = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
const isValidEmail = emailRe.test("user@example.com");

const slug = "Hello World!".toLowerCase().replace(/[^a-z0-9]+/g, "-").replace(/(^-|-$)/g, "");
// "hello-world"
```


---

## 19. Built-in Functions Grouped by Use Case

This section is your quick-lookup reference. Methods are grouped by **what you're trying to do**, not alphabetically.

### 19.1 String Methods

**Searching / checking**
```js
"hello world".includes("world");     // true
"hello world".startsWith("hello");    // true
"hello world".endsWith("world");       // true
"hello world".indexOf("o");             // 4 — first occurrence, -1 if not found
"hello world".lastIndexOf("o");          // 7 — last occurrence
```

**Extracting**
```js
"hello world".slice(0, 5);       // "hello" — supports negative indices
"hello world".substring(0, 5);    // "hello" — negative treated as 0, swaps if start>end
"hello world".charAt(0);           // "h"
"hello world"[0];                    // "h" — bracket access also works
"hello world".at(-1);                 // "d" — supports negative indices (ES2022)
```

**Transforming**
```js
"Hello".toUpperCase();               // "HELLO"
"Hello".toLowerCase();                // "hello"
"  hi  ".trim();                        // "hi"
"  hi  ".trimStart();                    // "hi  "
"  hi  ".trimEnd();                       // "  hi"
"5".padStart(3, "0");                      // "005"
"5".padEnd(3, "0");                         // "500"
"ab".repeat(3);                               // "ababab"
"Hello world".replace("world", "there");        // "Hello there" — first match only
"a-a-a".replaceAll("a", "b");                     // "b-b-b" — ES2021, all matches
```

**Splitting / joining**
```js
"a,b,c".split(",");                 // ["a", "b", "c"]
"hello".split("");                    // ["h","e","l","l","o"]
["a","b","c"].join("-");                // "a-b-c"
```

**Template literals & tagging**
```js
const name = "World";
`Hello, ${name}! 1 + 1 = ${1 + 1}`;   // multi-line strings, embedded expressions
function highlight(strings, ...values) {
  return strings.reduce((acc, str, i) => `${acc}${str}${values[i] ? `**${values[i]}**` : ""}`, "");
}
highlight`Name: ${name}`; // tagged templates — used by libraries like styled-components
```

**Comparing**
```js
"apple".localeCompare("banana"); // negative number (a comes before b)
```

**Conversion**
```js
String(123);            // "123"
(123).toString();         // "123"
(255).toString(16);        // "ff" — different radix
(3.14159).toFixed(2);       // "3.14" — fixed decimal places (returns a string!)
(3.14159).toPrecision(3);    // "3.14"
```

**Unicode / char codes**
```js
"A".charCodeAt(0);      // 65
String.fromCharCode(65);  // "A"
"😀".codePointAt(0);       // 128512 — needed for characters outside the BMP
```


### 19.2 Array Methods — Iteration (read-only, don't mutate)

```js
[1,2,3].forEach((val, i, arr) => console.log(val, i));    // just loops, returns undefined
[1,2,3].map(x => x * 2);                                      // [2,4,6] — transform each element
[1,2,3,4].filter(x => x % 2 === 0);                              // [2,4] — keep matching elements
[1,2,3,4].reduce((acc, x) => acc + x, 0);                          // 10 — fold into single value
[1,2,3,4].reduceRight((acc, x) => acc + x, 0);                       // same, right-to-left
[1,2,3].every(x => x > 0);                                             // true — ALL must pass
[1,2,3].some(x => x > 2);                                                // true — AT LEAST ONE
[1,[2,3],[4,[5]]].flat();                                                  // [1,2,3,4,[5]] — 1 level deep
[1,[2,3],[4,[5]]].flat(Infinity);                                           // [1,2,3,4,5] — fully flatten
[1,2,3].flatMap(x => [x, x * 2]);                                            // [1,2,2,4,3,6] — map + flatten
```

**`reduce` deep dive (the most powerful, most misunderstood array method):**
```js
// Sum
[1,2,3].reduce((sum, n) => sum + n, 0); // 6

// Group by property
const people = [{name:"A",age:20},{name:"B",age:30},{name:"C",age:20}];
const byAge = people.reduce((acc, p) => {
  (acc[p.age] ??= []).push(p);
  return acc;
}, {});
// { 20: [{...A}, {...C}], 30: [{...B}] }

// Count occurrences
["a","b","a","c","b","a"].reduce((acc, x) => {
  acc[x] = (acc[x] || 0) + 1;
  return acc;
}, {}); // { a: 3, b: 2, c: 1 }

// Build a pipeline (compose functions)
const pipeline = [x => x + 1, x => x * 2, x => x - 3];
pipeline.reduce((val, fn) => fn(val), 5); // ((5+1)*2)-3 = 9
```

### 19.3 Array Methods — Searching

```js
[1,2,3].includes(2);                    // true — checks for value existence
[1,2,3].indexOf(2);                      // 1 — first index, -1 if absent
[1,2,3].lastIndexOf(2);                    // last index
[1,2,3,4].find(x => x > 2);                  // 3 — first ELEMENT matching
[1,2,3,4].findIndex(x => x > 2);               // 2 — first INDEX matching
[1,2,3,4].findLast(x => x > 2);                  // 4 — last element matching (ES2023)
[1,2,3,4].findLastIndex(x => x > 2);               // 3 — last index matching (ES2023)
```

### 19.4 Array Methods — Transforming / Building (return NEW arrays, non-mutating)

```js
[1,2,3].slice(1);          // [2,3] — extract a portion, supports negative indices
[1,2,3].concat([4,5]);       // [1,2,3,4,5] — merge arrays
[3,1,2].toSorted();            // [1,2,3] — ES2023, non-mutating sort (vs .sort() which mutates)
[1,2,3].toReversed();            // [3,2,1] — ES2023, non-mutating reverse
[1,2,3].with(1, 99);                // [1,99,3] — ES2023, non-mutating index replace
Array.from({length: 5}, (_, i) => i * 2); // [0,2,4,6,8] — generate arrays
Array.of(1,2,3);                              // [1,2,3] — like Array() but no special single-number case
```

### 19.5 Array Methods — Mutating (change the array IN PLACE — use carefully!)

```js
const arr = [1,2,3];
arr.push(4);            // [1,2,3,4] — add to end, returns new length
arr.pop();               // [1,2,3] — remove from end, returns removed item
arr.unshift(0);           // [0,1,2,3] — add to start, returns new length
arr.shift();                // [1,2,3] — remove from start, returns removed item
arr.splice(1, 1, "x", "y"); // remove 1 item at index 1, insert "x","y" -> [1,"x","y",3]
arr.sort();                    // sorts IN PLACE, default is lexicographic (STRING) order!
arr.sort((a, b) => a - b);       // correct numeric ascending sort
arr.reverse();                     // reverses in place
arr.fill(0, 1, 3);                    // fill indices 1-2 with 0
arr.copyWithin(0, 3);                   // copies part of array to another location in itself
```

**⚠️ Common gotcha:** `[10, 1, 2].sort()` gives `[1, 10, 2]` because default sort converts to strings! Always pass a comparator for numbers: `.sort((a, b) => a - b)`.

### 19.6 Array Static Methods

```js
Array.isArray(value);                // true/false — the reliable array check
Array.from(iterable, mapFn?);         // convert iterable/array-like to array (with optional map)
Array.of(7);                            // [7] — unlike Array(7) which makes a length-7 empty array
```


### 19.7 Object Methods (Reference — full detail in Section 7)

```js
Object.keys(obj);                Object.values(obj);            Object.entries(obj);
Object.fromEntries(entries);      Object.assign(target, ...src);  { ...spread };
Object.freeze(obj);                Object.isFrozen(obj);
Object.seal(obj);                   Object.isSealed(obj);
Object.create(proto);                 Object.getPrototypeOf(obj);
Object.defineProperty(obj, key, descriptor);
Object.getOwnPropertyNames(obj);        Object.getOwnPropertyDescriptor(obj, key);
structuredClone(obj);                     // deep clone (native, modern runtimes)
```

### 19.8 Math

```js
Math.round(4.5);    // 5    (rounds .5 up)
Math.floor(4.9);      // 4    (rounds down)
Math.ceil(4.1);         // 5    (rounds up)
Math.trunc(4.9);          // 4    (removes decimal, no rounding)
Math.abs(-5);               // 5
Math.sign(-5);                // -1  (returns -1, 0, or 1)
Math.max(1,2,3);                // 3
Math.min(1,2,3);                  // 1
Math.max(...[4,1,9]);               // 9    (spread an array in)
Math.pow(2, 10);                      // 1024 (or use 2 ** 10)
Math.sqrt(16);                          // 4
Math.cbrt(27);                            // 3
Math.random();                              // random float in [0, 1)
Math.floor(Math.random() * 10);               // random integer 0-9
Math.PI;   Math.E;                              // constants
Math.log(x);  Math.log2(x);  Math.log10(x);        // logarithms
Math.sin(x); Math.cos(x); Math.tan(x);                // trig (radians)
Math.hypot(3, 4);                                        // 5 — sqrt(3² + 4²)
```

**Random integer in a range [min, max] inclusive:**
```js
function randomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}
```

### 19.9 Number

```js
Number.isInteger(4);          // true
Number.isFinite(Infinity);      // false
Number.isNaN(NaN);                // true — SAFER than global isNaN() (no coercion)
Number.parseInt("42px");            // 42 — same as global parseInt
Number.parseFloat("3.14m");           // 3.14
Number.MAX_SAFE_INTEGER;                // 9007199254740991
Number.MIN_SAFE_INTEGER;                  // -9007199254740991
Number.EPSILON;                             // smallest representable difference — for float comparison

// Safe float comparison (never use === for float math results!)
function nearlyEqual(a, b) { return Math.abs(a - b) < Number.EPSILON; }

(1234.5678).toFixed(2);       // "1234.57" (string!)
(1234).toLocaleString();        // "1,234" — locale-aware formatting
(0.1 + 0.2).toFixed(1);           // "0.3" — floating point precision workaround
```

### 19.10 Date

```js
const now = new Date();
new Date(2024, 0, 15);              // Jan 15, 2024 — NOTE: months are 0-indexed!
new Date("2024-01-15");                // parse ISO string
new Date(2024, 0, 15, 10, 30, 0);        // with time

now.getFullYear();   now.getMonth();    now.getDate();
now.getHours();        now.getMinutes();  now.getSeconds();
now.getDay();            // day of week, 0 = Sunday
now.getTime();             // milliseconds since epoch
Date.now();                  // current timestamp in ms (static, no `new` needed)

now.toISOString();             // "2024-01-15T10:30:00.000Z"
now.toLocaleDateString();        // locale-formatted date, e.g. "1/15/2024"
now.toLocaleTimeString();          // locale-formatted time
now.toLocaleString("en-US", { weekday: "long", year: "numeric", month: "long", day: "numeric" });

// Date arithmetic (dates are just numbers under the hood — ms since epoch)
const tomorrow = new Date(now);
tomorrow.setDate(now.getDate() + 1);

const diffMs = date2 - date1;                   // difference in milliseconds
const diffDays = diffMs / (1000 * 60 * 60 * 24);
```

**Modern alternative:** the `Temporal` API is coming to replace `Date`'s many quirks, but as of 2026 it's still landing across engines — for production code today, most projects use `Date` plus a library like `date-fns` or `Luxon` for anything nontrivial (timezones, formatting, durations).


### 19.11 JSON

```js
JSON.stringify(value);                          // -> string
JSON.stringify(value, null, 2);                    // pretty-print with 2-space indent
JSON.stringify(value, (key, val) => key === "password" ? undefined : val); // filter with replacer
JSON.parse(jsonString);                               // -> value
JSON.parse(jsonString, (key, val) =>
  key === "date" ? new Date(val) : val);              // revive with a reviver function
```

### 19.12 Global / Utility Functions

```js
parseInt("42", 10);        // 42 — always pass radix (base) explicitly!
parseFloat("3.14");          // 3.14
isNaN(value);                   // coerces first — prefer Number.isNaN()
isFinite(value);                  // coerces first — prefer Number.isFinite()
encodeURIComponent("a b&c");        // "a%20b%26c" — encode for use in a URL query/path segment
decodeURIComponent("a%20b%26c");      // "a b&c"
encodeURI("http://a.com/a b");           // encodes spaces but leaves URL structure chars alone
structuredClone(obj);                      // deep clone any structured-cloneable value
```

### 19.13 Console (debugging toolbox — most devs only know `.log`)

```js
console.log("info");
console.warn("warning");
console.error("error");
console.info("info");
console.table([{a:1,b:2},{a:3,b:4}]);    // renders a nice table — great for arrays of objects
console.group("Group label");
console.log("nested");
console.groupEnd();
console.time("label"); /* ...code... */ console.timeEnd("label"); // measures elapsed time
console.count("calls");                    // increments and logs a named counter
console.assert(x > 0, "x should be positive"); // logs only if the condition is FALSE
console.trace();                                 // prints a stack trace
console.dir(obj);                                  // logs a full property listing of an object
```

### 19.14 Timers & Scheduling

```js
setTimeout(fn, delay, ...args);       clearTimeout(id);
setInterval(fn, delay, ...args);        clearInterval(id);
queueMicrotask(fn);                        // schedule fn before the next macrotask
requestAnimationFrame(fn);                    // browser-only — sync with repaint, ~60fps
requestIdleCallback(fn);                        // browser-only — run when the browser is idle
```

### 19.15 Functional Utility Patterns (built from the above)

```js
// debounce — wait for a pause in calls before running (e.g., search-as-you-type)
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}

// throttle — run at most once per interval (e.g., scroll handlers)
function throttle(fn, limit) {
  let inThrottle = false;
  return (...args) => {
    if (!inThrottle) {
      fn(...args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}

// memoize — cache results of a pure function by its arguments
function memoize(fn) {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

// sleep (Promise-based delay for use with async/await)
const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));
await sleep(1000);
```


---

## 20. Browser & Runtime APIs (Quick Tour)

These aren't part of the JS language spec itself (they're host environment APIs), but you'll use them constantly.

### DOM (browser only)
```js
document.getElementById("id");
document.querySelector(".class");          // first match
document.querySelectorAll("div.item");       // NodeList of all matches (use .forEach or [...list])
document.createElement("div");

el.textContent = "text";        el.innerHTML = "<b>html</b>";
el.classList.add("active");     el.classList.remove("active");   el.classList.toggle("active");
el.setAttribute("data-id", "1"); el.getAttribute("data-id");
el.addEventListener("click", e => console.log(e.target));
el.removeEventListener("click", handler);
el.appendChild(child);              el.remove();
```

### Fetch API (browser + Node 18+)
```js
const res = await fetch("https://api.example.com/data", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ key: "value" }),
});
if (!res.ok) throw new Error(`HTTP ${res.status}`);
const data = await res.json();     // also: .text(), .blob(), .arrayBuffer()
```

### Storage (browser only)
```js
localStorage.setItem("key", JSON.stringify(value));   // persists across sessions
localStorage.getItem("key");
localStorage.removeItem("key");
sessionStorage.setItem(...);                              // cleared when tab closes
```

### Node.js Core Modules (quick reference)
```js
const fs = require("fs");                    // or: import fs from "node:fs";
fs.readFileSync("file.txt", "utf8");
fs.promises.readFile("file.txt", "utf8");        // Promise-based
const path = require("path");
path.join(__dirname, "file.txt");
const http = require("http");
process.env.NODE_ENV;                               // environment variables
process.argv;                                          // CLI arguments
```

> Note: this file is a JavaScript-language reference, not a browser/Node API manual — treat this section as pointers, and consult MDN (developer.mozilla.org) or the Node.js docs for exhaustive API details.


---

## 21. Using npm Effectively

npm (Node Package Manager) installs and manages JavaScript packages/dependencies. It ships automatically with Node.js. This section covers everything you need to use it well, not just the basic commands.

### 21.1 Setup & `package.json`

```bash
node -v          # check Node version
npm -v            # check npm version

npm init          # interactive prompts to create package.json
npm init -y        # accept all defaults, skip prompts
```

A `package.json` is your project's manifest:
```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "A sample app",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest",
    "build": "vite build"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.1"
  },
  "engines": { "node": ">=18" }
}
```

### 21.2 Installing Packages

```bash
npm install express            # or: npm i express — adds to "dependencies"
npm install express@4.17.0       # install a specific version
npm install express@latest         # install the latest published version
npm install -D nodemon               # or --save-dev — adds to "devDependencies" (build/test tools only)
npm install -g typescript              # global install — CLI tools available system-wide
npm install                              # (no package name) installs everything already listed in package.json
npm ci                                     # "clean install" — uses package-lock.json EXACTLY, deletes node_modules first. USE THIS IN CI/CD.
```

**`dependencies` vs `devDependencies`:** `dependencies` are needed to *run* the app in production (e.g., `express`, `react`). `devDependencies` are only needed while *developing* (e.g., `eslint`, `jest`, `webpack`, `nodemon`, TypeScript type definitions). This distinction matters because production deploys often run `npm install --omit=dev` (previously `--production`) to skip dev tools and keep the deploy small.

### 21.3 Semantic Versioning (semver) — read this carefully

Versions look like `MAJOR.MINOR.PATCH`, e.g. `4.18.2`:
- **MAJOR** — breaking changes
- **MINOR** — new features, backward-compatible
- **PATCH** — bug fixes, backward-compatible

**Version range symbols in `package.json`:**

| Symbol | Meaning | Example | Allows |
|---|---|---|---|
| `^4.18.2` | Caret — compatible with, allows MINOR & PATCH updates | `^4.18.2` | `4.18.2` up to (but not including) `5.0.0` |
| `~4.18.2` | Tilde — allows PATCH updates only | `~4.18.2` | `4.18.2` up to (but not including) `4.19.0` |
| `4.18.2` | Exact version, no updates | `4.18.2` | only that exact version |
| `>=4.18.2` | Greater than or equal | | any version 4.18.2 or newer |
| `*` or `latest` | Anything | | any version — risky, avoid in real projects |

`^` (caret) is the npm default and generally the right choice — it lets you get bug fixes and new features automatically while avoiding breaking changes (assuming the package author follows semver correctly, which isn't guaranteed).

### 21.4 `package-lock.json` — Don't Ignore This File

`package.json` specifies *ranges*; `package-lock.json` records the **exact** versions (including all nested dependencies) that were actually installed. This guarantees everyone on your team — and your CI/CD pipeline — gets identical dependency trees.

- **Always commit `package-lock.json` to git.**
- Never hand-edit it.
- Use `npm ci` (not `npm install`) in CI/CD and deployments — it's faster and fails if `package.json` and the lockfile are out of sync, catching drift early.

### 21.5 Running Scripts

Anything under `"scripts"` in `package.json` can be run with `npm run <name>`:
```bash
npm run dev
npm run build
npm test        # special case: "test", "start" don't need "run"
npm start
```

**Chaining and composing scripts:**
```json
{
  "scripts": {
    "clean": "rimraf dist",
    "build:css": "sass src/style.scss dist/style.css",
    "build:js": "esbuild src/index.js --bundle --outfile=dist/bundle.js",
    "build": "npm run clean && npm run build:css && npm run build:js",
    "prebuild": "echo 'runs automatically before build'",
    "postbuild": "echo 'runs automatically after build'"
  }
}
```
npm automatically runs `pre<script>` and `post<script>` hooks if they exist (e.g., `pretest`, `postinstall`) — a very common pattern for setup/teardown.


### 21.6 `npx` — Run Packages Without Installing Them Globally

```bash
npx create-react-app my-app         # runs a package's binary once, downloading temporarily if needed
npx cowsay "hello"                    # try a CLI tool without polluting global installs
npx -p typescript tsc --version        # specify a package explicitly
```
`npx` looks in local `node_modules/.bin` first, then falls back to a temporary download. This is why you should almost never `npm install -g` a project-specific build tool — use `npx` or a local devDependency + npm script instead, so version stays pinned per-project.

### 21.7 Updating, Removing, Auditing

```bash
npm outdated                  # shows which installed packages have newer versions available
npm update                     # updates packages within their allowed semver range (per package-lock)
npm update express               # update just one package
npm uninstall express              # remove a package (and its entry in package.json)
npm ls                                # list installed dependency tree
npm ls --depth=0                        # just top-level dependencies
npm audit                                 # scan for known security vulnerabilities
npm audit fix                               # attempt to auto-fix vulnerabilities (updates within semver range)
npm audit fix --force                         # allow breaking changes to fix vulnerabilities (use cautiously!)
npm dedupe                                      # simplify the dependency tree, remove duplicates
```

### 21.8 Inspecting Packages

```bash
npm view express                  # show metadata about a package (versions, deps, homepage, etc.)
npm view express versions            # list all published versions
npm search express                     # search the registry by keyword
npm docs express                         # opens the package's homepage/docs in a browser
npm repo express                           # opens the package's repository
```

### 21.9 Configuration

```bash
npm config list                    # show current config
npm config get registry              # e.g. https://registry.npmjs.org/
npm config set init-author-name "Your Name"

# Project-level: .npmrc file in project root
# save-exact=true       -> always install exact versions, no ^ or ~
# registry=https://my-private-registry.com/
```

### 21.10 `node_modules` Sanity & Common Fixes

The single most common troubleshooting step in the JS ecosystem:
```bash
rm -rf node_modules package-lock.json
npm install
```
This wipes and rebuilds the dependency tree from scratch — fixes a huge percentage of "works on my machine" and mysterious install errors.

```bash
npm cache clean --force     # clear npm's local cache if you suspect corruption (rarely needed)
```

### 21.11 Publishing Your Own Package

```bash
npm login                                    # authenticate with the registry
npm version patch                              # bump version (patch/minor/major) + creates a git tag
npm publish                                      # publish to the npm registry
npm publish --access public                        # required for scoped packages (@you/package) to be public
npm unpublish package@version                         # remove a version (restricted after 72 hours for safety)
```

A `.npmignore` file (or the `"files"` field in `package.json`) controls what gets published — keep published packages lean (exclude tests, source maps, docs source, CI config).

### 21.12 Workspaces (Monorepos)

npm supports monorepos natively (no extra tooling needed) since npm 7+:
```json
// root package.json
{
  "name": "my-monorepo",
  "workspaces": ["packages/*"]
}
```
```bash
npm install                          # installs deps for ALL workspaces, hoists shared deps to root
npm run build --workspace=packages/api     # run a script in one specific workspace
npm run build --workspaces                   # run a script across all workspaces
```

### 21.13 Practical Best Practices Checklist

- ✅ Always commit `package-lock.json`.
- ✅ Use `npm ci` in CI/CD pipelines, not `npm install`.
- ✅ Put build/test/lint tools in `devDependencies`, not `dependencies`.
- ✅ Run `npm audit` periodically; review before `--force` fixing.
- ✅ Pin exact versions (`save-exact` or explicit versions) for anything where reproducibility is critical (e.g., Docker base images, CI tool versions).
- ✅ Use `npx` instead of global installs for one-off or project-scoped CLI tools.
- ✅ Add a `.gitignore` with `node_modules/` — never commit it.
- ✅ Use `"engines"` in `package.json` to document required Node/npm versions.
- ❌ Don't hand-edit `package-lock.json`.
- ❌ Don't use `*` or unpinned `latest` for dependency versions in real projects.
- ❌ Don't install everything globally "just in case" — it causes version conflicts across projects.

### 21.14 npm vs Alternatives (yarn, pnpm)

| Tool | Notes |
|---|---|
| **npm** | Default, ships with Node, workspaces built in since v7 — good default choice today |
| **yarn** | Historically faster/stricter lockfiles; Yarn Berry (v2+) adds Plug'n'Play (no `node_modules`) |
| **pnpm** | Uses a global content-addressable store + symlinks — much faster installs, saves disk space, strict by default (prevents "phantom dependencies") |

All three read/understand `package.json` the same way — switching tools doesn't change your source code, just how dependencies get resolved/stored on disk. If starting fresh today, npm and pnpm are both excellent defaults; pick pnpm for large monorepos where disk/install speed matters most.


---

## 22. Best Practices & Common Pitfalls

### Best Practices
- Prefer `const`, then `let`; avoid `var`.
- Use `===`/`!==`, not `==`/`!=` (unless intentionally checking `== null`).
- Use strict mode (`"use strict"` or ES modules, which are strict by default).
- Favor pure functions and immutable updates (`{...obj, key: newVal}`) over in-place mutation, especially in UI frameworks (React, etc.) that rely on reference equality checks.
- Name things descriptively; keep functions small and single-purpose.
- Handle every Promise (`.catch()` or `try/catch` with `await`) — unhandled rejections crash Node processes in modern versions.
- Use `Array.isArray()` instead of `typeof x === "object"` to check for arrays.
- Use template literals instead of string concatenation.
- Destructure function parameters for clarity when there are many/optional options.

### Common Pitfalls

```js
// 1. Floating point precision
0.1 + 0.2 === 0.3;   // false! Use Math.abs(a-b) < Number.EPSILON instead

// 2. Default sort() is lexicographic
[10, 1, 2].sort();    // [1, 10, 2] — always pass a comparator for numbers

// 3. `this` inside a regular callback loses context
const obj = {
  name: "A",
  items: [1,2,3],
  logItems() {
    this.items.forEach(function (item) {
      console.log(this.name, item); // undefined! Regular function -> `this` isn't obj
    });
    this.items.forEach(item => console.log(this.name, item)); // fixed: arrow inherits `this`
  }
};

// 4. Mutating state directly (breaks referential equality checks in frameworks)
state.items.push(newItem);            // BAD — mutates in place
state.items = [...state.items, newItem]; // GOOD — creates a new array

// 5. NaN is never equal to itself
NaN === NaN;         // false
Number.isNaN(NaN);   // true — the correct check

// 6. Comparing objects/arrays with ==/===  compares REFERENCE, not contents
[1,2] === [1,2];     // false — different array instances
JSON.stringify([1,2]) === JSON.stringify([1,2]); // true, but fragile (key order matters for objects)

// 7. Closures in loops with `var`
for (var i = 0; i < 3; i++) { setTimeout(() => console.log(i), 0); } // 3, 3, 3
for (let i = 0; i < 3; i++) { setTimeout(() => console.log(i), 0); } // 0, 1, 2 — use let

// 8. Forgetting `await` — you get a Promise object, not the resolved value
async function get() { return fetchData(); }         // returns a Promise wrapping a Promise (still works via chaining, but confusing)
async function get() { return await fetchData(); }     // clearer intent

// 9. Modifying an array while iterating over it with forEach/map — leads to skipped elements
// Iterate over a copy, or use a plain for loop / filter, if you need to remove items during iteration.
```


---

## 23. Cheat Sheet Appendix

### Quick Method Chooser

| I want to... | Use |
|---|---|
| Transform every array item | `.map()` |
| Keep only some array items | `.filter()` |
| Reduce an array to one value | `.reduce()` |
| Find one matching item | `.find()` |
| Find the index of a matching item | `.findIndex()` |
| Check if any item matches | `.some()` |
| Check if all items match | `.every()` |
| Just loop, no return value needed | `.forEach()` |
| Check if array contains a value | `.includes()` |
| Remove duplicates | `[...new Set(arr)]` |
| Flatten nested arrays | `.flat(Infinity)` |
| Sort numbers correctly | `.sort((a,b) => a-b)` |
| Clone an array/object (shallow) | `[...arr]` / `{...obj}` |
| Clone deeply | `structuredClone(x)` |
| Merge objects | `{...a, ...b}` or `Object.assign({}, a, b)` |
| Loop over object keys | `Object.keys(obj)` or `for...in` |
| Loop over object key+value pairs | `Object.entries(obj)` |
| Convert array-like to array | `Array.from(x)` |
| Delay code / async wait | `await new Promise(r => setTimeout(r, ms))` |
| Run async tasks in parallel | `Promise.all([...])` |
| Safely access nested/possibly-missing props | `?.` optional chaining |
| Provide a fallback only for null/undefined | `??` nullish coalescing |

### Equality Quick Reference
```js
===   // strict equality — use this by default
==    // loose equality with coercion — avoid, except `x == null` (checks null OR undefined)
Object.is(NaN, NaN);   // true — like === but correctly handles NaN and -0 vs 0
```

### Truthy / Falsy
```
Falsy:  false, 0, -0, 0n, "", null, undefined, NaN
Truthy: everything else, including "0", "false", [], {}
```

### Common Array/String Index Tricks
```js
arr.at(-1);          // last element (also works on strings)
arr[arr.length - 1];  // last element, old-school way
```

---

## Where to Go Next

- **MDN Web Docs** (developer.mozilla.org) — the definitive JS/Web API reference.
- **javascript.info** — an excellent free, structured course covering everything in far more depth.
- **Node.js docs** (nodejs.org/docs) — for runtime-specific APIs (fs, http, streams, etc.).
- **TC39 proposals** (github.com/tc39/proposals) — to see what's coming to the language next.
- Practice: build small projects (a CLI tool, a REST API with Express, a small React app) — deliberate practice with real code cements this far better than reading alone.

*End of reference. Good luck studying!*

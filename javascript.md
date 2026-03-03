# 1. Execution Context & Call Stack

## Execution Context (EC)

An Execution Context is the environment where JavaScript code is
evaluated and executed.

It contains: - Variables - Functions - Scope references - `this` binding

JavaScript manages execution contexts using the Call Stack (Execution Context Stack).

## Types of Execution Context

### Global Execution Context (GEC)

- Created when script starts
- Only one exists
- In browsers: `globalThis` (equals `window`)
- In Node: `global`

### Function Execution Context (FEC)

- Created every time a function is invoked
- Each call gets its own isolated context

## Lifecycle of an Execution Context

Each context goes through two phases:

### Memory Creation Phase

- `var` → initialized as `undefined`
- `let` / `const` → allocated but uninitialized (TDZ)
- Function declarations → stored entirely

### Execution Phase

- Code runs line by line
- Variables get assigned real values
- Functions execute

## Components of an Execution Context

Internally, every context contains three vital components:

- Variable Environment: Stores var declarations and function arguments.
- Lexical Environment: Manages let/const and holds a Reference to the Outer Environment (the parent scope).
- This Binding: Determines the value of this based on how the function was called (e.g., as a method vs. a standalone function).

# 2. Hoisting & Temporal Dead Zone

## Hoisting|

It is the engine’s behavior of "lifting" declarations to the top of their scope during the Memory Creation Phase.

- `var` → hoisted and initialized to `undefined`
- `let` / `const` → hoisted but uninitialized
- Function declarations → fully hoisted

## Temporal Dead Zone (TDZ)

The period between start of scope and initialization of `let` / `const`.
Accessing during TDZ → ReferenceError.

# 3. Scope Chain & Closures

## Lexical Scoping

Scope is determined by where code is written, not where it is called.

## Scope Chain

1.  Check local scope
2.  Check outer lexical environment
3.  Continue upward
4.  Stop at global
5.  If not found → ReferenceError

### Rules

- Inner scopes can access outer variables
- Outer scopes cannot access inner variables
- Shadowing: inner variable overrides outer variable of same name

## Closures

A closure is a function that remembers its outer lexical environment
even after the outer function has finished executing.

### How It Works

- Inner function retains reference to parent scope
- As long as inner function exists, outer variables remain accessible
- Memory is preserved because references still exist

### Use Cases

- Data encapsulation
- React Hooks state
- Memoization
- Async callbacks

# 4. Asynchronous JavaScript & Event Loop

## Single Threaded Nature

JavaScript runs on one main thread.

## Concurrency vs Parallelism

- Concurrency → Switching between tasks (Event Loop)
- Parallelism → Multiple threads (Web Workers)

## Event Loop Algorithm

1.  Run synchronous code
2.  Flush ALL microtasks
3.  Browser may render
4.  Take ONE macrotask
5.  Repeat

## Microtasks

- Promise.then / catch / finally
- queueMicrotask
- MutationObserver
- process.nextTick (Node)

## Macrotasks

- setTimeout
- setInterval
- I/O
- UI events

---

# 5. Event Bubbling & Delegation

## Event Propagation Phases

1.  Capturing (top → down) : Event travels from the root (document) down to the target element
2.  Target : Event reaches the actual element that triggered it.
3.  Bubbling (bottom → up) : Event travels back up from the target to the root.

Root → … → Parent → Target → … → Parent → Root

## Event Bubbling

```js
addEventListener() defaults to { capture: false }
```

Events propagate upward to ancestors by default.
If a child inside parent is clicked → parent handler also runs.

## Stop Bubbling

Stops the event from moving upward.

```js
event.stopPropagation();
```

## Event Delegation

Event Delegation is a pattern where:

Instead of attaching multiple listeners to child elements,
you attach one listener to a common parent.

It works because of event bubbling.

Benefits: - Better memory usage - Works for dynamic elements - Cleaner
architecture

# 6. Browser Rendering Pipeline

Frame (\~16ms at 60fps):

1.  Run 1 macrotask
2.  Drain microtasks
3.  Run requestAnimationFrame
4.  Style calculation ( Browser calculates: CSS rules, computed styles, What changed?)
5.  Layout (Reflow) ( Browser calculates: Element sizes, positions, geometry )
6.  Paint ( Browser draws: backgrounds, text, borders, shadows ) still not on screen yet
7.  Composite ( GPU combines layers and then the final image appears on screen )
8.  Repeat

Pixel Pipeline: JS → Style → Layout → Paint → Composite

If JS blocks too long → frame drops.

# 7. Rendering Optimization

## Minimize Reflow

Avoid changing layout-triggering properties: - width - height -
font-size

## Minimize Repaint

Avoid frequent visual-only changes: - color - background

## Avoid Layout Thrashing

Batch reads and writes separately.

Bad:

```js
for (...) {
  element.style.width = box.offsetWidth + "px";
}
```

Good:

```js
const width = box.offsetWidth;
for (...) {
  element.style.width = width + "px";
}
```

## requestAnimationFrame

Syncs animations with browser refresh rate.

# 8. Prototypal Inheritance

## \[\[Prototype\]\]

Internal link to another object.

Access via: - `__proto__` - `Object.getPrototypeOf()`

## Prototype Chain

When accessing obj.prop:

1.  Check object
2.  Check prototype
3.  Continue upward, Check prototype’s prototype
4.  Stops when prototype is `null`

## prototype vs **proto**

| prototype                    | proto                |
| ---------------------------- | -------------------- |
| Property on functions        | Property on objects  |
| Used when creating instances | Actual internal link |
| Shared by all instances      | Specific to object   |

## Linking Objects

### Recommended

```js
const child = Object.create(parent);
```

### Legacy

```js
child.__proto__ = parent;
```

### Constructor Example

```js
function Person(name) {
  this.name = name;
}

Person.prototype.species = "Human";

const john = new Person("John");

console.log(john.species); // "Human"
console.log(john.__proto__ === Person.prototype); // true
```

# 9. Arrow vs Regular Functions

Arrow Functions: - No own `this` - No `prototype` - Cannot use `new` -
No own `arguments`

# 10. Shallow vs Deep Copy

## Shallow Copy

Copies top-level properties only.

```js
Object.assign({}, obj)
{ ...obj }
```

## Deep Copy

Creates fully independent clone.

```js
structuredClone(obj);
```

Alternative:

```js
JSON.parse(JSON.stringify(obj));
```

Limitations: - Fails for Date, Functions, undefined, Map, Set

# 11. Functional Programming

## Pure Functions

- Same input → same output
- No side effects

## First-Class Functions

Functions can be: - Stored - Passed - Returned
or just say they can be used as values

## Higher-Order Functions

A function that either takes a function as argument or returns a function

# 12. Performance Optimization

## Debounce

Executes after activity stops.

## Throttle

Executes at fixed interval.

## Memoization

Cache expensive results.

## Code Splitting

Break bundle into smaller chunks.

## Lazy Loading

Load resources only when needed.

## Tree Shaking

Remove unused exports during build. Requires ES Modules (`import` /
`export`).

## Minification

Removes unnecessary characters from code without changing its functionality.

---

2. Design Patterns
Singleton
Factory
Observer
Module pattern
3. Currying vs Partial Application
4. Map vs WeakMap
WeakMap keys are garbage collectible.
Callback hell, promises, async await
Implement Promise from scratch
Polyfill bind()
How to optimize large-scale JS apps
How to prevent memory leaks
Explain microfrontend architecture
What happens when you type a URL in browser?
Modular code structure
Dependency inversion
Clean architecture principles
State management patterns
SSR vs CSR
Microservices + BFF layer
Web security (XSS, CSRF, CORS)
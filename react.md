## What is React?

React is a JavaScript library for building user interfaces using **reusable components**.

React is built around two key ideas:

1. **Unidirectional Data Flow**: Data in React flows in **one direction only**
2. **Component-Based Architecture**: React applications are built by breaking the UI into **small, reusable, independent components**

## What is JSX (JavaScript XML)?

JSX is a syntax extension for JavaScript that lets you **write HTML-like code inside JavaScript**.

- It is syntactic sugar for `React.createElement`
- JSX gets compiled into `React.createElement()` calls. 
- It ultimately produces **React Elements** (plain JavaScript objects)

JSX is compiled by a **JavaScript compiler (transpiler)** — most commonly **Babel**.

### Example:
```jsx
<button className="btn">Click</button>
```

Behind the scenes:

```js
React.createElement('button', { className: 'btn' }, 'Click');

// Output (React Element)
{
  type: 'button',
  props: {
    className: 'btn',
    children: 'Click'
  }
}
```


## What is a React Element?

A **React Element** is a **plain immutable JavaScript object** that describes the UI.

### Structure of an Element

A React element has two main properties:

1. **type**: could be **DOM element** or **Component**
2. **props**: could be **attributes** (className, id, etc.) & **Children** (nested elements or text)

```js
{
  type: 'button' // string | Component,
  props: {
    // attributes + children
    className: 'btn',
    children: 'Click Me'
  }
}

// equivalent jsx
<button className="btn">Click Me</button>
```

## What is Virtual DOM (VDOM)?

The **Virtual DOM** is a **lightweight JavaScript representation of the real DOM**. 

## What is Reconciliation?

Reconciliation is the process React **uses to determine what changes are needed to update the UI efficiently**.
It works by **comparing the new Virtual DOM tree with the previous one (diffing)** and calculating the **minimal set of updates required**.
Reconciliation starts when state or props change (e.g., via `setState` or state updates in hooks).

By the end of this process:

- React knows what has changed
- A renderer like `react-dom` or `react-native` applies the minimal updates to the real DOM (or native views)

### Diffing Algorithm

The diffing algorithm is React’s O(n) heuristic **used during reconciliation to compare two trees efficiently**.

It is based on these assumptions:

1. Different element types produce different trees 
2. Same type → reuse instance and update props
3. Children are matched by position by default
4. Keys override position and provide stable identity
5. Comparison is done top-down (depth-first traversal)

## What is Shadow DOM?

Shadow DOM is a browser feature that lets you **attach a hidden, isolated DOM tree** to an element.

# React Batching

Batching is when React groups multiple state updates into a single re-render for better performance.

---

## Before (No Automatic Batching)

Previously, React only batched updates inside **React event handlers**. Updates inside `setTimeout`, promises, or native event handlers were **not batched**.

```js
// Before: only React events were batched.
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will render twice (no batching)
}, 1000);
```

---

## After (Automatic Batching)

With automatic batching, updates inside:

- setTimeout
- Promises
- Native event handlers
- Any async code

are now **batched automatically**.

```js
// After: updates are batched automatically.
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // React will re-render only once (batched)
}, 1000);
```

---

## Opting Out of Batching

You can force React to flush updates immediately using `flushSync`.

```js
import { flushSync } from "react-dom";

flushSync(() => {
  setCount(c => c + 1);
});

// DOM is updated immediately here
```

---

📒 React Fiber Architecture: Deep Dive Notes
1. The Fiber Node Structure
The "Fiber" is a plain JavaScript object representing a unit of work. It is both a pointer in a tree and a stack frame for a component.
Navigation Pointers:
child: Points to the first direct child.
sibling: Points to the next node at the same level.
return: Points to the parent (where the program returns after processing the fiber).
Identification:
type: The component (function/class) or the Host Component string (e.g., 'div').
key: Unique identifier used to optimize list diffing.
The "Double Buffer" Link:
alternate: The "Mirror" node. If this is the Current fiber, the alternate is the Work-in-Progress (WIP) fiber, and vice versa.
1. Prop Management & Optimization
React uses two specific fields to decide if it can "bail out" (skip) a component to save CPU cycles.
pendingProps: The incoming data. These are the props provided by the parent or the new state during the current render pass.
memoizedProps: The "Last Known Good" data. These are the props used in the last successfully committed render.
The Optimization Logic (beginWork phase):
If (pendingProps === memoizedProps) (Shallow Comparison) + No State Change:
→ Bailout! React reuses the existing output and skips the component’s children.
1. The Rendering Lifecycle
Phase 1: Render / Reconciliation (Asynchronous & Interruptible)
React works on the WIP Tree.
It traverses the tree using beginWork.
If a change is detected, it "marks" the fiber with an effect tag (e.g., Placement, Update).
Priority: Because this phase is asynchronous, React can pause work on a low-priority WIP tree to handle a high-priority event (like a keystroke).
Phase 2: Complete Work
As React climbs back up the tree (completeWork), it builds the physical DOM nodes for Host Components.
It prepares the "Effect List" (a linked list of all fibers that actually need changes).
Phase 3: Commit Phase (Synchronous & Uninterruptible)
Flushing: React takes the finished WIP tree and "flushes" the changes to the DOM in one single, synchronous burst.
The Switch: React updates the root pointer. The WIP Tree now becomes the Current Tree.
Clean up: memoizedProps are updated to match the pendingProps.
1. Why "Double Buffering" Matters
No "Tearing": Users never see a partial UI (e.g., a header updates but the footer doesn't).
Memory Efficiency: Instead of destroying and recreating objects, React simply recycles the alternate node, swapping data between the two.

---

📒 React Fiber: Incremental Rendering & The Virtual Stack
1. The Problem: The "Sync" Call Stack
Before Fiber, React used a recursive rendering model.
The Stack: When a function calls another, a "stack frame" is added. The engine cannot stop until the stack is empty.
The Issue: If you have 10,000 components, the browser's Main Thread stays blocked for a long time. The user can't click, scroll, or type because the "stack" is busy.
2. The Solution: Fiber as a "Virtual Stack Frame"
Fiber is a re-implementation of the stack specifically for components.
Manual Control: Unlike the built-in JS stack, React can manually pause a Fiber, save its state, and come back to it later.
Unit of Work: One component = One Fiber = One "frame" of work.
Work Loop: React runs a workLoop that constantly checks: "Do I have enough time left in this frame (16ms) to do more work? If not, pause and let the browser paint."
3. Incremental Rendering & Scheduling
This is the ability to split rendering into chunks. It introduces Priority-Based Updates:
Priority Level	Example Task	Behavior
Immediate/High	Typing in an input, clicking a button.	Interrupts current low-priority work to keep UI snappy.
Normal	Fetching data, list rendering.	Standard updates.
Low/Offscreen	Pre-rendering a hidden tab or analytics.	Only happens when the CPU is idle.
Abort/Reuse: If a user types "A" then "B" very quickly, React can abort the render for "A" mid-way and start working on "B" to stay up to date.
4. The Two-Phase Separation (The "Architecture")
React splits the process into two distinct phases to enable cross-platform support.
Phase 1: Reconciliation (Render Phase) — The Brain
Nature: Asynchronous & Interruptible.
Action: Compares the old tree vs. the new tree (Diffing).
Output: A list of "Effects" (changes needed).
Consistency: The same logic is used for Web, Mobile, and VR.
Phase 2: Rendering (Commit Phase) — The Muscles
Nature: Synchronous & Uninterruptible (Must be fast to prevent flickering).
Action: Takes the list of effects and physically applies them to the host environment.
Platform Specifics:
React DOM: Injects div, span, etc.
React Native: Calls Objective-C or Java to create UIView or android.view.
React Three Fiber: Updates Three.js objects (meshes, geometries).

---

In standard JavaScript, when you call a function, the engine creates a Stack Frame. 
The Problem (The "Stack" is a Prude):
The JS Call Stack is synchronous and recursive. If ComponentA calls ComponentB, which calls ComponentC, the stack looks like this:
[Frame: ComponentC] <— Active
[Frame: ComponentB]
[Frame: ComponentA]
The Catch: You cannot "pause" the stack. If ComponentC takes 200ms to calculate, the browser is frozen. It cannot respond to a user clicking "Cancel" because it is stuck at the top of that stack.

A Fiber is a Virtual Stack Frame. React decided: "The built-in JS stack is too rigid. We will build our own 'stack' using objects on the heap."
Because a Fiber is just a JavaScript Object, React can:
Save it: Keep the object in memory and stop execution.
Move it: Put it to the side and work on a "Higher Priority" Fiber.
Discard it: If a user types a new character, React can literally throw away the "Work-in-Progress" Fiber for the old character and start a new one.

1. How Component Rendering Relates to the Stack
When you write <MyComponent />, React treats it as a unit of work.

Feature	Standard Rendering (Pre-Fiber)	                    Fiber Rendering (Modern)
- Logic	Uses the JS Call Stack.	                            Uses the Fiber Tree (Virtual Stack).
- Flow	Recursive (Depth-first).	                          Loop-based (Iterative).
- Interruption	Impossible. The thread is blocked.	        Possible. React checks `shouldYield()` every few ms.
- State	Lost if the function crashes/stops.	                Persisted in the Fiber's memoizedState.

How React walks this "Stack":
Go Down: It follows child pointers until it hits a leaf (like a `div`).
Go Sideways: It checks for a sibling.
Go Up: If no sibling exists, it follows the return pointer back to the parent.
Why this navigation is genius:
In a normal stack, "returning" is automatic. In Fiber, "returning" is a manual pointer. This allows React to stay in a while loop:
```js
while (workInProgress !== null && !shouldYield()) {
  workInProgress = performUnitOfWork(workInProgress);
}
```
If `shouldYield()` is true (e.g., 5ms have passed), the loop breaks. The workInProgress pointer stays exactly where it was. When the browser is idle again, the loop restarts.

---

📒 React Fiber: The Lanes System
What are Lanes? Lanes are React’s modern Priority System (introduced in React 17). They replaced the old `pendingWorkPriority` (which used simple numbers) with a 31-bit bitmask.
The Bitmask (Lanes) is just the "Project Management" layer. It’s the metadata that tells React what needs to be done and how fast to do it. It is not the actual code, the component, or the DOM change itself.
The Goal: To enable Concurrent React, allowing multiple updates to exist at the same time without blocking or "clobbering" each other.
2. Why the change? (Numbers vs. Bits)
The Old Way (Numeric Priority): A component had a single number for priority. If a "High Priority" update came in while a "Low Priority" update was running, React would struggle to track both. It was usually "one or the other."
The New Way (Lanes): A Fiber has 31 lanes (bits). Think of these as 31 checkboxes. A component can have multiple boxes checked at once, meaning it can track multiple types of work simultaneously.
1. Real-World Example: The Search Bar
This is the perfect example to use in an interview to show how Lanes handle overlapping updates.
Scenario: You have an input field and a massive list of results below it.
Step 1: You type the letter "A". React assigns a SyncLane to update the text box and a TransitionLane to filter the big list.
Step 2: React starts the heavy work of filtering the list (TransitionLane).
Step 3 (The Overlap): While the list is still rendering, you type "B".
Step 4 (Bitmasking at work): Because Lanes are bits, React’s internal state for that component now looks like: 0b00001001 (Both the Sync bit and the Transition bit are "checked").
Step 5: React sees the SyncLane bit is active, pauses the list rendering, and updates the text box so the typing feels instant.
Step 6: Once the "B" is shown, React looks at its "checkboxes," sees the TransitionLane is still checked, and finishes the list filtering.
1. Key Technical Terms (For Deep Dives)
Bitwise Math: React uses operators like & (AND) and | (OR) to check priorities. This is O(1) speed — much faster than searching an array or list of tasks.
Lanes Entanglement: React can "link" two different lanes together if they need to finish at the exact same time (e.g., a header and footer updating together).
Starvation Prevention: If a "Slow Lane" is ignored for too long because of constant "Express" traffic, React will expire that lane and force it to finish (promoting it to High Priority).
childLanes: A property on Fiber nodes that "bubbles up" the priority of children. It allows React to skip entire subtrees if their childLanes bit is 0 (meaning no work is pending inside).

2. Common Lane Priority Levels
React categorizes work into several key buckets: 
Lane Name           Priority	Typical Use Case
SyncLane            Highest 	Discrete user events like clicks or keyboard presses that need an immediate response.
InputContinuousLane	High	    Constant user feedback like scrolling, dragging, or mouse movements.
DefaultLane	        Normal  	Standard setState updates not wrapped in any specific priority.
TransitionLanes	    Low	      Updates wrapped in startTransition or useTransition. These are interruptible by higher lanes.
IdleLane	          Lowest  	Background tasks like pre-fetching analytics or offscreen content.

---

1. The Core Concept: "The Two Trees"
React Fiber doesn't just have one tree; it maintains two at all times. They are linked together via the .alternate property. 
The Current Tree: This represents the UI that is currently on the screen. It is "flushed" and visible to the user.
The Work-in-Progress (WIP) Tree: This is the "draft" tree where React prepares the next state. It is built in the background and is not visible to the user. 
2. Why Use Double Buffering?
Without it, React would have to update the live DOM as it goes. This causes two major problems:
Tearing (Partial UI): If React updates the "Header" but gets interrupted before updating the "Footer," the user sees an inconsistent, "torn" UI.
Responsiveness: If React is busy updating the live tree, it can't easily "pause" to handle a user click without leaving the UI in a broken, half-finished state.
3. The "Switch" Mechanism
The transition from the "Draft" to the "Live" tree happens in two main steps:
Step 1: Cloning (Reconciliation Phase): When an update starts, React walks through the Current tree and clones nodes into the WIP tree. It reuses the old objects where possible (using the alternate pointer) to save memory.
Step 2: The Commit (Commit Phase): Once the WIP tree is 100% finished and ready, React simply swaps a pointer. The "Root" of the app, which used to point to the Current tree, now points to the WIP tree.
Instantly, the WIP tree becomes the Current tree.
The old Current tree is now ready to be reused for the next update. 
4. Real-World Analogy: The Painting Gallery
Imagine a museum gallery with a painting on the wall.
Current Tree: The painting currently hanging on the wall for visitors to see.
WIP Tree: An artist is in the back room painting a new version of that painting.
The Switch: Only when the artist is completely finished does the museum swap the paintings. Visitors never see the artist halfway through a brushstroke; they only see the old version or the new, finished version.
5. Interview "Deep Dive" Points
Memory Efficiency: React doesn't create two entirely new trees from scratch every time. It recycles the Fiber objects. The alternate of the Current Fiber is the WIP Fiber. They swap roles back and forth.
Interruptibility: Because work happens on a "Draft" (WIP) tree, React can throw it away if a higher-priority update comes in. If the user types a new character before the "draft" for the last character is finished, React just resets the WIP tree and starts over with the new data.


---  

## The rules of hooks
There are two main usage rules the React core team stipulates you need to follow to use hooks which they outline in the hooks proposal documentation.

1. **Don’t call** Hooks inside **loops, conditions, or nested functions**. Instead, always use Hooks at the **top level** of your React function.
2. Only Call Hooks from **React Functions**

---  

## useState
`useState` lets you add a state variable to your component.

```js
const [state, setState] = useState(initialState)
```

### The Nature of State
  - **Memory**: State is a component's private memory that persists across renders.
  - **Isolation**: State is local to a specific component instance.
  - **Initialization**: * Lazy Initialization: Use `useState(fn)` instead of `useState(fn())` for expensive calculations. React saves the initial state once and ignores it on the next renders..
  - **Stability**: The set function identity is stable; it won't change between renders, making it safe to omit from useEffect dependencies.

### Principles for structuring state 
| Principle            | Problem                                    | Solution                                                 |
| -------------------- | ------------------------------------------ | -------------------------------------------------------- |
| Group Related State  | Updating `x` and `y` always together       | Merge into one object: `setPos({ x, y })`                |
| Avoid Contradictions | `isSending` and `isSent` both being true   | Use a status string: `'typing' \| 'sending' \| 'sent'`   |
| Avoid Redundancy     | `fullName` derived from `first` and `last` | Calculate during render: `const fullName = first + last` |
| Avoid Duplication    | Storing a whole object in `selectedItem`   | Store only the id or index                               |
| Flatten State        | Deeply nested objects are hard to update   | Normalize your data (like a database)                    |


### Updates & Immutability
In React, you treat state as **immutable**!. When you store objects in state, mutating them will not trigger renders. So never mutate, **always create a new reference**. 
**Deep Nesting**: Use **Immer**. It allows you to write "mutating" code on a **draft proxy**, which Immer then converts into a clean, immutable update. 

### Internal Mechanics (The "Array" Secret)

Note: in real React, each component instance has its own hooks storage
Constraint: This code illustrates why you cannot change the number or order of Hooks—the pointer would point to the wrong index in the array.

```js
// assume for a single component
let componentHooks = [];
let currentHookIndex = 0;

// How useState works inside React (simplified).
function useState(initialState) {
  let pair = componentHooks[currentHookIndex];
  if (pair) {
    // This is not the first render,
    // so the state pair already exists.
    // Return it and prepare for next Hook call.
    currentHookIndex++;
    return pair;
  }

  // This is the first time we're rendering,
  // so create a state pair and store it.
  pair = [initialState, setState];

  function setState(nextState) {
    // When the user requests a state change,
    // put the new value into the pair.
    pair[0] = nextState;
    updateDOM();
  }

  // Store the pair for future renders
  // and prepare for the next Hook call.
  componentHooks[currentHookIndex] = pair;
  currentHookIndex++;
  return pair;
}
```

---

## State & Render Tree Position

State is tied to a **position in the render tree**, NOT the component itself.

### ✅ State Preservation Rule

* Same component
* Same position

👉 State is preserved

### Example

```jsx id="z9r0z9"
{isFancy ? (
  <Counter isFancy={true} />
) : (
  <Counter isFancy={false} />
)}
```

👉 Result:

* Same component at same position
* State is preserved ✅

---

## useReducer

`useReducer` lets you add a `reducer` to your component.

```js
function reducer(state, action) {
  // ...
}

const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

- **reducer**: The reducer function that **specifies how the state gets updated**. It must be **pure**, should take the **state** and **action** as arguments, and should **return the next state**.
- **initialArg**: The value from which the initial state is calculated. How the initial state is calculated from it depends on the next init argument.
- **optional init**: The initializer function that should return the initial state. If it’s not specified, the initial state is set to initialArg. Otherwise, the initial state is set to the result of calling init(initialArg).

### dispatch function

The `dispatch` function returned by `useReducer` lets you update the state to a different value and trigger a re-render. You need to pass the action as the only argument to the dispatch function
While you can pass anything to dispatch, the community standard is the Flux Standard Action (FSA) pattern:

- `type`: A string that describes the event.
- `payload`: The data needed to perform the update.

```js
dispatch({
  type: "changed_name",
  payload: "Alice",
});
```

### When to choose useReducer over useState

While `useState` is great for simple values, `useReducer` shines in these scenarios:

- **Complex State**: When your state is a nested object or array.
- **Related Logic**: When updating one piece of state depends on the value of another.
- **Predictability**: It moves the "how" of the state update out of the event handlers and into a single, pure function.

### The "Initial Init" Pattern

You mentioned the **optional init function**. This is specifically **useful for lazy initialization**. If you calculate your initial state based on an expensive computation, putting it in init ensures it only runs once during the initial mount, rather than on every render.

```js
function createInitialState(username) {
  // Imagine a heavy computation here
  return { name: username, points: 0 };
}

// Inside your component
const [state, dispatch] = useReducer(
  reducer,
  props.username,
  createInitialState,
);
```

### Avoiding the "Stale State" Trap

Because the `reducer` is a pure function, it always receives the current state as its first argument. This makes it much safer than `useState` when multiple updates are fired rapidly, as you don't have to worry about closures capturing an old version of the state.

### Pitfall

- State is read-only. Don’t modify any objects or arrays in state. Instead, always return new objects from your reducer:

```js
function reducer(state, action) {
  switch (action.type) {
    case "incremented_age": {
      // ✅ Instead, return a new object
      return {
        ...state,
        age: state.age + 1,
      };
    }
  }
}
```
---

## useMemo
`useMemo` lets you **cache the result of a **calculation** between re-renders.

```js
const cachedValue = useMemo(calculateValue, dependencies)
```

- React will **compare each dependency** with its previous value using the **Object.is comparison**.
- **React Compiler** automatically memoizes values and functions, reducing the need for manual `useMemo` & `useCallback` calls. You can use the compiler to handle memoization automatically.

---

## useCallback 
`useCallback` lets you **cache a function definition** between re-renders.

```js
const cachedFn = useCallback(fn, dependencies)
```

- React will **compare each dependency** with its previous value using the **Object.is comparison**.
- If you’re writing a **custom Hook**, it’s recommended to **wrap any functions that it returns into useCallback**
- **React Compiler** automatically memoizes values and functions, reducing the need for manual `useMemo` & `useCallback` calls. You can use the compiler to handle memoization automatically.

---

## useTransition 

"A **Transition** is a way to **mark a state update as 'non-urgent'**. In React, **updates are urgent by default (like typing in an input)**. By **wrapping a slow update** in `startTransition`, you tell React: **'Feel free to interrupt this if a more important task comes along.'**" 

### How it Works with Fiber (The Logic)
Transitions are the "Killer App" for the Lanes and Double Buffering system:
#### The Split: 
When you use `startTransition`, React actually performs two updates.
1. **High-Priority (SyncLane)**: React updates any immediate UI (like an isPending spinner).
2. **Low-Priority (TransitionLane)**: React starts building a Work-in-Progress (WIP) tree for the heavy update in a background lane.
#### The Interruption: 
If a user clicks or types while the WIP tree is being built, Fiber sees the SyncLane bit flip. It pauses or discards the transition work, handles the user input, and then restarts the transition with the newest state. 

**Real-World Example: The Tab Switcher**
Imagine a dashboard with three tabs: Home, Profile, and Massive Reports. 
**The Problem**: Without transitions, clicking "Massive Reports" freezes the whole app for 1 second while it renders 500 charts. If you accidentally clicked it and want to switch back to "Home" immediately, you can't—the UI is locked.
**The Solution**: Wrap the tab switch in a transition. 
const [isPending, startTransition] = useTransition();
const [tab, setTab] = useState('home');

```js
function selectTab(nextTab) {
  // Marking the heavy tab-switch as a transition
  startTransition(() => {
    setTab(nextTab);
  });
}

return (
  <div style={{ opacity: isPending ? 0.7 : 1 }}>
    <TabButton onClick={() => selectTab('home')}>Home</TabButton>
    <TabButton onClick={() => selectTab('reports')}>Reports</TabButton>
    
    {/* If 'reports' is slow, the UI doesn't freeze! */}
    {tab === 'reports' ? <HeavyReports /> : <Home />}
  </div>
);

```
---

## useDeferredValue
`useDeferredValue` is a hook that **takes a value and returns a 'deferred' version of it**. When the original value updates quickly (like a user typing), React will first render the UI with the old value to keep things responsive, and then—in the background—it will work on a new render with the new value.

### How it works with Fiber (Under the Hood)
This hook is a direct application of the **Double Buffering** and **Lanes** concepts: 
**Lane Switching**: When the original value changes, React treats the immediate update (e.g., updating an input field) as a SyncLane (High Priority).
**The Background Pass**: React then schedules a separate render for the deferred value in a TransitionLane (Low Priority).
**Interruptible Rendering**: Because the deferred render is in a low-priority lane, React can interrupt it. If the user types another character while React is halfway through rendering a huge list with the "old" deferred value, React throws away that work and starts fresh with the latest character.

###  Why use this instead of Debouncing/Throttling?
**Fixed Delays**: Debouncing uses a fixed timer (e.g., 300ms). If your computer is fast, you're waiting 300ms for no reason. If it's slow, 300ms might not be enough.
**Adaptability**: `useDeferredValue` has no fixed delay. It is "as fast as possible." On a powerful PC, the deferred update might happen in 10ms. On a slow phone, it might take 500ms. React adjusts based on the hardware

### Example 

**The Scenario**: Imagine a search bar where each keystroke filters a huge array. Without useDeferredValue, every letter you type triggers a heavy render, making the typing feel "laggy" or "stuck."

```js
import { useState, useDeferredValue, memo } from 'react';

function App() {
  const [query, setQuery] = useState("");
  
  // 1. Create a "lagging" version of the query
  const deferredQuery = useDeferredValue(query);

  return (
    <div>
      {/* 2. The Input uses the REAL query (High Priority) */}
      <input 
        value={query} 
        onChange={(e) => setQuery(e.target.value)} 
        placeholder="Type to search..."
      />

      {/* 3. The List uses the DEFERRED query (Low Priority) */}
      <SlowList text={deferredQuery} />
    </div>
  );
}

// 4. Wrap the slow component in memo!
// This tells React: "Only re-render if deferredQuery actually changes."
const SlowList = memo(({ text }) => {
  console.log(`Rendering list for: ${text}`);
  
  // Artificial delay: Imagine filtering 10,000 items here
  const items = [];
  for (let i = 0; i < 250; i++) {
    items.push(<li key={i}>Result for {text} #{i}</li>);
  }

  return <ul>{items}</ul>;
});
```

### How this works in the Fiber Tree (The "Why")
**High Priority Render:** When you type "A", setQuery("A") happens immediately. React renders the <input> with "A". Because deferredQuery hasn't updated yet (it's deferred), the SlowList sees the old value and memo tells it to skip rendering.
**Result:** The typing is instant and smooth.
**Low Priority Render (Background):** Once the input is painted on the screen, React starts a second background render pass. In this pass, it updates deferredQuery to "A".
**Interrupting:** If you type "B" while the background render for "A" is still happening:
React aborts the work for "A".
It handles the SyncLane for "B" in the input box.
It then starts a new background pass for "B".

---

## useEffect

`useEffect`lets you synchronize a component with an external system (like a chat service).

Here, external system means any piece of code that’s not controlled by React, such as:

- A timer managed with setInterval() and clearInterval().
- An event subscription using window.addEventListener() and window.removeEventListener().
- A third-party animation library with an API like animation.start() and animation.reset().

If you’re not connecting to any external system, you probably don’t need an Effect.

```js
useEffect(setup, dependencies?)
```

react on first render runs setup and on next renders (on dep change) it first runs cleanup function returned by setup on unmount and then run setup with new dependency values

Connecting to an external system
Some components need to stay connected to the network, some browser API, or a third-party library, while they are displayed on the page. These systems aren’t controlled by React, so they are called external.

1. A setup function with setup code that connects to that system.

- It should return a cleanup function with cleanup code that disconnects from that system.

2. A list of dependencies including every value from your component used inside of those functions.

React calls your setup and cleanup functions whenever it’s necessary, which may happen multiple times:

1. Your setup code runs when your component is added to the page (mounts).
2. After every commit of your component where the dependencies have changed:

- First, your cleanup code runs with the old props and state.
- Then, your setup code runs with the new props and state.

3. Your cleanup code runs one final time after your component is removed from the page (unmounts).

How to remove unnecessary Effects

- Fetching data
  Either use custom hook or use data fetching library
- Resetting all state when a prop changes

  ```js
  export default function Profile({ userId }) {
    const [comment, setComment] = useState("");

    // 🔴 Avoid: Resetting state on prop change in an Effect
    useEffect(() => {
      setComment("");
    }, [userId]);
    // ...
  }

  // Split your component in two and pass a key attribute from the outer component to the inner one:

  export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
  }
  ```

- Initializing the application

```js
function App() {
  // 🔴 Avoid: Effects with logic that should only ever run once
  useEffect(() => {
    loadDataFromLocalStorage();
    checkAuthToken();
  }, []);
  // ...
}

// You can also run it during module initialization and before the app renders
if (typeof window !== "undefined") {
  // Check if we're running in the browser.
  // ✅ Only runs once per app load
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

The lifecycle of an Effect
Every React component goes through the same lifecycle:

A component mounts when it’s added to the screen.
A component updates when it receives new props or state, usually in response to an interaction.
A component unmounts when it’s removed from the screen.
It’s a good way to think about components, but not about Effects.

Props, state, and other values declared inside the component are reactive because they’re calculated during rendering and participate in the React data flow.
Reactive values must be included in dependencies

A mutable value like `location.pathname` or `ref.current` can’t be a dependency. It’s mutable, so it can change at any time completely outside of the React rendering data flow. Changing it wouldn’t trigger a re-render of your component.

## useLayoutEffect

`useLayoutEffect` is a version of `useEffect` that fires before the browser repaints the screen.

```js
useLayoutEffect(setup, dependencies?)
```

Note: `useLayoutEffect` can hurt performance. Prefer `useEffect` when possible.

Usage
Call useLayoutEffect to perform the layout measurements before the browser repaints the screen
Measuring layout before the browser repaints the screen

---

## useRef
`useRef` lets you reference a value that’s not needed for rendering.

```js
const ref = useRef(initialValue)
```

`ref.current` property is mutable
When you change the ref.current property, React does not re-render your component. React is not aware of when you change it because a ref is a plain JavaScript object.

Do not write or read ref.current during rendering.

React expects that the body of your component behaves like a pure function:

If the inputs (props, state, and context) are the same, it should return exactly the same JSX.
Calling it in a different order or with different arguments should not affect the results of other calls.

Manipulating the DOM with a ref 
```js
    const inputRef = useRef(null);

    function handleClick() {
    inputRef.current.focus();
  }

  return <input ref={inputRef} />;
```

React saves the initial ref value once and ignores it on the next renders.
```js
function Video() {
// videoplayer will be initialized everytime but will be ignored as playerRef will use this only on initial render
const playerRef = useRef(new VideoPlayer());
  // ...
}

```

## useImperativeHandle
`useImperativeHandle` allows you to manually define what a parent component sees when it accesses a ref on your component. Instead of giving the parent the entire DOM node, you provide a custom object with specific methods.

### The React 19 Shift
- Old Way: `forwardRef` lets your component expose a DOM node via `ref`.
- New Way: Starting in React 19, `ref` is a standard prop. You no longer need `forwardRef` for new components.

### Example
To hide the raw DOM node and only expose specific actions (like focus), follow this structure:

```js
import { useRef, useImperativeHandle } from 'react';

function MyInput({ ref }) {
  // 1. Create an internal ref to hold the actual DOM node
  const inputRef = useRef(null);

  // 2. Customize the 'handle' exposed to the parent
  useImperativeHandle(ref, () => {
    return {
      focus() {
        inputRef.current.focus();
      },
      scrollIntoView() {
        inputRef.current.scrollIntoView();
      },
    };
  }, []); // Dependencies array works like useEffect

  // 3. Attach the internal ref to the DOM element
  return <input ref={inputRef} />;
};
```

### Important Pitfalls
- **Don't Overuse**: If a task can be achieved via a prop (like changing a color or toggling a boolean), use props.
- **Encapsulation**: Use this Hook to `limit access`. By `not passing the ref prop directly` to the <input>, you prevent the parent from accidentally changing styles or attributes it shouldn't touch.
- **Dependencies**: If the methods inside `useImperativeHandle` rely on props or state, ensure they are included in the dependency array to avoid stale closures.

---

## createRoot

`createRoot` lets you create a root to display React components inside a browser DOM node.

```js
const root = createRoot(domNode, options?)
```

Call createRoot to create a React root for displaying content inside a browser DOM element.
React will create a root for the domNode, and take over managing the DOM inside it. After you’ve created a root, you need to call root.render to display a React component inside of it:

```js
import { createRoot } from 'react-dom/client';

const domNode = document.getElementById('root'); // can also create element
const root = createRoot(domNode);

root.render(<App />);
```

An app fully built with React will usually only have one createRoot call for its root component. A page that uses “sprinkles” of React for parts of the page may have as many separate roots as needed.

If your app is server-rendered, using createRoot() is not supported. Use hydrateRoot() instead.
You’ll likely have only one createRoot call in your app. If you use a framework, it might do this call for you.
When you want to render a piece of JSX in a different part of the DOM tree that isn’t a child of your component (for example, a modal or a tooltip), use createPortal instead of createRoot.

To remove the React tree from the DOM node and clean up all the resources used by it, call root.unmount.

root.unmount();

Apps using server rendering or static generation must call hydrateRoot instead of createRoot. React will then hydrate (reuse) the DOM nodes from your HTML instead of destroying and re-creating them.


---

## createPortal
createPortal lets you render some children into a different part of the DOM.

```js
<div>
  <SomeComponent />
  {createPortal(children, domNode, key?)}
</div>
```

To create a portal, call createPortal, passing some JSX, and the DOM node where it should be rendered:

```js
import { createPortal } from 'react-dom';

// ...

<div>
  <p>This child is placed in the parent div.</p>
  {createPortal(
    <p>This child is placed in the document body.</p>,
    document.body
  )}
</div>
```

A portal only changes the physical placement of the DOM node. In every other way, the JSX you render into a portal acts as a child node of the React component that renders it. For example, the child can access the context provided by the parent tree, and events bubble up from children to parents according to the React tree.

Portals let your components render some of their children into a different place in the DOM. This lets a part of your component “escape” from whatever containers it may be in. For example, a component can display a modal dialog or a tooltip that appears above and outside of the rest of the page.

To create a portal, render the result of createPortal with some JSX and the DOM node where it should go:

---

## React.memo
`memo` lets you **skip re-rendering a component when its props are unchanged**.

```js
const MemoizedComponent = memo(SomeComponent, arePropsEqual?)
```

React normally re-renders a component whenever its parent re-renders. With `memo`, you can create a component that React **will not re-render when its parent re-renders so long as its new props are the same as the old props (Not a Guarantee)**. Such a component is said to be memoized. 

---

object.is 
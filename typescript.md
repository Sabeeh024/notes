# TypeScript Foundation (Compact Notes)

## 1. Types

### Primitive Types
`string` | `number` | `boolean` |	`bigint` | `symbol` | `null` | `undefined`

### Special Types
`void` | `any` | `unknown` | `never` 
* `any`: Disables type checking, fallback for unknown or mixed types. Should be avoided unless necessary.
  Use case: legacy JavaScript code or third‑party libraries without types.
* `unknown`: Safer alternative to `any`. Type must be checked before use. Useful when type will be determined at runtime.
  Use case: API responses, JSON.parse, or user input where data shape is uncertain.
* `never`: Represents values that never occur. Appears in impossible intersections, functions that always throw, or exhaustive checks
```ts
type Impossible = "success" & "error"; // never

function crash(): never {
  throw new Error("Boom");
}

type Status = "success" | "error";
function handle(status: Status) {
  switch (status) {
    case "success": return "OK";
    case "error": return "FAIL";
    default:
      const _exhaustive: never = status; // compile-time check
      return _exhaustive;
  }
}
```

### Literal Types
Represent exact values:

```ts
let a: 42;          // number literal type
let b: "hello";     // string literal type
let c: true;        // boolean literal type
```
### Union and Intersection Types

* Union Type (`|`) : Allow a value to be one of multiple types.
```ts
type Status = "success" | "error";
```

* Intersection Type (`&`) : Combines multiple types; must satisfy all

```ts
type WithId = { id: number };
type WithName = { name: string };

type User = WithId & WithName; // becomes User = { id: number, name: string } 
```

###  Utility / Advanced Types 

* `Record<Keys, Type>` → create an object type with keys of type Keys and values of type Type
* `Partial<T>` → make all properties optional
* `Required<T>` → make all properties required
* `Readonly<T>` → make all properties readonly
* `Pick<T, K>` → select subset of properties
* `Omit<T, K>` → remove properties from type

```ts
let phoneBook: Record<string, number> = {
  "Alice": 12345,
  "Bob": 67890
};

type User = { id: number; name: string; age: number };
type PartialUser = Partial<User>;
type NameAndAge = Pick<User, 'name' | 'age'>;
type PublicUser = Omit<User, 'id'>;
```

---

## 2. Type Inference

* TypeScript automatically infers types; so let TypeScript infer as much as possible.

```ts
let city = "Lahore"; // string inferred
```

---

## 3. Core Type Modeling

### Type Aliases

* Used to define reusable custom types.
* Commonly used for object shapes and unions.

```ts
type User = {
  id: number;
  name: string;
};
```

### Interfaces

* Similar to type aliases, mainly for object shapes.
* Preferred when working with classes or public APIs.

```ts
interface User {
  id: number;
  name: string;
}
```

### `type` vs `interface` (practical rule)

* Use `type` for unions and simple models.
* Use `interface` for objects meant to be extended.

---

## 4. Type Safety & Logic

### Type Narrowing

* Allows TypeScript to **refine a type at runtime** using checks

```ts
let value: string | number;
if (typeof value === "string") {
  console.log(value.toUpperCase()); // safe
}
```

### `in` Operator

* Check if property exists in object

```ts
type User = { name: string } | { id: number };
function logName(u: User) {
  if ("name" in u) console.log(u.name);
}
```

### Truthy Checks

* Safely narrow nullable types

```ts
let input: string | null;
if (input) {
  console.log(input.length);
}
```
// compile vs runtine 
// interface vs type 
// readonly vs as const

# What is the difference between typescript void and undefined?

1. undefined is a real value

2. void is not a value — it’s a TypeScript-only return type

```js
function log(msg: string): void {
  console.log(msg);
}
```

At runtime, the function actually returns undefined, but TypeScript forbids 
you from using that return value.

You cannot assign a void result to a non-void variable.

You cannot expect void to carry information.

void is a type-level concept, not a runtime value.

# Unknown vs any, what is the difference

`any` disables type checking. You can do anything with it.

`unknown` is the safe version of any: you can store anything in it, but cannot 
use it without narrowing.

```ts
let y: unknown = 123;

y.trim(); // ❌ error
y + 1;    // ❌ error

if (typeof y === "number") {
  console.log(y + 1); // ok
}
```

```ts
// Parsing something:
function parse(json: string): unknown {
  return JSON.parse(json);
}
```

```ts
// Error types
try {
  // ...
} catch (err: unknown) {
  if (err instanceof Error) console.error(err.message);
}
```

```ts
// Generic helpers
function identity<T = unknown>(value: T): T {
  return value;
}
```

# Why do we need "never"

No value can ever have type never.

You can’t assign anything to it.

You can’t produce it.

It’s the bottom type in TS’s type system lattice.

1. Exhaustiveness checking

```ts
type Shape =
  | { type: "circle"; r: number }
  | { type: "rect"; w: number; h: number };

function area(s: Shape) {
  switch (s.type) {
    case "circle": return Math.PI * s.r ** 2;
    case "rect": return s.w * s.h;
    default:
      const _exhaustive: never = s;  // ❌ error if a new shape is added
      return _exhaustive;
  }
}
```

2. Impossible paths in generics

```ts
type OnlyStrings<T> = T extends string ? T : never;

type A = OnlyStrings<string | number>; 
// A = string | never = string
```

3. Filter out union members

```ts
type Without<T, U> = T extends U ? never : T;

type A = Without<"a" | "b" | "c", "b">;
// "a" | "c"
```

4. The identity of empty intersections

```ts
type X = string & number;
// X = never
```

5. Advanced mapped types

```ts
type RemoveKeys<T, K extends keyof T> = {
  [P in keyof T as P extends K ? never : P]: T[P];
};
```

6. Uninhabited branches in utility types

```ts
type InferArray<T> = T extends (infer U)[] ? U : never;

type A = InferArray<string[]>; // string
type B = InferArray<number>;   // never
```

# What is a union type

A union type in TypeScript means “a value that can be one of several possible types.”

```ts
let value: string | number;

value = "hello"; // ok
value = 42;      // ok
value = true;    // ❌ not allowed
```

# Why do we need union types if we have interfaces?

Interfaces tell TypeScript:
“Here’s how this object looks.”

A union type means:
"A value might be one of several different interfaces."

```ts
interface Success {
  ok: true;
  data: string;
}

interface Failure {
  ok: false;
  error: string;
}

type Response = Success | Failure;
```

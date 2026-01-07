# Variadic Tuple Types (TypeScript 4.0)

## What it is

Variadic tuple types allow generic spreads in tuple type definitions, enabling you to create generic functions that preserve the exact types of rest parameters and tuple operations. This significantly improves type inference for functions like `concat`, `partial`, and tuple transformations.

## Before this feature

Before TypeScript 4.0, you couldn't preserve exact tuple types through generic operations:

```typescript
// TypeScript pre-4.0 - loses type information
function concat(arr1: any[], arr2: any[]): any[] {
  return [...arr1, ...arr2];
}

let result = concat([1, 2], ["a", "b"]);  // Type: any[] (not precise!)

// Couldn't model functions that transform tuples generically
```

## After this feature

Variadic tuple types preserve and transform tuple types precisely:

```typescript
// TypeScript 4.0+ - preserves exact types

// 1. Generic tuple concatenation
function concat<T extends any[], U extends any[]>(
  arr1: T,
  arr2: U
): [...T, ...U] {
  return [...arr1, ...arr2];
}

let result = concat([1, 2], ["a", "b"]);
// Type: [number, number, string, string] - exact!

// 2. Partial application (curry)
function partial<T extends any[], U extends any[], R>(
  fn: (...args: [...T, ...U]) => R,
  ...head: T
): (...tail: U) => R {
  return (...tail: U) => fn(...head, ...tail);
}

function greet(greeting: string, name: string, punctuation: string): string {
  return `${greeting}, ${name}${punctuation}`;
}

let sayHello = partial(greet, "Hello");
let result1 = sayHello("Alice", "!");  // "Hello, Alice!"
// Type of sayHello: (name: string, punctuation: string) => string

// 3. Prepend and append to tuples
type Prepend<T extends any[], E> = [E, ...T];
type Append<T extends any[], E> = [...T, E];

type Numbers = [1, 2, 3];
type WithZero = Prepend<Numbers, 0>;      // [0, 1, 2, 3]
type WithFour = Append<Numbers, 4>;       // [1, 2, 3, 4]

// 4. Tuple operations
type Reverse<T extends any[]> = 
  T extends [infer First, ...infer Rest]
    ? [...Reverse<Rest>, First]
    : [];

type Original = [1, 2, 3];
type Reversed = Reverse<Original>;  // [3, 2, 1]

// 5. Function composition with preserved types
function compose<A extends any[], B, C>(
  f: (b: B) => C,
  g: (...args: A) => B
): (...args: A) => C {
  return (...args: A) => f(g(...args));
}

const add = (a: number, b: number) => a + b;
const double = (n: number) => n * 2;

const addAndDouble = compose(double, add);
let value = addAndDouble(3, 4);  // (3 + 4) * 2 = 14
// Type: (a: number, b: number) => number - preserved!

// 6. Typed rest parameters with leading/trailing elements
type Strings = [string, string];
type Numbers = [number, number];

function merge<T extends any[], U extends any[]>(
  ...args: [...T, ...U]
): [...T, ...U] {
  return args;
}

// 7. Practical example: Event handler with typed arguments
type EventHandler<T extends any[]> = (...args: T) => void;

function on<T extends any[]>(
  event: string,
  handler: EventHandler<T>
): void {
  // Implementation
}

on<[MouseEvent]>("click", (e) => {
  console.log(e.clientX);  // TypeScript knows e is MouseEvent
});

on<[string, number]>("custom", (message, code) => {
  console.log(message.toUpperCase(), code.toFixed(2));
  // TypeScript knows message is string, code is number
});
```

## Why this is better

- **Precise type inference**: Exact tuple types preserved through operations
- **Generic tuple operations**: Create reusable tuple transformations
- **Better function composition**: Type-safe function composition and currying
- **Flexible API design**: Build libraries with complex generic signatures
- **No type loss**: Spread operations maintain exact types
- **Self-documenting**: Complex type relationships expressed precisely

## Key notes / edge cases

- Variadic tuple types require TypeScript 4.0 or higher
- Works with both named and anonymous tuple elements:
  ```typescript
  type Named = [name: string, age: number];
  type Anonymous = [string, number];
  ```

- Can be combined with labeled tuple elements (TS 4.0):
  ```typescript
  type Range = [start: number, end: number];
  type WithLabel = [...numbers: number[], label: string];
  ```

- Recursive tuple types are possible but can be complex:
  ```typescript
  type Reverse<T extends any[]> = 
    T extends [infer First, ...infer Rest]
      ? [...Reverse<Rest>, First]
      : [];
  ```

- Empty tuples are handled correctly:
  ```typescript
  type Empty = [];
  type WithEmpty = [...Empty, number];  // [number]
  ```

- Spreading non-tuple types requires constraints:
  ```typescript
  function spread<T extends readonly any[]>(...args: T): T {
    return args;
  }
  ```

## Quick practice

1. **Create a tail function**: Write a generic function `tail<T extends any[]>(arr: T): Tail<T>` that returns all elements except the first, with proper typing.

2. **Curry a function**: Implement a curry function that converts:
   ```typescript
   (a: number, b: string, c: boolean) => void
   ```
   into:
   ```typescript
   (a: number) => (b: string) => (c: boolean) => void
   ```

3. **Tuple zip**: Create a type `Zip<T, U>` that pairs elements from two tuples:
   ```typescript
   type A = [1, 2, 3];
   type B = ["a", "b", "c"];
   type Zipped = Zip<A, B>;  // [[1, "a"], [2, "b"], [3, "c"]]
   ```

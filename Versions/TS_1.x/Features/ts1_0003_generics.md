# Generics (TypeScript 1.0)

## What it is

Generics allow you to create reusable components that work with multiple types while maintaining type safety. Introduced in TypeScript 1.0, generics enable type parameters that let functions, classes, and interfaces work over a variety of types.

## Before this feature

Without generics, you'd either write duplicate code or use `any`:

```typescript
// Duplicate code for each type
function getFirstNumber(arr: number[]): number {
  return arr[0];
}

function getFirstString(arr: string[]): string {
  return arr[0];
}

// Or lose type safety with 'any'
function getFirst(arr: any[]): any {
  return arr[0];
}

let result = getFirst([1, 2, 3]);  // Type: any (not safe!)
result.toUpperCase();              // No error, but crashes at runtime
```

## After this feature

Generics provide reusability with type safety:

```typescript
// Generic function - works with any type, preserves type info
function getFirst<T>(arr: T[]): T {
  return arr[0];
}

let num = getFirst([1, 2, 3]);           // Type: number
let str = getFirst(["a", "b", "c"]);     // Type: string
let bool = getFirst([true, false]);      // Type: boolean

// num.toUpperCase();  // ✗ Error: Property 'toUpperCase' does not exist on type 'number'
str.toUpperCase();     // ✓ OK - TypeScript knows it's a string

// Generic interface
interface Box<T> {
  value: T;
}

let stringBox: Box<string> = { value: "hello" };
let numberBox: Box<number> = { value: 42 };

// Generic class
class Container<T> {
  private value: T;
  
  constructor(value: T) {
    this.value = value;
  }
  
  getValue(): T {
    return this.value;
  }
}

let container = new Container<string>("hello");
let value = container.getValue();  // Type: string
```

## Why this is better

- **Code reusability**: Write once, use with any type
- **Type safety**: No loss of type information like with `any`
- **Less duplication**: One generic function instead of many type-specific ones
- **Better refactoring**: Change generic logic in one place
- **Flexibility**: Works with types that don't exist yet

## Key notes / edge cases

- Type parameters are conventionally named `T`, `U`, `V`, etc., but can be any valid identifier
- TypeScript can infer generic types from usage, so explicit type arguments are often optional
- Generic constraints use `extends` to restrict what types can be used
- Multiple type parameters are comma-separated: `<T, U>`
- Default type parameters can be specified: `<T = string>`

```typescript
// Generic constraints
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(item: T): void {
  console.log(item.length);
}

logLength("hello");        // ✓ OK - string has length
logLength([1, 2, 3]);      // ✓ OK - array has length
// logLength(42);          // ✗ Error: number doesn't have length

// Multiple type parameters
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

let result = pair("hello", 42);  // Type: [string, number]

// keyof with generics
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

let user = { name: "Alice", age: 30 };
let name = getProperty(user, "name");  // Type: string
let age = getProperty(user, "age");    // Type: number
```

## Quick practice

1. **Create a generic function**: Write a `last<T>` function that returns the last element of an array of type `T[]`.

2. **Generic class**: Create a generic `Pair<T, U>` class that holds two values of potentially different types and has getters for both.

3. **Constrained generic**: Write a function `concatenate<T extends HasLength>` that takes two items with a `length` property and returns their combined length.

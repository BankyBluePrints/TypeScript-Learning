# Generics Basics

## What it is

Generics allow you to write reusable, type-safe code that works with multiple types instead of a single type. They act as type parameters that let you create components (functions, classes, interfaces) that can work over a variety of types while maintaining type safety.

## Before Generics (Type-Specific or Unsafe Code)

Without generics, you either write duplicate code for each type or lose type safety:

```typescript
// Option 1: Duplicate code for each type
function getFirstNumber(arr: number[]): number {
  return arr[0];
}

function getFirstString(arr: string[]): string {
  return arr[0];
}

function getFirstBoolean(arr: boolean[]): boolean {
  return arr[0];
}

// Option 2: Use 'any' and lose type safety
function getFirst(arr: any[]): any {
  return arr[0];
}

let num = getFirst([1, 2, 3]);     // Type: any (not safe!)
num.toUpperCase();                  // No error, but crashes at runtime
```

## With Generics (Reusable and Type-Safe)

Generics let you write one function that works with any type while preserving type information:

```typescript
// Generic function - works with any type
function getFirst<T>(arr: T[]): T {
  return arr[0];
}

// TypeScript infers the type automatically
let num = getFirst([1, 2, 3]);           // Type: number
let str = getFirst(["a", "b", "c"]);     // Type: string
let bool = getFirst([true, false]);      // Type: boolean

// num.toUpperCase();  // ✗ Error: Property 'toUpperCase' does not exist on type 'number'
str.toUpperCase();     // ✓ OK

// You can also explicitly specify the type
let result = getFirst<number>([1, 2, 3]);
```

## Generic Examples

```typescript
// 1. Generic Functions
function identity<T>(value: T): T {
  return value;
}

identity<string>("hello");  // Explicit type
identity(42);               // Type inferred as number

// 2. Generic Interfaces
interface Box<T> {
  value: T;
}

let stringBox: Box<string> = { value: "hello" };
let numberBox: Box<number> = { value: 42 };

// 3. Generic Classes
class Container<T> {
  private value: T;
  
  constructor(value: T) {
    this.value = value;
  }
  
  getValue(): T {
    return this.value;
  }
  
  setValue(value: T): void {
    this.value = value;
  }
}

let stringContainer = new Container<string>("hello");
let numberContainer = new Container(42);  // Type inferred

// 4. Generic Constraints (using 'extends')
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(item: T): void {
  console.log(item.length);  // OK - T is guaranteed to have 'length'
}

logLength("hello");        // ✓ OK - string has length
logLength([1, 2, 3]);      // ✓ OK - array has length
// logLength(42);          // ✗ Error: number doesn't have length

// 5. Multiple Type Parameters
function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

let p1 = pair("hello", 42);           // Type: [string, number]
let p2 = pair(true, "world");         // Type: [boolean, string]

// 6. Generic Type Aliases
type Result<T> = {
  success: boolean;
  data: T;
};

let userResult: Result<User> = {
  success: true,
  data: { name: "Alice", age: 30 }
};

// 7. Default Type Parameters
interface Response<T = any> {
  data: T;
  status: number;
}

let response1: Response = { data: "anything", status: 200 };  // T defaults to any
let response2: Response<User> = { data: { name: "Bob" }, status: 200 };

// 8. Keyof with Generics
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

let user = { name: "Alice", age: 30 };
let name = getProperty(user, "name");   // Type: string
let age = getProperty(user, "age");     // Type: number
// let bad = getProperty(user, "email"); // ✗ Error: "email" is not a key of user
```

## Real-World Generic Use Cases

```typescript
// Array methods (built into TypeScript)
const numbers = [1, 2, 3];
const doubled = numbers.map<number>((n) => n * 2);  // Type: number[]

// Promise (built-in generic)
async function fetchUser(): Promise<User> {
  const response = await fetch("/api/user");
  return response.json();  // TypeScript knows this returns User
}

// Generic utility functions
function toArray<T>(value: T): T[] {
  return [value];
}

function swap<T, U>(pair: [T, U]): [U, T] {
  return [pair[1], pair[0]];
}

// Generic repository pattern
interface Repository<T> {
  getById(id: string): Promise<T>;
  save(item: T): Promise<void>;
  delete(id: string): Promise<void>;
}

class UserRepository implements Repository<User> {
  async getById(id: string): Promise<User> {
    // Implementation
  }
  async save(user: User): Promise<void> {
    // Implementation
  }
  async delete(id: string): Promise<void> {
    // Implementation
  }
}
```

## Why This Is Better

- **Code reusability**: Write once, use with any type
- **Type safety**: No loss of type information like with `any`
- **Less duplication**: One generic function instead of many type-specific ones
- **Better refactoring**: Change the generic logic in one place
- **Self-documenting**: Clear about what types are expected and returned
- **Flexibility**: Works with types that don't exist yet

## Common Mistake + Fix

**Mistake**: Forgetting to constrain generics when you need specific properties

```typescript
// Bad - can't access length without constraint
function logLength<T>(item: T): void {
  console.log(item.length);  // ✗ Error: Property 'length' does not exist on type 'T'
}
```

**Fix**: Use generic constraints with `extends`

```typescript
// Good - constraint ensures 'length' exists
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(item: T): void {
  console.log(item.length);  // ✓ OK
}

// Or use inline constraint
function logLength2<T extends { length: number }>(item: T): void {
  console.log(item.length);
}
```

## Key Notes / Edge Cases

- **Generic inference**: TypeScript often infers generic types, so you don't always need to specify them explicitly:
  ```typescript
  function wrap<T>(value: T) { return { value }; }
  
  wrap("hello");  // T inferred as string
  wrap(42);       // T inferred as number
  ```

- **Variance**: Generics in TypeScript are covariant (can assign more specific types):
  ```typescript
  let boxes: Box<string | number>[] = [];
  let stringBoxes: Box<string>[] = [];
  boxes = stringBoxes;  // ✓ OK (covariant)
  ```

- **Cannot create generic instances**: You can't use `new T()` inside a generic function:
  ```typescript
  function create<T>(): T {
    return new T();  // ✗ Error: 'T' only refers to a type, but is being used as a value
  }
  ```

- **Generic type erasure**: Generics are a compile-time feature only and don't exist at runtime:
  ```typescript
  function isArray<T>(value: T[] | T): value is T[] {
    return Array.isArray(value);  // Can't use 'T' at runtime
  }
  ```

## Quick Practice

1. **Create a generic function**: Write a function `last<T>` that returns the last element of an array of type `T[]`.

2. **Add constraints**: Write a generic function that takes an object and a key, and returns the value at that key. Ensure the key actually exists on the object.

3. **Debug this code**: Why doesn't this compile?
   ```typescript
   function merge<T, U>(obj1: T, obj2: U): T & U {
     return { ...obj1, ...obj2 };
   }
   
   const result = merge({ name: "Alice" }, { age: 30 });
   console.log(result.email);  // What's the error here?
   ```

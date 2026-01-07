# Readonly Properties and Index Signatures (TypeScript 2.0)

## What it is

TypeScript 2.0 introduced the `readonly` modifier for properties and index signatures, ensuring that properties cannot be reassigned after initialization. This provides immutability guarantees at the type level.

## Before this feature

Before readonly, there was no way to prevent property reassignment:

```typescript
// TypeScript pre-2.0
interface Point {
  x: number;
  y: number;
}

let point: Point = { x: 10, y: 20 };
point.x = 30;  // Can mutate - no protection
point.y = 40;  // Can mutate - no protection

// No compile-time immutability guarantees
```

## After this feature

The `readonly` modifier prevents property reassignment:

```typescript
// TypeScript 2.0+ with readonly
interface Point {
  readonly x: number;
  readonly y: number;
}

let point: Point = { x: 10, y: 20 };
// point.x = 30;  // ✗ Error: Cannot assign to 'x' because it is a read-only property
// point.y = 40;  // ✗ Error: Cannot assign to 'y' because it is a read-only property

// Readonly class properties
class Config {
  readonly apiUrl: string = "https://api.example.com";
  readonly version: string;
  
  constructor(version: string) {
    this.version = version;  // ✓ OK in constructor
  }
  
  updateVersion(newVersion: string) {
    // this.version = newVersion;  // ✗ Error: Cannot assign to 'version'
  }
}

// Readonly index signature
interface ReadonlyStringMap {
  readonly [key: string]: string;
}

let map: ReadonlyStringMap = {
  name: "Alice",
  role: "Admin"
};

console.log(map.name);  // ✓ OK to read
// map.name = "Bob";    // ✗ Error: Index signature only permits reading

// ReadonlyArray<T> (built-in utility type)
let numbers: ReadonlyArray<number> = [1, 2, 3];
console.log(numbers[0]);       // ✓ OK to read
// numbers[0] = 10;            // ✗ Error: Index signature only permits reading
// numbers.push(4);            // ✗ Error: Property 'push' does not exist on ReadonlyArray
// numbers.length = 0;         // ✗ Error: Cannot assign to 'length'

// Readonly<T> utility type (maps all properties to readonly)
interface User {
  name: string;
  age: number;
}

type ReadonlyUser = Readonly<User>;
// Equivalent to: { readonly name: string; readonly age: number; }

let user: ReadonlyUser = { name: "Alice", age: 30 };
// user.name = "Bob";  // ✗ Error: Cannot assign to 'name'
```

## Why this is better

- **Immutability guarantees**: Prevents accidental mutations
- **Better reasoning**: Easier to understand code when values don't change
- **Functional programming**: Supports immutable data patterns
- **Thread safety**: Immutable objects are inherently thread-safe
- **Self-documenting**: Clear intent that values shouldn't change
- **Safer refactoring**: Compiler catches unintended mutations

## Key notes / edge cases

- `readonly` is a compile-time check only. At runtime (JavaScript), properties can still be mutated
- Shallow immutability: `readonly` doesn't make nested objects immutable:
  ```typescript
  interface User {
    readonly profile: {
      name: string;
      age: number;
    };
  }
  
  let user: User = { profile: { name: "Alice", age: 30 } };
  // user.profile = {};          // ✗ Error: Cannot assign to 'profile'
  user.profile.name = "Bob";     // ✓ OK - nested properties not readonly
  ```

- Deep readonly requires utility types or recursive readonly:
  ```typescript
  type DeepReadonly<T> = {
    readonly [P in keyof T]: DeepReadonly<T[P]>;
  };
  ```

- Readonly arrays can be created with `as const`:
  ```typescript
  let arr = [1, 2, 3] as const;  // readonly [1, 2, 3]
  ```

- Readonly is compatible with mutable types (covariant):
  ```typescript
  let mutable: number[] = [1, 2, 3];
  let readonly: ReadonlyArray<number> = mutable;  // ✓ OK
  ```

## Quick practice

1. **Create a readonly interface**: Design a `Configuration` interface where all properties (apiUrl, timeout, retries) are readonly.

2. **Fix the mutation**: This code tries to mutate a readonly array. How would you create a new array instead?
   ```typescript
   let numbers: ReadonlyArray<number> = [1, 2, 3];
   // How to add 4 to the array?
   ```

3. **Deep readonly challenge**: Given this interface, how would you make both the outer and nested properties readonly?
   ```typescript
   interface AppConfig {
     database: {
       host: string;
       port: number;
     };
     cache: {
       enabled: boolean;
       ttl: number;
     };
   }
   ```

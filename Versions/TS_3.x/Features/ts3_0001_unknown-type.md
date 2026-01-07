# Unknown Type (TypeScript 3.0)

## What it is

The `unknown` type is a type-safe counterpart to `any`. Introduced in TypeScript 3.0, it represents any value but requires type checking before performing operations on it. Unlike `any`, you cannot access properties or call methods on an `unknown` value without first narrowing its type.

## Before this feature

Before `unknown`, developers used `any` for values of uncertain type:

```typescript
// TypeScript pre-3.0 - using 'any'
function processValue(value: any) {
  return value.toUpperCase();  // No type checking - compiles but might crash!
}

processValue("hello");   // Works
processValue(42);        // Crashes at runtime!
processValue(null);      // Crashes at runtime!

// 'any' turns off type checking completely
let data: any = getSomeData();
data.foo.bar.baz();  // Compiles, but probably crashes
```

## After this feature

`unknown` provides type safety while allowing any value:

```typescript
// TypeScript 3.0+ - using 'unknown'
function processValue(value: unknown) {
  // Cannot use value without type checking
  // return value.toUpperCase();  // ✗ Error: Object is of type 'unknown'
  
  // Must narrow the type first
  if (typeof value === "string") {
    return value.toUpperCase();  // ✓ OK - TypeScript knows it's a string
  }
  return "Not a string";
}

processValue("hello");   // "HELLO"
processValue(42);        // "Not a string"
processValue(null);      // "Not a string"

// Type narrowing with unknown
function logValue(value: unknown) {
  // typeof guard
  if (typeof value === "string") {
    console.log(value.length);
  }
  
  // instanceof guard
  if (value instanceof Date) {
    console.log(value.toISOString());
  }
  
  // Type predicate
  if (isArray(value)) {
    console.log(value.length);
  }
  
  // Type assertion (use with caution!)
  const str = value as string;
  console.log(str.toUpperCase());
}

function isArray(value: unknown): value is any[] {
  return Array.isArray(value);
}

// Reading JSON safely
function parseJSON(jsonString: string): unknown {
  return JSON.parse(jsonString);  // Returns unknown instead of any
}

const data = parseJSON('{"name": "Alice", "age": 30}');
// console.log(data.name);  // ✗ Error: Object is of type 'unknown'

// Must validate structure
if (
  typeof data === "object" &&
  data !== null &&
  "name" in data &&
  "age" in data
) {
  const user = data as { name: string; age: number };
  console.log(user.name);  // ✓ OK after validation
}
```

## Why this is better

- **Type safety**: Forces you to check types before using values
- **No implicit any errors**: Better than `any` which disables type checking
- **Explicit validation**: Makes type checking explicit in code
- **Prevents runtime errors**: Catches type errors at compile time
- **Better for library authors**: Public APIs can accept unknown instead of any
- **Self-documenting**: Shows that type is uncertain and must be checked

## Key notes / edge cases

- `unknown` is the type-safe counterpart of `any`
- You can assign anything to `unknown`, but can't do anything with it without narrowing
- `unknown` vs `any`:
  ```typescript
  let a: any = 10;
  let b: unknown = 10;
  
  let c = a + 10;  // ✓ OK with any (no type checking)
  // let d = b + 10;  // ✗ Error with unknown (type checking required)
  ```

- Type assertions can convert `unknown` to specific types:
  ```typescript
  let value: unknown = "hello";
  let str = value as string;  // Assert it's a string
  ```

- Use type guards for safe narrowing:
  ```typescript
  function isString(value: unknown): value is string {
    return typeof value === "string";
  }
  
  function process(value: unknown) {
    if (isString(value)) {
      console.log(value.toUpperCase());  // ✓ Safe
    }
  }
  ```

- `unknown` is the top type in TypeScript's type hierarchy (everything is assignable to it):
  ```typescript
  let u: unknown;
  u = 5;          // ✓ OK
  u = "hello";    // ✓ OK
  u = true;       // ✓ OK
  u = null;       // ✓ OK
  u = undefined;  // ✓ OK
  u = {};         // ✓ OK
  ```

- Cannot assign `unknown` to other types without narrowing:
  ```typescript
  let u: unknown = "hello";
  // let s: string = u;  // ✗ Error
  
  if (typeof u === "string") {
    let s: string = u;  // ✓ OK after narrowing
  }
  ```

## Quick practice

1. **Refactor from any to unknown**: Convert this function to use `unknown` instead of `any`:
   ```typescript
   function stringify(value: any): string {
     return value.toString();
   }
   ```

2. **Validate API response**: Write a function that accepts `unknown` data and validates it has the shape `{ id: number; name: string }` before using it.

3. **Type guard**: Create a type guard function `isUser(value: unknown): value is User` that checks if a value matches a User interface.

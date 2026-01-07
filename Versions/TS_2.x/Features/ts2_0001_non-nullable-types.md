# Non-nullable Types (TypeScript 2.0)

## What it is

TypeScript 2.0 introduced the `--strictNullChecks` compiler flag, which makes `null` and `undefined` separate types that must be explicitly included in type unions. This prevents the common "billion-dollar mistake" of null reference errors.

## Before this feature

Before strictNullChecks, `null` and `undefined` were assignable to all types:

```typescript
// TypeScript pre-2.0 (without strictNullChecks)
let name: string = "Alice";
name = null;       // OK - no error!
name = undefined;  // OK - no error!

function getLength(str: string): number {
  return str.length;  // Can crash if str is null/undefined
}

getLength(null);       // Compiles, but crashes at runtime!
getLength(undefined);  // Compiles, but crashes at runtime!
```

## After this feature

With `--strictNullChecks` enabled, null and undefined are distinct types:

```typescript
// TypeScript 2.0+ with strictNullChecks
let name: string = "Alice";
// name = null;       // ✗ Error: Type 'null' is not assignable to type 'string'
// name = undefined;  // ✗ Error: Type 'undefined' is not assignable to type 'string'

// Must explicitly allow null/undefined with union types
let nullableName: string | null = "Alice";
nullableName = null;  // ✓ OK

let optionalName: string | undefined = "Alice";
optionalName = undefined;  // ✓ OK

// Function with nullable parameter
function getLength(str: string | null): number {
  if (str === null) {
    return 0;
  }
  return str.length;  // ✓ Safe - TypeScript knows str is not null here
}

getLength("hello");  // ✓ OK
getLength(null);     // ✓ OK - null is explicitly allowed
// getLength(undefined);  // ✗ Error: undefined not in union type

// Non-null assertion operator (!)
function processUser(user: User | null) {
  console.log(user!.name);  // "!" tells compiler: "I know this is not null"
  // Use with caution - bypasses type checking!
}

// Optional chaining (added in later TypeScript versions)
function getUserEmail(user: User | null): string | undefined {
  return user?.email;  // Returns undefined if user is null
}
```

## Why this is better

- **Prevents null reference errors**: Catches potential crashes at compile time
- **Explicit nullability**: Forces developers to handle null/undefined cases
- **Better code quality**: More defensive programming
- **Self-documenting**: Type signatures clearly show when null is possible
- **Safer refactoring**: Type system ensures null checks aren't forgotten

## Key notes / edge cases

- Enable with `"strictNullChecks": true` in tsconfig.json (part of `"strict": true`)
- Optional properties have `undefined` in their type automatically:
  ```typescript
  interface User {
    name: string;
    email?: string;  // Type is actually string | undefined
  }
  ```

- The non-null assertion operator (`!`) should be used sparingly as it bypasses type checking:
  ```typescript
  let value: string | null = getValue();
  console.log(value!.toUpperCase());  // Dangerous if value is actually null!
  ```

- Type guards help narrow types:
  ```typescript
  function printName(name: string | null) {
    if (name !== null) {
      console.log(name.toUpperCase());  // TypeScript knows name is string here
    }
  }
  ```

- Nullish coalescing operator (`??`) provides defaults:
  ```typescript
  let name: string | null = null;
  let displayName = name ?? "Guest";  // "Guest" if name is null/undefined
  ```

## Quick practice

1. **Fix the type error**: This code has strictNullChecks enabled. How do you fix it?
   ```typescript
   function greet(name: string) {
     return `Hello, ${name.toUpperCase()}`;
   }
   
   let userName: string | null = getUserName();
   console.log(greet(userName));
   ```

2. **Handle nullability**: Write a function `getStringLength` that accepts `string | null` and returns the length or 0 if null.

3. **Optional vs nullable**: What's the difference between these two types?
   ```typescript
   type A = { name: string | null };
   type B = { name?: string };
   ```

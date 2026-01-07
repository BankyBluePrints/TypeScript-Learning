# Basic Types

## What it is

TypeScript provides several basic types for annotating variables, function parameters, and return values. These primitive types include `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`, and special types like `any`, `unknown`, `void`, and `never`.

## Before TypeScript (Plain JavaScript)

In JavaScript, types are implicit and checked only at runtime:

```javascript
// JavaScript - types are dynamic
let username = "Alice";        // string (implicitly)
username = 42;                 // now a number - no error
username = true;               // now a boolean - no error

function greet(name) {
  // What type is name? We don't know until runtime
  return "Hello, " + name.toUpperCase();
}

greet("Bob");           // Works
greet(123);            // Runtime error: name.toUpperCase is not a function
```

## With TypeScript (Explicit Types)

TypeScript requires explicit type annotations (or infers them):

```typescript
// Basic type annotations
let username: string = "Alice";
let age: number = 30;
let isActive: boolean = true;
let notDefined: undefined = undefined;
let empty: null = null;

username = 42;  // ✗ Error: Type 'number' is not assignable to type 'string'

// Function with typed parameters and return type
function greet(name: string): string {
  return "Hello, " + name.toUpperCase();
}

greet("Bob");     // ✓ Works
greet(123);       // ✗ Error: Argument of type 'number' is not assignable to parameter of type 'string'
```

## All Basic Types with Examples

```typescript
// Primitive types
let str: string = "Hello";
let num: number = 42;
let bool: boolean = true;
let undef: undefined = undefined;
let nul: null = null;
let sym: symbol = Symbol("key");
let big: bigint = 100n;

// Arrays
let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ["a", "b", "c"];  // Generic syntax

// Tuple - fixed-length array with specific types
let tuple: [string, number] = ["Alice", 30];
tuple[0].toUpperCase();  // OK - TypeScript knows it's a string
tuple[1].toFixed(2);     // OK - TypeScript knows it's a number

// Enum - named constants
enum Color {
  Red,
  Green,
  Blue
}
let c: Color = Color.Green;

// Any - opt-out of type checking (avoid when possible!)
let anything: any = 4;
anything = "string";     // OK
anything = true;         // OK
anything.foo.bar.baz;    // No error, but might crash at runtime

// Unknown - type-safe version of any (requires type checking)
let uncertain: unknown = 4;
uncertain = "string";                    // OK to assign
// uncertain.toUpperCase();              // ✗ Error: Object is of type 'unknown'
if (typeof uncertain === "string") {
  uncertain.toUpperCase();               // ✓ OK inside type guard
}

// Void - absence of a return value
function logMessage(message: string): void {
  console.log(message);
  // No return statement, or return; without a value
}

// Never - function that never returns (throws or infinite loop)
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {
    // Never exits
  }
}

// Object type (non-primitive)
let obj: object = { name: "Alice" };
obj = {};         // OK
obj = [];         // OK (arrays are objects)
// obj = 42;      // ✗ Error: Type 'number' is not assignable to type 'object'
```

## Why This Is Better

- **Early error detection**: Type errors caught during development
- **Better autocomplete**: IDEs know what methods/properties are available
- **Self-documenting**: Types clarify what values are expected
- **Refactoring safety**: Changing types updates all usages
- **Prevents common bugs**: No more `undefined is not a function` surprises

## Common Mistake + Fix

**Mistake**: Overusing `any` type, which defeats the purpose of TypeScript

```typescript
// Bad - loses all type safety
function processData(data: any): any {
  return data.value.toUpperCase();  // No type checking, might crash
}

let result: any = processData(someValue);
result.foo.bar.baz();  // Compiles but might crash at runtime
```

**Fix**: Use specific types, or `unknown` if the type truly isn't known initially

```typescript
// Better - specific types
function processData(data: { value: string }): string {
  return data.value.toUpperCase();
}

// Or use unknown with type guards
function processSafely(data: unknown): string {
  if (typeof data === "object" && data !== null && "value" in data) {
    const obj = data as { value: unknown };
    if (typeof obj.value === "string") {
      return obj.value.toUpperCase();
    }
  }
  throw new Error("Invalid data format");
}
```

## Key Notes / Edge Cases

- **Type inference**: TypeScript can infer types, so annotations aren't always needed:
  ```typescript
  let x = 42;        // TypeScript infers x: number
  let y = "hello";   // TypeScript infers y: string
  ```

- **Null and undefined**: With `strictNullChecks` enabled (part of `strict`), null and undefined are separate types:
  ```typescript
  let name: string = null;      // ✗ Error with strictNullChecks
  let name: string | null = null;  // ✓ OK
  ```

- **Type widening**: Literal types can widen:
  ```typescript
  let x = "hello";   // Type: string (widened from literal "hello")
  const y = "hello"; // Type: "hello" (literal type preserved)
  ```

- **bigint vs number**: These are incompatible types:
  ```typescript
  let a: number = 42;
  let b: bigint = 42n;
  // a = b;  // ✗ Error: Type 'bigint' is not assignable to type 'number'
  ```

## Quick Practice

1. **Fix the types**: Add correct type annotations to make this code type-safe:
   ```typescript
   function calculateArea(width, height) {
     return width * height;
   }
   
   let result = calculateArea(10, 20);
   ```

2. **Choose the right type**: Which type should you use for a function that:
   - Logs to console but doesn't return anything?
   - Always throws an error and never returns?
   - Returns different types based on input?

3. **Debug this code**: Why does this fail with `strictNullChecks` enabled?
   ```typescript
   function getLength(str: string): number {
     return str.length;
   }
   
   let name: string = getUserName();  // might return null
   console.log(getLength(name));
   ```
   How would you fix it?

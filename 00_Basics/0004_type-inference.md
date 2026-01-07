# Type Inference

## What it is

Type inference is TypeScript's ability to automatically determine the type of a variable, function return value, or expression without explicit type annotations. The compiler analyzes the code and infers the most specific type possible based on the assigned value or usage context.

## Without Type Inference (Explicit Everything)

If TypeScript didn't have type inference, you'd need to annotate every single variable and expression:

```typescript
// Hypothetically without inference - very verbose
let name: string = "Alice";
let age: number = 30;
let isActive: boolean = true;
let numbers: number[] = [1, 2, 3];

function add(a: number, b: number): number {
  let result: number = a + b;
  return result;
}

let sum: number = add(5, 10);
```

## With Type Inference (TypeScript's Smart Defaults)

TypeScript infers types automatically, reducing verbosity while maintaining type safety:

```typescript
// TypeScript infers all these types automatically
let name = "Alice";              // inferred as string
let age = 30;                    // inferred as number
let isActive = true;             // inferred as boolean
let numbers = [1, 2, 3];         // inferred as number[]

function add(a: number, b: number) {
  let result = a + b;            // inferred as number
  return result;                 // function return type inferred as number
}

let sum = add(5, 10);            // inferred as number
```

Hover over any variable in an IDE to see the inferred type!

## How Type Inference Works

```typescript
// 1. From initialization value
let x = 42;                    // Type: number
let y = "hello";               // Type: string
let z = [1, 2, 3];            // Type: number[]
let mixed = [1, "two", true]; // Type: (string | number | boolean)[]

// 2. From function return
function getUser() {
  return { name: "Alice", age: 30 };
}
let user = getUser();  // Type: { name: string; age: number; }

// 3. From context (contextual typing)
window.addEventListener("click", (event) => {
  // 'event' type is inferred as MouseEvent based on "click"
  console.log(event.clientX, event.clientY);
});

const numbers = [1, 2, 3];
numbers.forEach((num) => {
  // 'num' is inferred as number
  console.log(num.toFixed(2));
});

// 4. Best common type
let items = [1, 2, "three"];   // Type: (string | number)[]
let values = [null, 42, 100];  // Type: (number | null)[]

// 5. Const vs let
let x = "hello";     // Type: string (widened)
const y = "hello";   // Type: "hello" (literal type preserved)
```

## Why This Is Better

- **Less verbose code**: Write less while maintaining type safety
- **Natural JavaScript feel**: Code looks like JavaScript but with type checking
- **Smart defaults**: TypeScript chooses the most specific type possible
- **Reduced maintenance**: Changing a value automatically updates the type
- **Better for refactoring**: Types flow through the codebase automatically

## When to Use Explicit Types

Sometimes explicit annotations are better than inference:

```typescript
// 1. When inference is too wide
let userId;              // Type: any (no initialization)
userId = "123";          // Still type: any
// Better:
let userId: string;      // Explicitly constrained

// 2. Function parameters (always annotate!)
function greet(name) {   // 'name' implicitly has 'any' type - BAD
  return `Hello, ${name}`;
}
// Better:
function greet(name: string) {
  return `Hello, ${name}`;
}

// 3. When you want a more general type
let value = "fixed";     // Type: "fixed" (literal, too specific)
let value: string = "fixed";  // Type: string (more flexible)

// 4. Public APIs (explicit for clarity)
// Good for library/API functions
export function calculateTax(amount: number, rate: number): number {
  return amount * rate;
}

// 5. Complex types that improve readability
type UserRole = "admin" | "user" | "guest";
let role: UserRole = "admin";  // Clearer than letting inference handle it
```

## Common Mistake + Fix

**Mistake**: Letting TypeScript infer `any` by not initializing variables

```typescript
// Bad - TypeScript infers 'any'
let data;
// ... many lines later
data = fetchFromAPI();  // data is still 'any'
data.foo.bar.baz;       // No type checking!
```

**Fix**: Always initialize variables or provide explicit types

```typescript
// Good - explicit type
let data: ApiResponse;
data = fetchFromAPI();  // Type is checked

// Or initialize with a value
let data = fetchFromAPI();  // Type inferred from function return
```

## Key Notes / Edge Cases

- **Return type inference**: TypeScript infers function return types, but it's often good practice to explicitly annotate public functions:
  ```typescript
  // Inferred return type
  function add(a: number, b: number) {
    return a + b;  // Return type inferred as number
  }
  
  // Explicit return type (better for public APIs)
  function add(a: number, b: number): number {
    return a + b;
  }
  ```

- **Literal types with const**: Constants preserve literal types:
  ```typescript
  let x = "hello";        // Type: string
  const y = "hello";      // Type: "hello"
  
  let obj = { name: "Alice" };  // Type: { name: string }
  const obj2 = { name: "Alice" } as const;  // Type: { readonly name: "Alice" }
  ```

- **Type widening**: TypeScript widens types to make them more usable:
  ```typescript
  let x = null;     // Type: any (too wide!)
  let y: string | null = null;  // Better
  ```

- **noImplicitAny**: With this compiler option (part of `strict`), TypeScript errors on inferred `any` types:
  ```typescript
  // With noImplicitAny enabled
  function greet(name) {  // ✗ Error: Parameter 'name' implicitly has an 'any' type
    return `Hello, ${name}`;
  }
  ```

## Quick Practice

1. **Predict the types**: What type does TypeScript infer for each variable?
   ```typescript
   let a = 42;
   let b = "hello";
   let c = [1, 2, 3];
   let d = [1, "two"];
   const e = "fixed";
   let f;
   ```

2. **Fix the inference**: This code has poor type inference. How would you improve it?
   ```typescript
   function processUser(data) {
     return {
       id: data.id,
       name: data.name.toUpperCase()
     };
   }
   ```

3. **Contextual typing challenge**: Why does this work without explicit types?
   ```typescript
   [1, 2, 3].map(num => num * 2);
   // How does TypeScript know 'num' is a number?
   ```

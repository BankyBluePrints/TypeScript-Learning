# Functions

## What it is

Functions in TypeScript allow you to specify types for parameters, return values, and the function itself. This includes standard functions, arrow functions, optional/default parameters, rest parameters, function overloads, and function types. TypeScript ensures type safety throughout the function signature and body.

## Before TypeScript (Plain JavaScript)

JavaScript functions have no type constraints:

```javascript
// JavaScript - no type safety
function calculatePrice(price, taxRate, discount) {
  return price * (1 + taxRate) - discount;
}

calculatePrice(100, 0.2, 10);        // 110 - correct
calculatePrice("100", "0.2", "10");  // "100.21010" - string concatenation!
calculatePrice(100);                 // NaN - missing arguments
calculatePrice(100, 0.2, 10, 999);   // 110 - extra argument ignored
```

## With TypeScript (Type-Safe Functions)

TypeScript enforces type safety on all function parameters and return values:

```typescript
// Basic function with typed parameters and return type
function calculatePrice(price: number, taxRate: number, discount: number): number {
  return price * (1 + taxRate) - discount;
}

calculatePrice(100, 0.2, 10);        // ✓ 110
calculatePrice("100", 0.2, 10);      // ✗ Error: Argument of type 'string' is not assignable
calculatePrice(100);                 // ✗ Error: Expected 3 arguments, but got 1
calculatePrice(100, 0.2, 10, 999);   // ✗ Error: Expected 3 arguments, but got 4

// Arrow function syntax
const add = (a: number, b: number): number => {
  return a + b;
};

// Concise arrow function (return type inferred)
const multiply = (a: number, b: number) => a * b;  // Return type: number
```

## Function Features in TypeScript

```typescript
// 1. Optional parameters (use ?)
function greet(name: string, greeting?: string): string {
  if (greeting) {
    return `${greeting}, ${name}!`;
  }
  return `Hello, ${name}!`;
}

greet("Alice");              // ✓ "Hello, Alice!"
greet("Bob", "Good morning"); // ✓ "Good morning, Bob!"

// 2. Default parameters
function createUser(name: string, role: string = "user"): object {
  return { name, role };
}

createUser("Alice");           // { name: "Alice", role: "user" }
createUser("Bob", "admin");    // { name: "Bob", role: "admin" }

// 3. Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((total, num) => total + num, 0);
}

sum(1, 2, 3);        // 6
sum(1, 2, 3, 4, 5); // 15

// 4. Function type annotations
let mathOp: (a: number, b: number) => number;

mathOp = (x, y) => x + y;  // ✓ OK
mathOp = (x, y) => x - y;  // ✓ OK
// mathOp = (x: string) => x.length;  // ✗ Error: type mismatch

// 5. Void return type (no return value)
function logMessage(message: string): void {
  console.log(message);
  // No return statement needed
}

// 6. Never return type (never returns)
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) { }
}

// 7. Function overloads (multiple signatures)
function format(value: string): string;
function format(value: number): string;
function format(value: boolean): string;
function format(value: string | number | boolean): string {
  return String(value);
}

format("hello");   // ✓ OK
format(42);        // ✓ OK
format(true);      // ✓ OK
// format([]);     // ✗ Error: No overload matches

// 8. this parameter type
interface User {
  name: string;
  greet(this: User): void;
}

const user: User = {
  name: "Alice",
  greet() {
    console.log(`Hello, I'm ${this.name}`);
  }
};

user.greet();  // OK
const greetFn = user.greet;
// greetFn();  // ✗ Error: The 'this' context of type 'void' is not assignable to method's 'this' of type 'User'
```

## Callback and Higher-Order Functions

```typescript
// Callback with function type
function processArray(arr: number[], callback: (item: number) => void): void {
  arr.forEach(callback);
}

processArray([1, 2, 3], (num) => {
  console.log(num * 2);  // num is inferred as number
});

// Higher-order function returning a function
function multiplier(factor: number): (num: number) => number {
  return (num) => num * factor;
}

const double = multiplier(2);
const triple = multiplier(3);
console.log(double(5));  // 10
console.log(triple(5));  // 15
```

## Why This Is Better

- **Prevent argument errors**: Wrong number or type of arguments caught at compile time
- **Clear contracts**: Function signatures document expected inputs and outputs
- **Better autocomplete**: IDEs suggest correct parameter types
- **Safer refactoring**: Changing function signatures updates all call sites
- **Self-documenting**: Types serve as inline documentation
- **Optional/default params**: Flexible function signatures with type safety

## Common Mistake + Fix

**Mistake**: Forgetting that optional parameters must come after required ones

```typescript
// Bad - optional parameter before required one
function createUser(role?: string, name: string) {  // ✗ Error: A required parameter cannot follow an optional parameter
  return { name, role };
}
```

**Fix**: Put optional parameters last

```typescript
// Good - optional parameter comes after required ones
function createUser(name: string, role?: string) {
  return { name, role: role || "user" };
}

// Or use default parameters (can be anywhere)
function createUser(name: string, role: string = "user") {
  return { name, role };
}
```

## Key Notes / Edge Cases

- **Type inference on return**: TypeScript infers return types, but explicit annotation is often clearer for public APIs
  ```typescript
  // Inferred
  function add(a: number, b: number) {
    return a + b;  // Return type inferred as number
  }
  
  // Explicit (better for exported/public functions)
  export function add(a: number, b: number): number {
    return a + b;
  }
  ```

- **Never vs void**: `void` means "doesn't return a value"; `never` means "never returns"
  ```typescript
  function log(): void {
    console.log("Done");
  }  // Function completes normally
  
  function fail(): never {
    throw new Error("Failed");
  }  // Function never completes
  ```

- **Function overloads order matters**: More specific overloads should come before general ones
  ```typescript
  function process(value: string): string;
  function process(value: any): any;  // More general - comes last
  function process(value: any): any {
    return value;
  }
  ```

- **Rest parameters must be array type**:
  ```typescript
  function sum(...nums: number[]): number {  // ✓ Correct
    return nums.reduce((a, b) => a + b, 0);
  }
  ```

## Quick Practice

1. **Add types**: Type this function properly:
   ```typescript
   function formatUser(firstName, lastName, age, email) {
     return {
       fullName: `${firstName} ${lastName}`,
       age: age,
       contact: email
     };
   }
   ```

2. **Fix the function**: This has a type error. What's wrong and how do you fix it?
   ```typescript
   function divide(a: number, b: number): number {
     if (b === 0) {
       throw new Error("Division by zero");
     }
     return a / b;
   }
   ```
   Should the return type be `number` or `never`? Why?

3. **Create an overload**: Write function overloads for a `format` function that:
   - Takes a Date and returns a formatted date string
   - Takes a number and returns a formatted currency string
   - Implement the function body that handles both cases

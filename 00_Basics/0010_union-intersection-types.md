# Union and Intersection Types

## What it is

Union types (`|`) allow a value to be one of several types, while intersection types (`&`) combine multiple types into one. These type operators enable flexible, precise type definitions for complex scenarios where values can have multiple possible types or must satisfy multiple type constraints.

## Before Union/Intersection (Imprecise Types)

Without these operators, you'd use `any` or awkward workarounds:

```typescript
// Without unions - forced to use 'any' or type assertions
function printId(id: any) {  // Too permissive!
  console.log(id);
}

printId(123);      // OK
printId("ABC123"); // OK
printId(true);     // OK but probably wrong!
printId({});       // OK but definitely wrong!

// Or create overloads (verbose)
function printId(id: number): void;
function printId(id: string): void;
function printId(id: number | string): void {
  console.log(id);
}
```

## With Union Types (Flexible and Type-Safe)

Union types allow a value to be one of several specific types:

```typescript
// Union type: value can be string OR number
function printId(id: string | number): void {
  console.log(id);
}

printId(123);        // ✓ OK
printId("ABC123");   // ✓ OK
// printId(true);    // ✗ Error: Argument of type 'boolean' is not assignable
// printId({});      // ✗ Error: Argument of type '{}' is not assignable

// Type narrowing required to access type-specific properties
function formatId(id: string | number): string {
  if (typeof id === "string") {
    return id.toUpperCase();  // TypeScript knows id is string here
  } else {
    return id.toFixed(2);     // TypeScript knows id is number here
  }
}

// Union with literal types
type Status = "pending" | "approved" | "rejected";

function updateStatus(status: Status): void {
  console.log(`Status: ${status}`);
}

updateStatus("pending");    // ✓ OK
updateStatus("approved");   // ✓ OK
// updateStatus("unknown"); // ✗ Error: not a valid Status value

// Union with custom types
type SuccessResponse = {
  status: "success";
  data: any;
};

type ErrorResponse = {
  status: "error";
  message: string;
};

type ApiResponse = SuccessResponse | ErrorResponse;

function handleResponse(response: ApiResponse): void {
  if (response.status === "success") {
    console.log(response.data);  // TypeScript knows it's SuccessResponse
  } else {
    console.log(response.message);  // TypeScript knows it's ErrorResponse
  }
}
```

## With Intersection Types (Combine Multiple Types)

Intersection types combine multiple types into one that has all properties:

```typescript
// Intersection: value must satisfy ALL types
type Person = {
  name: string;
  age: number;
};

type Employee = {
  employeeId: string;
  department: string;
};

type StaffMember = Person & Employee;  // Has all properties from both

const staff: StaffMember = {
  name: "Alice",
  age: 30,
  employeeId: "E123",
  department: "Engineering"
  // Must have ALL properties
};

// Intersection with interfaces
interface Printable {
  print(): void;
}

interface Saveable {
  save(): void;
}

type Document = Printable & Saveable;

const doc: Document = {
  print() { console.log("Printing..."); },
  save() { console.log("Saving..."); }
  // Must implement both methods
};

// Combining with type aliases
type Timestamped = {
  createdAt: Date;
  updatedAt: Date;
};

type User = {
  id: string;
  name: string;
} & Timestamped;  // User has id, name, createdAt, updatedAt
```

## Complex Type Combinations

```typescript
// Union of intersections
type Admin = Person & { role: "admin"; permissions: string[] };
type Guest = Person & { role: "guest" };
type User = Admin | Guest;  // Can be either Admin or Guest

function greetUser(user: User): void {
  console.log(`Hello, ${user.name}`);
  
  if (user.role === "admin") {
    console.log(`Permissions: ${user.permissions.join(", ")}`);
  }
}

// Intersection with conditional properties
type BaseConfig = {
  apiUrl: string;
};

type ProductionConfig = BaseConfig & {
  environment: "production";
  cacheEnabled: true;
};

type DevelopmentConfig = BaseConfig & {
  environment: "development";
  debugMode: true;
};

type Config = ProductionConfig | DevelopmentConfig;

// Array unions
type StringOrNumber = string | number;
type MixedArray = (string | number)[];  // Array of strings or numbers

const mixed: MixedArray = [1, "two", 3, "four"];

// Function unions
type Logger = (message: string) => void;
type Validator = (input: any) => boolean;
type Utility = Logger | Validator;
```

## Type Narrowing with Unions

```typescript
// Type guards for narrowing unions
function processValue(value: string | number | boolean): void {
  // typeof guard
  if (typeof value === "string") {
    console.log(value.toUpperCase());
  } else if (typeof value === "number") {
    console.log(value.toFixed(2));
  } else {
    console.log(value ? "yes" : "no");
  }
}

// instanceof guard
class Dog {
  bark() { console.log("Woof!"); }
}

class Cat {
  meow() { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat): void {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}

// Discriminated unions (tagged unions)
type Circle = {
  kind: "circle";
  radius: number;
};

type Rectangle = {
  kind: "rectangle";
  width: number;
  height: number;
};

type Shape = Circle | Rectangle;

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
  }
}
```

## Why This Is Better

- **Precise types**: Express exactly what values are valid
- **Type safety**: Compiler enforces correct usage
- **Better autocomplete**: IDE knows available properties/methods
- **Self-documenting**: Types clearly show what's allowed
- **Exhaustive checking**: TypeScript ensures all cases are handled
- **Flexibility**: Combine types in powerful ways

## Common Mistake + Fix

**Mistake**: Trying to access union type properties without narrowing

```typescript
function printLength(value: string | number): void {
  console.log(value.length);  // ✗ Error: Property 'length' does not exist on type 'number'
}
```

**Fix**: Use type narrowing before accessing type-specific properties

```typescript
function printLength(value: string | number): void {
  if (typeof value === "string") {
    console.log(value.length);  // ✓ OK - TypeScript knows it's a string
  } else {
    console.log("Not a string");
  }
}
```

## Key Notes / Edge Cases

- **Union vs intersection naming**: Union is "OR" (`|`), intersection is "AND" (`&`)

- **Never type in intersections**: Incompatible intersections result in `never`:
  ```typescript
  type Impossible = string & number;  // Type: never (can't be both)
  ```

- **Union distribution**: Unions distribute over conditional types:
  ```typescript
  type ToArray<T> = T extends any ? T[] : never;
  type Result = ToArray<string | number>;  // string[] | number[]
  ```

- **Order doesn't matter**:
  ```typescript
  type A = string | number;
  type B = number | string;  // Same as A
  ```

- **Intersection can create unexpected types**:
  ```typescript
  type A = { x: string };
  type B = { x: number };
  type C = A & B;  // { x: never } - x can't be both string and number
  ```

## Quick Practice

1. **Create a union type**: Define a `Result` type that can be either:
   - `{ success: true; data: string }`
   - `{ success: false; error: string }`
   Write a function that handles both cases.

2. **Fix the type error**: Why does this fail, and how do you fix it?
   ```typescript
   function double(value: string | number) {
     return value * 2;
   }
   ```

3. **Intersection challenge**: Create a type `Manager` that combines:
   - `Person` (name, age)
   - `Employee` (employeeId, department)
   - `Leader` (teamSize: number)
   Then create a valid Manager object.

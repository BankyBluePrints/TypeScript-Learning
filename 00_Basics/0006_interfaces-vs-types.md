# Interfaces vs Types

## What it is

Both `interface` and `type` in TypeScript allow you to define the shape of an object, but they have different capabilities and use cases. Understanding when to use each is important for writing idiomatic TypeScript code.

## Before (Plain Objects with No Shape Definition)

In JavaScript, object shapes are implicit and not enforced:

```javascript
// JavaScript - no shape enforcement
const user = {
  name: "Alice",
  age: 30
};

function printUser(user) {
  console.log(user.name, user.age);
  // Might crash if properties don't exist
}

printUser({ name: "Bob" });  // Missing 'age' - runtime error
printUser({ username: "Charlie", age: 25 });  // Wrong property name - runtime error
```

## With Interfaces

Interfaces define contracts for object shapes:

```typescript
// Define the shape
interface User {
  name: string;
  age: number;
  email?: string;  // optional property
}

// Enforce the shape
const user: User = {
  name: "Alice",
  age: 30
};  // ✓ OK

const badUser: User = {
  name: "Bob"
  // ✗ Error: Property 'age' is missing
};

function printUser(user: User): void {
  console.log(user.name, user.age);
}

printUser({ name: "Charlie", age: 25 });  // ✓ OK
```

## With Type Aliases

Type aliases can define the same shapes and more:

```typescript
// Object type
type User = {
  name: string;
  age: number;
  email?: string;
};

// Works exactly like interface for objects
const user: User = {
  name: "Alice",
  age: 30
};

// But types can also define primitives, unions, tuples, etc.
type ID = string | number;
type Point = [number, number];
type Callback = (data: string) => void;
```

## Key Differences

```typescript
// 1. EXTENDING/INHERITANCE

// Interface: uses 'extends'
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

const myDog: Dog = {
  name: "Buddy",
  breed: "Golden Retriever"
};

// Type: uses intersection (&)
type Animal2 = {
  name: string;
};

type Dog2 = Animal2 & {
  breed: string;
};

// 2. DECLARATION MERGING (only interfaces!)

// Interfaces with the same name merge automatically
interface Window {
  title: string;
}

interface Window {
  version: number;
}

// Result: Window has both title and version
const w: Window = {
  title: "My App",
  version: 1
};

// Types cannot be merged - redeclaration is an error
type Config = { theme: string };
// type Config = { version: number };  // ✗ Error: Duplicate identifier

// 3. WHAT THEY CAN REPRESENT

// Types can represent primitives, unions, tuples
type StringOrNumber = string | number;
type Point = [number, number];
type Callback = () => void;

// Interfaces can only represent object shapes
// interface StringOrNumber = string | number;  // ✗ Error: not allowed

// 4. COMPUTED PROPERTIES

// Types can use computed properties more flexibly
type Keys = "name" | "age";
type UserRecord = {
  [K in Keys]: string;
};

// Interfaces use index signatures
interface Dictionary {
  [key: string]: any;
}

// 5. TYPE PRIMITIVES AND UTILITY TYPES

type ReadonlyUser = Readonly<User>;  // Utility type
type PartialUser = Partial<User>;    // Utility type
type UserKeys = keyof User;           // "name" | "age" | "email"

// Interfaces don't work as well with utility types
```

## When to Use Which

```typescript
// Use INTERFACE when:
// 1. Defining object/class shapes
interface Person {
  name: string;
  age: number;
}

// 2. When you need declaration merging (e.g., extending global types)
interface Array<T> {
  customMethod(): void;
}

// 3. When defining public API contracts (convention)
export interface ApiResponse {
  data: any;
  status: number;
}

// Use TYPE when:
// 1. Defining unions or intersections
type Status = "pending" | "approved" | "rejected";
type Response = SuccessResponse | ErrorResponse;

// 2. Defining tuples
type Point = [number, number];
type RGB = [number, number, number];

// 3. Using mapped types or complex type transformations
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

// 4. Aliasing primitive types
type ID = string;
type Count = number;

// 5. Defining function types
type Comparator = (a: number, b: number) => number;
```

## Why This Is Better

- **Flexibility**: Choose the right tool for the job
- **Type safety**: Both enforce structure at compile time
- **Code organization**: Clear contracts between modules
- **Reusability**: Define once, use everywhere
- **Maintainability**: Change definition in one place

## Common Mistake + Fix

**Mistake**: Using type when interface would be more semantic, or vice versa

```typescript
// Less ideal - type for simple object shape
type User = {
  name: string;
  age: number;
};

// Less ideal - interface for union type (doesn't work!)
// interface Status = "active" | "inactive";  // ✗ Error
```

**Fix**: Use interface for object shapes, type for everything else

```typescript
// Better - interface for object shapes
interface User {
  name: string;
  age: number;
}

// Type for unions and complex types
type Status = "active" | "inactive";
type ID = string | number;
```

## Key Notes / Edge Cases

- **Performance**: Both interfaces and types have identical runtime performance (both compile away). There's no performance reason to prefer one over the other.

- **Classes can implement both**:
  ```typescript
  interface IPerson {
    name: string;
  }
  
  type TPerson = {
    name: string;
  };
  
  class Person implements IPerson { // ✓ OK
    name: string;
  }
  
  class Person2 implements TPerson { // ✓ Also OK
    name: string;
  }
  ```

- **Intersection vs extends behavior**:
  ```typescript
  interface A {
    x: string;
  }
  
  interface B extends A {
    x: number;  // ✗ Error: incompatible
  }
  
  type C = {
    x: string;
  };
  
  type D = C & {
    x: number;  // Creates type: { x: never } - usually not what you want
  };
  ```

- **Declaration merging use case**: Augmenting third-party library types
  ```typescript
  // Extending Express Request type
  declare global {
    namespace Express {
      interface Request {
        user?: User;
      }
    }
  }
  ```

## Quick Practice

1. **Choose the right tool**: For each scenario, should you use interface or type?
   - Defining the shape of a user object
   - Creating a union of "success" | "error" | "pending"
   - Defining a function signature for a callback
   - Extending a third-party library's global types
   - Creating a tuple type for coordinates

2. **Refactor**: Convert this type to an interface (if possible):
   ```typescript
   type Config = {
     theme: "light" | "dark";
     fontSize: number;
   } & {
     language: string;
   };
   ```

3. **Debug**: Why doesn't this work, and how would you fix it?
   ```typescript
   interface Status = "active" | "inactive";
   ```

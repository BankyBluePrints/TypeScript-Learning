# Modules (TypeScript 1.0)

## What it is

TypeScript 1.0 introduced module support using both internal modules (namespaces) and external modules (CommonJS/AMD). This allowed developers to organize code into separate files with explicit dependencies and avoid global namespace pollution.

## Before this feature

Before modules, all code shared the global namespace:

```javascript
// file1.js
var currentUser = "Alice";

function getUser() {
  return currentUser;
}

// file2.js
// Accessing variables from file1.js implicitly
console.log(currentUser);  // Works if file1 loaded first
console.log(getUser());    // Works if file1 loaded first

// Problems: name collisions, load order dependencies, no encapsulation
```

## After this feature

TypeScript 1.0 enabled explicit module exports and imports:

```typescript
// user.ts - external module
export interface User {
  name: string;
  age: number;
}

export function getUser(): User {
  return { name: "Alice", age: 30 };
}

// Internal (not exported)
function validateUser(user: User): boolean {
  return user.age >= 0;
}

// app.ts - consuming module
import { User, getUser } from "./user";

const user: User = getUser();
console.log(user);
// console.log(validateUser(user));  // ✗ Error: not exported

// Namespace (internal modules)
namespace Validation {
  export interface StringValidator {
    isValid(s: string): boolean;
  }
  
  export class EmailValidator implements StringValidator {
    isValid(s: string): boolean {
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(s);
    }
  }
}

const validator = new Validation.EmailValidator();
```

## Why this is better

- **Encapsulation**: Only exported items are public
- **No global pollution**: Each module has its own scope
- **Clear dependencies**: Imports show exactly what's needed
- **Code organization**: Logical grouping of related functionality
- **Reusability**: Modules can be shared across projects
- **Type safety**: Import types along with values

## Key notes / edge cases

- TypeScript 1.0 supported both CommonJS and AMD module systems
- External modules (now just called modules) are preferred over namespaces
- Files with `export` or `import` are automatically treated as modules
- Namespace (internal modules) are still supported but less common
- Module resolution depends on compiler settings in tsconfig.json
- Circular dependencies can cause initialization issues

```typescript
// Module with multiple exports
export const API_URL = "https://api.example.com";
export type UserId = string;

export function fetchUser(id: UserId): Promise<User> {
  // Implementation
}

export default class Application {
  // Default export
}

// Importing
import Application, { API_URL, fetchUser } from "./app";
import * as AppModule from "./app";  // Import all as namespace
```

## Quick practice

1. **Create a module**: Build a `math.ts` module that exports functions `add`, `subtract`, and a constant `PI`. Then import and use them in another file.

2. **Export types**: Create a module that exports an interface `Product` and a function `createProduct` that returns a `Product`.

3. **Understanding scope**: If you have a function in a module that's not exported, can you access it from another file? Why or why not?

# Modules - Import and Export

## What it is

TypeScript uses ES6 module syntax (`import` and `export`) to organize code into separate files and namespaces. Modules allow you to split your application into smaller, reusable pieces, control what's public vs private, and manage dependencies explicitly.

## Before Modules (Global Scope Pollution)

Without modules, all code shared a global namespace:

```javascript
// user.js
var currentUser = {
  name: "Alice",
  age: 30
};

function getUser() {
  return currentUser;
}

// app.js
console.log(currentUser);  // Accessible globally
console.log(getUser());    // Accessible globally

// Problems:
// 1. Name collisions - what if another file defines 'currentUser'?
// 2. No encapsulation - everything is public
// 3. Hard to track dependencies - what does this file need?
// 4. Load order matters - scripts must be in correct order
```

## With ES6 Modules (TypeScript)

Modules create isolated scopes with explicit imports and exports:

```typescript
// user.ts
export interface User {
  name: string;
  age: number;
}

export const currentUser: User = {
  name: "Alice",
  age: 30
};

export function getUser(): User {
  return currentUser;
}

// Private - not exported, not accessible outside this file
function validateUser(user: User): boolean {
  return user.age >= 0;
}

// app.ts
import { User, currentUser, getUser } from "./user";

console.log(currentUser);
console.log(getUser());
// console.log(validateUser(currentUser));  // ✗ Error: not exported
```

## Export Syntax Variations

```typescript
// 1. Named exports (multiple per file)
export const API_URL = "https://api.example.com";
export type UserId = string;

export function fetchUser(id: UserId) {
  // ...
}

export class UserService {
  // ...
}

// 2. Export statement (separate from declaration)
const CONFIG = { timeout: 5000 };
function helper() { }

export { CONFIG, helper };

// 3. Rename on export
const internalFunction = () => { };
export { internalFunction as publicFunction };

// 4. Re-export from another module
export { User, getUser } from "./user";
export * from "./types";  // Re-export everything

// 5. Default export (one per file)
export default class Application {
  // Main class
}

// Or
class App { }
export default App;

// 6. Mixed default and named exports
export default function main() { }
export const VERSION = "1.0.0";
export type Config = { /* ... */ };
```

## Import Syntax Variations

```typescript
// 1. Named imports
import { User, getUser } from "./user";
import { API_URL, fetchUser } from "./api";

// 2. Rename on import
import { User as UserType } from "./user";
import { default as UserClass } from "./userClass";

// 3. Import everything as namespace
import * as UserModule from "./user";
UserModule.getUser();
UserModule.currentUser;

// 4. Default import
import Application from "./app";  // No braces for default
import React from "react";

// 5. Mixed default and named imports
import React, { useState, useEffect } from "react";

// 6. Side-effect only import (no imports)
import "./styles.css";
import "reflect-metadata";

// 7. Type-only imports (TypeScript specific)
import type { User } from "./user";  // Only imports the type, not runtime code
import { type User, getUser } from "./user";  // Mixed type and value import

// 8. Dynamic imports (async)
async function loadModule() {
  const module = await import("./heavy-module");
  module.doSomething();
}
```

## Module Resolution

```typescript
// Relative imports (your own files)
import { User } from "./user";           // Same directory
import { Helper } from "../utils/helper"; // Parent directory
import { Config } from "./config/index";  // Directory with index.ts

// Non-relative imports (packages from node_modules)
import express from "express";
import { map } from "lodash";
import React from "react";

// TypeScript finds these based on moduleResolution in tsconfig.json
```

## Organizing a TypeScript Project

```typescript
// src/types/user.ts
export interface User {
  id: string;
  name: string;
  email: string;
}

export type UserRole = "admin" | "user" | "guest";

// src/services/userService.ts
import type { User } from "../types/user";

export class UserService {
  async getUser(id: string): Promise<User> {
    // Implementation
  }
}

// src/utils/validators.ts
export function isValidEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

// src/index.ts (entry point)
import { UserService } from "./services/userService";
import { isValidEmail } from "./utils/validators";
import type { User } from "./types/user";

const userService = new UserService();
// Use services...
```

## Why This Is Better

- **Encapsulation**: Only exported items are public
- **No global pollution**: Each module has its own scope
- **Clear dependencies**: Imports show exactly what each file needs
- **Better tooling**: IDEs can auto-import and suggest imports
- **Code splitting**: Only load modules when needed (dynamic imports)
- **Type safety**: Import types and enforce at compile time
- **Reusability**: Share code across projects via npm packages

## Common Mistake + Fix

**Mistake**: Circular dependencies causing initialization issues

```typescript
// a.ts
import { b } from "./b";
export const a = "A" + b;

// b.ts
import { a } from "./a";
export const b = "B" + a;

// Result: Circular dependency, values may be undefined at runtime
```

**Fix**: Restructure code to remove circular dependencies

```typescript
// shared.ts
export const BASE = "BASE";

// a.ts
import { BASE } from "./shared";
export const a = "A" + BASE;

// b.ts
import { BASE } from "./shared";
export const b = "B" + BASE;
```

## Key Notes / Edge Cases

- **File extensions in imports**: TypeScript imports don't include `.ts` extension:
  ```typescript
  import { User } from "./user";     // ✓ Correct
  // import { User } from "./user.ts";  // ✗ Don't include extension
  ```

- **Type-only imports**: Use `import type` to import only types, not runtime code. This can improve tree-shaking:
  ```typescript
  import type { User } from "./user";  // Type only, erased at runtime
  ```

- **CommonJS vs ES Modules**: TypeScript can compile to different module systems:
  ```json
  // tsconfig.json
  {
    "compilerOptions": {
      "module": "commonjs"  // For Node.js
      // or "module": "ES2015"  // For modern browsers/bundlers
    }
  }
  ```

- **Barrel exports**: Use index.ts to re-export from a directory:
  ```typescript
  // utils/index.ts
  export * from "./validators";
  export * from "./formatters";
  export * from "./helpers";
  
  // Then import from directory
  import { isValidEmail, formatDate, debounce } from "./utils";
  ```

- **esModuleInterop**: Set this to `true` in tsconfig.json for better CommonJS compatibility:
  ```json
  {
    "compilerOptions": {
      "esModuleInterop": true
    }
  }
  ```

## Quick Practice

1. **Organize a module**: Create a `math.ts` module that exports:
   - A `PI` constant
   - An `add` function
   - A `multiply` function
   - A `Circle` class with a `getArea` method
   Then import and use them in a separate file.

2. **Fix the import**: What's wrong with this import statement?
   ```typescript
   import User from "./user.ts";
   ```

3. **Prevent circular dependency**: You have files A and B that need to use code from each other. How would you restructure them to avoid circular dependencies?

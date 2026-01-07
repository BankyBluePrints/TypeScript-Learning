# Template Literal Types (TypeScript 4.1)

## What it is

Template literal types use template literal syntax to create new string literal types from existing ones. This enables powerful string manipulation at the type level, allowing you to create types that transform, concatenate, or pattern-match string literals.

## Before this feature

Before TypeScript 4.1, string manipulation at the type level was impossible:

```typescript
// TypeScript pre-4.1 - can't manipulate string literal types
type Color = "red" | "green" | "blue";

// Wanted: "setRed" | "setGreen" | "setBlue"
// Had to manually type each variation
type SetterNames = "setRed" | "setGreen" | "setBlue";  // Manual, error-prone

// No way to derive types from string patterns
```

## After this feature

Template literal types enable string manipulation at compile time:

```typescript
// TypeScript 4.1+ - template literal types

// 1. Simple concatenation
type Color = "red" | "green" | "blue";
type SetterName<T extends string> = `set${Capitalize<T>}`;

type Setters = SetterName<Color>;
// Result: "setRed" | "setGreen" | "setBlue"

// 2. Multiple string unions (Cartesian product)
type Size = "small" | "medium" | "large";
type ColorSize = `${Color}-${Size}`;
// Result: "red-small" | "red-medium" | "red-large" | 
//         "green-small" | "green-medium" | "green-large" |
//         "blue-small" | "blue-medium" | "blue-large"

// 3. Intrinsic string manipulation types
type Greeting = "hello world";

type Upper = Uppercase<Greeting>;      // "HELLO WORLD"
type Lower = Lowercase<Greeting>;      // "hello world"
type Cap = Capitalize<Greeting>;       // "Hello world"
type Uncap = Uncapitalize<Greeting>;   // "hello world"

// 4. Event handler pattern
type EventNames = "click" | "focus" | "blur";
type HandlerName<T extends string> = `on${Capitalize<T>}`;

type Handlers = HandlerName<EventNames>;
// Result: "onClick" | "onFocus" | "onBlur"

// 5. API endpoint types
type Entity = "user" | "post" | "comment";
type Method = "GET" | "POST" | "PUT" | "DELETE";
type Endpoint = `/${Entity}`;
type APIRoute = `${Method} ${Endpoint}`;

// Result includes: "GET /user", "POST /user", "PUT /user", etc.

// 6. Object key remapping
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};

interface Person {
  name: string;
  age: number;
}

type PersonGetters = Getters<Person>;
// Result: { getName: () => string; getAge: () => number; }

// 7. PropEventSource pattern (React-like)
type PropEventSource<T> = {
  on<K extends string & keyof T>(
    eventName: `${K}Changed`,
    callback: (newValue: T[K]) => void
  ): void;
};

interface User {
  name: string;
  age: number;
}

declare const source: PropEventSource<User>;

source.on("nameChanged", (newName) => {
  // newName is inferred as string
  console.log(newName.toUpperCase());
});

source.on("ageChanged", (newAge) => {
  // newAge is inferred as number
  console.log(newAge.toFixed(2));
});

// source.on("emailChanged", ...) // ✗ Error: "emailChanged" not valid

// 8. Pattern matching with infer
type ExtractRouteParams<T extends string> = 
  T extends `${infer _Start}/:${infer Param}/${infer Rest}`
    ? Param | ExtractRouteParams<`/${Rest}`>
    : T extends `${infer _Start}/:${infer Param}`
      ? Param
      : never;

type Route = "/users/:userId/posts/:postId";
type Params = ExtractRouteParams<Route>;  // "userId" | "postId"

// 9. CSS-in-JS type safety
type CSSProperty = "color" | "background" | "fontSize";
type CSSValue<T extends CSSProperty> = 
  T extends "fontSize" ? `${number}px` | `${number}rem` :
  T extends "color" ? `#${string}` | `rgb(${string})` :
  string;

type StyleRule = {
  [K in CSSProperty]?: CSSValue<K>
};

const styles: StyleRule = {
  color: "#ff0000",       // ✓ OK
  fontSize: "16px",       // ✓ OK
  // fontSize: "large",   // Could add validation
};
```

## Why this is better

- **Type-level string manipulation**: Transform strings at compile time
- **Derived types**: Generate related types automatically
- **API type safety**: Enforce string patterns in APIs
- **Reduced duplication**: No need to manually type all string variations
- **Better refactoring**: Rename base types and derived types update automatically
- **Self-documenting**: Type signatures show string patterns clearly

## Key notes / edge cases

- Template literal types can produce very large union types (combinatorial explosion):
  ```typescript
  type A = "a" | "b" | "c";
  type B = "x" | "y" | "z";
  type C = `${A}${B}`;  // 9 combinations
  ```

- Intrinsic types are built-in: `Uppercase`, `Lowercase`, `Capitalize`, `Uncapitalize`

- Works with `as` clause in mapped types for key remapping:
  ```typescript
  type Setters<T> = {
    [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void
  };
  ```

- Pattern matching uses `infer` in conditional types:
  ```typescript
  type GetParam<T> = T extends `/${infer Param}` ? Param : never;
  ```

- Empty string is a valid template literal:
  ```typescript
  type Empty = `${""}`;  // ""
  ```

## Quick practice

1. **Create getter types**: Given an interface, create a type that generates getter method names:
   ```typescript
   interface Config {
     apiUrl: string;
     timeout: number;
     retries: number;
   }
   // Should produce: "getApiUrl" | "getTimeout" | "getRetries"
   ```

2. **HTTP methods**: Create a type for HTTP endpoints that combines methods (GET, POST) with paths (/users, /posts):
   ```typescript
   type Method = "GET" | "POST";
   type Path = "/users" | "/posts";
   // Result should be: "GET /users" | "GET /posts" | "POST /users" | "POST /posts"
   ```

3. **Extract version**: Create a type that extracts the version number from strings like "v1.2.3":
   ```typescript
   type Version = "v1.2.3" | "v2.0.1";
   // Extract to: "1.2.3" | "2.0.1"
   ```

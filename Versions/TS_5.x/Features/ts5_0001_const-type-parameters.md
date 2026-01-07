# Const Type Parameters (TypeScript 5.0)

## What it is

Const type parameters (written as `<const T>`) allow you to infer the most specific type possible for generic type parameters, similar to `as const` assertions. This feature enables precise literal type inference in generic functions without requiring explicit `as const` at call sites.

## Before this feature

Before TypeScript 5.0, generic inference would widen literal types:

```typescript
// TypeScript pre-5.0
function makeObject<T>(value: T) {
  return { value };
}

let obj1 = makeObject("hello");
// Type: { value: string } - widened from "hello" to string

// Had to use 'as const' at call site
let obj2 = makeObject("hello" as const);
// Type: { value: "hello" } - literal type preserved

// Or use complex type gymnastics
function makeConstObject<T extends string>(value: T) {
  return { value } as { value: T };
}
```

## After this feature

Const type parameters preserve literal types automatically:

```typescript
// TypeScript 5.0+ - const type parameter

// 1. Basic const type parameter
function makeObject<const T>(value: T) {
  return { value };
}

let obj = makeObject("hello");
// Type: { value: "hello" } - literal type automatically!

let nums = makeObject([1, 2, 3]);
// Type: { value: [1, 2, 3] } - exact tuple type!

// 2. Array inference
function toArray<const T>(values: T[]) {
  return values;
}

let arr1 = toArray([1, 2, 3]);
// Type: (1 | 2 | 3)[] - literal union!

let arr2 = toArray(["a", "b", "c"]);
// Type: ("a" | "b" | "c")[] - literal strings!

// Compare with non-const:
function toArrayNormal<T>(values: T[]) {
  return values;
}

let arr3 = toArrayNormal([1, 2, 3]);
// Type: number[] - widened

// 3. Object literal preservation
function createConfig<const T extends Record<string, any>>(config: T) {
  return config;
}

let config = createConfig({
  apiUrl: "https://api.example.com",
  timeout: 5000,
  retries: 3
});
// Type: {
//   apiUrl: "https://api.example.com";
//   timeout: 5000;
//   retries: 3;
// }

// 4. Tuple inference
function tuple<const T extends readonly any[]>(...args: T) {
  return args;
}

let result = tuple("hello", 42, true);
// Type: readonly ["hello", 42, true] - exact tuple!

// 5. Route definitions (practical example)
function defineRoutes<const T extends Record<string, string>>(routes: T) {
  return routes;
}

const routes = defineRoutes({
  home: "/",
  about: "/about",
  user: "/users/:id"
});
// Type: {
//   readonly home: "/";
//   readonly about: "/about";
//   readonly user: "/users/:id";
// }

type RouteKeys = keyof typeof routes;  // "home" | "about" | "user"

// 6. Event system with const
function createEventEmitter<const T extends Record<string, any>>() {
  const listeners: any = {};
  
  return {
    on<K extends keyof T>(event: K, handler: (data: T[K]) => void) {
      listeners[event] = handler;
    },
    emit<K extends keyof T>(event: K, data: T[K]) {
      listeners[event]?.(data);
    }
  };
}

const emitter = createEventEmitter<{
  click: { x: number; y: number };
  submit: { form: string };
}>();

emitter.on("click", (data) => {
  console.log(data.x, data.y);  // Typed correctly!
});

// 7. Readonly deeply nested
function freeze<const T>(obj: T) {
  return Object.freeze(obj);
}

let frozen = freeze({
  user: {
    name: "Alice",
    settings: {
      theme: "dark"
    }
  }
});
// All properties inferred as readonly and literal types!

// 8. Type-safe action creators
function createAction<const T extends string, const P>(
  type: T,
  payload: P
) {
  return { type, payload };
}

const action = createAction("USER_LOGIN", { userId: 123 });
// Type: { type: "USER_LOGIN"; payload: { userId: 123 } }
// Not: { type: string; payload: { userId: number } }
```

## Why this is better

- **Automatic literal preservation**: No need for `as const` at call sites
- **More precise types**: Exact literal types inferred automatically
- **Better DX**: Less verbose API usage
- **Immutability hints**: Const suggests immutable intent
- **Library design**: Create APIs that preserve literal types naturally
- **Reduced type assertions**: Less need for manual type narrowing

## Key notes / edge cases

- `const` applies to the type parameter, making it behave like `as const` for inference
- Works with objects, arrays, tuples, and primitives
- Can be combined with constraints: `<const T extends Record<string, any>>`
- Deeply readonly: nested properties also become readonly
- Cannot be used with type parameter defaults in some contexts
- The `const` modifier affects inference, not the actual mutability at runtime

```typescript
// Const with constraints
function process<const T extends { id: number }>(item: T) {
  return item;
}

let result = process({ id: 1, name: "test" });
// Type: { readonly id: 1; readonly name: "test" }

// Multiple type parameters
function pair<const T, const U>(a: T, b: U) {
  return [a, b] as const;
}

let p = pair("hello", 42);
// Type: readonly ["hello", 42]

// Const doesn't affect runtime
function modify<const T extends any[]>(arr: T) {
  arr.push(999);  // ✓ Compiles (runtime mutation allowed)
  return arr;
}
```

## Quick practice

1. **Create a const function**: Write a function `defineConstants` that preserves literal types for an object of constants without requiring `as const` from users.

2. **Action creator**: Implement a type-safe action creator that preserves the exact action type string and payload type:
   ```typescript
   const action = createAction("INCREMENT", { by: 5 });
   // Should have type: { type: "INCREMENT"; payload: { by: 5 } }
   ```

3. **Compare with and without const**: Create two versions of a `createConfig` function - one with `<const T>` and one with `<T>`. Pass the same object to both and compare the inferred types.

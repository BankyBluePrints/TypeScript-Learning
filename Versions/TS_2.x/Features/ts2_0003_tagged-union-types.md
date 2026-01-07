# Tagged Union Types (TypeScript 2.0)

## What it is

Tagged unions (also called discriminated unions) are union types where each variant has a common property (the "tag" or "discriminant") with a literal type. TypeScript 2.0 improved support for these, enabling exhaustive type checking and automatic type narrowing based on the tag.

## Before this feature

Without tagged unions, handling multiple types required verbose type checking:

```typescript
// Pre-2.0 approach - no discriminant property
interface Circle {
  radius: number;
}

interface Rectangle {
  width: number;
  height: number;
}

type Shape = Circle | Rectangle;

function getArea(shape: Shape): number {
  // How do we know which type shape is?
  if ('radius' in shape) {
    return Math.PI * shape.radius ** 2;
  } else {
    return shape.width * shape.height;
  }
}
```

## After this feature

Tagged unions use a discriminant property for clean type narrowing:

```typescript
// TypeScript 2.0+ - tagged union with discriminant
interface Circle {
  kind: "circle";  // Tag/discriminant
  radius: number;
}

interface Rectangle {
  kind: "rectangle";  // Tag/discriminant
  width: number;
  height: number;
}

interface Triangle {
  kind: "triangle";
  base: number;
  height: number;
}

type Shape = Circle | Rectangle | Triangle;

function getArea(shape: Shape): number {
  switch (shape.kind) {  // TypeScript narrows type based on 'kind'
    case "circle":
      return Math.PI * shape.radius ** 2;  // shape is Circle here
    case "rectangle":
      return shape.width * shape.height;   // shape is Rectangle here
    case "triangle":
      return (shape.base * shape.height) / 2;  // shape is Triangle here
  }
}

// Exhaustiveness checking
function getAreaSafe(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
    case "triangle":
      return (shape.base * shape.height) / 2;
    default:
      const _exhaustive: never = shape;  // Error if a case is missing
      return _exhaustive;
  }
}

// Tagged unions for state machines
type LoadingState = {
  status: "loading";
};

type SuccessState = {
  status: "success";
  data: any;
};

type ErrorState = {
  status: "error";
  error: string;
};

type AsyncState = LoadingState | SuccessState | ErrorState;

function handleState(state: AsyncState) {
  switch (state.status) {
    case "loading":
      console.log("Loading...");
      break;
    case "success":
      console.log("Data:", state.data);  // TypeScript knows state has 'data'
      break;
    case "error":
      console.log("Error:", state.error);  // TypeScript knows state has 'error'
      break;
  }
}
```

## Why this is better

- **Type-safe narrowing**: TypeScript automatically narrows types based on the discriminant
- **Exhaustiveness checking**: Compiler ensures all cases are handled
- **Better IDE support**: Autocomplete suggests correct properties for each variant
- **Self-documenting**: Clear structure for variant types
- **Refactoring safety**: Adding new variants produces compile errors if not handled
- **Pattern matching**: Similar to pattern matching in functional languages

## Key notes / edge cases

- The discriminant property must have literal types (string literals, number literals, etc.)
- All variants must have the same discriminant property name
- Use `never` type in default case to ensure exhaustiveness:
  ```typescript
  function handle(shape: Shape): number {
    switch (shape.kind) {
      case "circle":
        return shape.radius;
      case "rectangle":
        return shape.width;
      default:
        const _exhaustive: never = shape;
        throw new Error(`Unhandled shape: ${_exhaustive}`);
    }
  }
  ```

- You can use `if` statements instead of `switch`:
  ```typescript
  function getArea(shape: Shape): number {
    if (shape.kind === "circle") {
      return Math.PI * shape.radius ** 2;
    } else if (shape.kind === "rectangle") {
      return shape.width * shape.height;
    } else {
      return (shape.base * shape.height) / 2;
    }
  }
  ```

- Tagged unions work well with Redux-style actions:
  ```typescript
  type Action =
    | { type: "INCREMENT" }
    | { type: "DECREMENT" }
    | { type: "SET_VALUE"; value: number };
  
  function reducer(state: number, action: Action): number {
    switch (action.type) {
      case "INCREMENT":
        return state + 1;
      case "DECREMENT":
        return state - 1;
      case "SET_VALUE":
        return action.value;  // TypeScript knows action has 'value'
    }
  }
  ```

## Quick practice

1. **Create a tagged union**: Design a `Result<T>` type that can be either:
   - `{ success: true; data: T }`
   - `{ success: false; error: string }`
   Write a function that handles both cases.

2. **Exhaustiveness check**: Add a new shape type (e.g., `Square`) to the `Shape` union. What happens to the `getArea` function if you forget to handle the new case?

3. **State machine**: Model a traffic light with three states (red, yellow, green) using a tagged union. Write a function that transitions to the next state.

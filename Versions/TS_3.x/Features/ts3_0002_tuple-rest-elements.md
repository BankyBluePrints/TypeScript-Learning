# Tuple Types with Rest Elements (TypeScript 3.0)

## What it is

TypeScript 3.0 extended tuple types to support rest elements (`...T[]`) and optional elements, allowing for more flexible and expressive tuple type definitions. This enables variadic tuples, tail/head patterns, and better function parameter modeling.

## Before this feature

Before TypeScript 3.0, tuples had fixed lengths and couldn't express variable-length patterns:

```typescript
// TypeScript pre-3.0 - fixed-length tuples only
type Pair = [string, number];
type Triple = [string, number, boolean];

// No way to express "at least one element"
// No way to express "optional trailing elements"
// No way to express variadic patterns

function concat(arr1: [number, number], arr2: [number, number]): [number, number, number, number] {
  return [...arr1, ...arr2];  // Type doesn't capture the pattern
}
```

## After this feature

Rest elements enable flexible tuple patterns:

```typescript
// TypeScript 3.0+ - tuples with rest elements

// 1. Rest at the end (variadic tail)
type StringAndNumbers = [string, ...number[]];

let valid: StringAndNumbers = ["hello", 1, 2, 3];
let alsoValid: StringAndNumbers = ["hello"];
// let invalid: StringAndNumbers = [1, 2, 3];  // ✗ Error: first must be string

// 2. Rest at the beginning (variadic head)
type NumbersAndString = [...number[], string];

let values: NumbersAndString = [1, 2, 3, "end"];
let minimal: NumbersAndString = ["end"];

// 3. Rest in the middle
type HeadAndTail = [number, ...string[], boolean];

let example: HeadAndTail = [1, "a", "b", "c", true];
let short: HeadAndTail = [1, true];

// 4. Optional elements (using ?)
type OptionalTuple = [string, number?];

let withOptional: OptionalTuple = ["hello", 42];
let withoutOptional: OptionalTuple = ["hello"];
// let invalid: OptionalTuple = ["hello", 42, 99];  // ✗ Error: too many elements

// 5. Practical example: function parameters
function tuple<T extends any[]>(...args: T): T {
  return args;
}

let result = tuple("hello", 42, true);  // Type: [string, number, boolean]

// 6. Concat function with proper types
function concat<T extends any[], U extends any[]>(
  arr1: T,
  arr2: U
): [...T, ...U] {
  return [...arr1, ...arr2];
}

let combined = concat(["a", "b"], [1, 2, 3]);
// Type: [string, string, number, number, number]

// 7. Head and tail pattern
type Head<T extends any[]> = T extends [infer H, ...any[]] ? H : never;
type Tail<T extends any[]> = T extends [any, ...infer T] ? T : never;

type MyTuple = [string, number, boolean];
type First = Head<MyTuple>;   // string
type Rest = Tail<MyTuple>;    // [number, boolean]

// 8. Function overloads with tuples
function format(template: string, ...values: [string, number]): string;
function format(template: string, ...values: [string, number, boolean]): string;
function format(template: string, ...values: any[]): string {
  return template.replace(/{(\d+)}/g, (_, i) => values[i]);
}

format("Hello {0}, you have {1} messages", "Alice", 5);
```

## Why this is better

- **Flexible tuple lengths**: Express variadic patterns in types
- **Better function typing**: Model rest parameters more precisely
- **Type inference**: TypeScript can infer exact tuple types
- **Optional elements**: Express tuples with optional trailing elements
- **Spread operations**: Type-safe tuple concatenation and spreading
- **Pattern matching**: Enable head/tail decomposition patterns

## Key notes / edge cases

- Rest elements can appear at any position, but there can be at most one rest element per tuple
- Optional elements must come after required elements
- Rest elements cannot be followed by optional elements or other rest elements
- Empty tuples are valid: `type Empty = []`
- Readonly tuples with rest: `type ReadonlyRest = readonly [string, ...number[]]`

```typescript
// Valid tuple rest patterns
type Valid1 = [string, ...number[]];              // ✓ Rest at end
type Valid2 = [...number[], string];              // ✓ Rest at start
type Valid3 = [string, ...number[], boolean];     // ✓ Rest in middle

// Invalid patterns
// type Invalid1 = [string, ...number[], ...string[]];  // ✗ Multiple rests
// type Invalid2 = [string?, number];                   // ✗ Optional before required
// type Invalid3 = [...number[], string?];              // ✗ Optional after rest

// Spreading tuples
type Tuple1 = [string, number];
type Tuple2 = [boolean, symbol];
type Combined = [...Tuple1, ...Tuple2];  // [string, number, boolean, symbol]

// Readonly and rest
type ReadonlyRest = readonly [string, ...number[]];
const value: ReadonlyRest = ["hello", 1, 2, 3];
// value[0] = "bye";  // ✗ Error: readonly
```

## Quick practice

1. **Create a variadic tuple**: Define a tuple type that starts with a string, followed by any number of numbers, and ends with a boolean.

2. **Type-safe concat**: Implement a function that concatenates two tuples and preserves the exact types:
   ```typescript
   concat([1, "hello"], [true, 42])  // Should return [1, "hello", true, 42]
   ```

3. **Optional elements**: Create a tuple type representing a database query result that has:
   - Required: id (number) and name (string)
   - Optional: email (string)
   - Optional: age (number)

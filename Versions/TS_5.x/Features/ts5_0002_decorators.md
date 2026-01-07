# Decorators (TypeScript 5.0)

## What it is

TypeScript 5.0 introduced support for ECMAScript Stage 3 decorators, which are functions that can modify or annotate classes, methods, accessors, properties, and parameters. This replaced the experimental legacy decorators with the official JavaScript specification.

## Before this feature

Before TypeScript 5.0, decorators were experimental and non-standard:

```typescript
// TypeScript pre-5.0 - legacy experimental decorators
// Required: "experimentalDecorators": true in tsconfig.json

function OldDecorator(target: any) {
  // Different signature and behavior than Stage 3
}

@OldDecorator
class MyClass {
  // Used non-standard decorator implementation
}
```

## After this feature

TypeScript 5.0 supports standard ECMAScript decorators:

```typescript
// TypeScript 5.0+ - Stage 3 decorators (standard)

// 1. Class decorator
function logged<T extends { new(...args: any[]): {} }>(
  target: T,
  context: ClassDecoratorContext
) {
  return class extends target {
    constructor(...args: any[]) {
      super(...args);
      console.log(`Creating instance of ${context.name}`);
    }
  };
}

@logged
class User {
  constructor(public name: string) {}
}

const user = new User("Alice");  // Logs: "Creating instance of User"

// 2. Method decorator
function measure(
  target: Function,
  context: ClassMethodDecoratorContext
) {
  const methodName = String(context.name);
  
  return function (this: any, ...args: any[]) {
    const start = performance.now();
    const result = target.apply(this, args);
    const end = performance.now();
    console.log(`${methodName} took ${end - start}ms`);
    return result;
  };
}

class Calculator {
  @measure
  add(a: number, b: number): number {
    return a + b;
  }
}

// 3. Accessor decorator
function logged(
  target: Function,
  context: ClassGetterDecoratorContext | ClassSetterDecoratorContext
) {
  return function (this: any, ...args: any[]) {
    console.log(`Accessing ${String(context.name)}`);
    return target.apply(this, args);
  };
}

class Person {
  private _age: number = 0;
  
  @logged
  get age() {
    return this._age;
  }
  
  @logged
  set age(value: number) {
    this._age = value;
  }
}

// 4. Auto-accessor decorator
function reactive(
  target: undefined,
  context: ClassAccessorDecoratorContext
) {
  return {
    get(this: any) {
      return this[`_${String(context.name)}`];
    },
    set(this: any, value: any) {
      console.log(`Setting ${String(context.name)} to ${value}`);
      this[`_${String(context.name)}`] = value;
    }
  };
}

class Component {
  @reactive
  accessor count = 0;  // Creates getter and setter
}

const comp = new Component();
comp.count = 5;  // Logs: "Setting count to 5"

// 5. Field decorator
function readonly(
  target: undefined,
  context: ClassFieldDecoratorContext
) {
  return function (this: any, initialValue: any) {
    return initialValue;
  };
}

class Config {
  @readonly
  apiUrl = "https://api.example.com";
}

// 6. Multiple decorators (execute bottom to top)
function first() {
  console.log("first(): factory evaluated");
  return function (target: any, context: any) {
    console.log("first(): decorator called");
  };
}

function second() {
  console.log("second(): factory evaluated");
  return function (target: any, context: any) {
    console.log("second(): decorator called");
  };
}

class Example {
  @first()
  @second()
  method() {}
}
// Output:
// first(): factory evaluated
// second(): factory evaluated
// second(): decorator called
// first(): decorator called

// 7. Metadata with decorators
function validate(
  target: Function,
  context: ClassMethodDecoratorContext
) {
  context.metadata[context.name] = { validated: true };
  return target;
}

class UserService {
  @validate
  createUser(name: string) {
    // ...
  }
}

// 8. Practical example: Dependency injection
const dependencies = new Map<string, any>();

function injectable(
  target: Function,
  context: ClassDecoratorContext
) {
  const name = context.name;
  dependencies.set(name, new (target as any)());
}

function inject(serviceName: string) {
  return function (
    target: undefined,
    context: ClassFieldDecoratorContext
  ) {
    return function (this: any) {
      return dependencies.get(serviceName);
    };
  };
}

@injectable
class Database {
  connect() {
    console.log("Connected to database");
  }
}

class UserRepository {
  @inject("Database")
  db!: Database;
  
  getUsers() {
    this.db.connect();
  }
}
```

## Why this is better

- **Standard JavaScript**: Based on official ECMAScript specification
- **Type safety**: Full TypeScript type checking for decorators
- **Metadata**: Decorators can attach metadata to classes
- **Composable**: Multiple decorators can be combined
- **Clean syntax**: Declarative way to modify class behavior
- **Framework support**: Used by Angular, NestJS, TypeORM, etc.

## Key notes / edge cases

- Enable with `"experimentalDecorators": false` (or omit) in tsconfig.json
- Stage 3 decorators are different from legacy decorators - not backward compatible
- Decorator execution order: bottom to top for multiple decorators on same target
- Decorators run at class definition time, not instantiation time
- Context object provides metadata about the decorated member
- Can return a replacement function/class or undefined

```typescript
// Decorator context types
interface ClassDecoratorContext {
  kind: "class";
  name: string | undefined;
  metadata: Record<PropertyKey, unknown>;
}

interface ClassMethodDecoratorContext {
  kind: "method";
  name: string | symbol;
  static: boolean;
  private: boolean;
  metadata: Record<PropertyKey, unknown>;
}

// Field vs accessor
class Example {
  // Field (no getter/setter)
  @fieldDecorator
  field = 0;
  
  // Auto-accessor (generates getter/setter)
  @accessorDecorator
  accessor count = 0;
}

// Cannot decorate constructor
class Invalid {
  // @decorator  // ✗ Error: Cannot decorate constructor
  constructor() {}
}
```

## Quick practice

1. **Create a logging decorator**: Write a method decorator that logs the method name and arguments every time it's called.

2. **Validation decorator**: Create a decorator that validates method parameters are not null/undefined before execution.

3. **Memoization**: Implement a `@memoize` decorator that caches method results based on arguments to avoid redundant computations.

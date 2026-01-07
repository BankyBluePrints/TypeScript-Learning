# Classes Basics

## What it is

Classes in TypeScript extend JavaScript ES6 classes with static type checking, access modifiers (`public`, `private`, `protected`), abstract classes, and type annotations for properties and methods. Classes provide object-oriented programming capabilities with compile-time type safety.

## Before TypeScript (JavaScript Classes)

JavaScript ES6 has classes, but no access modifiers or type safety:

```javascript
// JavaScript class - no type safety
class User {
  constructor(name, age) {
    this.name = name;
    this.age = age;
    this._password = "secret";  // Convention: _ means private (not enforced!)
  }
  
  greet() {
    return `Hello, ${this.name}`;
  }
  
  getAge() {
    return this.age;
  }
}

const user = new User("Alice", 30);
console.log(user.name);           // OK
console.log(user._password);      // "secret" - nothing prevents access!
user.age = "thirty";             // No error - type changed at runtime
```

## With TypeScript Classes

TypeScript adds type annotations and access modifiers:

```typescript
// TypeScript class with types and access modifiers
class User {
  public name: string;           // Accessible everywhere
  private age: number;           // Only accessible inside this class
  private password: string;      // Truly private
  
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
    this.password = "secret";
  }
  
  public greet(): string {
    return `Hello, ${this.name}`;
  }
  
  public getAge(): number {
    return this.age;
  }
  
  private validatePassword(pwd: string): boolean {
    return pwd === this.password;
  }
}

const user = new User("Alice", 30);
console.log(user.name);           // ✓ OK - public property
console.log(user.greet());        // ✓ OK - public method
// console.log(user.age);         // ✗ Error: 'age' is private
// console.log(user.password);    // ✗ Error: 'password' is private
// user.validatePassword("test"); // ✗ Error: 'validatePassword' is private
```

## Class Features in TypeScript

```typescript
// 1. Property shorthand (parameter properties)
class User {
  // Declare and initialize in one line
  constructor(
    public name: string,
    private age: number,
    protected email: string
  ) {
    // Properties automatically created and assigned
  }
}

// 2. Readonly properties
class Config {
  readonly apiUrl: string = "https://api.example.com";
  
  constructor(public readonly version: string) {}
}

const config = new Config("1.0");
// config.version = "2.0";  // ✗ Error: Cannot assign to 'version' because it is a read-only property

// 3. Access modifiers
class BankAccount {
  private balance: number = 0;      // Only in this class
  protected accountId: string;      // In this class and subclasses
  public owner: string;             // Everywhere (default)
  
  constructor(owner: string, accountId: string) {
    this.owner = owner;
    this.accountId = accountId;
  }
  
  public deposit(amount: number): void {
    this.balance += amount;
  }
  
  public getBalance(): number {
    return this.balance;
  }
}

// 4. Static members
class MathUtils {
  static PI: number = 3.14159;
  
  static square(x: number): number {
    return x * x;
  }
}

console.log(MathUtils.PI);       // ✓ Access without instance
console.log(MathUtils.square(5)); // 25

// 5. Inheritance
class Animal {
  constructor(protected name: string) {}
  
  move(distance: number): void {
    console.log(`${this.name} moved ${distance}m`);
  }
}

class Dog extends Animal {
  constructor(name: string, public breed: string) {
    super(name);  // Call parent constructor
  }
  
  bark(): void {
    console.log("Woof!");
  }
  
  // Override parent method
  move(distance: number): void {
    console.log("Running...");
    super.move(distance);  // Call parent implementation
  }
}

const dog = new Dog("Buddy", "Golden Retriever");
dog.bark();
dog.move(10);

// 6. Abstract classes (cannot instantiate directly)
abstract class Shape {
  constructor(public color: string) {}
  
  abstract getArea(): number;  // Must be implemented by subclasses
  
  describe(): string {
    return `A ${this.color} shape with area ${this.getArea()}`;
  }
}

class Circle extends Shape {
  constructor(color: string, private radius: number) {
    super(color);
  }
  
  getArea(): number {
    return Math.PI * this.radius ** 2;
  }
}

// const shape = new Shape("red");  // ✗ Error: Cannot create an instance of an abstract class
const circle = new Circle("red", 5);
console.log(circle.getArea());

// 7. Implementing interfaces
interface Printable {
  print(): void;
}

interface Saveable {
  save(): void;
}

class Document implements Printable, Saveable {
  constructor(private content: string) {}
  
  print(): void {
    console.log(this.content);
  }
  
  save(): void {
    console.log("Saving document...");
  }
}

// 8. Getters and Setters
class Temperature {
  private _celsius: number = 0;
  
  get celsius(): number {
    return this._celsius;
  }
  
  set celsius(value: number) {
    if (value < -273.15) {
      throw new Error("Temperature below absolute zero!");
    }
    this._celsius = value;
  }
  
  get fahrenheit(): number {
    return this._celsius * 9/5 + 32;
  }
  
  set fahrenheit(value: number) {
    this.celsius = (value - 32) * 5/9;
  }
}

const temp = new Temperature();
temp.celsius = 25;
console.log(temp.fahrenheit);  // 77
```

## Why This Is Better

- **Encapsulation**: Private/protected members are truly enforced
- **Type safety**: Properties and methods have type annotations
- **Better IDE support**: Autocomplete knows class structure
- **Compile-time errors**: Access violations caught before runtime
- **Clear contracts**: Interfaces ensure classes implement required methods
- **Code organization**: Object-oriented patterns with type safety

## Common Mistake + Fix

**Mistake**: Forgetting to call `super()` in a derived class constructor

```typescript
class Animal {
  constructor(public name: string) {}
}

class Dog extends Animal {
  constructor(name: string, public breed: string) {
    // Missing super(name)
    this.breed = breed;  // ✗ Error: 'super' must be called before accessing 'this'
  }
}
```

**Fix**: Always call `super()` before accessing `this` in derived classes

```typescript
class Dog extends Animal {
  constructor(name: string, public breed: string) {
    super(name);         // ✓ Call parent constructor first
    this.breed = breed;  // ✓ Now OK
  }
}
```

## Key Notes / Edge Cases

- **Access modifiers are compile-time only**: At runtime (in JavaScript), private members are still accessible. Use `#` for true runtime privacy:
  ```typescript
  class User {
    #password: string;  // True private field (ES2022)
    
    constructor(password: string) {
      this.#password = password;
    }
  }
  ```

- **Static blocks**: Initialize static members with complex logic:
  ```typescript
  class Config {
    static settings: Record<string, any>;
    
    static {
      // Static initialization block
      this.settings = loadConfig();
    }
  }
  ```

- **Abstract members**: Abstract classes can have both abstract and concrete members:
  ```typescript
  abstract class Base {
    abstract required(): void;  // Must implement
    optional(): void {           // Can override
      console.log("default");
    }
  }
  ```

- **Protected constructor**: Prevents direct instantiation but allows inheritance:
  ```typescript
  class Singleton {
    protected constructor() {}
    
    static instance: Singleton;
    static getInstance(): Singleton {
      if (!this.instance) {
        this.instance = new Singleton();
      }
      return this.instance;
    }
  }
  ```

## Quick Practice

1. **Create a class**: Design a `BankAccount` class with:
   - Private `balance` property
   - Public `deposit` and `withdraw` methods
   - A getter for the balance
   - Constructor that takes initial balance

2. **Fix the inheritance**: What's wrong with this code?
   ```typescript
   class Vehicle {
     constructor(private speed: number) {}
   }
   
   class Car extends Vehicle {
     constructor(speed: number, private brand: string) {
       this.brand = brand;
     }
   }
   ```

3. **Implement an interface**: Create a `Logger` interface with `log()` and `error()` methods. Then create a `ConsoleLogger` class that implements it.

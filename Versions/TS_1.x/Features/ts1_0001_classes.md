# Classes (TypeScript 1.0)

## What it is

TypeScript 1.0 introduced full ES6-style class support with type annotations, access modifiers (`public`, `private`), and inheritance. This was a foundational feature that brought object-oriented programming to TypeScript with compile-time type checking.

## Before this feature

JavaScript before ES6 used constructor functions and prototypes for object-oriented patterns:

```javascript
// Pre-ES6 JavaScript - constructor function pattern
function Person(name, age) {
  this.name = name;
  this.age = age;
  this._private = "should be private";  // No real privacy
}

Person.prototype.greet = function() {
  return "Hello, " + this.name;
};

var person = new Person("Alice", 30);
console.log(person._private);  // Can still access "private" properties!
```

This approach had no type safety and no true encapsulation.

## After this feature

TypeScript 1.0 introduced proper class syntax with types and access modifiers:

```typescript
// TypeScript 1.0 - classes with types and access modifiers
class Person {
  public name: string;
  private age: number;
  
  constructor(name: string, age: number) {
    this.name = name;
    this.age = age;
  }
  
  public greet(): string {
    return `Hello, ${this.name}`;
  }
  
  private getAge(): number {
    return this.age;
  }
}

const person = new Person("Alice", 30);
console.log(person.name);         // ✓ OK - public
console.log(person.greet());      // ✓ OK - public method
// console.log(person.age);       // ✗ Error: 'age' is private
// console.log(person.getAge());  // ✗ Error: 'getAge' is private
```

## Why this is better

- **Type safety**: All properties and methods have type annotations
- **True encapsulation**: Private members are enforced at compile time
- **Better IDE support**: Autocomplete knows class structure
- **Cleaner syntax**: More readable than constructor functions
- **Inheritance support**: Proper `extends` keyword for inheritance
- **Compile-time errors**: Access violations caught before runtime

## Key notes / edge cases

- Access modifiers (`public`, `private`) are compile-time only in TypeScript 1.0. At runtime (JavaScript), all members are still accessible
- The `public` modifier is the default and can be omitted
- Classes can implement interfaces to enforce structure
- Inheritance uses the `extends` keyword
- Use `super()` in derived classes to call the parent constructor

```typescript
// Inheritance example
class Animal {
  constructor(public name: string) {}
  
  move(distance: number): void {
    console.log(`${this.name} moved ${distance}m`);
  }
}

class Dog extends Animal {
  constructor(name: string, public breed: string) {
    super(name);  // Must call parent constructor
  }
  
  bark(): void {
    console.log("Woof!");
  }
}

const dog = new Dog("Buddy", "Golden Retriever");
dog.bark();
dog.move(10);
```

## Quick practice

1. **Create a class**: Build a `BankAccount` class with a private `balance` property, a constructor that sets the initial balance, and public `deposit()` and `withdraw()` methods.

2. **Inheritance challenge**: Create a `Vehicle` base class with a `speed` property and a `move()` method. Then create a `Car` class that extends `Vehicle` and adds a `brand` property.

3. **Access modifiers**: What happens if you try to access a private property from outside the class? Test it with a simple example.

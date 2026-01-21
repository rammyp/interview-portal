# TypeScript Interview Preparation Guide

A comprehensive guide covering fundamental concepts through advanced topics for TypeScript interviews.

---

## Part 1: Core Concepts

### What is TypeScript?

TypeScript is a statically typed superset of JavaScript developed by Microsoft. It compiles to plain JavaScript and adds optional static typing, classes, interfaces, and other features for building large-scale applications.

### Why TypeScript?

| Benefit | Description |
|---------|-------------|
| Type Safety | Catch errors at compile time, not runtime |
| Better IDE Support | Autocomplete, refactoring, navigation |
| Self-Documenting | Types serve as inline documentation |
| Easier Refactoring | Compiler catches breaking changes |
| Modern Features | Use latest JS features with backward compatibility |

### Basic Types

```typescript
// Primitives
let name: string = "Alice";
let age: number = 30;
let isActive: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;

// Arrays
let numbers: number[] = [1, 2, 3];
let names: Array<string> = ["Alice", "Bob"];

// Tuple (fixed-length array with specific types)
let person: [string, number] = ["Alice", 30];

// Enum
enum Status {
  Pending,    // 0
  Approved,   // 1
  Rejected    // 2
}
let currentStatus: Status = Status.Pending;

// String enum
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT"
}

// Any (avoid when possible)
let flexible: any = "hello";
flexible = 42; // No error

// Unknown (safer than any)
let uncertain: unknown = "hello";
// Must narrow type before using
if (typeof uncertain === "string") {
  console.log(uncertain.toUpperCase());
}

// Void (function returns nothing)
function log(message: string): void {
  console.log(message);
}

// Never (function never returns)
function throwError(message: string): never {
  throw new Error(message);
}
```

### Type Inference

TypeScript can infer types automatically:

```typescript
let message = "Hello"; // Inferred as string
let count = 42;        // Inferred as number

// Function return type inference
function add(a: number, b: number) {
  return a + b; // Return type inferred as number
}

// Best practice: Let TS infer when obvious, 
// annotate for function parameters and complex cases
```

---

## Part 2: Interfaces and Type Aliases

### Interfaces

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age?: number;              // Optional property
  readonly createdAt: Date;  // Cannot be modified
}

const user: User = {
  id: 1,
  name: "Alice",
  email: "alice@example.com",
  createdAt: new Date()
};

// Function signature in interface
interface Calculator {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
}

// Index signature
interface StringMap {
  [key: string]: string;
}

const headers: StringMap = {
  "Content-Type": "application/json",
  "Authorization": "Bearer token"
};
```

### Extending Interfaces

```typescript
interface Person {
  name: string;
  age: number;
}

interface Employee extends Person {
  employeeId: string;
  department: string;
}

// Multiple inheritance
interface Manager extends Employee {
  teamSize: number;
}

// Merging interfaces (declaration merging)
interface Config {
  apiUrl: string;
}

interface Config {
  timeout: number;
}

// Config now has both apiUrl and timeout
```

### Type Aliases

```typescript
// Primitive alias
type ID = string | number;

// Object type
type Point = {
  x: number;
  y: number;
};

// Union type
type Status = "pending" | "approved" | "rejected";

// Function type
type Callback = (data: string) => void;

// Intersection type
type Employee = Person & {
  employeeId: string;
};
```

### Interface vs Type Alias

| Feature | Interface | Type Alias |
|---------|-----------|------------|
| Extend/Inherit | `extends` keyword | Intersection `&` |
| Declaration Merging | ✅ Yes | ❌ No |
| Primitives/Unions | ❌ No | ✅ Yes |
| Computed Properties | ❌ No | ✅ Yes |
| Performance | Slightly better | Slightly slower |

**Rule of thumb**: Use `interface` for object shapes, `type` for unions, primitives, and complex types.

---

## Part 3: Functions

### Function Types

```typescript
// Function declaration with types
function greet(name: string): string {
  return `Hello, ${name}!`;
}

// Arrow function
const add = (a: number, b: number): number => a + b;

// Function type alias
type MathOperation = (a: number, b: number) => number;

const multiply: MathOperation = (a, b) => a * b;

// Optional and default parameters
function createUser(name: string, age?: number, role: string = "user"): void {
  console.log(name, age, role);
}

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, n) => acc + n, 0);
}
```

### Function Overloading

```typescript
// Overload signatures
function format(value: string): string;
function format(value: number): string;
function format(value: Date): string;

// Implementation signature
function format(value: string | number | Date): string {
  if (typeof value === "string") {
    return value.trim();
  } else if (typeof value === "number") {
    return value.toFixed(2);
  } else {
    return value.toISOString();
  }
}

format("hello");  // Returns string
format(42);       // Returns string
format(new Date()); // Returns string
```

### `this` Parameter

```typescript
interface Button {
  label: string;
  onClick(this: Button): void;
}

const button: Button = {
  label: "Submit",
  onClick() {
    console.log(this.label); // 'this' is typed as Button
  }
};
```

---

## Part 4: Generics

### Basic Generics

```typescript
// Generic function
function identity<T>(value: T): T {
  return value;
}

identity<string>("hello"); // Explicit type
identity(42);              // Type inferred as number

// Generic interface
interface Container<T> {
  value: T;
  getValue(): T;
}

// Generic class
class Box<T> {
  private content: T;

  constructor(value: T) {
    this.content = value;
  }

  getContent(): T {
    return this.content;
  }
}

const stringBox = new Box<string>("hello");
const numberBox = new Box(42); // Inferred
```

### Generic Constraints

```typescript
// Constraint with extends
function getLength<T extends { length: number }>(item: T): number {
  return item.length;
}

getLength("hello");     // ✅ string has length
getLength([1, 2, 3]);   // ✅ array has length
getLength({ length: 5 }); // ✅ object with length
// getLength(42);       // ❌ number has no length

// Constraint with keyof
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Alice", age: 30 };
getProperty(user, "name"); // ✅ Returns string
// getProperty(user, "email"); // ❌ "email" is not a key of user
```

### Multiple Type Parameters

```typescript
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const result = merge({ name: "Alice" }, { age: 30 });
// result: { name: string; age: number }

// With constraints
function combine<T extends object, U extends object>(a: T, b: U): T & U {
  return { ...a, ...b };
}
```

### Generic Defaults

```typescript
interface ApiResponse<T = unknown> {
  data: T;
  status: number;
  message: string;
}

// T defaults to unknown if not specified
const response: ApiResponse = {
  data: "anything",
  status: 200,
  message: "OK"
};

const userResponse: ApiResponse<User> = {
  data: { id: 1, name: "Alice", email: "alice@example.com", createdAt: new Date() },
  status: 200,
  message: "OK"
};
```

---

## Part 5: Advanced Types

### Union and Intersection Types

```typescript
// Union: value can be one of several types
type StringOrNumber = string | number;

function format(value: StringOrNumber): string {
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  return value.toFixed(2);
}

// Intersection: combine multiple types
type HasName = { name: string };
type HasAge = { age: number };
type Person = HasName & HasAge;

const person: Person = { name: "Alice", age: 30 };
```

### Literal Types

```typescript
// String literal
type Direction = "north" | "south" | "east" | "west";

// Numeric literal
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;

// Boolean literal
type Yes = true;

// Template literal types
type EventName = `on${string}`;
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE";
type Endpoint = `/${string}`;
type ApiRoute = `${HttpMethod} ${Endpoint}`;
```

### Type Guards

```typescript
// typeof guard
function process(value: string | number) {
  if (typeof value === "string") {
    return value.toUpperCase(); // TypeScript knows it's string
  }
  return value * 2; // TypeScript knows it's number
}

// instanceof guard
class Dog {
  bark() { console.log("Woof!"); }
}

class Cat {
  meow() { console.log("Meow!"); }
}

function speak(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}

// in operator guard
interface Fish {
  swim(): void;
}

interface Bird {
  fly(): void;
}

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    animal.swim();
  } else {
    animal.fly();
  }
}

// Custom type guard
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function example(value: unknown) {
  if (isString(value)) {
    console.log(value.toUpperCase()); // TypeScript knows it's string
  }
}
```

### Discriminated Unions

```typescript
interface Circle {
  kind: "circle";
  radius: number;
}

interface Rectangle {
  kind: "rectangle";
  width: number;
  height: number;
}

interface Triangle {
  kind: "triangle";
  base: number;
  height: number;
}

type Shape = Circle | Rectangle | Triangle;

function calculateArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
    case "triangle":
      return (shape.base * shape.height) / 2;
  }
}
```

### Conditional Types

```typescript
// Basic conditional type
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false

// Infer keyword
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type FuncReturn = ReturnType<() => string>; // string

// Extract element type from array
type ElementType<T> = T extends (infer E)[] ? E : never;

type Item = ElementType<string[]>; // string

// Distributive conditional types
type ToArray<T> = T extends any ? T[] : never;

type StrOrNumArray = ToArray<string | number>; // string[] | number[]
```

### Mapped Types

```typescript
// Make all properties optional
type Partial<T> = {
  [P in keyof T]?: T[P];
};

// Make all properties required
type Required<T> = {
  [P in keyof T]-?: T[P];
};

// Make all properties readonly
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

// Pick specific properties
type Pick<T, K extends keyof T> = {
  [P in K]: T[P];
};

// Custom mapped type
type Nullable<T> = {
  [P in keyof T]: T[P] | null;
};

interface User {
  name: string;
  age: number;
}

type NullableUser = Nullable<User>;
// { name: string | null; age: number | null }
```

### Utility Types Reference

```typescript
// Partial<T> - Make all properties optional
type PartialUser = Partial<User>;

// Required<T> - Make all properties required
type RequiredUser = Required<PartialUser>;

// Readonly<T> - Make all properties readonly
type ReadonlyUser = Readonly<User>;

// Pick<T, K> - Select specific properties
type UserName = Pick<User, "name">;

// Omit<T, K> - Exclude specific properties
type UserWithoutAge = Omit<User, "age">;

// Record<K, V> - Create object type with keys K and values V
type UserRoles = Record<string, string[]>;

// Exclude<T, U> - Remove types from union
type NotString = Exclude<string | number | boolean, string>; // number | boolean

// Extract<T, U> - Extract types from union
type OnlyString = Extract<string | number | boolean, string>; // string

// NonNullable<T> - Remove null and undefined
type NotNull = NonNullable<string | null | undefined>; // string

// ReturnType<T> - Get function return type
type FnReturn = ReturnType<() => string>; // string

// Parameters<T> - Get function parameter types as tuple
type FnParams = Parameters<(a: string, b: number) => void>; // [string, number]

// Awaited<T> - Unwrap Promise type
type Resolved = Awaited<Promise<string>>; // string
```

---

## Part 6: Object-Oriented Programming

### Classes

```typescript
class Animal {
  // Properties
  protected name: string;
  private age: number;
  public readonly species: string;

  // Constructor
  constructor(name: string, age: number, species: string) {
    this.name = name;
    this.age = age;
    this.species = species;
  }

  // Method
  speak(): void {
    console.log(`${this.name} makes a sound`);
  }

  // Getter
  get animalAge(): number {
    return this.age;
  }

  // Setter
  set animalAge(value: number) {
    if (value >= 0) {
      this.age = value;
    }
  }

  // Static method
  static isAnimal(obj: any): obj is Animal {
    return obj instanceof Animal;
  }
}
```

### Parameter Properties (Shorthand)

```typescript
class User {
  constructor(
    public name: string,
    private email: string,
    readonly id: number
  ) {}
  // Properties automatically created and assigned
}

// Equivalent to:
class UserVerbose {
  public name: string;
  private email: string;
  readonly id: number;

  constructor(name: string, email: string, id: number) {
    this.name = name;
    this.email = email;
    this.id = id;
  }
}
```

### Inheritance

```typescript
class Dog extends Animal {
  private breed: string;

  constructor(name: string, age: number, breed: string) {
    super(name, age, "Canine"); // Call parent constructor
    this.breed = breed;
  }

  // Override method
  speak(): void {
    console.log(`${this.name} barks!`);
  }

  // New method
  fetch(): void {
    console.log(`${this.name} fetches the ball`);
  }
}
```

### Abstract Classes

```typescript
abstract class Shape {
  abstract getArea(): number;
  abstract getPerimeter(): number;

  // Concrete method
  describe(): string {
    return `Area: ${this.getArea()}, Perimeter: ${this.getPerimeter()}`;
  }
}

class Circle extends Shape {
  constructor(private radius: number) {
    super();
  }

  getArea(): number {
    return Math.PI * this.radius ** 2;
  }

  getPerimeter(): number {
    return 2 * Math.PI * this.radius;
  }
}
```

### Implementing Interfaces

```typescript
interface Printable {
  print(): void;
}

interface Loggable {
  log(message: string): void;
}

class Report implements Printable, Loggable {
  print(): void {
    console.log("Printing report...");
  }

  log(message: string): void {
    console.log(`Log: ${message}`);
  }
}
```

### Access Modifiers

| Modifier | Class | Subclass | Outside |
|----------|-------|----------|---------|
| `public` | ✅ | ✅ | ✅ |
| `protected` | ✅ | ✅ | ❌ |
| `private` | ✅ | ❌ | ❌ |

---

## Part 7: Modules and Namespaces

### ES Modules

```typescript
// math.ts - Named exports
export const PI = 3.14159;

export function add(a: number, b: number): number {
  return a + b;
}

export interface MathResult {
  value: number;
  operation: string;
}

// Default export
export default class Calculator {
  // ...
}
```

```typescript
// main.ts - Importing
import Calculator, { PI, add, MathResult } from "./math";
import * as MathUtils from "./math";
import { add as addNumbers } from "./math";

// Type-only imports (erased at runtime)
import type { MathResult } from "./math";
```

### Re-exporting

```typescript
// index.ts - Barrel file
export { User } from "./user";
export { Product } from "./product";
export * from "./utils";
export { default as Calculator } from "./calculator";
```

### Namespaces (Legacy)

```typescript
namespace Validation {
  export interface Validator {
    isValid(value: string): boolean;
  }

  export class EmailValidator implements Validator {
    isValid(value: string): boolean {
      return value.includes("@");
    }
  }
}

const validator = new Validation.EmailValidator();
```

**Note**: Prefer ES modules over namespaces in modern TypeScript.

### Declaration Files (.d.ts)

```typescript
// types.d.ts
declare module "my-library" {
  export function doSomething(value: string): number;
  export interface Config {
    debug: boolean;
  }
}

// Global declarations
declare global {
  interface Window {
    myCustomProperty: string;
  }
}

// Ambient declarations
declare const VERSION: string;
declare function legacyFunction(x: number): void;
```

---

## Part 8: TypeScript Configuration

### tsconfig.json Essentials

```json
{
  "compilerOptions": {
    // Target and module
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2022", "DOM"],

    // Output
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "sourceMap": true,

    // Strict type checking
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "noImplicitThis": true,

    // Additional checks
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,

    // Interop
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,

    // Path mapping
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Key Compiler Options Explained

| Option | Purpose |
|--------|---------|
| `strict` | Enable all strict type checks |
| `strictNullChecks` | `null` and `undefined` are distinct types |
| `noImplicitAny` | Error on implicit `any` type |
| `esModuleInterop` | Better CommonJS/ES module interop |
| `skipLibCheck` | Skip type checking of .d.ts files (faster builds) |
| `resolveJsonModule` | Allow importing JSON files |
| `isolatedModules` | Ensure each file can be transpiled independently |

---

## Part 9: Advanced Interview Questions

### Question 1: What's the difference between `unknown` and `any`?

**Answer**:

```typescript
// any: Disables type checking entirely
let anyValue: any = "hello";
anyValue.foo();           // No error (but crashes at runtime)
anyValue.toFixed();       // No error

// unknown: Type-safe version of any
let unknownValue: unknown = "hello";
// unknownValue.foo();    // Error! Must narrow type first

if (typeof unknownValue === "string") {
  unknownValue.toUpperCase(); // OK after type guard
}
```

**Key difference**: `unknown` requires type narrowing before use, making it safer than `any`.

---

### Question 2: Explain the `infer` keyword

**Answer**:

`infer` is used in conditional types to extract and capture a type:

```typescript
// Extract return type
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type A = GetReturnType<() => string>;           // string
type B = GetReturnType<(x: number) => boolean>; // boolean

// Extract array element type
type Flatten<T> = T extends Array<infer Item> ? Item : T;

type C = Flatten<string[]>;  // string
type D = Flatten<number>;    // number

// Extract Promise resolved type
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type E = UnwrapPromise<Promise<string>>; // string
```

---

### Question 3: How does TypeScript's structural typing work?

**Answer**:

TypeScript uses structural typing (duck typing), not nominal typing:

```typescript
interface Point {
  x: number;
  y: number;
}

interface Coordinate {
  x: number;
  y: number;
}

const point: Point = { x: 1, y: 2 };
const coord: Coordinate = point; // ✅ OK - same structure

// Works with any object that has required properties
function logPoint(p: Point) {
  console.log(p.x, p.y);
}

logPoint({ x: 1, y: 2 });              // ✅ OK
logPoint({ x: 1, y: 2, z: 3 });        // ✅ OK - extra properties allowed
// logPoint({ x: 1 });                 // ❌ Error - missing y
```

---

### Question 4: What are template literal types?

**Answer**:

```typescript
// Basic template literal type
type Greeting = `Hello, ${string}!`;

const a: Greeting = "Hello, World!";  // ✅
// const b: Greeting = "Hi, World!";  // ❌

// Combining with unions
type Color = "red" | "blue" | "green";
type Size = "small" | "large";
type Style = `${Size}-${Color}`;
// "small-red" | "small-blue" | "small-green" | "large-red" | ...

// Practical example: Event handlers
type EventName = "click" | "focus" | "blur";
type Handler = `on${Capitalize<EventName>}`;
// "onClick" | "onFocus" | "onBlur"

// Intrinsic string manipulation types
type Upper = Uppercase<"hello">;      // "HELLO"
type Lower = Lowercase<"HELLO">;      // "hello"
type Cap = Capitalize<"hello">;       // "Hello"
type Uncap = Uncapitalize<"Hello">;   // "hello"
```

---

### Question 5: Explain variance in TypeScript (covariance, contravariance)

**Answer**:

```typescript
class Animal {
  name: string = "";
}

class Dog extends Animal {
  breed: string = "";
}

// Covariance (output positions) - subtype can replace supertype
type Producer<T> = () => T;

let produceDog: Producer<Dog> = () => new Dog();
let produceAnimal: Producer<Animal> = produceDog; // ✅ OK

// Contravariance (input positions) - supertype can replace subtype
type Consumer<T> = (item: T) => void;

let consumeAnimal: Consumer<Animal> = (a) => console.log(a.name);
let consumeDog: Consumer<Dog> = consumeAnimal; // ✅ OK

// Strict function types enforce this
// Enable with: "strictFunctionTypes": true
```

---

### Question 6: How do you make a type deeply readonly?

**Answer**:

```typescript
// Basic version
type DeepReadonlyBasic<T> = {
  readonly [P in keyof T]: T[P] extends object
    ? T[P] extends Function
      ? T[P]
      : DeepReadonlyBasic<T[P]>
    : T[P];
};

// Robust version (handles Date, Map, Set, Arrays, and other built-ins)
type Primitive = string | number | boolean | bigint | symbol | null | undefined;

type DeepReadonly<T> = T extends Primitive | Function
  ? T
  : T extends Date | RegExp | Error
    ? T
    : T extends Map<infer K, infer V>
      ? ReadonlyMap<K, DeepReadonly<V>>
      : T extends Set<infer U>
        ? ReadonlySet<DeepReadonly<U>>
        : T extends Array<infer U>
          ? ReadonlyArray<DeepReadonly<U>>
          : T extends object
            ? { readonly [K in keyof T]: DeepReadonly<T[K]> }
            : T;

interface User {
  name: string;
  createdAt: Date;
  tags: string[];
  metadata: Map<string, any>;
  address: {
    city: string;
    country: string;
  };
}

type ReadonlyUser = DeepReadonly<User>;

const user: ReadonlyUser = {
  name: "Alice",
  createdAt: new Date(),
  tags: ["admin", "active"],
  metadata: new Map(),
  address: { city: "NYC", country: "USA" }
};

// user.name = "Bob";              // ❌ Error
// user.address.city = "LA";       // ❌ Error
// user.tags.push("new");          // ❌ Error - ReadonlyArray
// user.tags[0] = "user";          // ❌ Error
// user.createdAt.getTime();       // ✅ OK - Date methods still work
```

---

### Question 7: What is declaration merging?

**Answer**:

TypeScript merges multiple declarations with the same name:

```typescript
// Interface merging
interface User {
  name: string;
}

interface User {
  age: number;
}

// User now has both name and age
const user: User = { name: "Alice", age: 30 };

// Namespace merging with class
class Album {
  label: Album.AlbumLabel = { name: "Default" };
}

namespace Album {
  export interface AlbumLabel {
    name: string;
  }
}

// Module augmentation
declare module "express" {
  interface Request {
    user?: { id: string };
  }
}
```

---

### Question 8: Explain the `satisfies` operator

**Answer**:

`satisfies` validates a type while preserving the narrowest inferred type:

```typescript
type Colors = "red" | "green" | "blue";
type RGB = [number, number, number];

// Without satisfies - type is widened
const palette1: Record<Colors, string | RGB> = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255]
};
palette1.green.toUpperCase(); // ❌ Error - could be RGB

// With satisfies - keeps narrow type
const palette2 = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255]
} satisfies Record<Colors, string | RGB>;

palette2.green.toUpperCase(); // ✅ OK - TypeScript knows it's string
palette2.red[0];              // ✅ OK - TypeScript knows it's RGB
```

---

### Question 9: How do you handle async/await typing?

**Answer**:

```typescript
// Async function return type is automatically Promise<T>
async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

// Typing async callbacks
const users: User[] = await Promise.all(
  ids.map(async (id): Promise<User> => {
    return fetchUser(id);
  })
);

// Using Awaited utility type
type Result = Awaited<Promise<Promise<string>>>; // string

// Error handling with types
async function safeFetch<T>(url: string): Promise<{ data: T } | { error: string }> {
  try {
    const response = await fetch(url);
    const data: T = await response.json();
    return { data };
  } catch (e) {
    return { error: e instanceof Error ? e.message : "Unknown error" };
  }
}
```

---

### Question 10: What are const assertions?

**Answer**:

```typescript
// Without const assertion
const config1 = {
  endpoint: "/api",
  method: "GET"
};
// Type: { endpoint: string; method: string }

// With const assertion
const config2 = {
  endpoint: "/api",
  method: "GET"
} as const;
// Type: { readonly endpoint: "/api"; readonly method: "GET" }

// Array becomes readonly tuple
const arr = [1, 2, 3] as const;
// Type: readonly [1, 2, 3]

// Useful for literal types
function request(method: "GET" | "POST") {}

request(config1.method); // ❌ Error - string not assignable
request(config2.method); // ✅ OK - "GET" literal type
```

---

## Part 10: Practical Patterns

### Branded Types (Nominal Typing)

```typescript
type UserId = string & { __brand: "UserId" };
type OrderId = string & { __brand: "OrderId" };

function createUserId(id: string): UserId {
  return id as UserId;
}

function getUser(id: UserId) {
  // ...
}

const userId = createUserId("user-123");
const orderId = "order-456" as OrderId;

getUser(userId);   // ✅ OK
// getUser(orderId); // ❌ Error - OrderId not assignable to UserId
```

### Builder Pattern

```typescript
class RequestBuilder {
  private url: string = "";
  private method: "GET" | "POST" = "GET";
  private headers: Record<string, string> = {};

  setUrl(url: string): this {
    this.url = url;
    return this;
  }

  setMethod(method: "GET" | "POST"): this {
    this.method = method;
    return this;
  }

  addHeader(key: string, value: string): this {
    this.headers[key] = value;
    return this;
  }

  build(): Request {
    return new Request(this.url, {
      method: this.method,
      headers: this.headers
    });
  }
}

const request = new RequestBuilder()
  .setUrl("/api/users")
  .setMethod("POST")
  .addHeader("Content-Type", "application/json")
  .build();
```

### Result Type Pattern

```typescript
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

function divide(a: number, b: number): Result<number, string> {
  if (b === 0) {
    return { success: false, error: "Division by zero" };
  }
  return { success: true, data: a / b };
}

const result = divide(10, 2);

if (result.success) {
  console.log(result.data); // TypeScript knows data exists
} else {
  console.log(result.error); // TypeScript knows error exists
}
```

### Type-Safe Event Emitter

```typescript
type EventMap = {
  userLogin: { userId: string; timestamp: Date };
  userLogout: { userId: string };
  error: { message: string; code: number };
};

class TypedEventEmitter<T extends Record<string, any>> {
  private listeners: Partial<{
    [K in keyof T]: Array<(data: T[K]) => void>;
  }> = {};

  on<K extends keyof T>(event: K, listener: (data: T[K]) => void): void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]!.push(listener);
  }

  emit<K extends keyof T>(event: K, data: T[K]): void {
    this.listeners[event]?.forEach((listener) => listener(data));
  }
}

const emitter = new TypedEventEmitter<EventMap>();

emitter.on("userLogin", (data) => {
  console.log(data.userId); // ✅ Typed correctly
});

emitter.emit("userLogin", { userId: "123", timestamp: new Date() }); // ✅
// emitter.emit("userLogin", { wrong: "data" }); // ❌ Type error
```

---

## Part 11: Common Mistakes and Best Practices

### Mistakes to Avoid

```typescript
// ❌ Using any
function process(data: any) { ... }

// ✅ Use unknown with type guards
function process(data: unknown) {
  if (typeof data === "string") { ... }
}

// ❌ Type assertions without validation
const user = JSON.parse(data) as User;

// ✅ Runtime validation
function isUser(obj: unknown): obj is User {
  return typeof obj === "object" && obj !== null && "name" in obj;
}

// ❌ Ignoring null/undefined
function getLength(str: string) {
  return str.length; // Crashes if str is undefined
}

// ✅ Enable strictNullChecks and handle null
function getLength(str: string | undefined) {
  return str?.length ?? 0;
}

// ❌ Empty interfaces
interface Props {}

// ✅ Use Record or specific types
type Props = Record<string, never>; // Explicitly empty
```

### Best Practices

| Practice | Reason |
|----------|--------|
| Enable `strict` mode | Catches more errors at compile time |
| Prefer `unknown` over `any` | Forces type narrowing |
| Use `readonly` for immutable data | Prevents accidental mutations |
| Prefer interfaces for objects | Better error messages, declaration merging |
| Use type aliases for unions | More readable, reusable |
| Avoid type assertions | Use type guards instead |
| Use `satisfies` when appropriate | Validates while preserving narrow types |
| Keep types close to usage | Easier to maintain |

---

## Quick Reference: TypeScript vs JavaScript

| Feature | JavaScript | TypeScript |
|---------|------------|------------|
| Type checking | Runtime only | Compile time |
| Interfaces | ❌ | ✅ |
| Generics | ❌ | ✅ |
| Enums | ❌ | ✅ |
| Access modifiers | Limited | public, private, protected |
| Decorators | Stage 3 | Experimental support |
| Module systems | ES Modules | ES + CommonJS + custom |

---

## Additional Resources

- Official TypeScript Handbook: https://www.typescriptlang.org/docs/handbook/
- TypeScript Playground: https://www.typescriptlang.org/play
- Type Challenges: https://github.com/type-challenges/type-challenges
- TypeScript Deep Dive: https://basarat.gitbook.io/typescript/

---

*Good luck with your interview!*

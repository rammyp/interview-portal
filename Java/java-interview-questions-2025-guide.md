# Top 20 Java Interview Questions for 2025 - Complete Guide

## Introduction

This guide covers the most frequently asked Java interview questions in 2025, including the latest features from Java 21, core concepts, multithreading, collections, and design patterns. Whether you're a fresher or an experienced developer, these questions will help you prepare for your next Java interview.

---

# Core Java Fundamentals

---

## 1. What are the new features in Java 21?

### Answer

Java 21 (LTS release) introduced several important features:

| Feature | Description |
|---------|-------------|
| **Virtual Threads** | Lightweight threads for high concurrency (Project Loom) |
| **Record Patterns** | Pattern matching with record components |
| **Sequenced Collections** | Predictable ordering with `getFirst()`, `getLast()` |
| **Pattern Matching for switch** | Enhanced switch expressions |
| **String Templates** (Preview) | Dynamic string building |

### Code Examples

```java
// Virtual Threads
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
executor.submit(() -> System.out.println("Running in virtual thread"));
executor.shutdown();

// Creating virtual thread directly
Thread.startVirtualThread(() -> {
    System.out.println("Hello from virtual thread!");
});

// Sequenced Collections
SequencedCollection<String> names = new ArrayList<>();
names.add("Middle");
names.addFirst("First");
names.addLast("Last");
System.out.println(names.getFirst());  // First
System.out.println(names.getLast());   // Last

// Record Patterns
record Person(String name, int age) {}

void printPerson(Object obj) {
    switch (obj) {
        case Person(String name, int age) -> 
            System.out.println(name + " is " + age + " years old");
        default -> 
            System.out.println("Unknown object");
    }
}

// Pattern Matching for switch
String formatValue(Object obj) {
    return switch (obj) {
        case Integer i -> "Integer: " + i;
        case String s -> "String: " + s;
        case null -> "Null value";
        default -> "Unknown type";
    };
}
```

---

## 2. What is the difference between Virtual Threads and Platform Threads?

### Answer

Virtual Threads (introduced in Java 21) are lightweight threads managed by the JVM, while Platform Threads are traditional OS-managed threads.

| Aspect | Virtual Threads | Platform Threads |
|--------|-----------------|------------------|
| **Managed by** | JVM | Operating System |
| **Weight** | Lightweight (cheap to create) | Heavyweight (expensive) |
| **Scalability** | Millions can run concurrently | Limited by OS resources |
| **Best for** | I/O-bound tasks | CPU-bound tasks |
| **Stack size** | Shallow, grows as needed | Large, fixed size |
| **Context switching** | Cheap (JVM-level) | Expensive (OS-level) |
| **Memory footprint** | Small (~1KB initially) | Large (~1MB per thread) |

### Code Examples

```java
// Virtual Thread - multiple ways to create
// Method 1: Using Thread.startVirtualThread()
Thread vThread1 = Thread.startVirtualThread(() -> {
    System.out.println("Virtual thread 1");
});

// Method 2: Using Thread.ofVirtual()
Thread vThread2 = Thread.ofVirtual()
    .name("my-virtual-thread")
    .start(() -> {
        System.out.println("Virtual thread 2");
    });

// Method 3: Using ExecutorService
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 10000; i++) {
        executor.submit(() -> {
            // Handle task - can create millions of these!
            Thread.sleep(1000);
            return "Done";
        });
    }
}

// Platform Thread (traditional)
Thread pThread = new Thread(() -> {
    System.out.println("Platform thread");
});
pThread.start();

// Or using Thread.ofPlatform()
Thread pThread2 = Thread.ofPlatform()
    .name("my-platform-thread")
    .start(() -> {
        System.out.println("Platform thread 2");
    });
```

### When to Use What

| Use Virtual Threads When | Use Platform Threads When |
|--------------------------|---------------------------|
| High number of concurrent I/O operations | CPU-intensive calculations |
| Web servers handling many requests | Tasks requiring thread affinity |
| Database connection handling | Native code integration |
| File I/O operations | Real-time systems |
| Network operations | When you need ThreadLocal heavily |

---

## 3. What is the difference between `==` and `.equals()` in Java?

### Answer

| Operator/Method | Compares | Use Case |
|-----------------|----------|----------|
| `==` | **Reference** (memory address) | Primitives, reference equality |
| `.equals()` | **Content/Value** (when overridden) | Object content comparison |

### Code Examples

```java
// String comparison
String s1 = "Hello";
String s2 = "Hello";
String s3 = new String("Hello");

System.out.println(s1 == s2);      // true (same string pool reference)
System.out.println(s1 == s3);      // false (different objects)
System.out.println(s1.equals(s3)); // true (same content)

// Integer comparison (Integer cache: -128 to 127)
Integer a = 100;
Integer b = 100;
Integer c = 200;
Integer d = 200;

System.out.println(a == b);        // true (cached)
System.out.println(c == d);        // false (not cached, > 127)
System.out.println(c.equals(d));   // true (same value)

// Custom object comparison
class Person {
    String name;
    int age;
    
    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Person person = (Person) obj;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}

Person p1 = new Person("John", 30);
Person p2 = new Person("John", 30);

System.out.println(p1 == p2);      // false (different objects)
System.out.println(p1.equals(p2)); // true (same content)
```

### Key Points

1. **Always override `hashCode()` when overriding `equals()`**
2. **For Strings, prefer `.equals()` for content comparison**
3. **`==` for primitives compares actual values**
4. **`==` for objects compares memory references**

---

## 4. How does HashMap work internally?

### Answer

HashMap stores key-value pairs using hashing for O(1) average-time operations.

### Internal Structure

```
Index    Bucket Array
─────    ─────────────────────────────────────────────
  0      [ ] → null
  1      [ ] → Node("John", 25) → null
  2      [ ] → Node("Jane", 30) → Node("Bob", 35) → null  ← Collision!
  3      [ ] → null
  ...
  15     [ ] → null
```

### Key Components

```java
// Internal Node structure
static class Node<K,V> {
    final int hash;       // Cached hash value
    final K key;          // Key
    V value;              // Value
    Node<K,V> next;       // Next node (for collision)
}

// Key constants
static final int DEFAULT_INITIAL_CAPACITY = 16;
static final float DEFAULT_LOAD_FACTOR = 0.75f;
static final int TREEIFY_THRESHOLD = 8;      // LinkedList → Tree
static final int UNTREEIFY_THRESHOLD = 6;    // Tree → LinkedList
```

### How put() Works

```
put(key, value)
       │
       ▼
┌─────────────────────────┐
│ 1. Calculate hash       │
│    hash = hash(key)     │
└─────────────────────────┘
       │
       ▼
┌─────────────────────────┐
│ 2. Calculate index      │
│    i = (n-1) & hash     │
└─────────────────────────┘
       │
       ▼
┌─────────────────────────┐
│ 3. Bucket empty?        │
└─────────────────────────┘
       │
   ┌───┴───┐
  Yes      No
   │       │
   ▼       ▼
Create   Handle collision
Node     (LinkedList or Tree)
```

### Collision Handling

| Before Java 8 | Java 8+ |
|---------------|---------|
| LinkedList only | LinkedList (≤8 nodes) → Red-Black Tree (>8 nodes) |
| O(n) worst case | O(log n) worst case |

### Time Complexity

| Operation | Average | Worst Case |
|-----------|---------|------------|
| put() | O(1) | O(log n) |
| get() | O(1) | O(log n) |
| remove() | O(1) | O(log n) |
| containsKey() | O(1) | O(log n) |

### Code Example

```java
// HashMap operations
Map<String, Integer> map = new HashMap<>();

// put - calculates hash, finds bucket, stores
map.put("Apple", 10);   // hash("Apple") → index 5 → store
map.put("Banana", 20);  // hash("Banana") → index 11 → store
map.put("Cherry", 30);  // hash("Cherry") → index 5 → collision! → chain

// get - calculates hash, finds bucket, searches
int value = map.get("Apple");  // hash("Apple") → index 5 → find → return 10

// Resize happens when: size > capacity * loadFactor
// Default: 16 * 0.75 = 12 entries triggers resize to 32
```

---

## 5. Why is String immutable in Java?

### Answer

String is immutable for several important reasons:

| Reason | Explanation |
|--------|-------------|
| **Security** | Used in class loading, network connections, file paths |
| **Thread Safety** | Can be shared across threads without synchronization |
| **String Pool** | Enables string interning for memory optimization |
| **HashCode Caching** | Calculated once, reused (good for HashMap keys) |
| **Performance** | JVM optimizations possible due to immutability |

### How String Pool Works

```java
String s1 = "Hello";           // Created in String Pool
String s2 = "Hello";           // Reuses same reference from Pool
String s3 = new String("Hello"); // New object in Heap

System.out.println(s1 == s2);  // true (same pool reference)
System.out.println(s1 == s3);  // false (different objects)

// Interning
String s4 = s3.intern();       // Returns pool reference
System.out.println(s1 == s4);  // true (both point to pool)
```

### Memory Visualization

```
        String Pool (in Heap)          Heap
        ┌─────────────────┐      ┌─────────────────┐
        │   "Hello"       │←─┐   │  String Object  │
        │   "World"       │  │   │  value="Hello"  │
        └─────────────────┘  │   └─────────────────┘
                             │           ↑
   s1 ───────────────────────┤           │
   s2 ───────────────────────┘           │
   s3 ──────────────────────────────────┘
```

### Immutability in Action

```java
String original = "Hello";
String modified = original.concat(" World");

System.out.println(original);  // "Hello" (unchanged!)
System.out.println(modified);  // "Hello World" (new String)

// Every "modification" creates a new String
String s = "Hello";
s = s.toUpperCase();      // New String created
s = s.substring(0, 3);    // New String created
s = s + "!";              // New String created
```

### Why char[] is Preferred for Passwords

```java
// String - stays in memory until GC
String password = "secret123";  // Bad!
// Even after password = null, "secret123" may still be in String Pool

// char[] - can be explicitly cleared
char[] passwordChars = "secret123".toCharArray();  // Better
// Use password...
Arrays.fill(passwordChars, '0');  // Clear from memory immediately
```

---

# Object-Oriented Programming

---

## 6. What is the difference between Abstract Class and Interface?

### Answer

| Feature | Abstract Class | Interface |
|---------|----------------|-----------|
| **Methods** | Abstract + concrete | Abstract + default + static |
| **Variables** | Any type | Only public static final |
| **Constructor** | Can have | Cannot have |
| **Inheritance** | Single | Multiple |
| **Access modifiers** | Any | Public (default) |
| **Use case** | IS-A relationship with shared code | Contract/capability definition |

### Java 8+ Interface Evolution

```java
// Modern Interface (Java 8+)
public interface Vehicle {
    
    // Abstract method (must implement)
    void start();
    
    // Default method (optional override)
    default void stop() {
        System.out.println("Vehicle stopped");
    }
    
    // Static method (cannot override)
    static void service() {
        System.out.println("Service vehicle");
    }
    
    // Private method (Java 9+) - helper for default methods
    private void log(String msg) {
        System.out.println("LOG: " + msg);
    }
}
```

### Abstract Class Example

```java
public abstract class Animal {
    
    protected String name;
    
    // Constructor
    public Animal(String name) {
        this.name = name;
    }
    
    // Abstract method
    public abstract void makeSound();
    
    // Concrete method
    public void sleep() {
        System.out.println(name + " is sleeping");
    }
}

public class Dog extends Animal {
    
    public Dog(String name) {
        super(name);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " says Woof!");
    }
}
```

### When to Use What

| Use Abstract Class | Use Interface |
|--------------------|---------------|
| Share code among related classes | Define a contract for unrelated classes |
| Need constructors | Need multiple inheritance |
| Need non-public members | Define capabilities (Comparable, Serializable) |
| Need instance variables | API definitions |
| Classes have IS-A relationship | Classes have CAN-DO relationship |

---

## 7. Explain SOLID Principles

### Answer

SOLID is an acronym for five design principles that help create maintainable software.

### S - Single Responsibility Principle

> A class should have only one reason to change.

```java
// ❌ Bad - Multiple responsibilities
class Employee {
    void calculatePay() { }
    void saveToDatabase() { }
    void generateReport() { }
}

// ✅ Good - Single responsibility each
class Employee { 
    private String name;
    private double salary;
}

class PayrollCalculator {
    double calculatePay(Employee e) { }
}

class EmployeeRepository {
    void save(Employee e) { }
}

class ReportGenerator {
    void generate(Employee e) { }
}
```

### O - Open/Closed Principle

> Open for extension, closed for modification.

```java
// ❌ Bad - Must modify class for new shapes
class AreaCalculator {
    double calculate(Object shape) {
        if (shape instanceof Rectangle) { }
        else if (shape instanceof Circle) { }
        // Must add new if-else for each new shape
    }
}

// ✅ Good - Extend without modifying
interface Shape {
    double area();
}

class Rectangle implements Shape {
    public double area() { return width * height; }
}

class Circle implements Shape {
    public double area() { return Math.PI * radius * radius; }
}

// Adding new shape doesn't require changing AreaCalculator
class Triangle implements Shape {
    public double area() { return 0.5 * base * height; }
}
```

### L - Liskov Substitution Principle

> Subtypes must be substitutable for their base types.

```java
// ❌ Bad - Square breaks Rectangle contract
class Rectangle {
    void setWidth(int w) { width = w; }
    void setHeight(int h) { height = h; }
}

class Square extends Rectangle {
    void setWidth(int w) { width = w; height = w; }  // Breaks contract!
    void setHeight(int h) { width = h; height = h; }
}

// ✅ Good - Proper abstraction
interface Shape {
    double area();
}

class Rectangle implements Shape { }
class Square implements Shape { }
```

### I - Interface Segregation Principle

> Many specific interfaces are better than one general interface.

```java
// ❌ Bad - Fat interface
interface Worker {
    void work();
    void eat();
    void sleep();
}

class Robot implements Worker {
    void work() { }
    void eat() { /* Robots don't eat! */ }   // Forced to implement
    void sleep() { /* Robots don't sleep! */ }
}

// ✅ Good - Segregated interfaces
interface Workable { void work(); }
interface Eatable { void eat(); }
interface Sleepable { void sleep(); }

class Human implements Workable, Eatable, Sleepable { }
class Robot implements Workable { }  // Only what's needed
```

### D - Dependency Inversion Principle

> Depend on abstractions, not concretions.

```java
// ❌ Bad - Depends on concrete class
class NotificationService {
    private EmailSender emailSender = new EmailSender();
    
    void notify(String message) {
        emailSender.send(message);
    }
}

// ✅ Good - Depends on abstraction
interface MessageSender {
    void send(String message);
}

class EmailSender implements MessageSender { }
class SmsSender implements MessageSender { }

class NotificationService {
    private MessageSender sender;
    
    NotificationService(MessageSender sender) {  // Inject dependency
        this.sender = sender;
    }
    
    void notify(String message) {
        sender.send(message);
    }
}
```

---

## 8. What are Records in Java?

### Answer

Records (introduced in Java 14, finalized in Java 16) are immutable data carriers that eliminate boilerplate code.

### Comparison: Traditional Class vs Record

```java
// Traditional class: ~50 lines
public final class Person {
    private final String name;
    private final int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
    
    @Override
    public String toString() {
        return "Person[name=" + name + ", age=" + age + "]";
    }
}

// Record: 1 line!
public record Person(String name, int age) {}
```

### What Records Auto-Generate

| Component | Generated |
|-----------|-----------|
| Private final fields | `private final String name;` |
| All-args constructor | `public Person(String name, int age)` |
| Accessor methods | `name()`, `age()` (no "get" prefix) |
| `equals()` | Component-based equality |
| `hashCode()` | Component-based hash |
| `toString()` | `Person[name=John, age=30]` |

### Record Features

```java
// Basic record
public record Person(String name, int age) {}

// Usage
Person p = new Person("John", 30);
String name = p.name();  // Accessor (not getName())
int age = p.age();

// Record with validation (compact constructor)
public record Person(String name, int age) {
    public Person {  // Compact constructor
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name required");
        }
        if (age < 0) {
            throw new IllegalArgumentException("Age must be positive");
        }
        name = name.trim();  // Can modify before assignment
    }
}

// Record with custom methods
public record Rectangle(double width, double height) {
    
    public double area() {
        return width * height;
    }
    
    public double perimeter() {
        return 2 * (width + height);
    }
    
    // "with" pattern for creating modified copies
    public Rectangle withWidth(double newWidth) {
        return new Rectangle(newWidth, this.height);
    }
}

// Record with static members
public record Point(int x, int y) {
    public static final Point ORIGIN = new Point(0, 0);
    
    public static Point of(int x, int y) {
        return new Point(x, y);
    }
}
```

### Record Limitations

| Cannot | Can |
|--------|-----|
| Extend other classes | Implement interfaces |
| Be extended (implicitly final) | Have static fields/methods |
| Have mutable fields | Have instance methods |
| Have additional instance fields | Have compact constructors |

### Records with Interfaces

```java
public interface Printable {
    void print();
}

public record Document(String title, String content) implements Printable {
    @Override
    public void print() {
        System.out.println(title + ": " + content);
    }
}
```

---

# Collections & Streams

---

## 9. What is the difference between ArrayList and LinkedList?

### Answer

| Aspect | ArrayList | LinkedList |
|--------|-----------|------------|
| **Internal structure** | Dynamic array | Doubly linked list |
| **Random access** | O(1) - direct index | O(n) - traverse |
| **Insert at end** | O(1) amortized | O(1) |
| **Insert at beginning** | O(n) - shift elements | O(1) |
| **Insert in middle** | O(n) | O(1)* |
| **Memory** | Less (contiguous) | More (node pointers) |
| **Cache locality** | Better | Worse |

*O(1) insertion after finding position, but finding is O(n)

### Visual Comparison

```
ArrayList (Dynamic Array):
┌───┬───┬───┬───┬───┬───┬───┬───┐
│ A │ B │ C │ D │ E │   │   │   │
└───┴───┴───┴───┴───┴───┴───┴───┘
  0   1   2   3   4   (contiguous memory)

LinkedList (Doubly Linked List):
┌───────┐    ┌───────┐    ┌───────┐    ┌───────┐
│ A     │◄──►│ B     │◄──►│ C     │◄──►│ D     │
│ prev  │    │ prev  │    │ prev  │    │ prev  │
│ next  │    │ next  │    │ next  │    │ next  │
└───────┘    └───────┘    └───────┘    └───────┘
  (scattered in memory)
```

### Code Examples

```java
// ArrayList - best for random access and iteration
List<String> arrayList = new ArrayList<>();
arrayList.add("A");
arrayList.add("B");
arrayList.get(0);  // O(1) - fast!

// LinkedList - best for frequent insertions/deletions
List<String> linkedList = new LinkedList<>();
linkedList.addFirst("A");  // O(1) - fast!
linkedList.addLast("B");   // O(1) - fast!
linkedList.get(0);         // O(n) - slow!

// LinkedList also implements Deque
Deque<String> deque = new LinkedList<>();
deque.push("A");    // Stack operation
deque.pop();        // Stack operation
deque.offer("B");   // Queue operation
deque.poll();       // Queue operation
```

### When to Use What

| Use ArrayList | Use LinkedList |
|---------------|----------------|
| Frequent random access by index | Frequent add/remove at ends |
| Mostly read operations | Implementing Queue/Deque |
| Need cache-friendly iteration | Don't need random access |
| Default choice for most cases | Memory not a concern |

---

## 10. Explain Java Streams and give examples

### Answer

Streams provide functional-style operations on sequences of elements.

### Stream Pipeline

```
Source → Intermediate Operations → Terminal Operation
         (lazy, return Stream)    (eager, produce result)
```

### Creating Streams

```java
// From Collection
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream1 = list.stream();

// From Array
String[] array = {"a", "b", "c"};
Stream<String> stream2 = Arrays.stream(array);

// Using Stream.of()
Stream<String> stream3 = Stream.of("a", "b", "c");

// Infinite streams
Stream<Integer> infinite = Stream.iterate(0, n -> n + 1);
Stream<Double> randoms = Stream.generate(Math::random);

// Primitive streams
IntStream intStream = IntStream.range(1, 10);  // 1 to 9
IntStream intStream2 = IntStream.rangeClosed(1, 10);  // 1 to 10
```

### Intermediate Operations (Lazy)

```java
List<String> names = Arrays.asList("John", "Jane", "Bob", "Alice", "Anna");

// filter - keep elements matching predicate
names.stream()
    .filter(n -> n.startsWith("A"))  // [Alice, Anna]

// map - transform elements
names.stream()
    .map(String::toUpperCase)  // [JOHN, JANE, BOB, ALICE, ANNA]

// flatMap - flatten nested structures
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4)
);
nested.stream()
    .flatMap(List::stream)  // [1, 2, 3, 4]

// sorted - sort elements
names.stream()
    .sorted()  // [Alice, Anna, Bob, Jane, John]

// distinct - remove duplicates
Stream.of(1, 2, 2, 3, 3, 3)
    .distinct()  // [1, 2, 3]

// limit & skip
names.stream()
    .skip(2)
    .limit(2)  // [Bob, Alice]

// peek - debug/inspect
names.stream()
    .peek(n -> System.out.println("Processing: " + n))
    .filter(n -> n.length() > 3)
    .collect(Collectors.toList());
```

### Terminal Operations (Eager)

```java
List<String> names = Arrays.asList("John", "Jane", "Bob", "Alice");

// collect - gather into collection
List<String> list = names.stream()
    .filter(n -> n.length() > 3)
    .collect(Collectors.toList());

// forEach - perform action
names.stream()
    .forEach(System.out::println);

// count - count elements
long count = names.stream()
    .filter(n -> n.startsWith("J"))
    .count();  // 2

// findFirst / findAny
Optional<String> first = names.stream()
    .filter(n -> n.startsWith("A"))
    .findFirst();  // Optional[Alice]

// anyMatch / allMatch / noneMatch
boolean anyStartsWithA = names.stream()
    .anyMatch(n -> n.startsWith("A"));  // true

boolean allLongerThan2 = names.stream()
    .allMatch(n -> n.length() > 2);  // true

boolean noneStartsWithZ = names.stream()
    .noneMatch(n -> n.startsWith("Z"));  // true

// reduce - combine elements
int sum = IntStream.of(1, 2, 3, 4, 5)
    .reduce(0, Integer::sum);  // 15

// min / max
Optional<String> longest = names.stream()
    .max(Comparator.comparing(String::length));
```

### Collectors Examples

```java
List<Person> people = Arrays.asList(
    new Person("John", 30, "Engineering"),
    new Person("Jane", 25, "Marketing"),
    new Person("Bob", 35, "Engineering")
);

// toList, toSet, toMap
List<String> nameList = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());

Map<String, Integer> nameToAge = people.stream()
    .collect(Collectors.toMap(Person::getName, Person::getAge));

// joining
String joined = people.stream()
    .map(Person::getName)
    .collect(Collectors.joining(", "));  // "John, Jane, Bob"

// groupingBy
Map<String, List<Person>> byDept = people.stream()
    .collect(Collectors.groupingBy(Person::getDepartment));

// groupingBy with counting
Map<String, Long> countByDept = people.stream()
    .collect(Collectors.groupingBy(
        Person::getDepartment, 
        Collectors.counting()
    ));

// partitioningBy
Map<Boolean, List<Person>> partitioned = people.stream()
    .collect(Collectors.partitioningBy(p -> p.getAge() > 30));

// summarizing
IntSummaryStatistics stats = people.stream()
    .collect(Collectors.summarizingInt(Person::getAge));
// count=3, sum=90, min=25, average=30.0, max=35
```

### Complete Example

```java
List<Employee> employees = getEmployees();

// Get names of top 3 highest paid engineers, sorted alphabetically
List<String> result = employees.stream()
    .filter(e -> e.getDepartment().equals("Engineering"))
    .sorted(Comparator.comparing(Employee::getSalary).reversed())
    .limit(3)
    .map(Employee::getName)
    .sorted()
    .collect(Collectors.toList());
```

---

## 11. What is the difference between `map()` and `flatMap()`?

### Answer

| Method | Input → Output | Use Case |
|--------|----------------|----------|
| `map()` | One element → One element | Transform each element |
| `flatMap()` | One element → Stream → Flattened | Flatten nested structures |

### Visual Comparison

```
map(): One-to-One
┌───┐     ┌───┐
│ A │ ──► │ a │
├───┤     ├───┤
│ B │ ──► │ b │
├───┤     ├───┤
│ C │ ──► │ c │
└───┘     └───┘

flatMap(): One-to-Many, then Flatten
┌───┐     ┌───┬───┐     ┌───┐
│ A │ ──► │ a │ 1 │ ──► │ a │
├───┤     └───┴───┘     ├───┤
│ B │ ──► ┌───┬───┐     │ 1 │
├───┤     │ b │ 2 │     ├───┤
│ C │ ──► └───┴───┘     │ b │
└───┘     ┌───┬───┐     ├───┤
          │ c │ 3 │     │ 2 │
          └───┴───┘     ├───┤
                        │ c │
                        ├───┤
                        │ 3 │
                        └───┘
```

### Code Examples

```java
// map - Transform each element (1:1)
List<String> words = Arrays.asList("Hello", "World");

List<Integer> lengths = words.stream()
    .map(String::length)
    .collect(Collectors.toList());
// [5, 5]

List<String> upperCase = words.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
// [HELLO, WORLD]


// flatMap - Flatten nested structures (1:N → flat)
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2, 3),
    Arrays.asList(4, 5, 6),
    Arrays.asList(7, 8, 9)
);

// Using map - gives Stream of Lists (not what we want)
List<Stream<Integer>> wrongResult = nested.stream()
    .map(List::stream)
    .collect(Collectors.toList());
// [Stream, Stream, Stream]

// Using flatMap - flattens to single stream
List<Integer> flatResult = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
// [1, 2, 3, 4, 5, 6, 7, 8, 9]


// Real-world example: Get all unique words from sentences
List<String> sentences = Arrays.asList(
    "Hello World",
    "Java Streams",
    "Hello Java"
);

List<String> allWords = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .distinct()
    .collect(Collectors.toList());
// [Hello, World, Java, Streams]


// Example with Optional
Optional<String> getName(int id) { }

List<Integer> ids = Arrays.asList(1, 2, 3);

// map gives Stream<Optional<String>>
// flatMap unwraps Optionals
List<String> names = ids.stream()
    .map(this::getName)
    .flatMap(Optional::stream)  // Java 9+
    .collect(Collectors.toList());


// Example: Orders with multiple items
List<Order> orders = getOrders();

// Get all items from all orders
List<Item> allItems = orders.stream()
    .flatMap(order -> order.getItems().stream())
    .collect(Collectors.toList());

// Total revenue
double total = orders.stream()
    .flatMap(order -> order.getItems().stream())
    .mapToDouble(item -> item.getPrice() * item.getQuantity())
    .sum();
```

---

# Multithreading & Concurrency

---

## 12. What is the difference between `synchronized` and `ReentrantLock`?

### Answer

| Feature | synchronized | ReentrantLock |
|---------|--------------|---------------|
| **Type** | Keyword (implicit) | Class (explicit) |
| **Lock acquisition** | Automatic | Manual (`lock()`) |
| **Lock release** | Automatic | Manual (`unlock()`) |
| **Try lock** | Not possible | `tryLock()`, `tryLock(timeout)` |
| **Interruptible** | No | `lockInterruptibly()` |
| **Fairness** | Not guaranteed | Optional fairness policy |
| **Condition variables** | Single (wait/notify) | Multiple (`newCondition()`) |
| **Performance** | Similar (modern JVM) | Similar |

### synchronized Examples

```java
// Method-level synchronization
public synchronized void increment() {
    count++;
}

// Block-level synchronization
public void increment() {
    synchronized(this) {
        count++;
    }
}

// Static synchronization (class-level lock)
public static synchronized void staticMethod() {
    // ...
}

// Synchronizing on specific object
private final Object lock = new Object();

public void method() {
    synchronized(lock) {
        // Critical section
    }
}
```

### ReentrantLock Examples

```java
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.Condition;

public class Counter {
    
    private final Lock lock = new ReentrantLock();
    private int count = 0;
    
    // Basic usage
    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();  // Always unlock in finally!
        }
    }
    
    // Try lock with timeout
    public boolean tryIncrement() {
        try {
            if (lock.tryLock(1, TimeUnit.SECONDS)) {
                try {
                    count++;
                    return true;
                } finally {
                    lock.unlock();
                }
            }
            return false;  // Couldn't acquire lock
        } catch (InterruptedException e) {
            return false;
        }
    }
    
    // Interruptible lock
    public void interruptibleIncrement() throws InterruptedException {
        lock.lockInterruptibly();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
}
```

### Condition Variables with ReentrantLock

```java
public class BoundedBuffer<T> {
    
    private final Lock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();
    
    private final Queue<T> queue = new LinkedList<>();
    private final int capacity;
    
    public void put(T item) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                notFull.await();  // Wait until not full
            }
            queue.add(item);
            notEmpty.signal();  // Signal that buffer is not empty
        } finally {
            lock.unlock();
        }
    }
    
    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                notEmpty.await();  // Wait until not empty
            }
            T item = queue.remove();
            notFull.signal();  // Signal that buffer is not full
            return item;
        } finally {
            lock.unlock();
        }
    }
}
```

### When to Use What

| Use synchronized | Use ReentrantLock |
|------------------|-------------------|
| Simple cases | Need try-lock or timeout |
| Don't need advanced features | Need interruptible locking |
| Prefer simpler syntax | Need multiple conditions |
| Block-scoped locking | Need fairness guarantee |

---

## 13. What is a Deadlock and how to prevent it?

### Answer

**Deadlock** occurs when two or more threads are blocked forever, each waiting for the other to release a lock.

### Deadlock Conditions (All 4 required)

| Condition | Description |
|-----------|-------------|
| **Mutual Exclusion** | Resources cannot be shared |
| **Hold and Wait** | Thread holds one resource, waits for another |
| **No Preemption** | Resources cannot be forcibly taken |
| **Circular Wait** | Circular chain of threads waiting |

### Deadlock Example

```java
public class DeadlockExample {
    
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void method1() {
        synchronized (lock1) {
            System.out.println("Thread 1: Holding lock1");
            try { Thread.sleep(100); } catch (Exception e) {}
            
            synchronized (lock2) {  // Waiting for lock2
                System.out.println("Thread 1: Holding lock1 & lock2");
            }
        }
    }
    
    public void method2() {
        synchronized (lock2) {
            System.out.println("Thread 2: Holding lock2");
            try { Thread.sleep(100); } catch (Exception e) {}
            
            synchronized (lock1) {  // Waiting for lock1 - DEADLOCK!
                System.out.println("Thread 2: Holding lock2 & lock1");
            }
        }
    }
}
```

### Prevention Strategies

#### 1. Lock Ordering (Most Common)

```java
// Always acquire locks in the same order
public void method1() {
    synchronized (lock1) {  // First lock1
        synchronized (lock2) {  // Then lock2
            // ...
        }
    }
}

public void method2() {
    synchronized (lock1) {  // First lock1 (same order!)
        synchronized (lock2) {  // Then lock2
            // ...
        }
    }
}
```

#### 2. Lock Timeout

```java
public void safeMethod() {
    boolean gotLock1 = false;
    boolean gotLock2 = false;
    
    try {
        gotLock1 = lock1.tryLock(1, TimeUnit.SECONDS);
        gotLock2 = lock2.tryLock(1, TimeUnit.SECONDS);
        
        if (gotLock1 && gotLock2) {
            // Critical section
        } else {
            // Couldn't get both locks - handle gracefully
        }
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    } finally {
        if (gotLock2) lock2.unlock();
        if (gotLock1) lock1.unlock();
    }
}
```

#### 3. Use Higher-Level Concurrency Utilities

```java
// Instead of manual locking, use:
ConcurrentHashMap<K, V> map = new ConcurrentHashMap<>();
BlockingQueue<T> queue = new LinkedBlockingQueue<>();
AtomicInteger counter = new AtomicInteger();
```

#### 4. Detect and Recover

```java
// Using ThreadMXBean to detect deadlocks
ThreadMXBean bean = ManagementFactory.getThreadMXBean();
long[] deadlockedThreads = bean.findDeadlockedThreads();

if (deadlockedThreads != null) {
    // Handle deadlock - log, recover, restart
}
```

### Prevention Summary

| Strategy | How |
|----------|-----|
| **Lock ordering** | Always acquire locks in same global order |
| **Lock timeout** | Use `tryLock()` with timeout |
| **Minimize lock scope** | Hold locks for shortest time |
| **Avoid nested locks** | Reduce number of locks held simultaneously |
| **Use concurrent utilities** | `ConcurrentHashMap`, `BlockingQueue` |

---

## 14. Explain CompletableFuture

### Answer

`CompletableFuture` enables asynchronous, non-blocking programming with composable operations.

### Basic Usage

```java
// Create and complete manually
CompletableFuture<String> future = new CompletableFuture<>();
future.complete("Result");

// Run async task (no return value)
CompletableFuture<Void> future1 = CompletableFuture.runAsync(() -> {
    System.out.println("Running async");
});

// Supply async result
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    return "Hello";
});

// Get result (blocking)
String result = future2.get();

// Get with timeout
String result = future2.get(5, TimeUnit.SECONDS);

// Non-blocking callback
future2.thenAccept(System.out::println);
```

### Chaining Operations

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> "Hello")           // Start async
    .thenApply(s -> s + " World")         // Transform (sync)
    .thenApply(String::toUpperCase)       // Transform (sync)
    .thenApplyAsync(s -> s + "!")         // Transform (async)
    .whenComplete((result, error) -> {    // Finally block
        if (error != null) {
            System.out.println("Error: " + error);
        } else {
            System.out.println("Result: " + result);
        }
    });

// Result: "HELLO WORLD!"
```

### Combining Futures

```java
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");

// Combine two futures
CompletableFuture<String> combined = future1.thenCombine(future2, 
    (s1, s2) -> s1 + " " + s2);
// "Hello World"

// Wait for both (no result combination)
CompletableFuture<Void> both = CompletableFuture.allOf(future1, future2);

// Wait for either (first completed)
CompletableFuture<Object> either = CompletableFuture.anyOf(future1, future2);

// Compose (flatMap equivalent)
CompletableFuture<String> composed = future1.thenCompose(s -> 
    CompletableFuture.supplyAsync(() -> s + " Composed"));
```

### Error Handling

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> {
        if (Math.random() > 0.5) {
            throw new RuntimeException("Error!");
        }
        return "Success";
    })
    .exceptionally(ex -> "Recovered from: " + ex.getMessage())
    .handle((result, ex) -> {
        if (ex != null) {
            return "Error: " + ex.getMessage();
        }
        return result;
    });
```

### Real-World Example

```java
public CompletableFuture<OrderResult> processOrder(Order order) {
    return CompletableFuture
        .supplyAsync(() -> validateOrder(order))
        .thenCompose(validOrder -> 
            checkInventory(validOrder))
        .thenCompose(inventoryResult -> 
            processPayment(inventoryResult))
        .thenCompose(paymentResult -> 
            shipOrder(paymentResult))
        .thenApply(shipmentResult -> 
            new OrderResult(shipmentResult))
        .exceptionally(ex -> {
            log.error("Order failed", ex);
            return OrderResult.failed(ex.getMessage());
        });
}

// Usage (non-blocking)
processOrder(order)
    .thenAccept(result -> sendNotification(result));
```

### Key Methods Summary

| Method | Purpose |
|--------|---------|
| `supplyAsync()` | Start async task with return value |
| `runAsync()` | Start async task without return value |
| `thenApply()` | Transform result (sync) |
| `thenApplyAsync()` | Transform result (async) |
| `thenAccept()` | Consume result (sync) |
| `thenCompose()` | Chain futures (flatMap) |
| `thenCombine()` | Combine two futures |
| `allOf()` | Wait for all futures |
| `anyOf()` | Wait for any future |
| `exceptionally()` | Handle exceptions |
| `handle()` | Handle result or exception |

---

# Exception Handling & Memory

---

## 15. What is the difference between Checked and Unchecked Exceptions?

### Answer

| Aspect | Checked Exceptions | Unchecked Exceptions |
|--------|-------------------|----------------------|
| **Checked at** | Compile time | Runtime |
| **Handling** | Must catch or declare | Optional |
| **Extends** | `Exception` | `RuntimeException` |
| **Examples** | `IOException`, `SQLException` | `NullPointerException`, `ArrayIndexOutOfBoundsException` |
| **Use case** | Recoverable errors | Programming errors |

### Exception Hierarchy

```
Throwable
├── Error (unchecked - don't catch)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── ...
└── Exception
    ├── RuntimeException (unchecked)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   ├── IllegalArgumentException
    │   └── ...
    └── Checked Exceptions
        ├── IOException
        ├── SQLException
        ├── ClassNotFoundException
        └── ...
```

### Code Examples

```java
// Checked Exception - must handle
public void readFile() throws IOException {  // Declare
    FileReader file = new FileReader("test.txt");
}

public void readFileSafe() {
    try {
        FileReader file = new FileReader("test.txt");
    } catch (IOException e) {  // Or catch
        e.printStackTrace();
    }
}

// Unchecked Exception - optional handling
public void divide(int a, int b) {
    int result = a / b;  // May throw ArithmeticException
}

// Creating custom exceptions
// Checked
public class BusinessException extends Exception {
    public BusinessException(String message) {
        super(message);
    }
}

// Unchecked
public class ValidationException extends RuntimeException {
    public ValidationException(String message) {
        super(message);
    }
}
```

### Try-with-Resources (Java 7+)

```java
// Automatically closes resources
try (FileReader reader = new FileReader("file.txt");
     BufferedReader br = new BufferedReader(reader)) {
    String line = br.readLine();
} catch (IOException e) {
    e.printStackTrace();
}
// Resources auto-closed even if exception occurs
```

### Multi-Catch (Java 7+)

```java
try {
    // code
} catch (IOException | SQLException e) {
    // Handle both exception types
    e.printStackTrace();
}
```

---

## 16. Explain Java Garbage Collection

### Answer

Garbage Collection (GC) automatically reclaims memory from unreachable objects.

### Memory Structure

```
┌─────────────────────────────────────────────────────────────┐
│                        Heap Memory                          │
├─────────────────────────────────┬───────────────────────────┤
│         Young Generation        │      Old Generation       │
├───────────┬─────────────────────┤                           │
│   Eden    │  Survivor (S0, S1)  │    (Tenured Space)        │
│           │                     │                           │
│  New      │  Objects that       │    Long-lived objects     │
│  objects  │  survived GC        │                           │
└───────────┴─────────────────────┴───────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                      Metaspace                               │
│              (Class metadata, method data)                   │
└─────────────────────────────────────────────────────────────┘
```

### GC Process

```
1. Object created in Eden
           │
           ▼
2. Eden full → Minor GC
           │
    ┌──────┴──────┐
    │             │
Unreachable   Reachable
(Collected)   (Move to Survivor)
                  │
                  ▼
3. Object survives multiple Minor GCs
           │
           ▼
4. Promoted to Old Generation
           │
           ▼
5. Old Gen full → Major GC (Full GC)
```

### GC Types

| GC Algorithm | Description | Use Case |
|--------------|-------------|----------|
| **Serial GC** | Single-threaded | Small apps, single CPU |
| **Parallel GC** | Multi-threaded | Throughput-focused |
| **G1 GC** | Low latency (default Java 9+) | General purpose |
| **ZGC** | Ultra-low latency (<10ms) | Large heaps |
| **Shenandoah** | Low latency | Large heaps |

### JVM Options

```bash
# Select GC algorithm
-XX:+UseSerialGC
-XX:+UseParallelGC
-XX:+UseG1GC
-XX:+UseZGC

# Heap size
-Xms512m          # Initial heap
-Xmx2g            # Maximum heap

# GC logging
-Xlog:gc*         # Java 9+
-verbose:gc       # Legacy

# G1 specific
-XX:MaxGCPauseMillis=200
```

### Making Objects Eligible for GC

```java
// 1. Nulling reference
Object obj = new Object();
obj = null;  // Now eligible

// 2. Reassigning reference
Object obj = new Object();
obj = new Object();  // First object eligible

// 3. Objects created inside method
void method() {
    Object obj = new Object();
}  // obj eligible after method returns

// 4. Island of isolation
class Node {
    Node next;
}
Node a = new Node();
Node b = new Node();
a.next = b;
b.next = a;
a = null;
b = null;  // Both eligible (circular reference detected)
```

### Best Practices

| Do | Don't |
|----|-------|
| Let GC manage memory | Call `System.gc()` |
| Use try-with-resources | Create unnecessary objects |
| Pool expensive objects | Hold references longer than needed |
| Profile before tuning | Assume GC is the problem |

---

# Java 8+ Features

---

## 17. What are Functional Interfaces and Lambda Expressions?

### Answer

**Functional Interface:** An interface with exactly one abstract method.

**Lambda Expression:** Anonymous function that implements a functional interface.

### Built-in Functional Interfaces

| Interface | Method | Signature | Use Case |
|-----------|--------|-----------|----------|
| `Predicate<T>` | `test()` | `T → boolean` | Filtering |
| `Function<T,R>` | `apply()` | `T → R` | Transformation |
| `Consumer<T>` | `accept()` | `T → void` | Side effects |
| `Supplier<T>` | `get()` | `() → T` | Factory |
| `BiFunction<T,U,R>` | `apply()` | `(T,U) → R` | Two-arg transform |
| `UnaryOperator<T>` | `apply()` | `T → T` | Same type transform |
| `BinaryOperator<T>` | `apply()` | `(T,T) → T` | Combine two same types |

### Lambda Syntax

```java
// Full syntax
(parameters) -> { statements; return value; }

// Simplified forms
(a, b) -> a + b              // Multiple params, expression
a -> a * 2                    // Single param (no parens needed)
() -> System.out.println("Hi") // No params
(String s) -> s.length()      // Explicit type
```

### Examples

```java
// Predicate - test condition
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<String> isEmpty = String::isEmpty;

isEven.test(4);         // true
isEven.and(n -> n > 0)  // Combine predicates
      .test(4);         // true

// Function - transform
Function<String, Integer> length = String::length;
Function<Integer, Integer> doubleIt = n -> n * 2;

length.apply("Hello");           // 5
length.andThen(doubleIt)         // Chain functions
      .apply("Hello");           // 10

// Consumer - perform action
Consumer<String> printer = System.out::println;
Consumer<String> logger = s -> log.info(s);

printer.accept("Hello");         // Prints "Hello"
printer.andThen(logger)          // Chain consumers
       .accept("Hello");

// Supplier - provide value
Supplier<Double> random = Math::random;
Supplier<List<String>> listFactory = ArrayList::new;

random.get();            // 0.xyz
listFactory.get();       // New empty ArrayList

// BiFunction - two inputs
BiFunction<String, String, String> concat = String::concat;
concat.apply("Hello", " World");  // "Hello World"
```

### Method References

| Type | Syntax | Lambda Equivalent |
|------|--------|-------------------|
| Static | `Class::staticMethod` | `x -> Class.staticMethod(x)` |
| Instance (bound) | `obj::method` | `x -> obj.method(x)` |
| Instance (unbound) | `Class::method` | `x -> x.method()` |
| Constructor | `Class::new` | `x -> new Class(x)` |

```java
// Method reference examples
Function<String, Integer> parse = Integer::parseInt;      // Static
Consumer<String> print = System.out::println;             // Instance (bound)
Function<String, String> upper = String::toUpperCase;     // Instance (unbound)
Supplier<ArrayList<String>> factory = ArrayList::new;     // Constructor
```

### Custom Functional Interface

```java
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);
    
    // Can have default methods
    default int calculateAndDouble(int a, int b) {
        return calculate(a, b) * 2;
    }
    
    // Can have static methods
    static Calculator adder() {
        return (a, b) -> a + b;
    }
}

// Usage
Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;

add.calculate(5, 3);              // 8
multiply.calculateAndDouble(5, 3); // 30
Calculator.adder().calculate(5, 3); // 8
```

---

## 18. What is Optional and why use it?

### Answer

`Optional` is a container that may or may not contain a non-null value. It helps avoid `NullPointerException` and makes APIs more expressive.

### Creating Optional

```java
// With value
Optional<String> opt1 = Optional.of("Hello");        // Throws if null
Optional<String> opt2 = Optional.ofNullable(value);  // Safe for null
Optional<String> opt3 = Optional.empty();            // Empty optional
```

### Checking and Getting Values

```java
Optional<String> opt = Optional.of("Hello");

// ❌ Bad - defeats the purpose
if (opt.isPresent()) {
    String value = opt.get();
}

// ✅ Good - functional approach
opt.ifPresent(System.out::println);

// ✅ Java 9+ - ifPresentOrElse
opt.ifPresentOrElse(
    value -> System.out.println("Found: " + value),
    () -> System.out.println("Not found")
);
```

### Providing Defaults

```java
Optional<String> opt = Optional.empty();

// Default value (always evaluated)
String result1 = opt.orElse("Default");

// Default with supplier (lazy - only evaluated if empty)
String result2 = opt.orElseGet(() -> computeDefault());

// Throw if empty
String result3 = opt.orElseThrow();  // NoSuchElementException
String result4 = opt.orElseThrow(() -> 
    new RuntimeException("Not found"));
```

### Transforming Optional

```java
Optional<String> opt = Optional.of("hello");

// map - transform value
Optional<Integer> length = opt.map(String::length);  // Optional[5]
Optional<String> upper = opt.map(String::toUpperCase);  // Optional[HELLO]

// filter - keep if condition matches
Optional<String> filtered = opt.filter(s -> s.length() > 3);  // Optional[hello]
Optional<String> filtered2 = opt.filter(s -> s.length() > 10);  // Optional.empty

// flatMap - when transformation returns Optional
Optional<String> result = opt.flatMap(s -> findUser(s));
```

### Chaining to Avoid Null Checks

```java
// ❌ Traditional null checking
String managerName = null;
if (employee != null) {
    Department dept = employee.getDepartment();
    if (dept != null) {
        Manager manager = dept.getManager();
        if (manager != null) {
            managerName = manager.getName();
        }
    }
}
if (managerName == null) {
    managerName = "Unknown";
}

// ✅ With Optional
String managerName = Optional.ofNullable(employee)
    .map(Employee::getDepartment)
    .map(Department::getManager)
    .map(Manager::getName)
    .orElse("Unknown");
```

### Best Practices

| Do | Don't |
|----|-------|
| Return `Optional` from methods that may not have a result | Use `Optional` for class fields |
| Use `orElse`, `orElseGet`, `orElseThrow` | Use `Optional` as method parameters |
| Chain `map`, `filter`, `flatMap` | Use `isPresent()` + `get()` |
| Return empty collection instead of `Optional<List>` | Wrap collections in Optional |

```java
// ✅ Good API design
public Optional<User> findUserById(Long id) {
    // ...
}

// ❌ Bad - don't use Optional as parameter
public void processUser(Optional<User> user) {
    // ...
}

// ✅ Good - use overloading or null
public void processUser(User user) {
    // ...
}
```

---

# Design Patterns

---

## 19. Explain Singleton Pattern and its thread-safe implementation

### Answer

Singleton ensures only one instance of a class exists.

### Implementation Methods

#### 1. Eager Initialization

```java
public class Singleton {
    // Created at class loading
    private static final Singleton INSTANCE = new Singleton();
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```
**Pros:** Simple, thread-safe
**Cons:** Instance created even if never used

#### 2. Lazy Initialization (Not Thread-Safe)

```java
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();  // Race condition!
        }
        return instance;
    }
}
```
**Cons:** Not thread-safe

#### 3. Synchronized Method (Thread-Safe but Slow)

```java
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {}
    
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
**Cons:** Synchronization overhead on every call

#### 4. Double-Checked Locking (Recommended)

```java
public class Singleton {
    private static volatile Singleton instance;  // volatile is crucial!
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {                  // First check (no locking)
            synchronized (Singleton.class) {
                if (instance == null) {          // Second check (with lock)
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```
**Pros:** Thread-safe, lazy, minimal synchronization

#### 5. Bill Pugh Singleton (Using Inner Class)

```java
public class Singleton {
    
    private Singleton() {}
    
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
}
```
**Pros:** Lazy, thread-safe, no synchronization needed

#### 6. Enum Singleton (Best Practice)

```java
public enum Singleton {
    INSTANCE;
    
    private int value;
    
    public int getValue() {
        return value;
    }
    
    public void setValue(int value) {
        this.value = value;
    }
    
    public void doSomething() {
        System.out.println("Doing something");
    }
}

// Usage
Singleton.INSTANCE.doSomething();
Singleton.INSTANCE.setValue(42);
```
**Pros:** 
- Thread-safe
- Serialization-safe
- Reflection-safe
- Simple
- Recommended by Joshua Bloch (Effective Java)

### Comparison

| Method | Thread-Safe | Lazy | Serialization-Safe | Reflection-Safe |
|--------|-------------|------|-------------------|-----------------|
| Eager | ✅ | ❌ | ❌ | ❌ |
| Synchronized | ✅ | ✅ | ❌ | ❌ |
| Double-Checked | ✅ | ✅ | ❌ | ❌ |
| Bill Pugh | ✅ | ✅ | ❌ | ❌ |
| Enum | ✅ | ✅ | ✅ | ✅ |

---

## 20. What are fail-fast and fail-safe iterators?

### Answer

| Aspect | Fail-Fast | Fail-Safe |
|--------|-----------|-----------|
| **Behavior** | Throws `ConcurrentModificationException` | No exception |
| **Works on** | Original collection | Copy of collection |
| **Collections** | `ArrayList`, `HashMap`, `HashSet` | `CopyOnWriteArrayList`, `ConcurrentHashMap` |
| **Memory** | Less (no copy) | More (creates copy) |
| **Performance** | Faster | Slower |
| **Reflects changes** | N/A (fails) | May not reflect latest |

### Fail-Fast Example

```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
list.add("C");

// Fail-fast - throws ConcurrentModificationException
for (String s : list) {
    if (s.equals("B")) {
        list.remove(s);  // Structural modification during iteration
    }
}

// Same with explicit iterator
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String s = iterator.next();
    list.add("D");  // ConcurrentModificationException!
}
```

### How Fail-Fast Works

```java
// ArrayList internally maintains modCount
transient int modCount;  // Incremented on structural changes

// Iterator checks modCount
public E next() {
    checkForComodification();  // Throws if modified
    // ...
}

final void checkForComodification() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
}
```

### Safe Removal During Iteration

```java
// ✅ Use Iterator.remove()
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    String s = iterator.next();
    if (s.equals("B")) {
        iterator.remove();  // Safe!
    }
}

// ✅ Use removeIf() (Java 8+)
list.removeIf(s -> s.equals("B"));

// ✅ Use CopyOnWriteArrayList
List<String> list = new CopyOnWriteArrayList<>();
for (String s : list) {
    list.remove(s);  // Safe - iterates over copy
}
```

### Fail-Safe Example

```java
// CopyOnWriteArrayList - fail-safe
List<String> list = new CopyOnWriteArrayList<>();
list.add("A");
list.add("B");
list.add("C");

for (String s : list) {
    System.out.println(s);
    list.add("D");  // No exception - iterates over copy
}
// Output: A, B, C (doesn't see D)

// ConcurrentHashMap - fail-safe
Map<String, Integer> map = new ConcurrentHashMap<>();
map.put("A", 1);
map.put("B", 2);

for (String key : map.keySet()) {
    map.put("C", 3);  // No exception
}
```

### When to Use What

| Use Fail-Fast (ArrayList, HashMap) | Use Fail-Safe (CopyOnWriteArrayList, ConcurrentHashMap) |
|------------------------------------|---------------------------------------------------------|
| Single-threaded environment | Multi-threaded environment |
| Read-heavy, rare modifications | Frequent concurrent modifications |
| Memory is a concern | Consistency during iteration needed |
| Performance is critical | Thread safety is critical |

---

# Quick Reference Summary

## Key Topics for 2025 Interviews

| Topic | Key Points |
|-------|------------|
| **Java 21** | Virtual Threads, Record Patterns, Sequenced Collections, Pattern Matching |
| **Virtual Threads** | Lightweight, JVM-managed, millions concurrent, I/O-bound tasks |
| **HashMap** | Array + LinkedList/Tree, O(1) average, TREEIFY_THRESHOLD=8 |
| **Immutability** | Thread-safe, cacheable, String pool, security |
| **Streams** | Lazy evaluation, functional operations, single-use |
| **CompletableFuture** | Async, non-blocking, chainable, composable |
| **GC** | G1 default, generations (Young/Old), automatic memory management |
| **Optional** | Null-safe container, use orElse/orElseGet, avoid isPresent+get |
| **Records** | Immutable data carriers, auto-generated methods |
| **Singleton** | Enum best, double-checked locking, Bill Pugh pattern |
| **Fail-fast vs Fail-safe** | ConcurrentModificationException vs copy-on-write |

## Top Mistakes to Avoid

1. Using `==` instead of `.equals()` for String comparison
2. Not overriding `hashCode()` when overriding `equals()`
3. Using `Optional.isPresent()` + `get()` pattern
4. Calling `System.gc()` manually
5. Modifying collection during iteration (without iterator.remove())
6. Not using try-with-resources for closeable resources
7. Ignoring thread safety in Singleton implementations
8. Using synchronized unnecessarily (prefer concurrent utilities)
9. Creating too many platform threads (use virtual threads)
10. Blocking in CompletableFuture callbacks

---

*Good luck with your interview!*

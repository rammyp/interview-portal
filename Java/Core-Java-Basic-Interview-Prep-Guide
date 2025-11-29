# Core Java Interview Questions - Complete Guide

## Table of Contents
1. [Java Basics](#java-basics)
2. [OOP Concepts](#oop-concepts)
3. [Data Types & Variables](#data-types-variables)
4. [String Handling](#string-handling)
5. [Collections Framework](#collections-framework)
6. [Exception Handling](#exception-handling)
7. [Multithreading & Concurrency](#multithreading-concurrency)
8. [Java 8+ Features](#java-8-features)
9. [Memory Management & JVM](#memory-management-jvm)
10. [Design Patterns](#design-patterns)
11. [JDBC & Database](#jdbc-database)
12. [Coding Problems](#coding-problems)

---

## 1. Java Basics

### Q1: What is Java? Why is it platform independent?
**Answer:**
Java is a high-level, object-oriented programming language. It's platform independent because:
- Java code compiles to bytecode (not native machine code)
- Bytecode runs on JVM (Java Virtual Machine)
- JVM acts as an abstraction layer between code and OS
- "Write Once, Run Anywhere" (WORA)

### Q2: Explain JDK vs JRE vs JVM
**Answer:**
```
JDK (Java Development Kit)
├── Development Tools (javac, java, jar, javadoc)
└── JRE (Java Runtime Environment)
    ├── Class Libraries (rt.jar, etc.)
    └── JVM (Java Virtual Machine)
        ├── Class Loader
        ├── Memory Area
        └── Execution Engine
```

- **JVM**: Executes bytecode, platform-specific
- **JRE**: JVM + Libraries needed to run Java programs
- **JDK**: JRE + Development tools to compile and debug

### Q3: What are the main features of Java?
**Answer:**
1. **Object-Oriented**: Everything is an object
2. **Platform Independent**: Bytecode runs on any JVM
3. **Simple**: No pointers, automatic garbage collection
4. **Secure**: Security manager, bytecode verification
5. **Robust**: Strong memory management, exception handling
6. **Multithreaded**: Built-in thread support
7. **High Performance**: JIT compiler
8. **Distributed**: RMI, networking support

### Q4: Explain public static void main(String[] args)
**Answer:**
```java
public static void main(String[] args) {
    // Entry point of Java program
}
```
- **public**: Accessible from anywhere (JVM needs to call it)
- **static**: No need to create object (JVM calls directly)
- **void**: Returns nothing
- **main**: Method name (JVM looks for this specific name)
- **String[] args**: Command line arguments

---

## 2. OOP Concepts

### Q5: What are the four pillars of OOP?

**1. Encapsulation:**
```java
public class BankAccount {
    private double balance;  // Hidden data
    
    public void deposit(double amount) {  // Controlled access
        if (amount > 0) {
            balance += amount;
        }
    }
    
    public double getBalance() {
        return balance;
    }
}
```

**2. Inheritance:**
```java
class Animal {
    void eat() { System.out.println("Eating..."); }
}

class Dog extends Animal {
    void bark() { System.out.println("Barking..."); }
}
```

**3. Polymorphism:**
```java
// Method Overloading (Compile-time)
class Calculator {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
}

// Method Overriding (Runtime)
class Shape {
    void draw() { System.out.println("Drawing shape"); }
}
class Circle extends Shape {
    @Override
    void draw() { System.out.println("Drawing circle"); }
}
```

**4. Abstraction:**
```java
abstract class Vehicle {
    abstract void start();  // Abstract method
    void stop() { System.out.println("Stopping..."); }  // Concrete method
}

interface Flyable {
    void fly();  // All methods are abstract by default
}
```

### Q6: Abstract Class vs Interface

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Multiple inheritance | No | Yes |
| Variables | Instance variables allowed | Only public static final |
| Methods | Abstract & concrete | Abstract, default, static (Java 8+) |
| Constructors | Yes | No |
| Access modifiers | All allowed | Public only (Java 9+ allows private) |

```java
// Abstract Class
abstract class Animal {
    String name;
    Animal(String name) { this.name = name; }
    abstract void makeSound();
    void sleep() { System.out.println("Sleeping..."); }
}

// Interface
interface Swimmer {
    int MAX_DEPTH = 100;  // public static final by default
    void swim();  // public abstract by default
    
    // Java 8+
    default void floatOnWater() {
        System.out.println("Floating...");
    }
    
    static void checkWaterTemperature() {
        System.out.println("Checking temperature...");
    }
}
```

### Q7: What is method overloading vs overriding?

**Method Overloading (Compile-time Polymorphism):**
```java
class Example {
    // Different parameters
    void print(int x) { }
    void print(String x) { }
    void print(int x, int y) { }
    
    // Return type alone doesn't work!
    // int print(int x) { }  // Compilation error
}
```

**Method Overriding (Runtime Polymorphism):**
```java
class Parent {
    void display() { System.out.println("Parent"); }
}

class Child extends Parent {
    @Override
    void display() { System.out.println("Child"); }
}

// Usage
Parent p = new Child();
p.display();  // Output: "Child" (runtime polymorphism)
```

---

## 3. Data Types & Variables

### Q8: What are primitive data types in Java?

```java
// Primitive Types (8 types)
byte b = 127;           // 8 bits, -128 to 127
short s = 32767;        // 16 bits, -32,768 to 32,767
int i = 2147483647;     // 32 bits, -2^31 to 2^31-1
long l = 9223372036854775807L;  // 64 bits, -2^63 to 2^63-1

float f = 3.14f;        // 32 bits, ~7 decimal digits
double d = 3.14159;     // 64 bits, ~15 decimal digits

char c = 'A';           // 16 bits, Unicode character
boolean flag = true;    // true or false

// Wrapper Classes (Object versions)
Integer intObj = 100;   // Autoboxing
int primitive = intObj; // Unboxing
```

### Q9: What is the difference between == and equals()?

```java
// == compares references (memory address)
// equals() compares content (when overridden)

String s1 = new String("Hello");
String s2 = new String("Hello");
String s3 = s1;

System.out.println(s1 == s2);      // false (different objects)
System.out.println(s1.equals(s2)); // true (same content)
System.out.println(s1 == s3);      // true (same reference)

// For primitives, == compares values
int a = 5, b = 5;
System.out.println(a == b);  // true
```

### Q10: What is pass by value vs pass by reference?

**Java is ALWAYS pass by value!**
```java
// Primitive - pass by value
void modify(int x) {
    x = 100;  // Doesn't affect original
}

// Object reference - pass by value of reference
void modify(List<String> list) {
    list.add("New");  // Affects original object
    list = new ArrayList<>();  // Doesn't affect original reference
}

public static void main(String[] args) {
    int num = 50;
    modify(num);
    System.out.println(num);  // Still 50
    
    List<String> list = new ArrayList<>();
    modify(list);
    System.out.println(list.size());  // 1 (added element)
}
```

---

## 4. String Handling

### Q11: String vs StringBuilder vs StringBuffer

```java
// String - Immutable
String str = "Hello";
str = str + " World";  // Creates new object

// StringBuilder - Mutable, Not thread-safe, Fast
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");  // Modifies existing object

// StringBuffer - Mutable, Thread-safe, Slower
StringBuffer sbf = new StringBuffer("Hello");
sbf.append(" World");  // Thread-safe modification
```

**Performance comparison:**
```java
// String concatenation in loop - SLOW
String s = "";
for(int i = 0; i < 1000; i++) {
    s += i;  // Creates 1000 objects!
}

// StringBuilder - FAST
StringBuilder sb = new StringBuilder();
for(int i = 0; i < 1000; i++) {
    sb.append(i);  // Modifies same object
}
```

### Q12: String Pool Concept

```java
// String literals go to String Pool
String s1 = "Hello";  // Pool
String s2 = "Hello";  // Same object from pool
String s3 = new String("Hello");  // Heap (not pool)
String s4 = s3.intern();  // Forced to pool

System.out.println(s1 == s2);  // true (same pool object)
System.out.println(s1 == s3);  // false (different objects)
System.out.println(s1 == s4);  // true (both in pool)
```

### Q13: Common String Methods

```java
String str = "  Hello World  ";

// Trimming and case
str.trim();              // "Hello World"
str.toUpperCase();       // "  HELLO WORLD  "
str.toLowerCase();       // "  hello world  "

// Searching
str.indexOf("World");    // 8
str.contains("Hello");   // true
str.startsWith("  He");  // true
str.endsWith("ld  ");    // true

// Extraction
str.substring(2, 7);     // "Hello"
str.charAt(2);           // 'H'

// Replacement
str.replace("World", "Java");  // "  Hello Java  "
str.replaceAll("\\s+", "");    // "HelloWorld"

// Splitting
"a,b,c".split(",");      // ["a", "b", "c"]

// Comparison
"abc".equals("ABC");     // false
"abc".equalsIgnoreCase("ABC");  // true
"abc".compareTo("abd");  // -1 (negative means less than)
```

---

## 5. Collections Framework

### Q14: Collections Hierarchy

```
Collection (Interface)
├── List (Interface)
│   ├── ArrayList (Class)
│   ├── LinkedList (Class)
│   └── Vector (Class)
│       └── Stack (Class)
├── Set (Interface)
│   ├── HashSet (Class)
│   ├── LinkedHashSet (Class)
│   └── TreeSet (Class)
└── Queue (Interface)
    ├── PriorityQueue (Class)
    └── Deque (Interface)
        └── ArrayDeque (Class)

Map (Interface) - Separate hierarchy
├── HashMap (Class)
├── LinkedHashMap (Class)
├── TreeMap (Class)
└── HashTable (Class)
```

### Q15: ArrayList vs LinkedList

```java
// ArrayList - Dynamic array
// Pros: Fast random access O(1), cache-friendly
// Cons: Slow insertion/deletion in middle O(n)
ArrayList<String> arrayList = new ArrayList<>();
arrayList.add("A");           // O(1) amortized
arrayList.get(0);             // O(1)
arrayList.remove(0);          // O(n)

// LinkedList - Doubly linked list
// Pros: Fast insertion/deletion O(1)
// Cons: Slow random access O(n), more memory
LinkedList<String> linkedList = new LinkedList<>();
linkedList.add("A");          // O(1)
linkedList.get(0);            // O(n)
linkedList.removeFirst();     // O(1)
```

### Q16: HashMap Internal Working

```java
// HashMap uses array of buckets + linked lists/trees
// Hash collision resolution: Chaining (Java 8+ uses trees after threshold)

HashMap<String, Integer> map = new HashMap<>();
map.put("John", 25);  // Calculates hash, finds bucket, stores

// Internal process:
// 1. hash = key.hashCode()
// 2. index = hash & (n-1)  // n is bucket array size
// 3. Store at bucket[index]
// 4. If collision, add to linked list/tree

// Load factor = 0.75 (default)
// When 75% full, resize (double size) and rehash
```

### Q17: HashSet vs TreeSet vs LinkedHashSet

```java
// HashSet - No order, O(1) operations, allows one null
HashSet<Integer> hashSet = new HashSet<>();
hashSet.add(3); hashSet.add(1); hashSet.add(2);
// Order: unpredictable [2, 3, 1]

// TreeSet - Sorted order, O(log n) operations, no null
TreeSet<Integer> treeSet = new TreeSet<>();
treeSet.add(3); treeSet.add(1); treeSet.add(2);
// Order: sorted [1, 2, 3]

// LinkedHashSet - Insertion order, O(1) operations
LinkedHashSet<Integer> linkedSet = new LinkedHashSet<>();
linkedSet.add(3); linkedSet.add(1); linkedSet.add(2);
// Order: insertion [3, 1, 2]
```

### Q18: Fail-Fast vs Fail-Safe Iterators

```java
// Fail-Fast (ArrayList, HashMap) - Throws ConcurrentModificationException
ArrayList<String> list = new ArrayList<>();
list.add("A"); list.add("B");
Iterator<String> it = list.iterator();
while(it.hasNext()) {
    String item = it.next();
    list.add("C");  // ConcurrentModificationException!
}

// Fail-Safe (CopyOnWriteArrayList, ConcurrentHashMap) - Works on copy
CopyOnWriteArrayList<String> safeList = new CopyOnWriteArrayList<>();
safeList.add("A"); safeList.add("B");
Iterator<String> safeIt = safeList.iterator();
while(safeIt.hasNext()) {
    String item = safeIt.next();
    safeList.add("C");  // No exception, but won't see "C" in this iteration
}
```

### Q19: Comparable vs Comparator

```java
// Comparable - Natural ordering (single way)
class Person implements Comparable<Person> {
    String name;
    int age;
    
    @Override
    public int compareTo(Person other) {
        return this.age - other.age;  // Sort by age
    }
}

// Comparator - Custom ordering (multiple ways)
class NameComparator implements Comparator<Person> {
    @Override
    public int compare(Person p1, Person p2) {
        return p1.name.compareTo(p2.name);
    }
}

// Lambda comparators (Java 8+)
List<Person> people = new ArrayList<>();
people.sort((p1, p2) -> p1.age - p2.age);  // By age
people.sort(Comparator.comparing(Person::getName));  // By name
```

---

## 6. Exception Handling

### Q20: Exception Hierarchy

```
Throwable
├── Error (Don't catch)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── VirtualMachineError
└── Exception (Catch these)
    ├── RuntimeException (Unchecked)
    │   ├── NullPointerException
    │   ├── IndexOutOfBoundsException
    │   └── IllegalArgumentException
    └── Checked Exceptions
        ├── IOException
        ├── SQLException
        └── ClassNotFoundException
```

### Q21: Checked vs Unchecked Exceptions

```java
// Checked Exception - Must handle or declare
public void readFile(String filename) throws IOException {  // Must declare
    FileReader file = new FileReader(filename);  // Throws IOException
}

// Or handle
public void readFile(String filename) {
    try {
        FileReader file = new FileReader(filename);
    } catch (IOException e) {
        e.printStackTrace();
    }
}

// Unchecked Exception - Optional to handle
public void divide(int a, int b) {
    int result = a / b;  // May throw ArithmeticException (unchecked)
}
```

### Q22: try-catch-finally vs try-with-resources

```java
// Traditional try-catch-finally
FileReader reader = null;
try {
    reader = new FileReader("file.txt");
    // Read file
} catch (IOException e) {
    e.printStackTrace();
} finally {
    if (reader != null) {
        try {
            reader.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

// try-with-resources (Java 7+) - Auto-closes
try (FileReader reader = new FileReader("file.txt")) {
    // Read file
} catch (IOException e) {
    e.printStackTrace();
}  // Automatically closes reader
```

### Q23: Custom Exceptions

```java
// Custom checked exception
class InsufficientFundsException extends Exception {
    private double amount;
    
    public InsufficientFundsException(double amount) {
        super("Insufficient funds: " + amount);
        this.amount = amount;
    }
}

// Custom unchecked exception
class InvalidAgeException extends RuntimeException {
    public InvalidAgeException(String message) {
        super(message);
    }
}

// Usage
class BankAccount {
    private double balance;
    
    public void withdraw(double amount) throws InsufficientFundsException {
        if (amount > balance) {
            throw new InsufficientFundsException(amount - balance);
        }
        balance -= amount;
    }
}
```

---

## 7. Multithreading & Concurrency

### Q24: Ways to Create Threads

```java
// Method 1: Extending Thread class
class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread running: " + Thread.currentThread().getName());
    }
}

// Method 2: Implementing Runnable
class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable running");
    }
}

// Method 3: Lambda (Java 8+)
Runnable task = () -> System.out.println("Lambda thread");

// Method 4: Callable (returns result)
Callable<Integer> callable = () -> {
    return 42;
};

// Usage
MyThread thread1 = new MyThread();
thread1.start();

Thread thread2 = new Thread(new MyRunnable());
thread2.start();

Thread thread3 = new Thread(task);
thread3.start();

ExecutorService executor = Executors.newFixedThreadPool(2);
Future<Integer> future = executor.submit(callable);
Integer result = future.get();  // Blocks until complete
```

### Q25: Thread Lifecycle

```
NEW → RUNNABLE → BLOCKED/WAITING/TIMED_WAITING → TERMINATED

NEW: Thread created but not started
RUNNABLE: Running or ready to run
BLOCKED: Waiting for monitor lock
WAITING: Waiting indefinitely
TIMED_WAITING: Waiting for specified time
TERMINATED: Completed execution
```

### Q26: Synchronization

```java
// Method synchronization
class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;  // Thread-safe
    }
    
    public synchronized int getCount() {
        return count;
    }
}

// Block synchronization
class BankAccount {
    private double balance;
    private final Object lock = new Object();
    
    public void deposit(double amount) {
        synchronized(lock) {
            balance += amount;
        }
    }
}

// Static synchronization
class Utility {
    public static synchronized void printNumbers() {
        // Locks on class object
    }
}
```

### Q27: wait() vs sleep()

```java
// wait() - Releases lock, waits for notify
synchronized(lock) {
    while(condition) {
        lock.wait();  // Releases lock
    }
}

// sleep() - Doesn't release lock
synchronized(lock) {
    Thread.sleep(1000);  // Keeps lock
}

// notify() and notifyAll()
synchronized(lock) {
    // Change condition
    lock.notify();     // Wakes one waiting thread
    lock.notifyAll();  // Wakes all waiting threads
}
```

### Q28: Deadlock Example

```java
class DeadlockExample {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void method1() {
        synchronized(lock1) {
            System.out.println("Holding lock1");
            synchronized(lock2) {  // Needs lock2
                System.out.println("Holding both locks");
            }
        }
    }
    
    public void method2() {
        synchronized(lock2) {
            System.out.println("Holding lock2");
            synchronized(lock1) {  // Needs lock1 - DEADLOCK!
                System.out.println("Holding both locks");
            }
        }
    }
}

// Prevention: Always acquire locks in same order
```

### Q29: Thread Pool and Executors

```java
// Fixed thread pool
ExecutorService fixedPool = Executors.newFixedThreadPool(5);

// Cached thread pool (creates as needed)
ExecutorService cachedPool = Executors.newCachedThreadPool();

// Single thread executor
ExecutorService singleExecutor = Executors.newSingleThreadExecutor();

// Scheduled executor
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
scheduler.scheduleAtFixedRate(() -> {
    System.out.println("Periodic task");
}, 0, 1, TimeUnit.SECONDS);

// Submit tasks
fixedPool.submit(() -> {
    System.out.println("Task executed");
});

// Shutdown
fixedPool.shutdown();  // No new tasks
fixedPool.awaitTermination(60, TimeUnit.SECONDS);
```

---

## 8. Java 8+ Features

### Q30: Lambda Expressions

```java
// Before Java 8
Comparator<String> comp = new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
};

// Lambda expression
Comparator<String> lambdaComp = (s1, s2) -> s1.compareTo(s2);

// Method reference
Comparator<String> methodRef = String::compareTo;

// Common lambda examples
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);
numbers.forEach(n -> System.out.println(n));
numbers.forEach(System.out::println);  // Method reference
```

### Q31: Functional Interfaces

```java
// Built-in functional interfaces
// Predicate<T> - test(T t) returns boolean
Predicate<Integer> isEven = n -> n % 2 == 0;

// Function<T,R> - apply(T t) returns R
Function<String, Integer> strLength = String::length;

// Consumer<T> - accept(T t) returns void
Consumer<String> printer = System.out::println;

// Supplier<T> - get() returns T
Supplier<Double> randomSupplier = Math::random;

// Custom functional interface
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
    // Can have default and static methods
    default void print() { System.out.println("Calculating..."); }
}

Calculator add = (a, b) -> a + b;
```

### Q32: Stream API

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

// Filter, map, collect
List<Integer> evenSquares = numbers.stream()
    .filter(n -> n % 2 == 0)           // 2, 4, 6
    .map(n -> n * n)                    // 4, 16, 36
    .collect(Collectors.toList());

// Reduce
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);        // 21

// Method references
int sum2 = numbers.stream()
    .reduce(0, Integer::sum);

// Parallel streams
long count = numbers.parallelStream()
    .filter(n -> n > 3)
    .count();

// Complex operations
Map<Boolean, List<Integer>> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
// {false=[1, 3, 5], true=[2, 4, 6]}

// FlatMap
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2),
    Arrays.asList(3, 4)
);
List<Integer> flat = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());      // [1, 2, 3, 4]
```

### Q33: Optional Class

```java
// Creating Optional
Optional<String> empty = Optional.empty();
Optional<String> present = Optional.of("Hello");
Optional<String> nullable = Optional.ofNullable(null);

// Checking presence
if (present.isPresent()) {
    System.out.println(present.get());
}

// Better way - functional style
present.ifPresent(System.out::println);

// Default values
String value = nullable.orElse("Default");
String value2 = nullable.orElseGet(() -> "Computed default");

// Throwing exception
String value3 = nullable.orElseThrow(() -> new RuntimeException("No value"));

// Transforming
Optional<Integer> length = present.map(String::length);
Optional<String> upperCase = present.filter(s -> s.length() > 3)
                                    .map(String::toUpperCase);
```

### Q34: Date and Time API (Java 8)

```java
// LocalDate - Date without time
LocalDate date = LocalDate.now();
LocalDate specific = LocalDate.of(2024, 3, 15);
LocalDate tomorrow = date.plusDays(1);

// LocalTime - Time without date
LocalTime time = LocalTime.now();
LocalTime specific2 = LocalTime.of(14, 30, 45);

// LocalDateTime - Both date and time
LocalDateTime dateTime = LocalDateTime.now();
LocalDateTime future = dateTime.plusHours(3).plusMinutes(30);

// ZonedDateTime - With timezone
ZonedDateTime zoned = ZonedDateTime.now(ZoneId.of("America/New_York"));

// Period and Duration
Period period = Period.between(date, tomorrow);  // 1 day
Duration duration = Duration.between(time, time.plusHours(2));  // 2 hours

// Formatting
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm");
String formatted = dateTime.format(formatter);
```

---

## 9. Memory Management & JVM

### Q35: JVM Architecture

```
Class Loader Subsystem
├── Loading (Bootstrap, Extension, Application)
├── Linking (Verify, Prepare, Resolve)
└── Initialization

Runtime Data Area
├── Method Area (Shared) - Class info, constants
├── Heap (Shared) - Objects, arrays
├── Stack (Per thread) - Method calls, local variables
├── PC Register (Per thread) - Current instruction
└── Native Method Stack (Per thread)

Execution Engine
├── Interpreter
├── JIT Compiler
└── Garbage Collector
```

### Q36: Stack vs Heap Memory

```java
public class MemoryExample {
    // Heap: Object created here
    private String name = "John";  
    
    public void method() {
        // Stack: Primitive stored here
        int age = 25;  
        
        // Stack: Reference 'str' stored here
        // Heap: String object stored here
        String str = new String("Hello");  
        
        // Stack: Reference 'list' stored here
        // Heap: ArrayList object stored here
        List<String> list = new ArrayList<>();
    }  // Stack frame removed after method ends
}
```

### Q37: Garbage Collection

```java
// Types of GC
// 1. Serial GC - Single thread
// 2. Parallel GC - Multiple threads
// 3. G1GC - Low latency
// 4. ZGC - Ultra-low latency (Java 11+)

// Object becomes eligible for GC when:
public void gcExample() {
    String str = new String("Hello");  // Object created
    str = null;  // Eligible for GC
    
    String str2 = new String("World");
    str2 = new String("Java");  // "World" eligible for GC
}  // All local objects eligible after method ends

// Requesting GC (not guaranteed)
System.gc();
Runtime.getRuntime().gc();

// Finalization (deprecated)
@Override
protected void finalize() throws Throwable {
    // Cleanup code - called before GC (unreliable)
}
```

### Q38: Memory Leaks in Java

```java
// Common causes of memory leaks:

// 1. Static collections
public class Cache {
    private static Map<String, Object> cache = new HashMap<>();
    // Objects in cache never GC'd unless explicitly removed
}

// 2. Unclosed resources
public void leakyMethod() {
    Connection conn = getConnection();
    // Missing: conn.close();
}

// 3. Inner class references
public class Outer {
    private String data = "Important";
    
    class Inner {
        // Inner class holds reference to Outer
        void method() {
            System.out.println(data);
        }
    }
}

// 4. Thread locals
ThreadLocal<Object> threadLocal = new ThreadLocal<>();
threadLocal.set(new Object());
// Missing: threadLocal.remove();
```

---

## 10. Design Patterns

### Q39: Singleton Pattern

```java
// Thread-safe Singleton with lazy initialization
public class Singleton {
    private static volatile Singleton instance;
    
    private Singleton() {
        // Prevent instantiation
    }
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {  // Double-checked locking
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

// Enum Singleton (Best approach)
public enum EnumSingleton {
    INSTANCE;
    
    public void doSomething() {
        // Method implementation
    }
}
```

### Q40: Factory Pattern

```java
// Factory Pattern
interface Vehicle {
    void drive();
}

class Car implements Vehicle {
    public void drive() { System.out.println("Driving car"); }
}

class Bike implements Vehicle {
    public void drive() { System.out.println("Riding bike"); }
}

class VehicleFactory {
    public static Vehicle createVehicle(String type) {
        switch(type) {
            case "car": return new Car();
            case "bike": return new Bike();
            default: throw new IllegalArgumentException("Unknown type");
        }
    }
}

// Usage
Vehicle vehicle = VehicleFactory.createVehicle("car");
vehicle.drive();
```

### Q41: Builder Pattern

```java
// Builder Pattern for complex object creation
public class User {
    private final String firstName;  // Required
    private final String lastName;   // Required
    private final int age;          // Optional
    private final String phone;     // Optional
    private final String address;   // Optional
    
    private User(UserBuilder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
        this.phone = builder.phone;
        this.address = builder.address;
    }
    
    public static class UserBuilder {
        private final String firstName;
        private final String lastName;
        private int age;
        private String phone;
        private String address;
        
        public UserBuilder(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }
        
        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }
        
        public UserBuilder phone(String phone) {
            this.phone = phone;
            return this;
        }
        
        public UserBuilder address(String address) {
            this.address = address;
            return this;
        }
        
        public User build() {
            return new User(this);
        }
    }
}

// Usage
User user = new User.UserBuilder("John", "Doe")
    .age(30)
    .phone("123-456-7890")
    .build();
```

### Q42: Observer Pattern

```java
// Observer Pattern
interface Observer {
    void update(String message);
}

class Subject {
    private List<Observer> observers = new ArrayList<>();
    
    public void attach(Observer observer) {
        observers.add(observer);
    }
    
    public void detach(Observer observer) {
        observers.remove(observer);
    }
    
    public void notifyObservers(String message) {
        for (Observer observer : observers) {
            observer.update(message);
        }
    }
}

class ConcreteObserver implements Observer {
    private String name;
    
    public ConcreteObserver(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String message) {
        System.out.println(name + " received: " + message);
    }
}
```

---

## 11. JDBC & Database

### Q43: JDBC Connection Steps

```java
// 1. Load driver (not needed for JDBC 4.0+)
Class.forName("com.mysql.cj.jdbc.Driver");

// 2. Create connection
String url = "jdbc:mysql://localhost:3306/mydb";
String username = "root";
String password = "password";
Connection conn = DriverManager.getConnection(url, username, password);

// 3. Create statement
Statement stmt = conn.createStatement();
// Or PreparedStatement for parameters
PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM users WHERE id = ?"
);

// 4. Execute query
ResultSet rs = stmt.executeQuery("SELECT * FROM users");
// Or update
int rowsAffected = stmt.executeUpdate("UPDATE users SET name = 'John'");

// 5. Process results
while (rs.next()) {
    int id = rs.getInt("id");
    String name = rs.getString("name");
}

// 6. Close resources (use try-with-resources)
rs.close();
stmt.close();
conn.close();
```

### Q44: Statement vs PreparedStatement

```java
// Statement - SQL injection vulnerable
String name = "'; DROP TABLE users; --";
Statement stmt = conn.createStatement();
stmt.executeQuery("SELECT * FROM users WHERE name = '" + name + "'");
// Dangerous!

// PreparedStatement - Safe from SQL injection
PreparedStatement pstmt = conn.prepareStatement(
    "SELECT * FROM users WHERE name = ? AND age = ?"
);
pstmt.setString(1, name);  // Safely escaped
pstmt.setInt(2, 25);
ResultSet rs = pstmt.executeQuery();

// PreparedStatement advantages:
// 1. Prevents SQL injection
// 2. Better performance (pre-compiled)
// 3. Easier to use with parameters
```

---

## 12. Common Coding Problems

### Q45: Reverse a String

```java
// Method 1: StringBuilder
String reverse1(String str) {
    return new StringBuilder(str).reverse().toString();
}

// Method 2: Char array
String reverse2(String str) {
    char[] chars = str.toCharArray();
    int left = 0, right = chars.length - 1;
    while (left < right) {
        char temp = chars[left];
        chars[left] = chars[right];
        chars[right] = temp;
        left++;
        right--;
    }
    return new String(chars);
}

// Method 3: Recursion
String reverse3(String str) {
    if (str.length() <= 1) return str;
    return reverse3(str.substring(1)) + str.charAt(0);
}
```

### Q46: Check Palindrome

```java
boolean isPalindrome(String str) {
    str = str.toLowerCase().replaceAll("[^a-z0-9]", "");
    int left = 0, right = str.length() - 1;
    while (left < right) {
        if (str.charAt(left) != str.charAt(right)) {
            return false;
        }
        left++;
        right--;
    }
    return true;
}
```

### Q47: Find Duplicate in Array

```java
// Method 1: Using HashSet
int findDuplicate1(int[] nums) {
    Set<Integer> seen = new HashSet<>();
    for (int num : nums) {
        if (!seen.add(num)) {
            return num;  // Found duplicate
        }
    }
    return -1;
}

// Method 2: Floyd's Cycle Detection (for 1 to n array)
int findDuplicate2(int[] nums) {
    int slow = nums[0];
    int fast = nums[0];
    
    // Find intersection point
    do {
        slow = nums[slow];
        fast = nums[nums[fast]];
    } while (slow != fast);
    
    // Find entrance to cycle
    slow = nums[0];
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[fast];
    }
    return slow;
}
```

### Q48: Fibonacci Series

```java
// Iterative
int fibonacci(int n) {
    if (n <= 1) return n;
    int prev = 0, curr = 1;
    for (int i = 2; i <= n; i++) {
        int next = prev + curr;
        prev = curr;
        curr = next;
    }
    return curr;
}

// Recursive with memoization
Map<Integer, Integer> memo = new HashMap<>();
int fibonacciMemo(int n) {
    if (n <= 1) return n;
    if (memo.containsKey(n)) return memo.get(n);
    int result = fibonacciMemo(n-1) + fibonacciMemo(n-2);
    memo.put(n, result);
    return result;
}
```

### Q49: Binary Search

```java
int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;  // Prevents overflow
        
        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return -1;  // Not found
}
```

### Q50: Implement Stack using Array

```java
class MyStack {
    private int[] arr;
    private int top;
    private int capacity;
    
    public MyStack(int capacity) {
        this.capacity = capacity;
        this.arr = new int[capacity];
        this.top = -1;
    }
    
    public void push(int x) {
        if (isFull()) {
            throw new StackOverflowError("Stack is full");
        }
        arr[++top] = x;
    }
    
    public int pop() {
        if (isEmpty()) {
            throw new EmptyStackException();
        }
        return arr[top--];
    }
    
    public int peek() {
        if (isEmpty()) {
            throw new EmptyStackException();
        }
        return arr[top];
    }
    
    public boolean isEmpty() {
        return top == -1;
    }
    
    public boolean isFull() {
        return top == capacity - 1;
    }
}
```

---

## Interview Tips

### Most Important Topics to Focus On:
1. **OOP Concepts** - Must be crystal clear
2. **Collections Framework** - HashMap internal working is favorite
3. **Multithreading** - Synchronization, thread safety
4. **Java 8 Features** - Lambda, Stream API, Optional
5. **Exception Handling** - Checked vs unchecked
6. **String Handling** - String pool, immutability
7. **Memory Management** - Stack vs Heap, GC

### Common Traps to Avoid:
- Confusing == with equals()
- Not understanding pass by value
- Mixing up ArrayList vs LinkedList performance
- Forgetting that Java doesn't have pass by reference
- Not knowing HashMap collision resolution

### Preparation Strategy:
1. **Understand concepts**, don't just memorize
2. **Write code by hand** during practice
3. **Explain your thought process** while coding
4. **Know time complexities** of operations
5. **Practice coding problems** on platforms like LeetCode

Good luck with your interview! 🚀

# Java Functional Programming Guide

## Introduction

Functional programming was introduced in **Java 8**. It allows you to write cleaner, more concise code by treating functions as first-class citizens.

---

## 1. Lambda Expressions

A lambda is an anonymous function — a short way to write a method without declaring it.

### Syntax

```java
(parameters) -> expression
(parameters) -> { statements; }
```

### Examples

**Before Java 8:**
```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};
```

**With Lambda:**
```java
Runnable r = () -> System.out.println("Hello");
```

**Comparator Example:**
```java
// Old way
Comparator<String> comp = new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.length() - b.length();
    }
};

// Lambda way
Comparator<String> comp = (a, b) -> a.length() - b.length();
```

### Lambda Syntax Rules

| Scenario | Syntax |
|----------|--------|
| No parameters | `() -> expression` |
| One parameter | `x -> expression` or `(x) -> expression` |
| Multiple parameters | `(x, y) -> expression` |
| Multiple statements | `(x, y) -> { statement1; statement2; return value; }` |

---

## 2. Functional Interfaces

An interface with **exactly one abstract method**. Lambdas work only with functional interfaces.

### Built-in Functional Interfaces

Located in `java.util.function` package:

| Interface | Method | Input → Output | Example |
|-----------|--------|----------------|---------|
| `Predicate<T>` | `test(T)` | T → boolean | Check condition |
| `Function<T,R>` | `apply(T)` | T → R | Transform data |
| `Consumer<T>` | `accept(T)` | T → void | Perform action |
| `Supplier<T>` | `get()` | () → T | Provide value |
| `BiFunction<T,U,R>` | `apply(T,U)` | (T, U) → R | Two inputs, one output |
| `BiPredicate<T,U>` | `test(T,U)` | (T, U) → boolean | Test two inputs |
| `BiConsumer<T,U>` | `accept(T,U)` | (T, U) → void | Consume two inputs |
| `UnaryOperator<T>` | `apply(T)` | T → T | Same type transform |
| `BinaryOperator<T>` | `apply(T,T)` | (T, T) → T | Combine two same types |

### Examples

```java
// Predicate - returns boolean
Predicate<Integer> isEven = n -> n % 2 == 0;
isEven.test(4);  // true
isEven.test(7);  // false

// Function - transforms input to output
Function<String, Integer> length = s -> s.length();
length.apply("Hello");  // 5

// Consumer - performs action, returns nothing
Consumer<String> printer = s -> System.out.println(s);
printer.accept("Hi");  // prints "Hi"

// Supplier - provides value, takes no input
Supplier<Double> random = () -> Math.random();
random.get();  // 0.xyz...

// BiFunction - two inputs, one output
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
add.apply(5, 3);  // 8
```

### Chaining Functional Interfaces

```java
// Predicate chaining
Predicate<Integer> isEven = n -> n % 2 == 0;
Predicate<Integer> isPositive = n -> n > 0;

Predicate<Integer> isEvenAndPositive = isEven.and(isPositive);
Predicate<Integer> isEvenOrPositive = isEven.or(isPositive);
Predicate<Integer> isOdd = isEven.negate();

// Function chaining
Function<String, String> trim = String::trim;
Function<String, String> upper = String::toUpperCase;

Function<String, String> trimAndUpper = trim.andThen(upper);
trimAndUpper.apply("  hello  ");  // "HELLO"
```

### Custom Functional Interface

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;
Calculator subtract = (a, b) -> a - b;

add.calculate(5, 3);       // 8
multiply.calculate(5, 3);  // 15
subtract.calculate(5, 3);  // 2
```

---

## 3. Method References

A shorthand for lambdas that call an existing method.

### Types of Method References

| Type | Syntax | Lambda Equivalent |
|------|--------|-------------------|
| Static method | `Class::staticMethod` | `x -> Class.staticMethod(x)` |
| Instance method (bound) | `object::method` | `() -> object.method()` |
| Instance method (unbound) | `Class::method` | `x -> x.method()` |
| Constructor | `Class::new` | `() -> new Class()` |

### Examples

```java
// Static method reference
Function<String, Integer> parse = Integer::parseInt;
parse.apply("123");  // 123

// Instance method reference (bound)
String str = "hello";
Supplier<Integer> len = str::length;
len.get();  // 5

// Instance method reference (unbound)
Function<String, String> upper = String::toUpperCase;
upper.apply("hello");  // "HELLO"

// Constructor reference
Supplier<ArrayList<String>> listMaker = ArrayList::new;
ArrayList<String> list = listMaker.get();

// Constructor with parameter
Function<String, StringBuilder> sbMaker = StringBuilder::new;
StringBuilder sb = sbMaker.apply("Hello");
```

---

## 4. Streams API

A powerful way to process collections in a functional style.

### Stream Pipeline

```
Source → Intermediate Operations → Terminal Operation
```

### Creating Streams

```java
// From Collection
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream1 = list.stream();

// From Array
String[] arr = {"a", "b", "c"};
Stream<String> stream2 = Arrays.stream(arr);

// Using Stream.of()
Stream<String> stream3 = Stream.of("a", "b", "c");

// Infinite streams
Stream<Integer> infinite = Stream.iterate(0, n -> n + 1);
Stream<Double> randoms = Stream.generate(Math::random);

// IntStream, LongStream, DoubleStream
IntStream intStream = IntStream.range(1, 10);      // 1 to 9
IntStream intStream2 = IntStream.rangeClosed(1, 10); // 1 to 10
```

### Intermediate Operations (Lazy)

These operations return a new stream and are not executed until a terminal operation is called.

| Method | Purpose | Example |
|--------|---------|---------|
| `filter(Predicate)` | Keep matching elements | `.filter(x -> x > 5)` |
| `map(Function)` | Transform elements | `.map(String::toUpperCase)` |
| `flatMap(Function)` | Flatten nested structures | `.flatMap(list -> list.stream())` |
| `sorted()` | Natural sort | `.sorted()` |
| `sorted(Comparator)` | Custom sort | `.sorted(Comparator.reverseOrder())` |
| `distinct()` | Remove duplicates | `.distinct()` |
| `limit(n)` | Take first n | `.limit(5)` |
| `skip(n)` | Skip first n | `.skip(3)` |
| `peek(Consumer)` | Debug/inspect | `.peek(System.out::println)` |

### Terminal Operations (Eager)

These operations trigger the stream pipeline execution.

| Method | Purpose | Return Type |
|--------|---------|-------------|
| `collect(Collector)` | Gather into collection | Collection |
| `toList()` | Gather into List (Java 16+) | List |
| `forEach(Consumer)` | Perform action | void |
| `count()` | Count elements | long |
| `findFirst()` | Get first element | Optional |
| `findAny()` | Get any element | Optional |
| `anyMatch(Predicate)` | Any match? | boolean |
| `allMatch(Predicate)` | All match? | boolean |
| `noneMatch(Predicate)` | None match? | boolean |
| `reduce(BinaryOperator)` | Combine into one | Optional |
| `min(Comparator)` | Find minimum | Optional |
| `max(Comparator)` | Find maximum | Optional |

### Stream Examples

```java
List<String> names = Arrays.asList("John", "Jane", "Bob", "Alice", "Anna");

// Filter and collect
List<String> filtered = names.stream()
    .filter(n -> n.startsWith("A"))
    .collect(Collectors.toList());
// [Alice, Anna]

// Map and collect
List<Integer> lengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());
// [4, 4, 3, 5, 4]

// Filter, map, sort, collect
List<String> result = names.stream()
    .filter(n -> n.length() > 3)
    .map(String::toUpperCase)
    .sorted()
    .collect(Collectors.toList());
// [ALICE, ANNA, JANE, JOHN]

// Count
long count = names.stream()
    .filter(n -> n.length() > 3)
    .count();
// 4

// Any match
boolean hasShortName = names.stream()
    .anyMatch(n -> n.length() <= 3);
// true (Bob)

// Find first
Optional<String> first = names.stream()
    .filter(n -> n.startsWith("J"))
    .findFirst();
// Optional[John]
```

### Collectors

```java
List<String> names = Arrays.asList("John", "Jane", "Bob");

// To List
List<String> list = names.stream().collect(Collectors.toList());

// To Set
Set<String> set = names.stream().collect(Collectors.toSet());

// To Map
Map<String, Integer> map = names.stream()
    .collect(Collectors.toMap(
        name -> name,           // key
        name -> name.length()   // value
    ));
// {Bob=3, John=4, Jane=4}

// Joining
String joined = names.stream()
    .collect(Collectors.joining(", "));
// "John, Jane, Bob"

// Grouping
Map<Integer, List<String>> byLength = names.stream()
    .collect(Collectors.groupingBy(String::length));
// {3=[Bob], 4=[John, Jane]}

// Counting
Map<Integer, Long> countByLength = names.stream()
    .collect(Collectors.groupingBy(String::length, Collectors.counting()));
// {3=1, 4=2}

// Partitioning (split into two groups)
Map<Boolean, List<String>> partitioned = names.stream()
    .collect(Collectors.partitioningBy(n -> n.length() > 3));
// {false=[Bob], true=[John, Jane]}

// Statistics
IntSummaryStatistics stats = names.stream()
    .collect(Collectors.summarizingInt(String::length));
// count=3, sum=11, min=3, average=3.67, max=4
```

### Reduce Operation

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Sum with identity
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);
// 15

// Sum using method reference
int sum2 = numbers.stream()
    .reduce(0, Integer::sum);
// 15

// Without identity (returns Optional)
Optional<Integer> sum3 = numbers.stream()
    .reduce(Integer::sum);
// Optional[15]

// Find max
Optional<Integer> max = numbers.stream()
    .reduce(Integer::max);
// Optional[5]

// Concatenate strings
List<String> words = Arrays.asList("Hello", "World");
String sentence = words.stream()
    .reduce("", (a, b) -> a + " " + b)
    .trim();
// "Hello World"
```

### Primitive Streams

```java
// IntStream, LongStream, DoubleStream have specialized methods
int[] numbers = {1, 2, 3, 4, 5};

int sum = Arrays.stream(numbers).sum();
OptionalDouble avg = Arrays.stream(numbers).average();
OptionalInt max = Arrays.stream(numbers).max();
OptionalInt min = Arrays.stream(numbers).min();

// Convert object stream to primitive
List<String> names = Arrays.asList("John", "Jane", "Bob");
int totalLength = names.stream()
    .mapToInt(String::length)
    .sum();
// 11
```

### Parallel Streams

```java
// Process in parallel for better performance on large datasets
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

int sum = numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .mapToInt(n -> n)
    .sum();
// 30

// Convert sequential to parallel
numbers.stream()
    .parallel()
    .forEach(System.out::println);
```

---

## 5. Optional

A container that may or may not contain a value. Helps avoid `NullPointerException`.

### Creating Optional

```java
// With value
Optional<String> opt = Optional.of("Hello");

// Empty
Optional<String> empty = Optional.empty();

// Nullable (use when value might be null)
String value = null;
Optional<String> nullable = Optional.ofNullable(value);
```

### Checking and Getting Values

```java
Optional<String> opt = Optional.of("Hello");

// Check if present
if (opt.isPresent()) {
    System.out.println(opt.get());
}

// Check if empty (Java 11+)
if (opt.isEmpty()) {
    System.out.println("Empty");
}

// Better approach - ifPresent
opt.ifPresent(System.out::println);

// ifPresentOrElse (Java 9+)
opt.ifPresentOrElse(
    value -> System.out.println("Found: " + value),
    () -> System.out.println("Not found")
);
```

### Default Values

```java
Optional<String> opt = Optional.empty();

// Default value
String result1 = opt.orElse("Default");
// "Default"

// Default with supplier (lazy evaluation)
String result2 = opt.orElseGet(() -> computeDefault());

// Throw if empty
String result3 = opt.orElseThrow();  // NoSuchElementException
String result4 = opt.orElseThrow(() -> new RuntimeException("Not found"));
```

### Transforming Optional

```java
Optional<String> opt = Optional.of("hello");

// Map - transform value
Optional<Integer> length = opt.map(String::length);
// Optional[5]

// Map with method reference
Optional<String> upper = opt.map(String::toUpperCase);
// Optional[HELLO]

// Filter - keep if condition matches
Optional<String> filtered = opt.filter(s -> s.length() > 3);
// Optional[hello]

Optional<String> filtered2 = opt.filter(s -> s.length() > 10);
// Optional.empty

// FlatMap - when transformation returns Optional
Optional<String> result = opt.flatMap(s -> Optional.of(s.toUpperCase()));
// Optional[HELLO]
```

### Chaining Optionals

```java
// Avoid nested null checks
// Old way:
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

// With Optional:
String managerName = Optional.ofNullable(employee)
    .map(Employee::getDepartment)
    .map(Department::getManager)
    .map(Manager::getName)
    .orElse("No Manager");
```

### Optional with Streams

```java
// Stream of Optionals
List<Optional<String>> optionals = Arrays.asList(
    Optional.of("a"),
    Optional.empty(),
    Optional.of("b")
);

// Filter and get values
List<String> values = optionals.stream()
    .filter(Optional::isPresent)
    .map(Optional::get)
    .collect(Collectors.toList());
// [a, b]

// Using flatMap (Java 9+)
List<String> values2 = optionals.stream()
    .flatMap(Optional::stream)
    .collect(Collectors.toList());
// [a, b]
```

---

## 6. Practical Examples

### Processing Employee Data

```java
List<Employee> employees = getEmployees();

// Get names of high earners, sorted
List<String> highEarners = employees.stream()
    .filter(e -> e.getSalary() > 50000)
    .map(Employee::getName)
    .sorted()
    .collect(Collectors.toList());

// Calculate average salary
double avgSalary = employees.stream()
    .mapToDouble(Employee::getSalary)
    .average()
    .orElse(0.0);

// Find highest paid employee
Optional<Employee> highestPaid = employees.stream()
    .max(Comparator.comparing(Employee::getSalary));

// Group by department
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// Average salary by department
Map<String, Double> avgByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));

// Top 3 earners
List<Employee> top3 = employees.stream()
    .sorted(Comparator.comparing(Employee::getSalary).reversed())
    .limit(3)
    .collect(Collectors.toList());
```

### Processing Nested Data

```java
List<Order> orders = getOrders();

// Flatten: Get all items from all orders
List<Item> allItems = orders.stream()
    .flatMap(order -> order.getItems().stream())
    .collect(Collectors.toList());

// Total revenue
double totalRevenue = orders.stream()
    .flatMap(order -> order.getItems().stream())
    .mapToDouble(item -> item.getPrice() * item.getQuantity())
    .sum();

// Unique products
Set<String> uniqueProducts = orders.stream()
    .flatMap(order -> order.getItems().stream())
    .map(Item::getProductName)
    .collect(Collectors.toSet());
```

### Word Frequency Counter

```java
String text = "the quick brown fox jumps over the lazy dog the fox";

Map<String, Long> wordFrequency = Arrays.stream(text.split(" "))
    .collect(Collectors.groupingBy(
        word -> word,
        Collectors.counting()
    ));
// {the=3, quick=1, brown=1, fox=2, jumps=1, over=1, lazy=1, dog=1}

// Most frequent word
Optional<Map.Entry<String, Long>> mostFrequent = wordFrequency.entrySet()
    .stream()
    .max(Map.Entry.comparingByValue());
```

---

## 7. Best Practices

1. **Prefer method references** over lambdas when possible for readability
2. **Keep lambdas short** — if complex, extract to a method
3. **Use primitive streams** (IntStream, etc.) for better performance
4. **Avoid side effects** in stream operations
5. **Use parallel streams carefully** — only for large datasets and stateless operations
6. **Return Optional** instead of null from methods
7. **Don't use Optional as method parameters** — use overloading instead
8. **Don't use Optional for collections** — return empty collection instead
9. **Streams are single-use** — create a new stream if needed again
10. **Use appropriate collectors** — don't reinvent the wheel

---

## 8. Common Mistakes to Avoid

```java
// ❌ Modifying source collection during stream
list.stream().forEach(item -> list.remove(item)); // ConcurrentModificationException

// ✅ Collect to new list, then remove
List<Item> toRemove = list.stream().filter(condition).collect(Collectors.toList());
list.removeAll(toRemove);

// ❌ Using get() without checking
Optional<String> opt = findValue();
String value = opt.get(); // NoSuchElementException if empty

// ✅ Use orElse or check first
String value = opt.orElse("default");

// ❌ Reusing streams
Stream<String> stream = list.stream();
stream.forEach(System.out::println);
stream.count(); // IllegalStateException: stream already closed

// ✅ Create new stream
list.stream().forEach(System.out::println);
long count = list.stream().count();
```

---

## Quick Reference Card

| Concept | Syntax |
|---------|--------|
| Lambda | `(params) -> expression` |
| Method Ref (static) | `Class::method` |
| Method Ref (instance) | `object::method` |
| Method Ref (constructor) | `Class::new` |
| Stream from List | `list.stream()` |
| Filter | `.filter(x -> condition)` |
| Map | `.map(x -> transform)` |
| Collect to List | `.collect(Collectors.toList())` |
| Optional create | `Optional.of(value)` |
| Optional default | `opt.orElse(default)` |

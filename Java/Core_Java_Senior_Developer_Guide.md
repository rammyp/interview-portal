# Core Java Senior Developer Preparation Guide
## Java 7+ Modern Features Only

---

## Table of Contents

1. [Java Version Features Overview](#1-java-version-features-overview)
2. [Lambda Expressions & Functional Interfaces](#2-lambda-expressions--functional-interfaces)
3. [Stream API](#3-stream-api)
4. [Optional API](#4-optional-api)
5. [New Date & Time API](#5-new-date--time-api)
6. [Concurrency & Multithreading](#6-concurrency--multithreading)
7. [Collections Framework Enhancements](#7-collections-framework-enhancements)
8. [Memory Management & Garbage Collection](#8-memory-management--garbage-collection)
9. [JVM Internals](#9-jvm-internals)
10. [Records, Sealed Classes & Pattern Matching](#10-records-sealed-classes--pattern-matching)
11. [Modules System (JPMS)](#11-modules-system-jpms)
12. [Virtual Threads (Project Loom)](#12-virtual-threads-project-loom)
13. [Exception Handling Enhancements](#13-exception-handling-enhancements)
14. [I/O Enhancements (NIO.2)](#14-io-enhancements-nio2)
15. [String Enhancements](#15-string-enhancements)
16. [Switch Expressions & Pattern Matching](#16-switch-expressions--pattern-matching)
17. [HTTP Client API](#17-http-client-api)
18. [Design Patterns in Modern Java](#18-design-patterns-in-modern-java)
19. [Performance Optimization](#19-performance-optimization)
20. [Interview Questions & Scenarios](#20-interview-questions--scenarios)

---

## 1. Java Version Features Overview

### Java 7 (2011)
- Try-with-resources statement
- Diamond operator `<>`
- Multi-catch exception handling
- String in switch statements
- Binary literals and underscores in numeric literals
- NIO.2 (New I/O) API
- Fork/Join Framework

### Java 8 (2014)
- Lambda expressions
- Stream API
- Optional class
- Default and static methods in interfaces
- New Date/Time API (java.time)
- Method references
- Functional interfaces

### Java 9 (2017)
- Module System (JPMS)
- JShell (REPL)
- Private methods in interfaces
- Collection factory methods
- Stream API enhancements
- Optional enhancements
- Process API improvements

### Java 10 (2018)
- Local variable type inference (`var`)
- Application Class-Data Sharing
- Parallel Full GC for G1

### Java 11 (2018 - LTS)
- HTTP Client API (standardized)
- String new methods
- Local-variable syntax for lambda
- Running single-file source code
- Removal of Java EE and CORBA modules

### Java 12-16 (2019-2021)
- Switch expressions (preview → standard)
- Text blocks
- Pattern matching for instanceof
- Records (preview → standard)
- Sealed classes (preview → standard)
- Helpful NullPointerExceptions

### Java 17 (2021 - LTS)
- Sealed classes (finalized)
- Pattern matching for switch (preview)
- Strongly encapsulate JDK internals
- Foreign Function & Memory API (incubator)

### Java 21 (2023 - LTS)
- Virtual Threads (Project Loom)
- Sequenced Collections
- Record Patterns
- Pattern Matching for switch (finalized)
- String Templates (preview)

---

## 2. Lambda Expressions & Functional Interfaces

### Lambda Syntax

```java
// No parameters
Runnable r = () -> System.out.println("Hello");

// Single parameter (parentheses optional)
Consumer<String> c = s -> System.out.println(s);

// Multiple parameters
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;

// With explicit types
BiFunction<Integer, Integer, Integer> multiply = (Integer a, Integer b) -> a * b;

// Multi-line body
Function<String, Integer> parser = s -> {
    System.out.println("Parsing: " + s);
    return Integer.parseInt(s);
};
```

### Core Functional Interfaces (java.util.function)

```java
// Predicate<T> - takes T, returns boolean
Predicate<String> isEmpty = String::isEmpty;
Predicate<Integer> isPositive = n -> n > 0;

// Combining predicates
Predicate<Integer> isPositiveEven = isPositive.and(n -> n % 2 == 0);
Predicate<String> notEmpty = isEmpty.negate();

// Function<T, R> - takes T, returns R
Function<String, Integer> length = String::length;
Function<Integer, Integer> square = n -> n * n;

// Function composition
Function<String, Integer> lengthSquared = length.andThen(square);
Function<String, Integer> alsoLengthSquared = square.compose(length);

// Consumer<T> - takes T, returns void
Consumer<String> printer = System.out::println;
Consumer<String> logger = s -> System.out.println("Log: " + s);

// Chaining consumers
Consumer<String> printAndLog = printer.andThen(logger);

// Supplier<T> - takes nothing, returns T
Supplier<LocalDateTime> now = LocalDateTime::now;
Supplier<UUID> uuidGenerator = UUID::randomUUID;

// BiFunction<T, U, R> - takes T and U, returns R
BiFunction<String, String, String> concat = String::concat;

// UnaryOperator<T> - Function where input and output are same type
UnaryOperator<String> toUpper = String::toUpperCase;

// BinaryOperator<T> - BiFunction where all types are same
BinaryOperator<Integer> max = Integer::max;
```

### Method References

```java
// Static method reference
Function<String, Integer> parser = Integer::parseInt;

// Instance method reference (bound)
String str = "hello";
Supplier<Integer> lengthSupplier = str::length;

// Instance method reference (unbound)
Function<String, Integer> lengthFunc = String::length;
BiPredicate<String, String> startsWithCheck = String::startsWith;

// Constructor reference
Supplier<ArrayList<String>> listFactory = ArrayList::new;
Function<Integer, int[]> arrayFactory = int[]::new;

// Constructor reference with parameters
Function<String, StringBuilder> sbFactory = StringBuilder::new;
```

### Custom Functional Interfaces

```java
@FunctionalInterface
public interface TriFunction<T, U, V, R> {
    R apply(T t, U u, V v);
    
    default <W> TriFunction<T, U, V, W> andThen(Function<? super R, ? extends W> after) {
        Objects.requireNonNull(after);
        return (t, u, v) -> after.apply(apply(t, u, v));
    }
}

// Usage
TriFunction<Integer, Integer, Integer, Integer> sumThree = (a, b, c) -> a + b + c;
```

### Effectively Final Variables

```java
public void process(List<String> items) {
    String prefix = "Item: ";  // effectively final
    // prefix = "New: ";       // uncommenting would cause compilation error
    
    items.forEach(item -> System.out.println(prefix + item));
}
```

### Key Interview Points

**Q: What is a functional interface?**
A: An interface with exactly one abstract method. It can have default and static methods. The `@FunctionalInterface` annotation is optional but recommended for compile-time checking.

**Q: Difference between lambda and anonymous class?**
- Lambdas don't create a new scope; `this` refers to the enclosing class
- Anonymous classes create their own scope; `this` refers to the anonymous class
- Lambdas are more concise and can be optimized by JVM using `invokedynamic`
- Lambdas can only implement functional interfaces

---

## 3. Stream API

### Creating Streams

```java
// From collections
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> streamFromList = list.stream();
Stream<String> parallelStream = list.parallelStream();

// From arrays
String[] array = {"a", "b", "c"};
Stream<String> streamFromArray = Arrays.stream(array);
Stream<String> partialStream = Arrays.stream(array, 1, 3);

// Using Stream.of()
Stream<String> streamOf = Stream.of("a", "b", "c");

// Infinite streams
Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 2);
Stream<Double> randomStream = Stream.generate(Math::random);

// With limit for infinite streams
List<Integer> evenNumbers = Stream.iterate(0, n -> n + 2)
    .limit(10)
    .collect(Collectors.toList());

// Stream.iterate with predicate (Java 9+)
Stream<Integer> bounded = Stream.iterate(0, n -> n < 100, n -> n + 10);

// Empty stream
Stream<String> emptyStream = Stream.empty();

// From other sources
IntStream chars = "hello".chars();
Stream<String> lines = Files.lines(Paths.get("file.txt"));
Stream<Path> paths = Files.walk(Paths.get("/directory"));
```

### Intermediate Operations

```java
List<Person> people = getPeople();

// filter - select elements matching predicate
Stream<Person> adults = people.stream()
    .filter(p -> p.getAge() >= 18);

// map - transform elements
Stream<String> names = people.stream()
    .map(Person::getName);

// flatMap - flatten nested structures
List<List<String>> nestedList = Arrays.asList(
    Arrays.asList("a", "b"),
    Arrays.asList("c", "d")
);
List<String> flattened = nestedList.stream()
    .flatMap(Collection::stream)
    .collect(Collectors.toList());

// distinct - remove duplicates
Stream<String> unique = Stream.of("a", "b", "a", "c").distinct();

// sorted - natural order or with comparator
Stream<Person> sortedByAge = people.stream()
    .sorted(Comparator.comparing(Person::getAge));

// sorted with multiple criteria
Stream<Person> sortedMultiple = people.stream()
    .sorted(Comparator.comparing(Person::getAge)
        .thenComparing(Person::getName)
        .reversed());

// peek - for debugging
List<String> result = Stream.of("one", "two", "three")
    .filter(s -> s.length() > 2)
    .peek(s -> System.out.println("Filtered: " + s))
    .map(String::toUpperCase)
    .peek(s -> System.out.println("Mapped: " + s))
    .collect(Collectors.toList());

// limit and skip
Stream<Integer> limitedStream = Stream.iterate(1, n -> n + 1)
    .skip(5)
    .limit(10);

// takeWhile and dropWhile (Java 9+)
List<Integer> taken = Stream.of(1, 2, 3, 4, 5, 1, 2)
    .takeWhile(n -> n < 4)
    .collect(Collectors.toList()); // [1, 2, 3]

List<Integer> dropped = Stream.of(1, 2, 3, 4, 5, 1, 2)
    .dropWhile(n -> n < 4)
    .collect(Collectors.toList()); // [4, 5, 1, 2]

// mapMulti (Java 16+)
Stream<Integer> expanded = Stream.of(1, 2, 3)
    .<Integer>mapMulti((num, consumer) -> {
        consumer.accept(num);
        consumer.accept(num * 10);
    }); // [1, 10, 2, 20, 3, 30]
```

### Terminal Operations

```java
List<Person> people = getPeople();

// forEach
people.stream().forEach(System.out::println);

// forEachOrdered - maintains encounter order in parallel streams
people.parallelStream().forEachOrdered(System.out::println);

// collect - most versatile terminal operation
List<String> list = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());

// toArray
Person[] array = people.stream().toArray(Person[]::new);

// reduce - combining elements
Optional<Integer> sum = Stream.of(1, 2, 3, 4, 5)
    .reduce((a, b) -> a + b);

Integer sumWithIdentity = Stream.of(1, 2, 3, 4, 5)
    .reduce(0, Integer::sum);

// Parallel-safe reduce with combiner
Integer parallelSum = people.parallelStream()
    .reduce(0, 
        (s, person) -> s + person.getAge(),  // accumulator
        Integer::sum);                        // combiner

// count, min, max
long count = people.stream().count();
Optional<Person> youngest = people.stream()
    .min(Comparator.comparing(Person::getAge));
Optional<Person> oldest = people.stream()
    .max(Comparator.comparing(Person::getAge));

// findFirst, findAny
Optional<Person> first = people.stream()
    .filter(p -> p.getAge() > 30)
    .findFirst();

Optional<Person> any = people.parallelStream()
    .filter(p -> p.getAge() > 30)
    .findAny();

// anyMatch, allMatch, noneMatch
boolean hasAdults = people.stream()
    .anyMatch(p -> p.getAge() >= 18);
boolean allAdults = people.stream()
    .allMatch(p -> p.getAge() >= 18);
boolean noMinors = people.stream()
    .noneMatch(p -> p.getAge() < 18);
```

### Collectors

```java
List<Person> people = getPeople();

// Basic collectors
List<String> list = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());

// toList() shorthand (Java 16+)
List<String> listNew = people.stream()
    .map(Person::getName)
    .toList();

Set<String> set = people.stream()
    .map(Person::getName)
    .collect(Collectors.toSet());

// Specific collection types
TreeSet<String> treeSet = people.stream()
    .map(Person::getName)
    .collect(Collectors.toCollection(TreeSet::new));

// joining
String names = people.stream()
    .map(Person::getName)
    .collect(Collectors.joining(", ", "[", "]"));

// summarizing
IntSummaryStatistics stats = people.stream()
    .collect(Collectors.summarizingInt(Person::getAge));

// groupingBy
Map<String, List<Person>> byCity = people.stream()
    .collect(Collectors.groupingBy(Person::getCity));

// groupingBy with downstream collector
Map<String, Long> countByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity, 
        Collectors.counting()
    ));

Map<String, Double> avgAgeByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.averagingInt(Person::getAge)
    ));

// Multi-level grouping
Map<String, Map<Integer, List<Person>>> byCityAndAge = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.groupingBy(Person::getAge)
    ));

// partitioningBy - special case with boolean key
Map<Boolean, List<Person>> adultsAndMinors = people.stream()
    .collect(Collectors.partitioningBy(p -> p.getAge() >= 18));

// toMap
Map<String, Integer> nameToAge = people.stream()
    .collect(Collectors.toMap(
        Person::getName,
        Person::getAge
    ));

// toMap with merge function for duplicates
Map<String, Integer> nameToMaxAge = people.stream()
    .collect(Collectors.toMap(
        Person::getName,
        Person::getAge,
        Integer::max
    ));

// collectingAndThen - transform result after collection
List<Person> unmodifiable = people.stream()
    .collect(Collectors.collectingAndThen(
        Collectors.toList(),
        Collections::unmodifiableList
    ));

// teeing (Java 12+)
var result = people.stream()
    .collect(Collectors.teeing(
        Collectors.minBy(Comparator.comparing(Person::getAge)),
        Collectors.maxBy(Comparator.comparing(Person::getAge)),
        (min, max) -> new AgeRange(min.orElse(null), max.orElse(null))
    ));
```

### Primitive Streams

```java
// Creating primitive streams
IntStream intStream = IntStream.range(1, 100);      // 1 to 99
IntStream intStreamClosed = IntStream.rangeClosed(1, 100); // 1 to 100
LongStream longStream = LongStream.of(1L, 2L, 3L);
DoubleStream doubleStream = DoubleStream.of(1.0, 2.0, 3.0);

// Converting from object stream
IntStream ages = people.stream()
    .mapToInt(Person::getAge);

// Primitive stream operations
int sum = IntStream.rangeClosed(1, 100).sum();
OptionalDouble average = IntStream.rangeClosed(1, 100).average();
OptionalInt max = IntStream.rangeClosed(1, 100).max();
IntSummaryStatistics stats = IntStream.rangeClosed(1, 100).summaryStatistics();

// Boxing
Stream<Integer> boxed = IntStream.range(1, 10).boxed();
```

### Parallel Streams

```java
// Creating parallel streams
Stream<Person> parallel = people.parallelStream();
Stream<Person> alsoParallel = people.stream().parallel();

// Back to sequential
Stream<Person> sequential = parallel.sequential();

// Custom thread pool for parallel streams
ForkJoinPool customPool = new ForkJoinPool(4);
List<String> result = customPool.submit(() ->
    people.parallelStream()
        .map(Person::getName)
        .collect(Collectors.toList())
).join();

// Check if parallel
boolean isParallel = people.stream().isParallel();
```

---

## 4. Optional API

### Creating Optionals

```java
// Creating optionals
Optional<String> empty = Optional.empty();
Optional<String> present = Optional.of("value");
Optional<String> nullable = Optional.ofNullable(nullableValue);

// NEVER do this - defeats the purpose
Optional<String> bad = Optional.of(null); // NullPointerException!
```

### Accessing Values

```java
Optional<String> opt = getOptionalValue();

// get() - throws NoSuchElementException if empty
String value = opt.get(); // Avoid this!

// orElse - returns default if empty
String value = opt.orElse("default");

// orElseGet - lazy evaluation of default
String value = opt.orElseGet(() -> computeDefaultValue());

// orElseThrow - throw custom exception
String value = opt.orElseThrow(() -> new CustomException("Not found"));

// orElseThrow() - throws NoSuchElementException (Java 10+)
String value = opt.orElseThrow();

// or() - return another Optional (Java 9+)
Optional<String> result = opt.or(() -> Optional.of("fallback"));

// ifPresent - execute action if present
opt.ifPresent(System.out::println);

// ifPresentOrElse (Java 9+)
opt.ifPresentOrElse(
    System.out::println,
    () -> System.out.println("Empty!")
);
```

### Transforming Optionals

```java
Optional<String> opt = Optional.of("hello");

// map - transform value
Optional<Integer> length = opt.map(String::length);

// flatMap - for nested optionals
Optional<String> city = person
    .flatMap(Person::getAddress)
    .flatMap(Address::getCity);

// filter - conditional unwrapping
Optional<String> longString = opt.filter(s -> s.length() > 3);

// stream() - convert to stream (Java 9+)
Stream<String> stream = opt.stream();

// Useful for flattening
List<String> values = optionals.stream()
    .flatMap(Optional::stream)
    .collect(Collectors.toList());
```

### Best Practices

```java
// 1. Don't use Optional for class fields
class Person {
    private String name;  // Good
    private Optional<String> nickname; // Bad - use null instead
}

// 2. Don't use Optional in parameters
void process(Optional<String> opt); // Bad
void process(String value);         // Good - use @Nullable annotation

// 3. Use Optional for return types that might be empty
Optional<Person> findById(Long id);  // Good
Person findById(Long id);            // Returns null - Less explicit

// 4. Prefer functional style over isPresent + get
// Bad
if (opt.isPresent()) {
    return opt.get().toUpperCase();
}
return "DEFAULT";

// Good
return opt.map(String::toUpperCase).orElse("DEFAULT");

// 5. Chaining optionals
Optional<String> result = getOptionalA()
    .or(this::getOptionalB)
    .or(this::getOptionalC);
```

---

## 5. New Date & Time API

### Core Classes (java.time)

```java
// LocalDate - date without time
LocalDate today = LocalDate.now();
LocalDate specific = LocalDate.of(2024, Month.MARCH, 15);
LocalDate parsed = LocalDate.parse("2024-03-15");

// LocalTime - time without date
LocalTime now = LocalTime.now();
LocalTime specific = LocalTime.of(14, 30, 45);
LocalTime parsed = LocalTime.parse("14:30:45");

// LocalDateTime - date and time without timezone
LocalDateTime current = LocalDateTime.now();
LocalDateTime combined = LocalDateTime.of(today, now);
LocalDateTime specific = LocalDateTime.of(2024, 3, 15, 14, 30, 45);

// ZonedDateTime - date/time with timezone
ZonedDateTime zoned = ZonedDateTime.now();
ZonedDateTime inTokyo = ZonedDateTime.now(ZoneId.of("Asia/Tokyo"));
ZonedDateTime converted = zoned.withZoneSameInstant(ZoneId.of("America/New_York"));

// Instant - machine timestamp
Instant instant = Instant.now();
Instant fromEpoch = Instant.ofEpochMilli(1234567890000L);
long epochMilli = instant.toEpochMilli();

// Duration - time-based amount
Duration duration = Duration.ofHours(2).plusMinutes(30);
Duration between = Duration.between(time1, time2);
long seconds = duration.getSeconds();
long minutes = duration.toMinutes();

// Period - date-based amount
Period period = Period.of(1, 2, 3); // 1 year, 2 months, 3 days
Period between = Period.between(date1, date2);
int years = period.getYears();
```

### Manipulating Dates and Times

```java
LocalDate date = LocalDate.now();

// Adding/Subtracting
LocalDate tomorrow = date.plusDays(1);
LocalDate lastMonth = date.minusMonths(1);
LocalDate nextYear = date.plus(1, ChronoUnit.YEARS);

// With methods - creating modified copy
LocalDate modified = date
    .withDayOfMonth(1)
    .withMonth(12)
    .withYear(2025);

// Adjusters
LocalDate firstDayOfMonth = date.with(TemporalAdjusters.firstDayOfMonth());
LocalDate lastDayOfYear = date.with(TemporalAdjusters.lastDayOfYear());
LocalDate nextMonday = date.with(TemporalAdjusters.next(DayOfWeek.MONDAY));
LocalDate lastFriday = date.with(TemporalAdjusters.lastInMonth(DayOfWeek.FRIDAY));
```

### Formatting and Parsing

```java
// Predefined formatters
LocalDateTime dateTime = LocalDateTime.now();
String iso = dateTime.format(DateTimeFormatter.ISO_LOCAL_DATE_TIME);

// Custom patterns
DateTimeFormatter custom = DateTimeFormatter.ofPattern("dd-MMM-yyyy HH:mm");
String formatted = dateTime.format(custom);
LocalDateTime parsed = LocalDateTime.parse("15-Mar-2024 14:30", custom);

// With locale
DateTimeFormatter localized = DateTimeFormatter
    .ofPattern("EEEE, MMMM d, yyyy", Locale.FRENCH);
```

### Interoperability with Legacy Code

```java
// Date to Instant
Date legacyDate = new Date();
Instant instant = legacyDate.toInstant();

// Instant to Date
Date backToDate = Date.from(instant);

// java.sql types
java.sql.Date sqlDate = java.sql.Date.valueOf(LocalDate.now());
LocalDate localDate = sqlDate.toLocalDate();

java.sql.Timestamp timestamp = java.sql.Timestamp.valueOf(LocalDateTime.now());
LocalDateTime localDateTime = timestamp.toLocalDateTime();
```

---

## 6. Concurrency & Multithreading

### ExecutorService Framework

```java
// Creating executors
ExecutorService single = Executors.newSingleThreadExecutor();
ExecutorService fixed = Executors.newFixedThreadPool(4);
ExecutorService cached = Executors.newCachedThreadPool();
ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(2);

// Custom thread pool
ThreadPoolExecutor custom = new ThreadPoolExecutor(
    2,                      // core pool size
    10,                     // maximum pool size
    60L, TimeUnit.SECONDS,  // keep-alive time
    new LinkedBlockingQueue<>(100), // work queue
    new ThreadPoolExecutor.CallerRunsPolicy() // rejection policy
);

// Submitting tasks
executor.execute(() -> doWork()); // Runnable, no result
Future<String> future = executor.submit(() -> computeResult()); // Callable

// Batch submission
List<Callable<String>> tasks = Arrays.asList(
    () -> "Result 1",
    () -> "Result 2",
    () -> "Result 3"
);
List<Future<String>> futures = executor.invokeAll(tasks);
String firstResult = executor.invokeAny(tasks); // Returns first completed

// Shutdown
executor.shutdown(); // Graceful shutdown
executor.shutdownNow(); // Force shutdown
boolean terminated = executor.awaitTermination(60, TimeUnit.SECONDS);
```

### CompletableFuture (Java 8+)

```java
// Creating CompletableFutures
CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<Void> cf2 = CompletableFuture.runAsync(() -> doWork());
CompletableFuture<String> completed = CompletableFuture.completedFuture("Done");

// With custom executor
ExecutorService executor = Executors.newFixedThreadPool(4);
CompletableFuture<String> cf = CompletableFuture.supplyAsync(() -> "Hello", executor);

// Transformations
CompletableFuture<Integer> lengthFuture = cf1
    .thenApply(String::toUpperCase)     // Transform result
    .thenApply(String::length);          // Chain transformations

cf1.thenAccept(System.out::println);    // Consume result
cf1.thenRun(() -> System.out.println("Done")); // Run after completion

// Combining futures
CompletableFuture<String> combined = cf1.thenCombine(
    CompletableFuture.supplyAsync(() -> " World"),
    (s1, s2) -> s1 + s2
);

// thenCompose for nested futures (flatMap equivalent)
CompletableFuture<String> composed = cf1.thenCompose(
    s -> CompletableFuture.supplyAsync(() -> s + " World")
);

// All/Any of multiple futures
CompletableFuture<Void> allOf = CompletableFuture.allOf(cf1, cf2, cf);
CompletableFuture<Object> anyOf = CompletableFuture.anyOf(cf1, cf);

// Collecting results from allOf
List<CompletableFuture<String>> futures = Arrays.asList(
    CompletableFuture.supplyAsync(() -> "A"),
    CompletableFuture.supplyAsync(() -> "B"),
    CompletableFuture.supplyAsync(() -> "C")
);

CompletableFuture<List<String>> allResults = CompletableFuture
    .allOf(futures.toArray(new CompletableFuture[0]))
    .thenApply(v -> futures.stream()
        .map(CompletableFuture::join)
        .collect(Collectors.toList()));

// Exception handling
CompletableFuture<String> handled = cf1
    .exceptionally(ex -> "Fallback value")
    .handle((result, ex) -> {
        if (ex != null) return "Error: " + ex.getMessage();
        return result;
    })
    .whenComplete((result, ex) -> {
        if (ex != null) System.err.println("Failed: " + ex);
        else System.out.println("Result: " + result);
    });

// Timeout (Java 9+)
CompletableFuture<String> withTimeout = cf1
    .orTimeout(5, TimeUnit.SECONDS)
    .completeOnTimeout("Default", 3, TimeUnit.SECONDS);

// Get results
String result = cf1.get();           // Blocking, throws checked exceptions
String result = cf1.join();          // Blocking, throws unchecked exceptions
String result = cf1.getNow("default"); // Non-blocking with default
```

### Synchronization Utilities

```java
// CountDownLatch - one-time barrier
CountDownLatch latch = new CountDownLatch(3);

for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        doWork();
        latch.countDown();
    }).start();
}

latch.await(); // Wait for all workers

// CyclicBarrier - reusable synchronization point
CyclicBarrier barrier = new CyclicBarrier(3, () -> {
    System.out.println("All parties arrived");
});

for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        try {
            doPhase1();
            barrier.await(); // Wait for all threads
            doPhase2();
            barrier.await(); // Barrier can be reused
        } catch (Exception e) {
            Thread.currentThread().interrupt();
        }
    }).start();
}

// Semaphore - limiting concurrent access
Semaphore semaphore = new Semaphore(3); // 3 permits

void accessResource() throws InterruptedException {
    semaphore.acquire();
    try {
        useResource();
    } finally {
        semaphore.release();
    }
}
```

### Locks (java.util.concurrent.locks)

```java
// ReentrantLock
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // Critical section
} finally {
    lock.unlock();
}

// Try lock with timeout
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        // Critical section
    } finally {
        lock.unlock();
    }
}

// ReadWriteLock
ReadWriteLock rwLock = new ReentrantReadWriteLock();
Lock readLock = rwLock.readLock();
Lock writeLock = rwLock.writeLock();

// StampedLock (Java 8+) - optimistic locking
StampedLock stampedLock = new StampedLock();

// Optimistic read
long stamp = stampedLock.tryOptimisticRead();
int currentX = x;
int currentY = y;
if (!stampedLock.validate(stamp)) {
    stamp = stampedLock.readLock();
    try {
        currentX = x;
        currentY = y;
    } finally {
        stampedLock.unlockRead(stamp);
    }
}
```

### Concurrent Collections

```java
// ConcurrentHashMap
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("key", 1);
map.putIfAbsent("key", 2);

// Atomic operations
map.compute("key", (k, v) -> v == null ? 1 : v + 1);
map.computeIfAbsent("key", k -> computeValue(k));
map.merge("key", 1, Integer::sum);

// Bulk operations (Java 8+)
map.forEach(4, (k, v) -> System.out.println(k + "=" + v)); // Parallel threshold
long count = map.reduceValues(4, v -> (long) v, Long::sum);

// CopyOnWriteArrayList - for read-heavy scenarios
CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();
cowList.add("element");

// BlockingQueue implementations
BlockingQueue<String> arrayBlocking = new ArrayBlockingQueue<>(100);
BlockingQueue<String> linkedBlocking = new LinkedBlockingQueue<>();
BlockingQueue<String> priority = new PriorityBlockingQueue<>();

// BlockingQueue operations
arrayBlocking.put("item");           // Blocks if full
String item = arrayBlocking.take();  // Blocks if empty
arrayBlocking.offer("item", 1, TimeUnit.SECONDS);
String item = arrayBlocking.poll(1, TimeUnit.SECONDS);
```

### Atomic Classes

```java
// AtomicInteger, AtomicLong, AtomicBoolean
AtomicInteger counter = new AtomicInteger(0);
counter.incrementAndGet();  // ++counter
counter.getAndIncrement();  // counter++
counter.addAndGet(5);       // counter += 5
counter.compareAndSet(5, 10); // CAS operation
counter.updateAndGet(x -> x * 2); // Atomic update

// AtomicReference
AtomicReference<String> ref = new AtomicReference<>("initial");
ref.set("new value");
ref.compareAndSet("initial", "updated");

// LongAdder, DoubleAdder - high contention counters (Java 8+)
LongAdder adder = new LongAdder();
adder.increment();
adder.add(10);
long sum = adder.sum();
```

### Fork/Join Framework

```java
// RecursiveTask - returns a result
class SumTask extends RecursiveTask<Long> {
    private final long[] array;
    private final int start, end;
    private static final int THRESHOLD = 1000;

    SumTask(long[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }

    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += array[i];
            }
            return sum;
        }
        
        int mid = (start + end) / 2;
        SumTask left = new SumTask(array, start, mid);
        SumTask right = new SumTask(array, mid, end);
        
        left.fork();  // Execute asynchronously
        long rightResult = right.compute();  // Compute directly
        long leftResult = left.join();  // Wait for left
        
        return leftResult + rightResult;
    }
}

// Using ForkJoinPool
ForkJoinPool pool = ForkJoinPool.commonPool();
long result = pool.invoke(new SumTask(array, 0, array.length));
```

---

## 7. Collections Framework Enhancements

### Collection Factory Methods (Java 9+)

```java
// Immutable list
List<String> list = List.of("a", "b", "c");

// Immutable set
Set<String> set = Set.of("a", "b", "c");

// Immutable map
Map<String, Integer> map = Map.of(
    "one", 1,
    "two", 2,
    "three", 3
);

// For more than 10 entries
Map<String, Integer> largeMap = Map.ofEntries(
    Map.entry("one", 1),
    Map.entry("two", 2),
    Map.entry("eleven", 11)
);

// Copying to immutable collections (Java 10+)
List<String> copy = List.copyOf(existingList);
Set<String> setCopy = Set.copyOf(existingSet);
Map<String, Integer> mapCopy = Map.copyOf(existingMap);
```

### New Map Methods (Java 8+)

```java
Map<String, Integer> map = new HashMap<>();

// getOrDefault
int value = map.getOrDefault("key", 0);

// putIfAbsent
map.putIfAbsent("key", 1);

// computeIfAbsent - lazy value computation
map.computeIfAbsent("key", k -> expensiveComputation(k));

// computeIfPresent
map.computeIfPresent("key", (k, v) -> v + 1);

// compute
map.compute("key", (k, v) -> v == null ? 1 : v + 1);

// merge
map.merge("key", 1, Integer::sum);

// replaceAll
map.replaceAll((k, v) -> v * 2);

// forEach
map.forEach((k, v) -> System.out.println(k + "=" + v));
```

### Sequenced Collections (Java 21+)

```java
// SequencedCollection interface
SequencedCollection<String> seq = new LinkedHashSet<>();
seq.addFirst("first");
seq.addLast("last");
String first = seq.getFirst();
String last = seq.getLast();
SequencedCollection<String> reversed = seq.reversed();

// SequencedMap interface
SequencedMap<String, Integer> seqMap = new LinkedHashMap<>();
seqMap.putFirst("first", 1);
seqMap.putLast("last", 100);
Map.Entry<String, Integer> firstEntry = seqMap.firstEntry();
Map.Entry<String, Integer> lastEntry = seqMap.lastEntry();
```

---

## 8. Memory Management & Garbage Collection

### JVM Memory Structure

```
+------------------------------------------+
|              JVM Memory                  |
+------------------------------------------+
|  Heap                                    |
|  +------------------------------------+  |
|  | Young Generation                   |  |
|  | +-------+ +-------+ +-------+      |  |
|  | | Eden  | | S0    | | S1    |      |  |
|  | +-------+ +-------+ +-------+      |  |
|  +------------------------------------+  |
|  | Old Generation (Tenured)           |  |
|  +------------------------------------+  |
+------------------------------------------+
|  Non-Heap                               |
|  +------------------------------------+  |
|  | Metaspace (Class metadata)         |  |
|  | Code Cache (JIT compiled code)     |  |
|  | Thread Stacks                      |  |
|  +------------------------------------+  |
+------------------------------------------+
```

### Garbage Collectors

```
Serial GC (-XX:+UseSerialGC)
- Single-threaded
- Best for: Small applications, single CPU

Parallel GC (-XX:+UseParallelGC)
- Multi-threaded for young and old generation
- Best for: Throughput-oriented applications
- Default in Java 8

G1 GC (-XX:+UseG1GC)
- Divides heap into regions
- Concurrent marking phase
- Best for: Large heaps, low latency
- Default in Java 9+
- Pause time goal: -XX:MaxGCPauseMillis=200

ZGC (-XX:+UseZGC)
- Scalable low-latency GC
- Pause times < 10ms (typically < 1ms)
- Best for: Very large heaps (TB scale)
- Production-ready in Java 15+

Shenandoah (-XX:+UseShenandoahGC)
- Low-pause-time GC
- Concurrent compaction
- Best for: Responsiveness-critical applications
```

### Key JVM Flags

```bash
# Heap sizing
-Xms2g                    # Initial heap size
-Xmx4g                    # Maximum heap size
-XX:NewRatio=2            # Old/Young ratio

# G1 specific
-XX:G1HeapRegionSize=4m   # Region size
-XX:MaxGCPauseMillis=200  # Target pause time

# Metaspace
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m

# GC logging (Java 9+)
-Xlog:gc*:file=gc.log:time,uptime:filecount=5,filesize=10m
```

### Weak References

```java
// Strong reference - prevents GC
Object strong = new Object();

// Weak reference - collected when no strong refs exist
WeakReference<Object> weak = new WeakReference<>(new Object());
Object obj = weak.get();  // May be null if GC'd

// Soft reference - collected only under memory pressure
SoftReference<Object> soft = new SoftReference<>(new Object());

// WeakHashMap - entries removed when keys are GC'd
WeakHashMap<Object, String> weakMap = new WeakHashMap<>();
```

---

## 9. JVM Internals

### Class Loading

```
Bootstrap ClassLoader     (loads core Java classes)
       ↓
Platform ClassLoader      (loads platform modules, Java 9+)
       ↓
Application ClassLoader   (loads application classes)
       ↓
Custom ClassLoaders       (user-defined)
```

### Class Loading Phases

```
Loading        → Read .class file, create Class object
Linking        → Verification, Preparation, Resolution
  Verification   → Bytecode validation
  Preparation    → Static field memory allocation
  Resolution     → Symbolic → Direct references
Initialization → Execute static initializers
```

### JVM Optimizations

```java
// 1. Escape Analysis - stack allocation
public void process() {
    Point p = new Point(1, 2);  // May be stack-allocated
    int sum = p.x + p.y;        // If 'p' doesn't escape
}

// 2. Lock Elision
public void syncMethod() {
    synchronized (localObject) {  // Lock removed if object doesn't escape
        // ...
    }
}

// 3. Method Inlining - small methods embedded at call site
```

---

## 10. Records, Sealed Classes & Pattern Matching

### Records (Java 16+)

```java
// Basic record
public record Person(String name, int age) {}

// Usage
Person p = new Person("Alice", 30);
String name = p.name();  // Accessor method
int age = p.age();

// Record with validation
public record Person(String name, int age) {
    public Person {
        if (age < 0) throw new IllegalArgumentException("Age cannot be negative");
        name = name.trim();
    }
}

// Record with additional methods
public record Rectangle(double width, double height) {
    public double area() {
        return width * height;
    }
}

// Record implementing interface
public record Circle(double radius) implements Shape {
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}
```

### Sealed Classes (Java 17+)

```java
// Sealed class - restricts which classes can extend it
public sealed class Shape 
    permits Circle, Rectangle, Square {
    
    public abstract double area();
}

// Permitted subclasses must be final, sealed, or non-sealed
public final class Circle extends Shape {
    private final double radius;
    
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

// non-sealed allows unrestricted extension
public non-sealed class Square extends Shape {
    private final double side;
    
    @Override
    public double area() {
        return side * side;
    }
}

// Sealed interfaces
public sealed interface Result<T> 
    permits Success, Failure {
}

public record Success<T>(T value) implements Result<T> {}
public record Failure<T>(String error) implements Result<T> {}
```

### Pattern Matching for instanceof (Java 16+)

```java
// Old way
if (obj instanceof String) {
    String s = (String) obj;
    System.out.println(s.length());
}

// New way with pattern variable
if (obj instanceof String s) {
    System.out.println(s.length());
}

// In expressions
public int getLength(Object obj) {
    return obj instanceof String s ? s.length() : 0;
}
```

---

## 11. Modules System (JPMS)

### Module Basics

```java
// module-info.java
module com.myapp {
    // Dependencies
    requires java.sql;
    requires java.logging;
    
    // Transitive dependency
    requires transitive java.desktop;
    
    // Export packages
    exports com.myapp.api;
    exports com.myapp.model;
    
    // Export to specific modules only
    exports com.myapp.internal to com.myapp.testing;
    
    // Open for reflection
    opens com.myapp.model;
    opens com.myapp.entities to hibernate.core;
    
    // Service provider
    provides com.myapp.spi.Service 
        with com.myapp.impl.ServiceImpl;
    
    // Service consumer
    uses com.myapp.spi.Service;
}
```

### Module Commands

```bash
# Compile module
javac -d out --module-source-path src $(find src -name "*.java")

# Run module
java --module-path out -m com.myapp/com.myapp.Main

# Create modular JAR
jar --create --file=myapp.jar --main-class=com.myapp.Main -C out/com.myapp .

# JLink - create custom runtime
jlink --module-path out:$JAVA_HOME/jmods --add-modules com.myapp --output myapp-runtime
```

---

## 12. Virtual Threads (Project Loom)

### Virtual Threads (Java 21+)

```java
// Creating virtual threads
Thread vThread = Thread.ofVirtual().start(() -> {
    System.out.println("Running in virtual thread");
});

// Named virtual thread
Thread named = Thread.ofVirtual()
    .name("my-virtual-thread")
    .start(() -> doWork());

// Virtual thread executor
try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 10_000; i++) {
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));
            return "Done";
        });
    }
}

// Checking thread type
Thread.currentThread().isVirtual();

// Structured concurrency (Preview in Java 21)
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> user = scope.fork(() -> fetchUser());
    Future<Integer> order = scope.fork(() -> fetchOrder());
    
    scope.join();
    scope.throwIfFailed();
    
    return new Response(user.resultNow(), order.resultNow());
}
```

### Virtual Threads Best Practices

```java
// 1. Don't pool virtual threads - create new ones
// Bad
ExecutorService pool = Executors.newFixedThreadPool(100);
// Good
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

// 2. Avoid synchronized for blocking operations
// Bad - pins the carrier thread
synchronized (lock) {
    blockingIO();
}
// Good - use ReentrantLock
lock.lock();
try {
    blockingIO();
} finally {
    lock.unlock();
}

// 3. Use virtual threads for I/O-bound tasks
ServerSocket server = new ServerSocket(8080);
while (true) {
    Socket socket = server.accept();
    Thread.ofVirtual().start(() -> handleConnection(socket));
}
```

---

## 13. Exception Handling Enhancements

### Try-with-Resources (Java 7+)

```java
// Basic try-with-resources
try (FileInputStream fis = new FileInputStream("file.txt");
     BufferedReader reader = new BufferedReader(new InputStreamReader(fis))) {
    return reader.readLine();
}

// Effectively final variables (Java 9+)
FileInputStream fis = new FileInputStream("file.txt");
BufferedReader reader = new BufferedReader(new InputStreamReader(fis));
try (fis; reader) {
    return reader.readLine();
}
```

### Multi-catch (Java 7+)

```java
try {
    riskyOperation();
} catch (IOException | SQLException e) {
    logger.error("Operation failed", e);
}
```

---

## 14. I/O Enhancements (NIO.2)

### Path and Files API (Java 7+)

```java
// Creating paths
Path path = Path.of("dir", "subdir", "file.txt");
Path path = Paths.get("/home/user/file.txt");

// Path operations
Path fileName = path.getFileName();
Path parent = path.getParent();
Path resolved = path.resolve("subdir");
Path relative = path.relativize(otherPath);
Path normalized = path.normalize();
```

### Files Utility Class

```java
// Reading files
String content = Files.readString(path);
List<String> lines = Files.readAllLines(path);
byte[] bytes = Files.readAllBytes(path);

// Writing files
Files.writeString(path, content);
Files.write(path, lines);

// Stream-based reading
try (Stream<String> lines = Files.lines(path)) {
    lines.filter(line -> line.contains("error"))
         .forEach(System.out::println);
}

// Copying and moving
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);

// Directory operations
try (Stream<Path> stream = Files.walk(dir)) {
    stream.filter(p -> p.toString().endsWith(".java"))
          .forEach(System.out::println);
}

// Find files
try (Stream<Path> stream = Files.find(dir, 10,
        (p, attrs) -> attrs.isRegularFile() && 
                      p.toString().endsWith(".java"))) {
    stream.forEach(System.out::println);
}
```

---

## 15. String Enhancements

### Java 11+ String Methods

```java
String str = "  hello world  ";

// Blank check (Java 11)
boolean isBlank = str.isBlank();

// Strip methods (Java 11) - Unicode aware
String stripped = str.strip();
String left = str.stripLeading();
String right = str.stripTrailing();

// Lines (Java 11)
Stream<String> lines = "line1\nline2\nline3".lines();

// Repeat (Java 11)
String repeated = "ab".repeat(3);  // "ababab"

// Indent (Java 12+)
String indented = "hello\nworld".indent(4);

// Formatted (Java 15+)
String formatted = "Hello %s".formatted("World");
```

### Text Blocks (Java 15+)

```java
String json = """
    {
        "name": "Alice",
        "age": 30
    }
    """;

// Line continuation with \
String oneLine = """
    This is a \
    single line\
    """;

// Formatting with text blocks
String html = """
    <html>
        <body>
            <h1>%s</h1>
        </body>
    </html>
    """.formatted(title);
```

---

## 16. Switch Expressions & Pattern Matching

### Switch Expressions (Java 14+)

```java
// Switch expression with arrow syntax
String result = switch (day) {
    case MONDAY, TUESDAY -> "Start of week";
    case WEDNESDAY -> "Midweek";
    case THURSDAY, FRIDAY -> "End of week";
    case SATURDAY, SUNDAY -> "Weekend";
};

// Switch expression with yield
String result = switch (day) {
    case MONDAY, TUESDAY -> "Start of week";
    case WEDNESDAY -> {
        System.out.println("Midweek!");
        yield "Midweek";
    }
    default -> "Other";
};
```

### Pattern Matching for Switch (Java 21+)

```java
// Type patterns
String describe(Object obj) {
    return switch (obj) {
        case Integer i -> "Integer: " + i;
        case String s -> "String: " + s;
        case null -> "null value";
        default -> "Unknown type";
    };
}

// Guarded patterns
String categorize(Object obj) {
    return switch (obj) {
        case Integer i when i < 0 -> "Negative integer";
        case Integer i when i == 0 -> "Zero";
        case Integer i -> "Positive integer";
        case String s when s.isEmpty() -> "Empty string";
        case String s -> "String: " + s;
        default -> "Other";
    };
}

// Pattern matching with sealed classes
double area(Shape shape) {
    return switch (shape) {
        case Circle c -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.width() * r.height();
        case Square s -> s.side() * s.side();
        // No default needed - exhaustive!
    };
}

// Record patterns
String describe(Object obj) {
    return switch (obj) {
        case Circle(double r) -> "Circle with radius " + r;
        case Rectangle(double w, double h) -> "Rectangle " + w + "x" + h;
        default -> "Unknown";
    };
}
```

---

## 17. HTTP Client API

### HTTP Client (Java 11+)

```java
// Creating client
HttpClient client = HttpClient.newBuilder()
    .version(HttpClient.Version.HTTP_2)
    .followRedirects(HttpClient.Redirect.NORMAL)
    .connectTimeout(Duration.ofSeconds(10))
    .build();

// Creating requests
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/data"))
    .GET()
    .build();

HttpRequest postRequest = HttpRequest.newBuilder()
    .uri(URI.create("https://api.example.com/data"))
    .header("Content-Type", "application/json")
    .POST(HttpRequest.BodyPublishers.ofString(jsonBody))
    .timeout(Duration.ofSeconds(30))
    .build();

// Sending requests synchronously
HttpResponse<String> response = client.send(
    request, 
    HttpResponse.BodyHandlers.ofString()
);

// Asynchronous requests
CompletableFuture<HttpResponse<String>> future = client.sendAsync(
    request,
    HttpResponse.BodyHandlers.ofString()
);

future.thenApply(HttpResponse::body)
      .thenAccept(System.out::println)
      .join();

// Different body handlers
HttpResponse<byte[]> bytesResponse = client.send(
    request, HttpResponse.BodyHandlers.ofByteArray());

HttpResponse<Path> fileResponse = client.send(
    request, HttpResponse.BodyHandlers.ofFile(Path.of("output.txt")));
```

---

## 18. Design Patterns in Modern Java

### Singleton (Thread-Safe)

```java
// Enum singleton (recommended)
public enum DatabaseConnection {
    INSTANCE;
    
    private final Connection connection;
    
    DatabaseConnection() {
        connection = createConnection();
    }
    
    public Connection getConnection() {
        return connection;
    }
}

// Bill Pugh singleton (lazy initialization)
public class Singleton {
    private Singleton() {}
    
    private static class Holder {
        static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

### Builder Pattern with Records

```java
public record Person(String name, int age, String email) {
    
    public static Builder builder() {
        return new Builder();
    }
    
    public static class Builder {
        private String name;
        private int age;
        private String email;
        
        public Builder name(String name) {
            this.name = name;
            return this;
        }
        
        public Builder age(int age) {
            this.age = age;
            return this;
        }
        
        public Builder email(String email) {
            this.email = email;
            return this;
        }
        
        public Person build() {
            return new Person(name, age, email);
        }
    }
}
```

### Strategy Pattern with Lambdas

```java
@FunctionalInterface
interface PaymentStrategy {
    void pay(BigDecimal amount);
}

class PaymentProcessor {
    public void process(BigDecimal amount, PaymentStrategy strategy) {
        strategy.pay(amount);
    }
}

// Usage
PaymentProcessor processor = new PaymentProcessor();
processor.process(new BigDecimal("100"), 
    amount -> System.out.println("Paid with credit card: " + amount));
processor.process(new BigDecimal("50"), PaymentMethods::paypal);
```

### Factory Pattern

```java
sealed interface Shape permits Circle, Rectangle, Triangle {}
record Circle(double radius) implements Shape {}
record Rectangle(double width, double height) implements Shape {}
record Triangle(double base, double height) implements Shape {}

class ShapeFactory {
    private static final Map<String, Function<double[], Shape>> FACTORIES = Map.of(
        "circle", args -> new Circle(args[0]),
        "rectangle", args -> new Rectangle(args[0], args[1]),
        "triangle", args -> new Triangle(args[0], args[1])
    );
    
    public static Shape create(String type, double... args) {
        return Optional.ofNullable(FACTORIES.get(type.toLowerCase()))
            .map(factory -> factory.apply(args))
            .orElseThrow(() -> new IllegalArgumentException("Unknown shape: " + type));
    }
}
```

---

## 19. Performance Optimization

### JVM Tuning

```bash
# Heap sizing
-Xms4g -Xmx4g            # Fixed heap (avoids resizing)
-XX:NewRatio=2           # Old = 2 * Young

# GC selection
-XX:+UseG1GC             # G1 for balanced performance
-XX:+UseZGC              # ZGC for low latency
-XX:MaxGCPauseMillis=200 # G1 pause target

# JIT compilation
-XX:+TieredCompilation   # Default, balanced
-XX:TieredStopAtLevel=1  # Faster startup (C1 only)
```

### Code Optimization Tips

```java
// 1. Use StringBuilder for string concatenation in loops
StringBuilder sb = new StringBuilder();
for (String s : strings) {
    sb.append(s);
}

// 2. Use primitive streams for numeric operations
int sum = IntStream.range(0, 1000).sum();

// 3. Avoid creating unnecessary objects
// Bad
for (int i = 0; i < 1000; i++) {
    String s = new String("constant");
}
// Good
String constant = "constant";
for (int i = 0; i < 1000; i++) {
    // use constant
}

// 4. Use appropriate collection sizes
List<String> list = new ArrayList<>(expectedSize);
Map<String, Integer> map = new HashMap<>(expectedSize);

// 5. Prefer EnumSet/EnumMap for enum keys
Set<DayOfWeek> weekend = EnumSet.of(SATURDAY, SUNDAY);
Map<DayOfWeek, String> schedule = new EnumMap<>(DayOfWeek.class);

// 6. Use parallelStream() wisely
// Good for CPU-intensive, independent operations
list.parallelStream()
    .map(this::expensiveComputation)
    .collect(Collectors.toList());
```

---

## 20. Interview Questions & Scenarios

### Lambda & Streams

**Q: What's the difference between map() and flatMap()?**
- `map()` transforms each element to one output element
- `flatMap()` transforms each element to zero or more elements, then flattens the result

**Q: When should you use parallel streams?**
- Large datasets (>10,000 elements)
- CPU-intensive operations
- Independent operations (no shared state)
- Avoid for I/O operations or small collections

**Q: What makes a stream lazy?**
- Intermediate operations return a new stream without processing
- Processing only happens when a terminal operation is invoked

### Concurrency

**Q: CompletableFuture vs Future?**
- `Future` blocks on get(), has no chaining
- `CompletableFuture` supports non-blocking operations, chaining, combining, exception handling

**Q: When to use ReentrantLock vs synchronized?**
- `ReentrantLock`: need fairness, try-lock, multiple conditions, interruptible
- `synchronized`: simpler, automatic release, sufficient for most cases

**Q: Virtual threads vs platform threads?**
- Virtual threads: lightweight, millions possible, best for I/O
- Platform threads: heavyweight, limited by OS, best for CPU-bound tasks

### Memory & GC

**Q: How does G1 GC work?**
- Divides heap into regions
- Prioritizes regions with most garbage (Garbage-First)
- Concurrent marking phase
- Aims for consistent pause times

**Q: What causes memory leaks in Java?**
- Static collections holding references
- Unclosed resources
- Inner classes holding outer references
- ThreadLocal not removed
- Listeners/callbacks not unregistered

### Records & Sealed Classes

**Q: Can records be extended?**
- No, records are implicitly final
- Records can implement interfaces

**Q: What are the benefits of sealed classes?**
- Exhaustive pattern matching
- Better encapsulation of class hierarchies
- Compiler can verify all subclasses are handled

### Module System

**Q: What problems does JPMS solve?**
- Strong encapsulation (internal APIs hidden)
- Reliable configuration (missing dependencies detected at compile time)
- Smaller runtime images (jlink)
- Better security (explicit access control)

---

## Quick Reference Card

### Stream Operations

```java
// Intermediate (lazy)
filter(), map(), flatMap(), distinct(), sorted(), 
peek(), limit(), skip(), takeWhile(), dropWhile()

// Terminal (triggers processing)
forEach(), collect(), reduce(), count(), 
min(), max(), findFirst(), findAny(),
anyMatch(), allMatch(), noneMatch(), toArray()
```

### Collectors

```java
toList(), toSet(), toMap(), toUnmodifiableList()
joining(), counting(), summarizing*()
groupingBy(), partitioningBy()
mapping(), filtering(), flatMapping()
reducing(), collectingAndThen(), teeing()
```

### CompletableFuture

```java
// Create
supplyAsync(), runAsync(), completedFuture()

// Transform
thenApply(), thenAccept(), thenRun()
thenApplyAsync(), thenAcceptAsync(), thenRunAsync()

// Combine
thenCombine(), thenCompose(), allOf(), anyOf()

// Handle errors
exceptionally(), handle(), whenComplete()

// Timeout (Java 9+)
orTimeout(), completeOnTimeout()
```

### Common JVM Flags

```bash
-Xms/-Xmx          # Heap size
-XX:+UseG1GC       # G1 Garbage Collector
-XX:+UseZGC        # Z Garbage Collector
-XX:MaxGCPauseMillis=N  # Target GC pause
-Xlog:gc*          # GC logging (Java 9+)
```

---

*Last updated: 2024 | Covers Java 7 through Java 21 LTS*

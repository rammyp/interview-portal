# Java Collections Framework - Interview Preparation Guide
## Java 8 through Java 17 Features

---

## Table of Contents

1. [Collection Framework Overview](#1-collection-framework-overview)
2. [List Implementations](#2-list-implementations)
3. [Set Implementations](#3-set-implementations)
4. [Map Implementations](#4-map-implementations)
5. [Queue and Deque](#5-queue-and-deque)
6. [Java 8 Collection Enhancements](#6-java-8-collection-enhancements)
7. [Java 9 Collection Factory Methods](#7-java-9-collection-factory-methods)
8. [Java 10-17 Enhancements](#8-java-10-17-enhancements)
9. [Concurrent Collections](#9-concurrent-collections)
10. [Common Interview Questions](#10-common-interview-questions)
11. [Performance Comparison Charts](#11-performance-comparison-charts)
12. [Best Practices](#12-best-practices)

---

## 1. Collection Framework Overview

### 1.1 Collection Hierarchy

```
Iterable<E>
  └── Collection<E>
        ├── List<E> (ordered, allows duplicates)
        │     ├── ArrayList
        │     ├── LinkedList
        │     ├── Vector (legacy, synchronized)
        │     ├── Stack (legacy)
        │     └── CopyOnWriteArrayList
        │
        ├── Set<E> (no duplicates)
        │     ├── HashSet
        │     ├── LinkedHashSet
        │     ├── TreeSet (sorted)
        │     ├── EnumSet
        │     └── CopyOnWriteArraySet
        │
        └── Queue<E>
              ├── PriorityQueue
              ├── ArrayDeque
              ├── LinkedList
              └── BlockingQueue implementations

Map<K,V> (separate hierarchy - NOT a Collection)
  ├── HashMap
  ├── LinkedHashMap
  ├── TreeMap (sorted)
  ├── Hashtable (legacy, synchronized)
  ├── ConcurrentHashMap
  ├── EnumMap
  └── WeakHashMap
```

### 1.2 Key Interfaces Summary

| Interface | Characteristics | Null Allowed | Ordered | Sorted |
|-----------|-----------------|--------------|---------|--------|
| List | Duplicates allowed, index-based | Yes | Yes (insertion) | No |
| Set | No duplicates | Depends on impl | Depends | Depends |
| Queue | FIFO (typically) | Depends | Yes | No |
| Deque | Double-ended queue | No (ArrayDeque) | Yes | No |
| Map | Key-value pairs | Depends on impl | Depends | Depends |

---

## 2. List Implementations

### 2.1 ArrayList vs LinkedList

| Aspect | ArrayList | LinkedList |
|--------|-----------|------------|
| **Internal Structure** | Dynamic array | Doubly linked list |
| **Random Access** | O(1) | O(n) |
| **Insert at End** | O(1) amortized | O(1) |
| **Insert at Middle** | O(n) | O(n) traversal + O(1) insert |
| **Delete** | O(n) | O(n) traversal + O(1) unlink |
| **Memory Overhead** | Low | High (node objects) |
| **Cache Performance** | Excellent | Poor |
| **Implements** | List, RandomAccess | List, Deque, Queue |

### 2.2 ArrayList Deep Dive

**Internal Working:**
- Default capacity: 10 (lazy initialization since Java 7)
- Growth formula: `newCapacity = oldCapacity + (oldCapacity >> 1)` (~1.5x)
- Uses `System.arraycopy()` for shifting elements

**Key Methods:**
```java
// Pre-sizing for better performance
List<String> list = new ArrayList<>(1000);

// Reduce memory after bulk removals
((ArrayList<String>) list).trimToSize();

// Ensure capacity before bulk additions
((ArrayList<String>) list).ensureCapacity(5000);

// Java 8+ removeIf
list.removeIf(s -> s.startsWith("temp"));

// Java 8+ replaceAll
list.replaceAll(String::toUpperCase);
```

**Interview Question: Why is ArrayList preferred over LinkedList in most cases?**
> ArrayList has better cache locality due to contiguous memory allocation. Modern CPUs are optimized for sequential memory access. LinkedList's nodes are scattered in heap memory, causing frequent cache misses. Even for insertions, ArrayList often performs better for small to medium lists.

### 2.3 LinkedList Deep Dive

**Unique Features:**
- Implements both `List` and `Deque` interfaces
- Can be used as a stack or queue
- Each node holds reference to previous and next nodes

```java
LinkedList<String> list = new LinkedList<>();

// Deque operations
list.addFirst("first");
list.addLast("last");
list.offerFirst("newFirst");
list.offerLast("newLast");

// Stack operations
list.push("pushed");
String popped = list.pop();

// Queue operations
list.offer("queued");
String polled = list.poll();

// Peek operations (don't remove)
String first = list.peekFirst();
String last = list.peekLast();
```

### 2.4 Vector and Stack (Legacy)

```java
// Vector - synchronized ArrayList (legacy)
Vector<String> vector = new Vector<>();

// Stack - extends Vector (legacy)
Stack<String> stack = new Stack<>();

// Modern alternatives:
// Instead of Vector -> Collections.synchronizedList(new ArrayList<>())
// Instead of Stack -> ArrayDeque
Deque<String> modernStack = new ArrayDeque<>();
```

### 2.5 CopyOnWriteArrayList

**Characteristics:**
- Thread-safe without explicit synchronization
- Creates a new copy on every write operation
- Iterators never throw `ConcurrentModificationException`
- Best for read-heavy, write-rare scenarios

```java
CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>();

// Safe iteration during modification
for (String item : cowList) {
    cowList.add("new item"); // Creates new array copy
}
```

---

## 3. Set Implementations

### 3.1 HashSet vs LinkedHashSet vs TreeSet

| Aspect | HashSet | LinkedHashSet | TreeSet |
|--------|---------|---------------|---------|
| **Ordering** | None | Insertion order | Sorted (natural/comparator) |
| **Null** | One null | One null | No null (if using natural order) |
| **Performance** | O(1) | O(1) | O(log n) |
| **Internal** | HashMap | LinkedHashMap | TreeMap (Red-Black tree) |
| **Use Case** | General purpose | Maintain insertion order | Sorted data |

### 3.2 HashSet Deep Dive

**Internal Working:**
- Backed by `HashMap` with dummy value `PRESENT`
- Uses `hashCode()` and `equals()` for uniqueness
- Load factor: 0.75 (default)
- Initial capacity: 16

```java
Set<String> set = new HashSet<>();

// Java 9+ factory methods
Set<String> immutableSet = Set.of("a", "b", "c");

// Custom initial capacity and load factor
Set<String> optimizedSet = new HashSet<>(100, 0.5f);
```

**Critical: hashCode() and equals() Contract**
```java
public class Employee {
    private int id;
    private String name;
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Employee employee = (Employee) o;
        return id == employee.id && Objects.equals(name, employee.name);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}
```

### 3.3 TreeSet Deep Dive

**Key Features:**
- Implements `NavigableSet` interface
- Backed by `TreeMap` (Red-Black tree)
- Elements must be `Comparable` or provide `Comparator`

```java
// Natural ordering
TreeSet<Integer> numbers = new TreeSet<>();

// Custom comparator
TreeSet<String> caseInsensitive = new TreeSet<>(String.CASE_INSENSITIVE_ORDER);

// Reverse order
TreeSet<Integer> descending = new TreeSet<>(Comparator.reverseOrder());

// NavigableSet methods
TreeSet<Integer> set = new TreeSet<>(List.of(1, 3, 5, 7, 9));
set.lower(5);      // 3 - strictly less than
set.floor(5);      // 5 - less than or equal
set.ceiling(5);    // 5 - greater than or equal
set.higher(5);     // 7 - strictly greater than
set.headSet(5);    // [1, 3] - elements < 5
set.tailSet(5);    // [5, 7, 9] - elements >= 5
set.subSet(3, 7);  // [3, 5] - range [3, 7)

// Descending view
NavigableSet<Integer> desc = set.descendingSet();
```

### 3.4 EnumSet

**Characteristics:**
- Specialized Set for enum types
- Extremely efficient (bit vector internally)
- All elements must come from same enum type
- Maintains natural order (declaration order)

```java
enum Day { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY }

// Create EnumSet
EnumSet<Day> weekdays = EnumSet.range(Day.MONDAY, Day.FRIDAY);
EnumSet<Day> weekend = EnumSet.of(Day.SATURDAY, Day.SUNDAY);
EnumSet<Day> allDays = EnumSet.allOf(Day.class);
EnumSet<Day> noDays = EnumSet.noneOf(Day.class);
EnumSet<Day> complement = EnumSet.complementOf(weekend); // weekdays
```

---

## 4. Map Implementations

### 4.1 HashMap vs LinkedHashMap vs TreeMap vs Hashtable

| Aspect | HashMap | LinkedHashMap | TreeMap | Hashtable |
|--------|---------|---------------|---------|-----------|
| **Ordering** | None | Insertion/Access | Sorted | None |
| **Null Keys** | One | One | No | No |
| **Null Values** | Multiple | Multiple | Multiple | No |
| **Thread-Safe** | No | No | No | Yes |
| **Performance** | O(1) | O(1) | O(log n) | O(1) |
| **Since** | Java 1.2 | Java 1.4 | Java 1.2 | Java 1.0 |

### 4.2 HashMap Deep Dive

**Internal Structure (Java 8+):**
- Array of `Node<K,V>` (buckets)
- Default capacity: 16
- Load factor: 0.75
- Tree threshold: 8 (converts to TreeNode)
- Untreeify threshold: 6

**Hash Collision Handling:**
- Java 7: Linked list only
- Java 8+: Linked list → Red-Black tree (when bucket size > 8)

```
Bucket Array:
[0] -> null
[1] -> Node -> Node -> Node (linked list if size <= 8)
[2] -> TreeNode (red-black tree if size > 8)
[3] -> null
...
```

**Key Operations:**
```java
Map<String, Integer> map = new HashMap<>();

// Basic operations
map.put("key", 1);
map.get("key");
map.remove("key");
map.containsKey("key");
map.containsValue(1);

// Java 8+ methods
map.getOrDefault("missing", 0);
map.putIfAbsent("key", 2);
map.computeIfAbsent("key", k -> expensiveComputation(k));
map.computeIfPresent("key", (k, v) -> v + 1);
map.compute("key", (k, v) -> v == null ? 1 : v + 1);
map.merge("key", 1, Integer::sum);
map.replaceAll((k, v) -> v * 2);

// Iteration (Java 8+)
map.forEach((k, v) -> System.out.println(k + ": " + v));
```

**Java 8 HashMap Improvements:**
```java
// Before Java 8: O(n) worst case for collisions
// After Java 8: O(log n) worst case (treeification)

// Treeification requirements:
// 1. Bucket size > TREEIFY_THRESHOLD (8)
// 2. Table size >= MIN_TREEIFY_CAPACITY (64)
// Otherwise, table is resized instead of treeified
```

### 4.3 LinkedHashMap Deep Dive

**Unique Features:**
- Maintains doubly-linked list for iteration order
- Can maintain insertion order or access order
- Perfect for implementing LRU cache

```java
// Insertion order (default)
Map<String, Integer> insertionOrder = new LinkedHashMap<>();

// Access order (true = access order, false = insertion order)
Map<String, Integer> accessOrder = new LinkedHashMap<>(16, 0.75f, true);

// LRU Cache implementation
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // access order
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}

// Usage
LRUCache<String, String> cache = new LRUCache<>(3);
cache.put("a", "1");
cache.put("b", "2");
cache.put("c", "3");
cache.get("a");        // Access "a"
cache.put("d", "4");   // Removes "b" (least recently used)
```

### 4.4 TreeMap Deep Dive

**Key Features:**
- Red-Black tree implementation
- Implements `NavigableMap`
- Keys must be `Comparable` or provide `Comparator`

```java
TreeMap<String, Integer> map = new TreeMap<>();

// NavigableMap methods
map.firstKey();              // Smallest key
map.lastKey();               // Largest key
map.lowerKey("c");           // Greatest key < "c"
map.floorKey("c");           // Greatest key <= "c"
map.ceilingKey("c");         // Smallest key >= "c"
map.higherKey("c");          // Smallest key > "c"

// Range views
map.headMap("c");            // Keys < "c"
map.tailMap("c");            // Keys >= "c"
map.subMap("a", "c");        // Keys in range [a, c)
map.subMap("a", true, "c", true); // Keys in range [a, c]

// Descending view
NavigableMap<String, Integer> desc = map.descendingMap();

// Poll methods (remove and return)
Map.Entry<String, Integer> first = map.pollFirstEntry();
Map.Entry<String, Integer> last = map.pollLastEntry();
```

### 4.5 ConcurrentHashMap

**Java 7 vs Java 8 Implementation:**

| Aspect | Java 7 | Java 8+ |
|--------|--------|---------|
| Locking | Segment-based (16 segments) | Node-level (CAS + synchronized) |
| Granularity | Segment lock | Per-bucket lock |
| Null | Not allowed | Not allowed |
| Tree | No | Yes (like HashMap) |

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Atomic operations
map.putIfAbsent("key", 1);
map.remove("key", 1);            // Remove only if value matches
map.replace("key", 1, 2);        // Replace only if old value matches

// Java 8+ bulk operations
map.forEach(1, (k, v) -> System.out.println(k + ": " + v));
map.search(1, (k, v) -> v > 100 ? k : null);
map.reduce(1, (k, v) -> v, Integer::sum);

// Compute operations
map.compute("key", (k, v) -> v == null ? 1 : v + 1);

// Java 8+ - atomic counter pattern
map.merge("counter", 1L, Long::sum);
```

### 4.6 WeakHashMap

**Characteristics:**
- Keys are weak references
- Entries automatically removed when key is garbage collected
- Perfect for caching scenarios

```java
WeakHashMap<Object, String> cache = new WeakHashMap<>();

Object key = new Object();
cache.put(key, "value");

key = null; // Key becomes eligible for GC
System.gc(); // Entry may be removed
```

### 4.7 EnumMap

**Characteristics:**
- Keys must be enum type
- Extremely efficient (array-based)
- Maintains natural enum order

```java
enum Status { PENDING, ACTIVE, COMPLETED }

EnumMap<Status, List<Task>> tasksByStatus = new EnumMap<>(Status.class);
tasksByStatus.put(Status.PENDING, new ArrayList<>());
```

---

## 5. Queue and Deque

### 5.1 Queue Interface Methods

| Operation | Throws Exception | Returns Special Value |
|-----------|------------------|----------------------|
| Insert | `add(e)` | `offer(e)` → boolean |
| Remove | `remove()` | `poll()` → null |
| Examine | `element()` | `peek()` → null |

### 5.2 PriorityQueue

**Characteristics:**
- Min-heap by default (natural ordering)
- NOT thread-safe
- Does not permit null
- O(log n) for offer/poll, O(1) for peek

```java
// Min-heap (default)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// Max-heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());

// Custom comparator
PriorityQueue<Task> taskQueue = new PriorityQueue<>(
    Comparator.comparing(Task::getPriority)
              .thenComparing(Task::getCreatedTime)
);

// Operations
minHeap.offer(5);
minHeap.offer(1);
minHeap.offer(3);
minHeap.peek();    // 1 (doesn't remove)
minHeap.poll();    // 1 (removes)
```

### 5.3 ArrayDeque

**Characteristics:**
- Resizable array implementation of Deque
- Faster than LinkedList for stack/queue operations
- No capacity restrictions
- NOT thread-safe, does not permit null

```java
Deque<String> deque = new ArrayDeque<>();

// Stack operations (LIFO)
deque.push("first");
deque.push("second");
deque.pop();           // "second"

// Queue operations (FIFO)
deque.offer("first");
deque.offer("second");
deque.poll();          // "first"

// Double-ended operations
deque.addFirst("a");
deque.addLast("b");
deque.removeFirst();
deque.removeLast();
deque.peekFirst();
deque.peekLast();
```

### 5.4 Deque Interface Methods Summary

| First Element (Head) | Last Element (Tail) |
|---------------------|---------------------|
| `addFirst(e)` / `offerFirst(e)` | `addLast(e)` / `offerLast(e)` |
| `removeFirst()` / `pollFirst()` | `removeLast()` / `pollLast()` |
| `getFirst()` / `peekFirst()` | `getLast()` / `peekLast()` |

---

## 6. Java 8 Collection Enhancements

### 6.1 forEach Method

```java
List<String> list = Arrays.asList("a", "b", "c");

// Iterable.forEach
list.forEach(System.out::println);

// Map.forEach
Map<String, Integer> map = new HashMap<>();
map.forEach((k, v) -> System.out.println(k + "=" + v));
```

### 6.2 removeIf Method

```java
List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
numbers.removeIf(n -> n % 2 == 0); // Remove even numbers
// Result: [1, 3, 5]
```

### 6.3 replaceAll Method

```java
List<String> list = new ArrayList<>(Arrays.asList("a", "b", "c"));
list.replaceAll(String::toUpperCase);
// Result: ["A", "B", "C"]
```

### 6.4 sort Method in List

```java
List<String> list = new ArrayList<>(Arrays.asList("banana", "apple", "cherry"));

// Natural order
list.sort(Comparator.naturalOrder());

// Reverse order
list.sort(Comparator.reverseOrder());

// Custom comparator
list.sort(Comparator.comparing(String::length)
                    .thenComparing(Comparator.naturalOrder()));
```

### 6.5 spliterator Method

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
Spliterator<Integer> spliterator = list.spliterator();

// Characteristics
spliterator.characteristics(); // SIZED, ORDERED, SUBSIZED

// Parallel processing
Spliterator<Integer> secondHalf = spliterator.trySplit();
```

### 6.6 Stream API Integration

```java
List<String> list = Arrays.asList("apple", "banana", "cherry");

// stream() and parallelStream()
list.stream()
    .filter(s -> s.startsWith("a"))
    .map(String::toUpperCase)
    .collect(Collectors.toList());

// Collection from Stream
Set<String> set = list.stream().collect(Collectors.toSet());
Map<String, Integer> map = list.stream()
    .collect(Collectors.toMap(s -> s, String::length));
```

### 6.7 Map Enhancements (Java 8)

```java
Map<String, Integer> map = new HashMap<>();

// getOrDefault
int value = map.getOrDefault("missing", 0);

// putIfAbsent
map.putIfAbsent("key", 1);

// computeIfAbsent - lazy computation
map.computeIfAbsent("key", k -> expensiveOperation(k));

// computeIfPresent
map.computeIfPresent("key", (k, v) -> v + 1);

// compute
map.compute("key", (k, v) -> v == null ? 1 : v + 1);

// merge - perfect for counting
map.merge("key", 1, Integer::sum);

// replaceAll
map.replaceAll((k, v) -> v * 2);

// Word frequency counter
String[] words = {"apple", "banana", "apple", "cherry", "banana", "apple"};
Map<String, Long> frequency = new HashMap<>();
for (String word : words) {
    frequency.merge(word, 1L, Long::sum);
}
// Or with Streams
Map<String, Long> freq = Arrays.stream(words)
    .collect(Collectors.groupingBy(w -> w, Collectors.counting()));
```

---

## 7. Java 9 Collection Factory Methods

### 7.1 Immutable Collections

```java
// List.of() - Immutable list
List<String> list = List.of("a", "b", "c");

// Set.of() - Immutable set
Set<String> set = Set.of("a", "b", "c");

// Map.of() - Immutable map (up to 10 entries)
Map<String, Integer> map = Map.of(
    "a", 1,
    "b", 2,
    "c", 3
);

// Map.ofEntries() - Immutable map (any number of entries)
Map<String, Integer> largeMap = Map.ofEntries(
    Map.entry("a", 1),
    Map.entry("b", 2),
    Map.entry("c", 3),
    Map.entry("d", 4)
);
```

### 7.2 Characteristics of Factory Methods

| Characteristic | Description |
|---------------|-------------|
| **Immutable** | No add, remove, or set operations |
| **Null prohibited** | No null elements or keys/values |
| **Serializable** | If all elements are serializable |
| **Duplicate rejection** | Set.of() and Map.of() reject duplicates |
| **Iteration order** | Undefined for Set and Map |

```java
// These throw UnsupportedOperationException
List<String> list = List.of("a", "b");
list.add("c");        // UnsupportedOperationException
list.remove("a");     // UnsupportedOperationException
list.set(0, "x");     // UnsupportedOperationException

// These throw NullPointerException
List<String> nullList = List.of("a", null); // NPE

// These throw IllegalArgumentException
Set<String> dupSet = Set.of("a", "a"); // IAE - duplicate
```

### 7.3 copyOf Methods (Java 10)

```java
List<String> mutableList = new ArrayList<>(Arrays.asList("a", "b", "c"));

// Create immutable copy
List<String> immutableCopy = List.copyOf(mutableList);

// Modifying original doesn't affect copy
mutableList.add("d");
// immutableCopy still has ["a", "b", "c"]

// If source is already immutable, same instance returned
List<String> immutable = List.of("a", "b");
List<String> sameCopy = List.copyOf(immutable); // Same instance
```

---

## 8. Java 10-17 Enhancements

### 8.1 Java 10: Local Variable Type Inference with Collections

```java
// var with collections
var list = new ArrayList<String>();    // ArrayList<String>
var map = new HashMap<String, Integer>(); // HashMap<String, Integer>
var set = Set.of("a", "b", "c");       // Set<String>

// Be careful with diamond operator
var list2 = new ArrayList<>();         // ArrayList<Object> - not recommended
```

### 8.2 Java 11: Collection.toArray(IntFunction)

```java
List<String> list = List.of("a", "b", "c");

// Old way
String[] arr1 = list.toArray(new String[0]);

// Java 11 way
String[] arr2 = list.toArray(String[]::new);
```

### 8.3 Java 12: Collectors.teeing

```java
// Process stream with two collectors simultaneously
var result = Stream.of(1, 2, 3, 4, 5)
    .collect(Collectors.teeing(
        Collectors.summingInt(i -> i),    // Sum
        Collectors.counting(),            // Count
        (sum, count) -> sum / count.doubleValue()  // Average
    ));
```

### 8.4 Java 14: Record Classes with Collections

```java
// Records as collection elements
record Person(String name, int age) {}

List<Person> people = List.of(
    new Person("Alice", 30),
    new Person("Bob", 25)
);

// Sorting records
people.stream()
    .sorted(Comparator.comparing(Person::age))
    .toList();
```

### 8.5 Java 16: Stream.toList()

```java
List<String> list = Arrays.asList("a", "b", "c");

// Before Java 16
List<String> filtered1 = list.stream()
    .filter(s -> !s.equals("a"))
    .collect(Collectors.toList());  // Mutable list

// Java 16+
List<String> filtered2 = list.stream()
    .filter(s -> !s.equals("a"))
    .toList();  // Unmodifiable list
```

### 8.6 Java 16: mapMulti

```java
// Alternative to flatMap for simple cases
List<Integer> numbers = List.of(1, 2, 3);

List<Integer> doubled = numbers.stream()
    .<Integer>mapMulti((num, consumer) -> {
        consumer.accept(num);
        consumer.accept(num * 2);
    })
    .toList();
// Result: [1, 2, 2, 4, 3, 6]
```

### 8.7 Java 17: Pattern Matching Preview Features

```java
// Enhanced instanceof with collections
Object obj = List.of("a", "b", "c");

if (obj instanceof List<?> list && !list.isEmpty()) {
    System.out.println("First element: " + list.get(0));
}
```

### 8.8 SequencedCollection (Java 21 Preview - for awareness)

```java
// New interface hierarchy for ordered collections
// SequencedCollection extends Collection
// SequencedSet extends Set, SequencedCollection
// SequencedMap extends Map

// New methods (preview):
// - addFirst(E), addLast(E)
// - getFirst(), getLast()
// - removeFirst(), removeLast()
// - reversed()
```

---

## 9. Concurrent Collections

### 9.1 Overview

| Collection | Thread-Safe | Lock Type | Null Elements |
|------------|-------------|-----------|---------------|
| ConcurrentHashMap | Yes | Segment/Node | No |
| CopyOnWriteArrayList | Yes | Copy on write | Yes |
| CopyOnWriteArraySet | Yes | Copy on write | Yes |
| ConcurrentSkipListMap | Yes | Lock-free | No |
| ConcurrentSkipListSet | Yes | Lock-free | No |
| ConcurrentLinkedQueue | Yes | Lock-free | No |
| ConcurrentLinkedDeque | Yes | Lock-free | No |
| LinkedBlockingQueue | Yes | ReentrantLock | No |
| ArrayBlockingQueue | Yes | ReentrantLock | No |
| PriorityBlockingQueue | Yes | ReentrantLock | No |

### 9.2 ConcurrentHashMap Operations

```java
ConcurrentHashMap<String, AtomicLong> counters = new ConcurrentHashMap<>();

// Atomic increment
counters.computeIfAbsent("visits", k -> new AtomicLong()).incrementAndGet();

// Bulk operations with parallelism threshold
counters.forEach(1, (k, v) -> System.out.println(k + ": " + v));

long total = counters.reduceValues(1, AtomicLong::get, Long::sum);

String maxKey = counters.search(1, (k, v) -> v.get() > 100 ? k : null);

// Aggregate operations
counters.forEachEntry(1, entry -> {
    System.out.println(entry.getKey() + " = " + entry.getValue());
});
```

### 9.3 BlockingQueue Operations

| Operation | Blocks | Timeout | Returns null/false |
|-----------|--------|---------|-------------------|
| `put(e)` | Yes | - | - |
| `take()` | Yes | - | - |
| `offer(e, time, unit)` | - | Yes | false |
| `poll(time, unit)` | - | Yes | null |

```java
BlockingQueue<Task> queue = new LinkedBlockingQueue<>(100);

// Producer
queue.put(new Task()); // Blocks if full

// Consumer
Task task = queue.take(); // Blocks if empty

// With timeout
boolean added = queue.offer(new Task(), 1, TimeUnit.SECONDS);
Task polled = queue.poll(1, TimeUnit.SECONDS);
```

### 9.4 Synchronized Wrappers

```java
// Collections utility methods
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
Set<String> syncSet = Collections.synchronizedSet(new HashSet<>());
Map<String, String> syncMap = Collections.synchronizedMap(new HashMap<>());

// IMPORTANT: Manual synchronization required for iteration
synchronized (syncList) {
    for (String item : syncList) {
        // Safe iteration
    }
}
```

---

## 10. Common Interview Questions

### 10.1 Core Concepts

**Q1: What is the difference between Collection and Collections?**
> `Collection` is an interface that is the root of the collection hierarchy. `Collections` is a utility class containing static methods for operating on collections (sort, shuffle, reverse, etc.).

**Q2: Why doesn't Map extend Collection interface?**
> Map stores key-value pairs while Collection stores single elements. Map's operations like `put(key, value)` don't fit Collection's contract of `add(element)`. However, Map provides collection views via `keySet()`, `values()`, and `entrySet()`.

**Q3: What is the difference between fail-fast and fail-safe iterators?**
> - **Fail-fast**: Throws `ConcurrentModificationException` if collection is modified during iteration. Used by ArrayList, HashMap, etc. Checks `modCount`.
> - **Fail-safe**: Works on a clone/snapshot, doesn't throw exception. Used by CopyOnWriteArrayList, ConcurrentHashMap. May not reflect latest changes.

**Q4: Explain the hashCode() and equals() contract.**
> 1. If `equals()` returns true, `hashCode()` must return same value
> 2. If `hashCode()` returns same value, `equals()` may return true or false
> 3. If `equals()` returns false, `hashCode()` may return same or different value
> 4. `hashCode()` must be consistent during object's lifetime (if equals fields don't change)

### 10.2 HashMap Questions

**Q5: How does HashMap handle collisions?**
> Java 7: Linked list at each bucket (O(n) worst case).
> Java 8+: Linked list until 8 elements, then converts to Red-Black tree (O(log n) worst case). Requires table size ≥ 64 for treeification.

**Q6: What happens when HashMap is resized?**
> When load factor (0.75) is exceeded:
> 1. New array with 2x capacity is created
> 2. All entries are rehashed and redistributed
> 3. In Java 8+, tree nodes may be split or untreeified
> 4. This is O(n) operation

**Q7: Why is the initial capacity a power of 2 in HashMap?**
> The bucket index is calculated as `(n-1) & hash` where n is capacity. When n is power of 2, this is equivalent to `hash % n` but faster (bitwise AND vs modulo). Also ensures even distribution across buckets.

**Q8: Can we use a custom object as HashMap key?**
> Yes, but the class must properly override `hashCode()` and `equals()`. The fields used in these methods should be immutable, or the object becomes "lost" in the map if modified.

### 10.3 Performance Questions

**Q9: When would you choose TreeMap over HashMap?**
> - Need sorted keys
> - Need range operations (headMap, tailMap, subMap)
> - Need navigation methods (firstKey, lastKey, floorKey, ceilingKey)
> - Can accept O(log n) vs O(1) performance

**Q10: Why is ArrayList faster than LinkedList for most operations?**
> - Cache locality: ArrayList uses contiguous memory, better CPU cache utilization
> - No node overhead: LinkedList creates Node objects for each element
> - Random access: ArrayList is O(1), LinkedList is O(n)
> - Even for insertions in middle, ArrayList's System.arraycopy is often faster than LinkedList's traversal

### 10.4 Concurrency Questions

**Q11: Difference between synchronized collections and concurrent collections?**
> | Aspect | Synchronized | Concurrent |
> |--------|--------------|------------|
> | Lock scope | Entire collection | Fine-grained |
> | Performance | Lower (single lock) | Higher (multiple locks) |
> | Iteration | Requires manual sync | Safe without sync |
> | Null | Depends on backing | Usually prohibited |

**Q12: Why ConcurrentHashMap doesn't allow null keys/values?**
> Ambiguity issue: If `get(key)` returns null, you can't distinguish between:
> 1. Key doesn't exist
> 2. Key exists with null value
> In concurrent context, this ambiguity can cause subtle bugs. `containsKey()` check followed by `get()` isn't atomic.

### 10.5 Java 8+ Questions

**Q13: Difference between Collectors.toList() and Stream.toList()?**
> | Aspect | Collectors.toList() | Stream.toList() |
> |--------|---------------------|-----------------|
> | Since | Java 8 | Java 16 |
> | Mutability | Mutable (ArrayList) | Unmodifiable |
> | Null | Allows | Allows |

**Q14: Explain the merge() method in Map.**
```java
// Signature: V merge(K key, V value, BiFunction<V,V,V> remapping)

// If key absent: puts (key, value)
// If key present: puts (key, remapping.apply(oldValue, value))
// If remapping returns null: removes entry

Map<String, Integer> map = new HashMap<>();
map.merge("a", 1, Integer::sum); // a=1
map.merge("a", 1, Integer::sum); // a=2
map.merge("a", 1, (old, new_) -> null); // removes "a"
```

### 10.6 Scenario-Based Questions

**Q15: Design an LRU Cache using Java Collections.**
```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true); // true = access order
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;
    }
}
```

**Q16: How to make a collection read-only?**
```java
// Java 8 and earlier
List<String> readOnly = Collections.unmodifiableList(list);

// Java 9+
List<String> immutable = List.of("a", "b", "c");

// Java 10+
List<String> copy = List.copyOf(existingList);
```

**Q17: How to sort a Map by values?**
```java
Map<String, Integer> map = Map.of("a", 3, "b", 1, "c", 2);

// Sort by values ascending
LinkedHashMap<String, Integer> sorted = map.entrySet().stream()
    .sorted(Map.Entry.comparingByValue())
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        (e1, e2) -> e1,
        LinkedHashMap::new
    ));
```

---

## 11. Performance Comparison Charts

### 11.1 List Operations

| Operation | ArrayList | LinkedList | CopyOnWriteArrayList |
|-----------|-----------|------------|---------------------|
| get(index) | O(1) | O(n) | O(1) |
| add(E) | O(1)* | O(1) | O(n) |
| add(index, E) | O(n) | O(n) | O(n) |
| remove(index) | O(n) | O(n) | O(n) |
| contains | O(n) | O(n) | O(n) |
| iterator.remove | O(n) | O(1) | O(n) |

*Amortized O(1), O(n) when resizing

### 11.2 Set Operations

| Operation | HashSet | LinkedHashSet | TreeSet | EnumSet |
|-----------|---------|---------------|---------|---------|
| add | O(1) | O(1) | O(log n) | O(1) |
| remove | O(1) | O(1) | O(log n) | O(1) |
| contains | O(1) | O(1) | O(log n) | O(1) |
| next | O(h/n)* | O(1) | O(log n) | O(1) |

*h = table capacity

### 11.3 Map Operations

| Operation | HashMap | LinkedHashMap | TreeMap | ConcurrentHashMap |
|-----------|---------|---------------|---------|-------------------|
| get | O(1) | O(1) | O(log n) | O(1) |
| put | O(1) | O(1) | O(log n) | O(1) |
| remove | O(1) | O(1) | O(log n) | O(1) |
| containsKey | O(1) | O(1) | O(log n) | O(1) |

### 11.4 Queue Operations

| Operation | PriorityQueue | ArrayDeque | LinkedList |
|-----------|---------------|------------|------------|
| offer | O(log n) | O(1)* | O(1) |
| poll | O(log n) | O(1) | O(1) |
| peek | O(1) | O(1) | O(1) |
| size | O(1) | O(1) | O(1) |

*Amortized

---

## 12. Best Practices

### 12.1 Choosing the Right Collection

```
Need ordered duplicates?
  ├─ Yes → ArrayList (most cases)
  │         LinkedList (frequent insertions at ends)
  └─ No → Need key-value pairs?
           ├─ Yes → Need sorting?
           │         ├─ Yes → TreeMap
           │         └─ No → HashMap (most cases)
           │                  LinkedHashMap (maintain order)
           └─ No → Need sorting?
                    ├─ Yes → TreeSet
                    └─ No → HashSet (most cases)
                             LinkedHashSet (maintain order)
```

### 12.2 Performance Tips

```java
// 1. Specify initial capacity for known sizes
List<String> list = new ArrayList<>(1000);
Map<String, Integer> map = new HashMap<>(1000);

// 2. Use appropriate collection type
// Bad: Using ArrayList for frequent contains() checks
// Good: Use HashSet for frequent lookups
Set<String> lookupSet = new HashSet<>(list);

// 3. Prefer isEmpty() over size() == 0
if (collection.isEmpty()) { } // Better

// 4. Use entrySet() when iterating over Map
for (Map.Entry<K, V> entry : map.entrySet()) {
    // Both key and value without extra lookup
}

// 5. Use computeIfAbsent for multimap pattern
Map<String, List<String>> multimap = new HashMap<>();
multimap.computeIfAbsent("key", k -> new ArrayList<>()).add("value");

// 6. Use EnumSet/EnumMap for enum keys
EnumSet<Day> workDays = EnumSet.range(Day.MONDAY, Day.FRIDAY);
EnumMap<Status, Handler> handlers = new EnumMap<>(Status.class);
```

### 12.3 Thread Safety Guidelines

```java
// 1. Prefer concurrent collections over synchronized wrappers
// Bad
Map<String, String> syncMap = Collections.synchronizedMap(new HashMap<>());

// Good
ConcurrentHashMap<String, String> concMap = new ConcurrentHashMap<>();

// 2. Use CopyOnWriteArrayList for read-heavy scenarios
List<Listener> listeners = new CopyOnWriteArrayList<>();

// 3. Use BlockingQueue for producer-consumer
BlockingQueue<Task> queue = new LinkedBlockingQueue<>(100);

// 4. Prefer immutable collections when possible
List<String> config = List.of("a", "b", "c");
```

### 12.4 Common Pitfalls to Avoid

```java
// 1. Modifying collection during iteration
// Bad - ConcurrentModificationException
for (String item : list) {
    if (condition) list.remove(item);
}

// Good - Use removeIf
list.removeIf(item -> condition);

// Good - Use iterator
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (condition) it.remove();
}

// 2. Using mutable objects as Map keys
// Bad
Map<List<String>, String> map = new HashMap<>();
List<String> key = new ArrayList<>();
map.put(key, "value");
key.add("item"); // Key is now "lost"!

// Good - Use immutable keys
Map<String, String> map = new HashMap<>();

// 3. Ignoring return values
// Bad
map.putIfAbsent(key, value); // Ignoring return

// Good
String existing = map.putIfAbsent(key, value);

// 4. Boxing overhead in primitive scenarios
// Bad
List<Integer> numbers = new ArrayList<>();

// Good - Use specialized libraries like Eclipse Collections, Trove
// IntArrayList numbers = new IntArrayList();
```

---

## Quick Reference Card

### Collection Selection Guide

| Need | Best Choice |
|------|-------------|
| General-purpose list | ArrayList |
| Frequent add/remove at ends | ArrayDeque |
| Fast lookup | HashSet / HashMap |
| Sorted data | TreeSet / TreeMap |
| Maintain insertion order | LinkedHashSet / LinkedHashMap |
| Thread-safe map | ConcurrentHashMap |
| Read-heavy concurrent list | CopyOnWriteArrayList |
| Producer-consumer queue | LinkedBlockingQueue |
| Priority ordering | PriorityQueue |
| Enum keys/elements | EnumMap / EnumSet |
| Weak references | WeakHashMap |
| LRU cache | LinkedHashMap (access order) |

### Java Version Features

| Version | Key Collection Features |
|---------|------------------------|
| Java 8 | forEach, removeIf, replaceAll, Stream API, Map compute methods |
| Java 9 | List.of(), Set.of(), Map.of(), Map.ofEntries() |
| Java 10 | var, List.copyOf(), Set.copyOf(), Map.copyOf() |
| Java 11 | Collection.toArray(IntFunction) |
| Java 12 | Collectors.teeing() |
| Java 16 | Stream.toList(), mapMulti() |
| Java 21 | SequencedCollection (Preview) |

---

*Last Updated: 2024 | Covers Java 8 - Java 17*

# How HashMap Works Internally in Java - Complete Guide

## Introduction

HashMap is one of the most important and frequently used data structures in Java. It stores data in **key-value pairs** using a technique called **hashing** to achieve near-constant time complexity for basic operations.

```java
HashMap<String, Integer> map = new HashMap<>();
map.put("John", 25);
map.put("Jane", 30);

int age = map.get("John");  // 25
```

---

## Internal Data Structure

### The Bucket Array

HashMap internally uses an **array of Nodes** (also called buckets).

```java
// Actual internal field in HashMap
transient Node<K,V>[] table;
```

### Node Structure

Each Node contains four fields:

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;       // Hash value of key (cached)
    final K key;          // The key
    V value;              // The value
    Node<K,V> next;       // Reference to next node (for collision handling)
    
    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }
}
```

### TreeNode Structure (Java 8+)

When collisions exceed threshold, nodes convert to TreeNodes:

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;   // Parent node
    TreeNode<K,V> left;     // Left child
    TreeNode<K,V> right;    // Right child
    TreeNode<K,V> prev;     // For unlinking
    boolean red;            // Red-Black tree color
}
```

### Visual Representation

```
Index    Bucket Array (table)
─────    ────────────────────────────────────────────────────────
  0      [ ] → null
  1      [ ] → Node("John", 25) → null
  2      [ ] → Node("Jane", 30) → Node("Bob", 35) → null  ← Collision!
  3      [ ] → null
  4      [ ] → Node("Alice", 28) → null
  5      [ ] → null
  ...
  15     [ ] → null
```

---

## Key Constants

```java
// Default initial capacity - must be power of 2
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;  // 16

// Maximum capacity
static final int MAXIMUM_CAPACITY = 1 << 30;  // ~1 billion

// Default load factor
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// Threshold for converting linked list to tree
static final int TREEIFY_THRESHOLD = 8;

// Threshold for converting tree back to linked list
static final int UNTREEIFY_THRESHOLD = 6;

// Minimum table capacity for treeification
static final int MIN_TREEIFY_CAPACITY = 64;
```

---

## Hash Function

### How Hashing Works

HashMap uses a special hash function to spread keys uniformly across buckets:

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

### Why XOR with Upper Bits?

The `^ (h >>> 16)` mixes the higher bits into lower bits. This helps when the table size is small (power of 2), ensuring better distribution.

```
Original hashCode:     1010 1100 0011 1111 0000 1111 0101 1010
Upper 16 bits:         0000 0000 0000 0000 1010 1100 0011 1111
                       ─────────────────────────────────────────
XOR Result:            1010 1100 0011 1111 1010 0011 0110 0101
```

### Index Calculation

```java
int index = (n - 1) & hash;  // n = table length (always power of 2)
```

This is equivalent to `hash % n` but faster since it uses bitwise AND.

```
Example:
hash = 2314
n = 16 (table length)

index = (16 - 1) & 2314
      = 15 & 2314
      = 0000 1111 & 1001 0000 1010
      = 0000 1010
      = 10
```

---

## How put() Works

### Step-by-Step Process

```java
map.put("John", 25);
```

**Step 1: Calculate Hash**
```java
int hash = hash("John");  // Let's say 2314
```

**Step 2: Calculate Bucket Index**
```java
int index = (n - 1) & hash;  // Let's say 10
```

**Step 3: Check Bucket and Store**
- If bucket is empty → Create new Node and store
- If bucket has nodes → Handle collision

### Detailed put() Algorithm

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent, boolean evict) {
    Node<K,V>[] tab; 
    Node<K,V> p; 
    int n, i;
    
    // Step 1: Initialize table if empty
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    
    // Step 2: Calculate index and check if bucket is empty
    if ((p = tab[i = (n - 1) & hash]) == null)
        // Bucket empty - create new node
        tab[i] = newNode(hash, key, value, null);
    else {
        // Bucket not empty - handle collision
        Node<K,V> e; K k;
        
        // Step 3a: Check if first node matches key
        if (p.hash == hash && 
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        
        // Step 3b: Check if it's a tree node
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        
        // Step 3c: Traverse linked list
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    // Key not found - add new node at end
                    p.next = newNode(hash, key, value, null);
                    
                    // Convert to tree if threshold exceeded
                    if (binCount >= TREEIFY_THRESHOLD - 1)
                        treeifyBin(tab, hash);
                    break;
                }
                // Key found - break to update value
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        
        // Step 4: Update value if key exists
        if (e != null) {
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            return oldValue;
        }
    }
    
    // Step 5: Increment size and check for resize
    ++modCount;
    if (++size > threshold)
        resize();
    
    return null;
}
```

### put() Flow Diagram

```
put(key, value)
       │
       ▼
┌─────────────────────────┐
│ Calculate hash          │
│ hash = hash(key)        │
└─────────────────────────┘
       │
       ▼
┌─────────────────────────┐
│ Calculate index         │
│ i = (n-1) & hash        │
└─────────────────────────┘
       │
       ▼
┌─────────────────────────┐
│ Is table[i] == null?    │
└─────────────────────────┘
       │
   ┌───┴───┐
  Yes      No
   │       │
   ▼       ▼
Create   ┌─────────────────────────┐
new      │ First node key matches? │
Node     └─────────────────────────┘
   │            │
   │        ┌───┴───┐
   │       Yes      No
   │        │       │
   │        ▼       ▼
   │     Update  ┌─────────────────┐
   │     value   │ Is it TreeNode? │
   │             └─────────────────┘
   │                    │
   │                ┌───┴───┐
   │               Yes      No
   │                │       │
   │                ▼       ▼
   │           putTreeVal  Traverse
   │                       linked list
   │                          │
   │                    ┌─────┴─────┐
   │                    │           │
   │                Key found    Key not found
   │                    │           │
   │                    ▼           ▼
   │                Update      Add new node
   │                value       (treeify if needed)
   │
   ▼
┌─────────────────────────┐
│ size > threshold?       │
│ If yes → resize()       │
└─────────────────────────┘
```

---

## How get() Works

### Step-by-Step Process

```java
map.get("John");
```

**Step 1:** Calculate hash of key
**Step 2:** Calculate bucket index
**Step 3:** Navigate to bucket
**Step 4:** Search for key in chain/tree
**Step 5:** Return value if found, null otherwise

### Detailed get() Algorithm

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; 
    Node<K,V> first, e; 
    int n; K k;
    
    // Check if table exists and bucket is not empty
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        
        // Check first node
        if (first.hash == hash &&
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        
        // Check rest of chain/tree
        if ((e = first.next) != null) {
            // If tree, use tree search
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            
            // Traverse linked list
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

### get() Flow Diagram

```
get(key)
       │
       ▼
┌─────────────────────────┐
│ Calculate hash          │
│ hash = hash(key)        │
└─────────────────────────┘
       │
       ▼
┌─────────────────────────┐
│ Calculate index         │
│ i = (n-1) & hash        │
└─────────────────────────┘
       │
       ▼
┌─────────────────────────┐
│ Is table[i] == null?    │
└─────────────────────────┘
       │
   ┌───┴───┐
  Yes      No
   │       │
   ▼       ▼
Return   ┌─────────────────────────┐
null     │ First node key matches? │
         └─────────────────────────┘
                │
            ┌───┴───┐
           Yes      No
            │       │
            ▼       ▼
         Return  ┌─────────────────┐
         value   │ Is it TreeNode? │
                 └─────────────────┘
                        │
                    ┌───┴───┐
                   Yes      No
                    │       │
                    ▼       ▼
               getTreeNode  Traverse
                    │       linked list
                    │           │
                    │     ┌─────┴─────┐
                    │     │           │
                    │  Key found   Key not found
                    │     │           │
                    │     ▼           ▼
                    │  Return      Return
                    │  value       null
                    │
                    ▼
              Return node or null
```

---

## Collision Handling

### What is a Collision?

A collision occurs when two different keys produce the same bucket index.

```java
// Example: Both keys might map to index 5
map.put("Aa", 1);   // hash("Aa") & 15 = 5
map.put("BB", 2);   // hash("BB") & 15 = 5  ← Collision!
```

### Before Java 8: Linked List Only

All collisions were handled using linked lists:

```
Bucket[5] → Node("Aa", 1) → Node("BB", 2) → Node("CC", 3) → null
```

**Problem:** If many collisions occur, searching becomes O(n).

### Java 8+: Linked List + Red-Black Tree

When a bucket exceeds **TREEIFY_THRESHOLD (8)** nodes, it converts to a Red-Black Tree.

```
Before Treeification (≤ 8 nodes):
────────────────────────────────
Bucket[5] → Node → Node → Node → Node → Node → Node → Node → Node → null
            (1)    (2)    (3)    (4)    (5)    (6)    (7)    (8)

After Treeification (> 8 nodes):
────────────────────────────────
Bucket[5] →         TreeNode(4)
                   /           \
            TreeNode(2)      TreeNode(6)
            /       \        /       \
       TreeNode(1) TreeNode(3) TreeNode(5) TreeNode(7)
                                             \
                                          TreeNode(8)
```

### Treeify Conditions

Conversion to tree happens when:
1. Bucket has more than 8 nodes (TREEIFY_THRESHOLD)
2. AND table capacity is at least 64 (MIN_TREEIFY_CAPACITY)

If capacity < 64, HashMap resizes instead of treeifying.

### Untreeify

When tree shrinks to 6 or fewer nodes (UNTREEIFY_THRESHOLD), it converts back to linked list.

---

## Resizing (Rehashing)

### When Does Resize Happen?

HashMap resizes when:
```
size > threshold

where:
threshold = capacity × loadFactor
          = 16 × 0.75
          = 12 (for default settings)
```

### Resize Process

1. Create new array with **double the capacity**
2. **Rehash** all existing entries (recalculate indexes)
3. Move entries to new positions

### Resize Algorithm

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // Double the capacity
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1;  // Double threshold
    }
    else if (oldThr > 0)
        newCap = oldThr;
    else {
        // First initialization
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    
    // Create new table
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    
    // Rehash all nodes
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                
                if (e.next == null)
                    // Single node - direct placement
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    // Split tree
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else {
                    // Linked list - split into two lists
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            // Stays at same index
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            // Moves to index + oldCap
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

### Resize Visualization

```
Before resize (capacity = 16, threshold = 12):
──────────────────────────────────────────────
Index:  0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
       [ ] [ ] [A] [ ] [ ] [B] [ ] [C] [ ] [ ] [D] [ ] [E] [ ] [ ] [F]

Size = 13 > 12 (threshold) → Resize triggered!


After resize (capacity = 32, threshold = 24):
──────────────────────────────────────────────
Index:  0   1   2   3   4   5   ... 18  ... 21  ... 26  ... 31
       [ ] [ ] [A] [ ] [ ] [B]     [C]     [D]     [E]     [F]
                                   ↑
                           Nodes moved to new positions
                           based on recalculated hash
```

### Why Power of 2?

Capacity is always a power of 2 because:
1. Makes `(n-1) & hash` work as modulo operation
2. During resize, nodes either stay or move by exactly `oldCap`
3. Enables efficient bit manipulation

---

## Null Key Handling

HashMap allows **one null key** (unlike Hashtable).

```java
map.put(null, 100);    // Allowed
map.get(null);         // Returns 100
```

### How Null Key Works

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

- Null key always has hash = 0
- Always stored at index 0 (since `0 & (n-1) = 0`)

---

## Time Complexity

| Operation | Best Case | Average Case | Worst Case (Before Java 8) | Worst Case (Java 8+) |
|-----------|-----------|--------------|---------------------------|---------------------|
| put() | O(1) | O(1) | O(n) | O(log n) |
| get() | O(1) | O(1) | O(n) | O(log n) |
| remove() | O(1) | O(1) | O(n) | O(log n) |
| containsKey() | O(1) | O(1) | O(n) | O(log n) |
| containsValue() | O(n) | O(n) | O(n) | O(n) |

### Space Complexity

- O(n) where n is the number of entries
- Additional overhead for array capacity and node objects

---

## hashCode() and equals() Contract

For HashMap to work correctly, keys must properly implement both methods.

### The Contract

1. **Consistency:** Multiple calls to `hashCode()` must return same value (if object hasn't changed)
2. **equals-hashCode relationship:**
   - If `a.equals(b)` is `true` → `a.hashCode() == b.hashCode()` **must** be `true`
   - If `a.hashCode() != b.hashCode()` → `a.equals(b)` **must** be `false`
   - If `a.hashCode() == b.hashCode()` → `a.equals(b)` **may or may not** be `true`

### Correct Implementation

```java
public class Employee {
    private int id;
    private String name;
    private String department;
    
    // Constructor, getters, setters...
    
    @Override
    public int hashCode() {
        return Objects.hash(id, name, department);
    }
    
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Employee other = (Employee) obj;
        return id == other.id && 
               Objects.equals(name, other.name) && 
               Objects.equals(department, other.department);
    }
}
```

### What Happens with Bad Implementation?

```java
// ❌ BAD: hashCode not overridden
public class BadKey {
    private int value;
    
    @Override
    public boolean equals(Object obj) {
        if (obj instanceof BadKey) {
            return this.value == ((BadKey) obj).value;
        }
        return false;
    }
    
    // hashCode() not overridden - uses Object's default (memory address)
}

// Problem:
BadKey key1 = new BadKey(1);
BadKey key2 = new BadKey(1);  // Same value as key1

map.put(key1, "Value1");
map.get(key2);  // Returns null! Different hashCode means different bucket
```

---

## Complete Example: Step-by-Step Walkthrough

```java
HashMap<String, Integer> map = new HashMap<>();  // capacity=16, loadFactor=0.75
```

### Adding Entries

```
Step 1: put("Apple", 10)
════════════════════════
hash("Apple") = 63476538
index = 63476538 & 15 = 10
Bucket[10] is empty → Create Node

table:
[0]  → null
[1]  → null
...
[10] → Node("Apple", 10) → null
...
[15] → null

size = 1


Step 2: put("Banana", 20)
═════════════════════════
hash("Banana") = 1982479237
index = 1982479237 & 15 = 5
Bucket[5] is empty → Create Node

table:
[0]  → null
...
[5]  → Node("Banana", 20) → null
...
[10] → Node("Apple", 10) → null
...
[15] → null

size = 2


Step 3: put("Cherry", 30) - Collision!
══════════════════════════════════════
hash("Cherry") = 2065445782
index = 2065445782 & 15 = 10  (Same as Apple!)

Bucket[10] not empty:
- Check first node: "Apple" != "Cherry"
- No more nodes → Add to end of chain

table:
[0]  → null
...
[5]  → Node("Banana", 20) → null
...
[10] → Node("Apple", 10) → Node("Cherry", 30) → null
...
[15] → null

size = 3


Step 4: put("Apple", 15) - Update existing key
══════════════════════════════════════════════
hash("Apple") = 63476538
index = 10

Bucket[10]:
- Check first node: "Apple" == "Apple" ✓
- Key found → Update value from 10 to 15

table:
[0]  → null
...
[5]  → Node("Banana", 20) → null
...
[10] → Node("Apple", 15) → Node("Cherry", 30) → null  ← Value updated!
...
[15] → null

size = 3 (unchanged - no new entry)
```

### Retrieving Entries

```
Step 5: get("Cherry")
═════════════════════
hash("Cherry") = 2065445782
index = 10

Go to Bucket[10]:
- Node 1: "Apple" != "Cherry" → Continue
- Node 2: "Cherry" == "Cherry" ✓ → Found!

Return 30


Step 6: get("Mango")
════════════════════
hash("Mango") = 74219037
index = 74219037 & 15 = 13

Go to Bucket[13]:
- Bucket is null

Return null
```

---

## HashMap vs Other Map Implementations

| Feature | HashMap | LinkedHashMap | TreeMap | Hashtable | ConcurrentHashMap |
|---------|---------|---------------|---------|-----------|-------------------|
| Order | No order | Insertion order | Sorted order | No order | No order |
| Null keys | 1 allowed | 1 allowed | Not allowed | Not allowed | Not allowed |
| Null values | Allowed | Allowed | Allowed | Not allowed | Not allowed |
| Thread-safe | No | No | No | Yes | Yes |
| Performance | O(1) | O(1) | O(log n) | O(1) | O(1) |
| Synchronization | None | None | None | Full sync | Segment-level |
| Introduced | Java 1.2 | Java 1.4 | Java 1.2 | Java 1.0 | Java 1.5 |

---

## Best Practices

### 1. Set Initial Capacity When Size is Known

```java
// ❌ Bad - causes multiple resize operations
Map<String, Integer> map = new HashMap<>();
for (int i = 0; i < 10000; i++) {
    map.put("key" + i, i);
}

// ✅ Good - single allocation
int expectedSize = 10000;
int initialCapacity = (int) (expectedSize / 0.75) + 1;
Map<String, Integer> map = new HashMap<>(initialCapacity);
```

### 2. Use Immutable Keys

```java
// ✅ Good - String is immutable
Map<String, Integer> map = new HashMap<>();
map.put("key", 100);

// ❌ Bad - mutable keys can cause problems
Map<List<String>, Integer> map = new HashMap<>();
List<String> key = new ArrayList<>();
key.add("a");
map.put(key, 100);
key.add("b");  // Changes hashCode!
map.get(key);  // Might return null!
```

### 3. Override hashCode() and equals() Correctly

```java
public class Person {
    private String name;
    private int age;
    
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
}
```

### 4. Choose Appropriate Load Factor

```java
// Memory-sensitive (more entries per bucket, more collisions)
Map<K, V> map = new HashMap<>(capacity, 0.9f);

// Performance-sensitive (fewer collisions, more memory)
Map<K, V> map = new HashMap<>(capacity, 0.5f);

// Default is usually fine
Map<K, V> map = new HashMap<>();  // 0.75 load factor
```

### 5. Use computeIfAbsent() for Lazy Initialization

```java
// ❌ Verbose way
Map<String, List<Integer>> map = new HashMap<>();
if (!map.containsKey("key")) {
    map.put("key", new ArrayList<>());
}
map.get("key").add(1);

// ✅ Better way
map.computeIfAbsent("key", k -> new ArrayList<>()).add(1);
```

### 6. Use getOrDefault() for Safe Retrieval

```java
// ❌ Verbose way
Integer value = map.get("key");
if (value == null) {
    value = 0;
}

// ✅ Better way
Integer value = map.getOrDefault("key", 0);
```

---

## Common Interview Questions

### Q1: How does HashMap handle collisions?

**Answer:** Before Java 8, collisions were handled using a linked list. From Java 8 onwards, when a bucket has more than 8 nodes and table capacity is at least 64, the linked list converts to a Red-Black tree for O(log n) search performance.

### Q2: Why is capacity always a power of 2?

**Answer:** 
1. Enables using bitwise AND (`hash & (n-1)`) instead of modulo for index calculation
2. During resize, nodes either stay at same index or move by exactly `oldCap`
3. More efficient bit manipulation

### Q3: What happens if hashCode() returns constant value?

**Answer:** All entries would go to the same bucket, degrading performance to O(n) for linked list or O(log n) for tree (Java 8+).

### Q4: Why is HashMap not thread-safe?

**Answer:** HashMap doesn't use synchronization for performance reasons. Concurrent modifications can cause infinite loops (during resize) or data loss. Use `ConcurrentHashMap` or `Collections.synchronizedMap()` for thread safety.

### Q5: What is the difference between HashMap and Hashtable?

**Answer:**
- HashMap allows null keys/values; Hashtable doesn't
- HashMap is not synchronized; Hashtable is synchronized
- HashMap is faster; Hashtable is slower due to synchronization
- HashMap uses fail-fast iterator; Hashtable uses enumerator

---

## Summary

| Aspect | Details |
|--------|---------|
| **Internal Structure** | Array of Nodes (buckets) + Linked List/Tree |
| **Default Capacity** | 16 |
| **Default Load Factor** | 0.75 |
| **Resize Trigger** | When size > capacity × loadFactor |
| **Collision Handling** | LinkedList (≤8 nodes) → Red-Black Tree (>8 nodes) |
| **Treeify Threshold** | 8 nodes |
| **Untreeify Threshold** | 6 nodes |
| **Null Keys** | One allowed (stored at index 0) |
| **Null Values** | Multiple allowed |
| **Time Complexity** | O(1) average, O(log n) worst case |
| **Space Complexity** | O(n) |
| **Thread-Safe** | No |
| **Key Requirements** | Proper hashCode() and equals() implementation |

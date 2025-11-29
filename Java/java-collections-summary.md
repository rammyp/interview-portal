# Java Collections Cheat Sheet

## List Implementations

| ArrayList | LinkedList |
|-----------|------------|
| Uses dynamic array internally | Uses doubly linked list |
| Fast random access O(1) | Slow random access O(n) |
| Slow insertion/deletion in middle | Fast insertion/deletion anywhere |
| Both maintain insertion order | Both maintain insertion order |

---

## Set Implementations

| HashSet | TreeSet |
|---------|---------|
| No ordering | Sorted order (natural or comparator) |
| Allows one null | Does not allow null |
| O(1) for add, remove, contains | O(log n) for operations |
| Uses hashing internally | Uses Red-Black tree internally |

---

## Map Implementations

| HashMap | Hashtable | LinkedHashMap | TreeMap |
|---------|-----------|---------------|---------|
| Not synchronized | Synchronized | Not synchronized | Not synchronized |
| Allows one null key | No null keys/values | Allows one null key | No null keys |
| No order | No order | Insertion order | Sorted order |
| Fast (O(1)) | Slower (due to sync) | Fast (O(1)) | O(log n) |

---

## Key Concepts

### Searching Performance
- **HashSet/HashMap**: O(1) - computes hash and jumps directly
- **ArrayList**: O(n) - must iterate through elements

### Null Handling
- **HashMap**: One null key, multiple null values allowed
- **Hashtable**: No nulls allowed
- **HashSet**: One null allowed
- **TreeSet/TreeMap**: No nulls (needs comparison)

### Ordering
- **HashMap/HashSet**: No order
- **LinkedHashMap/LinkedHashSet**: Insertion order
- **TreeMap/TreeSet**: Sorted order

---

## ConcurrentModificationException

Thrown when modifying a collection while iterating with an Iterator.

**Wrong:**
```java
for (String s : list) {
    list.remove(s);  // Throws exception
}
```

**Correct:**
```java
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (condition) {
        it.remove();  // Safe removal
    }
}
```

---

## Comparable vs Comparator

| Comparable | Comparator |
|------------|------------|
| `java.lang` package | `java.util` package |
| `compareTo(Object o)` | `compare(Object o1, Object o2)` |
| Natural ordering | Custom ordering |
| Modifies original class | Separate class |
| Single sort sequence | Multiple sort sequences |

---

## Quick Reference

- **Random access needed?** → ArrayList
- **Frequent insertions/deletions?** → LinkedList
- **No duplicates, no order?** → HashSet
- **No duplicates, sorted?** → TreeSet
- **Key-value, no order?** → HashMap
- **Key-value, insertion order?** → LinkedHashMap
- **Key-value, sorted?** → TreeMap
- **Thread-safe map?** → ConcurrentHashMap (prefer over Hashtable)

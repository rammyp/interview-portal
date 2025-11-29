# Java Immutable Classes - Complete Guide

## Introduction

An **immutable class** is a class whose objects cannot be modified after creation. Once an object is created, its state remains constant throughout its lifetime.

---

## Why Use Immutable Classes?

| Benefit | Description |
|---------|-------------|
| **Thread-safe** | No synchronization needed, safe for concurrent access |
| **Safe to share** | Can be freely passed around without defensive copying |
| **Good hash keys** | HashCode never changes, safe for HashMap/HashSet |
| **Simple to understand** | No unexpected state changes |
| **Cacheable** | Can be reused and cached safely |
| **Failure atomicity** | No partial state if exception occurs |

---

## Rules for Creating Immutable Classes

1. **Declare the class as `final`** - Prevents subclassing
2. **Make all fields `private` and `final`** - No direct access, assigned once
3. **No setter methods** - No way to modify state
4. **Initialize all fields via constructor**
5. **Return copies of mutable objects** - Deep copy in getters
6. **Don't allow mutable objects to be modified** - Defensive copying

---

## Basic Immutable Class

### Simple Example

```java
public final class Person {
    
    private final String name;
    private final int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() {
        return name;
    }
    
    public int getAge() {
        return age;
    }
    
    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }
    
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

### Usage

```java
Person person = new Person("John", 30);
System.out.println(person.getName());  // John
System.out.println(person.getAge());   // 30

// Cannot modify - no setters available!
// person.setName("Jane");  // Compile error!
```

---

## Handling Mutable Fields

When your class contains mutable objects (like `Date`, `List`, `Array`), you need **defensive copying**.

### The Problem: Mutable Field References

```java
// ❌ WRONG - Not Truly Immutable
public final class Employee {
    
    private final String name;
    private final Date joinDate;
    private final List<String> skills;
    
    public Employee(String name, Date joinDate, List<String> skills) {
        this.name = name;
        this.joinDate = joinDate;        // Problem: stores reference
        this.skills = skills;            // Problem: stores reference
    }
    
    public Date getJoinDate() {
        return joinDate;                 // Problem: returns reference
    }
    
    public List<String> getSkills() {
        return skills;                   // Problem: returns reference
    }
}
```

### Why This Fails

```java
Date date = new Date();
List<String> skills = new ArrayList<>();
skills.add("Java");

Employee emp = new Employee("John", date, skills);

// External modifications affect internal state!
date.setTime(0);                         // Modifies emp's joinDate!
skills.add("Python");                    // Modifies emp's skills!
emp.getJoinDate().setTime(0);            // Modifies internal state!
emp.getSkills().add("SQL");              // Modifies internal state!
```

### The Solution: Defensive Copying

```java
// ✅ CORRECT - Truly Immutable
import java.util.ArrayList;
import java.util.Collections;
import java.util.Date;
import java.util.List;

public final class Employee {
    
    private final String name;
    private final Date joinDate;
    private final List<String> skills;
    
    public Employee(String name, Date joinDate, List<String> skills) {
        this.name = name;
        // Defensive copy in constructor
        this.joinDate = new Date(joinDate.getTime());
        this.skills = new ArrayList<>(skills);
    }
    
    public String getName() {
        return name;
    }
    
    public Date getJoinDate() {
        // Return copy, not original
        return new Date(joinDate.getTime());
    }
    
    public List<String> getSkills() {
        // Return unmodifiable copy
        return Collections.unmodifiableList(skills);
    }
    
    @Override
    public String toString() {
        return "Employee{name='" + name + "', joinDate=" + joinDate + 
               ", skills=" + skills + "}";
    }
}
```

### Now It's Safe

```java
Date date = new Date();
List<String> skills = new ArrayList<>();
skills.add("Java");

Employee emp = new Employee("John", date, skills);

// External modifications don't affect internal state
date.setTime(0);                         // emp's joinDate unchanged
skills.add("Python");                    // emp's skills unchanged
emp.getJoinDate().setTime(0);            // emp's internal date unchanged
emp.getSkills().add("SQL");              // Throws UnsupportedOperationException!
```

---

## Common Mutable Types and How to Copy Them

| Mutable Type | Defensive Copy Method |
|--------------|----------------------|
| `Date` | `new Date(date.getTime())` |
| `ArrayList` | `new ArrayList<>(list)` |
| `HashSet` | `new HashSet<>(set)` |
| `HashMap` | `new HashMap<>(map)` |
| `Array` | `array.clone()` or `Arrays.copyOf(array, array.length)` |
| Custom Object | Implement and use `copy()` method |

### Returning Unmodifiable Collections

| Type | Unmodifiable Wrapper |
|------|---------------------|
| `List` | `Collections.unmodifiableList(list)` |
| `Set` | `Collections.unmodifiableSet(set)` |
| `Map` | `Collections.unmodifiableMap(map)` |
| Any Collection | `List.copyOf(list)` (Java 10+) |

---

## Handling Arrays

Arrays are always mutable, so extra care is needed.

```java
public final class Scores {
    
    private final int[] scores;
    
    public Scores(int[] scores) {
        // Defensive copy in constructor
        this.scores = scores.clone();
        // Or: this.scores = Arrays.copyOf(scores, scores.length);
    }
    
    public int[] getScores() {
        // Return copy in getter
        return scores.clone();
    }
    
    public int getScore(int index) {
        // Or provide element access
        return scores[index];
    }
    
    public int getLength() {
        return scores.length;
    }
}
```

---

## Deep Copying Nested Mutable Objects

When you have nested mutable objects, you need **deep copying**.

### Mutable Nested Class

```java
// If Address is mutable, we need to copy it
public class Address {
    
    private String street;
    private String city;
    
    public Address(String street, String city) {
        this.street = street;
        this.city = city;
    }
    
    // Getters and setters
    public String getStreet() { return street; }
    public void setStreet(String street) { this.street = street; }
    public String getCity() { return city; }
    public void setCity(String city) { this.city = city; }
    
    // Copy method for defensive copying
    public Address copy() {
        return new Address(street, city);
    }
}
```

### Immutable Class with Nested Mutable Object

```java
public final class Customer {
    
    private final String name;
    private final Address address;
    private final List<Address> previousAddresses;
    
    public Customer(String name, Address address, List<Address> previousAddresses) {
        this.name = name;
        
        // Deep copy of mutable object
        this.address = address.copy();
        
        // Deep copy of list containing mutable objects
        this.previousAddresses = new ArrayList<>();
        for (Address addr : previousAddresses) {
            this.previousAddresses.add(addr.copy());
        }
    }
    
    public String getName() {
        return name;
    }
    
    public Address getAddress() {
        // Return copy
        return address.copy();
    }
    
    public List<Address> getPreviousAddresses() {
        // Return deep copy
        List<Address> copy = new ArrayList<>();
        for (Address addr : previousAddresses) {
            copy.add(addr.copy());
        }
        return copy;
    }
}
```

### Better Approach: Make Nested Class Immutable Too

```java
// Immutable Address
public final class Address {
    
    private final String street;
    private final String city;
    
    public Address(String street, String city) {
        this.street = street;
        this.city = city;
    }
    
    public String getStreet() { return street; }
    public String getCity() { return city; }
    
    // Create modified copy
    public Address withStreet(String newStreet) {
        return new Address(newStreet, this.city);
    }
    
    public Address withCity(String newCity) {
        return new Address(this.street, newCity);
    }
}

// Now Customer is simpler
public final class Customer {
    
    private final String name;
    private final Address address;
    private final List<Address> previousAddresses;
    
    public Customer(String name, Address address, List<Address> previousAddresses) {
        this.name = name;
        this.address = address;  // No copy needed - Address is immutable!
        this.previousAddresses = List.copyOf(previousAddresses);  // Java 10+
    }
    
    public String getName() { return name; }
    public Address getAddress() { return address; }  // No copy needed!
    public List<Address> getPreviousAddresses() { return previousAddresses; }
}
```

---

## Immutable Class with Builder Pattern

For immutable classes with many fields, use the Builder pattern for better readability.

```java
public final class User {
    
    private final String firstName;
    private final String lastName;
    private final String email;
    private final int age;
    private final String phone;
    private final String address;
    private final List<String> roles;
    
    private User(Builder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.email = builder.email;
        this.age = builder.age;
        this.phone = builder.phone;
        this.address = builder.address;
        this.roles = Collections.unmodifiableList(new ArrayList<>(builder.roles));
    }
    
    // Only getters - no setters
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public String getEmail() { return email; }
    public int getAge() { return age; }
    public String getPhone() { return phone; }
    public String getAddress() { return address; }
    public List<String> getRoles() { return roles; }
    
    // Static Builder class
    public static class Builder {
        
        // Required fields
        private final String firstName;
        private final String lastName;
        
        // Optional fields with defaults
        private String email = "";
        private int age = 0;
        private String phone = "";
        private String address = "";
        private List<String> roles = new ArrayList<>();
        
        public Builder(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }
        
        public Builder email(String email) {
            this.email = email;
            return this;
        }
        
        public Builder age(int age) {
            this.age = age;
            return this;
        }
        
        public Builder phone(String phone) {
            this.phone = phone;
            return this;
        }
        
        public Builder address(String address) {
            this.address = address;
            return this;
        }
        
        public Builder roles(List<String> roles) {
            this.roles = new ArrayList<>(roles);
            return this;
        }
        
        public Builder addRole(String role) {
            this.roles.add(role);
            return this;
        }
        
        public User build() {
            // Validation can be done here
            if (firstName == null || firstName.isBlank()) {
                throw new IllegalStateException("First name is required");
            }
            return new User(this);
        }
    }
    
    @Override
    public String toString() {
        return "User{firstName='" + firstName + "', lastName='" + lastName + 
               "', email='" + email + "', age=" + age + 
               ", phone='" + phone + "', address='" + address + 
               "', roles=" + roles + "}";
    }
}
```

### Usage

```java
// Fluent, readable construction
User user = new User.Builder("John", "Doe")
    .email("john@example.com")
    .age(30)
    .phone("123-456-7890")
    .addRole("ADMIN")
    .addRole("USER")
    .build();

System.out.println(user);
```

---

## "Modifying" Immutable Objects

Since immutable objects can't be changed, you create new objects with modified values.

### Using "with" Methods

```java
public final class Money {
    
    private final double amount;
    private final String currency;
    
    public Money(double amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }
    
    public double getAmount() { return amount; }
    public String getCurrency() { return currency; }
    
    // Return new object with added amount
    public Money add(double value) {
        return new Money(this.amount + value, this.currency);
    }
    
    // Return new object with subtracted amount
    public Money subtract(double value) {
        return new Money(this.amount - value, this.currency);
    }
    
    // Return new object with multiplied amount
    public Money multiply(double factor) {
        return new Money(this.amount * factor, this.currency);
    }
    
    // Return new object with different currency
    public Money withCurrency(String newCurrency) {
        return new Money(this.amount, newCurrency);
    }
    
    // Return new object with different amount
    public Money withAmount(double newAmount) {
        return new Money(newAmount, this.currency);
    }
    
    @Override
    public String toString() {
        return String.format("%.2f %s", amount, currency);
    }
}
```

### Usage

```java
Money money = new Money(100, "USD");

Money more = money.add(50);           // New object: 150 USD
Money less = money.subtract(25);      // New object: 75 USD
Money doubled = money.multiply(2);    // New object: 200 USD
Money euros = money.withCurrency("EUR");  // New object: 100 EUR

System.out.println(money);    // 100.00 USD (unchanged!)
System.out.println(more);     // 150.00 USD
System.out.println(less);     // 75.00 USD
System.out.println(doubled);  // 200.00 USD
System.out.println(euros);    // 100.00 EUR
```

### Chaining Operations

```java
Money result = new Money(100, "USD")
    .add(50)
    .multiply(2)
    .subtract(25);
    
System.out.println(result);  // 275.00 USD
```

---

## Java Records (Java 14+)

Java Records provide a concise way to create immutable data classes.

### Basic Record

```java
// This single line creates an immutable class!
public record Person(String name, int age) {}
```

This automatically generates:
- Private final fields
- All-args constructor
- Getters (without "get" prefix)
- `equals()` method
- `hashCode()` method
- `toString()` method

### Usage

```java
Person person = new Person("John", 30);

String name = person.name();  // Getter (no "get" prefix)
int age = person.age();

System.out.println(person);  // Person[name=John, age=30]
```

### Record with Validation

```java
public record Person(String name, int age) {
    
    // Compact constructor for validation
    public Person {
        if (name == null || name.isBlank()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Invalid age: " + age);
        }
        // Fields are automatically assigned after this block
    }
}
```

### Record with Custom Methods

```java
public record Rectangle(double width, double height) {
    
    // Validation
    public Rectangle {
        if (width <= 0 || height <= 0) {
            throw new IllegalArgumentException("Dimensions must be positive");
        }
    }
    
    // Custom methods
    public double area() {
        return width * height;
    }
    
    public double perimeter() {
        return 2 * (width + height);
    }
    
    // "with" method for modification
    public Rectangle withWidth(double newWidth) {
        return new Rectangle(newWidth, this.height);
    }
    
    public Rectangle withHeight(double newHeight) {
        return new Rectangle(this.width, newHeight);
    }
}
```

### Record with Mutable Fields (Needs Care!)

```java
// ❌ PROBLEM - List is mutable
public record Employee(String name, List<String> skills) {}

Employee emp = new Employee("John", new ArrayList<>(List.of("Java")));
emp.skills().add("Python");  // Modifies internal state!
```

```java
// ✅ SOLUTION - Defensive copy
public record Employee(String name, List<String> skills) {
    
    public Employee(String name, List<String> skills) {
        this.name = name;
        this.skills = List.copyOf(skills);  // Immutable copy
    }
}

Employee emp = new Employee("John", new ArrayList<>(List.of("Java")));
emp.skills().add("Python");  // Throws UnsupportedOperationException!
```

### Record vs Traditional Class Comparison

| Feature | Traditional Class | Record |
|---------|------------------|--------|
| Boilerplate | High | Minimal |
| Fields | Manually declare | Auto-generated |
| Constructor | Manually write | Auto-generated |
| Getters | Manually write | Auto-generated |
| equals/hashCode | Manually write | Auto-generated |
| toString | Manually write | Auto-generated |
| Inheritance | Can extend classes | Cannot extend (implicitly final) |
| Additional methods | Yes | Yes |
| Mutable fields | Yes | Needs defensive copying |

---

## Immutable Classes in Java Standard Library

### Common Immutable Classes

| Class | Package | Notes |
|-------|---------|-------|
| `String` | java.lang | Most commonly used immutable class |
| `Integer` | java.lang | Wrapper for int |
| `Long` | java.lang | Wrapper for long |
| `Double` | java.lang | Wrapper for double |
| `Boolean` | java.lang | Wrapper for boolean |
| `Character` | java.lang | Wrapper for char |
| `BigInteger` | java.math | Arbitrary precision integer |
| `BigDecimal` | java.math | Arbitrary precision decimal |
| `LocalDate` | java.time | Date without time |
| `LocalTime` | java.time | Time without date |
| `LocalDateTime` | java.time | Date and time |
| `Instant` | java.time | Timestamp |
| `Duration` | java.time | Time duration |
| `Period` | java.time | Date-based period |
| `Optional` | java.util | Container for optional value |
| `Path` | java.nio.file | File path |
| `URI` | java.net | Uniform Resource Identifier |
| `URL` | java.net | Uniform Resource Locator |

### How String Demonstrates Immutability

```java
String str = "Hello";
String upper = str.toUpperCase();  // Returns NEW string

System.out.println(str);    // Hello (unchanged)
System.out.println(upper);  // HELLO (new object)

String concat = str.concat(" World");  // Returns NEW string
System.out.println(str);    // Hello (still unchanged)
System.out.println(concat); // Hello World (new object)
```

### How LocalDate Demonstrates Immutability

```java
LocalDate date = LocalDate.of(2024, 1, 15);
LocalDate nextMonth = date.plusMonths(1);  // Returns NEW date

System.out.println(date);       // 2024-01-15 (unchanged)
System.out.println(nextMonth);  // 2024-02-15 (new object)
```

---

## Validation in Immutable Classes

### Constructor Validation

```java
public final class Email {
    
    private final String address;
    
    public Email(String address) {
        // Validate before assignment
        if (address == null || address.isBlank()) {
            throw new IllegalArgumentException("Email cannot be empty");
        }
        if (!address.contains("@")) {
            throw new IllegalArgumentException("Invalid email format");
        }
        this.address = address.toLowerCase().trim();
    }
    
    public String getAddress() {
        return address;
    }
}
```

### Factory Method with Validation

```java
public final class Age {
    
    private final int value;
    
    private Age(int value) {
        this.value = value;
    }
    
    // Factory method with validation
    public static Age of(int value) {
        if (value < 0) {
            throw new IllegalArgumentException("Age cannot be negative");
        }
        if (value > 150) {
            throw new IllegalArgumentException("Age cannot exceed 150");
        }
        return new Age(value);
    }
    
    public int getValue() {
        return value;
    }
}

// Usage
Age age = Age.of(25);  // Valid
Age invalid = Age.of(-5);  // Throws exception
```

---

## Performance Considerations

### Object Creation Overhead

Immutable objects require creating new objects for every modification. This can be mitigated by:

1. **Caching common values**
```java
public final class Percentage {
    
    private static final Percentage ZERO = new Percentage(0);
    private static final Percentage HUNDRED = new Percentage(100);
    
    private final int value;
    
    private Percentage(int value) {
        this.value = value;
    }
    
    public static Percentage of(int value) {
        if (value == 0) return ZERO;
        if (value == 100) return HUNDRED;
        return new Percentage(value);
    }
}
```

2. **Using StringBuilder for string concatenation**
```java
// ❌ Inefficient - creates many String objects
String result = "";
for (int i = 0; i < 1000; i++) {
    result += i;  // Creates new String each time
}

// ✅ Efficient - uses mutable builder, then creates immutable result
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String result = sb.toString();
```

---

## Common Mistakes to Avoid

### Mistake 1: Not Making Class Final

```java
// ❌ WRONG - Can be subclassed
public class Person {
    private final String name;
    // ...
}

// Subclass can add mutable state
public class MutablePerson extends Person {
    private String nickname;  // Mutable!
    public void setNickname(String n) { this.nickname = n; }
}
```

### Mistake 2: Not Making Fields Final

```java
// ❌ WRONG - Field can be reassigned
public final class Person {
    private String name;  // Not final!
    
    public void setName(String name) {
        this.name = name;  // Breaks immutability
    }
}
```

### Mistake 3: Exposing Mutable References

```java
// ❌ WRONG - Exposes mutable reference
public final class Team {
    private final List<String> members;
    
    public List<String> getMembers() {
        return members;  // Can be modified externally!
    }
}
```

### Mistake 4: Not Copying in Constructor

```java
// ❌ WRONG - Stores external reference
public final class Team {
    private final List<String> members;
    
    public Team(List<String> members) {
        this.members = members;  // External changes affect this object!
    }
}
```

### Mistake 5: Using Mutable Default Values

```java
// ❌ WRONG - Shared mutable default
public final class Config {
    private final List<String> options;
    
    public Config() {
        this.options = new ArrayList<>();  // Same list could be shared
    }
}
```

---

## Complete Example: Bank Account

```java
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Objects;

public final class BankAccount {
    
    private final String accountNumber;
    private final String holderName;
    private final double balance;
    private final List<Transaction> transactions;
    private final LocalDateTime createdAt;
    
    // Private constructor - use factory methods
    private BankAccount(String accountNumber, String holderName, 
                        double balance, List<Transaction> transactions,
                        LocalDateTime createdAt) {
        this.accountNumber = accountNumber;
        this.holderName = holderName;
        this.balance = balance;
        this.transactions = Collections.unmodifiableList(new ArrayList<>(transactions));
        this.createdAt = createdAt;
    }
    
    // Factory method to create new account
    public static BankAccount create(String accountNumber, String holderName) {
        if (accountNumber == null || accountNumber.isBlank()) {
            throw new IllegalArgumentException("Account number required");
        }
        if (holderName == null || holderName.isBlank()) {
            throw new IllegalArgumentException("Holder name required");
        }
        return new BankAccount(
            accountNumber, 
            holderName, 
            0.0, 
            new ArrayList<>(),
            LocalDateTime.now()
        );
    }
    
    // "Modification" methods return new instances
    public BankAccount deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit amount must be positive");
        }
        
        List<Transaction> newTransactions = new ArrayList<>(this.transactions);
        newTransactions.add(new Transaction("DEPOSIT", amount, LocalDateTime.now()));
        
        return new BankAccount(
            this.accountNumber,
            this.holderName,
            this.balance + amount,
            newTransactions,
            this.createdAt
        );
    }
    
    public BankAccount withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Withdrawal amount must be positive");
        }
        if (amount > balance) {
            throw new IllegalArgumentException("Insufficient funds");
        }
        
        List<Transaction> newTransactions = new ArrayList<>(this.transactions);
        newTransactions.add(new Transaction("WITHDRAWAL", -amount, LocalDateTime.now()));
        
        return new BankAccount(
            this.accountNumber,
            this.holderName,
            this.balance - amount,
            newTransactions,
            this.createdAt
        );
    }
    
    // Getters
    public String getAccountNumber() { return accountNumber; }
    public String getHolderName() { return holderName; }
    public double getBalance() { return balance; }
    public List<Transaction> getTransactions() { return transactions; }
    public LocalDateTime getCreatedAt() { return createdAt; }
    
    @Override
    public String toString() {
        return String.format("BankAccount{number='%s', holder='%s', balance=%.2f}",
            accountNumber, holderName, balance);
    }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        BankAccount that = (BankAccount) o;
        return Objects.equals(accountNumber, that.accountNumber);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(accountNumber);
    }
    
    // Immutable Transaction record
    public record Transaction(String type, double amount, LocalDateTime timestamp) {}
}
```

### Usage

```java
BankAccount account = BankAccount.create("ACC001", "John Doe");
System.out.println(account);  // balance=0.00

BankAccount afterDeposit = account.deposit(1000);
System.out.println(afterDeposit);  // balance=1000.00

BankAccount afterWithdraw = afterDeposit.withdraw(250);
System.out.println(afterWithdraw);  // balance=750.00

// Original account unchanged!
System.out.println(account);  // balance=0.00

// Transaction history
afterWithdraw.getTransactions().forEach(System.out::println);
```

---

## Quick Checklist

| Rule | Implementation |
|------|----------------|
| Class is `final` | `public final class MyClass` |
| All fields are `private final` | `private final String field;` |
| No setter methods | Don't create setters |
| Constructor initializes all fields | Assign in constructor |
| Defensive copy mutable params | `this.list = new ArrayList<>(list);` |
| Return copies from getters | `return new ArrayList<>(list);` |
| Or return unmodifiable | `return Collections.unmodifiableList(list);` |
| Use "with" methods for changes | `return new MyClass(newValue, this.other);` |

---

## Summary Template

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.Objects;

public final class ImmutableClass {              // 1. final class
    
    private final String immutableField;         // 2. private final fields
    private final List<String> mutableField;
    
    public ImmutableClass(String immutableField, List<String> mutableField) {
        // 3. Validate
        if (immutableField == null) {
            throw new IllegalArgumentException("Field cannot be null");
        }
        
        this.immutableField = immutableField;
        // 4. Defensive copy for mutable fields
        this.mutableField = new ArrayList<>(mutableField);
    }
    
    // 5. Only getters - no setters
    public String getImmutableField() {
        return immutableField;
    }
    
    public List<String> getMutableField() {
        // 6. Return unmodifiable or copy
        return Collections.unmodifiableList(mutableField);
    }
    
    // 7. "with" methods for modifications
    public ImmutableClass withImmutableField(String newValue) {
        return new ImmutableClass(newValue, this.mutableField);
    }
    
    // 8. equals, hashCode, toString
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        ImmutableClass that = (ImmutableClass) o;
        return Objects.equals(immutableField, that.immutableField) &&
               Objects.equals(mutableField, that.mutableField);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(immutableField, mutableField);
    }
    
    @Override
    public String toString() {
        return "ImmutableClass{" +
               "immutableField='" + immutableField + '\'' +
               ", mutableField=" + mutableField +
               '}';
    }
}
```

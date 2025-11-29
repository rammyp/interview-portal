# Java Design Patterns - Complete Guide

## Introduction

Design patterns are reusable solutions to common software design problems. The 23 classic patterns were defined in the "Gang of Four" (GoF) book and are divided into three categories:

- **Creational** - Object creation mechanisms
- **Structural** - Object composition and relationships
- **Behavioral** - Object communication and responsibility

---

# Creational Patterns (5)

*Deal with object creation mechanisms*

---

## 1. Singleton

### Purpose
Ensure only ONE instance of a class exists throughout the application.

### Use Cases
- Database connections
- Logging
- Configuration/Settings
- Caches
- Thread pools

### Implementation

```java
// Basic Singleton
public class Singleton {
    
    private static Singleton instance;
    
    private Singleton() {}  // Private constructor
    
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

```java
// Thread-Safe Singleton (Double-Checked Locking)
public class Singleton {
    
    private static volatile Singleton instance;
    
    private Singleton() {}
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

```java
// Best Approach - Enum Singleton
public enum Singleton {
    INSTANCE;
    
    public void doSomething() {
        System.out.println("Doing something");
    }
}

// Usage
Singleton.INSTANCE.doSomething();
```

### Real-World Examples
- `java.lang.Runtime.getRuntime()`
- `java.awt.Desktop.getDesktop()`
- Spring Bean (default scope)

---

## 2. Factory Method

### Purpose
Define an interface for creating objects, but let subclasses decide which class to instantiate.

### Use Cases
- When object creation is complex
- When you need flexibility in creating objects
- Framework development
- Plugin architectures

### Implementation

```java
// Product interface
interface Animal {
    void speak();
}

// Concrete products
class Dog implements Animal {
    public void speak() {
        System.out.println("Woof!");
    }
}

class Cat implements Animal {
    public void speak() {
        System.out.println("Meow!");
    }
}

// Factory
class AnimalFactory {
    
    public static Animal createAnimal(String type) {
        switch (type.toLowerCase()) {
            case "dog":
                return new Dog();
            case "cat":
                return new Cat();
            default:
                throw new IllegalArgumentException("Unknown animal: " + type);
        }
    }
}

// Usage
Animal dog = AnimalFactory.createAnimal("dog");
dog.speak();  // Woof!
```

### Real-World Examples
- `java.util.Calendar.getInstance()`
- `java.text.NumberFormat.getInstance()`
- `java.nio.charset.Charset.forName()`

---

## 3. Abstract Factory

### Purpose
Create families of related objects without specifying their concrete classes.

### Use Cases
- Cross-platform UI toolkits
- Database access layers (MySQL, PostgreSQL, Oracle)
- Theme systems
- Document generation (PDF, Word, HTML)

### Implementation

```java
// Abstract products
interface Button {
    void render();
}

interface Checkbox {
    void render();
}

// Concrete products - Windows family
class WindowsButton implements Button {
    public void render() {
        System.out.println("Rendering Windows button");
    }
}

class WindowsCheckbox implements Checkbox {
    public void render() {
        System.out.println("Rendering Windows checkbox");
    }
}

// Concrete products - Mac family
class MacButton implements Button {
    public void render() {
        System.out.println("Rendering Mac button");
    }
}

class MacCheckbox implements Checkbox {
    public void render() {
        System.out.println("Rendering Mac checkbox");
    }
}

// Abstract factory
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete factories
class WindowsFactory implements GUIFactory {
    public Button createButton() {
        return new WindowsButton();
    }
    
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

class MacFactory implements GUIFactory {
    public Button createButton() {
        return new MacButton();
    }
    
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}

// Client code
class Application {
    private Button button;
    private Checkbox checkbox;
    
    public Application(GUIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }
    
    public void render() {
        button.render();
        checkbox.render();
    }
}

// Usage
GUIFactory factory;
String os = System.getProperty("os.name").toLowerCase();

if (os.contains("windows")) {
    factory = new WindowsFactory();
} else {
    factory = new MacFactory();
}

Application app = new Application(factory);
app.render();
```

### Real-World Examples
- `javax.xml.parsers.DocumentBuilderFactory`
- `javax.xml.transform.TransformerFactory`

---

## 4. Builder

### Purpose
Construct complex objects step by step. Separate construction from representation.

### Use Cases
- Objects with many optional parameters
- Immutable objects
- Fluent APIs
- Configuration objects
- Test data builders

### Implementation

```java
public class User {
    
    // Required fields
    private final String firstName;
    private final String lastName;
    
    // Optional fields
    private final int age;
    private final String email;
    private final String phone;
    private final String address;
    
    private User(Builder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.age = builder.age;
        this.email = builder.email;
        this.phone = builder.phone;
        this.address = builder.address;
    }
    
    // Getters...
    
    public static class Builder {
        
        // Required
        private final String firstName;
        private final String lastName;
        
        // Optional - defaults
        private int age = 0;
        private String email = "";
        private String phone = "";
        private String address = "";
        
        public Builder(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }
        
        public Builder age(int age) {
            this.age = age;
            return this;
        }
        
        public Builder email(String email) {
            this.email = email;
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
        
        public User build() {
            return new User(this);
        }
    }
}

// Usage
User user = new User.Builder("John", "Doe")
    .age(30)
    .email("john@example.com")
    .phone("123-456-7890")
    .build();
```

### Real-World Examples
- `java.lang.StringBuilder`
- `java.util.stream.Stream.Builder`
- `java.lang.ProcessBuilder`
- Lombok `@Builder`

---

## 5. Prototype

### Purpose
Create new objects by cloning existing ones instead of creating from scratch.

### Use Cases
- When object creation is expensive
- When you need copies of complex objects
- Configuration templates
- Game object spawning
- Undo/Redo implementations

### Implementation

```java
// Prototype interface
interface Prototype extends Cloneable {
    Prototype clone();
}

// Concrete prototype
class Employee implements Prototype {
    
    private String name;
    private String department;
    private List<String> skills;
    
    public Employee(String name, String department) {
        this.name = name;
        this.department = department;
        this.skills = new ArrayList<>();
    }
    
    // Copy constructor for deep clone
    private Employee(Employee source) {
        this.name = source.name;
        this.department = source.department;
        this.skills = new ArrayList<>(source.skills);  // Deep copy
    }
    
    public void addSkill(String skill) {
        skills.add(skill);
    }
    
    @Override
    public Employee clone() {
        return new Employee(this);
    }
    
    @Override
    public String toString() {
        return "Employee{name='" + name + "', dept='" + department + 
               "', skills=" + skills + "}";
    }
}

// Prototype registry
class EmployeeRegistry {
    
    private Map<String, Employee> templates = new HashMap<>();
    
    public void addTemplate(String key, Employee employee) {
        templates.put(key, employee);
    }
    
    public Employee createEmployee(String key) {
        Employee template = templates.get(key);
        if (template != null) {
            return template.clone();
        }
        return null;
    }
}

// Usage
EmployeeRegistry registry = new EmployeeRegistry();

// Create template
Employee devTemplate = new Employee("", "Engineering");
devTemplate.addSkill("Java");
devTemplate.addSkill("Git");
registry.addTemplate("developer", devTemplate);

// Clone from template
Employee dev1 = registry.createEmployee("developer");
Employee dev2 = registry.createEmployee("developer");

dev1.addSkill("Spring");  // Doesn't affect dev2
```

### Real-World Examples
- `java.lang.Object.clone()`
- `java.util.Arrays.copyOf()`

---

# Structural Patterns (7)

*Deal with object composition and relationships*

---

## 6. Adapter

### Purpose
Convert the interface of a class into another interface that clients expect. Makes incompatible interfaces work together.

### Use Cases
- Legacy code integration
- Third-party library integration
- API compatibility layers
- Data format conversion

### Implementation

```java
// Target interface (what client expects)
interface MediaPlayer {
    void play(String filename);
}

// Adaptee (incompatible interface)
class AdvancedVideoPlayer {
    public void playMp4(String filename) {
        System.out.println("Playing MP4: " + filename);
    }
    
    public void playAvi(String filename) {
        System.out.println("Playing AVI: " + filename);
    }
}

// Adapter
class VideoPlayerAdapter implements MediaPlayer {
    
    private AdvancedVideoPlayer videoPlayer;
    
    public VideoPlayerAdapter() {
        this.videoPlayer = new AdvancedVideoPlayer();
    }
    
    @Override
    public void play(String filename) {
        if (filename.endsWith(".mp4")) {
            videoPlayer.playMp4(filename);
        } else if (filename.endsWith(".avi")) {
            videoPlayer.playAvi(filename);
        } else {
            System.out.println("Unsupported format");
        }
    }
}

// Client
class AudioPlayer implements MediaPlayer {
    
    private MediaPlayer videoAdapter;
    
    public AudioPlayer() {
        this.videoAdapter = new VideoPlayerAdapter();
    }
    
    @Override
    public void play(String filename) {
        if (filename.endsWith(".mp3")) {
            System.out.println("Playing MP3: " + filename);
        } else {
            videoAdapter.play(filename);
        }
    }
}

// Usage
MediaPlayer player = new AudioPlayer();
player.play("song.mp3");    // Playing MP3: song.mp3
player.play("video.mp4");   // Playing MP4: video.mp4
```

### Real-World Examples
- `java.util.Arrays.asList()`
- `java.io.InputStreamReader`
- `java.io.OutputStreamWriter`

---

## 7. Bridge

### Purpose
Separate abstraction from implementation so both can vary independently.

### Use Cases
- Cross-platform applications
- Different rendering engines
- Database drivers
- Device drivers

### Implementation

```java
// Implementation interface
interface Color {
    void applyColor();
}

// Concrete implementations
class RedColor implements Color {
    public void applyColor() {
        System.out.print("Red");
    }
}

class BlueColor implements Color {
    public void applyColor() {
        System.out.print("Blue");
    }
}

class GreenColor implements Color {
    public void applyColor() {
        System.out.print("Green");
    }
}

// Abstraction
abstract class Shape {
    protected Color color;  // Bridge to implementation
    
    public Shape(Color color) {
        this.color = color;
    }
    
    abstract void draw();
}

// Refined abstractions
class Circle extends Shape {
    
    public Circle(Color color) {
        super(color);
    }
    
    @Override
    void draw() {
        System.out.print("Drawing Circle in ");
        color.applyColor();
        System.out.println();
    }
}

class Square extends Shape {
    
    public Square(Color color) {
        super(color);
    }
    
    @Override
    void draw() {
        System.out.print("Drawing Square in ");
        color.applyColor();
        System.out.println();
    }
}

// Usage - Mix and match independently
Shape redCircle = new Circle(new RedColor());
Shape blueSquare = new Square(new BlueColor());
Shape greenCircle = new Circle(new GreenColor());

redCircle.draw();    // Drawing Circle in Red
blueSquare.draw();   // Drawing Square in Blue
greenCircle.draw();  // Drawing Circle in Green
```

### Real-World Examples
- JDBC drivers
- `java.util.logging` handlers

---

## 8. Composite

### Purpose
Compose objects into tree structures. Treat individual objects and compositions uniformly.

### Use Cases
- File system (files and folders)
- Organization hierarchies
- GUI components
- Menu systems
- Expression trees

### Implementation

```java
// Component interface
interface FileSystemItem {
    String getName();
    int getSize();
    void display(String indent);
}

// Leaf
class File implements FileSystemItem {
    
    private String name;
    private int size;
    
    public File(String name, int size) {
        this.name = name;
        this.size = size;
    }
    
    public String getName() { return name; }
    public int getSize() { return size; }
    
    public void display(String indent) {
        System.out.println(indent + "📄 " + name + " (" + size + " KB)");
    }
}

// Composite
class Folder implements FileSystemItem {
    
    private String name;
    private List<FileSystemItem> children = new ArrayList<>();
    
    public Folder(String name) {
        this.name = name;
    }
    
    public void add(FileSystemItem item) {
        children.add(item);
    }
    
    public void remove(FileSystemItem item) {
        children.remove(item);
    }
    
    public String getName() { return name; }
    
    public int getSize() {
        return children.stream()
            .mapToInt(FileSystemItem::getSize)
            .sum();
    }
    
    public void display(String indent) {
        System.out.println(indent + "📁 " + name + " (" + getSize() + " KB)");
        for (FileSystemItem item : children) {
            item.display(indent + "  ");
        }
    }
}

// Usage
Folder root = new Folder("root");

Folder docs = new Folder("documents");
docs.add(new File("resume.pdf", 100));
docs.add(new File("notes.txt", 20));

Folder photos = new Folder("photos");
photos.add(new File("vacation.jpg", 500));
photos.add(new File("family.png", 300));

root.add(docs);
root.add(photos);
root.add(new File("readme.md", 5));

root.display("");
// 📁 root (925 KB)
//   📁 documents (120 KB)
//     📄 resume.pdf (100 KB)
//     📄 notes.txt (20 KB)
//   📁 photos (800 KB)
//     📄 vacation.jpg (500 KB)
//     📄 family.png (300 KB)
//   📄 readme.md (5 KB)
```

### Real-World Examples
- `java.awt.Container`
- `javax.swing.JComponent`
- `org.w3c.dom.Node`

---

## 9. Decorator

### Purpose
Add new functionality to objects dynamically without altering their structure.

### Use Cases
- Java I/O streams
- Adding features to UI components
- Logging wrappers
- Encryption layers
- Caching

### Implementation

```java
// Component interface
interface Coffee {
    String getDescription();
    double getCost();
}

// Concrete component
class SimpleCoffee implements Coffee {
    public String getDescription() {
        return "Simple Coffee";
    }
    
    public double getCost() {
        return 2.00;
    }
}

// Base decorator
abstract class CoffeeDecorator implements Coffee {
    
    protected Coffee coffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
    
    public String getDescription() {
        return coffee.getDescription();
    }
    
    public double getCost() {
        return coffee.getCost();
    }
}

// Concrete decorators
class MilkDecorator extends CoffeeDecorator {
    
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }
    
    public double getCost() {
        return coffee.getCost() + 0.50;
    }
}

class SugarDecorator extends CoffeeDecorator {
    
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }
    
    public double getCost() {
        return coffee.getCost() + 0.25;
    }
}

class WhippedCreamDecorator extends CoffeeDecorator {
    
    public WhippedCreamDecorator(Coffee coffee) {
        super(coffee);
    }
    
    public String getDescription() {
        return coffee.getDescription() + ", Whipped Cream";
    }
    
    public double getCost() {
        return coffee.getCost() + 0.75;
    }
}

// Usage - stack decorators
Coffee coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
coffee = new WhippedCreamDecorator(coffee);

System.out.println(coffee.getDescription());  // Simple Coffee, Milk, Sugar, Whipped Cream
System.out.println("$" + coffee.getCost());   // $3.5
```

### Real-World Examples
- `java.io.BufferedInputStream`
- `java.io.DataOutputStream`
- `java.util.Collections.synchronizedList()`

---

## 10. Facade

### Purpose
Provide a simplified, unified interface to a complex subsystem.

### Use Cases
- Simplifying complex APIs
- Wrapping legacy systems
- Service layers
- SDK wrappers

### Implementation

```java
// Complex subsystem classes
class InventoryService {
    public boolean checkStock(String productId) {
        System.out.println("Checking inventory for: " + productId);
        return true;
    }
    
    public void reduceStock(String productId, int qty) {
        System.out.println("Reducing stock: " + productId + " by " + qty);
    }
}

class PaymentService {
    public boolean processPayment(String userId, double amount) {
        System.out.println("Processing payment: $" + amount);
        return true;
    }
    
    public void refund(String userId, double amount) {
        System.out.println("Refunding: $" + amount);
    }
}

class ShippingService {
    public String createShipment(String address) {
        System.out.println("Creating shipment to: " + address);
        return "TRACK123";
    }
}

class NotificationService {
    public void sendEmail(String userId, String message) {
        System.out.println("Email to " + userId + ": " + message);
    }
}

// Facade - simplifies complex interactions
class OrderFacade {
    
    private InventoryService inventory = new InventoryService();
    private PaymentService payment = new PaymentService();
    private ShippingService shipping = new ShippingService();
    private NotificationService notification = new NotificationService();
    
    // One simple method handles entire order process
    public String placeOrder(String userId, String productId, 
                             double amount, String address) {
        
        if (!inventory.checkStock(productId)) {
            notification.sendEmail(userId, "Item out of stock");
            return null;
        }
        
        if (!payment.processPayment(userId, amount)) {
            notification.sendEmail(userId, "Payment failed");
            return null;
        }
        
        inventory.reduceStock(productId, 1);
        String trackingId = shipping.createShipment(address);
        notification.sendEmail(userId, "Order placed! Track: " + trackingId);
        
        return trackingId;
    }
    
    public void cancelOrder(String userId, double amount) {
        payment.refund(userId, amount);
        notification.sendEmail(userId, "Order cancelled");
    }
}

// Usage - simple and clean
OrderFacade orders = new OrderFacade();
orders.placeOrder("user123", "PROD456", 99.99, "123 Main St");
```

### Real-World Examples
- `javax.faces.context.FacesContext`
- Spring `JdbcTemplate`

---

## 11. Flyweight

### Purpose
Share common state between multiple objects to save memory.

### Use Cases
- Text editors (character formatting)
- Game development (particles, trees, NPCs)
- Caching
- String pool

### Implementation

```java
// Flyweight interface
interface TreeType {
    void draw(int x, int y);
}

// Concrete flyweight (shared)
class TreeTypeImpl implements TreeType {
    
    private String name;
    private String color;
    private String texture;  // Large data
    
    public TreeTypeImpl(String name, String color, String texture) {
        this.name = name;
        this.color = color;
        this.texture = texture;
        System.out.println("Creating tree type: " + name);
    }
    
    public void draw(int x, int y) {
        System.out.println("Drawing " + name + " tree at (" + x + "," + y + ")");
    }
}

// Flyweight factory
class TreeFactory {
    
    private static Map<String, TreeType> treeTypes = new HashMap<>();
    
    public static TreeType getTreeType(String name, String color, String texture) {
        String key = name + "_" + color;
        
        if (!treeTypes.containsKey(key)) {
            treeTypes.put(key, new TreeTypeImpl(name, color, texture));
        }
        
        return treeTypes.get(key);
    }
    
    public static int getTypeCount() {
        return treeTypes.size();
    }
}

// Context (extrinsic state)
class Tree {
    
    private int x;
    private int y;
    private TreeType type;  // Shared flyweight
    
    public Tree(int x, int y, TreeType type) {
        this.x = x;
        this.y = y;
        this.type = type;
    }
    
    public void draw() {
        type.draw(x, y);
    }
}

// Forest using flyweights
class Forest {
    
    private List<Tree> trees = new ArrayList<>();
    
    public void plantTree(int x, int y, String name, String color, String texture) {
        TreeType type = TreeFactory.getTreeType(name, color, texture);
        Tree tree = new Tree(x, y, type);
        trees.add(tree);
    }
    
    public void draw() {
        for (Tree tree : trees) {
            tree.draw();
        }
    }
}

// Usage
Forest forest = new Forest();

// Plant 1000 trees but only create 2 tree types
for (int i = 0; i < 500; i++) {
    forest.plantTree(i * 10, i * 5, "Oak", "Green", "oak_texture.png");
}
for (int i = 0; i < 500; i++) {
    forest.plantTree(i * 8, i * 3, "Pine", "DarkGreen", "pine_texture.png");
}

System.out.println("Trees planted: 1000");
System.out.println("Tree types created: " + TreeFactory.getTypeCount());  // Only 2!
```

### Real-World Examples
- `java.lang.Integer.valueOf()` (-128 to 127 cached)
- `java.lang.String` interning
- `java.lang.Boolean.valueOf()`

---

## 12. Proxy

### Purpose
Provide a surrogate or placeholder for another object to control access.

### Types
- **Virtual Proxy** - Lazy initialization
- **Protection Proxy** - Access control
- **Remote Proxy** - Remote object access
- **Caching Proxy** - Cache results

### Use Cases
- Lazy loading
- Access control
- Logging/Auditing
- Caching
- Remote service calls

### Implementation

```java
// Subject interface
interface Image {
    void display();
}

// Real subject (expensive to create)
class RealImage implements Image {
    
    private String filename;
    
    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }
    
    private void loadFromDisk() {
        System.out.println("Loading image: " + filename);
        // Simulate expensive operation
        try { Thread.sleep(2000); } catch (Exception e) {}
    }
    
    public void display() {
        System.out.println("Displaying: " + filename);
    }
}

// Virtual Proxy (lazy loading)
class ProxyImage implements Image {
    
    private String filename;
    private RealImage realImage;
    
    public ProxyImage(String filename) {
        this.filename = filename;
    }
    
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename);  // Load only when needed
        }
        realImage.display();
    }
}

// Protection Proxy (access control)
interface BankAccount {
    void withdraw(double amount);
    double getBalance();
}

class RealBankAccount implements BankAccount {
    private double balance = 10000;
    
    public void withdraw(double amount) {
        balance -= amount;
        System.out.println("Withdrew: $" + amount);
    }
    
    public double getBalance() {
        return balance;
    }
}

class BankAccountProxy implements BankAccount {
    
    private RealBankAccount account;
    private String userRole;
    
    public BankAccountProxy(String userRole) {
        this.account = new RealBankAccount();
        this.userRole = userRole;
    }
    
    public void withdraw(double amount) {
        if (userRole.equals("ADMIN") || amount <= 1000) {
            account.withdraw(amount);
        } else {
            System.out.println("Access denied! Max withdrawal: $1000");
        }
    }
    
    public double getBalance() {
        return account.getBalance();
    }
}

// Usage
Image image = new ProxyImage("photo.jpg");
// Image not loaded yet

image.display();  // Now it loads
image.display();  // Uses cached image
```

### Real-World Examples
- `java.lang.reflect.Proxy`
- Spring AOP proxies
- Hibernate lazy loading

---

# Behavioral Patterns (11)

*Deal with object communication and responsibility*

---

## 13. Chain of Responsibility

### Purpose
Pass request along a chain of handlers. Each handler decides to process or pass to next.

### Use Cases
- Event handling
- Logging frameworks
- Authentication/Authorization
- Request filtering
- Exception handling

### Implementation

```java
// Handler interface
abstract class SupportHandler {
    
    protected SupportHandler nextHandler;
    protected String handlerName;
    
    public SupportHandler(String name) {
        this.handlerName = name;
    }
    
    public void setNext(SupportHandler handler) {
        this.nextHandler = handler;
    }
    
    public void handle(SupportTicket ticket) {
        if (canHandle(ticket)) {
            process(ticket);
        } else if (nextHandler != null) {
            System.out.println(handlerName + " passing to next handler...");
            nextHandler.handle(ticket);
        } else {
            System.out.println("No handler available for: " + ticket.getIssue());
        }
    }
    
    protected abstract boolean canHandle(SupportTicket ticket);
    protected abstract void process(SupportTicket ticket);
}

class SupportTicket {
    public enum Priority { LOW, MEDIUM, HIGH, CRITICAL }
    
    private String issue;
    private Priority priority;
    
    public SupportTicket(String issue, Priority priority) {
        this.issue = issue;
        this.priority = priority;
    }
    
    public String getIssue() { return issue; }
    public Priority getPriority() { return priority; }
}

// Concrete handlers
class Level1Support extends SupportHandler {
    
    public Level1Support() { super("Level 1 Support"); }
    
    protected boolean canHandle(SupportTicket ticket) {
        return ticket.getPriority() == SupportTicket.Priority.LOW;
    }
    
    protected void process(SupportTicket ticket) {
        System.out.println("Level 1 handling: " + ticket.getIssue());
    }
}

class Level2Support extends SupportHandler {
    
    public Level2Support() { super("Level 2 Support"); }
    
    protected boolean canHandle(SupportTicket ticket) {
        return ticket.getPriority() == SupportTicket.Priority.MEDIUM;
    }
    
    protected void process(SupportTicket ticket) {
        System.out.println("Level 2 handling: " + ticket.getIssue());
    }
}

class ManagerSupport extends SupportHandler {
    
    public ManagerSupport() { super("Manager"); }
    
    protected boolean canHandle(SupportTicket ticket) {
        return ticket.getPriority() == SupportTicket.Priority.HIGH ||
               ticket.getPriority() == SupportTicket.Priority.CRITICAL;
    }
    
    protected void process(SupportTicket ticket) {
        System.out.println("Manager handling: " + ticket.getIssue());
    }
}

// Usage
SupportHandler level1 = new Level1Support();
SupportHandler level2 = new Level2Support();
SupportHandler manager = new ManagerSupport();

level1.setNext(level2);
level2.setNext(manager);

level1.handle(new SupportTicket("Password reset", SupportTicket.Priority.LOW));
level1.handle(new SupportTicket("Bug in app", SupportTicket.Priority.MEDIUM));
level1.handle(new SupportTicket("System down", SupportTicket.Priority.CRITICAL));
```

### Real-World Examples
- `java.util.logging.Logger`
- Servlet filters
- Spring Security filter chain

---

## 14. Command

### Purpose
Encapsulate a request as an object. Allows parameterization, queuing, and undo/redo.

### Use Cases
- GUI buttons and menus
- Undo/Redo functionality
- Transaction systems
- Task scheduling
- Macro recording

### Implementation

```java
// Command interface
interface Command {
    void execute();
    void undo();
}

// Receiver
class Light {
    
    private String location;
    private boolean isOn = false;
    
    public Light(String location) {
        this.location = location;
    }
    
    public void turnOn() {
        isOn = true;
        System.out.println(location + " light is ON");
    }
    
    public void turnOff() {
        isOn = false;
        System.out.println(location + " light is OFF");
    }
}

// Concrete commands
class LightOnCommand implements Command {
    
    private Light light;
    
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    public void execute() {
        light.turnOn();
    }
    
    public void undo() {
        light.turnOff();
    }
}

class LightOffCommand implements Command {
    
    private Light light;
    
    public LightOffCommand(Light light) {
        this.light = light;
    }
    
    public void execute() {
        light.turnOff();
    }
    
    public void undo() {
        light.turnOn();
    }
}

// Invoker
class RemoteControl {
    
    private Command[] onCommands;
    private Command[] offCommands;
    private Stack<Command> history = new Stack<>();
    
    public RemoteControl(int slots) {
        onCommands = new Command[slots];
        offCommands = new Command[slots];
    }
    
    public void setCommand(int slot, Command onCmd, Command offCmd) {
        onCommands[slot] = onCmd;
        offCommands[slot] = offCmd;
    }
    
    public void pressOn(int slot) {
        if (onCommands[slot] != null) {
            onCommands[slot].execute();
            history.push(onCommands[slot]);
        }
    }
    
    public void pressOff(int slot) {
        if (offCommands[slot] != null) {
            offCommands[slot].execute();
            history.push(offCommands[slot]);
        }
    }
    
    public void pressUndo() {
        if (!history.isEmpty()) {
            Command lastCommand = history.pop();
            lastCommand.undo();
        }
    }
}

// Usage
Light livingRoom = new Light("Living Room");
Light bedroom = new Light("Bedroom");

RemoteControl remote = new RemoteControl(2);
remote.setCommand(0, new LightOnCommand(livingRoom), new LightOffCommand(livingRoom));
remote.setCommand(1, new LightOnCommand(bedroom), new LightOffCommand(bedroom));

remote.pressOn(0);   // Living Room light is ON
remote.pressOn(1);   // Bedroom light is ON
remote.pressUndo();  // Bedroom light is OFF
remote.pressUndo();  // Living Room light is OFF
```

### Real-World Examples
- `java.lang.Runnable`
- `javax.swing.Action`

---

## 15. Iterator

### Purpose
Traverse a collection without exposing its internal structure.

### Use Cases
- Collection traversal
- Unified iteration interface
- Multiple traversal algorithms
- Lazy evaluation

### Implementation

```java
// Iterator interface
interface Iterator<T> {
    boolean hasNext();
    T next();
}

// Aggregate interface
interface Container<T> {
    Iterator<T> createIterator();
}

// Concrete collection
class BookCollection implements Container<String> {
    
    private String[] books;
    private int size = 0;
    
    public BookCollection(int capacity) {
        books = new String[capacity];
    }
    
    public void addBook(String book) {
        if (size < books.length) {
            books[size++] = book;
        }
    }
    
    @Override
    public Iterator<String> createIterator() {
        return new BookIterator();
    }
    
    // Inner iterator class
    private class BookIterator implements Iterator<String> {
        
        private int index = 0;
        
        @Override
        public boolean hasNext() {
            return index < size;
        }
        
        @Override
        public String next() {
            if (hasNext()) {
                return books[index++];
            }
            return null;
        }
    }
}

// Usage
BookCollection collection = new BookCollection(5);
collection.addBook("Design Patterns");
collection.addBook("Clean Code");
collection.addBook("Effective Java");

Iterator<String> iterator = collection.createIterator();
while (iterator.hasNext()) {
    System.out.println(iterator.next());
}
```

### Real-World Examples
- `java.util.Iterator`
- `java.util.Enumeration`
- `java.util.Scanner`

---

## 16. Mediator

### Purpose
Define an object that encapsulates how a set of objects interact. Promotes loose coupling.

### Use Cases
- Chat rooms
- Air traffic control
- GUI component communication
- Event systems

### Implementation

```java
// Mediator interface
interface ChatMediator {
    void sendMessage(String message, User sender);
    void addUser(User user);
}

// Concrete mediator
class ChatRoom implements ChatMediator {
    
    private List<User> users = new ArrayList<>();
    
    @Override
    public void addUser(User user) {
        users.add(user);
    }
    
    @Override
    public void sendMessage(String message, User sender) {
        for (User user : users) {
            if (user != sender) {
                user.receive(message, sender.getName());
            }
        }
    }
}

// Colleague
abstract class User {
    
    protected ChatMediator mediator;
    protected String name;
    
    public User(ChatMediator mediator, String name) {
        this.mediator = mediator;
        this.name = name;
    }
    
    public String getName() {
        return name;
    }
    
    public abstract void send(String message);
    public abstract void receive(String message, String from);
}

// Concrete colleague
class ChatUser extends User {
    
    public ChatUser(ChatMediator mediator, String name) {
        super(mediator, name);
    }
    
    @Override
    public void send(String message) {
        System.out.println(name + " sends: " + message);
        mediator.sendMessage(message, this);
    }
    
    @Override
    public void receive(String message, String from) {
        System.out.println(name + " received from " + from + ": " + message);
    }
}

// Usage
ChatMediator chatRoom = new ChatRoom();

User john = new ChatUser(chatRoom, "John");
User jane = new ChatUser(chatRoom, "Jane");
User bob = new ChatUser(chatRoom, "Bob");

chatRoom.addUser(john);
chatRoom.addUser(jane);
chatRoom.addUser(bob);

john.send("Hello everyone!");
// John sends: Hello everyone!
// Jane received from John: Hello everyone!
// Bob received from John: Hello everyone!
```

### Real-World Examples
- `java.util.Timer`
- `java.util.concurrent.Executor`

---

## 17. Memento

### Purpose
Capture and restore an object's internal state without violating encapsulation.

### Use Cases
- Undo/Redo functionality
- Snapshots
- Transaction rollback
- Game save states

### Implementation

```java
// Memento (stores state)
class EditorMemento {
    
    private final String content;
    private final int cursorPosition;
    private final Date timestamp;
    
    public EditorMemento(String content, int cursorPosition) {
        this.content = content;
        this.cursorPosition = cursorPosition;
        this.timestamp = new Date();
    }
    
    public String getContent() { return content; }
    public int getCursorPosition() { return cursorPosition; }
    public Date getTimestamp() { return timestamp; }
}

// Originator (creates and restores from memento)
class TextEditor {
    
    private String content = "";
    private int cursorPosition = 0;
    
    public void write(String text) {
        content += text;
        cursorPosition = content.length();
    }
    
    public void setCursor(int position) {
        this.cursorPosition = Math.min(position, content.length());
    }
    
    public String getContent() { return content; }
    
    // Create memento
    public EditorMemento save() {
        return new EditorMemento(content, cursorPosition);
    }
    
    // Restore from memento
    public void restore(EditorMemento memento) {
        this.content = memento.getContent();
        this.cursorPosition = memento.getCursorPosition();
    }
    
    @Override
    public String toString() {
        return "Content: '" + content + "' | Cursor: " + cursorPosition;
    }
}

// Caretaker (manages mementos)
class History {
    
    private Stack<EditorMemento> undoStack = new Stack<>();
    private Stack<EditorMemento> redoStack = new Stack<>();
    
    public void save(EditorMemento memento) {
        undoStack.push(memento);
        redoStack.clear();
    }
    
    public EditorMemento undo() {
        if (undoStack.size() > 1) {
            redoStack.push(undoStack.pop());
            return undoStack.peek();
        }
        return undoStack.isEmpty() ? null : undoStack.peek();
    }
    
    public EditorMemento redo() {
        if (!redoStack.isEmpty()) {
            EditorMemento memento = redoStack.pop();
            undoStack.push(memento);
            return memento;
        }
        return null;
    }
}

// Usage
TextEditor editor = new TextEditor();
History history = new History();

history.save(editor.save());  // Save initial state

editor.write("Hello");
history.save(editor.save());
System.out.println(editor);  // Content: 'Hello' | Cursor: 5

editor.write(" World");
history.save(editor.save());
System.out.println(editor);  // Content: 'Hello World' | Cursor: 11

editor.write("!");
history.save(editor.save());
System.out.println(editor);  // Content: 'Hello World!' | Cursor: 12

// Undo
editor.restore(history.undo());
System.out.println("Undo: " + editor);  // Content: 'Hello World' | Cursor: 11

editor.restore(history.undo());
System.out.println("Undo: " + editor);  // Content: 'Hello' | Cursor: 5

// Redo
editor.restore(history.redo());
System.out.println("Redo: " + editor);  // Content: 'Hello World' | Cursor: 11
```

### Real-World Examples
- `java.io.Serializable`
- IDE undo functionality

---

## 18. Observer

### Purpose
Define a one-to-many dependency. When one object changes, all dependents are notified.

### Use Cases
- Event systems
- MVC pattern (Model notifies Views)
- Pub/Sub messaging
- UI updates
- Stock price monitoring

### Implementation

```java
// Observer interface
interface Observer {
    void update(String stockName, double price);
}

// Subject interface
interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

// Concrete subject
class Stock implements Subject {
    
    private List<Observer> observers = new ArrayList<>();
    private String name;
    private double price;
    
    public Stock(String name, double price) {
        this.name = name;
        this.price = price;
    }
    
    public void setPrice(double price) {
        this.price = price;
        notifyObservers();
    }
    
    @Override
    public void attach(Observer observer) {
        observers.add(observer);
    }
    
    @Override
    public void detach(Observer observer) {
        observers.remove(observer);
    }
    
    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(name, price);
        }
    }
}

// Concrete observers
class StockTrader implements Observer {
    
    private String name;
    
    public StockTrader(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String stockName, double price) {
        System.out.println(name + " notified: " + stockName + " is now $" + price);
    }
}

class StockAlert implements Observer {
    
    private double threshold;
    
    public StockAlert(double threshold) {
        this.threshold = threshold;
    }
    
    @Override
    public void update(String stockName, double price) {
        if (price > threshold) {
            System.out.println("ALERT: " + stockName + " exceeded $" + threshold);
        }
    }
}

// Usage
Stock appleStock = new Stock("AAPL", 150.0);

Observer trader1 = new StockTrader("John");
Observer trader2 = new StockTrader("Jane");
Observer alert = new StockAlert(155.0);

appleStock.attach(trader1);
appleStock.attach(trader2);
appleStock.attach(alert);

appleStock.setPrice(152.0);
// John notified: AAPL is now $152.0
// Jane notified: AAPL is now $152.0

appleStock.setPrice(158.0);
// John notified: AAPL is now $158.0
// Jane notified: AAPL is now $158.0
// ALERT: AAPL exceeded $155.0
```

### Real-World Examples
- `java.util.Observer` (deprecated)
- `java.util.EventListener`
- JavaBeans PropertyChangeListener

---

## 19. State

### Purpose
Allow an object to change its behavior when its internal state changes.

### Use Cases
- Vending machines
- Document workflow (Draft → Review → Published)
- TCP connection states
- Game character states
- Media player (Play, Pause, Stop)

### Implementation

```java
// State interface
interface OrderState {
    void next(Order order);
    void prev(Order order);
    void printStatus();
}

// Concrete states
class OrderedState implements OrderState {
    
    @Override
    public void next(Order order) {
        order.setState(new ShippedState());
    }
    
    @Override
    public void prev(Order order) {
        System.out.println("Order is in its initial state.");
    }
    
    @Override
    public void printStatus() {
        System.out.println("Order placed. Waiting for shipment.");
    }
}

class ShippedState implements OrderState {
    
    @Override
    public void next(Order order) {
        order.setState(new DeliveredState());
    }
    
    @Override
    public void prev(Order order) {
        order.setState(new OrderedState());
    }
    
    @Override
    public void printStatus() {
        System.out.println("Order shipped. In transit.");
    }
}

class DeliveredState implements OrderState {
    
    @Override
    public void next(Order order) {
        System.out.println("Order already delivered.");
    }
    
    @Override
    public void prev(Order order) {
        order.setState(new ShippedState());
    }
    
    @Override
    public void printStatus() {
        System.out.println("Order delivered successfully!");
    }
}

// Context
class Order {
    
    private OrderState state;
    
    public Order() {
        this.state = new OrderedState();
    }
    
    public void setState(OrderState state) {
        this.state = state;
    }
    
    public void nextState() {
        state.next(this);
    }
    
    public void prevState() {
        state.prev(this);
    }
    
    public void printStatus() {
        state.printStatus();
    }
}

// Usage
Order order = new Order();

order.printStatus();  // Order placed. Waiting for shipment.
order.nextState();

order.printStatus();  // Order shipped. In transit.
order.nextState();

order.printStatus();  // Order delivered successfully!
order.nextState();    // Order already delivered.
```

### Real-World Examples
- `javax.faces.lifecycle.Lifecycle`

---

## 20. Strategy

### Purpose
Define a family of algorithms, encapsulate each one, and make them interchangeable.

### Use Cases
- Payment processing
- Sorting algorithms
- Compression algorithms
- Route calculation
- Validation strategies

### Implementation

```java
// Strategy interface
interface PaymentStrategy {
    void pay(double amount);
}

// Concrete strategies
class CreditCardPayment implements PaymentStrategy {
    
    private String cardNumber;
    
    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " with Credit Card: " + 
                          cardNumber.substring(12) + "****");
    }
}

class PayPalPayment implements PaymentStrategy {
    
    private String email;
    
    public PayPalPayment(String email) {
        this.email = email;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " with PayPal: " + email);
    }
}

class CryptoPayment implements PaymentStrategy {
    
    private String walletAddress;
    
    public CryptoPayment(String walletAddress) {
        this.walletAddress = walletAddress;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " with Crypto: " + 
                          walletAddress.substring(0, 8) + "...");
    }
}

// Context
class ShoppingCart {
    
    private List<Double> items = new ArrayList<>();
    private PaymentStrategy paymentStrategy;
    
    public void addItem(double price) {
        items.add(price);
    }
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public void checkout() {
        double total = items.stream().mapToDouble(Double::doubleValue).sum();
        paymentStrategy.pay(total);
    }
}

// Usage - swap strategies at runtime
ShoppingCart cart = new ShoppingCart();
cart.addItem(50.0);
cart.addItem(30.0);

cart.setPaymentStrategy(new CreditCardPayment("1234567890123456"));
cart.checkout();  // Paid $80.0 with Credit Card: 3456****

cart.setPaymentStrategy(new PayPalPayment("user@email.com"));
cart.checkout();  // Paid $80.0 with PayPal: user@email.com
```

### Real-World Examples
- `java.util.Comparator`
- `java.util.concurrent.RejectedExecutionHandler`

---

## 21. Template Method

### Purpose
Define the skeleton of an algorithm, letting subclasses override specific steps.

### Use Cases
- Frameworks
- Data processing pipelines
- Build processes
- Testing frameworks

### Implementation

```java
// Abstract class with template method
abstract class DataMiner {
    
    // Template method - final prevents override
    public final void mine(String path) {
        openFile(path);
        extractData();
        parseData();
        analyzeData();
        sendReport();
        closeFile();
    }
    
    // Steps to be implemented by subclasses
    protected abstract void openFile(String path);
    protected abstract void extractData();
    protected abstract void closeFile();
    
    // Common implementation
    protected void parseData() {
        System.out.println("Parsing data into common format...");
    }
    
    protected void analyzeData() {
        System.out.println("Analyzing data...");
    }
    
    // Hook - can be overridden
    protected void sendReport() {
        System.out.println("Sending report via email...");
    }
}

// Concrete implementations
class PDFDataMiner extends DataMiner {
    
    @Override
    protected void openFile(String path) {
        System.out.println("Opening PDF file: " + path);
    }
    
    @Override
    protected void extractData() {
        System.out.println("Extracting text from PDF...");
    }
    
    @Override
    protected void closeFile() {
        System.out.println("Closing PDF file");
    }
}

class CSVDataMiner extends DataMiner {
    
    @Override
    protected void openFile(String path) {
        System.out.println("Opening CSV file: " + path);
    }
    
    @Override
    protected void extractData() {
        System.out.println("Reading CSV rows...");
    }
    
    @Override
    protected void closeFile() {
        System.out.println("Closing CSV file");
    }
    
    @Override
    protected void sendReport() {
        System.out.println("Uploading report to cloud storage...");
    }
}

// Usage
DataMiner pdfMiner = new PDFDataMiner();
pdfMiner.mine("document.pdf");

System.out.println("---");

DataMiner csvMiner = new CSVDataMiner();
csvMiner.mine("data.csv");
```

### Real-World Examples
- `java.io.InputStream.read()`
- `java.util.AbstractList.get()`
- JUnit test lifecycle

---

## 22. Visitor

### Purpose
Add new operations to existing class structures without modifying them.

### Use Cases
- Compilers (AST traversal)
- Document exporters
- Tax calculation
- Report generation

### Implementation

```java
// Visitor interface
interface ShoppingCartVisitor {
    void visit(Book book);
    void visit(Electronics electronics);
    void visit(Clothing clothing);
}

// Element interface
interface ItemElement {
    void accept(ShoppingCartVisitor visitor);
}

// Concrete elements
class Book implements ItemElement {
    
    private String title;
    private double price;
    
    public Book(String title, double price) {
        this.title = title;
        this.price = price;
    }
    
    public String getTitle() { return title; }
    public double getPrice() { return price; }
    
    @Override
    public void accept(ShoppingCartVisitor visitor) {
        visitor.visit(this);
    }
}

class Electronics implements ItemElement {
    
    private String name;
    private double price;
    
    public Electronics(String name, double price) {
        this.name = name;
        this.price = price;
    }
    
    public String getName() { return name; }
    public double getPrice() { return price; }
    
    @Override
    public void accept(ShoppingCartVisitor visitor) {
        visitor.visit(this);
    }
}

class Clothing implements ItemElement {
    
    private String name;
    private double price;
    
    public Clothing(String name, double price) {
        this.name = name;
        this.price = price;
    }
    
    public String getName() { return name; }
    public double getPrice() { return price; }
    
    @Override
    public void accept(ShoppingCartVisitor visitor) {
        visitor.visit(this);
    }
}

// Concrete visitors
class TaxCalculatorVisitor implements ShoppingCartVisitor {
    
    private double totalTax = 0;
    
    @Override
    public void visit(Book book) {
        totalTax += book.getPrice() * 0.05;  // 5% tax on books
    }
    
    @Override
    public void visit(Electronics electronics) {
        totalTax += electronics.getPrice() * 0.18;  // 18% tax
    }
    
    @Override
    public void visit(Clothing clothing) {
        totalTax += clothing.getPrice() * 0.12;  // 12% tax
    }
    
    public double getTotalTax() { return totalTax; }
}

class ShippingCostVisitor implements ShoppingCartVisitor {
    
    private double totalShipping = 0;
    
    @Override
    public void visit(Book book) {
        totalShipping += 2.0;  // Flat rate for books
    }
    
    @Override
    public void visit(Electronics electronics) {
        totalShipping += 10.0;  // Higher for electronics
    }
    
    @Override
    public void visit(Clothing clothing) {
        totalShipping += 5.0;
    }
    
    public double getTotalShipping() { return totalShipping; }
}

// Usage
List<ItemElement> items = Arrays.asList(
    new Book("Design Patterns", 50.0),
    new Electronics("Laptop", 1000.0),
    new Clothing("T-Shirt", 25.0)
);

TaxCalculatorVisitor taxVisitor = new TaxCalculatorVisitor();
ShippingCostVisitor shippingVisitor = new ShippingCostVisitor();

for (ItemElement item : items) {
    item.accept(taxVisitor);
    item.accept(shippingVisitor);
}

System.out.println("Total Tax: $" + taxVisitor.getTotalTax());
System.out.println("Total Shipping: $" + shippingVisitor.getTotalShipping());
```

### Real-World Examples
- `java.nio.file.FileVisitor`
- `javax.lang.model.element.ElementVisitor`

---

## 23. Interpreter

### Purpose
Define a grammar and an interpreter to interpret sentences in that language.

### Use Cases
- SQL parsing
- Regular expressions
- Mathematical expressions
- Configuration languages
- DSLs (Domain Specific Languages)

### Implementation

```java
// Abstract expression
interface Expression {
    int interpret();
}

// Terminal expressions
class NumberExpression implements Expression {
    
    private int number;
    
    public NumberExpression(int number) {
        this.number = number;
    }
    
    @Override
    public int interpret() {
        return number;
    }
}

// Non-terminal expressions
class AddExpression implements Expression {
    
    private Expression left;
    private Expression right;
    
    public AddExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public int interpret() {
        return left.interpret() + right.interpret();
    }
}

class SubtractExpression implements Expression {
    
    private Expression left;
    private Expression right;
    
    public SubtractExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public int interpret() {
        return left.interpret() - right.interpret();
    }
}

class MultiplyExpression implements Expression {
    
    private Expression left;
    private Expression right;
    
    public MultiplyExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public int interpret() {
        return left.interpret() * right.interpret();
    }
}

// Parser/Client
class ExpressionParser {
    
    public static Expression parse(String expression) {
        Stack<Expression> stack = new Stack<>();
        String[] tokens = expression.split(" ");
        
        for (String token : tokens) {
            if (isNumber(token)) {
                stack.push(new NumberExpression(Integer.parseInt(token)));
            } else if (token.equals("+")) {
                Expression right = stack.pop();
                Expression left = stack.pop();
                stack.push(new AddExpression(left, right));
            } else if (token.equals("-")) {
                Expression right = stack.pop();
                Expression left = stack.pop();
                stack.push(new SubtractExpression(left, right));
            } else if (token.equals("*")) {
                Expression right = stack.pop();
                Expression left = stack.pop();
                stack.push(new MultiplyExpression(left, right));
            }
        }
        
        return stack.pop();
    }
    
    private static boolean isNumber(String token) {
        try {
            Integer.parseInt(token);
            return true;
        } catch (NumberFormatException e) {
            return false;
        }
    }
}

// Usage (Reverse Polish Notation)
// "5 3 +" means 5 + 3
// "10 5 3 + *" means 10 * (5 + 3)

Expression expr1 = ExpressionParser.parse("5 3 +");
System.out.println("5 + 3 = " + expr1.interpret());  // 8

Expression expr2 = ExpressionParser.parse("10 5 3 + *");
System.out.println("10 * (5 + 3) = " + expr2.interpret());  // 80

Expression expr3 = ExpressionParser.parse("20 10 5 - +");
System.out.println("20 + (10 - 5) = " + expr3.interpret());  // 25
```

### Real-World Examples
- `java.util.regex.Pattern`
- `java.text.Format`

---

# Quick Reference

## By Category

| Category | Pattern | Key Purpose |
|----------|---------|-------------|
| **Creational** | Singleton | One instance only |
| | Factory Method | Delegate creation to subclasses |
| | Abstract Factory | Create families of objects |
| | Builder | Complex object construction |
| | Prototype | Clone objects |
| **Structural** | Adapter | Bridge incompatible interfaces |
| | Bridge | Separate abstraction from implementation |
| | Composite | Tree structures |
| | Decorator | Add behavior dynamically |
| | Facade | Simplify complex subsystems |
| | Flyweight | Share state to save memory |
| | Proxy | Control access to object |
| **Behavioral** | Chain of Responsibility | Pass request along chain |
| | Command | Encapsulate request as object |
| | Iterator | Traverse collection |
| | Mediator | Centralize communication |
| | Memento | Capture/restore state |
| | Observer | Notify on state changes |
| | State | Behavior based on state |
| | Strategy | Swap algorithms at runtime |
| | Template Method | Algorithm skeleton |
| | Visitor | Add operations without modifying |
| | Interpreter | Grammar and interpretation |

## By Frequency of Use

| Frequently Used | Occasionally Used | Rarely Used |
|-----------------|-------------------|-------------|
| Singleton | Abstract Factory | Interpreter |
| Factory Method | Prototype | Flyweight |
| Builder | Bridge | Memento |
| Strategy | Composite | Visitor |
| Observer | Chain of Responsibility | |
| Decorator | Command | |
| Adapter | State | |
| Facade | Mediator | |
| Proxy | Iterator | |
| Template Method | | |

## When to Use What

| Problem | Pattern |
|---------|---------|
| Need only one instance | Singleton |
| Hide object creation complexity | Factory |
| Create families of related objects | Abstract Factory |
| Object has many optional parameters | Builder |
| Need copies of complex objects | Prototype |
| Make incompatible interfaces work | Adapter |
| Separate abstraction from implementation | Bridge |
| Handle tree/hierarchical structures | Composite |
| Add features without subclassing | Decorator |
| Simplify complex API | Facade |
| Many similar objects eating memory | Flyweight |
| Control access, lazy loading, logging | Proxy |
| Handle request by multiple handlers | Chain of Responsibility |
| Need undo/redo, queue operations | Command |
| Traverse collection uniformly | Iterator |
| Many objects need to communicate | Mediator |
| Save/restore object state | Memento |
| Notify multiple objects of changes | Observer |
| Object behavior depends on state | State |
| Multiple algorithms, choose at runtime | Strategy |
| Same algorithm, different steps | Template Method |
| Add operations to class hierarchy | Visitor |
| Parse/evaluate expressions | Interpreter |

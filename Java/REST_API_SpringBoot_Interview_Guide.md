# REST API & Spring Boot Interview Preparation Guide

## Table of Contents
1. [REST API Fundamentals](#rest-api-fundamentals)
2. [Spring Boot Basics](#spring-boot-basics)
3. [Spring Core Concepts](#spring-core-concepts)
4. [Spring Boot Annotations](#spring-boot-annotations)
5. [REST Controller & Request Handling](#rest-controller-request-handling)
6. [Data Access with Spring Data JPA](#data-access-spring-data-jpa)
7. [Security with Spring Security](#spring-security)
8. [Microservices Architecture](#microservices-architecture)
9. [Testing in Spring Boot](#testing-spring-boot)
10. [Performance & Best Practices](#performance-best-practices)
11. [Real-World Scenarios](#real-world-scenarios)
12. [Common Interview Questions](#common-interview-questions)

---

## 1. REST API Fundamentals

### Q1: What is REST?

**REST (Representational State Transfer)** is an architectural style for designing networked applications.

**Key Principles:**
1. **Client-Server Architecture**: Separation of concerns
2. **Stateless**: Each request contains all information needed
3. **Cacheable**: Responses must define themselves as cacheable or not
4. **Uniform Interface**: Standardized way of communication
5. **Layered System**: Architecture can be composed of hierarchical layers
6. **Code on Demand** (Optional): Server can send executable code

### Q2: REST vs SOAP

| Feature | REST | SOAP |
|---------|------|------|
| Protocol | Architectural style | Protocol |
| Format | JSON, XML, HTML, plain text | XML only |
| Transport | HTTP/HTTPS | HTTP, SMTP, TCP |
| Performance | Faster, less bandwidth | Slower, more overhead |
| Caching | Supported | Not supported |
| Security | HTTPS, OAuth | WS-Security |
| Use Case | Public APIs, Mobile apps | Enterprise, Financial services |

### Q3: HTTP Methods in REST

```java
// GET - Retrieve resource(s)
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    return userService.findById(id);
}

// POST - Create new resource
@PostMapping("/users")
@ResponseStatus(HttpStatus.CREATED)
public User createUser(@RequestBody @Valid User user) {
    return userService.save(user);
}

// PUT - Update entire resource
@PutMapping("/users/{id}")
public User updateUser(@PathVariable Long id, @RequestBody User user) {
    user.setId(id);
    return userService.update(user);
}

// PATCH - Partial update
@PatchMapping("/users/{id}")
public User partialUpdate(@PathVariable Long id, @RequestBody Map<String, Object> updates) {
    return userService.partialUpdate(id, updates);
}

// DELETE - Remove resource
@DeleteMapping("/users/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public void deleteUser(@PathVariable Long id) {
    userService.delete(id);
}
```

### Q4: HTTP Status Codes

```java
// Success Responses
200 OK              // Successful GET, PUT
201 Created         // Successful POST
202 Accepted        // Request accepted for processing
204 No Content      // Successful DELETE

// Client Errors
400 Bad Request     // Invalid syntax
401 Unauthorized    // Authentication required
403 Forbidden       // No permission
404 Not Found       // Resource doesn't exist
405 Method Not Allowed
409 Conflict        // Resource conflict
422 Unprocessable Entity  // Validation errors

// Server Errors
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

### Q5: REST Best Practices

```java
// 1. Use nouns for resources, not verbs
✅ GET /users
❌ GET /getUsers

// 2. Use plural nouns
✅ GET /users/123
❌ GET /user/123

// 3. Use sub-resources for relationships
✅ GET /users/123/orders
❌ GET /getUserOrders

// 4. Use query parameters for filtering
✅ GET /users?status=active&sort=name
❌ GET /activeUsers

// 5. Version your API
✅ /api/v1/users
✅ Header: Accept: application/vnd.company.v1+json

// 6. Handle errors consistently
{
    "timestamp": "2024-03-15T10:30:00",
    "status": 400,
    "error": "Bad Request",
    "message": "Validation failed",
    "path": "/api/users"
}
```

---

## 2. Spring Boot Basics

### Q6: What is Spring Boot?

Spring Boot is an opinionated framework built on top of Spring Framework that simplifies the bootstrapping and development of Spring applications.

**Key Features:**
- **Auto-configuration**: Automatically configures based on dependencies
- **Standalone**: Creates standalone applications with embedded server
- **Production-ready**: Metrics, health checks, externalized configuration
- **No XML configuration**: Annotation-based configuration
- **Starter dependencies**: Simplified dependency management

### Q7: Spring Boot vs Spring Framework

```java
// Spring Framework (Traditional)
// 1. Manual configuration
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = "com.example")
public class WebConfig implements WebMvcConfigurer {
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver resolver = new InternalResourceViewResolver();
        resolver.setPrefix("/WEB-INF/views/");
        resolver.setSuffix(".jsp");
        return resolver;
    }
}

// 2. web.xml configuration needed
// 3. Manual server deployment

// Spring Boot
// 1. Auto-configuration
@SpringBootApplication  // Includes @Configuration, @EnableAutoConfiguration, @ComponentScan
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
// 2. No XML configuration
// 3. Embedded server
```

### Q8: Spring Boot Project Structure

```
my-app/
├── src/main/java/
│   └── com/example/myapp/
│       ├── MyAppApplication.java          # Main class
│       ├── controller/
│       │   └── UserController.java        # REST endpoints
│       ├── service/
│       │   ├── UserService.java           # Interface
│       │   └── UserServiceImpl.java       # Business logic
│       ├── repository/
│       │   └── UserRepository.java        # Data access
│       ├── entity/
│       │   └── User.java                  # JPA entities
│       ├── dto/
│       │   ├── UserDTO.java               # Data transfer objects
│       │   └── UserRequest.java
│       ├── exception/
│       │   └── GlobalExceptionHandler.java
│       └── config/
│           └── SecurityConfig.java
├── src/main/resources/
│   ├── application.yml                    # Configuration
│   ├── application-dev.yml                # Profile-specific
│   ├── application-prod.yml
│   └── static/                           # Static resources
└── src/test/java/                        # Test classes
```

### Q9: application.properties vs application.yml

```properties
# application.properties
server.port=8080
server.servlet.context-path=/api

spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

```yaml
# application.yml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: secret
  
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

---

## 3. Spring Core Concepts

### Q10: Dependency Injection (DI) / Inversion of Control (IoC)

```java
// Without DI - Tight coupling
public class UserService {
    private UserRepository repository = new UserRepository(); // Hard dependency
    
    public User getUser(Long id) {
        return repository.findById(id);
    }
}

// With DI - Loose coupling
@Service
public class UserService {
    private final UserRepository repository;
    
    // Constructor injection (Recommended)
    @Autowired  // Optional in Spring 4.3+ for single constructor
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
    
    // Setter injection
    @Autowired
    public void setRepository(UserRepository repository) {
        this.repository = repository;
    }
    
    // Field injection (Not recommended)
    @Autowired
    private UserRepository fieldRepository;
}
```

### Q11: Spring Bean Scopes

```java
// 1. Singleton (default) - One instance per Spring container
@Component
@Scope("singleton")
public class SingletonBean {
    // Shared instance
}

// 2. Prototype - New instance every time
@Component
@Scope("prototype")
public class PrototypeBean {
    // New instance for each injection
}

// 3. Request - One instance per HTTP request (Web only)
@Component
@Scope("request")
@RequestScope
public class RequestBean {
    // Lives for single HTTP request
}

// 4. Session - One instance per HTTP session (Web only)
@Component
@Scope("session")
@SessionScope
public class SessionBean {
    // Lives for user session
}

// 5. Application - One instance per ServletContext (Web only)
@Component
@ApplicationScope
public class ApplicationBean {
    // Shared across entire web application
}
```

### Q12: Spring Bean Lifecycle

```java
@Component
public class MyBean {
    
    // 1. Constructor
    public MyBean() {
        System.out.println("1. Constructor called");
    }
    
    // 2. Dependency injection
    @Autowired
    public void setDependency(MyDependency dependency) {
        System.out.println("2. Dependency injected");
    }
    
    // 3. @PostConstruct
    @PostConstruct
    public void init() {
        System.out.println("3. @PostConstruct called");
    }
    
    // 4. InitializingBean interface
    @Override
    public void afterPropertiesSet() {
        System.out.println("4. afterPropertiesSet called");
    }
    
    // 5. Custom init method
    // Defined in @Bean(initMethod = "customInit")
    
    // ... Bean is ready for use ...
    
    // 6. @PreDestroy
    @PreDestroy
    public void preDestroy() {
        System.out.println("6. @PreDestroy called");
    }
    
    // 7. DisposableBean interface
    @Override
    public void destroy() {
        System.out.println("7. destroy() called");
    }
    
    // 8. Custom destroy method
    // Defined in @Bean(destroyMethod = "customDestroy")
}
```

---

## 4. Spring Boot Annotations

### Q13: Core Spring Boot Annotations

```java
// Main Application
@SpringBootApplication  // = @Configuration + @EnableAutoConfiguration + @ComponentScan
@EnableScheduling      // Enable scheduled tasks
@EnableAsync          // Enable async processing
@EnableCaching        // Enable caching
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// Component Annotations
@Component    // Generic component
@Service      // Service layer
@Repository   // Data access layer
@Controller   // MVC controller
@RestController  // @Controller + @ResponseBody
@Configuration   // Configuration class

// Request Mapping
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping("/{id}")           // GET /api/users/{id}
    @PostMapping                   // POST /api/users
    @PutMapping("/{id}")           // PUT /api/users/{id}
    @DeleteMapping("/{id}")        // DELETE /api/users/{id}
    @PatchMapping("/{id}")         // PATCH /api/users/{id}
    
    // Request parameters
    @GetMapping
    public List<User> getUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(required = false) String sort,
        @RequestHeader("X-Auth-Token") String token,
        @CookieValue("session-id") String sessionId
    ) {
        // Implementation
    }
}

// Bean Creation
@Configuration
public class AppConfig {
    
    @Bean
    @Primary  // Primary bean when multiple candidates
    @Qualifier("mainDataSource")
    @Conditional(OnProductionCondition.class)
    @Profile("production")
    @Lazy  // Lazy initialization
    public DataSource dataSource() {
        return new HikariDataSource();
    }
}

// Validation
public class UserRequest {
    @NotNull(message = "Name is required")
    @Size(min = 2, max = 50)
    private String name;
    
    @Email(message = "Invalid email")
    @NotBlank
    private String email;
    
    @Min(18)
    @Max(150)
    private int age;
    
    @Pattern(regexp = "^[0-9]{10}$")
    private String phone;
}

// Property Injection
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    @Value("${app.name}")
    private String name;
    
    @Value("${app.version:1.0}")  // Default value
    private String version;
    
    @Value("#{systemProperties['user.home']}")  // SpEL
    private String userHome;
}
```

---

## 5. REST Controller & Request Handling

### Q14: Building RESTful Controllers

```java
@RestController
@RequestMapping("/api/v1/users")
@Validated
@Slf4j  // Lombok for logging
public class UserController {
    
    private final UserService userService;
    
    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    // GET all users with pagination
    @GetMapping
    public ResponseEntity<Page<UserDTO>> getAllUsers(
            @PageableDefault(size = 20, sort = "id") Pageable pageable) {
        Page<UserDTO> users = userService.findAll(pageable);
        return ResponseEntity.ok(users);
    }
    
    // GET user by ID
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
        return userService.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
    
    // POST - Create user
    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody UserRequest request) {
        UserDTO created = userService.create(request);
        URI location = ServletUriComponentsBuilder
                .fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(created.getId())
                .toUri();
        return ResponseEntity.created(location).body(created);
    }
    
    // PUT - Update user
    @PutMapping("/{id}")
    public ResponseEntity<UserDTO> updateUser(
            @PathVariable Long id, 
            @Valid @RequestBody UserRequest request) {
        return userService.update(id, request)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }
    
    // DELETE user
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
    
    // Search users
    @GetMapping("/search")
    public ResponseEntity<List<UserDTO>> searchUsers(
            @RequestParam String query,
            @RequestParam(required = false) String status) {
        List<UserDTO> results = userService.search(query, status);
        return ResponseEntity.ok(results);
    }
    
    // Upload profile picture
    @PostMapping("/{id}/avatar")
    public ResponseEntity<String> uploadAvatar(
            @PathVariable Long id,
            @RequestParam("file") MultipartFile file) {
        String url = userService.uploadAvatar(id, file);
        return ResponseEntity.ok(url);
    }
}
```

### Q15: Exception Handling

```java
// Global Exception Handler
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleResourceNotFound(ResourceNotFoundException ex) {
        log.error("Resource not found: ", ex);
        return ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.NOT_FOUND.value())
                .error("Resource Not Found")
                .message(ex.getMessage())
                .path(getPath())
                .build();
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidationException(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error ->
            errors.put(error.getField(), error.getDefaultMessage())
        );
        
        return ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.BAD_REQUEST.value())
                .error("Validation Failed")
                .message("Invalid input parameters")
                .validationErrors(errors)
                .path(getPath())
                .build();
    }
    
    @ExceptionHandler(DataIntegrityViolationException.class)
    @ResponseStatus(HttpStatus.CONFLICT)
    public ErrorResponse handleDataIntegrity(DataIntegrityViolationException ex) {
        return ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.CONFLICT.value())
                .error("Data Integrity Violation")
                .message("Database constraint violation")
                .build();
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleGenericException(Exception ex) {
        log.error("Unexpected error: ", ex);
        return ErrorResponse.builder()
                .timestamp(LocalDateTime.now())
                .status(HttpStatus.INTERNAL_SERVER_ERROR.value())
                .error("Internal Server Error")
                .message("An unexpected error occurred")
                .build();
    }
    
    private String getPath() {
        return ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes())
                .getRequest().getRequestURI();
    }
}

// Custom Exceptions
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String resource, Long id) {
        super(String.format("%s not found with id: %d", resource, id));
    }
}

// Error Response DTO
@Data
@Builder
public class ErrorResponse {
    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;
    private Map<String, String> validationErrors;
}
```

---

## 6. Data Access with Spring Data JPA

### Q16: Entity Relationships

```java
// One-to-Many / Many-to-One
@Entity
@Table(name = "users")
@Data
@NoArgsConstructor
@AllArgsConstructor
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @Column(nullable = false)
    private String password;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    @JsonManagedReference
    private List<Order> orders = new ArrayList<>();
    
    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
    
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id", referencedColumnName = "id")
    private UserProfile profile;
    
    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
}

@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    @JsonBackReference
    private User user;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> items;
}
```

### Q17: Spring Data JPA Repository

```java
// Basic Repository
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    // Derived Query Methods
    Optional<User> findByEmail(String email);
    List<User> findByAgeGreaterThan(int age);
    List<User> findByNameContainingIgnoreCase(String name);
    boolean existsByEmail(String email);
    long countByStatus(String status);
    
    // @Query annotation
    @Query("SELECT u FROM User u WHERE u.email = ?1")
    Optional<User> findByEmailQuery(String email);
    
    @Query("SELECT u FROM User u WHERE u.name = :name AND u.age = :age")
    List<User> findByNameAndAge(@Param("name") String name, @Param("age") int age);
    
    // Native query
    @Query(value = "SELECT * FROM users WHERE email = ?1", nativeQuery = true)
    Optional<User> findByEmailNative(String email);
    
    // Modifying queries
    @Modifying
    @Transactional
    @Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
    int updateStatus(@Param("id") Long id, @Param("status") String status);
    
    // Pagination and Sorting
    Page<User> findByStatus(String status, Pageable pageable);
    List<User> findTop10ByOrderByCreatedAtDesc();
    
    // Projections
    @Query("SELECT u.name as name, u.email as email FROM User u")
    List<UserProjection> findAllProjections();
}

// Projection Interface
public interface UserProjection {
    String getName();
    String getEmail();
}

// Custom Repository Implementation
@Repository
public interface UserRepositoryCustom {
    List<User> findUsersByComplexQuery(String criteria);
}

@Repository
public class UserRepositoryImpl implements UserRepositoryCustom {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    @Override
    public List<User> findUsersByComplexQuery(String criteria) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        Root<User> root = query.from(User.class);
        
        // Build complex query
        Predicate predicate = cb.like(root.get("name"), "%" + criteria + "%");
        query.where(predicate);
        
        return entityManager.createQuery(query).getResultList();
    }
}
```

### Q18: Transaction Management

```java
@Service
@Transactional  // Class-level transaction
@Slf4j
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private OrderRepository orderRepository;
    
    // Read-only transaction
    @Transactional(readOnly = true)
    public List<User> findAll() {
        return userRepository.findAll();
    }
    
    // Transaction with rollback rules
    @Transactional(
        propagation = Propagation.REQUIRED,
        isolation = Isolation.READ_COMMITTED,
        timeout = 30,
        rollbackFor = {CustomException.class},
        noRollbackFor = {EmailException.class}
    )
    public User createUser(UserRequest request) {
        User user = new User();
        user.setEmail(request.getEmail());
        
        User saved = userRepository.save(user);
        
        // This will rollback entire transaction if fails
        sendWelcomeEmail(saved);
        
        return saved;
    }
    
    // Propagation types
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void auditLog(String action) {
        // Always creates new transaction
    }
    
    @Transactional(propagation = Propagation.MANDATORY)
    public void mustHaveTransaction() {
        // Throws exception if no existing transaction
    }
    
    // Programmatic transaction management
    @Autowired
    private PlatformTransactionManager transactionManager;
    
    public void programmaticTransaction() {
        TransactionTemplate template = new TransactionTemplate(transactionManager);
        
        template.execute(status -> {
            try {
                // Transactional code
                userRepository.save(new User());
                return "Success";
            } catch (Exception e) {
                status.setRollbackOnly();
                return "Failed";
            }
        });
    }
}
```

---

## 7. Security with Spring Security

### Q19: Spring Security Configuration

```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig {
    
    @Autowired
    private JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
    
    @Autowired
    private JwtRequestFilter jwtRequestFilter;
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .cors().and()
            .authorizeHttpRequests(authz -> authz
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/products/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/user/**").hasAnyRole("USER", "ADMIN")
                .anyRequest().authenticated()
            )
            .exceptionHandling()
                .authenticationEntryPoint(jwtAuthenticationEntryPoint)
            .and()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS);
        
        // Add JWT filter
        http.addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
}

// JWT Implementation
@Component
public class JwtTokenUtil {
    
    @Value("${jwt.secret}")
    private String secret;
    
    @Value("${jwt.expiration}")
    private Long expiration;
    
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        return createToken(claims, userDetails.getUsername());
    }
    
    private String createToken(Map<String, Object> claims, String subject) {
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(subject)
            .setIssuedAt(new Date(System.currentTimeMillis()))
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(SignatureAlgorithm.HS512, secret)
            .compact();
    }
    
    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = getUsernameFromToken(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }
    
    public String getUsernameFromToken(String token) {
        return getClaimFromToken(token, Claims::getSubject);
    }
    
    private <T> T getClaimFromToken(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = getAllClaimsFromToken(token);
        return claimsResolver.apply(claims);
    }
    
    private Claims getAllClaimsFromToken(String token) {
        return Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
    }
}

// Method-level Security
@RestController
@RequestMapping("/api/admin")
public class AdminController {
    
    @PreAuthorize("hasRole('ADMIN')")
    @GetMapping("/users")
    public List<User> getAllUsers() {
        // Only ADMIN can access
    }
    
    @PreAuthorize("hasRole('ADMIN') and #username == authentication.principal.username")
    @GetMapping("/profile/{username}")
    public UserProfile getProfile(@PathVariable String username) {
        // ADMIN can only access their own profile
    }
    
    @PostAuthorize("returnObject.owner == authentication.name")
    @GetMapping("/document/{id}")
    public Document getDocument(@PathVariable Long id) {
        // Can only return if user owns the document
    }
}
```

---

## 8. Microservices Architecture

### Q20: Microservices with Spring Cloud

```java
// 1. Service Discovery - Eureka Server
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}

// 2. Eureka Client
@SpringBootApplication
@EnableEurekaClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}

// application.yml
spring:
  application:
    name: user-service
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/

// 3. API Gateway - Spring Cloud Gateway
@SpringBootApplication
public class ApiGatewayApplication {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("user-service", r -> r
                .path("/api/users/**")
                .filters(f -> f
                    .rewritePath("/api/users/(?<segment>.*)", "/users/${segment}")
                    .addRequestHeader("X-Service", "user-service")
                    .circuitBreaker(config -> config.setName("userServiceCB")))
                .uri("lb://USER-SERVICE"))
            .route("order-service", r -> r
                .path("/api/orders/**")
                .uri("lb://ORDER-SERVICE"))
            .build();
    }
}

// 4. Circuit Breaker - Resilience4j
@RestController
public class UserController {
    
    @Autowired
    private OrderServiceClient orderServiceClient;
    
    @GetMapping("/users/{id}/orders")
    @CircuitBreaker(name = "order-service", fallbackMethod = "getOrdersFallback")
    @Retry(name = "order-service")
    @RateLimiter(name = "order-service")
    public List<Order> getUserOrders(@PathVariable Long id) {
        return orderServiceClient.getOrdersByUserId(id);
    }
    
    public List<Order> getOrdersFallback(Long id, Exception ex) {
        log.error("Fallback triggered for user {}: {}", id, ex.getMessage());
        return Collections.emptyList();
    }
}

// 5. Feign Client for Inter-service Communication
@FeignClient(name = "order-service", fallback = OrderServiceFallback.class)
public interface OrderServiceClient {
    
    @GetMapping("/orders/user/{userId}")
    List<Order> getOrdersByUserId(@PathVariable Long userId);
    
    @PostMapping("/orders")
    Order createOrder(@RequestBody OrderRequest request);
}

@Component
public class OrderServiceFallback implements OrderServiceClient {
    
    @Override
    public List<Order> getOrdersByUserId(Long userId) {
        return Collections.emptyList();
    }
    
    @Override
    public Order createOrder(OrderRequest request) {
        return new Order(); // Return default order
    }
}

// 6. Distributed Configuration - Spring Cloud Config
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}

// 7. Distributed Tracing - Sleuth & Zipkin
// application.yml
spring:
  sleuth:
    sampler:
      probability: 1.0
  zipkin:
    base-url: http://localhost:9411
```

### Q21: Event-Driven Architecture with Kafka

```java
// Kafka Configuration
@Configuration
@EnableKafka
public class KafkaConfig {
    
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;
    
    // Producer Configuration
    @Bean
    public ProducerFactory<String, Object> producerFactory() {
        Map<String, Object> configs = new HashMap<>();
        configs.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configs.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        configs.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);
        return new DefaultKafkaProducerFactory<>(configs);
    }
    
    @Bean
    public KafkaTemplate<String, Object> kafkaTemplate() {
        return new KafkaTemplate<>(producerFactory());
    }
    
    // Consumer Configuration
    @Bean
    public ConsumerFactory<String, Object> consumerFactory() {
        Map<String, Object> configs = new HashMap<>();
        configs.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        configs.put(ConsumerConfig.GROUP_ID_CONFIG, "user-service-group");
        configs.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        configs.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, JsonDeserializer.class);
        return new DefaultKafkaConsumerFactory<>(configs);
    }
    
    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, Object> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, Object> factory = 
            new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }
}

// Event Publisher
@Component
@Slf4j
public class UserEventPublisher {
    
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
    
    public void publishUserCreatedEvent(User user) {
        UserCreatedEvent event = UserCreatedEvent.builder()
            .userId(user.getId())
            .email(user.getEmail())
            .timestamp(LocalDateTime.now())
            .build();
        
        kafkaTemplate.send("user-created-topic", event)
            .addCallback(
                result -> log.info("Event published: {}", event),
                ex -> log.error("Failed to publish event", ex)
            );
    }
}

// Event Consumer
@Component
@Slf4j
public class UserEventConsumer {
    
    @KafkaListener(topics = "user-created-topic", groupId = "notification-service")
    public void handleUserCreatedEvent(UserCreatedEvent event) {
        log.info("Received user created event: {}", event);
        // Send welcome email
        // Update analytics
        // Trigger other workflows
    }
    
    @KafkaListener(topics = "order-placed-topic", groupId = "user-service")
    public void handleOrderPlacedEvent(OrderPlacedEvent event) {
        // Update user statistics
    }
}
```

---

## 9. Testing in Spring Boot

### Q22: Unit Testing

```java
// Controller Test
@WebMvcTest(UserController.class)
@AutoConfigureMockMvc(addFilters = false)  // Disable security for testing
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void testGetUser() throws Exception {
        // Given
        UserDTO user = new UserDTO(1L, "John", "john@example.com");
        when(userService.findById(1L)).thenReturn(Optional.of(user));
        
        // When & Then
        mockMvc.perform(get("/api/users/1")
                .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.id").value(1))
                .andExpect(jsonPath("$.name").value("John"))
                .andExpect(jsonPath("$.email").value("john@example.com"));
        
        verify(userService, times(1)).findById(1L);
    }
    
    @Test
    void testCreateUser() throws Exception {
        // Given
        UserRequest request = new UserRequest("John", "john@example.com");
        UserDTO created = new UserDTO(1L, "John", "john@example.com");
        when(userService.create(any(UserRequest.class))).thenReturn(created);
        
        // When & Then
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(new ObjectMapper().writeValueAsString(request)))
                .andExpect(status().isCreated())
                .andExpect(header().exists("Location"))
                .andExpect(jsonPath("$.id").value(1));
    }
}

// Service Test
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserServiceImpl userService;
    
    @Test
    void testCreateUser() {
        // Given
        UserRequest request = new UserRequest("John", "john@example.com");
        User user = new User(null, "John", "john@example.com");
        User saved = new User(1L, "John", "john@example.com");
        
        when(userRepository.existsByEmail(request.getEmail())).thenReturn(false);
        when(userRepository.save(any(User.class))).thenReturn(saved);
        
        // When
        UserDTO result = userService.create(request);
        
        // Then
        assertNotNull(result);
        assertEquals(1L, result.getId());
        assertEquals("John", result.getName());
        
        verify(userRepository).existsByEmail(request.getEmail());
        verify(userRepository).save(any(User.class));
        verify(emailService).sendWelcomeEmail(saved);
    }
    
    @Test
    void testCreateUser_EmailExists() {
        // Given
        UserRequest request = new UserRequest("John", "john@example.com");
        when(userRepository.existsByEmail(request.getEmail())).thenReturn(true);
        
        // When & Then
        assertThrows(EmailAlreadyExistsException.class, 
            () -> userService.create(request));
        
        verify(userRepository).existsByEmail(request.getEmail());
        verify(userRepository, never()).save(any());
    }
}
```

### Q23: Integration Testing

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureMockMvc
@Transactional
@ActiveProfiles("test")
class UserIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    @Sql("/test-data.sql")  // Load test data
    void testGetAllUsers() throws Exception {
        mockMvc.perform(get("/api/users"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$", hasSize(3)));
    }
    
    @Test
    @WithMockUser(username = "admin", roles = {"ADMIN"})
    void testAdminEndpoint() throws Exception {
        mockMvc.perform(get("/api/admin/users"))
                .andExpect(status().isOk());
    }
    
    @Test
    @DirtiesContext  // Reset context after test
    void testCreateAndRetrieveUser() throws Exception {
        // Create user
        UserRequest request = new UserRequest("John", "john@example.com");
        
        MvcResult result = mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content(new ObjectMapper().writeValueAsString(request)))
                .andExpect(status().isCreated())
                .andReturn();
        
        // Extract created user ID
        String content = result.getResponse().getContentAsString();
        Long userId = JsonPath.parse(content).read("$.id", Long.class);
        
        // Verify in database
        Optional<User> saved = userRepository.findById(userId);
        assertTrue(saved.isPresent());
        assertEquals("John", saved.get().getName());
    }
}

// Repository Test
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@TestPropertySource(properties = {
    "spring.test.database.replace=none",
    "spring.datasource.url=jdbc:tc:mysql:8.0:///testdb"  // Testcontainers
})
class UserRepositoryTest {
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void testFindByEmail() {
        // Given
        User user = new User();
        user.setEmail("test@example.com");
        user.setName("Test User");
        entityManager.persistAndFlush(user);
        
        // When
        Optional<User> found = userRepository.findByEmail("test@example.com");
        
        // Then
        assertTrue(found.isPresent());
        assertEquals("Test User", found.get().getName());
    }
}
```

---

## 10. Performance & Best Practices

### Q24: Caching with Spring Boot

```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("users", "products");
    }
    
    // Redis Cache Manager
    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(
            new RedisStandaloneConfiguration("localhost", 6379));
    }
    
    @Bean
    public CacheManager redisCacheManager() {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(60))
            .disableCachingNullValues()
            .serializeKeysWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()));
        
        return RedisCacheManager.builder(redisConnectionFactory())
            .cacheDefaults(config)
            .build();
    }
}

@Service
public class UserService {
    
    @Cacheable(value = "users", key = "#id")
    public User findById(Long id) {
        log.info("Fetching user from database: {}", id);
        return userRepository.findById(id).orElse(null);
    }
    
    @CachePut(value = "users", key = "#user.id")
    public User update(User user) {
        log.info("Updating user: {}", user.getId());
        return userRepository.save(user);
    }
    
    @CacheEvict(value = "users", key = "#id")
    public void delete(Long id) {
        log.info("Deleting user: {}", id);
        userRepository.deleteById(id);
    }
    
    @CacheEvict(value = "users", allEntries = true)
    public void clearCache() {
        log.info("Clearing user cache");
    }
    
    @Cacheable(value = "users", condition = "#age > 18")
    public List<User> findByAge(int age) {
        return userRepository.findByAge(age);
    }
}
```

### Q25: Performance Optimization

```java
// 1. Database Connection Pooling - HikariCP
spring:
  datasource:
    hikari:
      connection-timeout: 30000
      maximum-pool-size: 10
      minimum-idle: 5
      idle-timeout: 600000
      max-lifetime: 1800000
      connection-test-query: SELECT 1

// 2. Lazy Loading and Fetch Strategies
@Entity
public class User {
    @OneToMany(fetch = FetchType.LAZY, mappedBy = "user")
    private List<Order> orders;
    
    // Use @EntityGraph to avoid N+1 problem
    @EntityGraph(attributePaths = {"orders", "profile"})
    @Query("SELECT u FROM User u WHERE u.id = :id")
    Optional<User> findByIdWithOrdersAndProfile(@Param("id") Long id);
}

// 3. Async Processing
@Configuration
@EnableAsync
public class AsyncConfig {
    
    @Bean
    public TaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(500);
        executor.setThreadNamePrefix("Async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class EmailService {
    
    @Async
    public CompletableFuture<String> sendEmail(String email) {
        // Simulate long-running task
        Thread.sleep(5000);
        log.info("Email sent to: {}", email);
        return CompletableFuture.completedFuture("Success");
    }
}

// 4. Response Compression
server:
  compression:
    enabled: true
    mime-types: application/json,application/xml,text/html,text/xml,text/plain
    min-response-size: 1024

// 5. Pagination for Large Results
@GetMapping("/users")
public Page<User> getUsers(
    @PageableDefault(size = 20, sort = "id", direction = Sort.Direction.ASC) 
    Pageable pageable) {
    return userRepository.findAll(pageable);
}

// 6. Batch Processing
@Repository
public class UserBatchRepository {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    @Transactional
    public void batchInsert(List<User> users) {
        int batchSize = 50;
        for (int i = 0; i < users.size(); i++) {
            entityManager.persist(users.get(i));
            if (i % batchSize == 0 && i > 0) {
                entityManager.flush();
                entityManager.clear();
            }
        }
    }
}
```

---

## 11. Real-World Scenarios

### Q26: Building a Complete REST API

```java
// Complete User Management System
@RestController
@RequestMapping("/api/v1/users")
@Tag(name = "User Management", description = "User CRUD operations")
@Validated
@Slf4j
public class UserController {
    
    private final UserService userService;
    
    @Operation(summary = "Get users with filtering and pagination")
    @GetMapping
    public ResponseEntity<PagedResponse<UserDTO>> getUsers(
            @RequestParam(required = false) String search,
            @RequestParam(required = false) UserStatus status,
            @RequestParam(required = false) @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate fromDate,
            @RequestParam(required = false) @DateTimeFormat(iso = DateTimeFormat.ISO.DATE) LocalDate toDate,
            @PageableDefault(size = 20, sort = "createdAt", direction = Sort.Direction.DESC) Pageable pageable) {
        
        UserSearchCriteria criteria = UserSearchCriteria.builder()
                .search(search)
                .status(status)
                .fromDate(fromDate)
                .toDate(toDate)
                .build();
        
        Page<UserDTO> users = userService.searchUsers(criteria, pageable);
        
        PagedResponse<UserDTO> response = PagedResponse.<UserDTO>builder()
                .content(users.getContent())
                .page(users.getNumber())
                .size(users.getSize())
                .totalElements(users.getTotalElements())
                .totalPages(users.getTotalPages())
                .last(users.isLast())
                .build();
        
        return ResponseEntity.ok(response);
    }
    
    @Operation(summary = "Export users to CSV")
    @GetMapping("/export")
    @PreAuthorize("hasRole('ADMIN')")
    public void exportUsers(HttpServletResponse response) throws IOException {
        response.setContentType("text/csv");
        response.setHeader("Content-Disposition", "attachment; filename=\"users.csv\"");
        
        List<User> users = userService.findAll();
        CSVWriter writer = new CSVWriter(response.getWriter());
        
        // Write header
        String[] header = {"ID", "Name", "Email", "Status", "Created Date"};
        writer.writeNext(header);
        
        // Write data
        for (User user : users) {
            String[] data = {
                user.getId().toString(),
                user.getName(),
                user.getEmail(),
                user.getStatus().toString(),
                user.getCreatedAt().toString()
            };
            writer.writeNext(data);
        }
        
        writer.close();
    }
    
    @Operation(summary = "Bulk update users")
    @PostMapping("/bulk-update")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<BulkOperationResult> bulkUpdate(
            @RequestBody @Valid List<UserUpdateRequest> requests) {
        
        BulkOperationResult result = userService.bulkUpdate(requests);
        return ResponseEntity.ok(result);
    }
}

// Service Layer with Business Logic
@Service
@Transactional
@Slf4j
public class UserServiceImpl implements UserService {
    
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    private final EventPublisher eventPublisher;
    private final CacheManager cacheManager;
    
    @Override
    public Page<UserDTO> searchUsers(UserSearchCriteria criteria, Pageable pageable) {
        Specification<User> spec = Specification.where(null);
        
        if (StringUtils.hasText(criteria.getSearch())) {
            spec = spec.and((root, query, cb) -> 
                cb.or(
                    cb.like(cb.lower(root.get("name")), "%" + criteria.getSearch().toLowerCase() + "%"),
                    cb.like(cb.lower(root.get("email")), "%" + criteria.getSearch().toLowerCase() + "%")
                )
            );
        }
        
        if (criteria.getStatus() != null) {
            spec = spec.and((root, query, cb) -> 
                cb.equal(root.get("status"), criteria.getStatus())
            );
        }
        
        if (criteria.getFromDate() != null) {
            spec = spec.and((root, query, cb) -> 
                cb.greaterThanOrEqualTo(root.get("createdAt"), criteria.getFromDate())
            );
        }
        
        if (criteria.getToDate() != null) {
            spec = spec.and((root, query, cb) -> 
                cb.lessThanOrEqualTo(root.get("createdAt"), criteria.getToDate())
            );
        }
        
        Page<User> users = userRepository.findAll(spec, pageable);
        return users.map(userMapper::toDTO);
    }
    
    @Override
    @Retryable(value = {SQLException.class}, maxAttempts = 3, backoff = @Backoff(delay = 1000))
    public UserDTO create(UserCreateRequest request) {
        // Validate
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new EmailAlreadyExistsException(request.getEmail());
        }
        
        // Create user
        User user = userMapper.toEntity(request);
        user.setPassword(passwordEncoder.encode(request.getPassword()));
        user.setStatus(UserStatus.ACTIVE);
        user.setRoles(Set.of(roleRepository.findByName("ROLE_USER")));
        
        User saved = userRepository.save(user);
        
        // Publish event
        eventPublisher.publishEvent(new UserCreatedEvent(saved));
        
        // Send welcome email asynchronously
        emailService.sendWelcomeEmail(saved);
        
        return userMapper.toDTO(saved);
    }
    
    @Override
    public BulkOperationResult bulkUpdate(List<UserUpdateRequest> requests) {
        BulkOperationResult result = new BulkOperationResult();
        
        for (UserUpdateRequest request : requests) {
            try {
                User user = userRepository.findById(request.getId())
                    .orElseThrow(() -> new ResourceNotFoundException("User", request.getId()));
                
                userMapper.updateEntity(request, user);
                userRepository.save(user);
                
                result.addSuccess(request.getId());
            } catch (Exception e) {
                log.error("Failed to update user {}: {}", request.getId(), e.getMessage());
                result.addFailure(request.getId(), e.getMessage());
            }
        }
        
        // Clear cache after bulk operation
        cacheManager.getCache("users").clear();
        
        return result;
    }
}
```

---

## 12. Common Interview Questions

### Q27: Top 50 Spring Boot Interview Questions

1. **What is Spring Boot? How is it different from Spring?**
   - Auto-configuration, embedded server, starter dependencies

2. **What is @SpringBootApplication?**
   - Combines @Configuration, @EnableAutoConfiguration, @ComponentScan

3. **How does Spring Boot auto-configuration work?**
   - @Conditional annotations, spring.factories file

4. **What is Spring Boot Actuator?**
   - Production-ready features: health, metrics, info endpoints

5. **How to handle exceptions in Spring Boot?**
   - @ControllerAdvice, @ExceptionHandler

6. **What is Spring Data JPA?**
   - Abstraction over JPA, repository pattern

7. **Explain @Transactional annotation**
   - ACID properties, propagation, isolation levels

8. **How to implement security in Spring Boot?**
   - Spring Security, JWT, OAuth2

9. **What is dependency injection?**
   - IoC container, @Autowired, constructor injection

10. **Difference between @Component, @Service, @Repository?**
    - Semantic differences, exception translation

### Interview Preparation Tips

**Technical Preparation:**
1. **Understand REST principles** thoroughly
2. **Practice coding** REST endpoints
3. **Know Spring Boot annotations** by heart
4. **Understand transaction management**
5. **Be familiar with JPA and Hibernate**
6. **Know microservices patterns**
7. **Understand security concepts**

**Common Coding Challenges:**
1. Build a complete CRUD REST API
2. Implement JWT authentication
3. Handle file upload/download
4. Implement pagination and sorting
5. Create custom validators
6. Handle concurrent requests
7. Implement caching

**System Design Topics:**
1. Design URL shortener with Spring Boot
2. Design e-commerce backend
3. Design notification service
4. Design rate limiter
5. Design distributed cache

**Best Practices to Know:**
1. Use DTOs for API responses
2. Implement proper error handling
3. Use appropriate HTTP status codes
4. Version your APIs
5. Implement pagination for lists
6. Use async processing for long tasks
7. Implement proper logging
8. Write comprehensive tests
9. Use profiles for environments
10. Implement health checks

---

## Conclusion

This guide covers the essential topics for REST API and Spring Boot interviews. Focus on:

1. **Core Concepts**: REST principles, Spring fundamentals
2. **Hands-on Practice**: Build actual applications
3. **Best Practices**: Follow industry standards
4. **Testing**: Unit, integration, and end-to-end tests
5. **Production Readiness**: Security, monitoring, performance

Remember to practice coding these concepts, not just reading about them!

Good luck with your interview! 🚀

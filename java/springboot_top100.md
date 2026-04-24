# Top 100 Spring Boot Interview Questions & Answers

---

## 🟢 BEGINNER (Q1–Q30)

### Core Basics

**Q1. What is Spring Boot?**
Spring Boot is an opinionated framework built on top of the Spring Framework that simplifies the setup and development of Spring applications. It eliminates boilerplate configuration by providing auto-configuration, embedded servers, and starter dependencies.

---

**Q2. What are the main advantages of Spring Boot?**
- Auto-configuration reduces manual setup
- Embedded servers (Tomcat, Jetty) — no WAR deployment needed
- Spring Boot Starters simplify dependency management
- Production-ready features via Actuator
- Opinionated defaults that can be overridden

---

**Q3. What is the difference between Spring and Spring Boot?**
| Spring | Spring Boot |
|--------|-------------|
| Requires manual configuration (XML or Java) | Auto-configures based on classpath |
| No embedded server | Comes with embedded Tomcat/Jetty |
| More flexible, more setup | Convention over configuration |
| Standalone app needs extra work | Run with `main()` method directly |

---

**Q4. What is `@SpringBootApplication`?**
A convenience annotation that combines three annotations:
- `@Configuration` — marks the class as a source of bean definitions
- `@EnableAutoConfiguration` — enables Spring Boot's auto-configuration
- `@ComponentScan` — scans the package for Spring components

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

---

**Q5. What is auto-configuration in Spring Boot?**
Spring Boot automatically configures beans based on the dependencies present on the classpath. For example, if `spring-boot-starter-data-jpa` is on the classpath, it auto-configures a DataSource, EntityManagerFactory, and TransactionManager.

---

**Q6. How does Spring Boot auto-configuration work internally?**
It uses `@EnableAutoConfiguration`, which imports `AutoConfigurationImportSelector`. This reads the `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` file (or `spring.factories` in older versions) and conditionally registers beans using `@Conditional` annotations.

---

**Q7. What are Spring Boot Starters?**
Pre-packaged dependency descriptors that bundle commonly used libraries for a specific feature. For example:
- `spring-boot-starter-web` — Spring MVC + embedded Tomcat
- `spring-boot-starter-data-jpa` — JPA + Hibernate
- `spring-boot-starter-security` — Spring Security
- `spring-boot-starter-test` — JUnit + Mockito

---

**Q8. What is `application.properties` / `application.yml`?**
The central configuration file for a Spring Boot application. Used to configure server port, database connections, logging levels, and more.

```properties
server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
```

---

**Q9. What is the default embedded server in Spring Boot?**
**Apache Tomcat** is the default. You can switch to Jetty or Undertow by excluding Tomcat and adding the desired server dependency.

---

**Q10. How do you change the default port in Spring Boot?**
In `application.properties`:
```properties
server.port=9090
```
Or via command line: `java -jar app.jar --server.port=9090`

---

**Q11. What is Spring Initializr?**
A web-based tool at [start.spring.io](https://start.spring.io) that generates a Spring Boot project scaffold with chosen dependencies, build tool (Maven/Gradle), and Java version.

---

**Q12. What is `@RestController`?**
A combination of `@Controller` and `@ResponseBody`. It marks a class as a REST controller where every method returns data (JSON/XML) directly instead of a view name.

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() { return "Hello, World!"; }
}
```

---

**Q13. What is the difference between `@Controller` and `@RestController`?**
| `@Controller` | `@RestController` |
|---------------|-------------------|
| Returns view names (templates) | Returns data directly (JSON/XML) |
| Used with Thymeleaf/JSP | Used for REST APIs |
| Needs `@ResponseBody` on methods | `@ResponseBody` is implicit |

---

**Q14. What is `@RequestMapping`?**
Maps HTTP requests to handler methods. Can be applied at class or method level. Supports specifying HTTP method, path, headers, and more.

```java
@RequestMapping(value = "/users", method = RequestMethod.GET)
public List<User> getUsers() { ... }
```

---

**Q15. What are shortcut annotations for HTTP methods?**
- `@GetMapping` — HTTP GET
- `@PostMapping` — HTTP POST
- `@PutMapping` — HTTP PUT
- `@DeleteMapping` — HTTP DELETE
- `@PatchMapping` — HTTP PATCH

---

**Q16. What is `@PathVariable`?**
Extracts a value from the URI path.

```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) { ... }
```

---

**Q17. What is `@RequestParam`?**
Extracts a query parameter from the URL.

```java
@GetMapping("/search")
public List<User> search(@RequestParam String name) { ... }
// URL: /search?name=John
```

---

**Q18. What is `@RequestBody`?**
Binds the HTTP request body (typically JSON) to a Java object.

```java
@PostMapping("/users")
public User create(@RequestBody User user) { ... }
```

---

**Q19. What is `@ResponseBody`?**
Tells Spring to serialize the return value of a method directly to the HTTP response body (as JSON/XML).

---

**Q20. What is `ResponseEntity`?**
A wrapper for HTTP responses that allows you to control status code, headers, and body explicitly.

```java
@GetMapping("/user/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return ResponseEntity.ok(userService.findById(id));
}
```

---

**Q21. What is `@Component`?**
A generic stereotype annotation that marks a class as a Spring-managed bean. Detected during component scanning.

---

**Q22. What is the difference between `@Component`, `@Service`, `@Repository`, and `@Controller`?**
All are specializations of `@Component` and work the same for component scanning. Their distinction is semantic:
- `@Component` — generic bean
- `@Service` — business logic layer
- `@Repository` — data access layer (also enables exception translation)
- `@Controller` — presentation layer

---

**Q23. What is dependency injection?**
A design pattern where an object's dependencies are provided by an external entity (the Spring IoC container) rather than the object creating them itself. Types: constructor injection, setter injection, field injection.

---

**Q24. What is `@Autowired`?**
Tells Spring to inject a dependency automatically by type.

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

---

**Q25. Which type of injection is preferred — constructor or field?**
**Constructor injection** is preferred because:
- Dependencies are explicit and required
- Supports immutability (`final` fields)
- Easier to test without Spring context
- Avoids circular dependency issues being hidden

---

**Q26. What is the Spring IoC container?**
The Inversion of Control (IoC) container manages the lifecycle and configuration of application objects (beans). The main implementations are `BeanFactory` and `ApplicationContext`.

---

**Q27. What is a Spring Bean?**
An object managed by the Spring IoC container. Beans are created, assembled, and managed by Spring based on configuration metadata.

---

**Q28. What are bean scopes in Spring?**
- `singleton` (default) — one instance per Spring container
- `prototype` — new instance every time requested
- `request` — one per HTTP request (web only)
- `session` — one per HTTP session (web only)
- `application` — one per ServletContext (web only)

---

**Q29. What is `@Value`?**
Injects values from `application.properties` or environment variables into fields.

```java
@Value("${app.name}")
private String appName;
```

---

**Q30. What is Spring Boot DevTools?**
A development-time tool that enables automatic restart on code changes, live reload of static resources, and disables caching for templates during development.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
</dependency>
```

---

## 🟡 INTERMEDIATE (Q31–Q65)

### Data, JPA & Configuration

**Q31. What is Spring Data JPA?**
An abstraction layer over JPA that reduces boilerplate by providing repository interfaces. You define an interface extending `JpaRepository` and Spring generates the implementation.

```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByEmail(String email);
}
```

---

**Q32. What is `JpaRepository` and what does it provide?**
Extends `CrudRepository` and `PagingAndSortingRepository`. Provides methods like `save()`, `findById()`, `findAll()`, `deleteById()`, `count()`, and pagination/sorting support.

---

**Q33. What are derived query methods in Spring Data JPA?**
Methods whose implementation is automatically generated by Spring Data based on the method name.

```java
List<User> findByLastNameAndAge(String lastName, int age);
List<User> findByEmailContaining(String keyword);
```

---

**Q34. What is `@Entity` and `@Table`?**
- `@Entity` — marks a class as a JPA entity mapped to a database table
- `@Table(name="users")` — specifies the table name (optional; defaults to class name)

---

**Q35. What is `@Id` and `@GeneratedValue`?**
- `@Id` — marks the primary key field
- `@GeneratedValue(strategy = GenerationType.IDENTITY)` — auto-generates the primary key value

---

**Q36. What is `@Transactional`?**
Marks a method or class to run within a transaction. Spring handles commit/rollback automatically. A rollback occurs by default for unchecked exceptions.

```java
@Transactional
public void transferFunds(Long from, Long to, double amount) { ... }
```

---

**Q37. What is the difference between `@Query` and derived query methods?**
- **Derived methods**: Spring generates the query from the method name. Simple but limited.
- `@Query`: You write JPQL or native SQL. More powerful for complex queries.

```java
@Query("SELECT u FROM User u WHERE u.email = :email")
User findByEmail(@Param("email") String email);
```

---

**Q38. What is `application.yml` vs `application.properties`?**
Both serve the same purpose. YAML supports hierarchical structure and is more readable for nested config, while `.properties` is simpler for flat key-value pairs. Spring Boot supports both.

---

**Q39. What are Spring Profiles?**
A way to segregate configuration for different environments (dev, test, prod). Beans and properties can be conditionally loaded based on the active profile.

```properties
# application-dev.properties
spring.datasource.url=jdbc:h2:mem:devdb
```
Activate with: `spring.profiles.active=dev`

---

**Q40. What is `@Profile`?**
Marks a bean to be registered only when a specific profile is active.

```java
@Bean
@Profile("dev")
public DataSource devDataSource() { ... }
```

---

**Q41. What is `@ConfigurationProperties`?**
Binds a group of related properties from `application.properties` to a POJO.

```java
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String name;
    private int timeout;
    // getters/setters
}
```

---

**Q42. What is `@EnableConfigurationProperties`?**
Registers a `@ConfigurationProperties` class as a Spring Bean. Alternatively, you can annotate the class with `@Component`.

---

**Q43. What is the difference between `@Bean` and `@Component`?**
| `@Bean` | `@Component` |
|---------|--------------|
| Method-level annotation | Class-level annotation |
| Defined in `@Configuration` classes | Detected via component scan |
| More control over instantiation | Auto-detected |
| Used for third-party classes | Used for your own classes |

---

**Q44. What is `@Configuration`?**
Marks a class as a source of bean definitions. Methods annotated with `@Bean` inside it return beans managed by the Spring container.

---

**Q45. What is `CommandLineRunner` and `ApplicationRunner`?**
Interfaces that let you execute code after the Spring context has started. Both have a `run()` method; `ApplicationRunner` receives `ApplicationArguments` instead of raw strings.

```java
@Component
public class StartupRunner implements CommandLineRunner {
    public void run(String... args) {
        System.out.println("App started!");
    }
}
```

---

### REST & Web

**Q46. What is content negotiation in Spring Boot?**
The process by which Spring determines the response format (JSON, XML) based on the `Accept` header in the request or URL extension.

---

**Q47. How do you handle exceptions globally in Spring Boot?**
Using `@ControllerAdvice` and `@ExceptionHandler`.

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}
```

---

**Q48. What is `@ExceptionHandler`?**
Handles exceptions thrown by controller methods. Can be used locally in a controller or globally in a `@ControllerAdvice` class.

---

**Q49. What is `@ControllerAdvice`?**
A global component that applies `@ExceptionHandler`, `@ModelAttribute`, and `@InitBinder` methods across all controllers.

---

**Q50. What is `@Valid` and `@Validated`?**
Used to trigger bean validation on request bodies or method parameters. Works with JSR-303 annotations like `@NotNull`, `@Size`, `@Email`.

```java
@PostMapping("/users")
public User create(@Valid @RequestBody User user) { ... }
```

---

**Q51. What are common Bean Validation annotations?**
- `@NotNull` — field must not be null
- `@NotBlank` — string must not be blank
- `@Size(min, max)` — string/collection size range
- `@Min` / `@Max` — number range
- `@Email` — valid email format
- `@Pattern(regexp)` — regex match

---

**Q52. What is HATEOAS in Spring Boot?**
Hypermedia as the Engine of Application State — adding links to REST responses so clients can discover actions dynamically. Spring provides `spring-boot-starter-hateoas` for this.

---

**Q53. What is Spring Boot Actuator?**
A set of production-ready endpoints that expose operational information about the running application: health, metrics, environment, beans, loggers, etc.

```properties
management.endpoints.web.exposure.include=health,info,metrics
```

---

**Q54. What are common Actuator endpoints?**
- `/actuator/health` — app health status
- `/actuator/info` — app info
- `/actuator/metrics` — application metrics
- `/actuator/env` — environment properties
- `/actuator/beans` — all Spring beans
- `/actuator/loggers` — log level management

---

**Q55. How do you secure Actuator endpoints?**
By integrating Spring Security and restricting access to Actuator endpoints via security configuration or by limiting exposure in `application.properties`.

---

**Q56. What is Spring Security in Spring Boot?**
Spring Security provides authentication, authorization, and protection against common attacks (CSRF, XSS). In Spring Boot, adding `spring-boot-starter-security` auto-configures basic security with a default login page.

---

**Q57. What is `@PreAuthorize`?**
Method-level security annotation that restricts access based on roles or expressions evaluated before the method runs.

```java
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) { ... }
```

---

**Q58. What is JWT and how is it used in Spring Boot?**
JSON Web Token — a stateless token used for authentication. The client sends the JWT in the `Authorization: Bearer <token>` header. Spring Security validates it via a custom filter.

---

**Q59. What is `@EnableWebSecurity`?**
Enables Spring Security's web security support and allows customization via a `SecurityFilterChain` bean.

---

**Q60. What is CORS and how do you configure it in Spring Boot?**
Cross-Origin Resource Sharing — a browser security mechanism. Configure globally via `WebMvcConfigurer` or per-endpoint with `@CrossOrigin`.

```java
@CrossOrigin(origins = "http://localhost:3000")
@GetMapping("/data")
public Data getData() { ... }
```

---

**Q61. What is `@Async` in Spring Boot?**
Marks a method to run in a separate thread asynchronously. Requires `@EnableAsync` on a configuration class.

```java
@Async
public CompletableFuture<String> fetchData() { ... }
```

---

**Q62. What is `@Scheduled`?**
Marks a method to run on a schedule using cron expressions or fixed intervals. Requires `@EnableScheduling`.

```java
@Scheduled(fixedRate = 5000)
public void runEvery5Sec() { ... }

@Scheduled(cron = "0 0 * * * ?")
public void runHourly() { ... }
```

---

**Q63. What is Spring Cache?**
An abstraction for caching method results. Annotations like `@Cacheable`, `@CachePut`, `@CacheEvict` control caching behavior. Backed by providers like EhCache, Redis, Caffeine.

```java
@Cacheable("users")
public User findById(Long id) { ... }
```

---

**Q64. What is the difference between `@Cacheable` and `@CachePut`?**
- `@Cacheable` — returns cached result if available; skips method execution
- `@CachePut` — always executes the method and updates the cache

---

**Q65. What is an `@EventListener` in Spring Boot?**
Listens for application events published via `ApplicationEventPublisher`.

```java
@EventListener
public void handleUserCreated(UserCreatedEvent event) { ... }
```

---

## 🔴 ADVANCED (Q66–Q100)

### Internals, Architecture & Production

**Q66. What is the Spring Boot startup process?**
1. `SpringApplication.run()` is called
2. Bootstrap context is created
3. `ApplicationContext` is created and refreshed
4. Auto-configuration classes are loaded
5. Beans are instantiated and wired
6. Embedded server starts
7. `CommandLineRunner` / `ApplicationRunner` beans execute

---

**Q67. What is `ApplicationContext` vs `BeanFactory`?**
| `BeanFactory` | `ApplicationContext` |
|---------------|----------------------|
| Basic container | Extended container |
| Lazy initialization | Eager initialization by default |
| No AOP, events, i18n | Full AOP, event publishing, i18n |
| Lightweight | Preferred for most apps |

---

**Q68. What is `@Conditional` and its variants?**
Registers a bean only when certain conditions are met:
- `@ConditionalOnClass` — class is on classpath
- `@ConditionalOnMissingBean` — bean not already defined
- `@ConditionalOnProperty` — property has a specific value
- `@ConditionalOnWebApplication` — running as a web app

---

**Q69. How do you create a custom auto-configuration?**
1. Create a class annotated with `@AutoConfiguration`
2. Use `@Conditional` to guard bean registration
3. Register it in `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`

---

**Q70. What is the `BeanPostProcessor`?**
An interface that allows custom modification of beans before and after initialization. Called for every bean in the context.

```java
public class MyProcessor implements BeanPostProcessor {
    public Object postProcessAfterInitialization(Object bean, String name) {
        // modify or wrap bean
        return bean;
    }
}
```

---

**Q71. What is AOP in Spring Boot?**
Aspect-Oriented Programming separates cross-cutting concerns (logging, security, transactions) from business logic using aspects, advice, and pointcuts.

---

**Q72. What are the types of AOP advice?**
- `@Before` — runs before the method
- `@After` — runs after the method (always)
- `@AfterReturning` — runs after successful return
- `@AfterThrowing` — runs after exception
- `@Around` — wraps the method; most powerful

---

**Q73. What is a pointcut in AOP?**
An expression that selects which method executions an advice applies to.

```java
@Pointcut("execution(* com.example.service.*.*(..))")
public void serviceMethods() {}
```

---

**Q74. What is `@Around` advice?**
The most powerful advice type — controls whether the target method executes at all using `ProceedingJoinPoint.proceed()`.

```java
@Around("serviceMethods()")
public Object logTime(ProceedingJoinPoint pjp) throws Throwable {
    long start = System.currentTimeMillis();
    Object result = pjp.proceed();
    System.out.println("Time: " + (System.currentTimeMillis() - start));
    return result;
}
```

---

**Q75. What is transaction propagation?**
Defines how transactions relate to each other when methods call other transactional methods.
- `REQUIRED` (default) — join existing or create new
- `REQUIRES_NEW` — always create new, suspend existing
- `NESTED` — nested within existing
- `SUPPORTS` — join if exists, else non-transactional
- `MANDATORY` — must have existing; throws if none
- `NEVER` — must not have existing; throws if one exists

---

**Q76. What is transaction isolation?**
Controls visibility of changes made in one transaction to others. Levels: `READ_UNCOMMITTED`, `READ_COMMITTED`, `REPEATABLE_READ`, `SERIALIZABLE`.

---

**Q77. What is lazy vs eager loading in JPA?**
- **Eager** (`FetchType.EAGER`) — related entity is loaded immediately with the parent
- **Lazy** (`FetchType.LAZY`) — related entity is loaded only when accessed (default for collections)

---

**Q78. What is the N+1 problem in JPA?**
When fetching a list of N entities, JPA issues 1 query for the list and N additional queries for each associated entity — resulting in N+1 queries. Fixed with `JOIN FETCH` or `@EntityGraph`.

---

**Q79. What is `@EntityGraph`?**
Specifies which associations to eagerly load for a specific query, avoiding N+1 without changing the global fetch type.

```java
@EntityGraph(attributePaths = {"orders"})
List<User> findAll();
```

---

**Q80. What is Spring Data's `Specification` API?**
A type-safe way to build dynamic JPA queries using the Criteria API. Implement `Specification<T>` and combine with `and()`, `or()`.

---

**Q81. What is optimistic vs pessimistic locking?**
- **Optimistic**: Assumes no conflict; uses a `@Version` field to detect concurrent modifications at commit time.
- **Pessimistic**: Locks the row in the DB immediately using SQL `FOR UPDATE`.

---

**Q82. What is Spring Boot's externalized configuration order of precedence?**
From highest to lowest priority:
1. Command-line arguments
2. `SPRING_APPLICATION_JSON`
3. OS environment variables
4. `application-{profile}.properties`
5. `application.properties`
6. `@PropertySource` annotations
7. Default values

---

**Q83. What is a `HealthIndicator` in Actuator?**
A custom component that contributes to the `/actuator/health` endpoint. Implement `HealthIndicator` to add custom health checks (e.g., third-party API availability).

```java
@Component
public class MyHealthIndicator implements HealthIndicator {
    public Health health() {
        return Health.up().withDetail("service", "running").build();
    }
}
```

---

**Q84. What is Spring Boot's support for Micrometer?**
Micrometer is the metrics facade used by Spring Boot Actuator. It provides vendor-neutral APIs to collect metrics and export them to backends like Prometheus, Datadog, and Graphite.

---

**Q85. What is the difference between `@SpringBootTest` and `@WebMvcTest`?**
| `@SpringBootTest` | `@WebMvcTest` |
|-------------------|---------------|
| Loads full application context | Loads only web layer (controllers) |
| Integration test | Unit/slice test |
| Slower | Faster |
| Tests entire stack | Tests controller logic only |

---

**Q86. What is `MockMvc`?**
A Spring test utility for testing MVC controllers without starting a real HTTP server.

```java
mockMvc.perform(get("/users/1"))
       .andExpect(status().isOk())
       .andExpect(jsonPath("$.name").value("John"));
```

---

**Q87. What is `@MockBean`?**
A Spring Boot test annotation that creates a Mockito mock and adds it to the application context, replacing the real bean.

---

**Q88. What is `TestRestTemplate`?**
A test-friendly `RestTemplate` for integration tests with `@SpringBootTest`. Follows redirects and does not throw on 4xx/5xx responses.

---

**Q89. What is Spring WebFlux?**
A reactive, non-blocking web framework built on Project Reactor. An alternative to Spring MVC for building async, event-driven applications.

---

**Q90. What is the difference between Spring MVC and Spring WebFlux?**
| Spring MVC | Spring WebFlux |
|------------|----------------|
| Synchronous, blocking | Asynchronous, non-blocking |
| Servlet-based (Tomcat) | Reactive (Netty) |
| `RestTemplate` | `WebClient` |
| Thread-per-request | Event loop model |
| Better for traditional CRUD | Better for high-concurrency streaming |

---

**Q91. What is `WebClient`?**
A non-blocking, reactive HTTP client introduced in Spring 5. Preferred over `RestTemplate` for reactive applications.

```java
WebClient client = WebClient.create("https://api.example.com");
Mono<User> user = client.get().uri("/users/1").retrieve().bodyToMono(User.class);
```

---

**Q92. What is the Circuit Breaker pattern and how is it implemented in Spring Boot?**
Prevents cascading failures by stopping calls to a failing service. Spring Boot integrates with Resilience4j via `spring-boot-starter-aop` and `resilience4j-spring-boot3`.

```java
@CircuitBreaker(name = "userService", fallbackMethod = "fallback")
public User getUser(Long id) { ... }
```

---

**Q93. What is Spring Cloud?**
A suite of tools for building distributed systems and microservices: service discovery (Eureka), config server, API gateway (Spring Cloud Gateway), load balancing, distributed tracing, etc.

---

**Q94. What is `@FeignClient`?**
A declarative REST client from Spring Cloud OpenFeign. Define an interface and Spring generates the HTTP client implementation.

```java
@FeignClient(name = "user-service", url = "http://user-service")
public interface UserClient {
    @GetMapping("/users/{id}")
    User getUser(@PathVariable Long id);
}
```

---

**Q95. What is Spring Cloud Config?**
A centralized external configuration server that provides configuration to microservices from a Git repository or file system. Clients use `spring-cloud-starter-config`.

---

**Q96. What is service discovery in Spring Cloud?**
Microservices register themselves with a discovery server (Eureka). Other services look up instances by service name instead of hardcoded URLs.

---

**Q97. What is an API Gateway?**
A single entry point for all client requests. Spring Cloud Gateway provides routing, filtering, load balancing, and rate limiting.

---

**Q98. How do you containerize a Spring Boot application?**
Use a `Dockerfile` or Spring Boot's built-in Buildpacks support (`./mvnw spring-boot:build-image`).

```dockerfile
FROM eclipse-temurin:21-jre
COPY target/app.jar app.jar
ENTRYPOINT ["java", "-jar", "/app.jar"]
```

---

**Q99. What is GraalVM native image support in Spring Boot 3?**
Spring Boot 3 supports compiling to native executables using GraalVM. Native images start much faster and use less memory but require AOT (Ahead-of-Time) compilation and have some reflection limitations.

```bash
./mvnw -Pnative native:compile
```

---

**Q100. What are best practices for production-ready Spring Boot applications?**
- Use constructor injection over field injection
- Externalize all configuration (never hardcode secrets)
- Enable and secure Actuator endpoints
- Use connection pooling (HikariCP — default in Spring Boot)
- Implement global exception handling with `@ControllerAdvice`
- Add health checks and meaningful metrics
- Use profiles to separate dev/test/prod configs
- Write slice tests (`@WebMvcTest`, `@DataJpaTest`) for speed
- Enable `spring.jpa.open-in-view=false` to avoid lazy loading issues
- Use DTOs instead of exposing entities directly in APIs

---

*End of Top 100 Spring Boot Interview Q&A*

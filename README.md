# Overview
Spring Boot 4 represents a significant evolution in the Spring ecosystem, building upon the foundation of Spring Framework 6 and embracing modern Java features. 
This version focuses on improved performance, enhanced developer experience, and better support for cloud-native applications.

# What is Spring Boot
- Spring Boot is a framework that simplifies building Java applications by:
- Auto-configuring Spring components
- Providing embedded servers
- Reducing boilerplate code
- Making production-ready apps easier to build

# Key Features
- Java 17+ Requirement: Spring Boot 4 requires Java 17 as the minimum version, with full support for Java 21 LTS and newer features like virtual threads, pattern matching, and records.
- Native Compilation Support: Enhanced GraalVM native image support allows you to compile Spring Boot applications into native executables for faster startup times and reduced memory footprint.
- Observability: Built-in support for OpenTelemetry and Micrometer provides comprehensive observability out of the box, making it easier to monitor and trace your applications.
- Virtual Threads: Leverages Project Loom's virtual threads for improved concurrency handling with lower resource consumption.

# Getting Started
# Prerequisites
- Java 17 or higher (Java 21 LTS recommended)
- Maven 3.6+ or Gradle 7.5+
- Your favorite IDE (IntelliJ IDEA, Eclipse, or VS Code)

# Creating Your First Application
Using Spring Initializr, you can quickly bootstrap a new Spring Boot 4 project. Here's a simple example using Maven:
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>4.0.0</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

# Hello World Example
Create a simple REST API using modern Java features:
```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.*;

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}

@RestController
@RequestMapping("/api")
class HelloController {
    
    @GetMapping("/hello")
    public MessageResponse hello(@RequestParam(defaultValue = "World") String name) {
        return new MessageResponse("Hello, %s!".formatted(name), System.currentTimeMillis());
    }
    
    @PostMapping("/greet")
    public MessageResponse greet(@RequestBody GreetRequest request) {
        return new MessageResponse(
            "Welcome, %s %s!".formatted(request.firstName(), request.lastName()),
            System.currentTimeMillis()
        );
    }
}

// Using Java Records for DTOs
record MessageResponse(String message, long timestamp) {}
record GreetRequest(String firstName, String lastName) {}
```

# Working with Virtual Threads
Spring Boot 4 makes it easy to leverage virtual threads for improved concurrency:
```java
@Configuration
public class AsyncConfig {
    
    @Bean
    public AsyncTaskExecutor applicationTaskExecutor() {
        TaskExecutorAdapter adapter = new TaskExecutorAdapter(
            Executors.newVirtualThreadPerTaskExecutor()
        );
        adapter.setTaskDecorator(new PropagatingSecurityContextTaskDecorator());
        return adapter;
    }
}

@Service
class DataService {
    
    @Async
    public CompletableFuture<UserData> fetchUserData(String userId) {
        // This runs on a virtual thread
        return CompletableFuture.completedFuture(
            userRepository.findById(userId)
        );
    }
}
```

# Database Integration Example
Using Spring Data JPA with modern Java features:
```java
@Entity
@Table(name = "users")
class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    private String email;
    private LocalDateTime createdAt;
    
    // Getters, setters, constructors
}

interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    List<User> findByEmailContaining(String emailPart);
}

@Service
class UserService {
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public UserDTO createUser(CreateUserRequest request) {
        var user = new User();
        user.setUsername(request.username());
        user.setEmail(request.email());
        user.setCreatedAt(LocalDateTime.now());
        
        var saved = userRepository.save(user);
        return new UserDTO(saved.getId(), saved.getUsername(), saved.getEmail());
    }
}

record CreateUserRequest(String username, String email) {}
record UserDTO(Long id, String username, String email) {}
```

# Configuration Properties
Enhanced configuration with type-safe properties:
```java
@ConfigurationProperties(prefix = "app")
public record AppProperties(
    String name,
    String version,
    Security security,
    Database database
) {
    public record Security(String jwtSecret, long tokenExpiration) {}
    public record Database(int maxConnections, long timeout) {}
}

// In application.yml
app:
  name: MyApplication
  version: 1.0.0
  security:
    jwt-secret: ${JWT_SECRET}
    token-expiration: 3600000
  database:
    max-connections: 20
    timeout: 5000
```

# Native Image Compilation
Building a native executable with GraalVM:
```bash
# Using Maven
./mvnw -Pnative native:compile

# Using Gradle
./gradlew nativeCompile

# Run the native executable
./target/demo-application
```

# Observability with Micrometer
Spring Boot 4 includes enhanced observability features:
```java
@RestController
class OrderController {
    private final MeterRegistry meterRegistry;
    private final ObservationRegistry observationRegistry;
    
    @GetMapping("/orders/{id}")
    public Order getOrder(@PathVariable String id) {
        return Observation.createNotStarted("order.fetch", observationRegistry)
            .lowCardinalityKeyValue("order.id", id)
            .observe(() -> {
                meterRegistry.counter("orders.fetched").increment();
                return orderService.findById(id);
            });
    }
}
```

# Testing
Modern testing with JUnit 5 and test slices:
```java
@SpringBootTest
@AutoConfigureMockMvc
class ApplicationTests {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void shouldReturnHelloMessage() throws Exception {
        mockMvc.perform(get("/api/hello?name=Spring"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.message").value("Hello, Spring!"));
    }
}

@DataJpaTest
class UserRepositoryTests {
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    void shouldSaveAndFindUser() {
        var user = new User();
        user.setUsername("testuser");
        user.setEmail("test@example.com");
        
        userRepository.save(user);
        
        var found = userRepository.findByUsername("testuser");
        assertThat(found).isPresent();
        assertThat(found.get().getEmail()).isEqualTo("test@example.com");
    }
}
```

# Migration from Spring Boot 3
Key changes when migrating from Spring Boot 3 to Spring Boot 4 include updating your Java version to 17+, reviewing deprecated APIs that have been removed, updating third-party dependencies to compatible versions, and testing native image compilation if you plan to use it.

# Resources
- Official Documentation: https://spring.io/projects/spring-boot
- Spring Guides: https://spring.io/guides
- GitHub Repository: https://github.com/spring-projects/spring-boot
- Community Forum: https://stackoverflow.com/questions/tagged/spring-boot

# Contributing
Contributions are welcome! Please read our contributing guidelines and code of conduct before submitting pull requests.

# spring-boot-4-fundamentals
Spring Boot 4 is the next major version of Spring Boot, built on top of Spring Framework 7 and designed for modern Java development.
It focuses on performance, simplicity, and cloud-native applications.

# What is Spring Boot
- Spring Boot is a framework that simplifies building Java applications by:
- Auto-configuring Spring components
- Providing embedded servers
- Reducing boilerplate code
- Making production-ready apps easier to build

# Key Highlights of Spring Boot 4
- Requires Java 21 or higher
- Built on Spring Framework 7
- Optimized for modern Java features
- Designed for high-performance and scalable systems
- Removes legacy and deprecated APIs

# Java 21 Baseline
Spring Boot 4 fully embraces Java 21 features such as:
- Virtual Threads
- Records
- Pattern Matching
- Improved performance and memory usage

# Virtual Threads Support
Spring Boot 4 supports virtual threads, allowing applications to handle many concurrent requests efficiently.
Example:
```java
Executors.newVirtualThreadPerTaskExecutor();
```

# Configuration Improvements
Configuration is more type-safe and cleaner using records.
Example:
```java
@ConfigurationProperties(prefix = "app")
public record AppProperties(String name, int timeout) {}
```


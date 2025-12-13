---
title: Spring Data JPA Framework
date: 2025-12-12
tags:
  - computer-science/coding
  - java
  - database
  - orm
  - spring
  - jpa
draft: false
---

## Overview

Spring Data JPA is built on top of JPA and provides additional abstractions to reduce boilerplate code. It's part of the larger Spring Data family and integrates seamlessly with the Spring ecosystem.

> [!note] Relationship with Hibernate
> **Spring Data JPA uses [[Hibernate]] as its default JPA implementation.** This means:
> - All [[Hibernate]] features (HQL, Criteria API, caching) are available
> - Spring Data JPA adds convenience features on top (repository pattern, method name queries)
> - You can mix Spring Data JPA repositories with direct EntityManager usage when needed

## Key Features

- Repository pattern implementation
- Automatic query generation from method names
- Custom query support with @Query
- Pagination and sorting out of the box
- Integration with Spring ecosystem
- Auditing support
- Specifications for dynamic queries
- Projections for custom result types

## Entity Models

All examples use the `User` and `Order` entity models defined in [[Object Relational Mapping Explained#JPA Entity Classes (Hibernate, Spring Data JPA)|Entity Models]].

## Basic Repository Example

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import java.util.List;

public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByUsername(String username);

    @Query("SELECT u FROM User u WHERE u.email LIKE %:domain")
    List<User> findByEmailDomain(@Param("domain") String domain);
}
```

## Use Case Examples

### 1. Retrieve Records with Criteria

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import java.util.List;

public interface UserRepository extends JpaRepository<User, Long> {
    // Method name query
    List<User> findByStatus(String status);

    List<User> findByStatusAndEmailContaining(String status, String emailDomain);

    // Custom JPQL query
    @Query("SELECT u FROM User u WHERE u.status = :status AND u.email LIKE %:domain%")
    List<User> findActiveUsersByDomain(@Param("status") String status,
                                       @Param("domain") String domain);
}

// Usage
List<User> activeUsers = userRepository.findByStatus("ACTIVE");
```

### 2. Retrieve Records with JOIN

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import java.math.BigDecimal;
import java.util.List;

public interface UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT DISTINCT u FROM User u LEFT JOIN FETCH u.orders WHERE u.status = :status")
    List<User> findUsersWithOrders(@Param("status") String status);

    @Query("SELECT DISTINCT u FROM User u " +
           "INNER JOIN u.orders o " +
           "WHERE o.totalAmount > :amount")
    List<User> findUsersWithOrdersAbove(@Param("amount") BigDecimal amount);
}
```

### 3. Insert New Data

```java
// No additional imports needed beyond repository injection

public User createUser(UserRepository userRepository, String username, String email) {
    User user = new User();
    user.setUsername(username);
    user.setEmail(email);
    user.setStatus("ACTIVE");

    return userRepository.save(user);
}
```

### 4. Update a Record

```java
// No additional imports needed beyond repository injection

public void updateUserEmail(UserRepository userRepository, Long userId, String newEmail) {
    userRepository.findById(userId).ifPresent(user -> {
        user.setEmail(newEmail);
        userRepository.save(user);
    });
}

// Custom update query
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

public interface UserRepository extends JpaRepository<User, Long> {
    @Modifying
    @Query("UPDATE User u SET u.status = :newStatus WHERE u.status = :oldStatus")
    int updateUserStatus(@Param("oldStatus") String oldStatus,
                        @Param("newStatus") String newStatus);
}
```

### 5. Transactional Write with Multiple Operations

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.annotation.Propagation;
import java.time.LocalDateTime;
import java.util.List;

@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    @Autowired
    private OrderRepository orderRepository;

    @Transactional
    public User createUserWithOrders(String username, String email,
                                     List<OrderData> orderDataList) {
        // Create user
        User user = new User();
        user.setUsername(username);
        user.setEmail(email);
        user.setStatus("ACTIVE");
        user = userRepository.save(user);

        // Create orders
        for (OrderData orderData : orderDataList) {
            Order order = new Order();
            order.setOrderNumber(orderData.getOrderNumber());
            order.setTotalAmount(orderData.getAmount());
            order.setOrderDate(LocalDateTime.now());
            order.setUser(user);
            orderRepository.save(order);
        }

        return user;
        // Transaction commits automatically if no exception
        // Rolls back if any exception occurs
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void independentTransaction() {
        // This runs in a separate transaction
    }
}
```

### 6. Optimistic Locking

```java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.Version;
import jakarta.persistence.EntityNotFoundException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Entity
public class User {
    @Id
    @GeneratedValue
    private Long id;

    private String email;

    @Version
    private Long version;

    // getters, setters
}

@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void updateUserEmail(Long userId, String newEmail) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new EntityNotFoundException());
        user.setEmail(newEmail);
        userRepository.save(user);
        // OptimisticLockingFailureException thrown if version mismatch
    }
}
```

### 7. Pessimistic Locking

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Lock;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import jakarta.persistence.LockModeType;
import jakarta.persistence.EntityNotFoundException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;
import java.util.Optional;

public interface UserRepository extends JpaRepository<User, Long> {
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT u FROM User u WHERE u.id = :id")
    Optional<User> findByIdWithLock(@Param("id") Long id);

    @Lock(LockModeType.PESSIMISTIC_READ)
    @Query("SELECT u FROM User u WHERE u.status = :status")
    List<User> findByStatusWithReadLock(@Param("status") String status);
}

@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void updateUserWithLock(Long userId, String newEmail) {
        User user = userRepository.findByIdWithLock(userId)
            .orElseThrow(() -> new EntityNotFoundException());
        user.setEmail(newEmail);
        // Lock held until transaction commits
    }
}
```

## Bulk Operations

Spring Data JPA is convenient but not optimal for large bulk operations.

### Naive Approach (Avoid for Large Datasets)

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;

@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    // ❌ AVOID: This is very slow for large datasets
    @Transactional
    public void bulkInsertNaive(List<User> users) {
        userRepository.saveAll(users); // Slow for 10,000+ records
    }
}
```

### Optimized Approach with Chunking

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;

@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    // ✅ BETTER: Use batch configuration and chunking
    @Transactional
    public void bulkInsertOptimized(List<User> users) {
        int batchSize = 50;

        for (int i = 0; i < users.size(); i += batchSize) {
            int end = Math.min(i + batchSize, users.size());
            List<User> batch = users.subList(i, end);
            userRepository.saveAll(batch);
            userRepository.flush();
        }
    }
}
```

### Using Native Queries

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Modifying;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

public interface UserRepository extends JpaRepository<User, Long> {
    // ✅ BEST: Use native query for bulk operations
    @Modifying
    @Query(value = "INSERT INTO users (username, email, status) VALUES (:username, :email, :status)",
           nativeQuery = true)
    void insertUser(@Param("username") String username,
                   @Param("email") String email,
                   @Param("status") String status);
}
```

## Advanced Features

### Pagination and Sorting

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Sort;

public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByStatus(String status, Pageable pageable);
}

// Usage
Pageable pageable = PageRequest.of(0, 20, Sort.by("username").ascending());
Page<User> users = userRepository.findByStatus("ACTIVE", pageable);
```

### Specifications for Dynamic Queries

```java
import org.springframework.data.jpa.domain.Specification;
import java.util.List;

public class UserSpecifications {
    public static Specification<User> hasStatus(String status) {
        return (root, query, cb) -> cb.equal(root.get("status"), status);
    }

    public static Specification<User> emailContains(String email) {
        return (root, query, cb) -> cb.like(root.get("email"), "%" + email + "%");
    }
}

// Usage
Specification<User> spec = Specification
    .where(UserSpecifications.hasStatus("ACTIVE"))
    .and(UserSpecifications.emailContains("@example.com"));

List<User> users = userRepository.findAll(spec);
```

### Projections

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;
import java.util.List;

// Interface-based projection
public interface UserSummary {
    String getUsername();
    String getEmail();
}

public interface UserRepository extends JpaRepository<User, Long> {
    List<UserSummary> findByStatus(String status);
}

// Class-based projection (DTO)
public class UserDTO {
    private String username;
    private String email;

    public UserDTO(String username, String email) {
        this.username = username;
        this.email = email;
    }
    // getters, setters
}

@Query("SELECT new com.example.UserDTO(u.username, u.email) FROM User u WHERE u.status = :status")
List<UserDTO> findUserDTOsByStatus(@Param("status") String status);
```

### Auditing

```java
import jakarta.persistence.Entity;
import jakarta.persistence.EntityListeners;
import jakarta.persistence.Id;
import jakarta.persistence.GeneratedValue;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.annotation.CreatedBy;
import org.springframework.data.annotation.LastModifiedBy;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;
import org.springframework.data.domain.AuditorAware;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.core.context.SecurityContextHolder;
import java.time.LocalDateTime;
import java.util.Optional;

@Entity
@EntityListeners(AuditingEntityListener.class)
public class User {
    @Id
    @GeneratedValue
    private Long id;

    @CreatedDate
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime lastModifiedDate;

    @CreatedBy
    private String createdBy;

    @LastModifiedBy
    private String lastModifiedBy;

    // other fields
}

// Enable auditing in configuration
@Configuration
@EnableJpaAuditing
public class JpaConfig {
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.of(SecurityContextHolder.getContext()
            .getAuthentication().getName());
    }
}
```

## When to Use Spring Data JPA

- Already using Spring Framework
- Want to minimize boilerplate code
- Standard repository patterns fit your needs
- Need pagination and sorting out of the box
- Building CRUD-heavy applications
- Want automatic query generation from method names

## Performance Considerations

- Not optimal for bulk operations (10,000+ records)
- Use chunking for large datasets
- Consider native queries for complex operations
- Enable batch processing in configuration
- Use projections to fetch only needed data
- Be aware of N+1 query problems
- Use @EntityGraph or JOIN FETCH for associations

## Common Pitfalls

- Poor performance for large bulk operations
- Method name queries can become verbose
- Hidden complexity in automatic query generation
- May generate inefficient queries
- Requires understanding of underlying JPA/[[Hibernate]]

## Related

- [[Object Relational Mapping Explained]] - Overview and comparison of all ORM frameworks
- [[Hibernate]] - The underlying JPA implementation used by Spring Data JPA
- [[jOOQ]] - Better alternative for bulk operations and type-safe SQL
- [[MyBatis]] - Alternative for fine-grained SQL control

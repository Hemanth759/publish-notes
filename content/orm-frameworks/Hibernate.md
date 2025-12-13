---
title: Hibernate ORM Framework
date: 2025-12-12
tags:
  - computer-science/coding
  - java
  - database
  - orm
  - hibernate
draft: false
---

## Overview

Hibernate is the most widely used ORM framework in Java. It implements the JPA (Java Persistence API) specification and adds additional features beyond the standard.

> [!note] Relationship with Spring Data JPA
> **Spring Data JPA uses Hibernate as its default JPA implementation.** When you use Spring Data JPA, Hibernate runs under the hood doing the actual ORM work. This means:
> - All Hibernate features (HQL, Criteria API, caching, etc.) are available when using Spring Data JPA
> - You can mix Spring Data JPA repositories with direct Hibernate/EntityManager usage
> - Spring Data JPA adds convenience (repository pattern, method name queries) on top of Hibernate's capabilities
>
> See [[Spring Data JPA]] for the abstraction layer built on top of Hibernate.

## Key Features

- Full JPA implementation
- HQL (Hibernate Query Language) - object-oriented query language
- Automatic schema generation
- Caching (first-level and second-level)
- Lazy loading support
- Support for multiple database dialects
- Advanced mapping capabilities
- Batch processing support

## Entity Models

All examples use the `User` and `Order` entity models defined in [[Object Relational Mapping Explained#JPA Entity Classes (Hibernate, Spring Data JPA)|Entity Models]].

## Use Case Examples

### 1. Retrieve Records with Criteria

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.criteria.CriteriaBuilder;
import jakarta.persistence.criteria.CriteriaQuery;
import jakarta.persistence.criteria.Root;
import jakarta.persistence.criteria.Predicate;
import java.util.List;

// Using JPQL
public List<User> findActiveUsers(EntityManager em) {
    return em.createQuery(
        "SELECT u FROM User u WHERE u.status = :status", User.class)
        .setParameter("status", "ACTIVE")
        .getResultList();
}

// Using Criteria API (type-safe)
public List<User> findUsersByStatusAndEmail(EntityManager em, String status, String emailDomain) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<User> query = cb.createQuery(User.class);
    Root<User> user = query.from(User.class);

    Predicate statusPredicate = cb.equal(user.get("status"), status);
    Predicate emailPredicate = cb.like(user.get("email"), "%" + emailDomain);

    query.select(user).where(cb.and(statusPredicate, emailPredicate));

    return em.createQuery(query).getResultList();
}
```

### 2. Retrieve Records with JOIN

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.criteria.CriteriaBuilder;
import jakarta.persistence.criteria.CriteriaQuery;
import jakarta.persistence.criteria.Root;
import jakarta.persistence.criteria.Join;
import jakarta.persistence.criteria.JoinType;
import java.math.BigDecimal;
import java.util.List;

// Using JPQL with JOIN FETCH (avoids N+1 problem)
public List<User> findUsersWithOrders(EntityManager em) {
    return em.createQuery(
        "SELECT DISTINCT u FROM User u " +
        "LEFT JOIN FETCH u.orders o " +
        "WHERE u.status = :status", User.class)
        .setParameter("status", "ACTIVE")
        .getResultList();
}

// Using Criteria API
public List<User> findUsersWithOrdersAboveAmount(EntityManager em, BigDecimal amount) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<User> query = cb.createQuery(User.class);
    Root<User> user = query.from(User.class);
    Join<User, Order> orders = user.join("orders", JoinType.INNER);

    query.select(user)
         .distinct(true)
         .where(cb.greaterThan(orders.get("totalAmount"), amount));

    return em.createQuery(query).getResultList();
}
```

### 3. Insert New Data

```java
import jakarta.persistence.EntityManager;

public User createUser(EntityManager em, String username, String email) {
    User user = new User();
    user.setUsername(username);
    user.setEmail(email);
    user.setStatus("ACTIVE");

    em.persist(user);
    return user; // ID will be populated after flush
}
```

### 4. Update a Record

```java
import jakarta.persistence.EntityManager;

public void updateUserEmail(EntityManager em, Long userId, String newEmail) {
    User user = em.find(User.class, userId);
    if (user != null) {
        user.setEmail(newEmail);
        // No explicit update call needed - changes tracked automatically
    }
}

// Or using JPQL for bulk update
public int updateUserStatus(EntityManager em, String oldStatus, String newStatus) {
    return em.createQuery(
        "UPDATE User u SET u.status = :newStatus WHERE u.status = :oldStatus")
        .setParameter("newStatus", newStatus)
        .setParameter("oldStatus", oldStatus)
        .executeUpdate();
}
```

### 5. Transactional Write with Multiple Operations

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityTransaction;
import org.springframework.transaction.annotation.Transactional;
import java.time.LocalDateTime;
import java.util.List;

@Transactional
public void createUserWithOrders(EntityManager em, String username, String email,
                                 List<OrderData> orderDataList) {
    // Create user
    User user = new User();
    user.setUsername(username);
    user.setEmail(email);
    user.setStatus("ACTIVE");
    em.persist(user);

    // Create orders
    for (OrderData orderData : orderDataList) {
        Order order = new Order();
        order.setOrderNumber(orderData.getOrderNumber());
        order.setTotalAmount(orderData.getAmount());
        order.setOrderDate(LocalDateTime.now());
        order.setUser(user);
        em.persist(order);
    }

    // If any exception occurs, entire transaction rolls back
}

// Manual transaction management
public void createUserWithOrdersManual(EntityManager em, String username, String email,
                                       List<OrderData> orderDataList) {
    EntityTransaction tx = em.getTransaction();
    try {
        tx.begin();

        User user = new User();
        user.setUsername(username);
        user.setEmail(email);
        user.setStatus("ACTIVE");
        em.persist(user);

        for (OrderData orderData : orderDataList) {
            Order order = new Order();
            order.setOrderNumber(orderData.getOrderNumber());
            order.setTotalAmount(orderData.getAmount());
            order.setOrderDate(LocalDateTime.now());
            order.setUser(user);
            em.persist(order);
        }

        tx.commit();
    } catch (Exception e) {
        if (tx.isActive()) {
            tx.rollback();
        }
        throw e;
    }
}
```

### 6. Optimistic Locking

```java
import jakarta.persistence.Entity;
import jakarta.persistence.Id;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.Version;
import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityTransaction;
import jakarta.persistence.OptimisticLockException;
import org.springframework.transaction.annotation.Transactional;

@Entity
public class User {
    @Id
    @GeneratedValue
    private Long id;

    private String email;

    @Version
    private Long version; // Automatically managed by JPA

    // getters, setters
}

// Usage
@Transactional
public void updateUserEmailOptimistic(EntityManager em, Long userId, String newEmail) {
    User user = em.find(User.class, userId);
    user.setEmail(newEmail);
    // On commit, JPA checks if version matches
    // If version changed, throws OptimisticLockException
}

// Handling optimistic lock exception
public void updateWithRetry(EntityManager em, Long userId, String newEmail) {
    int maxRetries = 3;
    int attempt = 0;

    while (attempt < maxRetries) {
        try {
            EntityTransaction tx = em.getTransaction();
            tx.begin();

            User user = em.find(User.class, userId);
            user.setEmail(newEmail);

            tx.commit();
            return; // Success
        } catch (OptimisticLockException e) {
            attempt++;
            if (attempt >= maxRetries) {
                throw new RuntimeException("Failed after " + maxRetries + " attempts", e);
            }
            // Retry with fresh data
        }
    }
}
```

### 7. Pessimistic Locking

```java
import jakarta.persistence.EntityManager;
import jakarta.persistence.LockModeType;
import org.springframework.transaction.annotation.Transactional;
import java.util.HashMap;
import java.util.Map;

// Pessimistic Read Lock (shared lock)
@Transactional
public User getUserWithReadLock(EntityManager em, Long userId) {
    return em.find(User.class, userId, LockModeType.PESSIMISTIC_READ);
    // Other transactions can read but not modify
}

// Pessimistic Write Lock (exclusive lock)
@Transactional
public void updateUserWithWriteLock(EntityManager em, Long userId, String newEmail) {
    User user = em.find(User.class, userId, LockModeType.PESSIMISTIC_WRITE);
    // Row is locked, other transactions must wait
    user.setEmail(newEmail);
    // Lock released on commit
}

// Using JPQL with lock
public User getUserWithLock(EntityManager em, Long userId) {
    return em.createQuery("SELECT u FROM User u WHERE u.id = :id", User.class)
        .setParameter("id", userId)
        .setLockMode(LockModeType.PESSIMISTIC_WRITE)
        .getSingleResult();
}

// Lock with timeout
public User getUserWithLockTimeout(EntityManager em, Long userId) {
    Map<String, Object> properties = new HashMap<>();
    properties.put("javax.persistence.lock.timeout", 5000); // 5 seconds

    return em.find(User.class, userId, LockModeType.PESSIMISTIC_WRITE, properties);
}
```

## Bulk Operations

Hibernate can perform well for bulk operations with proper configuration.

### Configuration

```properties
# hibernate.properties or application.properties
hibernate.jdbc.batch_size=50
hibernate.order_inserts=true
hibernate.order_updates=true
hibernate.jdbc.batch_versioned_data=true
```

### Bulk Insert

```java
import jakarta.persistence.EntityManager;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;

@Transactional
public void bulkInsertWithHibernate(EntityManager em, List<User> users) {
    int batchSize = 50;

    for (int i = 0; i < users.size(); i++) {
        em.persist(users.get(i));

        if (i > 0 && i % batchSize == 0) {
            // Flush and clear to avoid memory issues
            em.flush();
            em.clear();
        }
    }

    em.flush();
    em.clear();
}
```

### Bulk Update

```java
import jakarta.persistence.EntityManager;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;

@Transactional
public void bulkUpdateWithHibernate(EntityManager em, List<User> users) {
    int batchSize = 50;

    for (int i = 0; i < users.size(); i++) {
        User user = users.get(i);
        User managed = em.find(User.class, user.getId());
        if (managed != null) {
            managed.setEmail(user.getEmail());
            managed.setStatus(user.getStatus());
        }

        if (i > 0 && i % batchSize == 0) {
            em.flush();
            em.clear();
        }
    }

    em.flush();
    em.clear();
}

// For pure bulk updates without loading entities
@Transactional
public int bulkUpdateStatus(EntityManager em, String oldStatus, String newStatus) {
    return em.createQuery(
        "UPDATE User u SET u.status = :newStatus WHERE u.status = :oldStatus")
        .setParameter("newStatus", newStatus)
        .setParameter("oldStatus", oldStatus)
        .executeUpdate();
}
```

## Common Pitfalls

- Must configure batch size properly
- Must flush and clear periodically to avoid OutOfMemoryError
- Slower than jOOQ/MyBatis for bulk operations
- First-level cache can cause memory issues
- Requires understanding of entity lifecycle
- N+1 query problem if not using JOIN FETCH

## When to Use Hibernate

- Building standard CRUD applications
- You want database independence
- Team is familiar with ORM concepts
- Rapid development is priority
- Need advanced caching strategies
- Working with complex object graphs

## Performance Considerations

- Configure batch processing for bulk operations
- Use JOIN FETCH to avoid N+1 queries
- Enable second-level cache for read-heavy workloads
- Monitor and tune query performance
- Consider using native queries for complex operations
- Flush and clear EntityManager periodically in batch operations
- For bulk operations (10,000+ records), consider [[jOOQ]] or [[MyBatis]] for better performance

## Related

- [[Object Relational Mapping Explained]] - Overview and comparison of all ORM frameworks
- [[Spring Data JPA]] - Abstraction layer built on top of Hibernate
- [[jOOQ]] - Alternative for type-safe SQL and bulk operations
- [[MyBatis]] - Alternative for fine-grained SQL control

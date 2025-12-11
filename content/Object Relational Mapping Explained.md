---
title: Object Relational Mapping Explained
date: 2025-12-12
permalink:
aliases:
  - ORM
tags:
  - computer-science/coding
  - java
  - database
draft: false
---

## What is Object Relational Mapping (ORM)?

Object Relational Mapping (ORM) is a programming technique that converts data between incompatible type systems in object-oriented programming languages. It creates a "virtual object database" that can be used from within the programming language.

### The Problem ORM Solves

Relational databases store data in tables with rows and columns, while object-oriented programs work with objects and classes. This creates an impedance mismatch:

- **Databases**: Tables, rows, columns, foreign keys, SQL queries
- **Objects**: Classes, instances, properties, references, method calls

ORM frameworks bridge this gap by automatically handling the conversion between database records and programming objects.

### Core Concepts

**Entity**: A class that maps to a database table. Each instance represents a row.

**Mapping**: Configuration that defines how object properties correspond to database columns.

**Session/Context**: Manages the connection between your application and the database, tracking changes to objects.

**Query Language**: Most ORMs provide an object-oriented way to query data instead of writing raw SQL.

## Java ORM Frameworks

### 1. Hibernate

The most widely used ORM framework in Java. It implements the JPA (Java Persistence API) specification and adds additional features.

**Key Features**:
- Full JPA implementation
- HQL (Hibernate Query Language) - object-oriented query language
- Automatic schema generation
- Caching (first-level and second-level)
- Lazy loading support
- Support for multiple database dialects

**Example**:
```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "username")
    private String username;

    @OneToMany(mappedBy = "user")
    private List<Order> orders;
}
```

### 2. JPA (Java Persistence API)

JPA is not a framework itself but a specification. It defines a standard way to manage relational data in Java applications. Hibernate, EclipseLink, and others are implementations of JPA.

**Key Features**:
- Standard API across different implementations
- Annotation-based mapping
- JPQL (Java Persistence Query Language)
- Entity lifecycle management
- Criteria API for type-safe queries

### 3. EclipseLink

The reference implementation of JPA, originally developed by Oracle as TopLink.

**Key Features**:
- Full JPA implementation
- Advanced caching strategies
- Support for NoSQL databases
- Multi-tenancy support
- Object-XML mapping

### 4. MyBatis

A persistence framework that takes a different approach - it's a SQL mapper rather than a full ORM.

**Key Features**:
- Maps SQL queries to Java methods
- More control over SQL compared to full ORMs
- XML or annotation-based configuration
- Dynamic SQL generation
- Simpler learning curve

**Example**:
```java
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM users WHERE id = #{id}")
    User findById(Long id);

    @Insert("INSERT INTO users(username, email) VALUES(#{username}, #{email})")
    void insert(User user);
}
```

### 5. Spring Data JPA

Built on top of JPA, provides additional abstractions and reduces boilerplate code.

**Key Features**:
- Repository pattern implementation
- Automatic query generation from method names
- Custom query support with @Query
- Pagination and sorting
- Integration with Spring ecosystem

**Example**:
```java
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByUsername(String username);

    @Query("SELECT u FROM User u WHERE u.email LIKE %:domain")
    List<User> findByEmailDomain(@Param("domain") String domain);
}
```

### 6. jOOQ (Java Object Oriented Querying)

A type-safe SQL builder that generates Java code from your database schema.

**Key Features**:
- Type-safe SQL queries
- Code generation from database schema
- Full SQL support including advanced features
- Compile-time query validation
- Works alongside other ORMs

### 7. QueryDSL

Provides a type-safe way to write queries for multiple backends including JPA, SQL, MongoDB.

**Key Features**:
- Type-safe query construction
- Works with JPA, SQL, MongoDB, Lucene
- Fluent API
- Compile-time validation

## Comparison Summary

| Framework | Type | Learning Curve | SQL Control | Performance |
|-----------|------|----------------|-------------|-------------|
| Hibernate | Full ORM | Steep | Low | Good with tuning |
| JPA | Specification | Moderate | Low | Depends on implementation |
| EclipseLink | Full ORM | Moderate | Low | Good |
| MyBatis | SQL Mapper | Gentle | High | Excellent |
| Spring Data JPA | ORM Abstraction | Gentle | Low-Medium | Good |
| jOOQ | SQL Builder | Moderate | High | Excellent |
| QueryDSL | Query Builder | Moderate | Medium | Good |

## When to Use What

**Use Hibernate/JPA when**:
- Building standard CRUD applications
- You want database independence
- Team is familiar with ORM concepts
- Rapid development is priority

**Use MyBatis when**:
- You need fine-grained SQL control
- Working with complex legacy databases
- Performance is critical
- Team prefers SQL over ORM abstractions

**Use Spring Data JPA when**:
- Already using Spring Framework
- Want to minimize boilerplate code
- Standard repository patterns fit your needs

**Use jOOQ when**:
- You need type-safe SQL
- Working with complex SQL queries
- Want compile-time query validation
- Database-first approach

## Common Use Cases with Code Examples

### Entity Models

First, let's define the entity models we'll use across all examples:

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "username", nullable = false)
    private String username;

    @Column(name = "email", nullable = false)
    private String email;

    @Column(name = "status")
    private String status;

    @Version
    private Long version; // For optimistic locking

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Order> orders = new ArrayList<>();

    // Constructors, getters, setters
}

@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "order_number")
    private String orderNumber;

    @Column(name = "total_amount")
    private BigDecimal totalAmount;

    @Column(name = "order_date")
    private LocalDateTime orderDate;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;

    @Version
    private Long version;

    // Constructors, getters, setters
}
```

### Use Case 1: Retrieve Records with Criteria (WHERE clause)

#### Hibernate/JPA
```java
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

#### Spring Data JPA
```java
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

#### MyBatis
```java
// Mapper interface
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM users WHERE status = #{status}")
    List<User> findByStatus(@Param("status") String status);

    @Select("SELECT * FROM users WHERE status = #{status} AND email LIKE CONCAT('%', #{domain}, '%')")
    List<User> findByStatusAndEmailDomain(@Param("status") String status,
                                          @Param("domain") String domain);
}

// Or using XML mapper
// UserMapper.xml
<select id="findByStatus" resultType="User">
    SELECT * FROM users WHERE status = #{status}
</select>
```

#### jOOQ
```java
public List<User> findActiveUsers(DSLContext dsl) {
    return dsl.selectFrom(USERS)
        .where(USERS.STATUS.eq("ACTIVE"))
        .fetchInto(User.class);
}

public List<User> findUsersByStatusAndEmail(DSLContext dsl, String status, String domain) {
    return dsl.selectFrom(USERS)
        .where(USERS.STATUS.eq(status)
            .and(USERS.EMAIL.like("%" + domain + "%")))
        .fetchInto(User.class);
}
```

#### QueryDSL
```java
public List<User> findActiveUsers(JPAQueryFactory queryFactory) {
    QUser user = QUser.user;

    return queryFactory.selectFrom(user)
        .where(user.status.eq("ACTIVE"))
        .fetch();
}

public List<User> findUsersByStatusAndEmail(JPAQueryFactory queryFactory,
                                            String status, String domain) {
    QUser user = QUser.user;

    return queryFactory.selectFrom(user)
        .where(user.status.eq(status)
            .and(user.email.contains(domain)))
        .fetch();
}
```

### Use Case 2: Retrieve Records with JOIN

#### Hibernate/JPA
```java
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

#### Spring Data JPA
```java
public interface UserRepository extends JpaRepository<User, Long> {
    @Query("SELECT DISTINCT u FROM User u LEFT JOIN FETCH u.orders WHERE u.status = :status")
    List<User> findUsersWithOrders(@Param("status") String status);

    @Query("SELECT DISTINCT u FROM User u " +
           "INNER JOIN u.orders o " +
           "WHERE o.totalAmount > :amount")
    List<User> findUsersWithOrdersAbove(@Param("amount") BigDecimal amount);
}
```

#### MyBatis
```java
// Mapper interface
@Mapper
public interface UserMapper {
    @Select("SELECT u.*, o.id as order_id, o.order_number, o.total_amount, o.order_date " +
            "FROM users u " +
            "LEFT JOIN orders o ON u.id = o.user_id " +
            "WHERE u.status = #{status}")
    @Results({
        @Result(property = "id", column = "id"),
        @Result(property = "username", column = "username"),
        @Result(property = "orders", javaType = List.class,
                column = "id",
                many = @Many(select = "findOrdersByUserId"))
    })
    List<User> findUsersWithOrders(@Param("status") String status);
}

// Or using XML with nested results
// UserMapper.xml
<resultMap id="UserWithOrdersMap" type="User">
    <id property="id" column="id"/>
    <result property="username" column="username"/>
    <collection property="orders" ofType="Order">
        <id property="id" column="order_id"/>
        <result property="orderNumber" column="order_number"/>
        <result property="totalAmount" column="total_amount"/>
    </collection>
</resultMap>

<select id="findUsersWithOrders" resultMap="UserWithOrdersMap">
    SELECT u.*, o.id as order_id, o.order_number, o.total_amount
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    WHERE u.status = #{status}
</select>
```

#### jOOQ
```java
public Map<User, List<Order>> findUsersWithOrders(DSLContext dsl) {
    return dsl.select()
        .from(USERS)
        .leftJoin(ORDERS).on(USERS.ID.eq(ORDERS.USER_ID))
        .where(USERS.STATUS.eq("ACTIVE"))
        .fetchGroups(
            r -> r.into(USERS).into(User.class),
            r -> r.into(ORDERS).into(Order.class)
        );
}
```

#### QueryDSL
```java
public List<User> findUsersWithOrders(JPAQueryFactory queryFactory) {
    QUser user = QUser.user;
    QOrder order = QOrder.order;

    return queryFactory.selectFrom(user)
        .distinct()
        .leftJoin(user.orders, order).fetchJoin()
        .where(user.status.eq("ACTIVE"))
        .fetch();
}
```

### Use Case 3: Insert New Data

#### Hibernate/JPA
```java
public User createUser(EntityManager em, String username, String email) {
    User user = new User();
    user.setUsername(username);
    user.setEmail(email);
    user.setStatus("ACTIVE");

    em.persist(user);
    return user; // ID will be populated after flush
}
```

#### Spring Data JPA
```java
public User createUser(UserRepository userRepository, String username, String email) {
    User user = new User();
    user.setUsername(username);
    user.setEmail(email);
    user.setStatus("ACTIVE");

    return userRepository.save(user);
}
```

#### MyBatis
```java
@Mapper
public interface UserMapper {
    @Insert("INSERT INTO users(username, email, status) VALUES(#{username}, #{email}, #{status})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    void insert(User user);
}

// Usage
User user = new User();
user.setUsername("john");
user.setEmail("john@example.com");
user.setStatus("ACTIVE");
userMapper.insert(user);
// user.getId() now contains the generated ID
```

#### jOOQ
```java
public User createUser(DSLContext dsl, String username, String email) {
    UsersRecord record = dsl.insertInto(USERS)
        .set(USERS.USERNAME, username)
        .set(USERS.EMAIL, email)
        .set(USERS.STATUS, "ACTIVE")
        .returning()
        .fetchOne();

    return record.into(User.class);
}
```

### Use Case 4: Update a Record

#### Hibernate/JPA
```java
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

#### Spring Data JPA
```java
public void updateUserEmail(UserRepository userRepository, Long userId, String newEmail) {
    userRepository.findById(userId).ifPresent(user -> {
        user.setEmail(newEmail);
        userRepository.save(user);
    });
}

// Custom update query
public interface UserRepository extends JpaRepository<User, Long> {
    @Modifying
    @Query("UPDATE User u SET u.status = :newStatus WHERE u.status = :oldStatus")
    int updateUserStatus(@Param("oldStatus") String oldStatus,
                        @Param("newStatus") String newStatus);
}
```

#### MyBatis
```java
@Mapper
public interface UserMapper {
    @Update("UPDATE users SET email = #{email} WHERE id = #{id}")
    int updateEmail(@Param("id") Long id, @Param("email") String email);

    @Update("UPDATE users SET status = #{newStatus} WHERE status = #{oldStatus}")
    int updateStatus(@Param("oldStatus") String oldStatus,
                    @Param("newStatus") String newStatus);
}
```

#### jOOQ
```java
public int updateUserEmail(DSLContext dsl, Long userId, String newEmail) {
    return dsl.update(USERS)
        .set(USERS.EMAIL, newEmail)
        .where(USERS.ID.eq(userId))
        .execute();
}
```

### Use Case 5: Transactional Write with Multiple Operations

#### Hibernate/JPA
```java
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

#### Spring Data JPA
```java
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

#### MyBatis
```java
// Using SqlSession
public void createUserWithOrders(SqlSessionFactory sessionFactory,
                                String username, String email,
                                List<OrderData> orderDataList) {
    try (SqlSession session = sessionFactory.openSession()) {
        try {
            UserMapper userMapper = session.getMapper(UserMapper.class);
            OrderMapper orderMapper = session.getMapper(OrderMapper.class);

            // Create user
            User user = new User();
            user.setUsername(username);
            user.setEmail(email);
            user.setStatus("ACTIVE");
            userMapper.insert(user);

            // Create orders
            for (OrderData orderData : orderDataList) {
                Order order = new Order();
                order.setOrderNumber(orderData.getOrderNumber());
                order.setTotalAmount(orderData.getAmount());
                order.setOrderDate(LocalDateTime.now());
                order.setUserId(user.getId());
                orderMapper.insert(order);
            }

            session.commit();
        } catch (Exception e) {
            session.rollback();
            throw e;
        }
    }
}

// With Spring @Transactional
@Service
public class UserService {
    @Autowired
    private UserMapper userMapper;

    @Autowired
    private OrderMapper orderMapper;

    @Transactional
    public void createUserWithOrders(String username, String email,
                                     List<OrderData> orderDataList) {
        User user = new User();
        user.setUsername(username);
        user.setEmail(email);
        user.setStatus("ACTIVE");
        userMapper.insert(user);

        for (OrderData orderData : orderDataList) {
            Order order = new Order();
            order.setOrderNumber(orderData.getOrderNumber());
            order.setTotalAmount(orderData.getAmount());
            order.setOrderDate(LocalDateTime.now());
            order.setUserId(user.getId());
            orderMapper.insert(order);
        }
    }
}
```

#### jOOQ
```java
public void createUserWithOrders(DSLContext dsl, String username, String email,
                                List<OrderData> orderDataList) {
    dsl.transaction(configuration -> {
        DSLContext ctx = DSL.using(configuration);

        // Insert user
        UsersRecord userRecord = ctx.insertInto(USERS)
            .set(USERS.USERNAME, username)
            .set(USERS.EMAIL, email)
            .set(USERS.STATUS, "ACTIVE")
            .returning()
            .fetchOne();

        Long userId = userRecord.getId();

        // Insert orders
        for (OrderData orderData : orderDataList) {
            ctx.insertInto(ORDERS)
                .set(ORDERS.ORDER_NUMBER, orderData.getOrderNumber())
                .set(ORDERS.TOTAL_AMOUNT, orderData.getAmount())
                .set(ORDERS.ORDER_DATE, LocalDateTime.now())
                .set(ORDERS.USER_ID, userId)
                .execute();
        }

        // Transaction commits automatically if no exception
        // Rolls back if exception occurs
    });
}
```

### Use Case 6: Optimistic and Pessimistic Locking

#### Optimistic Locking

Optimistic locking assumes conflicts are rare. It uses a version field to detect concurrent modifications.

**Hibernate/JPA**
```java
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

**Spring Data JPA**
```java
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

**MyBatis**
```java
// Manual optimistic locking implementation
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM users WHERE id = #{id}")
    User findById(Long id);

    @Update("UPDATE users SET email = #{email}, version = version + 1 " +
            "WHERE id = #{id} AND version = #{version}")
    int updateWithVersion(User user);
}

// Usage
public void updateUserEmailOptimistic(UserMapper mapper, Long userId, String newEmail) {
    User user = mapper.findById(userId);
    Long originalVersion = user.getVersion();
    user.setEmail(newEmail);

    int rowsUpdated = mapper.updateWithVersion(user);
    if (rowsUpdated == 0) {
        throw new OptimisticLockException("User was modified by another transaction");
    }
}
```

**jOOQ**
```java
// Using optimistic locking with version field
public void updateUserEmailOptimistic(DSLContext dsl, Long userId, String newEmail) {
    UsersRecord user = dsl.selectFrom(USERS)
        .where(USERS.ID.eq(userId))
        .fetchOne();

    if (user != null) {
        Long originalVersion = user.getVersion();

        int rowsUpdated = dsl.update(USERS)
            .set(USERS.EMAIL, newEmail)
            .set(USERS.VERSION, USERS.VERSION.add(1))
            .where(USERS.ID.eq(userId)
                .and(USERS.VERSION.eq(originalVersion)))
            .execute();

        if (rowsUpdated == 0) {
            throw new OptimisticLockException("User was modified by another transaction");
        }
    }
}
```

#### Pessimistic Locking

Pessimistic locking assumes conflicts are likely. It locks the row immediately when reading.

**Hibernate/JPA**
```java
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

**Spring Data JPA**
```java
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

**MyBatis**
```java
// Pessimistic locking with SELECT FOR UPDATE
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM users WHERE id = #{id} FOR UPDATE")
    User findByIdForUpdate(Long id);

    @Select("SELECT * FROM users WHERE id = #{id} LOCK IN SHARE MODE")
    User findByIdWithSharedLock(Long id);
}

// Usage with transaction
public void updateUserWithLock(SqlSessionFactory sessionFactory,
                               Long userId, String newEmail) {
    try (SqlSession session = sessionFactory.openSession()) {
        try {
            UserMapper mapper = session.getMapper(UserMapper.class);

            // Acquire lock
            User user = mapper.findByIdForUpdate(userId);
            user.setEmail(newEmail);
            mapper.update(user);

            session.commit();
            // Lock released on commit
        } catch (Exception e) {
            session.rollback();
            throw e;
        }
    }
}
```

**jOOQ**
```java
// Pessimistic write lock
public void updateUserWithLock(DSLContext dsl, Long userId, String newEmail) {
    dsl.transaction(configuration -> {
        DSLContext ctx = DSL.using(configuration);

        // SELECT FOR UPDATE
        UsersRecord user = ctx.selectFrom(USERS)
            .where(USERS.ID.eq(userId))
            .forUpdate()
            .fetchOne();

        if (user != null) {
            user.setEmail(newEmail);
            user.update();
        }
        // Lock released on commit
    });
}

// Pessimistic read lock (shared lock)
public User getUserWithSharedLock(DSLContext dsl, Long userId) {
    return dsl.selectFrom(USERS)
        .where(USERS.ID.eq(userId))
        .forShare()
        .fetchOneInto(User.class);
}

// Lock with NOWAIT (fail immediately if locked)
public User getUserWithNoWait(DSLContext dsl, Long userId) {
    return dsl.selectFrom(USERS)
        .where(USERS.ID.eq(userId))
        .forUpdate()
        .noWait()
        .fetchOneInto(User.class);
}

// Lock with SKIP LOCKED (skip locked rows)
public List<User> getAvailableUsers(DSLContext dsl) {
    return dsl.selectFrom(USERS)
        .where(USERS.STATUS.eq("PENDING"))
        .forUpdate()
        .skipLocked()
        .fetchInto(User.class);
}
```

### Locking Comparison

| Lock Type | When to Use | Pros | Cons |
|-----------|-------------|------|------|
| Optimistic | Low contention scenarios | Better performance, no blocking | Requires retry logic, can fail |
| Pessimistic Read | Need consistent read across transaction | Prevents modifications | Can cause deadlocks |
| Pessimistic Write | High contention, critical updates | Guarantees exclusive access | Reduces concurrency, can cause deadlocks |

## Common Limitations and Challenges

*To be documented as we progress through discussions*

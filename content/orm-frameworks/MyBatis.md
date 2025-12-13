---
title: MyBatis Framework
date: 2025-12-12
tags:
  - computer-science/coding
  - java
  - database
  - orm
  - mybatis
draft: false
---

## Overview

MyBatis is a persistence framework that takes a different approach - it's a SQL mapper rather than a full ORM. It provides more control over SQL while still mapping results to Java objects.

## Key Features

- Maps SQL queries to Java methods
- More control over SQL compared to full ORMs
- XML or annotation-based configuration
- Dynamic SQL generation
- Simpler learning curve
- Excellent performance
- Support for stored procedures
- Flexible result mapping

## Entity Models

> [!important] How MyBatis Differs from JPA Frameworks
> Unlike [[Hibernate]] and [[Spring Data JPA]], MyBatis does **not** use JPA annotations. It works with plain POJOs and handles all mapping through:
> - **XML result maps** - Define how columns map to properties
> - **`@Results` annotations** - Inline mapping definitions
> - **Convention** - Automatic mapping when column names match property names
>
> This means:
> - No `@Entity`, `@Table`, `@Column` annotations needed
> - No automatic relationship management (`@OneToMany`, `@ManyToOne`)
> - No automatic dirty checking or change tracking
> - You write explicit SQL for all operations

### User Entity (Plain POJO)

```java
import java.util.List;

public class User {
    private Long id;
    private String username;
    private String email;
    private String status;
    private Long version;

    // For nested results (optional)
    private List<Order> orders;

    // Default constructor required
    public User() {}

    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public String getStatus() { return status; }
    public void setStatus(String status) { this.status = status; }

    public Long getVersion() { return version; }
    public void setVersion(Long version) { this.version = version; }

    public List<Order> getOrders() { return orders; }
    public void setOrders(List<Order> orders) { this.orders = orders; }
}
```

### Order Entity (Plain POJO)

```java
import java.math.BigDecimal;
import java.time.LocalDateTime;

public class Order {
    private Long id;
    private String orderNumber;
    private BigDecimal totalAmount;
    private LocalDateTime orderDate;
    private Long userId;  // Foreign key stored as simple field, NOT object reference
    private Long version;

    // Default constructor required
    public Order() {}

    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getOrderNumber() { return orderNumber; }
    public void setOrderNumber(String orderNumber) { this.orderNumber = orderNumber; }

    public BigDecimal getTotalAmount() { return totalAmount; }
    public void setTotalAmount(BigDecimal totalAmount) { this.totalAmount = totalAmount; }

    public LocalDateTime getOrderDate() { return orderDate; }
    public void setOrderDate(LocalDateTime orderDate) { this.orderDate = orderDate; }

    public Long getUserId() { return userId; }  // Note: Long, not User object
    public void setUserId(Long userId) { this.userId = userId; }

    public Long getVersion() { return version; }
    public void setVersion(Long version) { this.version = version; }
}
```

### Key Differences from JPA

| Aspect | JPA (Hibernate/Spring Data JPA) | MyBatis |
|--------|--------------------------------|---------|
| Entity Definition | `@Entity` annotation required | Plain POJO, no annotations |
| Column Mapping | `@Column(name = "...")` | XML result map or convention |
| Primary Key | `@Id` + `@GeneratedValue` | `@Options(useGeneratedKeys = true)` on INSERT |
| Relationships | `@OneToMany`, `@ManyToOne` with object references | Manual JOIN + nested result maps |
| Foreign Keys | Object reference (`private User user`) | Simple field (`private Long userId`) |
| Lazy Loading | Automatic with `FetchType.LAZY` | Manual with separate queries |
| Change Tracking | Automatic dirty checking | None - explicit UPDATE required |

### Handling Relationships in MyBatis

Since MyBatis doesn't manage relationships automatically, you handle them explicitly:

```java
// Option 1: Store foreign key as simple field
public class Order {
    private Long userId;  // Just the ID
}

// Option 2: Include related object for read operations
public class Order {
    private Long userId;
    private User user;  // Populated via JOIN query + result map
}
```

## Basic Mapper Example

```java
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Options;

@Mapper
public interface UserMapper {
    @Select("SELECT * FROM users WHERE id = #{id}")
    User findById(Long id);

    @Insert("INSERT INTO users(username, email, status) VALUES(#{username}, #{email}, #{status})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    void insert(User user);
}
```

## Use Case Examples

### 1. Retrieve Records with Criteria

```java
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Param;
import java.util.List;

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

### 2. Retrieve Records with JOIN

```java
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Results;
import org.apache.ibatis.annotations.Result;
import org.apache.ibatis.annotations.Many;
import org.apache.ibatis.annotations.Param;
import java.util.List;

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

### 3. Insert New Data

```java
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Options;

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

### 4. Update a Record

```java
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Update;
import org.apache.ibatis.annotations.Param;

@Mapper
public interface UserMapper {
    @Update("UPDATE users SET email = #{email} WHERE id = #{id}")
    int updateEmail(@Param("id") Long id, @Param("email") String email);

    @Update("UPDATE users SET status = #{newStatus} WHERE status = #{oldStatus}")
    int updateStatus(@Param("oldStatus") String oldStatus,
                    @Param("newStatus") String newStatus);
}
```

### 5. Transactional Write with Multiple Operations

```java
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.time.LocalDateTime;
import java.util.List;

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

### 6. Optimistic Locking

```java
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;
import javax.persistence.OptimisticLockException;

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

### 7. Pessimistic Locking

```java
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;

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

## Bulk Operations

MyBatis provides excellent performance for bulk operations.

### Batch Insert

```java
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import java.util.List;

// Mapper interface
@Mapper
public interface UserMapper {
    void batchInsert(@Param("users") List<User> users);
}
```

```xml
<!-- UserMapper.xml -->
<insert id="batchInsert" parameterType="java.util.List">
    INSERT INTO users (username, email, status) VALUES
    <foreach collection="users" item="user" separator=",">
        (#{user.username}, #{user.email}, #{user.status})
    </foreach>
</insert>
```

### Batch Update

```xml
<update id="batchUpdate" parameterType="java.util.List">
    <foreach collection="users" item="user" separator=";">
        UPDATE users
        SET email = #{user.email}, status = #{user.status}
        WHERE id = #{user.id}
    </foreach>
</update>
```

### Usage with Batch Executor

```java
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.ExecutorType;
import java.util.List;

public void bulkOperations(SqlSessionFactory sessionFactory,
                          List<User> usersToInsert,
                          List<User> usersToUpdate) {
    // Configure batch executor
    try (SqlSession session = sessionFactory.openSession(ExecutorType.BATCH)) {
        UserMapper mapper = session.getMapper(UserMapper.class);

        // Process in chunks to avoid memory issues
        int chunkSize = 1000;

        // Batch inserts
        for (int i = 0; i < usersToInsert.size(); i += chunkSize) {
            int end = Math.min(i + chunkSize, usersToInsert.size());
            List<User> chunk = usersToInsert.subList(i, end);
            mapper.batchInsert(chunk);
        }

        // Batch updates
        for (int i = 0; i < usersToUpdate.size(); i += chunkSize) {
            int end = Math.min(i + chunkSize, usersToUpdate.size());
            List<User> chunk = usersToUpdate.subList(i, end);
            mapper.batchUpdate(chunk);
        }

        session.commit();
    }
}
```

## Dynamic SQL

MyBatis excels at dynamic SQL generation.

```xml
<select id="findUsers" resultType="User">
    SELECT * FROM users
    <where>
        <if test="status != null">
            AND status = #{status}
        </if>
        <if test="email != null">
            AND email LIKE CONCAT('%', #{email}, '%')
        </if>
        <if test="ids != null and ids.size() > 0">
            AND id IN
            <foreach collection="ids" item="id" open="(" separator="," close=")">
                #{id}
            </foreach>
        </if>
    </where>
    <choose>
        <when test="orderBy == 'username'">
            ORDER BY username
        </when>
        <when test="orderBy == 'email'">
            ORDER BY email
        </when>
        <otherwise>
            ORDER BY id
        </otherwise>
    </choose>
</select>
```

## Advanced Features

### Result Maps

```xml
<resultMap id="UserResultMap" type="User">
    <id property="id" column="user_id"/>
    <result property="username" column="user_name"/>
    <result property="email" column="user_email"/>
    <association property="profile" javaType="Profile">
        <id property="id" column="profile_id"/>
        <result property="bio" column="bio"/>
    </association>
    <collection property="orders" ofType="Order">
        <id property="id" column="order_id"/>
        <result property="orderNumber" column="order_number"/>
    </collection>
</resultMap>
```

### Type Handlers

```java
import org.apache.ibatis.type.BaseTypeHandler;
import org.apache.ibatis.type.JdbcType;
import org.apache.ibatis.type.MappedTypes;
import java.sql.CallableStatement;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

@MappedTypes(Status.class)
public class StatusTypeHandler extends BaseTypeHandler<Status> {
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, Status parameter, JdbcType jdbcType) throws SQLException {
        ps.setString(i, parameter.getCode());
    }

    @Override
    public Status getNullableResult(ResultSet rs, String columnName) throws SQLException {
        return Status.fromCode(rs.getString(columnName));
    }

    @Override
    public Status getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return Status.fromCode(rs.getString(columnIndex));
    }

    @Override
    public Status getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        return Status.fromCode(cs.getString(columnIndex));
    }
}
```

### Stored Procedures

```xml
<select id="getUserCount" statementType="CALLABLE">
    {call get_user_count(#{status, mode=IN, jdbcType=VARCHAR}, #{count, mode=OUT, jdbcType=INTEGER})}
</select>
```

## When to Use MyBatis

- You need fine-grained SQL control
- Working with complex legacy databases
- Performance is critical
- Team prefers SQL over ORM abstractions
- Need to optimize specific queries
- Working with stored procedures
- Want explicit control over result mapping

## Performance Considerations

- Excellent performance for bulk operations
- Full control over SQL optimization
- Can use database-specific features
- Efficient result mapping
- Minimal overhead compared to full ORMs
- Use batch executor for bulk operations
- Consider chunking for very large datasets

## Common Pitfalls

- More boilerplate than full ORMs
- Manual management of relationships
- No automatic dirty checking
- Requires more SQL knowledge
- XML configuration can become verbose
- Need to handle caching manually

## Related

- [[Object Relational Mapping Explained]] - Overview and comparison of all ORM frameworks
- [[jOOQ]] - Similar performance, adds type-safe SQL
- [[Hibernate]] - Full ORM alternative with automatic mapping
- [[Spring Data JPA]] - Higher-level abstraction built on Hibernate

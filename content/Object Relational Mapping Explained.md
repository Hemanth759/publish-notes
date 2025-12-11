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

## Common Limitations and Challenges

*To be documented as we progress through discussions*

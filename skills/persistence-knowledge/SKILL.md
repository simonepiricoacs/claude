---
name: persistence-knowledge
description: Comprehensive knowledge base for Water Framework persistence layer, including Repository pattern, JPA implementation, QueryBuilder fluent API, transaction management, and SystemService integration. Use when designing, implementing, or debugging data access, queries, pagination, or transaction logic in Water modules.
allowed-tools: Read, Glob, Grep
---

You are an expert Water Framework persistence architect with deep knowledge of the Repository abstraction layer, JPA implementation, QueryBuilder API, transaction management, and SystemService patterns. You use this knowledge to guide correct data access implementations, query building, and transactional design in Water modules.

---

# WATER FRAMEWORK PERSISTENCE KNOWLEDGE BASE

## Table of Contents

0. [Package & Import Reference (on-demand)](#0-package--import-reference)
1. [Persistence Architecture Overview](#1-persistence-architecture-overview)
2. [Module Dependency Map](#2-module-dependency-map)
3. [BaseRepository Interface](#3-baserepository-interface)
4. [JpaRepository Extension](#4-jparepository-extension)
5. [AbstractJpaEntity Base Class](#5-abstractjpaentity-base-class)
6. [QueryBuilder Fluent API](#6-querybuilder-fluent-api)
7. [Query Operations Hierarchy](#7-query-operations-hierarchy)
8. [String-based Query Parser](#8-string-based-query-parser)
9. [PredicateBuilder - JPA Translation](#9-predicatebuilder---jpa-translation)
10. [Pagination & Ordering](#10-pagination--ordering)
10a. [findAll Signature Reference — Critical Layer Differences](#10a-findall-signature-reference--critical-layer-differences)
10b. [Find Method Signatures — Complete Reference](#10b-find-method-signatures--complete-reference)
11. [Transaction Management](#11-transaction-management)
12. [BaseEntitySystemServiceImpl](#12-baseentitysystemserviceimpl)
13. [BaseJpaRepositoryImpl Deep Dive](#13-basejparepositoryimpl-deep-dive)
14. [Spring JPA Integration](#14-spring-jpa-integration)
15. [Entity Extensions (Expandable Entity)](#15-entity-extensions-expandable-entity)
16. [Database Constraints Validation](#16-database-constraints-validation)
17. [Entity Lifecycle Events](#17-entity-lifecycle-events)
18. [Persistence Unit Configuration](#18-persistence-unit-configuration)
19. [Best Practices & Anti-Patterns](#19-best-practices--anti-patterns)
20. [Decision Trees](#20-decision-trees)
21. [Quick Reference Tables](#21-quick-reference-tables)

---

## 0. Package & Import Reference

> **Package reference loaded** — the complete FQCN table, standard import blocks, and critical code-generation traps from `shared/package-reference.md` are already available in this skill's context.

### Persistence-specific packages — quick lookup

| Class | Package |
|---|---|
| `BaseRepository<T>` | `it.water.core.api.repository` |
| `Query` | `it.water.core.api.repository.query` |
| `QueryBuilder` | `it.water.core.api.repository.query` |
| `QueryOrder` | `it.water.core.api.repository.query` |
| `FieldNameOperand` | `it.water.core.api.repository.query.operands` |
| `PaginableResult<T>` | `it.water.core.api.model` |
| `AbstractJpaEntity` | `it.water.repository.jpa.model` |
| `JpaRepository<T>` | `it.water.repository.jpa.api` |
| `BaseJpaRepositoryImpl<T>` | `it.water.repository.jpa` |
| `DefaultQueryBuilder` | `it.water.repository.query` |
| `DefaultQueryOrder` | `it.water.repository.query.order` |
| `BaseEntitySystemServiceImpl<T>` | `it.water.repository.service` |
| `BaseEntityServiceImpl<T>` | `it.water.repository.service` |
| `EntityNotFound` | `it.water.repository.entity.model.exceptions` |
| `DuplicateEntityException` | `it.water.repository.entity.model.exceptions` |

### Source files — read for exact signatures
```
Core/Core-api/src/main/java/it/water/core/api/repository/BaseRepository.java
Core/Core-api/src/main/java/it/water/core/api/repository/query/QueryBuilder.java
Core/Core-api/src/main/java/it/water/core/api/repository/query/operands/FieldNameOperand.java
JpaRepository/JpaRepository-api/src/main/java/it/water/repository/jpa/model/AbstractJpaEntity.java
JpaRepository/JpaRepository-api/src/main/java/it/water/repository/jpa/api/JpaRepository.java
Repository/Repository-service/src/main/java/it/water/repository/service/BaseEntitySystemServiceImpl.java
```
(source root: Water Framework source repository root)

---

## 1. Persistence Architecture Overview

Water Framework implements a **technology-agnostic repository pattern** with a clear separation between query abstraction and JPA implementation.

### Layer Architecture

```
Api Layer (BaseEntityApi)
  |  annotated with security (@AllowPermissions, etc.)
  v
SystemService Layer (BaseEntitySystemServiceImpl)
  |  validation, events, asset management
  v
Repository Layer (BaseRepository / JpaRepository)
  |  CRUD operations, query execution, transactions
  v
JPA / EntityManager
  |  entity mapping, SQL generation
  v
Database
```

### Key Principle

- **Core-api** defines `BaseRepository`, `Query`, `QueryBuilder` interfaces (technology-agnostic)
- **Repository** module provides `DefaultQueryBuilder`, `QueryParser`, `BaseEntitySystemServiceImpl`
- **JpaRepository** module provides `BaseJpaRepositoryImpl`, `PredicateBuilder`, `AbstractJpaEntity`
- Application code NEVER directly touches `EntityManager` — always goes through Repository

---

## 2. Module Dependency Map

```
Core-api (interfaces: BaseRepository, Query, QueryBuilder, QueryOrder, PaginableResult)
  |
  +--- Repository-entity (exception classes: EntityNotFound, DuplicateEntityException, NoResultException)
  |
  +--- Repository-persistence (DefaultQueryBuilder, QueryParser)
  |         depends on: Core-api
  |
  +--- Repository-service (BaseEntitySystemServiceImpl)
  |         depends on: Core-api, Repository-entity
  |
  +--- JpaRepository-api (BaseJpaRepositoryImpl, AbstractJpaEntity, PredicateBuilder, JpaRepository)
  |         depends on: Core-api, Repository-entity, Repository-persistence
  |
  +--- JpaRepository-spring (SpringBaseJpaRepositoryImpl)
            depends on: JpaRepository-api, Spring Framework
```

---

## 3. BaseRepository Interface

**Package:** `it.water.core.api.repository`
**File:** `Core/Core-api/src/main/java/it/water/core/api/repository/BaseRepository.java`

```java
package it.water.core.api.repository;

import it.water.core.api.model.BaseEntity;
import it.water.core.api.model.PaginableResult;
import it.water.core.api.repository.query.Query;
import it.water.core.api.repository.query.QueryBuilder;
import it.water.core.api.repository.query.QueryOrder;
import it.water.core.api.service.Service;

public interface BaseRepository<T extends BaseEntity> extends Service {
    // Entity type
    Class<T> getEntityType();

    // CREATE
    T persist(T entity);
    T persist(T entity, Runnable executeInTransaction);

    // UPDATE
    T update(T entity);
    T update(T entity, Runnable executeInTransaction);

    // DELETE
    void remove(long id);
    void remove(long id, Runnable executeInTransaction);
    void remove(T entity);
    void removeAllByIds(Iterable<Long> ids);
    void removeAll(Iterable<T> entities);
    void removeAll();

    // READ — ⚠️ Query is 3rd param in findAll (differs from SystemApi)
    T find(long id);
    T find(Query filter);
    T find(String filterStr);
    PaginableResult<T> findAll(int delta, int page, Query filter, QueryOrder queryOrder);
    long countAll(Query filter);

    // Utility
    QueryBuilder getQueryBuilderInstance();
}
```

### Key Features

- **Transactional callbacks**: `persist(entity, runnable)` and `update(entity, runnable)` execute the `Runnable` within the SAME transaction as the main operation.
- **Three find modes**: by ID, by `Query` object, or by string filter expression.
- **QueryBuilder access**: Each repository provides a `QueryBuilder` for constructing type-safe queries.

---

## 4. JpaRepository Extension

**Package:** `it.water.repository.jpa.api`
**File:** `JpaRepository/JpaRepository-api/src/main/java/it/water/repository/jpa/api/JpaRepository.java`

```java
package it.water.repository.jpa.api;

import it.water.core.api.model.BaseEntity;
import it.water.core.api.repository.BaseRepository;
import jakarta.persistence.EntityManager;
import jakarta.transaction.Transactional;
import java.util.function.Consumer;
import java.util.function.Function;

public interface JpaRepository<T extends BaseEntity> extends BaseRepository<T> {
    EntityManager getEntityManager();

    // Execute code in transaction without result
    void txExpr(Transactional.TxType txType, Consumer<EntityManager> function);

    // Execute code in transaction with result
    <R> R tx(Transactional.TxType txType, Function<EntityManager, R> function);
}
```

### Transaction Types

| TxType | Behavior |
|--------|----------|
| `REQUIRED` | Uses existing transaction or creates new one (default) |
| `REQUIRES_NEW` | Always creates a new transaction |
| `SUPPORTS` | Uses existing transaction if available, no TX otherwise |
| `NOT_SUPPORTED` | Suspends current transaction |
| `MANDATORY` | Requires existing transaction, throws if none |
| `NEVER` | Throws if transaction exists |

### Custom Transaction Usage

```java
// Execute arbitrary code in a transaction
repository.tx(Transactional.TxType.REQUIRED, em -> {
    // em is a managed EntityManager
    MyEntity entity = em.find(MyEntity.class, id);
    entity.setStatus("processed");
    em.merge(entity);
    return entity;
});

// Execute without returning a result
repository.txExpr(Transactional.TxType.REQUIRED, em -> {
    em.createQuery("DELETE FROM MyEntity e WHERE e.status = 'expired'")
      .executeUpdate();
});
```

---

## 5. AbstractJpaEntity Base Class

**Package:** `it.water.repository.jpa.model`
**File:** `JpaRepository/JpaRepository-api/src/main/java/it/water/repository/jpa/model/AbstractJpaEntity.java`

Extends `AbstractEntity` (`it.water.repository.entity.model.AbstractEntity`) which in turn extends `AbstractResource` (`it.water.core.model.AbstractResource`).

```java
package it.water.repository.jpa.model;

import it.water.core.api.model.BaseEntity;
import it.water.repository.entity.model.AbstractEntity;
import jakarta.persistence.*;
import java.util.Date;

@MappedSuperclass
@Embeddable
public abstract class AbstractJpaEntity extends AbstractEntity implements BaseEntity {

    @Override @Id @GeneratedValue
    public long getId();

    @Override @Version
    @Column(name = "entity_version", columnDefinition = "INTEGER default 1")
    public Integer getEntityVersion();      // Optimistic locking

    @Override @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "entity_create_date")
    public Date getEntityCreateDate();      // Auto-set on persist

    @Override @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "entity_modify_date")
    public Date getEntityModifyDate();      // Auto-set on persist & update

    @PrePersist
    protected void prePersist() { ... }     // Sets both dates to now

    @PreUpdate
    protected void preUpdate() { ... }      // Updates entityModifyDate to now

    // Override these (not prePersist/preUpdate) for custom JPA lifecycle logic
    protected void doPrePersist() {}
    protected void doPreUpdate()  {}
}
```

### AbstractEntity fields (inherited by AbstractJpaEntity)

**Package:** `it.water.repository.entity.model.AbstractEntity`

```java
protected long    id;
protected Integer entityVersion;
protected Date    entityCreateDate;
protected Date    entityModifyDate;
protected long[]  categoryIds;   // asset management
protected long[]  tagIds;        // asset management
```

### Built-in Features

- **Optimistic locking** via `@Version` — prevents concurrent modification
- **Automatic timestamps** — `entityCreateDate` and `entityModifyDate` managed by JPA lifecycle
- **Auto-generated IDs** — `@Id @GeneratedValue`
- **Extensible lifecycle hooks** — Override `prePersist()` / `preUpdate()` for custom logic

### Defining a New Entity

```java
@Entity
@Table(name = "my_entity")
@Access(AccessType.FIELD)
public class MyEntity extends AbstractJpaEntity {
    @Column @NotBlank
    private String name;

    @Column
    private int quantity;

    // getters/setters via Lombok or manual
}
```

---

## 6. QueryBuilder Fluent API

**Interface package:** `it.water.core.api.repository.query.QueryBuilder`
**Implementation package:** `it.water.repository.query.DefaultQueryBuilder`
**File:** `Core/Core-api/src/main/java/it/water/core/api/repository/query/QueryBuilder.java`

```java
package it.water.core.api.repository.query;

import it.water.core.api.repository.query.operands.FieldNameOperand;

public interface QueryBuilder {
    Query createQueryFilter(String filter);     // Parse string filter expression
    FieldNameOperand field(String name);        // Start fluent query from a field name
}
```

### Getting a QueryBuilder

```java
// From Repository (preferred inside SystemServiceImpl)
QueryBuilder qb = repository.getQueryBuilderInstance();

// From SystemApi (preferred inside ServiceImpl)
QueryBuilder qb = systemApi.getQueryBuilderInstance();

// Standalone (unit tests)
QueryBuilder qb = new DefaultQueryBuilder(); // it.water.repository.query.DefaultQueryBuilder
```

### FieldNameOperand — Exact Method Signatures

**Package:** `it.water.core.api.repository.query.operands.FieldNameOperand`

`FieldNameOperand` is returned by `qb.field("name")` and provides comparison methods that all return `Query`.

```java
// Exact method signatures from FieldNameOperand source:

Query equalTo(Object value);                    // any type
Query notEqualTo(Object value);                 // any type

Query greaterThan(Number value);                // numeric only
Query greaterThan(Date value);                  // java.util.Date overload

Query greaterOrEqualThan(Number value);
Query greaterOrEqualThan(Date value);

Query lowerThan(Number value);
Query lowerThan(Date value);

Query lowerOrEqualThan(Number value);
Query lowerOrEqualThan(Date value);

Query like(String value);                       // String only
```

> **⚠️ No `in()` or `not()` on `FieldNameOperand`** — those methods are on the `Query` interface (returned by the above methods). Call `.in(list)` or `.not()` on the result of a comparison, not on the field operand itself.

### Query Interface — Composition Methods

**Package:** `it.water.core.api.repository.query.Query`

```java
public interface Query {
    Query and(Query rightQuery);          // AND composition
    Query or(Query rightQuery);           // OR composition
    Query in(List<?> values);             // IN clause
    Query not();                          // NOT negation
    String getDefinition();               // Human-readable string representation
    void defineOperands(Query... operands);
}
```

### Fluent API Usage — Complete Reference

| Expression | Returns | Notes |
|-----------|---------|-------|
| `qb.field("name").equalTo("John")` | `Query` | Object equality |
| `qb.field("name").notEqualTo("John")` | `Query` | Inequality |
| `qb.field("age").greaterThan(25)` | `Query` | `Number` or `Date` only |
| `qb.field("age").greaterOrEqualThan(25)` | `Query` | `Number` or `Date` only |
| `qb.field("age").lowerThan(50)` | `Query` | `Number` or `Date` only |
| `qb.field("age").lowerOrEqualThan(50)` | `Query` | `Number` or `Date` only |
| `qb.field("name").like("%mario%")` | `Query` | `String` only |
| `q1.and(q2)` | `Query` | Combine with AND |
| `q1.or(q2)` | `Query` | Combine with OR |
| `q1.not()` | `Query` | Negate |
| `q1.in(List.of(1L, 2L, 3L))` | `Query` | IN list — call on any Query result |

### Composing Queries

```java
QueryBuilder qb = repository.getQueryBuilderInstance();

// Simple equality
Query q = qb.field("name").equalTo("John");

// AND composition
Query q = qb.field("name").equalTo("John")
    .and(qb.field("age").greaterThan(25));

// OR composition
Query q = qb.field("status").equalTo("active")
    .or(qb.field("status").equalTo("pending"));

// Complex nested query
Query nameFilter = qb.field("name").like("%mario%");
Query ageFilter = qb.field("age").greaterThan(20)
    .and(qb.field("age").lowerThan(50));
Query combined = nameFilter.or(ageFilter);

// IN clause
Query q = qb.field("id").equalTo(1).in(List.of(1L, 2L, 3L));

// NOT
Query q = qb.field("deleted").equalTo(true).not();

// Nested field paths (JPA relationships)
Query q = qb.field("user.address.city").equalTo("Rome");
```

### Query Definition Output

Every `Query` has a `getDefinition()` method that returns a human-readable string:
```java
qb.field("name").equalTo("John").getDefinition()
// Returns: "name = John"

qb.field("age").greaterThan(25).and(qb.field("status").like("%active%")).getDefinition()
// Returns: "age > 25 AND status LIKE %active%"
```

---

## 7. Query Operations Hierarchy

**Base path:** `Core/Core-api/src/main/java/it/water/core/api/repository/query/operations/`

```
AbstractOperation (implements Query, QueryFilterOperation)
|
+-- UnaryOperation (1 operand)
|   +-- NotOperation          -> NOT
|
+-- BinaryOperation (2 operands)
    |
    +-- BinaryLogicOperation
    |   +-- AndOperation      -> AND
    |   +-- OrOperation       -> OR
    |
    +-- BinaryValueOperation (field + value)
    |   +-- EqualTo           -> =
    |   +-- NotEqualTo        -> <>
    |   +-- GreaterThan       -> >
    |   +-- GreaterOrEqualThan -> >=
    |   +-- LowerThan         -> <
    |   +-- LowerOrEqualThan  -> <=
    |   +-- Like              -> LIKE
    |
    +-- BinaryValueListOperation
        +-- In                -> IN
```

### Operand Types

| Class | Purpose |
|-------|---------|
| `FieldNameOperand` | Represents a field name (e.g., `"name"`, `"user.address"`) |
| `FieldValueOperand` | Represents a single value (e.g., `"John"`, `42`, `true`) |
| `FieldValueListOperand` | Represents a list of values for IN clause |
| `ParenthesisNode` | Groups nested expressions |

---

## 8. String-based Query Parser

**File:** `Repository/Repository-persistence/src/main/java/it/water/repository/query/parser/QueryParser.java`

Parses string filter expressions into `Query` objects using `StreamTokenizer`.

### Supported Syntax

```
<field> <operator> <value> [AND|OR <field> <operator> <value>]*

Operators: =, <>, >, >=, <, <=, LIKE, IN, NOT
Logic: AND, OR
Grouping: ( )
Values: 'quoted string', "double quoted", numeric, true/false
```

### Examples

```java
QueryBuilder qb = new DefaultQueryBuilder();

// Simple
Query q = qb.createQueryFilter("name = 'Mario'");

// Multiple conditions
Query q = qb.createQueryFilter("age > 20 AND name LIKE 'mario'");

// IN clause
Query q = qb.createQueryFilter("id IN (1,2,3)");

// Parentheses
Query q = qb.createQueryFilter("(age > 20 AND name LIKE 'mario') OR status = 'active'");

// Nested fields
Query q = qb.createQueryFilter("user.address.city = 'Rome'");

// Mixed
Query q = qb.createQueryFilter("ownerUserId = 52 OR id IN (452,454)");
```

### When to Use String vs Fluent API

| Use String Parser | Use Fluent API |
|-------------------|----------------|
| Filters from REST query parameters | Programmatic query building in code |
| User-defined dynamic filters | Type-safe queries in services |
| Simple expressions | Complex nested conditions |
| Test scenarios | Production service code |

---

## 9. PredicateBuilder - JPA Translation

**File:** `JpaRepository/JpaRepository-api/src/main/java/it/water/repository/jpa/query/PredicateBuilder.java`

Converts Water `Query` objects into JPA `Predicate` objects for `CriteriaBuilder`.

### Translation Map

| Water Operation | JPA CriteriaBuilder Method |
|----------------|---------------------------|
| `AndOperation` | `cb.and(left, right)` |
| `OrOperation` | `cb.or(left, right)` |
| `NotOperation` | `cb.not(operand)` |
| `EqualTo` | `cb.equal(path, value)` / `cb.isNull(path)` for null |
| `NotEqualTo` | `cb.notEqual(path, value)` |
| `GreaterThan` | `cb.greaterThan(path, value)` |
| `GreaterOrEqualThan` | `cb.greaterThanOrEqualTo(path, value)` |
| `LowerThan` | `cb.lessThan(path, value)` |
| `LowerOrEqualThan` | `cb.lessThanOrEqualTo(path, value)` |
| `Like` | `cb.like(path, value)` |
| `In` | `path.in(values)` |

### Type Conversion

PredicateBuilder automatically converts value types:
- **String** → direct use
- **Number** (Integer, Long, Float, Double) → parsed from string
- **Boolean** → `Boolean.parseBoolean()`
- **Date types** → `java.util.Date`, `java.sql.Date`, `java.sql.Timestamp`, `Instant`, `LocalDateTime`, `LocalDate`
- **Enum** → tries `Enum.valueOf()` first, falls back to ordinal

### Nested Field Paths

Dot-separated field names are resolved through JPA `Path.get()`:
```
"user.address.city" → root.get("user").get("address").get("city")
```

---

## 10. Pagination & Ordering

### PaginableResult

**Package:** `it.water.core.api.model.PaginableResult`
**File:** `Core/Core-api/src/main/java/it/water/core/api/model/PaginableResult.java`

```java
package it.water.core.api.model;

import java.util.Collection;

public interface PaginableResult<T extends Resource> {
    int getNumPages();               // Total number of pages
    int getCurrentPage();            // Current page (1-based)
    int getNextPage();               // Next page number (or 1 if on last page)
    int getDelta();                  // Items per page
    Collection<T> getResults();      // ⚠️ Returns Collection<T>, NOT List<T>
}
```

### QueryOrder

**Interface package:** `it.water.core.api.repository.query.QueryOrder`
**Implementation package:** `it.water.repository.query.order.DefaultQueryOrder`
**File:** `Core/Core-api/src/main/java/it/water/core/api/repository/query/QueryOrder.java`

```java
package it.water.core.api.repository.query;

public interface QueryOrder {
    QueryOrder addOrderField(String name, boolean asc);   // returns this (fluent)
    List<QueryOrderParameter> getParametersList();
}
```

Instantiate with `new DefaultQueryOrder()` from package `it.water.repository.query.order`.

### Usage Examples

```java
QueryBuilder qb = repository.getQueryBuilderInstance();
Query filter = qb.field("status").equalTo("active");

// Page 1, 10 items per page, no ordering
PaginableResult<MyEntity> page1 = repository.findAll(10, 1, filter, null);

// With ordering (ascending by name)
QueryOrder order = new DefaultQueryOrder();
order.addOrderField("name", true);
PaginableResult<MyEntity> sorted = repository.findAll(10, 1, filter, order);

// Multiple order fields (descending by date, ascending by name)
QueryOrder order = new DefaultQueryOrder();
order.addOrderField("entityCreateDate", false);  // DESC
order.addOrderField("name", true);               // ASC
PaginableResult<MyEntity> results = repository.findAll(20, 2, filter, order);

// Count matching records
long total = repository.countAll(filter);

// Find single entity by filter
MyEntity entity = repository.find(qb.field("email").equalTo("john@example.com"));
```

### Pagination Notes

- `delta` = -1 means no pagination (return all results)
- `page` is 1-based
- `filter` can be null (no filtering)
- `queryOrder` can be null (no ordering)

---

## 10a. findAll Signature Reference — Critical Layer Differences

> **⚠️ THIS IS THE MOST COMMON SOURCE OF CODE GENERATION BUGS.**
> The `findAll` method has **different parameter order** depending on the layer. Never assume they are the same.

### Signatures by layer — side-by-side comparison

| Layer | Class/Interface | Exact Signature |
|-------|----------------|-----------------|
| **Repository** | `BaseRepository<T>` | `findAll(int delta, int page, Query filter, QueryOrder queryOrder)` |
| **SystemApi** | `BaseEntitySystemApi<T>` | `findAll(Query filter, int delta, int page, QueryOrder queryOrder)` |
| **Api (public)** | `BaseEntityServiceImpl<T>` | `findAll(Query filter, int delta, int page, QueryOrder queryOrder)` |

**Rule: `Query` is the 3rd argument at Repository level, and the 1st argument at SystemApi/Api level.**

```java
// ✅ CORRECT — calling Repository directly (e.g. inside SystemServiceImpl)
repository.findAll(-1, -1, query, null);

// ✅ CORRECT — calling SystemApi (e.g. inside ServiceImpl or test)
systemApi.findAll(query, -1, -1, null);

// ✅ CORRECT — calling public Api
myEntityApi.findAll(query, -1, -1, null);

// ❌ WRONG — mixing up the layers
repository.findAll(query, -1, -1, null);   // compile error: first param must be int
systemApi.findAll(-1, -1, query, null);    // compile error: first param must be Query
```

### BaseEntitySystemServiceImpl.findAll — forwards with correct transposition

`BaseEntitySystemServiceImpl` receives `findAll(Query filter, int delta, int page, QueryOrder queryOrder)` and internally calls `repository.findAll(delta, page, filter, queryOrder)` — the transposition happens here. Never bypass it.

```java
// Inside BaseEntitySystemServiceImpl — this is the bridge
@Override
public PaginableResult<T> findAll(Query filter, int delta, int page, QueryOrder queryOrder) {
    return this.getRepository().findAll(delta, page, filter, queryOrder);
}
```

---

## 10b. Find Method Signatures — Complete Reference

This is the authoritative reference for every read method on `BaseRepository`. Always verify parameter order and return types here before writing repository code.

### All read methods on BaseRepository

```java
// package: it.water.core.api.repository.BaseRepository

// Find by primary key — returns T, throws EntityNotFound if missing
T find(long id);

// Find single entity matching a Query — returns T, throws NoResultException if none
T find(Query filter);  // filter: it.water.core.api.repository.query.Query

// Find single entity matching a string filter expression
T find(String filterStr);

// Find all with pagination, filtering, ordering
// delta  : page size (-1 = no limit, returns all)
// page   : 1-based page number (ignored when delta = -1)
// filter : it.water.core.api.repository.query.Query — null means no filter
// queryOrder : it.water.core.api.repository.query.QueryOrder — null means no ordering
// returns: PaginableResult<T> — getResults() returns Collection<T>, NOT List<T>
PaginableResult<T> findAll(int delta, int page, Query filter, QueryOrder queryOrder);

// Count entities matching a filter — filter can be null (counts everything)
long countAll(Query filter);
```

### All read methods on BaseEntitySystemApi / BaseEntityServiceImpl

```java
// package: it.water.core.api.service.BaseEntitySystemApi

T find(long id);
T find(Query filter);

// NOTE: Query is the FIRST parameter here — opposite of BaseRepository.findAll
PaginableResult<T> findAll(Query filter, int delta, int page, QueryOrder queryOrder);

long countAll(Query filter);
```

### ⚠️ Return type trap — `getResults()`

`PaginableResult.getResults()` returns `Collection<T>`, **not** `List<T>`. Assigning directly to `List` causes a compile error.

```java
// WRONG — compile error:
List<MyEntity> list = repository.findAll(-1, -1, q, null).getResults();

// CORRECT — wrap with ArrayList:
List<MyEntity> list = new ArrayList<>(repository.findAll(-1, -1, q, null).getResults());
```

### Common find patterns

```java
// All records via Repository (inside SystemServiceImpl)
List<MyEntity> all = new ArrayList<>(repository.findAll(-1, -1, null, null).getResults());

// All records via SystemApi (inside ServiceImpl or test)
List<MyEntity> all = new ArrayList<>(systemApi.findAll(null, -1, -1, null).getResults());

// Filtered, no pagination via Repository
Query q = getQueryBuilderInstance().field("status").equalTo("ACTIVE");
List<MyEntity> active = new ArrayList<>(repository.findAll(-1, -1, q, null).getResults());

// Filtered, no pagination via SystemApi
List<MyEntity> active = new ArrayList<>(systemApi.findAll(q, -1, -1, null).getResults());

// Paginated — page 2, 10 items per page, via Repository
QueryOrder order = new DefaultQueryOrder();   // it.water.repository.query.order.DefaultQueryOrder
order.addOrderField("entityCreateDate", false); // DESC
PaginableResult<MyEntity> page2 = repository.findAll(10, 2, q, order);
int totalPages = page2.getNumPages();
Collection<MyEntity> items = page2.getResults();

// Single entity by filter
MyEntity single = repository.find(getQueryBuilderInstance().field("email").equalTo("a@b.com"));

// Count
long total = repository.countAll(q);
```

---

## 11. Transaction Management

### Transaction Flow in BaseJpaRepositoryImpl

```
persist(entity) / update(entity)
  |
  v
tx(TxType.REQUIRED, em -> doPersist(entity, null, em))
  |
  v
startTransactionIfNeeded(em)
  |  - If isTransactionalSupported() == false (non-Spring):
  |      em.getTransaction().begin()
  |  - If Spring: no-op (Spring manages TX)
  v
dbConstraintsValidatorManager.runCheck(entity)    // Unique constraint check
  |
  v
em.persist(entity) / em.merge(entity)
  |
  v
doPersistOnExpandableEntity(entity)               // Handle extensions
  |
  v
commitTransactionIfNeeded(em)
  |  - If isTransactionalSupported() == false:
  |      em.getTransaction().commit()
  |  - If Spring: no-op (Spring commits at TX boundary)
  v
Return entity
  |
  (On RuntimeException: rollback if non-Spring)
```

### Transactional Callback Pattern

Execute additional operations within the SAME transaction as the entity save:

```java
// Save entity and create related records in the same TX
repository.persist(entity, () -> {
    // This runs INSIDE the same transaction
    relatedRepository.persist(relatedEntity);
    auditRepository.persist(auditRecord);
});
```

### Spring vs Non-Spring Transaction Handling

| Aspect | Spring | Non-Spring (OSGi/Standalone) |
|--------|--------|------------------------------|
| TX start | `TransactionTemplate` | `em.getTransaction().begin()` |
| TX commit | Automatic at TX boundary | `em.getTransaction().commit()` |
| TX rollback | Automatic on exception | Manual `em.getTransaction().rollback()` |
| `isTransactionalSupported()` | `true` | `false` |
| TX propagation | Spring `TransactionDefinition` | Manual management |

### Custom Transaction in Repository

```java
// Get result from transactional code
MyEntity result = repository.tx(Transactional.TxType.REQUIRED, em -> {
    MyEntity entity = em.find(MyEntity.class, id);
    entity.setProcessed(true);
    return em.merge(entity);
});

// Execute without result
repository.txExpr(Transactional.TxType.REQUIRES_NEW, em -> {
    em.createQuery("UPDATE MyEntity e SET e.status = 'archived' WHERE e.age > 365")
      .executeUpdate();
});
```

---

## 12. BaseEntitySystemServiceImpl

**Package:** `it.water.repository.service`
**File:** `Repository/Repository-service/src/main/java/it/water/repository/service/BaseEntitySystemServiceImpl.java`

The bridge between Api/SystemApi and Repository. All CRUD operations go through this class.

### Architecture

```java
package it.water.repository.service;

import it.water.core.api.model.BaseEntity;
import it.water.core.api.model.PaginableResult;
import it.water.core.api.registry.ComponentRegistry;
import it.water.core.api.repository.BaseRepository;
import it.water.core.api.repository.query.Query;
import it.water.core.api.repository.query.QueryBuilder;
import it.water.core.api.repository.query.QueryOrder;
import it.water.core.api.service.BaseEntitySystemApi;
import it.water.core.api.validation.WaterValidator;
import it.water.core.interceptors.annotations.Inject;
import it.water.core.service.BaseAbstractSystemService;
import lombok.Setter;

public abstract class BaseEntitySystemServiceImpl<T extends BaseEntity>
    extends BaseAbstractSystemService implements BaseEntitySystemApi<T> {

    @Inject @Setter
    protected ComponentRegistry componentRegistry;

    @Inject @Setter
    protected WaterValidator waterValidator;

    // Constructor MUST pass entity class
    protected BaseEntitySystemServiceImpl(Class<T> type) { ... }

    // ⚠️ findAll signature: Query is FIRST param (differs from BaseRepository.findAll)
    @Override
    public PaginableResult<T> findAll(Query filter, int delta, int page, QueryOrder queryOrder) {
        // Internally calls repository.findAll(delta, page, filter, queryOrder) — args transposed
        return this.getRepository().findAll(delta, page, filter, queryOrder);
    }

    // Subclasses MUST provide the repository
    protected abstract BaseRepository<T> getRepository();
}
```

### CRUD Operation Flow

**Save:**
```
save(entity)
  -> validate(entity)
  -> validateEntityExtension(entity)
  -> produceEvent(entity, PreSaveEvent.class)
  -> repository.persist(entity)
  -> manageAssets(entity, ADD)
  -> produceEvent(entity, PostSaveEvent.class)
  -> return entity
```

**Update:**
```
update(entity)
  -> validate(entity)
  -> validateEntityExtension(entity)
  -> entityBeforeUpdate = find(entity.getId())
  -> produceEvent(entity, PreUpdateEvent.class)
  -> produceDetailedEvent(before, entity, PreUpdateDetailedEvent.class)
  -> repository.update(entity)
  -> manageAssets(entity, UPDATE)
  -> produceDetailedEvent(before, updated, PostUpdateDetailedEvent.class)
  -> return updatedEntity
```

**Remove:**
```
remove(id)
  -> entity = find(id)
  -> if entity == null: throw EntityNotFound
  -> produceEvent(entity, PreRemoveEvent.class)
  -> repository.remove(id)
  -> manageAssets(entity, DELETE)
  -> produceEvent(entity, PostRemoveEvent.class)
```

### Implementing a SystemService

```java
import it.water.core.interceptors.annotations.FrameworkComponent;
import it.water.core.interceptors.annotations.Inject;
import it.water.core.api.repository.BaseRepository;
import it.water.core.api.repository.query.QueryBuilder;
import it.water.repository.service.BaseEntitySystemServiceImpl;
import lombok.Setter;

@FrameworkComponent
public class MyEntitySystemServiceImpl
    extends BaseEntitySystemServiceImpl<MyEntity>
    implements MyEntitySystemApi {

    @Inject @Setter
    private MyEntityRepository repository;

    public MyEntitySystemServiceImpl() {
        super(MyEntity.class);   // MUST pass entity class to super
    }

    @Override
    protected BaseRepository<MyEntity> getRepository() {
        return repository;
    }

    // Custom business methods — use getQueryBuilderInstance() from super
    public MyEntity findByEmail(String email) {
        QueryBuilder qb = getQueryBuilderInstance();
        return getRepository().find(qb.field("email").equalTo(email));
    }

    // Paginated custom query — note: inside SystemService, call repository.findAll(delta,page,filter,order)
    public PaginableResult<MyEntity> findActiveEntities(int delta, int page) {
        Query q = getQueryBuilderInstance().field("status").equalTo("ACTIVE");
        return getRepository().findAll(delta, page, q, null);
    }
}
```

### BaseEntityServiceImpl — Api Layer (permission-checked)

**Package:** `it.water.repository.service.BaseEntityServiceImpl`

```java
import it.water.repository.service.BaseEntityServiceImpl;
import it.water.core.api.service.BaseEntitySystemApi;
import it.water.core.api.registry.ComponentRegistry;
import it.water.core.interceptors.annotations.FrameworkComponent;
import it.water.core.interceptors.annotations.Inject;
import lombok.Setter;

@FrameworkComponent
public class MyEntityServiceImpl
    extends BaseEntityServiceImpl<MyEntity>
    implements MyEntityApi {

    @Inject @Setter
    private MyEntitySystemApi systemService;

    @Inject @Setter
    private ComponentRegistry componentRegistry;

    public MyEntityServiceImpl() {
        super(MyEntity.class);
    }

    @Override
    protected BaseEntitySystemApi<MyEntity> getSystemService() {
        return systemService;
    }

    @Override
    protected ComponentRegistry getComponentRegistry() {
        return componentRegistry;
    }
}
```

> `BaseEntityServiceImpl.findAll` signature: `findAll(Query filter, int delta, int page, QueryOrder queryOrder)` — Query is first, same as `BaseEntitySystemApi`.

---

## 13. BaseJpaRepositoryImpl Deep Dive

**File:** `JpaRepository/JpaRepository-api/src/main/java/it/water/repository/jpa/BaseJpaRepositoryImpl.java`

Core JPA repository (~735 lines) handling all persistence operations.

### Key Configuration

```java
public abstract class BaseJpaRepositoryImpl<T extends BaseEntity>
    implements JpaRepository<T> {

    // Persistence unit name (default: "water-default-persistence-unit")
    protected String persistenceUnitName;

    // Global EntityManager (OSGi)
    private static EntityManager globalEntityManager;

    // Instance EntityManager
    private EntityManager entityManager;
}
```

### EntityManager Resolution

The repository resolves `EntityManager` in this priority:
1. Instance-level `entityManager` (if set, e.g., by Spring)
2. Global static `entityManager` (if set, e.g., by OSGi)
3. Create new from `EntityManagerFactory` via persistence unit name

### Find with Filter

```java
@Override
public T find(Query filter) {
    return tx(Transactional.TxType.REQUIRED, em -> {
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<T> cq = cb.createQuery(type);
        Root<T> root = cq.from(type);

        PredicateBuilder pb = new PredicateBuilder(root, cb, cq);
        Predicate predicate = pb.buildPredicate(filter);
        cq.select(root).where(predicate);

        TypedQuery<T> query = em.createQuery(cq);
        return query.getSingleResult();
    });
}
```

### FindAll with Pagination

```java
@Override
public PaginableResult<T> findAll(int delta, int page, Query filter, QueryOrder order) {
    return tx(Transactional.TxType.REQUIRED, em -> {
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<T> cq = cb.createQuery(type);
        Root<T> root = cq.from(type);

        // Apply filter
        if (filter != null) {
            Predicate predicate = new PredicateBuilder(root, cb, cq).buildPredicate(filter);
            cq.select(root).where(predicate);
        }

        // Apply ordering
        if (order != null) {
            List<Order> orders = order.getParametersList().stream()
                .map(p -> p.isAsc() ? cb.asc(root.get(p.getName())) : cb.desc(root.get(p.getName())))
                .collect(Collectors.toList());
            cq.orderBy(orders);
        }

        TypedQuery<T> query = em.createQuery(cq);

        // Pagination
        if (delta > 0) {
            query.setFirstResult((page - 1) * delta);
            query.setMaxResults(delta);
        }

        // Build result
        List<T> results = query.getResultList();
        long count = countAll(filter);
        return new WaterPaginableResult<>(...);
    });
}
```

---

## 14. Spring JPA Integration

**File:** `JpaRepository/JpaRepository-spring/src/main/java/it/water/repository/jpa/spring/SpringBaseJpaRepositoryImpl.java`

### Spring-specific Features

```java
public abstract class SpringBaseJpaRepositoryImpl<T extends BaseEntity>
    extends BaseJpaRepositoryImpl<T> {

    @Autowired private PlatformTransactionManager transactionManager;
    @Autowired private EntityManagerFactory entityManagerFactory;
    private TransactionTemplate transactionTemplate;

    @PostConstruct
    void init() {
        transactionTemplate = new TransactionTemplate(transactionManager);
    }

    @Override
    public <R> R tx(Transactional.TxType txType, Function<EntityManager, R> function) {
        transactionTemplate.setPropagationBehavior(mapTxType(txType));
        return transactionTemplate.execute(status -> {
            EntityManager em = EntityManagerFactoryUtils
                .getTransactionalEntityManager(entityManagerFactory);
            return function.apply(em);
        });
    }
}
```

### TxType to Spring Propagation Mapping

| JPA TxType | Spring Propagation |
|------------|-------------------|
| `REQUIRED` | `PROPAGATION_REQUIRED` |
| `REQUIRES_NEW` | `PROPAGATION_REQUIRES_NEW` |
| `SUPPORTS` | `PROPAGATION_SUPPORTS` |
| `NEVER` | `PROPAGATION_NEVER` |
| `MANDATORY` | `PROPAGATION_MANDATORY` |
| `NOT_SUPPORTED` | `PROPAGATION_NOT_SUPPORTED` |

---

## 15. Entity Extensions (Expandable Entity)

**File:** `JpaRepository/JpaRepository-api/src/main/java/it/water/repository/jpa/model/AbstractJpaExpandableEntity.java`

Entities can be extended at runtime using the Expandable Entity pattern:

```java
public interface ExpandableEntity {
    EntityExtension getExtension();
    void setExtension(EntityExtension extension);
    boolean isExpandableEntity();
}
```

- Extension data is stored in a separate table (one-to-one relationship)
- Extension is validated alongside the main entity
- Extension is persisted/updated within the same transaction

---

## 16. Database Constraints Validation

**Pre-persist validation** checks unique constraints BEFORE sending to the database:

```java
dbConstraintsValidatorManager.runCheck(entity, type, this);
```

This prevents duplicate entity exceptions by checking constraints proactively, providing clearer error messages via `DuplicateEntityException`.

---

## 17. Entity Lifecycle Events

`BaseEntitySystemServiceImpl` generates events at each CRUD stage:

| Event Class | When | Contains |
|------------|------|----------|
| `PreSaveEvent` | Before persist | New entity |
| `PostSaveEvent` | After persist | Saved entity (with ID) |
| `PreUpdateEvent` | Before update | Updated entity |
| `PreUpdateDetailedEvent` | Before update | Before + After entity |
| `PostUpdateDetailedEvent` | After update | Before + After entity |
| `PreRemoveEvent` | Before remove | Entity to remove |
| `PostRemoveEvent` | After remove | Removed entity |

Events are produced via `ApplicationEventProducer` (if registered). If no producer is registered, events are silently skipped.

---

## 18. Persistence Unit Configuration

### One Persistence Unit per Module — Mandatory Rule

**Each Water module MUST define exactly ONE persistence unit**, and its name MUST reflect the module name.

**Naming convention:** `<module-kebab-case>-persistence-unit`

| Module | Persistence Unit Name |
|--------|----------------------|
| `ApiGateway` | `api-gateway-persistence-unit` |
| `Permission` | `permission-persistence-unit` |
| `SharedEntity` | `sharedentity-persistence-unit` |
| `User` | `user-persistence-unit` |
| `MyCustomModule` | `my-custom-module-persistence-unit` |

**Why this matters:**
- When multiple modules run in the same JVM (test or production), each must have its own isolated persistence unit
- Using the generic `water-default-persistence-unit` across multiple modules causes schema conflicts, entity class clashes, and unpredictable H2 in-memory DB collisions in tests
- The module's repository must reference its own persistence unit name explicitly in the constructor

### persistence.xml Location

Place in `src/test/resources/META-INF/persistence.xml` (for tests) and `src/main/resources/META-INF/persistence.xml` (for production if not using Spring auto-configuration).

### Example persistence.xml (test, HSQLDB in-memory)

```xml
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence
                                 http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd"
             version="2.2">

    <persistence-unit name="my-module-persistence-unit" transaction-type="RESOURCE_LOCAL">
        <class>it.water.mymodule.model.MyEntity</class>
        <properties>
            <property name="javax.persistence.jdbc.driver" value="org.hsqldb.jdbcDriver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:hsqldb:mem:mymoduledb;sql.syntax_mys=true"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.HSQLDialect"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="create-drop"/>
            <property name="hibernate.archive.autodetection" value="class"/>
        </properties>
    </persistence-unit>
</persistence>
```

> **Note:** Use a unique in-memory DB name per module (`jdbc:hsqldb:mem:mymoduledb`) to prevent H2/HSQLDB schema sharing between modules running in the same JVM.

### Wiring the Persistence Unit in the Repository

The repository implementation MUST pass the entity-specific persistence unit name to the superclass:

```java
@FrameworkComponent
public class MyEntityRepositoryImpl extends BaseJpaRepositoryImpl<MyEntity>
        implements MyEntityRepository {

    public MyEntityRepositoryImpl() {
        // MUST use the entity-specific persistence unit name
        super("myentity-persistence-unit", MyEntity.class);
    }
}
```

> **⚠️ Generator gotcha — `yo water:add-entity`**: When adding a new entity to an existing project, the generator copies the persistence unit name from the first/main entity of the project (e.g., `"book-persistence-unit"`). The generated `*RepositoryImpl.java` for the new entity will have the WRONG persistence unit constant. **Always manually update it** to match the new entity:
>
> ```java
> // Generated (WRONG — copied from first entity):
> private static final String LOAN_PERSISTENCE_UNIT = "book-persistence-unit";
>
> // Correct — must be updated manually:
> private static final String LOAN_PERSISTENCE_UNIT = "loan-persistence-unit";
> ```
>
> Failing to update this causes the new entity's repository to attach to the wrong persistence context at runtime.

### Default Persistence Unit (Framework Internal Only)

Name: `water-default-persistence-unit` — used only by `JpaRepository-test-utils` and framework-level tests. **Never use this name in application modules.**

---

## 19. Best Practices & Anti-Patterns

### Best Practices

1. **Always use QueryBuilder fluent API for programmatic queries.** It's type-safe and technology-agnostic.

2. **Use string parser for REST filters only.** Let the `QueryParser` handle user input from REST query parameters.

3. **Never access EntityManager directly in services.** Always go through `BaseRepository` or `JpaRepository` methods.

4. **Use transactional callbacks for multi-entity saves.**
   ```java
   repository.persist(parent, () -> {
       childRepository.persist(child);
   });
   ```

5. **Always extend AbstractJpaEntity for JPA entities.** It provides ID generation, optimistic locking, and timestamps.

6. **Implement custom finders in SystemServiceImpl, not in Api.** Keep business query logic in the system service layer.

7. **Use `TxType.REQUIRED` as default.** Only use `REQUIRES_NEW` when you need an independent transaction (e.g., audit logging that must persist even if the main TX fails).

8. **Handle `NoResultException` and `EntityNotFound`.** Repository throws these when `find()` returns no results.

9. **Use `countAll(filter)` before `findAll()` for accurate pagination.** The framework does this automatically in `BaseJpaRepositoryImpl`.

10. **Leverage `QueryOrder` for consistent sorting.** Don't rely on database default ordering.

### Anti-Patterns

1. **Sharing `water-default-persistence-unit` across application modules.** Each module must declare its own persistence unit with a name derived from the module name (`<module-kebab-case>-persistence-unit`). Sharing the default unit causes schema conflicts and test isolation failures when multiple modules run in the same JVM.

2. **Copy-pasting `*RepositoryImpl` without updating the persistence unit constant.** The `yo water:add-entity` generator inherits the persistence unit name from the project's first entity. Every new `*RepositoryImpl` must have its constant updated to `"<entity-kebab-case>-persistence-unit"`. A wrong persistence unit silently attaches the repository to the wrong JPA context — no compile error, only runtime data corruption or entity-not-found failures.

3. **Calling `repository.removeAll()` in production.** This deletes ALL records without any filter.

4. **Building JPA CriteriaQuery manually in service code.** Always use QueryBuilder.

5. **Confusing `findAll` parameter order across layers.** `BaseRepository.findAll` has `Query` as the **3rd** parameter: `findAll(int delta, int page, Query filter, QueryOrder order)`. `BaseEntitySystemApi.findAll` and `BaseEntityServiceImpl.findAll` have `Query` as the **1st** parameter: `findAll(Query filter, int delta, int page, QueryOrder order)`. Mixing these up causes compile errors or silently wrong pagination.

6. **Assigning `getResults()` directly to `List<T>`.** `PaginableResult.getResults()` returns `Collection<T>`. Assigning it to `List<T>` causes a compile error. Always wrap: `new ArrayList<>(result.getResults())`.

3. **Ignoring optimistic locking.** If you bypass `entityVersion`, concurrent updates may overwrite each other silently.

4. **Using `find(String filterStr)` with user input without sanitization.** The QueryParser has basic safety but isn't designed for arbitrary untrusted input.

5. **Creating multiple transactions when one would suffice.** Use the callback pattern instead of separate persist calls.

6. **Putting query logic in the Api layer.** The Api layer is for security-annotated methods; query building belongs in SystemService.

---

## 20. Decision Trees

### How to Query Data

```
Need to query entities?
  |
  +-- Single entity by ID?
  |     --> repository.find(id)
  |
  +-- Single entity by criteria?
  |     |
  |     +-- Programmatic (in service code)?
  |     |     --> qb.field("email").equalTo(email)
  |     |     --> repository.find(query)
  |     |
  |     +-- From REST parameter string?
  |           --> qb.createQueryFilter(filterString)
  |           --> repository.find(query)
  |
  +-- Multiple entities with pagination?
  |     --> repository.findAll(delta, page, filter, order)
  |
  +-- Just need the count?
        --> repository.countAll(filter)
```

### How to Handle Transactions

```
Need transactional behavior?
  |
  +-- Simple save/update/remove?
  |     --> Use repository.persist/update/remove (auto-managed TX)
  |
  +-- Save + related operations in same TX?
  |     --> repository.persist(entity, () -> { /* related ops */ })
  |
  +-- Custom EntityManager operations?
  |     --> repository.tx(TxType.REQUIRED, em -> { /* custom logic */ })
  |
  +-- Need independent TX (audit, logging)?
        --> repository.tx(TxType.REQUIRES_NEW, em -> { ... })
```

---

## 21. Quick Reference Tables

### ⚠️ findAll Signature — Layer Comparison (Most Critical)

| Layer | Class | findAll Signature |
|-------|-------|------------------|
| Repository | `it.water.core.api.repository.BaseRepository` | `findAll(int delta, int page, Query filter, QueryOrder queryOrder)` |
| SystemApi | `it.water.core.api.service.BaseEntitySystemApi` | `findAll(Query filter, int delta, int page, QueryOrder queryOrder)` |
| Api | `it.water.repository.service.BaseEntityServiceImpl` | `findAll(Query filter, int delta, int page, QueryOrder queryOrder)` |

**Query is 3rd at Repository level; Query is 1st at SystemApi/Api level.**

### BaseRepository Methods

| Method | Signature | Returns |
|--------|-----------|---------|
| `persist` | `persist(T entity)` | Saved entity with ID |
| `persist` | `persist(T entity, Runnable executeInTransaction)` | Saved entity |
| `update` | `update(T entity)` | Updated entity |
| `update` | `update(T entity, Runnable executeInTransaction)` | Updated entity |
| `remove` | `remove(long id)` | void |
| `remove` | `remove(long id, Runnable executeInTransaction)` | void |
| `remove` | `remove(T entity)` | void |
| `removeAllByIds` | `removeAllByIds(Iterable<Long> ids)` | void |
| `removeAll` | `removeAll()` | void — **deletes everything** |
| `find` | `find(long id)` | T or EntityNotFound |
| `find` | `find(Query filter)` | T or NoResultException |
| `find` | `find(String filterStr)` | T or NoResultException |
| `findAll` | `findAll(int delta, int page, Query filter, QueryOrder queryOrder)` | `PaginableResult<T>` |
| `countAll` | `countAll(Query filter)` | long |
| `getQueryBuilderInstance` | `getQueryBuilderInstance()` | QueryBuilder |

### FieldNameOperand Methods (on `qb.field("x")`)

| Method | Accepted Types | SQL |
|--------|---------------|-----|
| `equalTo(Object value)` | Any | `=` |
| `notEqualTo(Object value)` | Any | `<>` |
| `greaterThan(Number value)` | Number | `>` |
| `greaterThan(Date value)` | `java.util.Date` | `>` |
| `greaterOrEqualThan(Number value)` | Number | `>=` |
| `greaterOrEqualThan(Date value)` | `java.util.Date` | `>=` |
| `lowerThan(Number value)` | Number | `<` |
| `lowerThan(Date value)` | `java.util.Date` | `<` |
| `lowerOrEqualThan(Number value)` | Number | `<=` |
| `lowerOrEqualThan(Date value)` | `java.util.Date` | `<=` |
| `like(String value)` | String | `LIKE` |

### Query Interface Methods (call on Query result)

| Method | Purpose |
|--------|---------|
| `q.and(q2)` | AND composition |
| `q.or(q2)` | OR composition |
| `q.not()` | Negate |
| `q.in(List<?> values)` | IN list |
| `q.getDefinition()` | Debug: human-readable string |

### Key Packages

| Component | Package |
|-----------|---------|
| `BaseEntity` | `it.water.core.api.model` |
| `PaginableResult` | `it.water.core.api.model` |
| `BaseEntityApi` | `it.water.core.api.service` |
| `BaseEntitySystemApi` | `it.water.core.api.service` |
| `BaseRepository` | `it.water.core.api.repository` |
| `Query` | `it.water.core.api.repository.query` |
| `QueryBuilder` | `it.water.core.api.repository.query` |
| `QueryOrder` | `it.water.core.api.repository.query` |
| `FieldNameOperand` | `it.water.core.api.repository.query.operands` |
| `AbstractEntity` | `it.water.repository.entity.model` |
| `EntityNotFound` | `it.water.repository.entity.model.exceptions` |
| `DuplicateEntityException` | `it.water.repository.entity.model.exceptions` |
| `NoResultException` | `it.water.repository.entity.model.exceptions` |
| `DefaultQueryBuilder` | `it.water.repository.query` |
| `DefaultQueryOrder` | `it.water.repository.query.order` |
| `QueryParser` | `it.water.repository.query.parser` |
| `BaseEntityServiceImpl` | `it.water.repository.service` |
| `BaseEntitySystemServiceImpl` | `it.water.repository.service` |
| `AbstractJpaEntity` | `it.water.repository.jpa.model` |
| `AbstractJpaExpandableEntity` | `it.water.repository.jpa.model` |
| `JpaRepository` (interface) | `it.water.repository.jpa.api` |
| `BaseJpaRepositoryImpl` | `it.water.repository.jpa` |
| `PredicateBuilder` | `it.water.repository.jpa.query` |
| `SpringBaseJpaRepositoryImpl` | `it.water.repository.jpa.spring` |
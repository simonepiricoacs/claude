# Water Framework — Shared Package & Source Reference

> **Usage**: skills load this file on demand via `Read` instead of embedding inline content.
> Source root: `<water-framework-source-root>`

---

## 1. Master FQCN Table

| Class / Interface / Annotation | Package | Module |
|---|---|---|
| `BaseEntity` | `it.water.core.api.model` | Core-api |
| `Resource` | `it.water.core.api.model` | Core-api |
| `PaginableResult<T>` | `it.water.core.api.model` | Core-api |
| `ProtectedResource` | `it.water.core.api.model` | Core-api |
| `ExpandableEntity` | `it.water.core.api.model` | Core-api |
| `EntityExtension` | `it.water.core.api.model` | Core-api |
| `User` | `it.water.core.api.model` | Core-api |
| `Role` | `it.water.core.api.model` | Core-api |
| `BaseEntityApi<T>` | `it.water.core.api.service` | Core-api |
| `BaseEntitySystemApi<T>` | `it.water.core.api.service` | Core-api |
| `BaseApi` | `it.water.core.api.service` | Core-api |
| `BaseSystemApi` | `it.water.core.api.service` | Core-api |
| `Service` | `it.water.core.api.service` | Core-api |
| `RestApi` | `it.water.core.api.service.rest` | Core-api |
| `WaterJsonView` | `it.water.core.api.service.rest` | Core-api |
| `BaseRepository<T>` | `it.water.core.api.repository` | Core-api |
| `Query` | `it.water.core.api.repository.query` | Core-api |
| `QueryBuilder` | `it.water.core.api.repository.query` | Core-api |
| `QueryOrder` | `it.water.core.api.repository.query` | Core-api |
| `QueryOrderParameter` | `it.water.core.api.repository.query` | Core-api |
| `FieldNameOperand` | `it.water.core.api.repository.query.operands` | Core-api |
| `ComponentRegistry` | `it.water.core.api.registry` | Core-api |
| `ComponentRegistration` | `it.water.core.api.registry` | Core-api |
| `ComponentConfiguration` | `it.water.core.api.registry` | Core-api |
| `ComponentFilter` | `it.water.core.api.registry.filter` | Core-api |
| `ComponentFilterBuilder` | `it.water.core.api.registry.filter` | Core-api |
| `Runtime` | `it.water.core.api.bundle` | Core-api |
| `ApplicationProperties` | `it.water.core.api.bundle` | Core-api |
| `PermissionManager` | `it.water.core.api.permission` | Core-api |
| `SecurityContext` | `it.water.core.api.permission` | Core-api |
| `Action` | `it.water.core.api.action` | Core-api |
| `AuthenticationProvider` | `it.water.core.api.security` | Core-api |
| `Authenticable` | `it.water.core.api.security` | Core-api |
| `WaterValidator` | `it.water.core.api.validation` | Core-api |
| `@FrameworkComponent` | `it.water.core.interceptors.annotations` | Core-interceptors |
| `@Inject` | `it.water.core.interceptors.annotations` | Core-interceptors |
| `@OnActivate` | `it.water.core.api.interceptors` | Core-api |
| `@OnDeactivate` | `it.water.core.api.interceptors` | Core-api |
| `@AllowPermissions` | `it.water.core.permission.annotations` | Core-permission |
| `@AllowRoles` | `it.water.core.permission.annotations` | Core-permission |
| `@AllowGenericPermissions` | `it.water.core.permission.annotations` | Core-permission |
| `@AllowPermissionsOnReturn` | `it.water.core.permission.annotations` | Core-permission |
| `@AccessControl` | `it.water.core.permission.annotations` | Core-permission |
| `@DefaultRoleAccess` | `it.water.core.permission.annotations` | Core-permission |
| `CrudActions` | `it.water.core.permission.action` | Core-permission |
| `UnauthorizedException` | `it.water.core.permission.exceptions` | Core-permission |
| `WaterRuntimeException` | `it.water.core.model.exceptions` | Core-model |
| `AbstractResource` | `it.water.core.model` | Core-model |
| `AbstractEntity` | `it.water.repository.entity.model` | Repository-entity |
| `EntityNotFound` | `it.water.repository.entity.model.exceptions` | Repository-entity |
| `DuplicateEntityException` | `it.water.repository.entity.model.exceptions` | Repository-entity |
| `NoResultException` | `it.water.repository.entity.model.exceptions` | Repository-entity |
| `DefaultQueryBuilder` | `it.water.repository.query` | Repository-persistence |
| `DefaultQueryOrder` | `it.water.repository.query.order` | Repository-persistence |
| `QueryParser` | `it.water.repository.query.parser` | Repository-persistence |
| `BaseEntityServiceImpl<T>` | `it.water.repository.service` | Repository-service |
| `BaseEntitySystemServiceImpl<T>` | `it.water.repository.service` | Repository-service |
| `AbstractJpaEntity` | `it.water.repository.jpa.model` | JpaRepository-api |
| `AbstractJpaExpandableEntity` | `it.water.repository.jpa.model` | JpaRepository-api |
| `JpaRepository<T>` | `it.water.repository.jpa.api` | JpaRepository-api |
| `BaseJpaRepositoryImpl<T>` | `it.water.repository.jpa` | JpaRepository-api |
| `PredicateBuilder` | `it.water.repository.jpa.query` | JpaRepository-api |
| `SpringBaseJpaRepositoryImpl<T>` | `it.water.repository.jpa.spring` | JpaRepository-spring |
| `BaseEntityRestApi<T>` | `it.water.service.rest.persistence` | Rest-persistence |
| `AuthenticationApi` | `it.water.authentication.api` | Authentication-api |
| `AuthenticationSystemApi` | `it.water.authentication.api` | Authentication-api |
| `WaterUser` | `it.water.user.model` | User-model |

---

## 2. Source File Paths (relative to source root)

Read these files directly for exact up-to-date signatures.

| Key Interface | Relative Path |
|---|---|
| `BaseEntity` | `Core/Core-api/src/main/java/it/water/core/api/model/BaseEntity.java` |
| `BaseEntityApi` | `Core/Core-api/src/main/java/it/water/core/api/service/BaseEntityApi.java` |
| `BaseEntitySystemApi` | `Core/Core-api/src/main/java/it/water/core/api/service/BaseEntitySystemApi.java` |
| `BaseRepository` | `Core/Core-api/src/main/java/it/water/core/api/repository/BaseRepository.java` |
| `Query` | `Core/Core-api/src/main/java/it/water/core/api/repository/query/Query.java` |
| `QueryBuilder` | `Core/Core-api/src/main/java/it/water/core/api/repository/query/QueryBuilder.java` |
| `QueryOrder` | `Core/Core-api/src/main/java/it/water/core/api/repository/query/QueryOrder.java` |
| `FieldNameOperand` | `Core/Core-api/src/main/java/it/water/core/api/repository/query/operands/FieldNameOperand.java` |
| `PaginableResult` | `Core/Core-api/src/main/java/it/water/core/api/model/PaginableResult.java` |
| `ComponentRegistry` | `Core/Core-api/src/main/java/it/water/core/api/registry/ComponentRegistry.java` |
| `Runtime` | `Core/Core-api/src/main/java/it/water/core/api/bundle/Runtime.java` |
| `ApplicationProperties` | `Core/Core-api/src/main/java/it/water/core/api/bundle/ApplicationProperties.java` |
| `PermissionManager` | `Core/Core-api/src/main/java/it/water/core/api/permission/PermissionManager.java` |
| `SecurityContext` | `Core/Core-api/src/main/java/it/water/core/api/permission/SecurityContext.java` |
| `AuthenticationProvider` | `Core/Core-api/src/main/java/it/water/core/api/security/AuthenticationProvider.java` |
| `@FrameworkComponent` | `Core/Core-interceptors/src/main/java/it/water/core/interceptors/annotations/FrameworkComponent.java` |
| `@Inject` | `Core/Core-interceptors/src/main/java/it/water/core/interceptors/annotations/Inject.java` |
| `@OnActivate` | `Core/Core-api/src/main/java/it/water/core/api/interceptors/OnActivate.java` |
| `@AllowPermissions` | `Core/Core-permission/src/main/java/it/water/core/permission/annotations/AllowPermissions.java` |
| `@AllowRoles` | `Core/Core-permission/src/main/java/it/water/core/permission/annotations/AllowRoles.java` |
| `@AccessControl` | `Core/Core-permission/src/main/java/it/water/core/permission/annotations/AccessControl.java` |
| `AbstractEntity` | `Repository/Repository-entity/src/main/java/it/water/repository/entity/model/AbstractEntity.java` |
| `EntityNotFound` | `Repository/Repository-entity/src/main/java/it/water/repository/entity/model/exceptions/EntityNotFound.java` |
| `DefaultQueryBuilder` | `Repository/Repository-persistence/src/main/java/it/water/repository/query/DefaultQueryBuilder.java` |
| `DefaultQueryOrder` | `Repository/Repository-persistence/src/main/java/it/water/repository/query/order/DefaultQueryOrder.java` |
| `BaseEntitySystemServiceImpl` | `Repository/Repository-service/src/main/java/it/water/repository/service/BaseEntitySystemServiceImpl.java` |
| `BaseEntityServiceImpl` | `Repository/Repository-service/src/main/java/it/water/repository/service/BaseEntityServiceImpl.java` |
| `AbstractJpaEntity` | `JpaRepository/JpaRepository-api/src/main/java/it/water/repository/jpa/model/AbstractJpaEntity.java` |
| `JpaRepository` | `JpaRepository/JpaRepository-api/src/main/java/it/water/repository/jpa/api/JpaRepository.java` |
| `BaseJpaRepositoryImpl` | `JpaRepository/JpaRepository-api/src/main/java/it/water/repository/jpa/BaseJpaRepositoryImpl.java` |
| `BaseEntityRestApi` | `Rest/Rest-persistence/src/main/java/it/water/service/rest/persistence/BaseEntityRestApi.java` |
| `RestApi` | `Core/Core-api/src/main/java/it/water/core/api/service/rest/RestApi.java` |
| `AuthenticationApi` | `Authentication/Authentication-api/src/main/java/it/water/authentication/api/AuthenticationApi.java` |
| `AuthenticationSystemApi` | `Authentication/Authentication-api/src/main/java/it/water/authentication/api/AuthenticationSystemApi.java` |
| `WaterUser` | `User/User-model/src/main/java/it/water/user/model/WaterUser.java` |

---

## 3. Standard Import Blocks by Use-Case

### Service Implementation (XxxServiceImpl)
```java
import it.water.core.interceptors.annotations.FrameworkComponent;
import it.water.core.interceptors.annotations.Inject;
import it.water.core.api.registry.ComponentRegistry;
import it.water.core.api.service.BaseEntitySystemApi;
import it.water.core.api.model.PaginableResult;
import it.water.core.api.repository.query.Query;
import it.water.core.api.repository.query.QueryOrder;
import it.water.core.permission.annotations.AllowPermissions;
import it.water.core.permission.action.CrudActions;
import it.water.repository.service.BaseEntityServiceImpl;
import lombok.Setter;
```

### SystemService Implementation (XxxSystemServiceImpl)
```java
import it.water.core.interceptors.annotations.FrameworkComponent;
import it.water.core.interceptors.annotations.Inject;
import it.water.core.api.registry.ComponentRegistry;
import it.water.core.api.repository.BaseRepository;
import it.water.core.api.repository.query.Query;
import it.water.core.api.repository.query.QueryBuilder;
import it.water.core.api.repository.query.QueryOrder;
import it.water.core.api.model.PaginableResult;
import it.water.repository.service.BaseEntitySystemServiceImpl;
import it.water.repository.query.order.DefaultQueryOrder;
import lombok.Setter;
```

### Repository Implementation (XxxRepositoryImpl)
```java
import it.water.core.interceptors.annotations.FrameworkComponent;
import it.water.core.api.model.BaseEntity;
import it.water.core.api.model.PaginableResult;
import it.water.core.api.repository.query.Query;
import it.water.core.api.repository.query.QueryOrder;
import it.water.repository.jpa.BaseJpaRepositoryImpl;
import it.water.repository.query.DefaultQueryBuilder;
import it.water.repository.entity.model.exceptions.EntityNotFound;
import it.water.repository.entity.model.exceptions.DuplicateEntityException;
import jakarta.persistence.*;
```

### REST Controller (XxxRestControllerImpl)
```java
import it.water.core.interceptors.annotations.FrameworkComponent;
import it.water.core.interceptors.annotations.Inject;
import it.water.core.api.service.BaseEntityApi;
import it.water.core.api.model.PaginableResult;
import it.water.core.api.repository.query.Query;
import it.water.core.api.repository.query.QueryOrder;
import it.water.service.rest.persistence.BaseEntityRestApi;
import lombok.Setter;
```

### Permission / Security
```java
import it.water.core.permission.annotations.AllowPermissions;
import it.water.core.permission.annotations.AllowRoles;
import it.water.core.permission.annotations.AllowGenericPermissions;
import it.water.core.permission.annotations.AllowPermissionsOnReturn;
import it.water.core.permission.annotations.AccessControl;
import it.water.core.permission.annotations.DefaultRoleAccess;
import it.water.core.permission.action.CrudActions;
import it.water.core.permission.exceptions.UnauthorizedException;
import it.water.core.api.permission.PermissionManager;
import it.water.core.api.permission.SecurityContext;
```

### Entity Model (JPA)
```java
import it.water.repository.jpa.model.AbstractJpaEntity;
import it.water.core.api.service.rest.WaterJsonView;
import it.water.core.permission.annotations.AccessControl;
import it.water.core.api.entity.owned.OwnedResource;
import com.fasterxml.jackson.annotation.JsonView;
import jakarta.persistence.*;
import jakarta.validation.constraints.*;
import lombok.*;
```

---

## 4. Critical Code-Generation Traps

These are non-obvious mistakes that cause compile errors or silent bugs. **Always verify before generating code**.

### Trap 1 — findAll parameter order differs by layer

| Layer | Class | findAll Signature |
|---|---|---|
| Repository | `it.water.core.api.repository.BaseRepository` | `findAll(int delta, int page, Query filter, QueryOrder order)` — Query 3rd |
| SystemApi | `it.water.core.api.service.BaseEntitySystemApi` | `findAll(Query filter, int delta, int page, QueryOrder order)` — Query 1st |
| Api | `it.water.repository.service.BaseEntityServiceImpl` | `findAll(Query filter, int delta, int page, QueryOrder order)` — Query 1st |
| REST | `it.water.service.rest.persistence.BaseEntityRestApi` | `findAll(Integer delta, Integer page, Query filter, QueryOrder order)` — Integer (nullable) |

### Trap 2 — @Path prefix in CXF REST
CXF adds `/water` as base context automatically. Never include `/water` in `@Path`:
```java
@Path("/myentities")       // ✅ → full URL: /water/myentities
@Path("/water/myentities") // ❌ → full URL: /water/water/myentities (404)
```

### Trap 3 — PaginableResult.getResults() returns Collection, not List
```java
List<T> list = result.getResults();              // ❌ compile error
List<T> list = new ArrayList<>(result.getResults()); // ✅
```

### Trap 4 — Persistence unit name per entity
Every `*RepositoryImpl` must pass its own persistence unit name (not the default):
```java
super("myentity-persistence-unit", MyEntity.class);  // ✅
super("water-default-persistence-unit", ...);          // ❌ never in app code
```
`yo water:add-entity` copies the first entity's persistence unit — always fix it manually.

### Trap 5 — @FrameworkComponent.priority default is 1 (lowest)
Higher numbers = higher priority. Custom components need `priority > 1` to override framework defaults.

### Trap 6 — @Inject.injectOnceAtStartup default is false (dynamic injection)
Set `true` only for global singletons that must not be re-injected dynamically.

### Trap 7 — CrudActions string values (not enum)
```java
CrudActions.SAVE     = "save"
CrudActions.UPDATE   = "update"
CrudActions.REMOVE   = "remove"
CrudActions.FIND     = "find"
CrudActions.FIND_ALL = "findAll"   // NOT "find-all", NOT "find_all"
```

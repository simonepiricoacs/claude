---
name: reference-water-module-template
description: Canonical 4-module decomposition for any Water domain module (model → api → service → service-spring)
metadata:
  type: reference
---

The canonical Water Framework decomposition for a new domain module is four sub-modules:

1. `<Domain>-model` — JPA entities, enums, DTOs, criteria POJOs. Depends only on Water `Core-model` + `Core-api` + Jakarta Persistence/Validation APIs.
2. `<Domain>-api` — `*Api`, `*SystemApi`, `*RestApi`, `*Repository` interfaces. Depends on `-model` + Water `Core-api`, `Core-security`, `Repository-api`, `Core-rest-api`, plus JAX-RS annotations.
3. `<Domain>-service` — `*ServiceImpl` (FrameworkComponent), `*RepositoryImpl` (extends `BaseJpaRepositoryImpl`). Depends on `-api` + Water `Core-service`, `Core-interceptors`, `Repository-jpa`, `Repository-service`, Hibernate.
4. `<Domain>-service-spring` (or `-service-osgi`, `-service-quarkus`) — Spring Boot fat JAR / OSGi bundle. Contains `*RestController` implementations, `Application` class, `application.properties`, schedulers, runtime-specific glue.

The yo water generator scaffolds this layout by default. Use `applicationType=service` for connector/integration modules and add code manually after.

This template applies to all Water domain modules including ApiGateway, User, and new modules like HomeLibrary [[project-homelibrary]].

# WATER FRAMEWORK KARATE TESTING KNOWLEDGE BASE

You are an expert Water Framework Karate testing specialist with deep knowledge of REST API testing patterns, feature file structure, karate-config.js configuration, test runner setup, and integration with Water test runtime (WaterTestExtension, TestRuntimeInitializer).

---

## Table of Contents

1. [Karate Testing Overview](#1-karate-testing-overview)
2. [Project Structure](#2-project-structure)
3. [Test Runner Configuration](#3-test-runner-configuration)
4. [karate-config.js Setup](#4-karate-configjs-setup)
5. [Feature File Patterns](#5-feature-file-patterns)
6. [URL Construction Standard](#6-url-construction-standard)
7. [Standard Scenario Patterns](#7-standard-scenario-patterns)
8. [Entity CRUD Testing Pattern](#8-entity-crud-testing-pattern)
9. [Custom REST API Testing](#9-custom-rest-api-testing)
10. [Authentication & Security Testing](#10-authentication--security-testing)
11. [Response Matching Patterns](#11-response-matching-patterns)
12. [Error Testing Patterns](#12-error-testing-patterns)
13. [Workflow Testing Patterns](#13-workflow-testing-patterns)
14. [Water Framework Integration](#14-water-framework-integration)
15. [Common Mistakes & Fixes](#15-common-mistakes--fixes)
16. [Best Practices](#16-best-practices)
17. [Troubleshooting Guide](#17-troubleshooting-guide)
18. [Quick Reference](#18-quick-reference)

---

## 1. Karate Testing Overview

**Karate** is a BDD-style REST API testing framework that Water Framework uses for integration testing of REST endpoints.

### Key Features in Water Context

- **Zero-code REST testing**: Gherkin-style feature files
- **JSON matching**: Schema-less validation with fuzzy matching
- **Variable capture**: Extract response values for subsequent requests
- **Dynamic port handling**: Integration with `TestRuntimeInitializer`
- **Full HTTP support**: Headers, query params, path params, request bodies
- **Built-in retry logic**: Via Karate's `retry until` feature

### Karate Version

Water Framework uses **Karate 1.5.0.RC3** (io.karatelabs:karate-junit5).

---

## 2. Project Structure

### Standard Module Structure

```
MyEntity-service/
  ├── build.gradle
  │   └── testImplementation ('io.karatelabs:karate-junit5:1.5.0.RC3')
  │
  ├── src/test/java/
  │   └── com/example/myentity/
  │       └── MyEntityRestApiTest.java          # JUnit test runner
  │
  └── src/test/resources/
      ├── karate-config.js                       # Global Karate config
      ├── karate/
      │   └── entity.feature                     # Feature file(s)
      ├── it.water.application.properties        # Water properties
      └── META-INF/
          └── persistence.xml                    # JPA config (if needed)
```

### File Purposes

| File | Purpose |
|------|---------|
| `MyEntityRestApiTest.java` | JUnit 5 test runner that launches Karate, integrates with Water test runtime |
| `karate-config.js` | Global configuration loaded before all features, builds `serviceBaseUrl` |
| `entity.feature` | Gherkin feature file with REST API test scenarios |
| `it.water.application.properties` | Water Framework properties (context root, JWT config, etc.) |

---

## 3. Test Runner Configuration

### Standard Test Runner Pattern

```java
package it.water.mymodule;

import com.intuit.karate.junit5.Karate;
import it.water.core.api.registry.ComponentRegistry;
import it.water.core.api.service.Service;
import it.water.core.interceptors.annotations.Inject;
import it.water.core.testing.utils.bundle.TestRuntimeInitializer;
import it.water.core.testing.utils.junit.WaterTestExtension;
import it.water.core.testing.utils.runtime.TestRuntimeUtils;
import lombok.Setter;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.extension.ExtendWith;

@ExtendWith(WaterTestExtension.class)
public class MyEntityRestApiTest implements Service {

    @Inject
    @Setter
    private ComponentRegistry componentRegistry;

    @BeforeEach
    void impersonateAdmin() {
        // JWT token service is disabled in tests, we inject admin user for bypassing permission system
        // Remove this line if you want to test with permission system working
        TestRuntimeUtils.impersonateAdmin(componentRegistry);
    }

    @Karate.Test
    Karate restInterfaceTest() {
        return Karate.run("classpath:karate")
                .systemProperty("webServerPort", TestRuntimeInitializer.getInstance().getRestServerPort())
                .systemProperty("host", "localhost")
                .systemProperty("protocol", "http");
    }
}
```

### Key Components

| Component | Purpose |
|-----------|---------|
| `@ExtendWith(WaterTestExtension.class)` | Initializes Water test runtime (component registry, REST server) |
| `implements Service` | Marks class as Water service (enables @Inject) |
| `@Inject ComponentRegistry` | Injects component registry for test setup |
| `TestRuntimeUtils.impersonateAdmin()` | Bypasses permission checks by injecting admin user |
| `TestRuntimeInitializer.getInstance()` | Access to test REST server and port |
| `Karate.run("classpath:karate")` | Runs all .feature files in src/test/resources/karate/ |
| `.systemProperty("webServerPort", ...)` | Passes dynamic port to karate-config.js |

### Important Notes

- **Dynamic Port**: Water test server uses random port to avoid conflicts — `TestRuntimeInitializer.getInstance().getRestServerPort()` provides it
- **Admin Impersonation**: `impersonateAdmin()` bypasses `@LoggedIn` and `@AllowPermissions` checks
- **Feature Path**: `"classpath:karate"` runs ALL .feature files in `src/test/resources/karate/` directory

---

## 4. karate-config.js Setup

### Standard Configuration

**File:** `src/test/resources/karate-config.js`

```javascript
function fn() {
    let webServerPort = karate.properties['webServerPort'];
    let host = karate.properties['host'];
    let protocol = karate.properties['protocol'];
    let serviceBaseUrl = protocol+"://"+host+":"+webServerPort;
    return {
        "serviceBaseUrl": serviceBaseUrl
    }
}
```

### What It Does

1. Reads system properties passed from Java test runner
2. Constructs `serviceBaseUrl` = `http://localhost:PORT` (dynamic port)
3. Returns config object available to all feature files

### Extended Configuration (with JWT)

```javascript
function fn() {
    let webServerPort = karate.properties['webServerPort'];
    let host = karate.properties['host'];
    let protocol = karate.properties['protocol'];
    let serviceBaseUrl = protocol+"://"+host+":"+webServerPort;

    // Optional: JWT token for authenticated tests
    let jwtToken = karate.properties['jwtToken'];

    return {
        "serviceBaseUrl": serviceBaseUrl,
        "jwtToken": jwtToken
    }
}
```

### Variable Usage in Features

Variables defined in `karate-config.js` are globally available:

```gherkin
Given url serviceBaseUrl+'/water/myendpoint'
# serviceBaseUrl is automatically available — no need to define it
```

---

## 5. Feature File Patterns

### Feature File Header

```gherkin
# Generated with Water Generator
# The Goal of feature test is to ensure the correct format of json responses
# If you want to perform functional test please refer to ApiTest
Feature: Check MyEntity Rest Api Response
```

### Feature Structure

```
Feature: <Name>

  Scenario: <Test 1>
    Given <preconditions>
    When <action>
    Then <assertions>

  Scenario: <Test 2>
    ...

  Scenario Outline: <Parameterized Test>
    Given <preconditions with <param>>
    When <action>
    Then <assertion>

    Examples:
      | param | expected |
      | val1  | result1  |
      | val2  | result2  |
```

### **CRITICAL: NO Background Section**

**❌ WRONG:**
```gherkin
Feature: My Tests

  Background:
    * url serviceBaseUrl   # DO NOT USE THIS PATTERN

  Scenario: Test endpoint
    Given path '/myendpoint'
    When method GET
    Then status 200
```

**✅ CORRECT:**
```gherkin
Feature: My Tests

  Scenario: Test endpoint
    Given url serviceBaseUrl+'/water/myendpoint'
    When method GET
    Then status 200
```

**Why?** Water Framework REST endpoints include a context root (`/water` by default). Using `Background` with `* url serviceBaseUrl` + `Given path '/endpoint'` does NOT correctly append the context root. Always use `Given url serviceBaseUrl+'/water/endpoint'` pattern.

---

## 6. URL Construction Standard

### URL Pattern

```
Given url serviceBaseUrl + '/water' + '/endpoint'
         └─────┬─────┘     └──┬──┘   └───┬────┘
               │              │           │
               │              │           └─ REST API path from @Path annotation
               │              └─ Water Framework context root (configurable)
               └─ Protocol://host:port (from karate-config.js)
```

### Standard Examples

```gherkin
# Entity CRUD endpoints
Given url serviceBaseUrl+'/water/roles'              # List/Create
Given url serviceBaseUrl+'/water/roles/'+entityId    # Get/Update/Delete by ID

# Custom endpoints
Given url serviceBaseUrl+'/water/messaging/health'
Given url serviceBaseUrl+'/water/messaging/publish?channel=test-channel'
Given url serviceBaseUrl+'/water/entities/shared/findByPK'

# With path parameters
Given url serviceBaseUrl+'/water/channels/'+channelName+'/subscribers'

# With query parameters
Given url serviceBaseUrl+'/water/roles?delta=10&page=1&filter=name==test'
```

### Context Root Configuration

**File:** `src/test/resources/it.water.application.properties`

```properties
# Standard Water Framework context root
water.rest.root.context=/water

# Or empty for no context root (non-standard)
water.rest.root.context=
```

If using empty context root, URLs become:
```gherkin
Given url serviceBaseUrl+'/messaging/health'  # No /water prefix
```

**Best Practice:** Always use `/water` context root to match production deployments.

---

## 7. Standard Scenario Patterns

### GET Request

```gherkin
Scenario: Get entity by ID
  Given header Content-Type = 'application/json'
  And header Accept = 'application/json'
  Given url serviceBaseUrl+'/water/roles/'+entityId
  When method GET
  Then status 200
  And match response == { id: '#number', name: '#string' }
```

### POST Request

```gherkin
Scenario: Create entity
  Given header Content-Type = 'application/json'
  And header Accept = 'application/json'
  Given url serviceBaseUrl+'/water/roles'
  And request { "name": "Admin", "description": "Administrator role" }
  When method POST
  Then status 200
  And match response.id == '#number'
  And match response.name == 'Admin'
  * def entityId = response.id
```

### PUT Request

```gherkin
Scenario: Update entity
  Given header Content-Type = 'application/json'
  And header Accept = 'application/json'
  Given url serviceBaseUrl+'/water/roles'
  And request { "id": '#(entityId)', "entityVersion": 1, "name": "Updated Name" }
  When method PUT
  Then status 200
  And match response.entityVersion == 2
  And match response.name == 'Updated Name'
```

### DELETE Request

```gherkin
Scenario: Delete entity
  Given header Content-Type = 'application/json'
  And header Accept = 'application/json'
  Given url serviceBaseUrl+'/water/roles/'+entityId
  When method DELETE
  Then status 204  # No Content
```

### Query Parameters

```gherkin
Scenario: Paginated list
  Given header Content-Type = 'application/json'
  And header Accept = 'application/json'
  Given url serviceBaseUrl+'/water/roles?delta=10&page=1'
  When method GET
  Then status 200
  And match response.results == '#array'
  And match response.numPages == '#number'
```

---

## 8. Entity CRUD Testing Pattern

### Standard Entity CRUD Workflow

**File:** `src/test/resources/karate/entity.feature`

```gherkin
Feature: Check Role Rest Api Response

  Scenario: Role CRUD Operations

    # ==================== CREATE ====================
    Given header Content-Type = 'application/json'
    And header Accept = 'application/json'
    Given url serviceBaseUrl+'/water/roles'
    And request { "name": "exampleField", "description": "role" }
    When method POST
    Then status 200
    And match response ==
    """
      { "id": #number,
        "name": "exampleField",
        "entityVersion": 1,
        "entityCreateDate": '#number',
        "entityModifyDate": '#number',
        "description": "role"
       }
    """
    * def entityId = response.id

    # ==================== UPDATE ====================
    Given header Content-Type = 'application/json'
    And header Accept = 'application/json'
    Given url serviceBaseUrl+'/water/roles'
    And request { "id": "#(entityId)", "entityVersion": 1, "name": "nameUpdated", "description": "description" }
    When method PUT
    Then status 200
    And match response ==
    """
      { "id": #number,
        "entityVersion": 2,
        "entityCreateDate": '#number',
        "entityModifyDate": '#number',
        "name": "nameUpdated",
        "description": "description"
       }
    """

    # ==================== FIND ====================
    Given header Content-Type = 'application/json'
    And header Accept = 'application/json'
    Given url serviceBaseUrl+'/water/roles/'+entityId
    When method GET
    Then status 200
    And match response ==
    """
      { "id": #number,
        "entityVersion": 2,
        "entityCreateDate": '#number',
        "entityModifyDate": '#number',
        "name": 'nameUpdated',
        "description": "description"
       }
    """

    # ==================== FIND ALL ====================
    Given header Content-Type = 'application/json'
    And header Accept = 'application/json'
    Given url serviceBaseUrl+'/water/roles'
    When method GET
    Then status 200
    And match response.results contains
    """
    {
      "id": #number,
      "entityVersion": 2,
      "entityCreateDate": '#number',
      "entityModifyDate": '#number',
      "name": 'nameUpdated',
      "description": "description"
    }
    """

    # ==================== DELETE ====================
    Given header Content-Type = 'application/json'
    And header Accept = 'application/json'
    Given url serviceBaseUrl+'/water/roles/'+entityId
    When method DELETE
    # 204 because delete response is empty, so the status code is "no content" but is ok
    Then status 204
```

### Key Points

- **Single Scenario**: All CRUD operations in one scenario (ensures proper cleanup)
- **Variable Capture**: `* def entityId = response.id` stores ID for subsequent requests
- **Entity Version**: Water entities have `entityVersion` that increments on update
- **Timestamps**: `entityCreateDate` and `entityModifyDate` are epoch milliseconds
- **Delete Status**: 204 No Content (not 200)

---

## 9. Custom REST API Testing

### Non-Entity Endpoints

For custom REST APIs (not entity CRUD), test each endpoint individually:

```gherkin
Feature: Redis Messaging Gateway REST API Tests

  Scenario: Health check should return status
    Given header Content-Type = 'application/json'
    And header Accept = 'application/json'
    Given url serviceBaseUrl+'/water/messaging/health'
    When method GET
    Then status 200
    And match response.timestamp == '#number'
    And match response.status == '#string'

  Scenario: Publish message to channel
    Given header Content-Type = 'application/json'
    And header Accept = 'application/json'
    Given url serviceBaseUrl+'/water/messaging/publish?channel=test-channel'
    And request { message: 'Test message' }
    When method POST
    Then status 200
    And match response.success == true
    And match response.channel == 'test-channel'
    And match response.subscribers == '#number'

  Scenario: Get metrics
    Given header Content-Type = 'application/json'
    And header Accept = 'application/json'
    Given url serviceBaseUrl+'/water/messaging/metrics'
    When method GET
    Then status 200
    And match response.totalPublishedMessages == '#number'
    And match response.averageLatencyMs == '#number'
```

---

## 10. Authentication & Security Testing

### With JWT Token (if enabled)

If JWT validation is enabled (`water.rest.security.jwt.validate=true`), you need to pass a token:

**karate-config.js:**
```javascript
function fn() {
    let webServerPort = karate.properties['webServerPort'];
    let host = karate.properties['host'];
    let protocol = karate.properties['protocol'];
    let jwtToken = karate.properties['jwtToken']; // Passed from test runner
    let serviceBaseUrl = protocol+"://"+host+":"+webServerPort;
    return {
        "serviceBaseUrl": serviceBaseUrl,
        "authToken": "Bearer " + jwtToken
    }
}
```

**Feature file:**
```gherkin
Scenario: Authenticated request
  Given header Content-Type = 'application/json'
  And header Accept = 'application/json'
  And header Authorization = authToken
  Given url serviceBaseUrl+'/water/roles'
  When method GET
  Then status 200
```

### Without JWT (Test Mode)

**File:** `src/test/resources/it.water.application.properties`

```properties
# Disable JWT validation in tests
water.rest.security.jwt.validate=false
```

**Test Runner:**
```java
@BeforeEach
void impersonateAdmin() {
    // Bypass permission system by injecting admin user
    TestRuntimeUtils.impersonateAdmin(componentRegistry);
}
```

This allows testing without JWT tokens while still respecting permission annotations.

### Testing Unauthorized Access

```gherkin
Scenario: Request without authentication should fail
  Given header Content-Type = 'application/json'
  And header Accept = 'application/json'
  # No Authorization header
  Given url serviceBaseUrl+'/water/messaging/publish?channel=test'
  And request { message: 'test' }
  When method POST
  Then status 401
```

---

## 11. Response Matching Patterns

### Exact Match

```gherkin
Then match response == { id: 1, name: 'Admin' }
```

### Fuzzy Match (Schema Validation)

```gherkin
# Type matching
Then match response == { id: '#number', name: '#string' }

# Optional field
Then match response == { id: '#number', name: '#string', description: '##string' }

# Array
Then match response.results == '#array'

# Non-null
Then match response.id == '#notnull'

# Regex
Then match response.name == '#regex [A-Z][a-z]+'

# Object
Then match response.details == '#object'

# Boolean
Then match response.active == '#boolean'

# Present (any type)
Then match response.timestamp == '#present'

# Null
Then match response.deletedAt == '#null'
```

### Array Matching

```gherkin
# Array contains specific item
Then match response.results contains { id: '#number', name: 'Admin' }

# Array has exact length
Then match response.results == '#[3]'  # Exactly 3 elements

# Array has minimum length
Then assert response.results.length >= 2
```

### Partial Match

```gherkin
# Match subset of fields
Then match response contains { name: 'Admin', active: true }
# (ignores other fields)
```

### Nested Match

```gherkin
Then match response ==
"""
{
  id: '#number',
  user: {
    name: '#string',
    roles: '#array'
  },
  metadata: '#object'
}
"""
```

---

## 12. Error Testing Patterns

### Validation Errors (400 Bad Request)

```gherkin
Scenario: Publish without channel should fail
  Given header Content-Type = 'application/json'
  And header Accept = 'application/json'
  Given url serviceBaseUrl+'/water/messaging/publish'
  And request { message: 'test' }
  When method POST
  Then status 400
  And match response.error == '#string'
```

### Not Found (404)

```gherkin
Scenario: Get non-existent entity
  Given header Content-Type = 'application/json'
  And header Accept = 'application/json'
  Given url serviceBaseUrl+'/water/roles/99999'
  When method GET
  Then status 404
```

### Unauthorized (401)

```gherkin
Scenario: Request without authentication
  # No Authorization header
  Given url serviceBaseUrl+'/water/roles'
  When method GET
  Then status 401
```

### Forbidden (403)

```gherkin
Scenario: User without permission
  # User with valid token but no permission for the resource
  Given header Authorization = userToken
  Given url serviceBaseUrl+'/water/admin/users'
  When method GET
  Then status 403
```

### Method Not Allowed (405)

```gherkin
Scenario: Invalid HTTP method
  Given url serviceBaseUrl+'/water/messaging/health'
  When method POST
  Then status 405
```

### Conflict (409)

```gherkin
Scenario: Duplicate entity
  Given url serviceBaseUrl+'/water/roles'
  And request { "name": "Admin", "description": "role" }
  When method POST
  Then status 200

  # Try to create again with same name
  Given url serviceBaseUrl+'/water/roles'
  And request { "name": "Admin", "description": "role" }
  When method POST
  Then status 409
  And match response.type == 'DuplicateEntityException'
```

### Validation Error (422)

```gherkin
Scenario: Invalid entity fields
  Given url serviceBaseUrl+'/water/roles'
  And request { "name": "", "description": "role" }
  When method POST
  Then status 422
  And match response.type == 'ValidationException'
  And match response.validationErrors == '#array'
  And match response.validationErrors[0].field == 'name'
```

---

## 13. Workflow Testing Patterns

### Multi-Step Workflow

```gherkin
Scenario: Complete stream workflow
  * def workflowStream = 'test-stream-workflow'

  # Step 1: Add message 1
  Given url serviceBaseUrl+'/water/messaging/stream/'+workflowStream
  And request { event: 'first', seq: 1 }
  When method POST
  Then status 200
  * def messageId1 = response.messageId

  # Step 2: Add message 2
  Given url serviceBaseUrl+'/water/messaging/stream/'+workflowStream
  And request { event: 'second', seq: 2 }
  When method POST
  Then status 200
  * def messageId2 = response.messageId

  # Step 3: Read messages
  Given url serviceBaseUrl+'/water/messaging/stream/'+workflowStream+'?messageId=0&count=10'
  When method GET
  Then status 200
  And assert response.messages.length >= 2

  # Step 4: Get stream length
  Given url serviceBaseUrl+'/water/messaging/stream/'+workflowStream+'/length'
  When method GET
  Then status 200
  And assert response.length >= 2

  # Step 5: Create consumer group
  Given url serviceBaseUrl+'/water/messaging/stream/'+workflowStream+'/group/test-group?startId=0'
  When method POST
  Then status 200

  # Step 6: Read as consumer
  Given url serviceBaseUrl+'/water/messaging/stream/'+workflowStream+'/group/test-group/consumer/worker-1?count=10'
  When method GET
  Then status 200
  And match response.messages == '#array'
```

### Parameterized Tests (Scenario Outline)

```gherkin
Scenario Outline: Channel name validation
  Given url serviceBaseUrl+'/water/messaging/publish?channel=<channel>'
  And request { message: 'test' }
  When method POST
  Then status <expectedStatus>

  Examples:
    | channel           | expectedStatus |
    | valid-channel     | 200            |
    | valid_channel     | 200            |
    | valid:channel     | 200            |
    | valid.channel     | 200            |
    | channel123        | 200            |
    | invalid channel   | 400            |
    | invalid!channel   | 400            |
    | invalid@channel   | 400            |
```

---

## 14. Water Framework Integration

### Test Runtime Initialization

When `@ExtendWith(WaterTestExtension.class)` is used, the following happens:

1. **ComponentRegistry** is initialized
2. **@FrameworkComponent** classes are discovered and registered
3. **RestApiRegistry** is populated with `@FrameworkRestApi` interfaces
4. **RestApiManager** starts embedded Jetty server on random port
5. **REST controllers** are registered as endpoints
6. **Test is ready** to make HTTP requests

### Properties Configuration

**File:** `src/test/resources/it.water.application.properties`

```properties
# Keystore (for JWT)
water.keystore.password=water.
water.keystore.alias=server-cert
water.keystore.file=src/test/resources/certs/server.keystore
water.private.key.password=water.

# JWT Security
water.rest.security.jwt.validate=false
water.rest.security.jwt.duration.millis=3600000

# Test Mode
water.testMode=true

# REST Context Root
water.rest.root.context=/water
```

### Component Discovery

Water Framework automatically discovers components via **Atteo ClassIndex** (compile-time indexing):

- `@FrameworkComponent` → Services, repositories, options
- `@FrameworkRestApi` → REST API interfaces
- `@FrameworkRestController` → REST controller implementations

**No explicit registration needed** — classpath scanning is automatic.

### Admin Impersonation

```java
@BeforeEach
void impersonateAdmin() {
    TestRuntimeUtils.impersonateAdmin(componentRegistry);
}
```

This injects a fake admin user into the security context, allowing tests to bypass:
- `@LoggedIn` authentication checks
- `@AllowPermissions` authorization checks

### Accessing Test Components

```java
@Inject
@Setter
private ComponentRegistry componentRegistry;

@Test
void customSetup() {
    MyService service = componentRegistry.findComponent(MyService.class, null);
    // Custom test setup with service
}
```

---

## 15. Common Mistakes & Fixes

### ❌ Mistake 1: Using Background with `* url serviceBaseUrl`

**WRONG:**
```gherkin
Background:
  * url serviceBaseUrl

Scenario: Test endpoint
  Given path '/roles'
  When method GET
  Then status 200
```

**CORRECT:**
```gherkin
Scenario: Test endpoint
  Given url serviceBaseUrl+'/water/roles'
  When method GET
  Then status 200
```

**Why?** `Background` + `path` doesn't correctly append the Water context root.

---

### ❌ Mistake 2: Forgetting Context Root

**WRONG:**
```gherkin
Given url serviceBaseUrl+'/roles'
```

**CORRECT:**
```gherkin
Given url serviceBaseUrl+'/water/roles'
```

**Why?** Water Framework uses `/water` as default context root.

---

### ❌ Mistake 3: Hardcoding Port

**WRONG:**
```gherkin
Given url 'http://localhost:8080/water/roles'
```

**CORRECT:**
```gherkin
Given url serviceBaseUrl+'/water/roles'
```

**Why?** Test server uses dynamic random port to avoid conflicts.

---

### ❌ Mistake 4: Incorrect Status Code for DELETE

**WRONG:**
```gherkin
When method DELETE
Then status 200
```

**CORRECT:**
```gherkin
When method DELETE
Then status 204  # No Content
```

**Why?** Water Framework DELETE operations return 204 (no body).

---

### ❌ Mistake 5: Not Capturing Variables for Multi-Step Tests

**WRONG:**
```gherkin
# Create entity
Given url serviceBaseUrl+'/water/roles'
And request { "name": "Admin" }
When method POST
Then status 200

# Update entity (how do we know the ID?)
Given url serviceBaseUrl+'/water/roles'
And request { "id": ???, "name": "Updated" }
When method PUT
```

**CORRECT:**
```gherkin
# Create entity
Given url serviceBaseUrl+'/water/roles'
And request { "name": "Admin" }
When method POST
Then status 200
* def entityId = response.id

# Update entity
Given url serviceBaseUrl+'/water/roles'
And request { "id": "#(entityId)", "name": "Updated" }
When method PUT
```

---

### ❌ Mistake 6: Incorrect EntityVersion in Update

**WRONG:**
```gherkin
And request { "id": "#(entityId)", "name": "Updated" }
```

**CORRECT:**
```gherkin
And request { "id": "#(entityId)", "entityVersion": 1, "name": "Updated" }
```

**Why?** Water entities require `entityVersion` for optimistic locking.

---

## 16. Best Practices

### 1. One Scenario Per Workflow

✅ **DO:** Group related CRUD operations in a single scenario:
```gherkin
Scenario: Role CRUD Operations
  # CREATE
  ...
  * def entityId = response.id

  # UPDATE
  ...

  # FIND
  ...

  # DELETE
  ...
```

✅ **Benefit:** Ensures cleanup, easier to debug, captures entity lifecycle.

---

### 2. Always Set Content-Type and Accept Headers

✅ **DO:**
```gherkin
Given header Content-Type = 'application/json'
And header Accept = 'application/json'
```

✅ **Benefit:** Ensures correct serialization/deserialization, avoids 406/415 errors.

---

### 3. Use Fuzzy Matching for Timestamps and IDs

✅ **DO:**
```gherkin
And match response == { id: '#number', entityCreateDate: '#number' }
```

❌ **DON'T:**
```gherkin
And match response == { id: 1, entityCreateDate: 1708619400000 }
```

✅ **Benefit:** Tests are not brittle to dynamic values.

---

### 4. Test Both Success and Error Cases

✅ **DO:**
```gherkin
Scenario: Valid request
  ...
  Then status 200

Scenario: Missing required field
  ...
  Then status 400

Scenario: Invalid ID
  ...
  Then status 404
```

---

### 5. Use Descriptive Scenario Names

✅ **DO:**
```gherkin
Scenario: Publish message to channel
Scenario: Publish without channel should fail
Scenario: Publish with invalid channel name should fail
```

❌ **DON'T:**
```gherkin
Scenario: Test 1
Scenario: Test 2
```

---

### 6. Capture Variables for Dependent Steps

✅ **DO:**
```gherkin
* def entityId = response.id
* def messageId = response.messageId
* def publishedChannel = response.channel
```

✅ **Use in subsequent steps:**
```gherkin
Given url serviceBaseUrl+'/water/roles/'+entityId
```

---

### 7. Comment Test Sections

✅ **DO:**
```gherkin
# ==================== CREATE ====================
...

# ==================== UPDATE ====================
...

# ==================== FIND ====================
...
```

---

### 8. Use Scenario Outline for Validation Tests

✅ **DO:**
```gherkin
Scenario Outline: Field validation
  Given url serviceBaseUrl+'/water/roles'
  And request { "name": "<name>" }
  When method POST
  Then status <status>

  Examples:
    | name      | status |
    | Valid     | 200    |
    |           | 400    |
    | 123!@#    | 400    |
```

---

## 17. Troubleshooting Guide

### Issue 1: 404 Not Found on All Endpoints

**Symptoms:**
```
status code was: 404, expected: 200
url: http://localhost:PORT/water/roles
```

**Causes:**
1. REST API not registered in `RestApiRegistry`
2. Missing `@FrameworkRestController` annotation
3. Incorrect `referredRestApi` parameter
4. Wrong context root configuration

**Fixes:**
- Verify `@FrameworkRestController(referredRestApi = MyEntityRestApi.class)`
- Check `water.rest.root.context=/water` in properties
- Ensure URL pattern: `serviceBaseUrl+'/water/endpoint'`
- Clean and rebuild: `gradle clean build`

---

### Issue 2: 503 Service Unavailable

**Symptoms:**
```
status code was: 503, expected: 200
response: {"error":"Cannot invoke ... because ... is null"}
```

**Cause:** Dependencies not injected into REST controller.

**Fixes:**
- Ensure all `@Inject` fields in service have corresponding `@FrameworkComponent` implementations
- Check that components are in test classpath
- Verify `@Inject` fields have `@Setter` (Lombok) for injection
- Review component registration in test output

---

### Issue 3: 401 Unauthorized (Unexpected)

**Symptoms:**
```
status code was: 401, expected: 200
```

**Causes:**
1. JWT validation enabled but no token provided
2. `impersonateAdmin()` not called in test

**Fixes:**
- Set `water.rest.security.jwt.validate=false` in test properties
- Ensure `@BeforeEach impersonateAdmin()` is present in test runner
- If testing JWT: pass token via `karate.properties['jwtToken']`

---

### Issue 4: 422 Validation Error (Unexpected)

**Symptoms:**
```
status code was: 422, expected: 200
response: {"type":"ValidationException","validationErrors":[...]}
```

**Cause:** Request body missing required fields or has invalid values.

**Fixes:**
- Check entity class `@NotNull`, `@NotBlank` constraints
- Ensure request JSON includes all required fields
- Verify field types (e.g., String vs number)

---

### Issue 5: Tests Pass Individually but Fail When Run Together

**Symptoms:** Single test passes, but full suite has failures.

**Causes:**
1. State pollution between tests (e.g., shared entity IDs)
2. Database not cleaned between tests
3. REST server state persists

**Fixes:**
- Use unique entity names per test
- Ensure each scenario creates its own test data
- Use cleanup logic (DELETE) at end of scenario

---

### Issue 6: Variables Not Recognized

**Symptoms:**
```
javascript evaluation failed: serviceBaseUrl
```

**Cause:** `karate-config.js` not loaded or incorrect variable name.

**Fixes:**
- Ensure `karate-config.js` is in `src/test/resources/`
- Verify function returns object with `serviceBaseUrl` key
- Check test runner passes system properties

---

## 18. Quick Reference

### URL Construction

```
Given url serviceBaseUrl + '/water' + '/endpoint'
```

### Headers

```gherkin
Given header Content-Type = 'application/json'
And header Accept = 'application/json'
And header Authorization = authToken
```

### HTTP Methods

```gherkin
When method GET
When method POST
When method PUT
When method DELETE
```

### Status Codes

| Operation | Status Code |
|-----------|-------------|
| GET success | 200 |
| POST success | 200 |
| PUT success | 200 |
| DELETE success | 204 |
| Bad Request | 400 |
| Unauthorized | 401 |
| Forbidden | 403 |
| Not Found | 404 |
| Method Not Allowed | 405 |
| Conflict | 409 |
| Validation Error | 422 |
| Server Error | 500 |

### Matching

```gherkin
# Exact
Then match response == { id: 1 }

# Type
Then match response == { id: '#number' }

# Fuzzy
Then match response contains { name: 'Admin' }

# Array
Then match response.results == '#array'

# Assertion
Then assert response.length >= 2
```

### Variables

```gherkin
* def entityId = response.id
* def workflowStream = 'test-stream'

# Use in URL
Given url serviceBaseUrl+'/water/roles/'+entityId

# Use in request
And request { id: "#(entityId)", name: "Updated" }
```

### Karate-Config.js Template

```javascript
function fn() {
    let webServerPort = karate.properties['webServerPort'];
    let host = karate.properties['host'];
    let protocol = karate.properties['protocol'];
    let serviceBaseUrl = protocol+"://"+host+":"+webServerPort;
    return {
        "serviceBaseUrl": serviceBaseUrl
    }
}
```

### Test Runner Template

```java
@ExtendWith(WaterTestExtension.class)
public class MyEntityRestApiTest implements Service {

    @Inject @Setter
    private ComponentRegistry componentRegistry;

    @BeforeEach
    void impersonateAdmin() {
        TestRuntimeUtils.impersonateAdmin(componentRegistry);
    }

    @Karate.Test
    Karate restInterfaceTest() {
        return Karate.run("classpath:karate")
                .systemProperty("webServerPort", TestRuntimeInitializer.getInstance().getRestServerPort())
                .systemProperty("host", "localhost")
                .systemProperty("protocol", "http");
    }
}
```

---

## Summary

- **URL Pattern:** `Given url serviceBaseUrl+'/water/endpoint'` (ALWAYS include `/water`)
- **No Background:** Each scenario defines its own URL
- **Headers:** Always set `Content-Type` and `Accept`
- **Entity CRUD:** Single scenario with all operations
- **Variables:** Capture IDs with `* def entityId = response.id`
- **Status Codes:** 200 for GET/POST/PUT, 204 for DELETE
- **Fuzzy Matching:** Use `'#number'`, `'#string'`, `'#array'` for dynamic values
- **Security:** Disable JWT (`water.rest.security.jwt.validate=false`) and use `impersonateAdmin()`
- **Dynamic Port:** Always use `serviceBaseUrl` from `karate-config.js`
- **Context Root:** `/water` is standard (configurable via `water.rest.root.context`)

---

**End of Karate Testing Knowledge Base**
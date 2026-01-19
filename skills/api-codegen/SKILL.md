---
name: api-codegen
description: Generate production-ready API client code from Swagger/OpenAPI documentation by analyzing project environment and asking for clarification interactively.
user-invocable: true
---

# API Code Generator

## Description
Generates production-ready API client code from Swagger/OpenAPI documentation. Analyzes your project environment (language, framework, code style) and creates type-safe API clients, DTOs, configuration, and tests through interactive clarification.

## Core Principle
**ALWAYS ASK, NEVER ASSUME**: This skill operates through continuous dialogue with the user. At every decision point - from technology choices to code structure - it asks for clarification rather than making assumptions. The goal is to generate code that perfectly fits the user's existing project and coding standards.

## Features
- Parse Swagger/OpenAPI documentation (URL or local file)
- Analyze existing project structure and coding standards
- Generate API client code in multiple languages/frameworks
- Create type-safe Request/Response DTOs
- Generate configuration classes
- Create comprehensive test code
- Automatically add required dependencies
- Follow project's existing code style and patterns

## Supported Languages & Frameworks

### Backend
- **Kotlin**: Spring Boot (WebClient, RestTemplate, Retrofit), Ktor Client
- **Java**: Spring Boot (RestTemplate, WebClient), OkHttp, Retrofit
- **Python**: requests, httpx, aiohttp, FastAPI client
- **TypeScript/JavaScript**: axios, fetch, node-fetch

### Frontend
- **TypeScript/React**: axios, fetch with React Query/SWR
- **TypeScript/Vue**: axios, composables
- **TypeScript/Angular**: HttpClient

## Process

### Phase 1: Project Environment Analysis

#### Step 1.1: Detect Project Type
Scan the current directory to identify:
- Programming language (from file extensions, build files)
- Framework (from dependencies, imports)
- Build tool (Gradle, Maven, npm, pip, etc.)

**Ask for confirmation:**
```
Detected project environment:
- Language: Kotlin
- Framework: Spring Boot 3.2.1
- Build Tool: Gradle (Kotlin DSL)
- HTTP Client: WebClient (detected from existing code)

Is this correct? (yes/no)
If no, please specify your environment:
```

#### Step 1.2: Analyze Code Style
Scan existing code to understand:
- Package naming conventions
- Class naming patterns
- DTO structure (data class, POJO, etc.)
- Annotation usage (@RestClient, @Component, etc.)
- Null handling (Optional, nullable, non-null)
- Async patterns (suspend, CompletableFuture, Promise)

**Ask for confirmation:**
```
Analyzed code style from existing files:
- DTOs use data classes
- Nullable types with ? notation
- Suspend functions for async operations
- Jackson annotations for JSON mapping
- Package structure: com.example.client.api

Should I follow these patterns? (yes/no/customize)
```

#### Step 1.3: Ask About Project Preferences
```
Project Configuration:

1. Where should I place the generated API client code?
   Suggested: src/main/kotlin/com/example/client/api
   Custom path: [enter path or press Enter to accept]

2. Package name for generated code?
   Suggested: com.example.client.myapi
   Custom: [enter or press Enter]

3. Base class name for the API client?
   Suggested: MyApiClient
   Custom: [enter or press Enter]
```

### Phase 2: Swagger Analysis & Endpoint Selection

#### Step 2.1: Parse Swagger Document
**Input Swagger URL or file:**
```
User: /api-codegen https://api.example.com/swagger.json

Parsing OpenAPI specification...
✅ Loaded: My Example API v2.1.0
Found: 45 endpoints across 8 tags
```

#### Step 2.2: Ask About Scope
```
What would you like to generate?

1. All endpoints (45 total) - Complete API client
2. Specific tags - Select by functional groups
3. Specific endpoints - Choose individual endpoints
4. Let me guide you - Interactive selection

Enter choice (1-4):
```

**If user selects option 2 (Specific tags):**
```
Available tags:
1. Users (12 endpoints) - User management operations
2. Products (8 endpoints) - Product catalog operations
3. Orders (10 endpoints) - Order processing
4. Payments (5 endpoints) - Payment operations
5. Auth (3 endpoints) - Authentication & authorization
6. Admin (7 endpoints) - Admin operations

Select tags (comma-separated numbers, e.g., 1,3,4):
```

**If user selects option 3 (Specific endpoints):**
```
Available endpoints:

Users:
  1. GET /api/users - List users
  2. POST /api/users - Create user
  3. GET /api/users/{id} - Get user by ID
  ...

Products:
  12. GET /api/products - List products
  13. POST /api/products - Create product
  ...

Select endpoints (comma-separated numbers):
```

### Phase 3: Technology & Architecture Decisions

#### Step 3.1: HTTP Client Selection
```
Which HTTP client library should I use?

Detected in your project: WebClient (org.springframework.web.reactive.function.client)

1. Use existing: WebClient ✓ (Recommended - already in project)
2. RestTemplate (Spring's traditional HTTP client)
3. Retrofit (Type-safe HTTP client)
4. OkHttp (Low-level HTTP client)
5. Other (please specify)

Enter choice (1-5): 1
```

#### Step 3.2: Async/Sync Pattern
```
Should the generated client be synchronous or asynchronous?

Based on your project:
- Found suspend functions in existing code
- Kotlin Coroutines dependency detected

1. Asynchronous with Coroutines (suspend functions) ✓ Recommended
2. Synchronous (blocking calls)
3. Reactive (Mono/Flux)
4. Mix - ask for each endpoint

Enter choice (1-4):
```

#### Step 3.3: DTO Generation Strategy
```
How should I handle Request/Response DTOs?

1. Generate all DTOs from OpenAPI schemas ✓ Recommended
2. Generate only new DTOs (reuse existing if found)
3. Skip DTO generation (use Map/JsonNode)
4. Let me decide for each schema

Enter choice (1-4): 1

Additional DTO options:
- Use data classes? (yes/no): yes
- Generate builder pattern? (yes/no): no
- Include validation annotations? (yes/no): yes
- Generate documentation from OpenAPI descriptions? (yes/no): yes
```

#### Step 3.4: Error Handling
```
How should the client handle API errors?

1. Throw exceptions (follow existing pattern in project)
2. Return Result/Either type (functional approach)
3. Return nullable with logging
4. Custom error handler (I'll specify)

Enter choice (1-4): 1

Exception hierarchy:
1. Use existing exceptions (ApiException detected)
2. Generate new exception classes
3. Use standard HTTP exceptions

Enter choice (1-3):
```

#### Step 3.5: Authentication
```
Authentication method detected in OpenAPI spec: Bearer Token

How should authentication be handled?

1. Inject token via constructor/property
2. Token provider interface (I'll implement)
3. Interceptor/Filter for automatic token injection ✓ Recommended
4. Manual token passing per request

Enter choice (1-4): 3

Where should I get the token from?
1. Configuration properties (application.yml)
2. Token provider bean
3. SecurityContext (Spring Security)
4. Custom (I'll specify)

Enter choice (1-4):
```

### Phase 4: Code Structure & Patterns

#### Step 4.1: Client Architecture
```
How should the API client be structured?

1. Single client class with all methods
   MyApiClient with 45 methods

2. Separate clients by tag ✓ Recommended for large APIs
   - UserApiClient (12 methods)
   - ProductApiClient (8 methods)
   - OrderApiClient (10 methods)
   ...

3. Separate clients by domain logic (I'll specify grouping)

4. Mix of approaches (I'll guide)

Enter choice (1-4): 2
```

#### Step 4.2: Configuration Management
```
How should API configuration be managed?

Detected: application.yml with Spring Boot configuration

1. Generate configuration properties class ✓ Recommended
   @ConfigurationProperties("myapi")

2. Use environment variables directly
3. Hardcode configuration (not recommended)
4. Custom configuration (I'll specify)

Enter choice (1-4): 1

Configuration properties to include:
✓ Base URL (myapi.base-url)
✓ Connection timeout (myapi.timeout.connection)
✓ Read timeout (myapi.timeout.read)
□ Retry configuration
□ Circuit breaker settings
□ Rate limiting

Select additional options (comma-separated numbers, or Enter to skip):
```

#### Step 4.3: Logging & Monitoring
```
Add logging and monitoring?

1. Add logging statements (SLF4J detected) ✓ Recommended
   - Log requests at DEBUG level
   - Log responses at DEBUG level
   - Log errors at ERROR level

2. Add metrics (Micrometer detected)
   - Request counters
   - Duration timers
   - Error rates

3. Add distributed tracing (Sleuth detected)
   - Trace ID propagation
   - Span creation for API calls

4. Skip logging/monitoring

Select all that apply (comma-separated, e.g., 1,2,3):
```

### Phase 5: Advanced Features

#### Step 5.1: Request/Response Interceptors
```
Would you like to add interceptors?

1. Request interceptor (modify outgoing requests)
   Examples: Add headers, log requests, modify body

2. Response interceptor (process responses)
   Examples: Parse errors, log responses, extract headers

3. Both

4. None

Enter choice (1-4): 3

Request interceptor functionality:
□ Add custom headers
□ Log request details
□ Add correlation ID
□ Request validation
☑ Add authentication token

Response interceptor functionality:
□ Parse error responses
□ Log response details
☑ Extract response headers
□ Response validation
□ Retry on failure

Select (comma-separated numbers or Enter to use defaults):
```

#### Step 5.2: Pagination Support
```
API uses pagination. Generate pagination helpers?

Detected pagination pattern: offset/limit

1. Generate pagination utilities
   - PageRequest builder
   - Page wrapper for responses
   - Automatic page fetching

2. Skip pagination helpers

Enter choice (1-2): 1

Pagination style:
1. Offset-based (offset/limit)
2. Page-based (page/size)
3. Cursor-based (cursor/next)

Detected in API: Option 1 (offset/limit)
Use this? (yes/no): yes
```

#### Step 5.3: Response Caching
```
Add response caching?

1. No caching
2. In-memory caching (Caffeine)
3. Distributed caching (Redis)
4. HTTP cache headers only

Enter choice (1-4): 1

(Selected: No caching - can be added later if needed)
```

### Phase 6: Test Code Generation

#### Step 6.1: Test Framework
```
Generate tests?

Detected: JUnit 5, MockK

1. Generate unit tests ✓
   - Mock HTTP client
   - Test request building
   - Test response parsing
   - Test error handling

2. Generate integration tests
   - Use WireMock for API mocking
   - Test full request/response cycle

3. Both unit and integration tests ✓ Recommended

4. Skip tests

Enter choice (1-4): 3
```

#### Step 6.2: Test Coverage
```
Test coverage level?

1. Basic tests (happy path only)
2. Standard tests (happy path + common errors) ✓ Recommended
3. Comprehensive tests (all scenarios + edge cases)

Enter choice (1-3): 2

Test scenarios to include:
✓ Successful responses (200, 201, 204)
✓ Client errors (400, 401, 403, 404)
✓ Server errors (500, 503)
□ Network errors (timeout, connection refused)
□ Malformed responses
□ Rate limiting (429)

Select additional scenarios (comma-separated numbers or Enter to skip):
```

### Phase 7: Code Generation & Review

#### Step 7.1: Generate Code Preview
```
=== CODE GENERATION PREVIEW ===

Files to be created:

API Clients (3 files):
  ✓ src/main/kotlin/com/example/client/myapi/UserApiClient.kt (245 lines)
  ✓ src/main/kotlin/com/example/client/myapi/ProductApiClient.kt (180 lines)
  ✓ src/main/kotlin/com/example/client/myapi/OrderApiClient.kt (210 lines)

DTOs (28 files):
  ✓ src/main/kotlin/com/example/client/myapi/dto/UserDto.kt
  ✓ src/main/kotlin/com/example/client/myapi/dto/CreateUserRequest.kt
  ... (26 more)

Configuration (2 files):
  ✓ src/main/kotlin/com/example/client/myapi/config/MyApiClientConfig.kt
  ✓ src/main/kotlin/com/example/client/myapi/config/MyApiProperties.kt

Interceptors (2 files):
  ✓ src/main/kotlin/com/example/client/myapi/interceptor/AuthInterceptor.kt
  ✓ src/main/kotlin/com/example/client/myapi/interceptor/LoggingInterceptor.kt

Tests (6 files):
  ✓ src/test/kotlin/com/example/client/myapi/UserApiClientTest.kt
  ... (5 more)

Dependencies to add:
  ✓ build.gradle.kts
    - org.springframework.boot:spring-boot-starter-webflux (already exists)
    - com.fasterxml.jackson.module:jackson-module-kotlin (already exists)

Configuration files to update:
  ✓ src/main/resources/application.yml
    Add myapi configuration section

Total: 41 files, ~2,450 lines of code

Proceed with generation? (yes/no/show-sample):
```

#### Step 7.2: Show Sample Code (if requested)
```
User selects "show-sample"

=== SAMPLE: UserApiClient.kt ===

package com.example.client.myapi

import com.example.client.myapi.dto.*
import com.example.client.myapi.config.MyApiProperties
import org.springframework.stereotype.Component
import org.springframework.web.reactive.function.client.WebClient
import org.springframework.web.reactive.function.client.awaitBody

/**
 * API client for User management operations.
 * Generated from OpenAPI specification: My Example API v2.1.0
 */
@Component
class UserApiClient(
    private val webClient: WebClient,
    private val properties: MyApiProperties
) {
    /**
     * List all users with optional filtering.
     *
     * @param page Page number for pagination (default: 0)
     * @param limit Number of items per page (default: 20)
     * @param search Optional search term
     * @return List of users
     * @throws ApiException if the API returns an error
     */
    suspend fun listUsers(
        page: Int = 0,
        limit: Int = 20,
        search: String? = null
    ): UserListResponse {
        return webClient
            .get()
            .uri { builder ->
                builder.path("/api/users")
                    .queryParam("page", page)
                    .queryParam("limit", limit)
                    .apply { search?.let { queryParam("search", it) } }
                    .build()
            }
            .retrieve()
            .awaitBody()
    }

    /**
     * Create a new user.
     *
     * @param request User creation request
     * @return Created user details
     * @throws ApiException if the API returns an error
     */
    suspend fun createUser(request: CreateUserRequest): UserDto {
        return webClient
            .post()
            .uri("/api/users")
            .bodyValue(request)
            .retrieve()
            .awaitBody()
    }

    // ... more methods
}

=== SAMPLE: UserDto.kt ===

package com.example.client.myapi.dto

import com.fasterxml.jackson.annotation.JsonProperty
import java.time.Instant
import javax.validation.constraints.Email
import javax.validation.constraints.NotBlank

/**
 * User data transfer object.
 * Generated from OpenAPI schema: User
 */
data class UserDto(
    @JsonProperty("id")
    val id: String,

    @JsonProperty("name")
    @field:NotBlank
    val name: String,

    @JsonProperty("email")
    @field:Email
    val email: String,

    @JsonProperty("age")
    val age: Int?,

    @JsonProperty("created_at")
    val createdAt: Instant
)

=== SAMPLE: UserApiClientTest.kt ===

package com.example.client.myapi

import com.example.client.myapi.dto.*
import io.mockk.*
import kotlinx.coroutines.runBlocking
import org.junit.jupiter.api.Test
import org.junit.jupiter.api.assertThrows
import org.springframework.web.reactive.function.client.WebClient
import kotlin.test.assertEquals

class UserApiClientTest {

    private val webClient: WebClient = mockk()
    private val properties: MyApiProperties = mockk()
    private val client = UserApiClient(webClient, properties)

    @Test
    fun `listUsers should return user list successfully`() = runBlocking {
        // Given
        val expectedResponse = UserListResponse(
            users = listOf(
                UserDto(id = "1", name = "John", email = "john@example.com", age = 30, createdAt = Instant.now())
            ),
            total = 1
        )

        // Mock WebClient chain
        // ... (detailed mocking code)

        // When
        val result = client.listUsers(page = 0, limit = 20)

        // Then
        assertEquals(expectedResponse, result)
    }

    // ... more tests
}

Proceed with generation? (yes/no):
```

#### Step 7.3: Generate Code
```
User: yes

Generating code...

✓ Created UserApiClient.kt
✓ Created ProductApiClient.kt
✓ Created OrderApiClient.kt
✓ Generated 28 DTO files
✓ Created configuration classes
✓ Created interceptors
✓ Generated 6 test files
✓ Updated build.gradle.kts
✓ Updated application.yml

=== Generation Complete ===

Summary:
- 41 files created
- 2,450 lines of code generated
- 0 errors, 0 warnings

Next steps:
1. Review generated code
2. Run tests: ./gradlew test
3. Update application.yml with your API base URL:

   myapi:
     base-url: https://api.example.com
     timeout:
       connection: 5000
       read: 10000

4. Use the client:

   @Autowired
   lateinit var userApiClient: UserApiClient

   suspend fun example() {
       val users = userApiClient.listUsers(page = 0, limit = 10)
       println("Found ${users.total} users")
   }

Would you like me to:
1. Run the tests now
2. Show usage examples
3. Generate documentation
4. Done

Enter choice (1-4):
```

### Phase 8: Post-Generation Options

#### Option 1: Run Tests
```
User: 1

Running tests...

$ ./gradlew test --tests "*ApiClientTest"

> Task :test

UserApiClientTest
  ✓ listUsers should return user list successfully (245ms)
  ✓ createUser should create user successfully (180ms)
  ✓ listUsers should handle 401 error (95ms)
  ... (12 more tests)

ProductApiClientTest
  ... (8 tests)

OrderApiClientTest
  ... (10 tests)

BUILD SUCCESSFUL in 5s
30 tests completed, 30 passed

All tests passed! ✓
```

#### Option 2: Show Usage Examples
```
User: 2

=== Usage Examples ===

1. Basic Usage (Single Endpoint):

```kotlin
@Service
class UserService(
    private val userApiClient: UserApiClient
) {
    suspend fun fetchUsers(): List<UserDto> {
        val response = userApiClient.listUsers(
            page = 0,
            limit = 50
        )
        return response.users
    }
}
```

2. Error Handling:

```kotlin
suspend fun createUserSafely(request: CreateUserRequest): UserDto? {
    return try {
        userApiClient.createUser(request)
    } catch (e: ApiException) {
        when (e.statusCode) {
            400 -> logger.warn("Invalid request: ${e.message}")
            401 -> logger.error("Unauthorized: Check API token")
            else -> logger.error("API error: ${e.message}", e)
        }
        null
    }
}
```

3. Pagination:

```kotlin
suspend fun fetchAllUsers(): List<UserDto> {
    val allUsers = mutableListOf<UserDto>()
    var page = 0
    var hasMore = true

    while (hasMore) {
        val response = userApiClient.listUsers(page = page, limit = 100)
        allUsers.addAll(response.users)
        hasMore = response.users.size == 100
        page++
    }

    return allUsers
}
```

4. Configuration:

```yaml
# application.yml
myapi:
  base-url: https://api.example.com
  timeout:
    connection: 5000
    read: 10000
  auth:
    token: ${API_TOKEN:}  # Set via environment variable
```
```

#### Option 3: Generate Documentation
```
User: 3

Generating documentation...

✓ Created docs/API_CLIENT_GUIDE.md
✓ Created docs/API_REFERENCE.md
✓ Added KDoc comments to all generated classes

Documentation includes:
- Setup instructions
- Configuration guide
- Usage examples for each endpoint
- Error handling guide
- Testing guide
- Troubleshooting

View documentation: docs/API_CLIENT_GUIDE.md
```

## Advanced Scenarios

### Scenario 1: Update Existing Generated Code
```
User: /api-codegen --update https://api.example.com/swagger.json

Detected existing generated code in com.example.client.myapi

Comparing with new OpenAPI specification...

Changes detected:
- 3 new endpoints added
- 2 endpoints modified
- 1 endpoint removed (deprecated)
- 5 DTOs updated

What would you like to do?
1. Regenerate all (overwrite existing)
2. Update only changed files ✓ Recommended
3. Generate new files only (keep existing)
4. Show detailed diff first

Enter choice (1-4):
```

### Scenario 2: Multiple API Sources
```
User: /api-codegen --multiple

How many APIs do you want to integrate?
Enter number: 3

API 1:
  Name: User Service
  Swagger URL: https://users-api.example.com/swagger.json
  Package: com.example.client.users

API 2:
  Name: Product Service
  Swagger URL: https://products-api.example.com/swagger.json
  Package: com.example.client.products

API 3:
  Name: Order Service
  Swagger URL: https://orders-api.example.com/swagger.json
  Package: com.example.client.orders

Generate shared configuration? (yes/no): yes
Generate facade for cross-service operations? (yes/no): yes
```

### Scenario 3: Custom Templates
```
User: /api-codegen --template custom

Using custom code templates?

1. Use default templates
2. Use custom templates from: .claude/api-codegen-templates/
3. Specify custom template directory

Enter choice (1-3): 2

Detected custom templates:
✓ client.kt.template
✓ dto.kt.template
✓ test.kt.template

Validate templates? (yes/no): yes
Templates validated successfully ✓

Proceed with custom templates? (yes/no):
```

## Configuration File

`~/.claude/api-codegen-config.json`:
```json
{
  "defaults": {
    "language": "kotlin",
    "framework": "spring-boot",
    "httpClient": "webclient",
    "asyncPattern": "coroutines",
    "generateTests": true,
    "testFramework": "junit5"
  },
  "codeStyle": {
    "indentation": "4spaces",
    "lineLength": 120,
    "importOrder": ["java", "javax", "kotlin", "org.springframework", "*"],
    "nullability": "explicit"
  },
  "templates": {
    "directory": "~/.claude/api-codegen-templates",
    "enabled": false
  },
  "projects": {
    "my-service": {
      "rootPackage": "com.example.myservice",
      "clientPackage": "client.api",
      "dtoPackage": "client.dto",
      "baseUrl": "https://api.example.com"
    }
  }
}
```

## Error Handling

### Common Issues

1. **OpenAPI Spec Parsing Errors**
```
❌ Failed to parse OpenAPI specification

Error: Invalid JSON at line 45, column 12

Would you like me to:
1. Show the problematic section
2. Try to fix automatically
3. Skip invalid sections
4. Cancel

Enter choice (1-4):
```

2. **Dependency Conflicts**
```
⚠️ Dependency conflict detected

Required: org.springframework.boot:spring-boot-starter-webflux:3.2.1
Existing: org.springframework.boot:spring-boot-starter-web:3.2.0

These dependencies may conflict (WebFlux vs Web MVC)

Resolution options:
1. Keep existing and use RestTemplate instead
2. Replace with WebFlux (may affect other code)
3. Add both (not recommended - may cause conflicts)
4. Cancel generation

Enter choice (1-4):
```

3. **Code Generation Failures**
```
❌ Failed to generate UserApiClient.kt

Error: File already exists and has local modifications

Options:
1. Overwrite (lose local changes)
2. Merge (attempt to preserve local changes)
3. Skip this file
4. Cancel entire generation

Enter choice (1-4):
```

## Security Considerations

- **API Tokens**: Never hardcode in generated code, always use configuration
- **Sensitive Data**: Mask sensitive fields in logs
- **HTTPS Only**: Warn if base URL is not HTTPS
- **Input Validation**: Generate validation for all DTOs
- **Rate Limiting**: Respect API rate limits

## Tips for Users

1. **Start Small**: Generate code for a few endpoints first, then expand
2. **Review Generated Code**: Always review before committing
3. **Customize Templates**: Create templates for consistent code style
4. **Keep in Sync**: Regenerate when API changes
5. **Test Thoroughly**: Run generated tests and add custom tests
6. **Version Control**: Commit generated code with clear messages

## Example Session

```
User: /api-codegen https://api.example.com/swagger.json
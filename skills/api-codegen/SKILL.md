---
name: api-codegen
description: Generate production-ready API client code from Swagger/OpenAPI documentation by analyzing project environment and asking for clarification interactively.
user-invocable: true
---

# API Code Generator

## Description
Generates production-ready API client code from Swagger/OpenAPI documentation. Analyzes your project environment (language, framework, code style) and creates type-safe API clients, DTOs, configuration, and tests through interactive clarification.

## Core Principle
**항상 질문하고, 절대 추측하지 않기**: 이 스킬은 사용자와의 지속적인 대화를 통해 동작합니다. 기술 선택부터 코드 구조/ㅔ

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

**사용자에게 확인 요청:**
```
감지된 프로젝트 환경:
- 언어: Kotlin
- 프레임워크: Spring Boot 3.2.1
- 빌드 도구: Gradle (Kotlin DSL)
- HTTP 클라이언트: WebClient (기존 코드에서 감지됨)

맞습니까? (yes/no)
아니면 환경을 직접 지정해주세요:
```

#### Step 1.2: Analyze Code Style
Scan existing code to understand:
- Package naming conventions
- Class naming patterns
- DTO structure (data class, POJO, etc.)
- Annotation usage (@RestClient, @Component, etc.)
- Null handling (Optional, nullable, non-null)
- Async patterns (suspend, CompletableFuture, Promise)

**사용자에게 확인 요청:**
```
기존 파일에서 분석한 코드 스타일:
- DTO는 data class 사용
- ? 표기법으로 nullable 타입 처리
- 비동기 작업은 suspend 함수 사용
- JSON 매핑은 Jackson 어노테이션 사용
- 패키지 구조: com.example.client.api

이 패턴들을 따를까요? (yes/no/customize)
```

#### Step 1.3: Ask About Project Preferences
```
프로젝트 설정:

1. 생성된 API 클라이언트 코드를 어디에 배치할까요?
   제안: src/main/kotlin/com/example/client/api
   커스텀 경로: [경로 입력 또는 Enter로 승인]

2. 생성된 코드의 패키지명은?
   제안: com.example.client.myapi
   커스텀: [입력 또는 Enter]

3. API 클라이언트의 기본 클래스명은?
   제안: MyApiClient
   커스텀: [입력 또는 Enter]
```

### Phase 2: Swagger Analysis & Endpoint Selection

#### Step 2.1: Parse Swagger Document
**Swagger URL 또는 파일 입력:**
```
사용자: /api-codegen https://api.example.com/swagger.json

OpenAPI 명세 파싱 중...
✅ 로드 완료: My Example API v2.1.0
발견: 8개 태그에 걸쳐 45개 엔드포인트
```

#### Step 2.2: Ask About Scope
```
무엇을 생성하시겠습니까?

1. 모든 엔드포인트 (총 45개) - 전체 API 클라이언트
2. 특정 태그 - 기능별 그룹으로 선택
3. 특정 엔드포인트 - 개별 엔드포인트 선택
4. 안내받기 - 대화형 선택

선택 (1-4):
```

**사용자가 옵션 2 선택시 (특정 태그):**
```
사용 가능한 태그:
1. Users (12개 엔드포인트) - 사용자 관리 작업
2. Products (8개 엔드포인트) - 상품 카탈로그 작업
3. Orders (10개 엔드포인트) - 주문 처리
4. Payments (5개 엔드포인트) - 결제 작업
5. Auth (3개 엔드포인트) - 인증 및 권한
6. Admin (7개 엔드포인트) - 관리자 작업

태그 선택 (쉼표로 구분, 예: 1,3,4):
```

**사용자가 옵션 3 선택시 (특정 엔드포인트):**
```
사용 가능한 엔드포인트:

Users:
  1. GET /api/users - 사용자 목록 조회
  2. POST /api/users - 사용자 생성
  3. GET /api/users/{id} - ID로 사용자 조회
  ...

Products:
  12. GET /api/products - 상품 목록 조회
  13. POST /api/products - 상품 생성
  ...

엔드포인트 선택 (쉼표로 구분된 번호):
```

### Phase 3: Technology & Architecture Decisions

#### Step 3.1: HTTP Client Selection
```
어떤 HTTP 클라이언트 라이브러리를 사용할까요?

프로젝트에서 감지됨: WebClient (org.springframework.web.reactive.function.client)

1. 기존 사용: WebClient ✓ (권장 - 이미 프로젝트에 존재)
2. RestTemplate (Spring의 전통적인 HTTP 클라이언트)
3. Retrofit (타입 안전한 HTTP 클라이언트)
4. OkHttp (저수준 HTTP 클라이언트)
5. 기타 (직접 지정)

선택 (1-5): 1
```

#### Step 3.2: Async/Sync Pattern
```
생성된 클라이언트를 동기식으로 할까요, 비동기식으로 할까요?

프로젝트 분석 결과:
- 기존 코드에서 suspend 함수 발견
- Kotlin Coroutines 의존성 감지됨

1. Coroutines 비동기 (suspend 함수) ✓ 권장
2. 동기식 (블로킹 호출)
3. 리액티브 (Mono/Flux)
4. 혼합 - 각 엔드포인트마다 질문

선택 (1-4):
```

#### Step 3.3: DTO Generation Strategy
```
Request/Response DTO를 어떻게 처리할까요?

1. OpenAPI 스키마로부터 모든 DTO 생성 ✓ 권장
2. 새로운 DTO만 생성 (발견되면 기존 재사용)
3. DTO 생성 건너뛰기 (Map/JsonNode 사용)
4. 각 스키마마다 결정

선택 (1-4): 1

추가 DTO 옵션:
- data class 사용? (yes/no): yes
- builder 패턴 생성? (yes/no): no
- 검증 어노테이션 포함? (yes/no): yes
- OpenAPI 설명으로 문서 생성? (yes/no): yes
```

#### Step 3.4: Error Handling
```
클라이언트가 API 오류를 어떻게 처리할까요?

1. 예외 던지기 (프로젝트의 기존 패턴 따르기)
2. Result/Either 타입 반환 (함수형 접근)
3. nullable 반환 및 로깅
4. 커스텀 에러 핸들러 (직접 지정)

선택 (1-4): 1

예외 계층구조:
1. 기존 예외 사용 (ApiException 감지됨)
2. 새로운 예외 클래스 생성
3. 표준 HTTP 예외 사용

선택 (1-3):
```

#### Step 3.5: Authentication
```
OpenAPI 명세에서 감지된 인증 방식: Bearer Token

인증을 어떻게 처리할까요?

1. 생성자/프로퍼티로 토큰 주입
2. 토큰 제공자 인터페이스 (직접 구현)
3. 자동 토큰 주입을 위한 인터셉터/필터 ✓ 권장
4. 요청마다 수동으로 토큰 전달

선택 (1-4): 3

토큰을 어디서 가져올까요?
1. 설정 프로퍼티 (application.yml)
2. 토큰 제공자 빈
3. SecurityContext (Spring Security)
4. 커스텀 (직접 지정)

선택 (1-4):
```

### Phase 4: Code Structure & Patterns

#### Step 4.1: Client Architecture
```
API 클라이언트를 어떻게 구조화할까요?

1. 모든 메서드를 포함하는 단일 클라이언트 클래스
   MyApiClient에 45개 메서드

2. 태그별로 클라이언트 분리 ✓ 대규모 API에 권장
   - UserApiClient (12개 메서드)
   - ProductApiClient (8개 메서드)
   - OrderApiClient (10개 메서드)
   ...

3. 도메인 로직별로 클라이언트 분리 (그룹화 직접 지정)

4. 접근 방식 혼합 (안내받기)

선택 (1-4): 2
```

#### Step 4.2: Configuration Management
```
API 설정을 어떻게 관리할까요?

감지됨: Spring Boot 설정이 있는 application.yml

1. 설정 프로퍼티 클래스 생성 ✓ 권장
   @ConfigurationProperties("myapi")

2. 환경 변수 직접 사용
3. 설정 하드코딩 (권장하지 않음)
4. 커스텀 설정 (직접 지정)

선택 (1-4): 1

포함할 설정 프로퍼티:
✓ Base URL (myapi.base-url)
✓ 연결 타임아웃 (myapi.timeout.connection)
✓ 읽기 타임아웃 (myapi.timeout.read)
□ 재시도 설정
□ 서킷 브레이커 설정
□ Rate limiting

추가 옵션 선택 (쉼표로 구분된 번호, 또는 Enter로 건너뛰기):
```

#### Step 4.3: Logging & Monitoring
```
로깅과 모니터링을 추가할까요?

1. 로깅 문구 추가 (SLF4J 감지됨) ✓ 권장
   - DEBUG 레벨에서 요청 로그
   - DEBUG 레벨에서 응답 로그
   - ERROR 레벨에서 오류 로그

2. 메트릭 추가 (Micrometer 감지됨)
   - 요청 카운터
   - 지속시간 타이머
   - 오류율

3. 분산 추적 추가 (Sleuth 감지됨)
   - 추적 ID 전파
   - API 호출용 Span 생성

4. 로깅/모니터링 건너뛰기

적용할 항목 모두 선택 (쉼표로 구분, 예: 1,2,3):
```

### Phase 5: Advanced Features

#### Step 5.1: Request/Response Interceptors
```
인터셉터를 추가하시겠습니까?

1. 요청 인터셉터 (나가는 요청 수정)
   예시: 헤더 추가, 요청 로그, 본문 수정

2. 응답 인터셉터 (응답 처리)
   예시: 오류 파싱, 응답 로그, 헤더 추출

3. 둘 다

4. 없음

선택 (1-4): 3

요청 인터셉터 기능:
□ 커스텀 헤더 추가
□ 요청 상세 로그
□ Correlation ID 추가
□ 요청 검증
☑ 인증 토큰 추가

응답 인터셉터 기능:
□ 오류 응답 파싱
□ 응답 상세 로그
☑ 응답 헤더 추출
□ 응답 검증
□ 실패 시 재시도

선택 (쉼표로 구분된 번호 또는 Enter로 기본값 사용):
```

#### Step 5.2: Pagination Support
```
API가 페이지네이션을 사용합니다. 페이지네이션 헬퍼를 생성할까요?

감지된 페이지네이션 패턴: offset/limit

1. 페이지네이션 유틸리티 생성
   - PageRequest 빌더
   - 응답용 Page 래퍼
   - 자동 페이지 가져오기

2. 페이지네이션 헬퍼 건너뛰기

선택 (1-2): 1

페이지네이션 스타일:
1. Offset 기반 (offset/limit)
2. Page 기반 (page/size)
3. Cursor 기반 (cursor/next)

API에서 감지됨: 옵션 1 (offset/limit)
사용할까요? (yes/no): yes
```

#### Step 5.3: Response Caching
```
응답 캐싱을 추가할까요?

1. 캐싱 없음
2. 인메모리 캐싱 (Caffeine)
3. 분산 캐싱 (Redis)
4. HTTP 캐시 헤더만 사용

선택 (1-4): 1

(선택됨: 캐싱 없음 - 필요시 나중에 추가 가능)
```

### Phase 6: Test Code Generation

#### Step 6.1: Test Framework
```
테스트를 생성할까요?

감지됨: JUnit 5, MockK

1. 단위 테스트 생성 ✓
   - HTTP 클라이언트 모킹
   - 요청 빌딩 테스트
   - 응답 파싱 테스트
   - 오류 처리 테스트

2. 통합 테스트 생성
   - API 모킹에 WireMock 사용
   - 전체 요청/응답 사이클 테스트

3. 단위 및 통합 테스트 모두 ✓ 권장

4. 테스트 건너뛰기

선택 (1-4): 3
```

#### Step 6.2: Test Coverage
```
테스트 커버리지 레벨은?

1. 기본 테스트 (정상 경로만)
2. 표준 테스트 (정상 경로 + 일반적인 오류) ✓ 권장
3. 포괄적 테스트 (모든 시나리오 + 엣지 케이스)

선택 (1-3): 2

포함할 테스트 시나리오:
✓ 성공 응답 (200, 201, 204)
✓ 클라이언트 오류 (400, 401, 403, 404)
✓ 서버 오류 (500, 503)
□ 네트워크 오류 (타임아웃, 연결 거부)
□ 잘못된 형식의 응답
□ Rate limiting (429)

추가 시나리오 선택 (쉼표로 구분된 번호 또는 Enter로 건너뛰기):
```

### Phase 7: Code Generation & Review

#### Step 7.1: Generate Code Preview
```
=== 코드 생성 미리보기 ===

생성될 파일:

API 클라이언트 (3개 파일):
  ✓ src/main/kotlin/com/example/client/myapi/UserApiClient.kt (245줄)
  ✓ src/main/kotlin/com/example/client/myapi/ProductApiClient.kt (180줄)
  ✓ src/main/kotlin/com/example/client/myapi/OrderApiClient.kt (210줄)

DTO (28개 파일):
  ✓ src/main/kotlin/com/example/client/myapi/dto/UserDto.kt
  ✓ src/main/kotlin/com/example/client/myapi/dto/CreateUserRequest.kt
  ... (26개 더)

설정 (2개 파일):
  ✓ src/main/kotlin/com/example/client/myapi/config/MyApiClientConfig.kt
  ✓ src/main/kotlin/com/example/client/myapi/config/MyApiProperties.kt

인터셉터 (2개 파일):
  ✓ src/main/kotlin/com/example/client/myapi/interceptor/AuthInterceptor.kt
  ✓ src/main/kotlin/com/example/client/myapi/interceptor/LoggingInterceptor.kt

테스트 (6개 파일):
  ✓ src/test/kotlin/com/example/client/myapi/UserApiClientTest.kt
  ... (5개 더)

추가할 의존성:
  ✓ build.gradle.kts
    - org.springframework.boot:spring-boot-starter-webflux (이미 존재)
    - com.fasterxml.jackson.module:jackson-module-kotlin (이미 존재)

업데이트할 설정 파일:
  ✓ src/main/resources/application.yml
    myapi 설정 섹션 추가

총: 41개 파일, 약 2,450줄의 코드

생성을 진행할까요? (yes/no/show-sample):
```

#### Step 7.2: Show Sample Code (if requested)
```
사용자가 "show-sample" 선택

=== 샘플: UserApiClient.kt ===

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

=== 샘플: UserDto.kt ===

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

=== 샘플: UserApiClientTest.kt ===

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

생성을 진행할까요? (yes/no):
```

#### Step 7.3: Generate Code
```
사용자: yes

코드 생성 중...

✓ UserApiClient.kt 생성됨
✓ ProductApiClient.kt 생성됨
✓ OrderApiClient.kt 생성됨
✓ 28개 DTO 파일 생성됨
✓ 설정 클래스 생성됨
✓ 인터셉터 생성됨
✓ 6개 테스트 파일 생성됨
✓ build.gradle.kts 업데이트됨
✓ application.yml 업데이트됨

=== 생성 완료 ===

요약:
- 41개 파일 생성됨
- 2,450줄의 코드 생성됨
- 0개 오류, 0개 경고

다음 단계:
1. 생성된 코드 검토
2. 테스트 실행: ./gradlew test
3. application.yml을 API base URL로 업데이트:

   myapi:
     base-url: https://api.example.com
     timeout:
       connection: 5000
       read: 10000

4. 클라이언트 사용:

   @Autowired
   lateinit var userApiClient: UserApiClient

   suspend fun example() {
       val users = userApiClient.listUsers(page = 0, limit = 10)
       println("Found ${users.total} users")
   }

무엇을 도와드릴까요?
1. 지금 테스트 실행
2. 사용 예제 보기
3. 문서 생성
4. 완료

선택 (1-4):
```

### Phase 8: Post-Generation Options

#### Option 1: Run Tests
```
사용자: 1

테스트 실행 중...

$ ./gradlew test --tests "*ApiClientTest"

> Task :test

UserApiClientTest
  ✓ listUsers should return user list successfully (245ms)
  ✓ createUser should create user successfully (180ms)
  ✓ listUsers should handle 401 error (95ms)
  ... (12개 더)

ProductApiClientTest
  ... (8개 테스트)

OrderApiClientTest
  ... (10개 테스트)

BUILD SUCCESSFUL in 5s
30개 테스트 완료, 30개 통과

모든 테스트 통과! ✓
```

#### Option 2: Show Usage Examples
```
사용자: 2

=== 사용 예제 ===

1. 기본 사용법 (단일 엔드포인트):

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

2. 오류 처리:

```kotlin
suspend fun createUserSafely(request: CreateUserRequest): UserDto? {
    return try {
        userApiClient.createUser(request)
    } catch (e: ApiException) {
        when (e.statusCode) {
            400 -> logger.warn("잘못된 요청: ${e.message}")
            401 -> logger.error("인증되지 않음: API 토큰 확인")
            else -> logger.error("API 오류: ${e.message}", e)
        }
        null
    }
}
```

3. 페이지네이션:

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

4. 설정:

```yaml
# application.yml
myapi:
  base-url: https://api.example.com
  timeout:
    connection: 5000
    read: 10000
  auth:
    token: ${API_TOKEN:}  # 환경 변수로 설정
```
```

#### Option 3: Generate Documentation
```
사용자: 3

문서 생성 중...

✓ docs/API_CLIENT_GUIDE.md 생성됨
✓ docs/API_REFERENCE.md 생성됨
✓ 모든 생성된 클래스에 KDoc 주석 추가됨

문서 포함 내용:
- 설정 지침
- 설정 가이드
- 각 엔드포인트 사용 예제
- 오류 처리 가이드
- 테스팅 가이드
- 문제 해결

문서 보기: docs/API_CLIENT_GUIDE.md
```

## Advanced Scenarios

### Scenario 1: Update Existing Generated Code
```
사용자: /api-codegen --update https://api.example.com/swagger.json

com.example.client.myapi에서 기존 생성 코드 감지됨

새 OpenAPI 명세와 비교 중...

감지된 변경사항:
- 3개 새 엔드포인트 추가됨
- 2개 엔드포인트 수정됨
- 1개 엔드포인트 제거됨 (deprecated)
- 5개 DTO 업데이트됨

어떻게 하시겠습니까?
1. 모두 재생성 (기존 덮어쓰기)
2. 변경된 파일만 업데이트 ✓ 권장
3. 새 파일만 생성 (기존 유지)
4. 먼저 상세한 diff 보기

선택 (1-4):
```

### Scenario 2: Multiple API Sources
```
사용자: /api-codegen --multiple

통합하려는 API가 몇 개입니까?
숫자 입력: 3

API 1:
  이름: User Service
  Swagger URL: https://users-api.example.com/swagger.json
  패키지: com.example.client.users

API 2:
  이름: Product Service
  Swagger URL: https://products-api.example.com/swagger.json
  패키지: com.example.client.products

API 3:
  이름: Order Service
  Swagger URL: https://orders-api.example.com/swagger.json
  패키지: com.example.client.orders

공유 설정 생성? (yes/no): yes
크로스 서비스 작업을 위한 facade 생성? (yes/no): yes
```

### Scenario 3: Custom Templates
```
사용자: /api-codegen --template custom

커스텀 코드 템플릿을 사용하시겠습니까?

1. 기본 템플릿 사용
2. 다음 경로의 커스텀 템플릿 사용: .claude/api-codegen-templates/
3. 커스텀 템플릿 디렉토리 지정

선택 (1-3): 2

감지된 커스텀 템플릿:
✓ client.kt.template
✓ dto.kt.template
✓ test.kt.template

템플릿을 검증할까요? (yes/no): yes
템플릿 검증 성공 ✓

커스텀 템플릿으로 진행할까요? (yes/no):
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
❌ OpenAPI 명세 파싱 실패

오류: 45번째 줄, 12번째 열에서 잘못된 JSON

어떻게 하시겠습니까?
1. 문제가 있는 섹션 보기
2. 자동으로 수정 시도
3. 잘못된 섹션 건너뛰기
4. 취소

선택 (1-4):
```

2. **Dependency Conflicts**
```
⚠️ 의존성 충돌 감지됨

필요: org.springframework.boot:spring-boot-starter-webflux:3.2.1
기존: org.springframework.boot:spring-boot-starter-web:3.2.0

이 의존성들은 충돌할 수 있습니다 (WebFlux vs Web MVC)

해결 옵션:
1. 기존 유지 및 대신 RestTemplate 사용
2. WebFlux로 교체 (다른 코드에 영향 가능)
3. 둘 다 추가 (권장하지 않음 - 충돌 가능)
4. 생성 취소

선택 (1-4):
```

3. **Code Generation Failures**
```
❌ UserApiClient.kt 생성 실패

오류: 파일이 이미 존재하며 로컬 수정이 있습니다

옵션:
1. 덮어쓰기 (로컬 변경사항 손실)
2. 병합 (로컬 변경사항 보존 시도)
3. 이 파일 건너뛰기
4. 전체 생성 취소

선택 (1-4):
```

## Security Considerations

- **API Tokens**: 생성된 코드에 절대 하드코딩하지 말고, 항상 설정 사용
- **민감 데이터**: 로그에서 민감한 필드 마스킹
- **HTTPS Only**: base URL이 HTTPS가 아니면 경고
- **Input Validation**: 모든 DTO에 대한 검증 생성
- **Rate Limiting**: API rate limit 준수

## Tips for Users

1. **작게 시작하기**: 먼저 몇 개의 엔드포인트만 생성한 후 확장
2. **생성된 코드 검토**: 커밋하기 전에 항상 검토
3. **템플릿 커스터마이징**: 일관된 코드 스타일을 위한 템플릿 생성
4. **동기화 유지**: API 변경 시 재생성
5. **철저한 테스트**: 생성된 테스트 실행 및 커스텀 테스트 추가
6. **버전 관리**: 명확한 메시지와 함께 생성된 코드 커밋

## Example Session

```
사용자: /api-codegen https://api.example.com/swagger.json

https://api.example.com/swagger.json의 OpenAPI 명세로부터 API 클라이언트 코드를 생성하도록 돕겠습니다.

프로젝트 환경을 분석하고 OpenAPI 명세를 가져오겠습니다.
```

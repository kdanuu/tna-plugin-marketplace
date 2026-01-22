# Kotlin/Spring 프로젝트 구조 가이드

## 표준 디렉토리 레이아웃

```
src/main/kotlin/com/company/project/
├── [feature]/                     # 기능별 패키지
│   ├── adapter/
│   │   ├── in/web/               # Inbound Adapters (Controllers)
│   │   │   └── [Feature]Controller.kt
│   │   └── out/client/            # Outbound Adapters (API Clients)
│   │       ├── [Feature]ApiClient.kt
│   │       └── config/
│   │           └── [Feature]ClientConfig.kt
│   │
│   ├── application/               # 비즈니스 로직
│   │   ├── port/
│   │   │   ├── in/               # Use Case 인터페이스
│   │   │   │   └── [Feature]UseCase.kt
│   │   │   └── out/              # Port 인터페이스
│   │   │       └── [Feature]Port.kt
│   │   └── service/
│   │       └── [Feature]Service.kt
│   │
│   ├── domain/                    # 도메인 모델
│   │   ├── model/
│   │   │   ├── [Entity].kt
│   │   │   └── [ValueObject].kt
│   │   └── enums/
│   │       └── [Enum].kt
│   │
│   └── dto/                       # Data Transfer Objects
│       ├── request/
│       │   └── [Feature]Request.kt
│       └── response/
│           └── [Feature]Response.kt
│
├── common/                        # 공통 모듈
│   ├── config/
│   │   └── WebClientConfig.kt
│   ├── exception/
│   │   ├── [Custom]Exception.kt
│   │   └── GlobalExceptionHandler.kt
│   └── util/
│       └── [Util].kt
│
└── infrastructure/                # 인프라 계층
    └── cache/
        └── TokenCacheAdapter.kt
```

## 파일 배치 규칙

### Controller (adapter/in/web/)
**위치**: `src/main/kotlin/[package]/[feature]/adapter/in/web/`

**예시**:
- `PaymentController.kt` → `com.company.project.payment/adapter/in/web/PaymentController.kt`
- `UserController.kt` → `com.company.project.user/adapter/in/web/UserController.kt`

### Service (application/service/)
**위치**: `src/main/kotlin/[package]/[feature]/application/service/`

**예시**:
- `PaymentService.kt` → `com.company.project.payment/application/service/PaymentService.kt`

### API Client (adapter/out/client/)
**위치**: `src/main/kotlin/[package]/[feature]/adapter/out/client/`

**예시**:
- `PaymentApiClient.kt` → `com.company.project.payment/adapter/out/client/PaymentApiClient.kt`

### Domain Model (domain/model/)
**위치**: `src/main/kotlin/[package]/[feature]/domain/model/`

**예시**:
- `Payment.kt` → `com.company.project.payment/domain/model/Payment.kt`
- `User.kt` → `com.company.project.user/domain/model/User.kt`

### DTO (dto/request 또는 dto/response/)
**위치**: `src/main/kotlin/[package]/[feature]/dto/request/` 또는 `response/`

**예시**:
- `CreatePaymentRequest.kt` → `com.company.project.payment/dto/request/CreatePaymentRequest.kt`
- `PaymentResponse.kt` → `com.company.project.payment/dto/response/PaymentResponse.kt`

### Exception (common/exception/)
**위치**: `src/main/kotlin/[package]/common/exception/`

**예시**:
- `ApiException.kt` → `com.company.project.common/exception/ApiException.kt`
- `GlobalExceptionHandler.kt` → `com.company.project.common/exception/GlobalExceptionHandler.kt`

### Configuration (common/config/ 또는 feature/adapter/out/client/config/)
**위치**:
- 공통 설정: `src/main/kotlin/[package]/common/config/`
- 기능별 설정: `src/main/kotlin/[package]/[feature]/adapter/out/client/config/`

**예시**:
- `WebClientConfig.kt` → `common/config/WebClientConfig.kt`
- `PaymentClientConfig.kt` → `payment/adapter/out/client/config/PaymentClientConfig.kt`

## 테스트 디렉토리 구조

```
src/test/kotlin/com/company/project/
├── [feature]/
│   ├── adapter/
│   │   ├── in/web/
│   │   │   └── [Feature]ControllerTest.kt
│   │   └── out/client/
│   │       └── [Feature]ApiClientTest.kt
│   ├── application/service/
│   │   └── [Feature]ServiceTest.kt
│   └── domain/model/
│       └── [Entity]Test.kt
└── integration/
    └── [Feature]IntegrationTest.kt
```

## 명명 규칙

| 종류 | 패턴 | 예시 |
|-----|------|------|
| Controller | `{기능}Controller.kt` | `PaymentController.kt` |
| Service | `{기능}Service.kt` | `PaymentService.kt` |
| API Client | `{대상}ApiClient.kt` | `PaymentApiClient.kt` |
| Domain Model | `{엔티티명}.kt` | `Payment.kt`, `User.kt` |
| Request DTO | `{동작}{엔티티}Request.kt` | `CreatePaymentRequest.kt` |
| Response DTO | `{엔티티}Response.kt` | `PaymentResponse.kt` |
| Exception | `{원인}Exception.kt` | `AuthenticationException.kt` |
| Enum | `{개념}Type.kt` 또는 `{개념}Status.kt` | `PaymentStatus.kt` |

## 환경 설정 파일

### application.yml
**위치**: `src/main/resources/application.yml`

```yaml
[feature]:
  api:
    base-url: ${FEATURE_API_URL:https://api.example.com}
    timeout:
      connect: 5000
      read: 30000
```

### application.properties
**위치**: `src/main/resources/application.properties`

```properties
feature.api.base-url=${FEATURE_API_URL:https://api.example.com}
feature.api.timeout.connect=5000
feature.api.timeout.read=30000
```

## 의존성 방향

```
[Adapter] ─────> [Application] ─────> [Domain]
    ↑                                      ↑
    └─────────> [DTO] ──────────────────┘
```

- 의존성은 항상 안쪽(Domain)으로
- Domain은 외부 의존성 없음
- Application은 Port를 통해서만 외부 접근

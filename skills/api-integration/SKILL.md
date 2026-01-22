---
name: api-integration
description: Swagger/OpenAPI 명세서를 분석하여 프로덕션 레벨의 API 연동 코드를 자동 생성합니다. 상세한 실행 계획 수립 → 백엔드 API 클라이언트 생성 → 프론트엔드 API 훅 생성까지 전 과정을 자동화합니다. Swagger 파일 배치 → `/api-integration` 실행으로 즉시 사용 가능. Kotlin/Spring, React/TypeScript, Next.js 지원.
---

# API Integration Generator

Swagger/OpenAPI 명세서에서 프로덕션 레벨의 API 연동 코드를 자동으로 생성하는 플러그인입니다.

## 개요

이 플러그인은 Swagger 문서를 분석하여:
1. **상세한 실행 계획 생성** (7개 문서)
2. **백엔드 API 클라이언트 생성** (Kotlin/Spring)
3. **프론트엔드 API 클라이언트 생성** (React/Next.js)

모든 과정이 **서브에이전트**를 통해 자동화됩니다.

## 사용 방법

### 1. Swagger 파일 배치

프로젝트 루트에 `swagger/` 디렉토리를 만들고 Swagger 파일을 배치:

```
project-root/
├── swagger/
│   └── payment-api.js     # 또는 .json, .yaml
├── src/
└── ...
```

### 2. 플러그인 실행

```bash
/api-integration
```

### 3. 생성되는 것들

#### 📋 실행 계획 (`.claude-plans/[날짜]-[작업명]/`)
- `README.md`: 전체 개요
- `01-api-analysis.md`: API 명세서 분석
- `02-package-structure.md`: 패키지 구조
- `03-implementation-details.md`: 구현 상세 (코드 예시 포함)
- `04-implementation-phases.md`: Phase별 작업 계획
- `05-testing-strategy.md`: 테스트 전략
- `06-risks-and-mitigations.md`: 위험 및 완화책

#### 🔨 백엔드 코드 (Kotlin/Spring)
- API 클라이언트 (`[Feature]ApiClient.kt`)
- DTO 모델 (`[Feature]Models.kt`)
- 도메인 모델
- 테스트 코드

#### ⚛️ 프론트엔드 코드 (React/TypeScript)
- API 클라이언트 (`src/api/[feature]Api.ts`)
- 타입 정의 (`src/types/[feature].ts`)
- React Query Hooks (`src/hooks/use[Feature].ts`)
- 테스트 코드

## 워크플로우

```
1. Swagger 파일 배치 (swagger/payment-api.js)
     ↓
2. /api-integration 실행
     ↓
3. Planner 에이전트 실행
   - Swagger 분석
   - 작업명 추출: "payment"
   - 실행 계획 생성: .claude-plans/2026-01-23-payment/
     ↓
4. Backend 구현 에이전트 실행
   - PaymentApiClient.kt 생성
   - PaymentModels.kt 생성
   - 테스트 코드 생성
     ↓
5. Frontend 구현 에이전트 실행
   - src/api/paymentApi.ts 생성
   - src/types/payment.ts 생성
   - src/hooks/usePayment.ts 생성
     ↓
6. ✅ 완료!
```

## 실행 시 동작

### Step 1: Swagger 파일 확인

플러그인은 먼저 `swagger/` 디렉토리를 확인합니다:

```
✅ swagger/ 디렉토리 발견
✅ Swagger 파일 발견: payment-gateway-api.js
🔍 핵심 작업명 추출: "payment-gateway"
```

### Step 2: Planner 에이전트 (자동 실행)

Swagger를 분석하여 상세한 실행 계획을 생성합니다:

**생성되는 문서**:
- `.claude-plans/2026-01-23-payment-gateway/README.md`
- `.claude-plans/2026-01-23-payment-gateway/01-api-analysis.md`
- `.claude-plans/2026-01-23-payment-gateway/02-package-structure.md`
- `.claude-plans/2026-01-23-payment-gateway/03-implementation-details.md`
- `.claude-plans/2026-01-23-payment-gateway/04-implementation-phases.md`
- `.claude-plans/2026-01-23-payment-gateway/05-testing-strategy.md`
- `.claude-plans/2026-01-23-payment-gateway/06-risks-and-mitigations.md`

**계획 내용**:
- API 엔드포인트 목록 (표 형식)
- 데이터 모델 의존성 다이어그램
- Phase별 작업 목록 (순서, 의존성, 예상 시간)
- 실제 구현 가능한 코드 예시
- 테스트 전략 및 위험 완화책

### Step 3: Backend 구현 에이전트 (자동 실행)

계획을 바탕으로 백엔드 코드를 생성합니다:

**생성되는 파일**:
```kotlin
// src/main/kotlin/.../client/PaymentApiClient.kt
@Component
class PaymentApiClient(
    private val restTemplate: RestTemplate,
    @Value("\${payment.api.base-url}") private val baseUrl: String
) {
    fun createPayment(request: CreatePaymentRequest): Result<PaymentResponse> {
        // 실제 구현 코드
    }
}

// src/main/kotlin/.../dto/PaymentModels.kt
data class CreatePaymentRequest(
    val amount: Long,
    val currency: String,
    val customerId: String
)

// src/test/kotlin/.../PaymentApiClientTest.kt
class PaymentApiClientTest {
    // MockWebServer 기반 테스트
}
```

### Step 4: Frontend 구현 에이전트 (자동 실행)

프론트엔드 코드를 생성합니다:

**생성되는 파일**:
```typescript
// src/api/paymentApi.ts
export const paymentApi = {
  async createPayment(request: CreatePaymentRequest): Promise<Payment> {
    // 실제 구현 코드
  }
}

// src/types/payment.ts
export interface Payment {
  id: string
  amount: number
  currency: string
  status: PaymentStatus
}

// src/hooks/usePayment.ts
export function usePayment(paymentId: string) {
  return useQuery({
    queryKey: ['payment', paymentId],
    queryFn: () => paymentApi.getPayment(paymentId)
  })
}
```

## 지원하는 환경

### 백엔드
- ☕ **Kotlin/Spring**: RestTemplate, WebClient
- ☕ **Java/Spring**: RestTemplate

### 프론트엔드
- ⚛️ **React/TypeScript**: Axios + React Query
- ⚡ **Next.js**: App Router, Server/Client Components

## 주요 기능

### 1. 지능형 Swagger 분석
- 엔드포인트 자동 감지
- 데이터 모델 의존성 파악
- Enum 타입 추출
- 인증 방식 감지 (Bearer Token, API Key 등)

### 2. 프로젝트 환경 자동 감지
- 백엔드: Gradle/Maven, 패키지 구조 파악
- 프론트엔드: package.json, 디렉토리 구조 파악
- 기존 코드 스타일 학습

### 3. 코드 품질 보장
- **불변성**: 모든 프로퍼티 `val` 사용
- **타입 안전성**: 명시적 타입 정의
- **에러 핸들링**: Result 타입 또는 예외 처리
- **테스트**: MockWebServer, MSW 기반 테스트 자동 생성
- **로깅**: 적절한 로그 레벨

### 4. 프로덕션 레벨 코드
- Circuit Breaker, Retry 로직 고려
- 토큰 캐싱 전략
- 환경 변수로 URL 관리
- 페이지네이션 지원

## 생성된 코드 위치

### Kotlin/Spring
```
src/main/kotlin/[package]/
├── [feature]/
│   ├── adapter/out/client/
│   │   └── [Feature]ApiClient.kt
│   ├── domain/model/
│   │   └── [Model].kt
│   └── dto/
│       ├── request/[Request].kt
│       └── response/[Response].kt
└── src/test/kotlin/.../
    └── [Feature]ApiClientTest.kt
```

### React/TypeScript
```
src/
├── api/
│   └── [feature]Api.ts
├── types/
│   └── [feature].ts
├── hooks/
│   ├── use[Feature].ts
│   └── useCreate[Feature].ts
└── api/__tests__/
    └── [feature]Api.test.ts
```

## 코딩 스타일 준수

이 플러그인은 사용자의 코딩 스타일 가이드를 자동으로 준수합니다:

- **불변성 원칙** (val, 새 객체 생성)
- **Null 안전성** (Nullable 최소화)
- **함수형 프로그래밍** (순수 함수, 부수 효과 최소화)
- **적절한 패키지 구조**
- **테스트 코드 작성**

## 사용 예시

### 입력: Swagger 파일

```javascript
// swagger/payment-gateway-api.js
export const spec = {
  info: {
    title: "Payment Gateway API",
    version: "1.0.0"
  },
  paths: {
    "/api/v1/payments": {
      post: {
        operationId: "createPayment",
        requestBody: { /* ... */ },
        responses: { /* ... */ }
      }
    }
  }
}
```

### 출력: 실행 계획 + 코드

```
✅ 계획 수립 완료!

📂 실행 계획: .claude-plans/2026-01-23-payment-gateway/
  - README.md (전체 개요)
  - 01-api-analysis.md (12개 엔드포인트 분석)
  - 02-package-structure.md (패키지 구조)
  - 03-implementation-details.md (실제 코드 예시)
  - 04-implementation-phases.md (6개 Phase, 45개 작업)
  - 05-testing-strategy.md (단위/통합/E2E 테스트)
  - 06-risks-and-mitigations.md (위험 및 완화책)

✅ 백엔드 코드 생성 완료!
  - PaymentApiClient.kt
  - PaymentModels.kt
  - PaymentApiClientTest.kt

✅ 프론트엔드 코드 생성 완료!
  - src/api/paymentApi.ts
  - src/types/payment.ts
  - src/hooks/usePayment.ts
  - src/api/__tests__/paymentApi.test.ts
```

## 서브에이전트 구조

이 플러그인은 3개의 전문 서브에이전트로 구성됩니다:

1. **Planner**: Swagger 분석 → 실행 계획 생성
2. **Backend Implementer**: 백엔드 API 클라이언트 생성
3. **Frontend Implementer**: 프론트엔드 API 클라이언트 생성

각 에이전트는 독립적으로 실행되며, 사용자 개입 없이 자동으로 작업을 완료합니다.

## 참조 가이드

- `references/project-structure-kotlin-spring.md`: Kotlin/Spring 프로젝트 구조
- `references/project-structure-react-typescript.md`: React 프로젝트 구조
- `references/project-structure-nextjs.md`: Next.js 프로젝트 구조

## 문제 해결

### Swagger 파일을 찾을 수 없습니다

```
❌ Swagger 파일을 찾을 수 없습니다.

📁 확인할 위치: swagger/
💡 해결 방법:
   1. swagger/ 디렉토리 생성
   2. Swagger 파일 (.js, .json, .yaml) 배치
   3. 다시 실행
```

### 프로젝트 환경을 감지할 수 없습니다

플러그인이 백엔드 또는 프론트엔드 환경을 감지하지 못하면, 수동으로 환경을 지정할 수 있습니다.

## 기여 및 피드백

이슈 또는 개선 제안은 GitHub Issues를 통해 제출해주세요.

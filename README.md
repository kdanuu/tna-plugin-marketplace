# Claude Code 스킬 컬렉션

[Claude Code](https://github.com/anthropics/claude-code)를 위한 프로덕션 레벨의 스킬 모음으로, AI 기반 자동화를 통해 개발 워크플로우를 향상시킵니다.

## 📦 제공되는 스킬

이 저장소는 **모노레포**로 구성되어 있으며, 각각의 특정 개발 과제를 해결하도록 설계된 여러 Claude Code 스킬을 포함합니다:

### 🔄 [change-log](skills/change-log/)
**Jira & Confluence 자동 변경로그 생성기**

feature 브랜치를 develop으로 머지할 때마다 수동으로 변경로그를 작성하느라 시간을 낭비하시나요? 이 스킬은 git 커밋 히스토리, Jira 티켓 정보, 코드 변경사항을 AI가 자동으로 분석하여 팀원 누구나 이해할 수 있는 상세한 변경로그를 생성하고 Confluence에 자동으로 게시합니다.

#### 💡 해결하는 문제

**기존 방식의 문제점:**
- 🕒 **시간 낭비**: 매번 머지할 때마다 변경로그를 수동으로 작성 (평균 20-30분 소요)
- 📝 **불완전한 문서화**: 바쁘면 대충 작성하거나 아예 건너뛰기
- 🔍 **파악 어려움**: 몇 달 전 변경사항이 무엇이었는지 찾기 힘듦
- 👥 **팀 소통 단절**: 다른 팀원이 무슨 작업을 했는지 알기 어려움
- 🐛 **버그 추적 곤란**: 언제 어떤 변경이 있었는지 히스토리 부족

**이 스킬의 해결책:**
- ⚡ **3분 만에 완료**: 명령어 한 줄로 상세한 변경로그 자동 생성
- 📊 **AI 분석**: git diff, 커밋 메시지, Jira 티켓을 종합 분석하여 영향도까지 자동 파악
- 📝 **자동 문서화**: Confluence에 자동 게시 + 로컬 마크다운 파일 보관
- 🔗 **완벽한 추적성**: Jira 티켓에 자동 댓글, PR 링크 포함
- 👥 **팀 투명성**: 모든 변경사항이 체계적으로 기록되어 팀 전체가 공유

#### 🎯 주요 기능

**자동화된 워크플로우:**
1. **Jira 티켓 자동 감지**: `feature/SIM-123-oauth-impl` 같은 브랜치명에서 티켓 번호 추출
2. **AI 기반 코드 분석**:
   - Git diff 분석 (추가/수정/삭제된 파일)
   - 커밋 히스토리 분석
   - 코드 변경의 비즈니스 의미 파악
3. **종합 리포트 생성**:
   - 📋 비즈니스 맥락 (왜 이 변경이 필요했는가?)
   - 🔧 기술적 변경사항 (무엇이 바뀌었는가?)
   - 📊 영향도 분석 (어떤 모듈에 영향을 주는가?)
   - ⚠️ Breaking Changes 감지
   - ✅ 테스트 가이드
4. **자동 배포**:
   - Confluence 페이지 생성/업데이트
   - Jira 티켓에 댓글 추가
   - 로컬에 마크다운 파일 저장 (`change-log/2026-01-21-SIM-123.md`)

**고급 기능:**
- 🔐 **안전한 인증**: Atlassian MCP 플러그인을 통한 OAuth 인증 (API 토큰 노출 없음)
- 🔄 **세션 관리**: 세션 만료 시 자동으로 브라우저 열고 재인증 안내 (v2.1.0+)
- 🌍 **한국어 지원**: 모든 안내 메시지와 에러 메시지가 한국어
- 📄 **스마트 페이지 관리**: 월별로 자동 그룹핑 (예: "Change Log - 2026-01 - OAuth 인증 구현")
- 🔗 **PR 연동**: GitHub PR 정보 자동 추출 및 링크 포함

#### 📈 실제 사용 예시

```bash
# feature/SIM-71-oauth-implementation 브랜치에서
$ /change-log

# 3분 후...
✅ 변경 로그가 성공적으로 생성되었습니다!

📄 Confluence: https://mycompany.atlassian.net/wiki/spaces/DEV/pages/987654321
💾 로컬 파일: change-log/2026-01-21-SIM-71.md
📝 Jira 티켓 업데이트됨: https://mycompany.atlassian.net/browse/SIM-71
📊 통계: 15 파일, +342/-89 줄, 8개 커밋
```

**생성되는 변경로그 예시:**

```markdown
# SIM-71: OAuth2 사용자 인증 구현

**날짜:** 2026-01-21
**담당자:** 김단우 (danwoo.kim@company.com)
**브랜치:** feature/SIM-71-oauth-implementation
**Jira 티켓:** [SIM-71](https://mycompany.atlassian.net/browse/SIM-71) - In Progress
**PR:** [#142](https://github.com/company/project/pull/142) - Add OAuth2 authentication

## 📋 개요
기존 세션 기반 인증 방식에서 OAuth2 표준을 도입하여 더 안전하고 확장 가능한
인증 시스템으로 전환. Google, GitHub 등 외부 인증 제공자 연동 가능.

## 🔧 주요 기술적 변경사항
1. **인증 서비스 리팩토링**
   - AuthService에 OAuth2 provider 추상화 추가
   - JWT 토큰 발급 및 검증 로직 구현
   - Refresh token 메커니즘 추가

2. **데이터베이스 스키마 변경**
   - oauth_tokens 테이블 신규 추가
   - users 테이블에 oauth_provider 컬럼 추가

3. **API 엔드포인트 추가**
   - POST /api/auth/oauth/authorize
   - POST /api/auth/oauth/callback
   - POST /api/auth/refresh

## 📊 영향도 분석
### 영향 받는 모듈
- 인증 시스템 (auth/)
- 사용자 관리 (users/)
- API Gateway

### Breaking Changes
⚠️ 기존 /api/login 엔드포인트는 deprecated 처리. v2.0에서 제거 예정.

## 📁 변경된 파일 (15개)
### 신규 추가
- src/auth/oauth/OAuthService.kt
- src/auth/oauth/providers/GoogleProvider.kt
- src/auth/oauth/providers/GitHubProvider.kt

### 수정
- src/auth/AuthService.kt
- src/auth/AuthController.kt
...
```

#### 💼 비즈니스 가치

**시간 절감:**
- 변경로그 작성 시간: 30분 → 3분 (90% 감소)
- 주 5회 머지 기준: 월 10시간 절감

**품질 향상:**
- 모든 변경사항이 빠짐없이 기록
- AI가 파악한 영향도 분석으로 잠재적 이슈 조기 발견
- 일관된 포맷으로 가독성 향상

**협업 강화:**
- 팀원 누구나 최근 변경사항 쉽게 파악
- 신입 온보딩 시 히스토리 학습 용이
- 크로스팀 커뮤니케이션 개선

#### ⚙️ 사전 요구사항

- ⚠️ [Atlassian MCP 플러그인](https://github.com/modelcontextprotocol/servers/tree/main/src/atlassian) 설치 및 인증 필수
- Git 저장소
- Confluence Space 및 Parent Page 정보

#### 🎬 빠른 시작

```bash
# 1. 스킬 설치
/plugin install change-log

# 2. 첫 실행 시 설정 (Confluence URL만 입력)
/change-log
# → Confluence 페이지 URL을 입력하면 Space Key와 Page ID 자동 추출

# 3. 이후 사용 (자동화!)
/change-log
```

[→ 전체 문서 및 고급 설정 보기](skills/change-log/)

---

### 🛠️ [api-codegen](skills/api-codegen/)
**프로덕션 레벨 API 클라이언트 생성기**

Swagger/OpenAPI 명세서를 보고 API 클라이언트 코드를 수동으로 작성하느라 시간을 낭비하시나요? 이 스킬은 Swagger 문서를 분석하여 프로젝트의 코드 스타일과 프레임워크에 완벽하게 맞는 타입 안전한 API 클라이언트 코드를 자동으로 생성하고, 테스트 코드까지 함께 만들어줍니다.

#### 💡 해결하는 문제

**기존 방식의 문제점:**
- 🕒 **시간 낭비**: API 엔드포인트마다 수동으로 클라이언트 코드 작성 (엔드포인트당 15-30분)
- 🐛 **타입 에러**: API 스펙과 클라이언트 코드 불일치로 런타임 에러 발생
- 📝 **불일치**: 백엔드 API 변경 시 프론트엔드 코드 동기화 누락
- 🔧 **일관성 부족**: 개발자마다 다른 스타일로 API 호출 코드 작성
- 🧪 **테스트 부족**: API 클라이언트 테스트 코드 작성을 자주 건너뜀

**이 스킬의 해결책:**
- ⚡ **5분 만에 완료**: Swagger URL 하나로 전체 API 클라이언트 + 테스트 생성
- 🎯 **타입 안전성**: OpenAPI 스펙에서 정확한 타입 정보 추출하여 컴파일 타임에 에러 감지
- 🎨 **프로젝트 통합**: 기존 프로젝트의 코드 스타일, 네이밍 규칙, 프레임워크 자동 감지 및 적용
- ✅ **테스트 자동 생성**: 단위 테스트 + 통합 테스트 + 모킹 코드까지 함께 생성
- 🔄 **동기화 쉬움**: API 스펙 변경 시 재생성만 하면 즉시 동기화

#### 🎯 주요 기능

**지능형 프로젝트 분석:**
1. **코드 스타일 자동 감지**:
   - 네이밍 규칙 (camelCase, snake_case, PascalCase)
   - 들여쓰기 (spaces vs tabs, 2 vs 4)
   - Import 스타일 및 순서
   - 주석 스타일
2. **프레임워크 자동 인식**:
   - **Backend**: Spring Boot, Express, FastAPI, Django, NestJS
   - **Frontend**: React, Vue, Angular, Next.js
   - **Mobile**: Kotlin (Android), Swift (iOS)
3. **프로젝트 구조 분석**:
   - 패키지/모듈 구조 파악
   - 기존 API 클라이언트 코드 패턴 학습
   - 의존성 관리 방식 (Gradle, Maven, npm, pip)

**자동 코드 생성:**
1. **타입 정의**: OpenAPI 스키마에서 정확한 타입/인터페이스 생성
2. **API 클라이언트 클래스**: 모든 엔드포인트에 대한 메서드 생성
3. **에러 핸들링**: 표준 에러 처리 로직 포함
4. **인증 지원**: Bearer Token, API Key, OAuth 등
5. **Request/Response 변환**: 직렬화/역직렬화 로직 자동 생성

**포괄적인 테스트 생성:**
- **단위 테스트**: 각 API 메서드별 테스트
- **통합 테스트**: 실제 API 호출 시나리오 테스트
- **모킹**: MockWebServer, MSW 등을 활용한 테스트 더미 데이터

#### 📈 실제 사용 예시

**시나리오 1: Spring Boot 프로젝트에서 결제 API 연동**

```bash
# Swagger URL로 API 클라이언트 생성
$ /api-codegen https://api.payment-gateway.com/swagger.json

🔍 프로젝트 분석 중...
✅ Spring Boot (Kotlin) 프로젝트 감지
✅ Gradle 빌드 시스템
✅ 패키지 구조: com.company.payment

📋 API 스펙 분석 중...
✅ 12개 엔드포인트 발견
✅ 5개 스키마 정의

🎨 코드 생성 중...
✅ PaymentApiClient.kt 생성
✅ PaymentModels.kt (5개 data class)
✅ PaymentApiClientTest.kt (12개 테스트)

📦 의존성 추가됨:
- implementation("com.squareup.retrofit2:retrofit:2.9.0")
- testImplementation("com.squareup.okhttp3:mockwebserver:4.10.0")
```

**생성된 코드 예시:**

```kotlin
// PaymentApiClient.kt
@Service
class PaymentApiClient(
    private val restTemplate: RestTemplate
) {
    private val baseUrl = "https://api.payment-gateway.com"

    /**
     * 결제 생성
     * POST /api/v1/payments
     */
    fun createPayment(request: CreatePaymentRequest): PaymentResponse {
        return restTemplate.postForObject(
            "$baseUrl/api/v1/payments",
            request,
            PaymentResponse::class.java
        ) ?: throw ApiException("Payment creation failed")
    }

    /**
     * 결제 상태 조회
     * GET /api/v1/payments/{paymentId}
     */
    fun getPayment(paymentId: String): PaymentResponse {
        return restTemplate.getForObject(
            "$baseUrl/api/v1/payments/$paymentId",
            PaymentResponse::class.java
        ) ?: throw ApiException("Payment not found")
    }
}

// PaymentModels.kt
data class CreatePaymentRequest(
    val amount: Long,
    val currency: String,
    val customerId: String,
    val description: String?
)

data class PaymentResponse(
    val id: String,
    val amount: Long,
    val currency: String,
    val status: PaymentStatus,
    val createdAt: Instant
)

enum class PaymentStatus {
    PENDING, COMPLETED, FAILED, CANCELLED
}
```

**시나리오 2: React 프로젝트에서 백엔드 API 연동**

```bash
$ /api-codegen http://localhost:8080/v3/api-docs

🔍 프로젝트 분석 중...
✅ React (TypeScript) 프로젝트 감지
✅ Axios 사용 중
✅ 디렉토리 구조: src/api/

📋 API 스펙 분석 중...
✅ 25개 엔드포인트 발견

🎨 코드 생성 중...
✅ src/api/userApi.ts
✅ src/api/productApi.ts
✅ src/api/types.ts
✅ src/api/__tests__/userApi.test.ts

💡 React Query 훅도 생성할까요? (y/n)
> y

✅ src/api/hooks/useUser.ts
✅ src/api/hooks/useProduct.ts
```

**생성된 코드 예시:**

```typescript
// src/api/userApi.ts
import axios from 'axios';
import { User, CreateUserRequest, UpdateUserRequest } from './types';

const BASE_URL = process.env.REACT_APP_API_URL || 'http://localhost:8080';

export const userApi = {
  /**
   * 사용자 목록 조회
   * GET /api/users
   */
  async getUsers(page = 0, size = 20): Promise<User[]> {
    const response = await axios.get(`${BASE_URL}/api/users`, {
      params: { page, size }
    });
    return response.data;
  },

  /**
   * 사용자 생성
   * POST /api/users
   */
  async createUser(request: CreateUserRequest): Promise<User> {
    const response = await axios.post(`${BASE_URL}/api/users`, request);
    return response.data;
  }
};

// src/api/hooks/useUser.ts (React Query)
import { useQuery, useMutation } from '@tanstack/react-query';
import { userApi } from '../userApi';

export function useUsers(page = 0, size = 20) {
  return useQuery({
    queryKey: ['users', page, size],
    queryFn: () => userApi.getUsers(page, size)
  });
}

export function useCreateUser() {
  return useMutation({
    mutationFn: userApi.createUser
  });
}
```

#### 💼 비즈니스 가치

**개발 시간 절감:**
- API 클라이언트 작성 시간: 엔드포인트당 30분 → 5분 (전체 생성)
- 20개 엔드포인트 기준: 10시간 → 5분 (98% 감소)
- 테스트 코드까지 포함하면 더 큰 시간 절감

**품질 향상:**
- 타입 안전성으로 런타임 에러 90% 감소
- 일관된 코드 스타일로 유지보수성 향상
- 자동 생성된 테스트로 커버리지 증가

**협업 강화:**
- 백엔드-프론트엔드 스펙 불일치 제로
- 신규 개발자도 즉시 API 사용 가능
- API 문서 = 실제 코드 (항상 동기화)

#### 🎛️ 지원하는 언어 & 프레임워크

**Backend:**
- ☕ **Java**: Spring Boot, Spring WebClient, OkHttp
- 🔷 **Kotlin**: Spring Boot, Ktor Client, Retrofit
- 🟦 **TypeScript**: Express, NestJS, Axios
- 🐍 **Python**: FastAPI, Django, requests, httpx

**Frontend:**
- ⚛️ **React**: Axios, Fetch API, React Query
- 💚 **Vue**: Axios, Fetch API, VueUse
- 🅰️ **Angular**: HttpClient
- ⚡ **Next.js**: Server/Client Components, SWR

**Mobile:**
- 🤖 **Android**: Retrofit, OkHttp, Ktor
- 🍎 **iOS**: Alamofire, URLSession

#### ⚙️ 사전 요구사항

- Swagger/OpenAPI 스펙 (URL 또는 로컬 파일)
- 기존 프로젝트 (또는 새 프로젝트 생성 가능)

#### 🎬 빠른 시작

```bash
# 1. 스킬 설치
/plugin install api-codegen

# 2. Swagger URL로 생성
/api-codegen https://api.example.com/swagger.json

# 3. 또는 로컬 파일로 생성
/api-codegen ./docs/openapi.yaml

# 4. 대화형으로 커스터마이징
# → 언어 선택
# → HTTP 라이브러리 선택
# → 테스트 프레임워크 선택
# → 생성 위치 지정
```

[→ 전체 문서 및 고급 설정 보기](skills/api-codegen/)

---

## 🚀 빠른 시작

### 마켓플레이스를 통한 설치

1. **마켓플레이스 추가:**
```bash
/plugin marketplace add kdanuu/tna-plugin-marketplace
```

2. **스킬 설치:**
```bash
# 변경로그 생성기 설치
/plugin install change-log

# 또는 API 코드 생성기 설치
/plugin install api-codegen
```

3. **다음 대화에서 사용:**
```bash
/change-log
# 또는
/api-codegen https://api.example.com/swagger.json
```

### 설치 확인

Claude에게 사용 가능한 스킬 목록을 요청하세요:
```
What skills are available?
```

응답에서 설치된 스킬을 확인할 수 있습니다.

## 📖 사용 방법

각 스킬은 자체 종합 문서를 제공합니다:
- [change-log 사용 가이드](skills/change-log/)
- [api-codegen 사용 가이드](skills/api-codegen/)

기본 사용 패턴:
```bash
# 스킬 명령어 사용
/스킬이름 [옵션]

# 또는 자연어로
"변경로그 생성해줘"
"swagger에서 API 클라이언트 만들어줘"
```

## 🗂️ 저장소 구조

```
.
├── skills/
│   ├── change-log/                    # 변경로그 생성 플러그인
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json            # 플러그인 매니페스트
│   │   └── SKILL.md                   # 스킬 프롬프트 & 문서
│   └── api-codegen/                   # API 코드 생성기 플러그인
│       ├── .claude-plugin/
│       │   └── plugin.json            # 플러그인 매니페스트
│       └── SKILL.md                   # 스킬 프롬프트 & 문서
├── .claude-plugin/
│   └── marketplace.json               # 마켓플레이스 설정
└── README.md                          # 이 파일
```

각 플러그인은 다음을 포함합니다:
- **`.claude-plugin/plugin.json`**: 플러그인 메타데이터 (이름, 버전, 설명, 작성자)
- **`SKILL.md`**: 실제 스킬 프롬프트 및 상세 문서

## 🔮 로드맵 & 향후 스킬

생산성을 높이는 새로운 스킬로 이 컬렉션을 지속적으로 확장하고 있습니다. 계획된 추가 사항:

- 🧪 **test-generator**: 기존 코드에서 지능형 테스트 생성
- 📚 **doc-sync**: 코드 변경사항과 문서 동기화 유지
- 🔐 **security-audit**: 자동화된 보안 취약점 스캐닝
- 🎯 **code-reviewer**: AI 기반 코드 리뷰 및 제안
- 🔄 **migration-helper**: 프레임워크/라이브러리 마이그레이션 지원

*새로운 스킬에 대한 아이디어가 있으신가요?* [이슈 열기](https://github.com/kdanuu/tna-plugin-marketplace/issues) 또는 풀 리퀘스트를 제출해주세요!

## 🤝 기여하기

기여를 환영합니다! 다음과 같이 도울 수 있습니다:

1. **버그 리포트 또는 기능 요청** - [GitHub Issues](https://github.com/kdanuu/tna-plugin-marketplace/issues)를 통해
2. **개선사항 제출** - 풀 리퀘스트를 통해
3. **자신의 스킬 공유** - 포함시키고 싶습니다!

### 새 스킬 추가하기

1. `skills/` 아래에 새 디렉토리 생성
2. 스킬 프롬프트가 포함된 `SKILL.md` 파일 추가
3. `.claude-plugin/marketplace.json` 업데이트
4. 스킬을 철저히 테스트
5. 풀 리퀘스트 제출

기존 스킬을 참조 구조로 확인하세요.

## 📋 요구사항

- [Claude Code CLI](https://github.com/anthropics/claude-code) (최신 버전 권장)
- Git (버전 관리 기능용)
- 스킬별 추가 요구사항:
  - **change-log**: Jira/Confluence 통합을 위한 [Atlassian MCP 플러그인](https://github.com/modelcontextprotocol/servers/tree/main/src/atlassian) 필수
  - 전체 요구사항은 개별 스킬 문서 참조

## 📄 라이선스

MIT License - 자세한 내용은 [LICENSE](LICENSE) 파일 참조

## 👤 제작자

**danwoo-kim** ([@kdanuu](https://github.com/kdanuu))이 제작 및 유지 관리합니다.

## 🌟 지원하기

이 스킬들이 도움이 되었다면:
- ⭐ 이 저장소에 스타를 눌러주세요
- 🐛 발견한 이슈를 리포트해주세요
- 💡 새로운 기능이나 스킬을 제안해주세요
- 📢 팀과 공유해주세요

---

**Claude와 함께 즐거운 코딩하세요!** 🎉

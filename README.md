# Claude Code 스킬 컬렉션

[Claude Code](https://github.com/anthropics/claude-code)를 위한 프로덕션 레벨의 스킬 모음으로, AI 기반 자동화를 통해 개발 워크플로우를 향상시킵니다.

## 📦 제공되는 스킬

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

- ⚠️ **Atlassian MCP 플러그인 설치 및 인증 필수**
  - `/plugin` 명령어로 플러그인 동작후
  - `Marketplace` 으로 이동후`anthropics/claude-plugins-official ` 검색 및 설치
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

## 🚀 빠른 시작

### 사전 준비: Atlassian MCP 플러그인 설치 (change-log 사용 시 필수)

⚠️ **change-log 스킬을 사용하려면 Atlassian MCP 플러그인 설치가 필수입니다.**

1. **Claude Code에서 플러그인 브라우저 열기:**
```bash
/plugin
```

2. **Atlassian 플러그인 검색 및 설치:**
   - 플러그인 목록에서 `atlassian@claude-plugins-official` 찾기
   - 없다면 마켓플레이스에서 `anthropics/claude-plugins-official` 설치
     - Discover 탭에서 `atlassian@claude-plugins-official` 설치
   - 재시작 후 플러그인 목록에서 `atlassian@claude-plugins-official` 찾기

3. **Atlassian 계정 인증:**
   - 플러그인 설치 후 브라우저가 자동으로 열립니다
   - Atlassian 계정으로 로그인
   - Claude Code에 권한 승인
   - 인증 완료!

4. **인증 확인:**
    - 플러그인 상태가 `enabled`인지 확인
    - atlassian MCP 서버와 연결됨을 의미하는 `Status: ✔ connected`, `Auth: ✔ authenticated` 확인
---

### 마켓플레이스를 통한 스킬 설치

1. **마켓플레이스 추가:**
```bash
/plugin marketplace add kdanuu/tna-plugin-marketplace
```

2. **스킬 설치:**
```bash
# 변경로그 생성기 설치 (MCP 플러그인 필수)
/plugin install change-log
```

3. **다음 대화에서 사용:**
```bash
/change-log
```

### 설치 확인

Claude에게 사용 가능한 스킬 목록을 요청하세요:
```
What skills are available?
```

응답에서 설치된 스킬을 확인할 수 있습니다.

### 🔄 MCP 토큰 만료 시 재인증

change-log 사용 중 "MCP 세션이 만료되었습니다" 에러가 발생하면:

1. **플러그인 목록 열기:**
```bash
/plugin
```

2. **Atlassian 플러그인 선택:**
   - 플러그인 목록에서 `atlassian@claude-plugins-official` 또는 `Plugin:atlassian:atlassian MCP Server` 찾기
   - Enter를 눌러 선택

3. **재인증 실행:**
   - 메뉴에서 `2. Re-authenticate` 선택
   - 브라우저가 자동으로 열립니다
   - Atlassian 계정으로 다시 로그인
   - 권한 승인 후 완료!

4. **재인증 확인:**
   - 플러그인 상태가 `Status: ✔ connected`, `Auth: ✔ authenticated`인지 확인
   - 이제 /change-log를 다시 사용할 수 있습니다

> 💡 **v2.1.0+**: change-log 스킬은 세션 만료 감지 시 에러 메시지를 표시합니다.

## 📖 사용 방법

change-log 스킬은 자체 종합 문서를 제공합니다:
- [change-log 사용 가이드](skills/change-log/)

기본 사용 패턴:
```bash
# 스킬 명령어 사용
/change-log

# 또는 자연어로
"변경로그 생성해줘"
```

## 🗂️ 저장소 구조

```
.
├── skills/
│   └── change-log/                    # 변경로그 생성 스킬
│       ├── .claude-plugin/
│       │   └── plugin.json            # 플러그인 매니페스트
│       └── SKILL.md                   # 스킬 프롬프트 & 문서
├── .claude-plugin/
│   └── marketplace.json               # 마켓플레이스 설정
└── README.md                          # 이 파일
```

각 스킬은 다음을 포함합니다:
- **`.claude-plugin/plugin.json`**: 스킬 메타데이터 (이름, 버전, 설명, 작성자)
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
- **change-log** 스킬 요구사항:
  - Jira/Confluence 통합을 위한 [Atlassian MCP 플러그인](https://github.com/modelcontextprotocol/servers/tree/main/src/atlassian) 필수
  - 전체 요구사항은 [스킬 문서](skills/change-log/) 참조

## 📄 라이선스

MIT License - 자세한 내용은 [LICENSE](LICENSE) 파일 참조

## 👤 제작자

**danwoo-kim** ([@kdanuu](https://github.com/kdanuu))이 제작 및 유지 관리합니다.

## 🌟 지원하기

이 스킬이 도움이 되었다면:
- ⭐ 이 저장소에 스타를 눌러주세요
- 🐛 발견한 이슈를 리포트해주세요
- 💡 새로운 기능이나 스킬을 제안해주세요
- 📢 팀과 공유해주세요

---

**Claude와 함께 즐거운 코딩하세요!** 🎉

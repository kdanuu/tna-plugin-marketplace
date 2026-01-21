# Change Log - 2026-01 - Atlassian MCP 플러그인 마이그레이션 및 모노레포 전환

**날짜:** 2026-01-21

**담당자:** 김단우 (danwoo.kim@myrealtrip.com)

**브랜치:** main

**커밋 범위:** 최근 20개 커밋 (9458b95..886d93c)

---

## 📋 개요

본 변경 로그는 Claude Code Skills 프로젝트의 주요 아키텍처 변경사항을 포함합니다. 핵심 변경사항은 **change-log 스킬을 Atlassian MCP 플러그인 기반으로 마이그레이션**한 것이며, 이는 **BREAKING CHANGE**를 포함합니다. 또한 프로젝트가 단일 스킬에서 **모노레포 구조의 스킬 컬렉션**으로 전환되었으며, **api-codegen** 스킬이 새롭게 추가되었습니다.

이러한 변경은 다음과 같은 비즈니스 가치를 제공합니다:
- **보안 강화**: 로컬 토큰 저장 없이 MCP 플러그인을 통한 안전한 인증
- **설정 간소화**: 설정 항목 80% 감소로 초기 사용자 진입 장벽 완화
- **확장성**: 모노레포 구조로 여러 스킬을 효율적으로 관리
- **사용자 경험 개선**: 한국어 인터페이스 지원 및 대화형 설정 프로세스

---

## 🔧 주요 기술적 변경사항

### 1. **change-log 스킬 v2.0.0 - Atlassian MCP 플러그인 마이그레이션** ⚠️ BREAKING CHANGE

**배경:**
기존에는 사용자가 Jira/Confluence API 토큰, 이메일, URL 등을 직접 설정 파일에 저장해야 했습니다. 이는 보안 위험이 있고 설정이 복잡했습니다.

**주요 변경:**
- **인증 방식 변경**: REST API 직접 호출 → Atlassian MCP 플러그인 사용
- **설정 간소화**:
  - 기존: API 토큰, 이메일, Jira URL, Confluence URL, Space Key, Parent Page ID
  - 변경 후: Space Key, Parent Page ID만 필요 (80% 감소)
- **API 호출 교체**:
  - `mcp__plugin_atlassian_atlassian__getJiraIssue`: Jira 티켓 정보 조회
  - `mcp__plugin_atlassian_atlassian__createConfluencePage`: 새 페이지 생성
  - `mcp__plugin_atlassian_atlassian__updateConfluencePage`: 기존 페이지 업데이트
  - `mcp__plugin_atlassian_atlassian__addCommentToJiraIssue`: Jira 티켓 코멘트 추가

**이점:**
- 🔒 **보안**: 로컬에 API 토큰 저장 불필요
- 🎯 **사용성**: 설정 프로세스 단순화 (URL 복사-붙여넣기로 자동 파싱)
- 🔄 **표준화**: MCP 표준 인터페이스 사용
- 🚀 **유지보수**: Atlassian API 변경 시 MCP 플러그인이 자동 대응

**마이그레이션 가이드:**
기존 사용자는 다음 단계를 따라야 합니다:
1. `@modelcontextprotocol/server-atlassian` 설치
2. Claude Code 설정에서 Atlassian MCP 인증 완료
3. 기존 `~/.claude/confluence-changelog.json`에서 불필요한 필드 제거 (API 토큰, 이메일, URL)
4. `/change-log` 재실행 시 자동으로 새로운 설정 프로세스 안내

### 2. **한국어 지원 및 PR 추적 (v1.3.0-v1.3.1)**

- **한국어 인터페이스**: 모든 질문, 확인, 오류 메시지를 한국어로 제공
- **GitHub PR 추적**: `gh pr list` 명령어를 통해 자동으로 PR 정보 추출
- **로컬 마크다운 저장**: 프로젝트 루트의 `change-log/` 디렉토리에 마크다운 형식으로 저장
  - 파일명 형식: `{YYYY-MM-DD}-{jira-ticket-or-branch-name}.md`
  - 버전 관리 및 오프라인 참조 가능

### 3. **api-codegen 스킬 추가 (v1.0.0 → v1.1.0)**

**새로운 스킬**: Swagger/OpenAPI 문서에서 프로덕션 레디 API 클라이언트 코드를 생성하는 스킬

**핵심 기능:**
- 📋 Swagger/OpenAPI 파싱 (URL 또는 로컬 파일)
- 🔍 프로젝트 환경 자동 감지 (언어, 프레임워크, 코드 스타일)
- 🎨 프로젝트 규약에 맞는 코드 생성
- ✅ 단위/통합 테스트 자동 생성
- 🔧 다국어 지원: Kotlin, Java, TypeScript, Python
- 🏗️ 프레임워크 인식: Spring Boot, React, Vue, FastAPI 등
- 🇰🇷 **v1.1.0**: 사용자 대면 메시지 한글화

**사용 시나리오:**
- 마이크로서비스 간 통합
- 서드파티 API 연동
- 백엔드-프론트엔드 간 계약 동기화

### 4. **모노레포 구조로 전환**

**아키텍처 변경:**
```
Before:
├── SKILL.md
└── .claude-plugin/

After:
├── skills/
│   ├── change-log/
│   │   ├── .claude-plugin/plugin.json
│   │   └── SKILL.md
│   └── api-codegen/
│       ├── .claude-plugin/plugin.json
│       └── SKILL.md
├── .claude-plugin/marketplace.json
└── README.md
```

**이점:**
- 여러 스킬을 하나의 저장소에서 관리
- 각 스킬의 독립적인 버전 관리
- 공통 문서 및 리소스 공유
- 마켓플레이스 등록 간소화

### 5. **플러그인 매니페스트 표준화**

각 스킬에 Claude Code 플러그인 표준을 따르는 `plugin.json` 추가:
- 메타데이터: name, description, version, author
- 키워드: 검색 최적화
- 사용자 호출 가능 여부: `user-invocable: true`

**마켓플레이스 등록:**
- `.claude-plugin/marketplace.json` 추가
- GitHub 기반 소스 정의: `{ "source": "github", "repo": "kdanuu/tna-plugin-marketplace" }`

---

## 📊 영향도 분석

### 영향 받는 모듈
1. **change-log 스킬**: BREAKING CHANGE - MCP 플러그인 필수
2. **api-codegen 스킬**: 신규 추가 - 기존 시스템에 영향 없음
3. **프로젝트 구조**: 모노레포 전환 - 기존 사용자 마이그레이션 필요
4. **문서**: README 및 설정 가이드 전면 개편

### 신규 의존성
- `@modelcontextprotocol/server-atlassian`: Atlassian MCP 플러그인 (필수)
- `gh` CLI: GitHub PR 정보 추출용 (선택)

### 호환성
- **하위 호환성 없음**: change-log v1.x 사용자는 v2.0.0으로 업그레이드 시 재설정 필요
- **순방향 호환성**: 새로운 설정 형식은 향후 업데이트와 호환
- **Claude Code 버전**: 최신 버전 권장 (MCP 플러그인 지원 필요)

---

## 📁 변경된 파일 (8개)

### 신규 추가 (주요 파일)
- **skills/api-codegen/.claude-plugin/plugin.json** (27줄): api-codegen 플러그인 매니페스트
- **skills/api-codegen/SKILL.md** (974줄): api-codegen 스킬 전체 문서
- **skills/change-log/.claude-plugin/plugin.json** (23줄): change-log 플러그인 매니페스트

### 수정
- **skills/change-log/SKILL.md** (+392/-기존): MCP 플러그인 마이그레이션, 한국어 지원, PR 추적
- **.claude-plugin/marketplace.json** (31줄): 모노레포 구조 반영, 각 스킬 경로 정의
- **README.md** (+205/-기존): 스킬 컬렉션 소개, 설치 가이드, 로드맵
- **README_KR.md** (+205/-기존): 한국어 버전 README

### 삭제
- **.claude-plugin/plugin.json** (11줄): 루트 플러그인 매니페스트 (모노레포로 이동)

---

## 💻 주요 커밋 히스토리 (20개 커밋)

1. **886d93c** - Migrate change-log skill to use Atlassian MCP plugin (v2.0.0)
2. **3e88abb** - Add Korean language support to change-log skill (v1.3.1)
3. **e9d4479** - Add PR tracking and local markdown export to change-log skill (v1.3.0)
4. **ad02147** - Update repository references from change-log to tna-plugin-marketplace
5. **dfb202b** - Add plugin manifests to each skill following Claude Code plugin standards
6. **25d6c18** - Bump api-codegen version to 1.1.0
7. **368fabc** - api-codegen 스킬의 사용자 대면 메시지를 한글로 변경
8. **686b359** - Remove obsolete plugin.json from root .claude-plugin directory
9. **14ed65a** - Update README to reflect monorepo structure with multiple skills
10. **0f36c4c** - Remove api-explorer from marketplace.json
11. **1760ef1** - Remove api-explorer skill
12. **7a575b0** - Fix marketplace: Add path field for each plugin in monorepo
13. **0b8c316** - Add api-codegen plugin: Generate API client code from Swagger
14. **c9d20f8** - Add api-explorer plugin with interactive clarification
15. **98090a8** - Fix changelog format: Confluence detailed, Jira brief
16. **f318a90** - Bump version to 1.1.0
17. **03f8334** - Improve change-log skill: concise Confluence format + Jira comment support
18. **9573c88** - Remove unsupported category field from plugin manifests
19. **e3a2fab** - Fix marketplace.json source format
20. **9458b95** - Convert to Claude Code plugin marketplace format

---

## 📈 코드 통계
- **변경된 파일:** 8개
- **추가:** +1,610줄
- **삭제:** -258줄
- **순 증가:** +1,352줄

---

## ✅ 완료 사항

### change-log 스킬 (v2.0.0)
- ✅ Atlassian MCP 플러그인 통합
- ✅ 설정 간소화 (80% 감소)
- ✅ 한국어 인터페이스
- ✅ PR 자동 추적
- ✅ 로컬 마크다운 저장
- ✅ 대화형 URL 파싱 설정

### api-codegen 스킬 (v1.1.0)
- ✅ Swagger/OpenAPI 파서
- ✅ 프로젝트 환경 자동 감지
- ✅ 다국어 코드 생성 (Kotlin, Java, TypeScript, Python)
- ✅ 테스트 코드 자동 생성
- ✅ 한국어 인터페이스

### 인프라
- ✅ 모노레포 구조 전환
- ✅ 플러그인 매니페스트 표준화
- ✅ 마켓플레이스 등록
- ✅ README 개편 (EN/KR)

---

## 🔮 향후 계획

### 단기 (Q1 2026)
- test-generator: AI 기반 단위 테스트 생성
- doc-sync: 코드와 문서 동기화

### 중기 (Q2 2026)
- code-reviewer: PR 자동 리뷰
- migration-assistant: 프레임워크 마이그레이션 도우미

---

*생성일: 2026-01-21*
*작성자: Claude Code - change-log skill v2.0.0*

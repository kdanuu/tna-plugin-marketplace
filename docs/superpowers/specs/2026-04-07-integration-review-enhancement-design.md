# integration-review 스킬 고도화 설계

## 목표

integration-review 스킬의 인증 방식을 Atlassian MCP 기반으로 전환하고, 온보딩을 개선하며, 인증 사전 체크를 자동화한다.

## 변경 사항

### 1. 인증 방식 전환: Basic Auth curl → Atlassian MCP 도구

**현재**: `~/.claude/integration-review.json`에 email + API Token 저장 → curl에 Basic Auth 헤더 사용
**변경후**: Atlassian MCP 플러그인 OAuth 로그인 (브라우저) → MCP 도구로 Confluence 조작

- `integration-review.json` 설정 파일 완전 제거
- Confluence API 호출: **curl 제거, MCP 도구로 전면 전환** (change-log 스킬과 동일 패턴)
- Google Sheets API 호출: 기존 gcloud + curl 방식 유지 (변경 없음)
- Confluence 경로 정보(spaceKey, parentPageId 등)는 SKILL.md에 하드코딩 유지

#### 사용할 MCP 도구 (change-log 스킬에서 검증 완료)

| MCP 도구 | 용도 |
|----------|------|
| `getAccessibleAtlassianResources` | Cloud ID 조회 |
| `getConfluenceSpaces` | Space Key → Space ID 변환 |
| `getPagesInConfluenceSpace` | 동일 제목 페이지 중복 확인 |
| `createConfluencePage` | 1Pager 페이지 생성 |

#### Confluence 인증 검증 변경

```
Before: curl + Basic Auth로 /wiki/api/v2/spaces 호출해서 200 확인
After:  getAccessibleAtlassianResources 호출 성공 여부로 확인
```

#### Markdown → Confluence 변환

기존 SKILL.md의 Confluence Storage Format(XHTML) 템플릿을 그대로 사용.
`createConfluencePage`에 body를 storage format으로 전달.

### 2. 온보딩 강화

스킬 첫 실행 시 (Step 1 인트로 확장):

```
안녕하세요! 신규 연동 1Pager 작성을 도와드릴게요.

이 스킬은 다음을 자동으로 처리합니다:
1. 대화형 질의응답으로 1Pager 문서 작성
2. Confluence에 자동 게시 (TNA 3.0 1Pager 하위)
3. 연동 검토 Google Sheet 업데이트

필요한 준비:
- Atlassian 로그인 (/login)
- gcloud 인증 (gcloud auth login)

약 5~10분 정도 소요되며, 잘 모르는 항목은 "모르겠어요"라고 하셔도 됩니다.
그럼 시작할게요!
```

### 3. 인증 사전 체크 (스킬 내 Step 0)

bash hook으로는 MCP 세션 확인이 불가능하므로, **스킬 Process의 Step 0**에서 인증 체크를 수행한다 (change-log 스킬과 동일 패턴).

**동작**:
1. `getAccessibleAtlassianResources` 호출로 Atlassian MCP 세션 확인
2. `gcloud auth print-access-token` 실행으로 gcloud 상태 확인
3. 결과에 따른 안내:

| Atlassian | gcloud | 동작 |
|-----------|--------|------|
| OK | OK | 그대로 진행 |
| FAIL | OK | "Atlassian 로그인이 필요합니다. `/login`으로 로그인해주세요. 질문 수집은 먼저 진행할게요." |
| OK | FAIL | "gcloud 인증이 필요합니다. `! gcloud auth login`을 실행해주세요." |
| FAIL | FAIL | 둘 다 안내 |

**핵심**: 인증 실패해도 **차단하지 않음**. 질문 수집은 인증 없이 진행 가능. Confluence 게시 직전에 다시 인증 확인.

### 4. SKILL.md 구조 변경 요약

#### 삭제
- Prerequisites > Confluence (API Token) 섹션
- Initial Setup 전체 (Step 0, Setup Process, Configuration Fields)
- Process > 0. 인증 검증 (Basic Auth curl)
- Confluence 게시의 모든 curl 호출 (Space ID 조회, 중복 확인, 페이지 생성)
- Error Handling > Confluence 인증 실패 (401) 토큰 갱신/발급 안내
- Error Handling > 설정 파일 오류
- Tools Required > Read/Write (설정 파일용)

#### 수정
- Prerequisites: MCP 로그인 + gcloud로 간소화
- Process > 1. 인트로: 온보딩 가이드 강화
- Process > 6. Confluence 게시: curl → MCP 도구 (createConfluencePage 등)
- Error Handling: MCP 재로그인 안내로 대체
- allowed-tools: MCP Atlassian 도구 추가
- frontmatter > allowed-tools에 MCP 도구 명시

#### 추가
- Step 0: 인증 사전 체크 (MCP + gcloud)
- 온보딩 메시지

## 영향 범위

| 파일 | 변경 |
|------|------|
| `skills/integration-review/SKILL.md` | 인증/온보딩/Confluence 연동/에러 핸들링 전면 수정 |
| `~/.claude/integration-review.json` | 삭제 (더 이상 불필요) |
| `.claude-plugin/plugin.json` | 버전 업 (1.4.2 → 1.5.0) |
| `.claude-plugin/marketplace.json` | 버전 업 (1.4.2 → 1.5.0) |

## MCP 플러그인 미설치 시

Atlassian MCP 플러그인이 설치되어 있지 않으면 스킬 실행을 차단하고 안내:
```
이 스킬은 Atlassian MCP 플러그인이 필요합니다.
Claude Code 설정에서 Atlassian 플러그인을 설치해주세요.
```

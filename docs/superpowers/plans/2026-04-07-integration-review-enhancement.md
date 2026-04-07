# integration-review 스킬 고도화 구현 계획

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** integration-review 스킬의 인증을 Atlassian MCP로 전환하고, 온보딩을 강화하며, Confluence 연동을 MCP 도구로 교체한다.

**Architecture:** Basic Auth + curl 기반 Confluence 연동을 Atlassian MCP 도구로 전면 교체. change-log 스킬과 동일한 MCP 패턴 적용. Google Sheets는 기존 gcloud + curl 유지.

**Tech Stack:** Claude Code Skill (SKILL.md), Atlassian MCP Plugin, gcloud CLI

**Spec:** `docs/superpowers/specs/2026-04-07-integration-review-enhancement-design.md`

---

## Chunk 1: SKILL.md 전면 수정

### Task 1: frontmatter 및 Prerequisites 수정

**Files:**
- Modify: `skills/integration-review/SKILL.md:1-11` (frontmatter)
- Modify: `skills/integration-review/SKILL.md:14-31` (Prerequisites)

- [ ] **Step 1: frontmatter의 allowed-tools 업데이트**

기존:
```yaml
allowed-tools:
  - Bash
  - Read
  - Write
```

변경:
```yaml
allowed-tools:
  - Bash
  - Read
  - mcp__plugin_atlassian_atlassian__getAccessibleAtlassianResources
  - mcp__plugin_atlassian_atlassian__getConfluenceSpaces
  - mcp__plugin_atlassian_atlassian__getPagesInConfluenceSpace
  - mcp__plugin_atlassian_atlassian__createConfluencePage
```

`Write` 제거 (설정 파일 생성 불필요), MCP 도구 4개 추가.

- [ ] **Step 2: Prerequisites 섹션을 MCP 기반으로 교체**

기존 Prerequisites (API Token 발급 안내 등) 전체를 아래로 교체:

```markdown
## Prerequisites

**⚠️ REQUIRED: Atlassian MCP Plugin**

이 스킬은 Atlassian MCP 플러그인이 설치되고 인증되어야 합니다.

1. **Atlassian MCP 플러그인 설치 확인**:
   - `mcp__plugin_atlassian_atlassian__` 접두사를 가진 MCP 도구가 사용 가능한지 확인
   - 사용 불가 시 안내:
     ```
     ⚠️ 이 스킬을 사용하려면 Atlassian MCP 플러그인이 필요합니다.

     설치 방법:
     1. /plugin 명령어를 입력하여 플러그인 목록을 여세요
     2. 'atlassian@claude-plugins-official'를 검색하세요
     3. Enter를 눌러 설치하세요
     4. 브라우저에서 Atlassian 계정으로 로그인하세요
     5. 인증 완료 후 이 스킬을 다시 실행해주세요
     ```

2. **Google Sheets (gcloud)**:
   - `gcloud` CLI 설치 및 인증 완료
   - `gcloud auth print-access-token`으로 토큰 발급 가능한 상태
```

---

### Task 2: Initial Setup 섹션 제거 & 고정 설정 유지

**Files:**
- Modify: `skills/integration-review/SKILL.md:35-100` (Initial Setup 전체)

- [ ] **Step 1: Initial Setup 섹션 정리**

아래 섹션을 **전부 삭제**:
- Step 0: 설정 확인 (line 38~41)
- Setup Process (line 59~94)
- Configuration Fields (line 96~100)

아래 섹션만 **유지** (고정 설정 테이블):

```markdown
### 고정 설정 (스킬에 하드코딩)

다음 값은 스킬에 고정되어 있으며 변경되지 않습니다:

| 항목 | 값 |
|------|-----|
| Atlassian 도메인 | `myrealtrip.atlassian.net` |
| Confluence Space Key | `DEVX` |
| Confluence 부모 페이지 ID | `5715984961` |
| Confluence 부모 페이지 | [TNA 3.0 1Pager 2026.2Q](https://myrealtrip.atlassian.net/wiki/spaces/DEVX/pages/5715984961/TNA+3.0+1Pager+2026.2Q) |
| Google Sheet ID | `1uOrXjUGchGw-BJRrhHvfsKMgx6LOXPQBu8k5-_Nf-bM` |
| Google Sheet 시트명 | `신규 연동 검토 (2026~)` |
| 페이지 제목 템플릿 | `1 pager {supplier}` |
```

---

### Task 3: Process Step 0 (인증 사전 체크) 추가

**Files:**
- Modify: `skills/integration-review/SKILL.md:101-120` (기존 Process > 0. 인증 검증)

- [ ] **Step 1: 기존 인증 검증(curl Basic Auth) 삭제 후 MCP 기반 체크로 교체**

기존 Step 0 (curl로 Confluence/Sheets 검증) 전체를 아래로 교체:

```markdown
### 0. 인증 사전 체크

**Atlassian MCP 확인:**
- `mcp__plugin_atlassian_atlassian__getAccessibleAtlassianResources` 호출
- 성공 시: Cloud ID를 저장하고 진행
- 실패 시:
  ```
  Atlassian 로그인이 필요합니다. `/login`으로 로그인해주세요.
  질문 수집은 먼저 진행할게요 — Confluence 게시 전에 다시 확인합니다.
  ```

**Google Sheets 확인:**
```bash
gcloud auth print-access-token > /dev/null 2>&1 && echo "OK" || echo "FAIL"
```
- 성공 시: 진행
- 실패 시:
  ```
  gcloud 인증이 필요합니다. `! gcloud auth login`을 실행해주세요.
  질문 수집은 먼저 진행할게요.
  ```

**핵심**: 인증 실패해도 차단하지 않음. 질문 수집(Step 1~4)은 인증 없이 진행. Confluence 게시(Step 6) 직전에 다시 확인.
```

---

### Task 4: Step 1 인트로 온보딩 강화

**Files:**
- Modify: `skills/integration-review/SKILL.md:122-135` (Step 1 인트로)

- [ ] **Step 1: 인트로 메시지를 온보딩 가이드로 교체**

기존 인트로를 아래로 교체:

```markdown
### 1. 인트로 & 공급사명 확인

인트로 메시지 출력:
```
안녕하세요! 신규 연동 1Pager 작성을 도와드릴게요.

이 스킬은 다음을 자동으로 처리합니다:
1. 대화형 질의응답으로 1Pager 문서 작성
2. Confluence에 자동 게시 (TNA 3.0 1Pager 하위)
3. 연동 검토 Google Sheet 업데이트

약 5~10분 정도 소요되며, 잘 모르는 항목은 "모르겠어요"라고 하셔도 됩니다.
그럼 시작할게요!
```

- `$ARGUMENTS`가 있으면 공급사명으로 사용, 바로 2단계로
- 없으면: "연동하려는 공급사 이름이 뭔가요?"
```

---

### Task 5: Confluence 게시 로직을 MCP 도구로 교체

**Files:**
- Modify: `skills/integration-review/SKILL.md:258-303` (Step 6 Confluence 게시)

- [ ] **Step 1: curl 기반 Confluence 게시를 MCP 도구로 전면 교체**

기존 Step 6 (curl로 Space ID 조회, 중복 확인, 페이지 생성) 전체를 아래로 교체:

```markdown
### 6. 사용자 확인 & Confluence 게시

**게시 전 반드시 사용자 확인을 받습니다.**

1. 생성된 문서를 사용자에게 보여주기
2. "이 내용으로 Confluence에 게시하고 연동 검토 시트를 업데이트할까요? 수정할 부분이 있으면 말씀해주세요." 확인
3. 수정 요청 시 → 해당 부분 수정 후 다시 확인
4. 승인 시 → Confluence 게시 + Google Sheet 업데이트 진행

**Confluence 게시 전 인증 재확인:**
- Step 0에서 인증을 건너뛰었다면, 이 시점에서 반드시 `getAccessibleAtlassianResources`로 확인
- 인증 안 되어있으면: `/login`으로 로그인 안내 후 대기

**Confluence 게시 (MCP 도구):**

1. **Cloud ID 조회** (Step 0에서 이미 조회했으면 재사용):
   - `mcp__plugin_atlassian_atlassian__getAccessibleAtlassianResources` 호출
   - `myrealtrip.atlassian.net`에 해당하는 cloud ID 추출

2. **Space ID 조회**:
   - `mcp__plugin_atlassian_atlassian__getConfluenceSpaces` 호출 (cloudId 전달)
   - Space Key `DEVX`에 해당하는 Space ID 추출

3. **중복 확인**:
   - `mcp__plugin_atlassian_atlassian__getPagesInConfluenceSpace` 호출 (cloudId, spaceId 전달)
   - `1 pager {공급사명}` 제목의 기존 페이지 검색
   - 존재 시: `{제목} (Update N)` 형태로 번호 부여

4. **페이지 생성** (항상 새 페이지, 기존 페이지 덮어쓰기 금지):
   - `mcp__plugin_atlassian_atlassian__createConfluencePage` 호출
   - Parameters: `cloudId`, `spaceId`, `parentId: "5715984961"`, `title`, `body`, `contentFormat: "markdown"`
   - body는 Step 5에서 생성한 1Pager 마크다운을 그대로 전달

5. 응답에서 페이지 URL 추출
```

---

### Task 6: Confluence Storage Format 섹션 삭제

**Files:**
- Modify: `skills/integration-review/SKILL.md:375-430` (Confluence Storage Format 참고)

- [ ] **Step 1: Confluence Storage Format 섹션 전체 삭제**

`contentFormat: "markdown"`를 사용하므로 XHTML 변환 템플릿이 더 이상 필요 없음.
아래 전체를 삭제:
- `## Confluence Storage Format 참고` (패널, 테이블, 정보/경고 박스, 상태 매크로, Heading, 구분선 템플릿)

---

### Task 7: Error Handling 업데이트

**Files:**
- Modify: `skills/integration-review/SKILL.md:436-463` (Error Handling)

- [ ] **Step 1: 에러 메시지를 MCP 기반으로 교체**

기존 에러 핸들링을 아래로 교체:

```markdown
## Error Handling

모든 에러 메시지는 한국어로 표시합니다:

- **Atlassian MCP 인증 실패**:
  ```
  🔐 Atlassian 인증에 실패했습니다.

  확인 사항:
  1. `/login` 명령어로 Atlassian에 로그인하세요
  2. 브라우저에서 인증을 완료하세요
  3. 인증 완료 후 다시 실행해주세요

  그래도 안 되면:
  1. `/plugin` 에서 Atlassian 플러그인 상태를 확인하세요
  2. 'Status: ✔ connected', 'Auth: ✔ authenticated'인지 확인하세요
  ```
- **Atlassian Cloud ID 없음**: "Atlassian Cloud ID를 찾을 수 없습니다. MCP 플러그인 설정을 확인해주세요."
- **Google Sheets 인증 실패**:
  ```
  🔐 Google Sheets 접근에 실패했습니다.

  확인 사항:
  1. `! gcloud auth login` 으로 재인증하세요
  2. 재인증 완료 후 이 스킬을 다시 실행해주세요
  ```
- **Confluence 게시 실패**: "Confluence 페이지 생성에 실패했습니다. MCP 플러그인 인증을 확인해주세요." + 재시도 안내
- **Google Sheet 업데이트 실패**: "Google Sheet 업데이트에 실패했습니다." + gcloud 재인증 안내
- **Confluence 게시 성공 but Sheet 실패**: Confluence URL은 출력하고, Sheet 수동 업데이트 안내
- **MCP 플러그인 미설치**: 설치 안내 (Prerequisites 참조)
```

---

### Task 8: Tools Required 업데이트

**Files:**
- Modify: `skills/integration-review/SKILL.md:464-470` (Tools Required)

- [ ] **Step 1: Tools Required를 MCP 기반으로 교체**

```markdown
## Tools Required

- Bash: curl (Google Sheets API), gcloud, python3 (JSON 파싱)
- Read: 참조 파일 읽기 (settlement-type.md)
- MCP Atlassian tools (required):
  - `mcp__plugin_atlassian_atlassian__getAccessibleAtlassianResources`: Cloud ID 조회
  - `mcp__plugin_atlassian_atlassian__getConfluenceSpaces`: Space Key → Space ID 변환
  - `mcp__plugin_atlassian_atlassian__getPagesInConfluenceSpace`: 동일 제목 페이지 중복 확인
  - `mcp__plugin_atlassian_atlassian__createConfluencePage`: 1Pager 페이지 생성
```

---

### Task 9: 커밋

- [ ] **Step 1: 변경 사항 커밋**

```bash
git add skills/integration-review/SKILL.md
git commit -m "feat(integration-review): switch to Atlassian MCP auth and enhance onboarding"
```

---

## Chunk 2: 버전 업 & 정리

### Task 10: plugin.json, marketplace.json 버전 업

**Files:**
- Modify: `.claude-plugin/plugin.json:2` (version)
- Modify: `.claude-plugin/marketplace.json:11,22` (version)

- [ ] **Step 1: plugin.json 버전 업**

`"version": "1.4.2"` → `"version": "1.5.0"`

- [ ] **Step 2: marketplace.json 버전 업**

plugins[0].version과 최상위 version 모두: `"1.4.2"` → `"1.5.0"`

- [ ] **Step 3: 커밋**

```bash
git add .claude-plugin/plugin.json .claude-plugin/marketplace.json
git commit -m "chore: bump version to 1.5.0"
```

---

### Task 11: integration-review.json 삭제 안내

- [ ] **Step 1: 기존 설정 파일 삭제 확인**

`~/.claude/integration-review.json`이 존재하면 삭제:
```bash
rm -f ~/.claude/integration-review.json
```

이 파일은 사용자 홈 디렉토리에 있으므로 git 추적 대상이 아님. 수동 삭제.

---

### Task 12: 최종 검증

- [ ] **Step 1: SKILL.md 구조 검증**

변경된 SKILL.md가 다음 구조를 따르는지 확인:
1. frontmatter (allowed-tools에 MCP 도구 포함)
2. Description
3. Prerequisites (MCP + gcloud)
4. 고정 설정 테이블
5. Process: Step 0 (인증 체크) → Step 1~4 (질문 수집) → Step 5 (문서 생성) → Step 6 (MCP로 Confluence 게시) → Step 7 (Google Sheet) → Step 8 (완료)
6. Error Handling (MCP 기반)
7. Tone & Style
8. Tools Required (MCP 도구 목록)

- [ ] **Step 2: curl 잔존 확인**

SKILL.md에서 Confluence 관련 curl 호출이 모두 제거되었는지 확인.
Google Sheets curl은 유지되어야 함.

- [ ] **Step 3: 최종 커밋 (필요 시)**

```bash
git add -A
git commit -m "docs: add spec and plan for integration-review enhancement"
```

---
name: integration-review
description: >
  신규 공급사 API 연동 검토를 위한 1Pager 문서를 대화형으로 작성하고 Confluence에 게시 + Google Sheet를 업데이트합니다.
  "1Pager 작성", "연동 검토 문서", "신규 공급사 연동", "integration review" 등을 요청할 때 사용합니다.
user-invocable: true
argument-hint: "[공급사명]"
allowed-tools:
  - "Bash(curl:*)"
  - "Bash(gcloud:*)"
  - "Bash(*googleapis.com*)"
  - "Bash(*gcloud auth*)"
  - Read
  - mcp__plugin_atlassian_atlassian__authenticate
  - mcp__plugin_atlassian_atlassian__getAccessibleAtlassianResources
  - mcp__plugin_atlassian_atlassian__getConfluenceSpaces
  - mcp__plugin_atlassian_atlassian__getPagesInConfluenceSpace
  - mcp__plugin_atlassian_atlassian__createConfluencePage
---

# 신규 연동 1Pager & Confluence 게시 스킬

## Description

마리트(MyRealTrip) 사업팀이 신규 공급사 API 연동을 검토할 때 사용하는 1Pager 문서를 대화형 질의응답으로 작성하고:
1. **Confluence에 자동 게시** (Atlassian MCP 플러그인)
2. **Google Sheet 업데이트** (gcloud + Sheets API)

**IMPORTANT**: 모든 사용자 커뮤니케이션은 한국어로 합니다.

**CRITICAL - 절대 하지 말 것:**
- `~/.claude/integration-review.json` 등 설정 파일을 읽거나 생성하지 않는다
- API Token 발급을 요청하지 않는다 (https://id.atlassian.com/manage-profile/security/api-tokens 링크 안내 금지)
- Basic Auth, email+apiToken 조합을 사용하지 않는다
- curl로 Confluence API를 호출하지 않는다
- 인증은 오직 Atlassian MCP 플러그인(`mcp__plugin_atlassian_atlassian__*`)을 통해서만 한다

## Prerequisites

**Atlassian MCP Plugin + Google Sheets (gcloud)**

1. **Atlassian**: MCP 플러그인이 설치되어 있어야 합니다. 미설치/미인증 시 스킬이 자동으로 인증 플로우를 시작합니다.
2. **Google Sheets**: `gcloud` CLI 설치 및 인증 완료 (`gcloud auth print-access-token`으로 토큰 발급 가능한 상태)

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

**모든 1Pager는 위 부모 페이지 하위에 강제 생성됩니다.**

## Process

### 0. 인증 사전 체크

고정값(domain, spaceKey, parentPageId, spreadsheetId, sheetName)은 스킬에서 직접 사용합니다.

**Atlassian MCP 확인:**
1. `mcp__plugin_atlassian_atlassian__getAccessibleAtlassianResources` 호출
2. 성공 시: `myrealtrip.atlassian.net`에 해당하는 Cloud ID를 저장하고 진행
3. 실패 시 (미인증 또는 미설치):
   - `mcp__plugin_atlassian_atlassian__authenticate` 를 자동 호출하여 인증 플로우 시작
   - 사용자에게 안내:
     ```
     Atlassian 인증이 필요합니다. 브라우저에서 로그인 창이 열립니다.
     로그인을 완료하면 자동으로 진행됩니다.
     질문 수집은 인증 없이도 먼저 진행할 수 있어요.
     ```
   - 인증 완료 후 `getAccessibleAtlassianResources`를 다시 호출하여 Cloud ID 확보
   - `authenticate` 도구 자체가 없으면 (플러그인 미설치):
     ```
     Atlassian MCP 플러그인이 설치되어 있지 않습니다.
     /plugin 에서 'atlassian@claude-plugins-official'을 설치해주세요.
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

### 2. 기본 정보 수집 (1단계)

아래 3가지를 자연스럽게 물어봅니다:

1. **공급사 이름**과 **카테고리** (숙소 / 투어·티켓 / 교통 / 기타)
2. **어떤 계기**로 이 연동을 검토하게 되었는지
3. **현재 이 카테고리 상품을 어떻게 공급**받고 있는지 (직접 입점, 다른 OTA 경유, 없음 등)

추가로 Google Sheet 컬럼에 필요한 정보도 자연스럽게 수집:
- **담당팀** (예: 일본, 동남아, 중부유럽 등)
- **파트너사 유형** (공급사 / 직연동 / 기타)
- **사업 담당자** (이름)

→ 답변을 받으면 자연스럽게 3단계로 넘어갑니다.

### 3. 심화 질문 (2단계)

아래 항목을 **2~3개씩 나눠서** 물어봅니다. 절대 한꺼번에 다 묻지 않습니다:

**문제 & 타이밍:**
- 현재 구조에서 가장 큰 문제는? (가격경쟁력 부족 / 셀렉션 부족 / 확정·취소 이슈 / 운영 리소스 과다 / 프로모션 대응 불가 등)
- 왜 "지금" 해야 하는지? (시즌성, 경쟁사 움직임, 수요 변화 등)

**변화 & 강점:**
- 연동하면 구체적으로 뭐가 달라지는지? (가격, 셀렉션, 확정속도, 운영방식 등)
- 이 공급사만의 차별점은? (독점 셀렉션, 가격구조, 지역 강점, 타 OTA 사례 등)

**임팩트:**
- 예상 월 거래액 또는 매출 규모 (대략적이어도 OK)
- 기대하는 정성적 효과 (브랜드 강화, 운영 효율 등)

**정산 관련:**
- 공급사가 제시하는 가격 구조는? (판매가 고정 + 수수료 vs 공급가(net price) 제공)
- 미리 충전 방식인지, 예약 후 주기적으로 정산하는 방식인지?
- 정산 통화 및 주기에 대해 논의된 내용이 있는지?
- MSP(최저판매가) 제한이 있는지?

> 사용자가 "잘 모르겠다" 또는 "아직 미정"이라고 하면 "확인 후 추가해도 됩니다"라고 안내하고 넘어갑니다.

### 4. 정산 유형 분류

수집된 정산 정보를 기반으로 `references/settlement-type.md`의 분류 로직에 따라 정산 유형을 판별합니다.

정보가 부족하면 추가 질문:
- **Q1**: 공급사가 제공하는 가격 구조는? (판매 수수료 vs 공급가)
- **Q2**: 정산 시점은? (밸런스 충전/선정산 vs 예약 후 정산/후정산)

분류 결과와 해당 유형의 체크리스트를 문서에 포함합니다.

### 5. 1Pager 문서 생성

정보가 충분히 모이면 Markdown 형식으로 1Pager를 생성합니다.

**문서 구조:**

```markdown
# 신규 연동 1Pager: {공급사명}

**한 줄 요약:** {설득력 있는 한 줄 요약}

**작성일:** {YYYY-MM-DD}  **카테고리:** {숙소/투어·티켓/교통/기타}

## 1. 이 연동을 검토하게 된 배경

### 1-1. 지금 어떤 문제가 있는가?
{현재 구조의 문제점}

### 1-2. 이 문제가 왜 지금 중요한가?
{타이밍의 중요성}

## 2. 이 공급사를 연동하면 무엇이 달라지는가?

### 2-1. 연동 전 vs 연동 후

| 항목 | 현재 (AS-IS) | 연동 후 (TO-BE) |
|------|-------------|----------------|
| ... | ... | ... |

### 2-2. 이 공급사만의 강점
{차별점}

## 3. 기대 비즈니스 임팩트

### 3-1. 정량적 임팩트
{예상 거래액, 매출 등}

### 3-2. 정성적 임팩트
{브랜드, 운영효율 등}

## 4. 정산 유형

**{유형N}** - {분류 요약} (예: 유형2 - 공급가(net price) 기반 / 선결제 방식)

### 체크리스트
{해당 유형의 체크리스트 - references/settlement-type.md 참조}
- 체크리스트는 `- [ ]` 가 아닌 `- 항목명` 형식으로 작성한다

{유형2/4일 경우: 개발팀 사전 공유 필요}

상세 내용은 Confluence 연동 가이드 정산 유형별 안내 참고
```

**한 줄 요약 톤 예시:**
- "최저가 대응이 불가능해 놓치고 있는 핵심 수요를 회복하기 위한 연동"
- "이미 검증된 수요가 있는 카테고리를 운영 리스크 없이 확장하기 위한 연동"
- "OTA 의존 구조에서 벗어나 마진과 프로모션 주도권을 확보하기 위한 연동"

**작성 원칙:**
- 읽는 사람은 연동 배경을 모르는 의사결정자
- 설득력 있는 문장으로 — "아, 이거 해야겠다"고 느끼게
- 데이터 + 정성적 스토리 둘 다 포함
- 사용자가 제공하지 않은 정보는 `[확인 필요]` 라벨로 표시
- 추측을 사실처럼 쓰지 않기

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
   - `myrealtrip.atlassian.net`에 해당하는 Cloud ID 추출

2. **Space ID 조회**:
   - `mcp__plugin_atlassian_atlassian__getConfluenceSpaces` 호출 (cloudId 전달)
   - Space Key `DEVX`에 해당하는 Space ID 추출

3. **페이지 제목 생성**: `1 pager {공급사명}`
   - 기존 시트의 Confluence 링크 패턴과 일관되게 유지

4. **중복 확인**:
   - `mcp__plugin_atlassian_atlassian__getPagesInConfluenceSpace` 호출 (cloudId, spaceId 전달)
   - `1 pager {공급사명}` 제목의 기존 페이지 검색
   - 존재 시: `{제목} (Update N)` 형태로 번호 부여

5. **페이지 생성** (항상 새 페이지, 기존 페이지 덮어쓰기 금지):
   - `mcp__plugin_atlassian_atlassian__createConfluencePage` 호출
   - Parameters: `cloudId`, `spaceId`, `parentId: "5715984961"`, `title`, `body`, `contentFormat: "markdown"`
   - body는 Step 5에서 생성한 1Pager 마크다운을 그대로 전달

   **마크다운 호환성 (CRITICAL - 반드시 준수):**
   - `>` blockquote 사용 금지 → 일반 **bold** 텍스트로 대체
   - `- [ ]` 체크박스 사용 금지 → `- 항목명` 일반 리스트로 대체
   - `---` 수평선 사용 금지 → 섹션 구분은 `##` 헤딩으로만
   - 문자열 내 이스케이프된 따옴표 `\"` 사용 금지 → 따옴표 제거하거나 다른 표현 사용
   - 이모지(⚠️ 등) 사용 금지
   - 첫 시도에서 에러 발생 시 내용을 간소화하지 말고, 위 규칙을 적용하여 재시도

6. 응답에서 페이지 URL 추출

### 7. Google Sheet 업데이트

Confluence 게시 성공 후, Google Sheet "신규 연동 검토 (2026~)" 시트를 업데이트합니다.

**시트 컬럼 매핑 (A~N):**

| 열 | 헤더 | 채울 값 |
|----|------|---------|
| A | (빈칸) | 빈칸 |
| B | 담당팀 | 수집된 담당팀 |
| C | 파트너사 | 공급사명 |
| D | 파트너사 유형 | 수집된 파트너사 유형 |
| E | 상품 카테고리 | 수집된 카테고리 |
| F | 기대효과 | 수집된 기대효과 요약 |
| G | 사업 검토 상태 | "검토중" |
| H | 비즈니스 임팩트 | Confluence 페이지 URL |
| I | 사업 담당자 | 수집된 담당자명 |
| J | 개발 문서 확인 | (빈칸) |
| K | 사업 홀드 사유 | (빈칸) |
| L | 개발 검토 상태 | (빈칸) |
| M | 담당자 | (빈칸) |
| N | 개발 홀드 사유 | (빈칸) |

**동작 로직:**

1. **기존 행 검색** — 파트너사명(C열)으로 기존 행이 있는지 확인:
   ```bash
   curl -s "https://sheets.googleapis.com/v4/spreadsheets/{spreadsheetId}/values/{sheetName}!C:C" \
     -H "Authorization: Bearer $(gcloud auth print-access-token)" \
     | python3 -c "
   import sys,json
   data = json.load(sys.stdin).get('values',[])
   for i,row in enumerate(data):
       if row and '{supplier}' in row[0]:
           print(f'FOUND:{i+1}')
           break
   else:
       print('NOT_FOUND')
   "
   ```

2. **기존 행이 있으면** → H열(비즈니스 임팩트)만 업데이트:
   ```bash
   curl -s -X PUT \
     "https://sheets.googleapis.com/v4/spreadsheets/{spreadsheetId}/values/{sheetName}!H{row}?valueInputOption=USER_ENTERED" \
     -H "Authorization: Bearer $(gcloud auth print-access-token)" \
     -H "Content-Type: application/json" \
     -d '{"values": [["{confluenceUrl}"]]}'
   ```

3. **기존 행이 없으면** → 새 행 추가 (append):
   ```bash
   curl -s -X POST \
     "https://sheets.googleapis.com/v4/spreadsheets/{spreadsheetId}/values/{sheetName}!A:N:append?valueInputOption=USER_ENTERED&insertDataOption=INSERT_ROWS" \
     -H "Authorization: Bearer $(gcloud auth print-access-token)" \
     -H "Content-Type: application/json" \
     -d '{"values": [["", "{담당팀}", "{공급사명}", "{파트너사유형}", "{카테고리}", "{기대효과}", "검토중", "{confluenceUrl}", "{담당자}", "", "", "", "", ""]]}'
   ```

### 8. 완료 안내

```
✅ 1Pager가 Confluence에 게시되고 연동 검토 시트가 업데이트되었습니다!

📄 Confluence: {페이지 URL}
📊 Google Sheet: {시트에서 업데이트/추가된 행 번호}
📋 공급사: {공급사명}
💰 정산 유형: {유형N} - {요약}
```

## Error Handling

모든 에러 메시지는 한국어로 표시합니다:

- **Atlassian MCP 인증 실패**:
  - 먼저 `mcp__plugin_atlassian_atlassian__authenticate`를 자동 호출하여 재인증 시도
  - 그래도 실패 시:
    ```
    🔐 Atlassian 인증에 실패했습니다.

    확인 사항:
    1. `/plugin` 에서 Atlassian 플러그인 상태를 확인하세요
    2. 'Status: ✔ connected', 'Auth: ✔ authenticated'인지 확인하세요
    3. 플러그인이 없으면 'atlassian@claude-plugins-official'을 설치하세요
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

## Tone & Style

- 친근하지만 전문적 (반말 안 씀, 과도한 존댓말도 안 씀)
- "~해주세요", "~알려주세요" 같은 요청형
- 사용자가 모르는 것은 괜찮다고 안심시켜줌
- 이모지는 적절히 사용하되 과하지 않게

## Tools Required

- Bash: curl (Google Sheets API), gcloud, python3 (JSON 파싱)
- Read: 참조 파일 읽기 (settlement-type.md)
- MCP Atlassian tools (required):
  - `mcp__plugin_atlassian_atlassian__getAccessibleAtlassianResources`: Cloud ID 조회
  - `mcp__plugin_atlassian_atlassian__getConfluenceSpaces`: Space Key → Space ID 변환
  - `mcp__plugin_atlassian_atlassian__getPagesInConfluenceSpace`: 동일 제목 페이지 중복 확인
  - `mcp__plugin_atlassian_atlassian__createConfluencePage`: 1Pager 페이지 생성

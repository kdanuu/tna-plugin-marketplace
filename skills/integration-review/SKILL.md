---
name: integration-review
description: >
  신규 공급사 API 연동 검토를 위한 1Pager 문서를 대화형으로 작성하고 Confluence에 게시 + Google Sheet를 업데이트합니다.
  "1Pager 작성", "연동 검토 문서", "신규 공급사 연동", "integration review" 등을 요청할 때 사용합니다.
user-invocable: true
argument-hint: "[공급사명]"
allowed-tools:
  - Bash
  - Read
  - Write
---

# 신규 연동 1Pager & Confluence 게시 스킬

## Description

마리트(MyRealTrip) 사업팀이 신규 공급사 API 연동을 검토할 때 사용하는 1Pager 문서를 대화형 질의응답으로 작성하고:
1. **Confluence에 자동 게시** (API Token + curl)
2. **Google Sheet 업데이트** (gcloud + Sheets API)

**IMPORTANT**: 모든 사용자 커뮤니케이션은 한국어로 합니다.

## Prerequisites

### Confluence (API Token)
- Atlassian API Token 필요 (https://id.atlassian.com/manage-profile/security/api-tokens 에서 발급)
- 설정 파일에 email + API Token 저장

### Google Sheets (gcloud)
- `gcloud` CLI 설치 및 인증 완료
- `gcloud auth print-access-token` 으로 토큰 발급 가능한 상태
- Google Sheets API 스코프 접근 가능

## Initial Setup (First-time Use Only)

### Step 0: 설정 확인

1. Read 도구로 `~/.claude/integration-review.json` 존재 여부 확인
2. 파일이 없으면 → Setup Process 진행
3. 파일이 있으면 → 인증 검증 후 대화 시작

### 고정 설정 (스킬에 하드코딩)

다음 값은 스킬에 고정되어 있으며 설정 파일에 저장하지 않습니다:

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

### Setup Process

질문은 하나씩, 사용자 응답을 기다린 후 다음으로 진행합니다.

1. **안내 & 인증 정보 요청**:
   ```
   신규 연동 1Pager를 Confluence에 게시하려면 Atlassian 인증이 필요합니다.
   (게시 위치: TNA 3.0 1Pager 2026.2Q 하위)

   👉 여기서 API Token을 발급하세요: https://id.atlassian.com/manage-profile/security/api-tokens

   발급 후 아래 두 가지를 알려주세요:
   1. Atlassian 이메일 주소
   2. 발급받은 API Token
   ```

2. **gcloud 인증 확인**:
   - `gcloud auth print-access-token` 실행하여 토큰 발급 가능 여부 확인
   - 실패 시: "gcloud 인증이 필요합니다. `! gcloud auth login` 을 실행해주세요."

5. **설정 파일 생성** (`~/.claude/integration-review.json`):
   ```json
   {
     "atlassianEmail": "user@myrealtrip.com",
     "atlassianApiToken": "ATATT3xFfGF0..."
   }
   ```
   - Confluence 경로, Google Sheet 정보는 설정 파일에 저장하지 않음 (스킬에 고정)

6. **완료 안내**:
   ```
   ✅ 설정이 ~/.claude/integration-review.json에 저장되었습니다.
   📄 1Pager 게시 위치: TNA 3.0 1Pager 2026.2Q 하위

   이제 /integration-review [공급사명]으로 1Pager를 작성할 수 있습니다.
   ```

### Configuration Fields (설정 파일)

- `atlassianEmail`: Atlassian 로그인 이메일
- `atlassianApiToken`: Atlassian API Token

## Process

### 0. 인증 검증

설정 파일에서 email, apiToken을 로드하고, 고정값(domain, spaceKey, parentPageId, spreadsheetId, sheetName)은 스킬에서 직접 사용합니다.

**Confluence 검증:**
```bash
curl -s -o /dev/null -w "%{http_code}" \
  "https://myrealtrip.atlassian.net/wiki/api/v2/spaces" \
  -H "Authorization: Basic $(echo -n '{email}:{apiToken}' | base64)"
```
- 200이면 정상, 401이면 토큰 만료/오류 안내

**Google Sheets 검증:**
```bash
curl -s -o /dev/null -w "%{http_code}" \
  "https://sheets.googleapis.com/v4/spreadsheets/1uOrXjUGchGw-BJRrhHvfsKMgx6LOXPQBu8k5-_Nf-bM?fields=spreadsheetId" \
  -H "Authorization: Bearer $(gcloud auth print-access-token)"
```
- 200이면 정상, 그 외 gcloud 재인증 안내

### 1. 인트로 & 공급사명 확인

인트로 메시지 출력:
```
안녕하세요! 신규 연동 1Pager 작성을 도와드릴게요 📝
몇 가지 질문에 답해주시면, Confluence에 게시하고 연동 검토 시트도 업데이트해드립니다.
잘 모르는 항목은 "모르겠어요"라고 하셔도 괜찮아요 — 나중에 채워넣을 수 있게 표시해둘게요.
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

> **한 줄 요약:** {설득력 있는 한 줄 요약}

**작성일:** {YYYY-MM-DD}
**카테고리:** {숙소/투어·티켓/교통/기타}

---

## 1. 이 연동을 검토하게 된 배경

### 1-1. 지금 어떤 문제가 있는가?
{현재 구조의 문제점}

### 1-2. 이 문제가 "왜 지금" 중요한가?
{타이밍의 중요성}

---

## 2. 이 공급사를 연동하면 무엇이 달라지는가?

### 2-1. 연동 전 vs 연동 후

| 항목 | 현재 (AS-IS) | 연동 후 (TO-BE) |
|------|-------------|----------------|
| ... | ... | ... |

### 2-2. 이 공급사만의 강점
{차별점}

---

## 3. 기대 비즈니스 임팩트

### 3-1. 정량적 임팩트
{예상 거래액, 매출 등}

### 3-2. 정성적 임팩트
{브랜드, 운영효율 등}

---

## 4. 정산 유형

> **{유형N}** — {분류 요약} (예: 유형2 — 공급가(net price) 기반 / 선결제 방식)

### 체크리스트
{해당 유형의 체크리스트 - references/settlement-type.md 참조}

⚠️ {유형2/4일 경우 개발팀 사전 공유 필요 경고}

*상세 내용은 Confluence > 연동 가이드 > 정산 유형별 안내 참고*
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

**Confluence 게시 (curl + API Token):**

1. 설정에서 domain, email, apiToken, spaceKey, parentPageId 로드
2. Space ID 조회:
   ```bash
   SPACE_ID=$(curl -s "https://{domain}/wiki/api/v2/spaces?keys={spaceKey}" \
     -H "Authorization: Basic $(echo -n '{email}:{apiToken}' | base64)" \
     | python3 -c "import sys,json; print(json.load(sys.stdin)['results'][0]['id'])")
   ```
3. 페이지 제목 생성: `1 pager {공급사명}`
   - 기존 시트의 Confluence 링크 패턴과 일관되게 유지
4. **중복 확인**: 동일 부모 하위에 같은 제목 페이지 존재 여부 확인
   ```bash
   curl -s "https://{domain}/wiki/api/v2/pages?space-id={spaceId}&title={encodedTitle}" \
     -H "Authorization: Basic $(echo -n '{email}:{apiToken}' | base64)"
   ```
   - 존재 시: `{제목} (Update N)` 형태로 번호 부여
5. 새 페이지 생성 (항상 새 페이지, 기존 페이지 덮어쓰기 금지):
   ```bash
   curl -s -X POST "https://{domain}/wiki/api/v2/pages" \
     -H "Authorization: Basic $(echo -n '{email}:{apiToken}' | base64)" \
     -H "Content-Type: application/json" \
     -d '{
       "spaceId": "{spaceId}",
       "parentId": "{parentPageId}",
       "title": "{pageTitle}",
       "body": {
         "representation": "storage",
         "value": "{confluenceStorageFormatBody}"
       },
       "status": "current"
     }'
   ```
   - **IMPORTANT**: Confluence REST API v2의 body는 `storage` 형식(XHTML)을 사용한다.
   - Markdown → Confluence Storage Format(XHTML) 변환이 필요하다.
   - 변환 시 `references/confluence-format.md`의 매크로 템플릿을 참고한다.
6. 응답에서 페이지 URL 추출: `_links.webui` 또는 `https://{domain}/wiki{webui}`

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

## Confluence Storage Format 참고

1Pager를 Confluence에 게시할 때는 Markdown을 Confluence Storage Format(XHTML)으로 변환해야 합니다.

**패널 (한 줄 요약용):**
```html
<ac:structured-macro ac:name="panel">
  <ac:parameter ac:name="borderStyle">solid</ac:parameter>
  <ac:parameter ac:name="title">한 줄 요약</ac:parameter>
  <ac:parameter ac:name="borderColor">#4C9AFF</ac:parameter>
  <ac:parameter ac:name="titleBGColor">#DEEBFF</ac:parameter>
  <ac:parameter ac:name="bgColor">#F4F5F7</ac:parameter>
  <ac:rich-text-body>
    <p>요약 내용</p>
  </ac:rich-text-body>
</ac:structured-macro>
```

**테이블 (AS-IS / TO-BE):**
```html
<table data-layout="default">
  <colgroup><col style="width: 200px;" /><col style="width: 300px;" /><col style="width: 300px;" /></colgroup>
  <tbody>
    <tr>
      <th><p>항목</p></th>
      <th><p>현재 (AS-IS)</p></th>
      <th><p>연동 후 (TO-BE)</p></th>
    </tr>
    <tr>
      <td><p>내용</p></td>
      <td><p>내용</p></td>
      <td><p>내용</p></td>
    </tr>
  </tbody>
</table>
```

**정보/경고 박스:**
```html
<ac:structured-macro ac:name="info"><ac:rich-text-body><p>내용</p></ac:rich-text-body></ac:structured-macro>
<ac:structured-macro ac:name="warning"><ac:rich-text-body><p>내용</p></ac:rich-text-body></ac:structured-macro>
```

**미확인 항목 표시:**
```html
<ac:structured-macro ac:name="status">
  <ac:parameter ac:name="title">확인 필요</ac:parameter>
  <ac:parameter ac:name="colour">Red</ac:parameter>
</ac:structured-macro>
```

**Heading:**
```html
<h2>제목</h2>
<h3>소제목</h3>
```

**구분선:**
```html
<hr />
```

## Error Handling

모든 에러 메시지는 한국어로 표시합니다:

- **Confluence 인증 실패 (401)**:
  ```
  🔐 Confluence API 인증에 실패했습니다.

  확인 사항:
  1. ~/.claude/integration-review.json의 atlassianEmail이 정확한지 확인하세요
  2. API Token이 만료되지 않았는지 확인하세요
  3. 새 토큰 발급: https://id.atlassian.com/manage-profile/security/api-tokens
  4. 토큰 갱신 후 다시 실행해주세요
  ```
- **Google Sheets 인증 실패**:
  ```
  🔐 Google Sheets 접근에 실패했습니다.

  확인 사항:
  1. `! gcloud auth login` 으로 재인증하세요
  2. 재인증 완료 후 이 스킬을 다시 실행해주세요
  ```
- **Confluence 게시 실패**: "Confluence 페이지 생성에 실패했습니다. API 응답: {error}" + 재시도 안내
- **Google Sheet 업데이트 실패**: "Google Sheet 업데이트에 실패했습니다." + gcloud 재인증 안내
- **설정 파일 오류**: "설정 파일을 확인해주세요: ~/.claude/integration-review.json"
- **Confluence 게시 성공 but Sheet 실패**: Confluence URL은 출력하고, Sheet 수동 업데이트 안내

## Tone & Style

- 친근하지만 전문적 (반말 안 씀, 과도한 존댓말도 안 씀)
- "~해주세요", "~알려주세요" 같은 요청형
- 사용자가 모르는 것은 괜찮다고 안심시켜줌
- 이모지는 적절히 사용하되 과하지 않게

## Tools Required

- Bash: curl (Confluence API, Google Sheets API), gcloud, python3 (JSON 파싱)
- Read: 설정 파일 읽기
- Write: 설정 파일 생성

---
name: daily-digest
description: >
  하루 회의록을 통합 분석하여 슬랙 AgentDeck 봇(상세 정리본)과 poke 문자(비서 스타일 요약)로 전송.
  caret MCP와 Gemini 회의 요약(Google Docs)에서 회의록을 병렬 조회하고, 참석자·요약·액션 아이템을
  분석하여 두 가지 포맷으로 전달한다.
  트리거: /daily-digest, "일일 회의 정리", "오늘 회의 요약", "daily digest",
  "데일리 다이제스트", "회의록 분석", "오늘 미팅 정리"
  플래그: --auto (확인 없이 자동 전송, coworks 스케줄용), --preview (전송 전 미리보기 강제)
user-invocable: true
argument-hint: "[--auto] [--preview]"
allowed-tools: Read, Write, Edit, Bash, Agent
---

# Daily Digest

하루 회의록을 통합 분석하여 슬랙 AgentDeck 봇과 poke 문자로 전송한다.

**IMPORTANT**: 모든 사용자 커뮤니케이션은 한국어로 한다.

## 플래그

| 플래그 | 설명 |
|--------|------|
| (없음) | 기본 모드. 확인 없이 바로 전송 |
| `--preview` | 항상 미리보기 + 전송 확인 |
| `--auto` | 모든 확인 생략, 자동 전송 (coworks 스케줄용) |

`--auto`와 `--preview`는 상호 배타적. 동시 지정 시 `--preview`가 우선한다.

플래그 파싱: 인자 문자열에서 `--auto`, `--preview` 키워드를 단순 매칭.

---

## Step 0: 설정 체크

1. Read 도구로 `~/.claude/daily-digest.json` 읽기 시도
2. 분기:
   - **파일 없음** → Step 0a (온보딩) 진행
   - **JSON 파싱 실패 또는 `config_version` 필드 없음** →
     "설정 파일이 손상된 것 같습니다. 재설정할까요?" 확인
     - Y → Step 0a 진행
     - N → 종료
   - **정상** → `sources`와 `channels`에서 `enabled: true`인 항목을 안내하고 바로 Step 1 진행:
     ```
     이전 설정을 확인했습니다.
     회의록 소스: caret ✅, Gemini 요약 ✅
     전송 채널: 슬랙 ✅, poke ❌ (미설정)
     바로 회의록 조회를 시작합니다.
     ```
   - **슬랙 활성화인데 `AGENTDECK_BOT_TOKEN` 환경변수 없음** →
     "AgentDeck 봇 토큰이 설정되지 않았습니다." 안내 후
     Step 0a Phase 2의 "슬랙 선택 시" 1단계(토큰 입력 → settings.local.json 등록)를 실행.
     등록 확인 후 Step 1로 진행.

---

## Step 0a: 온보딩

### Phase 1: 회의록 소스 선택 (최소 1개 필수)

사용자에게 질문 (AskUserQuestion 사용하지 않고 대화로 진행):

```
어떤 회의록 소스를 연동하시겠어요?
[1] caret (AI 회의 어시스턴트)
[2] Gemini 회의 요약 (Google Meet에서 Gemini가 생성한 요약본, Google Docs에 저장됨)
[3] 둘 다
```

#### caret 선택 시:

1. **바로 검증 시도**: caret MCP 도구 `caret_list_notes`를 직접 호출하여 회의록 1건 조회 시도.
   - **성공** → "caret 연동 확인 완료!" 안내 후 다음 단계
   - **도구를 찾을 수 없음** → `~/.mcp.json`에 `caret` 항목이 있는지 Read 도구로 확인
     - **있음** → "caret MCP가 등록되어 있지만 현재 세션에서 로드되지 않았습니다. Claude Code 재시작 후 다시 시도해주세요." 안내
     - **없음** → 2단계(등록) 진행
2. API 키 발급 및 MCP 등록 (미등록 시에만):
   ```
   caret API 키가 필요합니다.

   준비 방법:
   1. caret PC 앱 설치: https://caret.so
   2. 앱 내 설정에서 API 키 생성 (참고: https://docs.caret.so/api-reference/introduction)

   API 키를 입력해주세요:
   ```
   - API 키 입력받은 후 Read 도구로 `~/.mcp.json` 읽기
   - `mcpServers`에 `caret` 항목 추가 (Edit 도구 사용):
     ```json
     "caret": {
       "type": "http",
       "url": "https://api-v2.caret.so/mcp",
       "headers": {
         "Authorization": "Bearer {입력한 API 키}"
       }
     }
     ```
   - "caret MCP를 등록했습니다. Claude Code 재시작 후 활성화됩니다." 안내

#### Gemini 회의 요약 선택 시:

Gemini 회의 요약은 Google Meet에서 Gemini가 생성한 요약본으로, Google Docs에 자동 저장된다.
`gcloud` CLI + Google Drive REST API로 직접 조회한다 (MCP 서버 불필요).

> 모든 명령은 Claude가 즉시 자동 실행한다. 미설치/미인증 발견 시 묻지 않고 바로 설치/인증한다.
> 사용자는 브라우저 Google 로그인만 하면 된다. "설치할까요?" 같은 확인 질문 금지.

아래 단계를 순서대로 실행한다. 각 단계에서 이미 완료된 상태면 즉시 다음 단계로 넘어간다. 중간에 멈추거나 확인을 요청하지 않는다.

1. **gcloud 설치 확인 → 미설치 시 자동 설치**:
   ```bash
   which gcloud 2>&1 || echo "NOT_INSTALLED"
   ```
   - `NOT_INSTALLED` 출력 시, 묻지 않고 바로:
     ```bash
     brew install --cask google-cloud-sdk 2>&1
     ```
   - 설치 후 PATH 반영:
     ```bash
     source "$(brew --prefix)/share/google-cloud-sdk/path.zsh.inc" 2>/dev/null && which gcloud
     ```
   - brew 자체가 없으면 brew 먼저 설치 후 위 반복

2. **인증 확인 → 미인증 시 자동 로그인**:
   ```bash
   gcloud auth list 2>&1
   ```
   - `No credentialed accounts` 시, "Google 로그인 창이 열립니다. 브라우저에서 로그인해 주세요!" 안내 후 바로:
     ```bash
     gcloud auth login --enable-gdrive-access --launch-browser 2>&1
     ```
   - 로그인 완료 대기 후 다음 단계로 자동 진행

3. **Drive API 검증**:
   ```bash
   curl -s -G -H "Authorization: Bearer $(gcloud auth print-access-token)" \
     --data-urlencode "q=name contains 'Gemini가 작성한 회의록'" \
     --data-urlencode "fields=files(id,name,modifiedTime)" \
     --data-urlencode "orderBy=modifiedTime desc" \
     --data-urlencode "pageSize=3" \
     "https://www.googleapis.com/drive/v3/files"
   ```
   - 성공 → 다음 단계
   - 403 `insufficientPermissions` → `gcloud auth login --enable-gdrive-access --launch-browser`로 재인증 후 재시도
   - 실패 → 에러 안내 + "건너뛰고 나중에 재시도할까요?"

**핵심**: 설치 → 인증 → 검증을 끊김 없이 한 흐름으로 실행한다. 각 단계 사이에 불필요한 상태 확인이나 경로 탐색을 하지 않는다.

### Phase 2: 전송 채널 설정 (최소 1개 필수)

```
어디로 리포트를 받으시겠어요?
[1] 슬랙 AgentDeck 봇 (회의별 상세 정리본, AgentDeck 봇이 DM으로 전송)
[2] poke 문자 (비서 스타일 요약)
[3] 둘 다
```

#### 슬랙 선택 시:

슬랙은 AgentDeck 전용 봇의 Token(환경변수 `AGENTDECK_BOT_TOKEN`)으로 REST API를 직접 호출하여 DM을 전송한다 (MCP 서버 불필요).

1. **환경변수 확인**: Bash로 `echo $AGENTDECK_BOT_TOKEN` 실행
   - **값 있음** → 바로 2단계로
   - **값 없음** → Bot Token 안내 및 자동 등록:
     ```
     AgentDeck 봇 토큰이 설정되지 않았습니다.
     AgentDeck 앱 담당자에게 Bot Token(xoxb-...)을 받아서 알려주세요.

     필요 Bot Token Scopes:
     • chat:write (메시지 전송)
     • users:read.email (이메일로 유저 조회)
     • im:write (DM 채널 열기)

     Bot Token을 입력해주세요:
     ```
     사용자가 토큰을 입력하면 `.claude/settings.local.json`의 `env` 필드에 자동 등록:
     - Read 도구로 `.claude/settings.local.json` 읽기 (없으면 `{}` 기본값)
     - Edit 도구로 `env` 객체에 `"AGENTDECK_BOT_TOKEN": "{입력한 토큰}"` 추가 (기존 env 필드가 있으면 병합)
     - 등록 즉시 현재 세션에서 `$AGENTDECK_BOT_TOKEN` 사용 가능
     - `echo $AGENTDECK_BOT_TOKEN`으로 확인. 값이 나오면 다음 단계로.
2. **이메일로 User ID 자동 조회**:
   - "슬랙에 등록된 이메일 주소를 입력해주세요:" 로 이메일 입력받기
   - Bash로 `users.lookupByEmail` API 호출:
     ```bash
     curl -s -G 'https://slack.com/api/users.lookupByEmail' \
       -H "Authorization: Bearer $AGENTDECK_BOT_TOKEN" \
       --data-urlencode "email={입력한 이메일}"
     ```
   - 응답에서 `user.id` 추출 → "슬랙 User ID: {user_id} 확인되었습니다." 안내
   - 실패 시 (`ok: false`): "이메일을 찾을 수 없습니다. 슬랙에 등록된 이메일인지 확인해주세요." → 재입력
3. **검증**: Bash로 테스트 DM 전송
   ```bash
   curl -s -X POST 'https://slack.com/api/chat.postMessage' \
     -H "Authorization: Bearer $AGENTDECK_BOT_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"channel": "{USER_ID}", "text": "daily-digest 테스트 메시지입니다."}'
   ```
   - 성공 (`ok: true`) → "테스트 메시지를 보냈습니다. 받으셨나요?"
   - 실패 → 에러 안내 + 재시도

#### poke 선택 시:

poke는 MCP 서버가 아닌 REST API로 메시지를 전송한다.
API 엔드포인트: `POST https://poke.com/api/v1/inbound-sms/webhook`

1. **이미 등록 여부 확인**: Read 도구로 `~/.claude/daily-digest.json` 읽고 `channels.poke.api_key`가 있는지 확인
   - **이미 등록됨** → "poke API 키가 이미 설정되어 있습니다." 안내 후 바로 2단계로
   - **미등록** → API 키 발급 안내:
     ```
     poke API 키가 필요합니다.

     발급 방법:
     1. poke 앱 가입/로그인: https://poke.com
     2. 설정 > Advanced로 이동: https://poke.com/settings/advanced
     3. "Add API Key" 클릭하여 API 키 생성

     API 키를 입력해주세요:
     ```
   - API 키는 설정 파일(`~/.claude/daily-digest.json`)의 `channels.poke.api_key`에 저장
2. 검증: Bash로 테스트 메시지 전송
   ```bash
   curl -s -X POST 'https://poke.com/api/v1/inbound-sms/webhook' \
     -H 'Authorization: Bearer {API_KEY}' \
     -H 'Content-Type: application/json' \
     -d '{"message": "daily-digest 테스트 메시지입니다."}'
   ```
   - 성공 → "테스트 메시지를 보냈습니다. 받으셨나요?"
   - 실패 → 에러 안내 + 재시도

### Phase 3: 설정 저장

설정은 이 Phase에서만 원자적으로 저장한다 (중간 저장 없음).

Write 도구로 `~/.claude/daily-digest.json` 생성:

```json
{
  "config_version": "1.0",
  "created_at": "{현재 ISO8601 타임스탬프}",
  "sources": {
    "caret": {
      "enabled": true/false
    },
    "gemini": {
      "enabled": true/false
    }
  },
  "channels": {
    "slack": {
      "enabled": true/false,
      "user_id": "{슬랙 User ID}"
    },
    "poke": {
      "enabled": true/false,
      "api_key": "{poke API 키}"
    }
  }
}
```

저장 후 Bash로 `chmod 0600 ~/.claude/daily-digest.json` 실행.

MCP 설정이 추가된 경우 "MCP 설정이 등록되었습니다. 새 MCP 서버를 활성화하려면 Claude Code를 재시작해야 합니다. 지금 온보딩 설정을 먼저 완료하고, 재시작 후 `/daily-digest`를 다시 실행하면 검증을 진행합니다." 안내.
설정 파일에 MCP 서버 이름은 빈 값으로 저장하고, 다음 실행 시 자동 감지하여 업데이트한다.

최종 출력:
```
✅ 설정 완료!
회의록 소스: {활성화된 소스 목록}
전송 채널: {활성화된 채널 목록}
이제 바로 회의록 분석을 시작하겠습니다.
```

온보딩 완료 후 자동으로 Step 1 (회의록 조회)로 진행한다. 사용자가 별도 명령을 입력할 필요 없다.

### 온보딩 실패 처리

- 소스 0개 → "최소 1개의 회의록 소스가 필요합니다" → Phase 1로 복귀
- 채널 0개 → "최소 1개의 전송 채널이 필요합니다" → Phase 2로 복귀

---

## Step 1: 회의록 조회

설정 파일에서 `enabled: true`인 소스만 조회한다.

활성화된 소스가 2개면 **병렬**로 조회 (Agent 도구로 서브에이전트 활용 가능):

- **caret**: caret MCP 도구 `caret_list_notes`를 호출하여 오늘 날짜 기준 회의록 전체 조회
  - 개별 노트 조회 시 `caret_get_note`의 파라미터명은 **`id`** (NOT `noteId`)
  - 노트가 너무 길어 토큰 초과 에러 발생 시 (파일로 저장됨): 해당 파일을 Read 도구로 읽어서 내용을 파싱한 뒤, Bash로 즉시 삭제한다 (`rm -f {파일경로}`)
- **Gemini 요약**: Bash로 Google Drive REST API 직접 호출하여 오늘의 Gemini 회의록 검색.
  ```bash
  curl -s -G -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    --data-urlencode "q=name contains 'Gemini가 작성한 회의록' and modifiedTime > '$(date -u +%Y-%m-%dT00:00:00Z)'" \
    --data-urlencode "fields=files(id,name,modifiedTime)" \
    --data-urlencode "orderBy=modifiedTime desc" \
    --data-urlencode "pageSize=20" \
    "https://www.googleapis.com/drive/v3/files"
  ```
  - 파일명 패턴: `{회의명} - {YYYY/MM/DD} {HH:MM} KST - Gemini가 작성한 회의록`
    - 예: `[해외렌트카] Sync Meeting - 2026/03/19 16:00 KST - Gemini가 작성한 회의록`
  - 검색된 문서의 본문 조회:
    ```bash
    curl -s -H "Authorization: Bearer $(gcloud auth print-access-token)" \
      "https://docs.googleapis.com/v1/documents/{문서ID}"
    ```
  - 본문에서 요약, 액션 아이템, 참석자 정보 추출
  - 파일명에서 회의명과 시간을 파싱하여 caret 데이터와 중복 매칭에 활용

### 중복 회의 처리 (두 소스 모두 활성화된 경우)

양쪽 소스의 회의를 **시간 + 제목**으로 매칭하여 중복 여부 판단:
- **caret에만 있는 회의** → caret 데이터로 분석
- **Gemini에만 있는 회의** → Gemini 요약 데이터로 분석
- **양쪽 모두 있는 회의** → 두 데이터를 교차 분석하여 더 풍부한 요약 생성 (caret의 상세 기록 + Gemini의 AI 요약을 결합)

### 회의 0건 처리

양쪽 소스 모두 결과가 0건이면:
```
오늘은 회의가 없습니다 🎉
```
출력 후 종료. Step 2 이후로 진행하지 않는다.

---

## Step 1.5: 미리보기 (조건부)

**활성화 조건**: `--preview` 플래그만
**비활성화 조건**: 기본 모드, `--auto` 플래그

조회된 회의 목록을 출력:
```
📋 조회된 회의 {N}건:
1. {제목} ({시간}, {소스})
2. {제목} ({시간}, {소스})
...

이 내용으로 리포트를 생성하고 전송할까요?
```

- Y → Step 2로
- N → 종료

---

## Step 2: 통합 분석 및 포맷 생성

1. 두 소스의 회의록을 **시간순**으로 통합
2. 회의록 총량이 큰 경우 (>50K 토큰 추정): 회의별로 먼저 요약 후 통합 (2단계 분석)
3. 활성화된 채널에 맞는 포맷만 생성:
   - 슬랙 활성화 시 → `references/slack-format.md` 참조하여 맥락이 풍부한 정리본 생성
   - poke 활성화 시 → `references/poke-format.md` 참조하여 비서 스타일 요약 생성
4. 두 포맷은 1회 호출로 동시 생성 (프롬프트에서 `## 슬랙 리포트`와 `## poke 요약`으로 구분)

---

## Step 2.5: 전송 전 확인 (조건부)

**활성화 조건**: `--preview` 플래그만
**비활성화 조건**: 기본 모드, `--auto` 플래그

생성된 리포트 미리보기 출력 후:
```
전송할까요?
```

- Y → Step 3로
- N → "수정할 부분을 알려주세요" → 수정 반영 → Step 2 재실행

---

## Step 3: 전송

활성화된 채널만 **병렬** 전송:

### 슬랙 AgentDeck 봇 전송

Bash로 Slack Web API를 직접 호출하여 DM을 전송한다.
Bot Token은 환경변수 `$AGENTDECK_BOT_TOKEN` 사용.
Slack mrkdwn 문법을 사용하며, `*bold*`/`_italic_` 대신 backtick(`` ` ``)으로 강조한다.

전송 순서 (회의별로 개별 DM 전송):
1. 각 회의마다 Bash로 `chat.postMessage` 호출하여 회의 제목 DM 전송 → 응답에서 `ts` 확보
   ```bash
   curl -s -X POST 'https://slack.com/api/chat.postMessage' \
     -H "Authorization: Bearer $AGENTDECK_BOT_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"channel": "{USER_ID}", "text": "{회의 제목 메시지}"}'
   ```
2. 확보된 `ts`를 `thread_ts`로 사용하여 해당 회의의 상세 정리본을 **스레드 답글**로 전송
   ```bash
   curl -s -X POST 'https://slack.com/api/chat.postMessage' \
     -H "Authorization: Bearer $AGENTDECK_BOT_TOKEN" \
     -H 'Content-Type: application/json' \
     -d '{"channel": "{USER_ID}", "text": "{상세 정리본}", "thread_ts": "{ts}"}'
   ```
   - 스레드 본문 5,000자 초과 시 여러 스레드 답글로 분할
3. 모든 회의 전송 후, 마지막에 종합 요약 DM을 별도로 1건 전송
4. 본문은 `references/slack-format.md` 참조
5. 전송 완료 후 Slack 메시지 링크를 사용자에게 안내

인증 에러 시 (`invalid_auth` 또는 `token_revoked`): "슬랙 Bot Token이 만료되었습니다. https://api.slack.com/apps 에서 Token을 재발급하고 `/digest-setup`으로 업데이트해주세요." 한 줄만 안내.

### poke 전송

Bash로 poke REST API 호출:
```bash
curl -s -X POST 'https://poke.com/api/v1/inbound-sms/webhook' \
  -H 'Authorization: Bearer {API_KEY}' \
  -H 'Content-Type: application/json' \
  -d '{"message": "{poke 요약 메시지}"}'
```
- API 키는 설정 파일의 `channels.poke.api_key` 사용
- 본문은 `references/poke-format.md` 참조

### 전송 결과

```
✅ 슬랙 전송 완료 (스레드 링크: {url})
✅ poke 전송 완료
```

일부 채널 전송 실패 시:
```
✅ 슬랙 전송 완료
❌ poke 전송 실패: {에러 메시지}
재시도하시겠어요?
```

---

## NEVER

- **슬랙 MCP 도구(`mcp__slack__*`, `slack_send_message` 등)를 절대 사용하지 않는다. 슬랙 전송은 반드시 Bash의 `curl`로 Slack Web API를 직접 호출해야 한다.**
- 회의록 원본을 외부 서비스에 저장하지 않는다
- 회의록 내용을 로컬 파일이나 캐시에 저장하지 않는다
- API 키를 로그, 콘솔 출력, 에러 메시지에 노출하지 않는다
- 온보딩 완료 전에 본 워크플로우를 실행하지 않는다
- 사용자 확인 없이 외부 채널로 메시지를 전송하지 않는다 (`--auto` 플래그 제외)
- 다른 사용자에게 슬랙 메시지를 전송하지 않는다
- 과거 날짜의 회의록을 조회하지 않는다 (당일 회의록만 대상)

---

## Error Handling

| 상황 | 대응 |
|------|------|
| MCP 서버 연결 실패 | 서버 이름 + 에러 안내. "MCP 설정을 확인해주세요" |
| 인증 만료 (401/403) | caret: 새 API 키 입력받아 `~/.mcp.json` 자동 업데이트. Gemini: `gcloud auth login --enable-gdrive-access` 자동 실행하여 재인증 |
| 설정 파일 없음/손상 | 자동으로 온보딩 시작. 손상 시 "설정을 재구성하겠습니다." 안내 후 진행 |
| 일부 소스 조회 실패 | "caret 조회에 실패했습니다. Gemini 요약 결과만으로 진행할까요?" 확인 |
| 일부 채널 전송 실패 | 성공한 채널은 유지, 실패 채널 재시도 여부 확인 |

---
name: daily-digest
description: >
  하루 회의록을 통합 분석하여 슬랙 DM(상세 리포트)과 poke 문자(비서 스타일 요약)로 전송.
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

하루 회의록을 통합 분석하여 슬랙 DM과 poke 문자로 전송한다.

**IMPORTANT**: 모든 사용자 커뮤니케이션은 한국어로 한다.

## 플래그

| 플래그 | 설명 |
|--------|------|
| (없음) | 기본 모드. 첫 실행 시 미리보기, 전송 전 확인 |
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
   - **정상** → `sources`와 `channels`에서 `enabled: true`인 항목 확인
     - MCP 서버 이름이 빈 값인 소스/채널이 있으면 (이전 온보딩에서 재시작 필요했던 경우):
       해당 MCP 도구를 자동 감지하고 설정 파일 업데이트 후 상태 안내:
       ```
       이전 설정을 확인했습니다.
       회의록 소스: caret ✅, Gemini 요약 ✅
       전송 채널: 슬랙 ✅, poke ❌ (미설정)
       ```
     - 모든 설정이 완료 상태 → Step 1 진행

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

1. **이미 등록 여부 확인**: Read 도구로 `~/.mcp.json` 읽고 `caret` 항목이 있는지 확인
   - **이미 등록됨** → "caret MCP가 이미 등록되어 있습니다." 안내 후 바로 3단계(감지)로
   - **미등록** → 2단계 진행
2. API 키 발급 및 MCP 등록:
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
   - 등록 완료 메시지 출력 (모든 MCP 등록이 끝난 후 한 번에 재시작 안내)
3. MCP 서버 이름 감지: 사용 가능한 MCP 도구 목록에서 `mcp__*caret*__` 패턴으로 실제 서버 이름을 감지. 아직 감지되지 않으면 온보딩 마지막에 재시작 안내.
4. 검증: 감지된 caret MCP 도구로 회의록 1건 조회 시도
   - 성공 → 다음 단계
   - 실패 → 에러 원인 안내 + "건너뛰고 나중에 재시도할까요?"

#### Gemini 회의 요약 선택 시:

Gemini 회의 요약은 Google Meet에서 Gemini가 생성한 요약본으로, Google Docs에 자동 저장된다.
이를 조회하려면 Google Drive/Docs MCP 서버가 필요하다.

1. **이미 등록 여부 확인**: Read 도구로 `~/.mcp.json` 읽고 Google Drive 관련 MCP 항목이 있는지 확인
   - **이미 등록됨** → "Google Drive MCP가 이미 등록되어 있습니다." 안내 후 바로 3단계(감지)로
   - **미등록** → 2단계 진행
2. Google Drive MCP 설정:
   - 사전 요구사항 안내 (각 단계에 링크 포함):
     ```
     Gemini 회의 요약을 가져오려면 Google Drive MCP 서버가 필요합니다.
     (Gemini가 생성한 회의 요약은 Google Docs에 자동 저장됩니다)

     사전 준비:
     1. Google Cloud Console에서 프로젝트 생성/선택: https://console.cloud.google.com/projectcreate
     2. Google Drive API 활성화: https://console.cloud.google.com/apis/library/drive.googleapis.com
     3. Google Docs API 활성화: https://console.cloud.google.com/apis/library/docs.googleapis.com
     4. OAuth 동의 화면 설정: https://console.cloud.google.com/apis/credentials/consent
     5. OAuth 클라이언트 ID 생성 (Desktop App): https://console.cloud.google.com/apis/credentials/oauthclient

     위 준비가 되셨으면 알려주세요.
     ```
   - 사용자가 준비 완료를 확인하면, "Google 계정 인증을 진행합니다. 브라우저가 열립니다." 안내 후 **반드시 Bash 도구로 직접** `gcloud auth login --brief` 실행. 절대 사용자에게 직접 실행하라고 안내하지 않는다.
   - 인증 완료 후 Read 도구로 `~/.mcp.json` 읽고 Edit 도구로 `mcpServers`에 Google Drive MCP 항목 자동 추가
   - 등록 완료 메시지 출력 (모든 MCP 등록이 끝난 후 한 번에 재시작 안내)
3. MCP 서버 이름 감지: `mcp__*drive*__` 또는 `mcp__*google*__` 패턴으로 감지. 아직 감지되지 않으면 온보딩 마지막에 재시작 안내.
4. 검증: 감지된 Drive MCP 도구로 "Gemini" 또는 "회의 요약" 키워드가 포함된 최근 Google Docs 1건 검색 시도
   - 성공 → 다음 단계
   - 실패 → 에러 원인 안내 + "건너뛰고 나중에 재시도할까요?"

### Phase 2: 전송 채널 설정 (최소 1개 필수)

```
어디로 리포트를 받으시겠어요?
[1] 슬랙 DM (상세 리포트)
[2] poke 문자 (비서 스타일 요약)
[3] 둘 다
```

#### 슬랙 선택 시:

1. 슬랙 MCP 연결 확인: `mcp__*slack*__` 패턴으로 도구 감지
2. 본인 슬랙 User ID 확인:
   - 슬랙 MCP의 auth 관련 도구로 자동 감지 시도
   - 실패 시: "슬랙 앱 > 프로필 > ⋯ > Member ID 복사 후 입력해주세요"
3. 검증: 테스트 DM 전송
   - "테스트 메시지를 보냈습니다. 받으셨나요?"
   - 실패 → 에러 안내 + 재시도

#### poke 선택 시:

1. poke MCP 연결 확인: `mcp__*poke*__` 패턴으로 도구 감지
2. 검증: 테스트 문자 전송
   - "테스트 문자를 보냈습니다. 받으셨나요?"
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
      "enabled": true/false,
      "mcp_server_name": "{감지된 서버 이름}"
    },
    "gemini": {
      "enabled": true/false,
      "mcp_server_name": "{감지된 서버 이름}"
    }
  },
  "channels": {
    "slack": {
      "enabled": true/false,
      "user_id": "{슬랙 User ID}",
      "mcp_server_name": "{감지된 서버 이름}"
    },
    "poke": {
      "enabled": true/false,
      "mcp_server_name": "{감지된 서버 이름}"
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

활성화된 소스가 2개면 **병렬**로 조회 (Agent 도구로 서브에이전트 활용):

- **caret**: `mcp__{caret_server_name}__` 접두사 도구로 오늘 날짜 기준 회의록 전체 조회
- **Gemini 요약**: `mcp__{drive_server_name}__` 접두사 도구로 오늘 날짜 기준 Google Docs에서 Gemini 회의 요약 검색 (제목에 "회의 요약", "Meeting notes", "Gemini" 등 포함된 문서)

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

**활성화 조건**: `--preview` 플래그, 또는 첫 실행 (설정 파일 생성 직후)
**비활성화 조건**: `--auto` 플래그

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
   - 슬랙 활성화 시 → `references/slack-format.md` 참조하여 상세 리포트 생성
   - poke 활성화 시 → `references/poke-format.md` 참조하여 비서 스타일 요약 생성
4. 두 포맷은 1회 호출로 동시 생성 (프롬프트에서 `## 슬랙 리포트`와 `## poke 요약`으로 구분)

---

## Step 2.5: 전송 전 확인 (조건부)

**활성화 조건**: `--preview` 플래그, 또는 기본 모드
**비활성화 조건**: `--auto` 플래그

생성된 리포트 미리보기 출력 후:
```
전송할까요?
```

- Y → Step 3로
- N → "수정할 부분을 알려주세요" → 수정 반영 → Step 2 재실행

---

## Step 3: 전송

활성화된 채널만 **병렬** 전송:

- **슬랙**: `mcp__{slack_server_name}__` 도구로 설정된 `user_id`에게 DM 전송
- **poke**: `mcp__{poke_server_name}__` 도구로 문자 전송

전송 결과:
```
✅ 슬랙 DM 전송 완료
✅ poke 전송 완료
```

일부 채널 전송 실패 시:
```
✅ 슬랙 DM 전송 완료
❌ poke 전송 실패: {에러 메시지}
재시도하시겠어요?
```

---

## NEVER

- 회의록 원본을 외부 서비스에 저장하지 않는다
- 회의록 내용을 로컬 파일이나 캐시에 저장하지 않는다
- API 키를 로그, 콘솔 출력, 에러 메시지에 노출하지 않는다
- 온보딩 완료 전에 본 워크플로우를 실행하지 않는다
- 사용자 확인 없이 외부 채널로 메시지를 전송하지 않는다 (`--auto` 플래그 제외)
- 다른 사용자의 슬랙 DM으로 메시지를 전송하지 않는다
- 과거 날짜의 회의록을 조회하지 않는다 (당일 회의록만 대상)

---

## Error Handling

| 상황 | 대응 |
|------|------|
| MCP 서버 연결 실패 | 서버 이름 + 에러 안내. "MCP 설정을 확인해주세요" |
| 인증 만료 (401/403) | caret: 새 API 키 입력받아 `~/.mcp.json` 자동 업데이트. Gemini: Bash로 `gcloud auth login --brief` 실행하여 재인증 |
| 설정 파일 없음/손상 | 자동으로 온보딩 시작. 손상 시 "설정을 재구성하겠습니다." 안내 후 진행 |
| 일부 소스 조회 실패 | "caret 조회에 실패했습니다. Gemini 요약 결과만으로 진행할까요?" 확인 |
| 일부 채널 전송 실패 | 성공한 채널은 유지, 실패 채널 재시도 여부 확인 |

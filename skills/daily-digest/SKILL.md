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

# Daily Digest — Orchestrator

하루 회의록을 통합 분석하여 슬랙 AgentDeck 봇과 poke 문자로 전송한다.

**IMPORTANT**: 모든 사용자 커뮤니케이션은 한국어로 한다.

## 플래그

| 플래그 | 설명 |
|--------|------|
| (없음) | 기본 모드. 확인 없이 바로 전송 |
| `--preview` | 항상 미리보기 + 전송 확인 |
| `--auto` | 모든 확인 생략, 자동 전송 (coworks 스케줄용) |

`--auto`와 `--preview`는 상호 배타적. 동시 지정 시 `--preview`가 우선한다.

---

## 실행 흐름

```
setup → collect → analyze → deliver
```

각 Phase의 상세 내용은 `phases/*.md` 파일을 Read 도구로 읽어서 실행한다.

| Phase | 파일 | 역할 |
|-------|------|------|
| 0 | `phases/00-setup.md` | 설정 체크 + 온보딩 |
| 1 | `phases/01-collect.md` | 회의록 조회 (caret + Gemini) |
| 2 | `phases/02-analyze.md` | 통합 분석 + 포맷 생성 |
| 3 | `phases/03-deliver.md` | 전송 (슬랙 + poke) |

**Phase 전환 규칙:**
- 각 Phase는 진입 조건과 종료 조건을 명시한다
- Phase 실행 전 해당 `phases/*.md`를 Read 도구로 읽고 지시를 따른다
- 이전 Phase가 정상 완료되어야 다음 Phase로 진행
- Phase 0에서 회의 0건 → Phase 1 이후 진행하지 않고 종료

---

## 글로벌 에러 핸들링

### 재시도 규칙
- 같은 API 호출이 **3회 연속 실패** → 해당 소스/채널 포기하고 사용자에게 안내
- 인증 에러 (401/403) → 1회 자동 재인증 시도 후 재실패 시 중단
- 타임아웃 → 1회 재시도 후 실패 시 건너뛰기

### 에러 대응표

| 상황 | 대응 |
|------|------|
| MCP 서버 연결 실패 | 서버 이름 + 에러 안내. "MCP 설정을 확인해주세요" |
| 인증 만료 (401/403) | caret: 새 API 키 입력받아 `~/.mcp.json` 자동 업데이트. Gemini: `gcloud auth login --enable-gdrive-access` 자동 실행 |
| 설정 파일 없음/손상 | 자동으로 온보딩 시작 |
| 일부 소스 조회 실패 | "caret 조회에 실패했습니다. Gemini 요약 결과만으로 진행할까요?" 확인 |
| 일부 채널 전송 실패 | 성공한 채널은 유지, 실패 채널 재시도 여부 확인 |

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

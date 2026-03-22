# Daily Digest

하루 회의록을 통합 분석하여 슬랙 DM(상세 리포트)과 poke 문자(비서 스타일 요약)로 전송하는 Claude Code 플러그인.

## 지원 소스

| 소스 | 설명 |
|------|------|
| **caret** | AI 회의 어시스턴트 (caret.so) |
| **Google Meet** | Gemini 트랜스크립트 |

## 지원 채널

| 채널 | 설명 |
|------|------|
| **슬랙 DM** | 상세 리포트 (회의 목록, 요약, 액션 아이템) |
| **poke 문자** | 비서 스타일 핵심 요약 |

## 설치

Claude Code 플러그인으로 설치:

```bash
claude plugin add /path/to/daily-digest
```

## 필수 MCP 서버

사용하려는 소스/채널에 맞는 MCP 서버가 필요합니다 (전부 필요하지 않음, 최소 소스 1개 + 채널 1개):

- **caret MCP** — 회의록 소스
- **Google Meet MCP** — 회의록 소스
- **Slack MCP** — 전송 채널
- **poke MCP** — 전송 채널

## 사용법

### 기본 실행

```
/daily-digest
```

첫 실행 시 온보딩이 시작됩니다 (소스/채널 선택 + 연결 테스트).

### 플래그

| 플래그 | 설명 |
|--------|------|
| `--auto` | 확인 없이 자동 전송 (coworks 스케줄용) |
| `--preview` | 전송 전 미리보기 강제 |

`--auto`와 `--preview`는 상호 배타적입니다. 동시 지정 시 `--preview`가 우선합니다.

### 설정 변경

```
/digest-setup          # 현재 설정 확인 + 변경
/digest-setup --reset  # 전체 초기화
```

## 설정 파일

`~/.claude/daily-digest.json`에 저장됩니다 (권한: 0600).

API 키는 이 파일에 저장되지 않습니다. 각 MCP 서버의 자체 설정에서 관리합니다.

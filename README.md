# tna-tools

개발 워크플로우 자동화를 위한 [Claude Code](https://github.com/anthropics/claude-code) 플러그인.

## 제공되는 스킬

| 스킬 | 설명 |
|------|------|
| `/change-log` | Jira 기반 feature 브랜치의 변경로그를 자동 생성하여 Confluence에 게시 |
| `/daily-digest` | 하루 회의록을 통합 분석하여 슬랙 DM + poke 문자로 전송 |
| `/digest-setup` | daily-digest 설정 관리 (소스/채널 추가/제거/초기화) |

## 빠른 시작

### 1. 마켓플레이스 추가

```bash
/plugin marketplace add kdanuu/tna-plugin-marketplace
```

### 2. 플러그인 설치

```bash
/plugin install tna-tools
```

### 3. 사용

```bash
/change-log          # 변경로그 생성
/daily-digest        # 오늘 회의 요약
/digest-setup        # 다이제스트 설정 변경
```

## 사전 준비

### change-log 스킬

- **Atlassian MCP 플러그인** 필수 (`atlassian@claude-plugins-official`)
- `/plugin` -> `atlassian@claude-plugins-official` 설치 -> Atlassian 계정 인증

### daily-digest 스킬

- 회의록 소스 MCP 1개 이상: caret MCP 또는 Google Meet MCP
- 전송 채널 MCP 1개 이상: Slack MCP 또는 poke MCP
- 첫 실행 시 `/daily-digest`로 온보딩 진행

## 저장소 구조

```
.
├── .claude-plugin/
│   ├── plugin.json              # 플러그인 매니페스트
│   └── marketplace.json         # 마켓플레이스 설정
├── skills/
│   ├── change-log/
│   │   └── SKILL.md             # 변경로그 생성 스킬
│   ├── daily-digest/
│   │   ├── SKILL.md             # 회의 다이제스트 스킬
│   │   └── references/
│   │       ├── poke-format.md   # poke 포맷 가이드
│   │       └── slack-format.md  # 슬랙 포맷 가이드
│   └── digest-setup/
│       └── SKILL.md             # 다이제스트 설정 스킬
├── CLAUDE.md                    # 플러그인 규칙
└── README.md
```

## MCP 토큰 만료 시 재인증

1. `/plugin` -> `atlassian@claude-plugins-official` 선택
2. `2. Re-authenticate` 선택
3. 브라우저에서 재로그인 후 권한 승인

## 기여하기

1. `skills/` 아래에 새 스킬 디렉토리 생성
2. `SKILL.md` 파일 추가 (frontmatter + 스킬 프롬프트)
3. 테스트 후 PR 제출

## 라이선스

MIT License

## 제작자

**danwoo-kim** ([@kdanuu](https://github.com/kdanuu))

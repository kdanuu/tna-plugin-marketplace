# tna-tools

개발 워크플로우 자동화를 위한 [Claude Code](https://github.com/anthropics/claude-code) 플러그인.

## 제공되는 스킬

### `/change-log` — 변경로그 자동 생성

feature 브랜치의 git diff, 커밋 히스토리, Jira 티켓을 AI가 종합 분석하여 상세한 변경로그를 생성하고 Confluence에 자동으로 게시합니다.

- Git diff + 커밋 히스토리 + Jira 티켓 종합 분석
- Confluence 페이지 자동 생성 (매 배포마다 별도 페이지)
- 로컬 마크다운 파일 보관
- Jira 티켓에 자동 댓글 + PR 링크 포함

### `/daily-digest` — 일일 회의 다이제스트

하루 회의록을 통합 분석하여 슬랙 DM(상세 리포트)과 poke 문자(비서 스타일 요약)로 전송합니다.

- **회의록 소스**: caret (AI 회의 어시스턴트), Gemini 회의 요약 (Google Docs)
- **전송 채널**: 슬랙 DM (제목 + 스레드 상세 리포트), poke 문자 (200자 이내 요약)
- 두 소스에 같은 회의가 있으면 교차 분석하여 더 풍부한 요약 생성
- `--auto` 플래그로 자동 전송 (스케줄 자동화용)
- `--preview` 플래그로 전송 전 미리보기

### `/digest-setup` — 다이제스트 설정 관리

daily-digest의 소스/채널 추가, 제거, 전체 초기화.

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

### change-log

| 필수 항목 | 설정 방법 |
|-----------|-----------|
| Atlassian MCP 플러그인 | `/plugin` → `atlassian@claude-plugins-official` 설치 → Atlassian 계정 인증 |
| Confluence 페이지 URL | 첫 실행 시 입력하면 Space Key + Page ID 자동 추출 |

### daily-digest

최소 **소스 1개 + 채널 1개** 필요. 첫 실행 시 온보딩이 자동으로 진행됩니다.

#### 회의록 소스

| 소스 | 필요한 것 | 설정 |
|------|-----------|------|
| **caret** | API 키 | caret 앱 설치 ([caret.so](https://caret.so)) → 설정 > API 키 생성 ([가이드](https://docs.caret.so/api-reference/introduction)) → 온보딩 시 키 입력하면 `~/.mcp.json`에 자동 등록 |
| **Gemini 요약** | gcloud CLI + Google Drive 권한 | `gcloud auth login --enable-gdrive-access`로 인증 (온보딩 시 자동 실행) |

#### 전송 채널

| 채널 | 필요한 것 | 설정 |
|------|-----------|------|
| **슬랙 DM** | Slack 플러그인 | `/plugin` → `slack@claude-plugins-official` 설치 → 슬랙 계정 인증 |
| **poke 문자** | API 키 | [poke.com](https://poke.com) 가입 → [설정 > Advanced](https://poke.com/settings/advanced) → API 키 생성 → 온보딩 시 키 입력 |

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
│   │       ├── slack-format.md  # 슬랙 리포트 포맷 가이드
│   │       └── poke-format.md   # poke 요약 포맷 가이드
│   └── digest-setup/
│       └── SKILL.md             # 다이제스트 설정 스킬
├── CLAUDE.md                    # 플러그인 규칙
└── README.md
```

## 트러블슈팅

### change-log

| 증상 | 해결 |
|------|------|
| MCP 세션 만료 | `/plugin` → `atlassian@claude-plugins-official` → `Re-authenticate` |

### daily-digest

| 증상 | 해결 |
|------|------|
| caret MCP 로드 안 됨 | `~/.mcp.json` JSON 문법 확인 (trailing comma 등) → Claude Code 재시작 |
| Drive API 403 | `gcloud auth login --enable-gdrive-access`로 재인증 |
| Gemini 회의록 안 나옴 | Google Meet에서 Gemini 노트 기능이 활성화되어 있는지 확인 |
| 슬랙 전송 실패 | 별도 터미널에서 `claude` → `/plugin` → `slack@claude-plugins-official` 재인증 |

## 기여하기

1. `skills/` 아래에 새 스킬 디렉토리 생성
2. `SKILL.md` 파일 추가 (YAML frontmatter + 스킬 프롬프트)
3. 테스트 후 PR 제출

기존 스킬을 참조 구조로 확인하세요.

## 라이선스

MIT License

## 제작자

**danwoo-kim** ([@kdanuu](https://github.com/kdanuu))

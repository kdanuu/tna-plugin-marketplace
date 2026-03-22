# Claude Code 플러그인 마켓플레이스

[Claude Code](https://github.com/anthropics/claude-code)를 위한 프로덕션 레벨의 플러그인 모음으로, AI 기반 자동화를 통해 개발 워크플로우를 향상시킵니다.

## 📦 제공되는 플러그인

### 🔄 [change-log](plugins/change-log/)
**Jira & Confluence 자동 변경로그 생성기** | `v2.3.2`

feature 브랜치를 develop으로 머지할 때마다 git 커밋 히스토리, Jira 티켓 정보, 코드 변경사항을 AI가 자동으로 분석하여 상세한 변경로그를 생성하고 Confluence에 자동으로 게시합니다.

- Git diff + 커밋 히스토리 + Jira 티켓 종합 분석
- Confluence 페이지 자동 생성 + 로컬 마크다운 보관
- Jira 티켓 자동 댓글 + PR 링크 포함
- **필수 MCP**: Atlassian MCP 플러그인

```bash
/change-log
```

[-> 상세 문서](plugins/change-log/)

---

### 📋 [daily-digest](plugins/daily-digest/)
**일일 회의록 통합 분석 & 전송** | `v0.1.0`

하루 회의록을 통합 분석하여 슬랙 DM(상세 리포트)과 poke 문자(비서 스타일 요약)로 전송합니다. caret MCP와 Google Meet MCP에서 회의록을 병렬 조회하고, 참석자/요약/액션 아이템을 분석합니다.

- caret + Google Meet 회의록 병렬 조회
- 슬랙 DM (상세 리포트) + poke 문자 (비서 요약) 동시 전송
- `--auto` 플래그로 coworks 스케줄 자동화 지원
- **필수 MCP**: 소스 1개 이상 (caret/Google Meet) + 채널 1개 이상 (Slack/poke)

```bash
/daily-digest
/digest-setup        # 설정 관리
```

[-> 상세 문서](plugins/daily-digest/)

---

## 🚀 빠른 시작

### 1. 마켓플레이스 추가

```bash
/plugin marketplace add kdanuu/tna-plugin-marketplace
```

### 2. 플러그인 설치

```bash
# 변경로그 생성기
/plugin install change-log

# 일일 회의 다이제스트
/plugin install daily-digest
```

### 3. 사용

```bash
/change-log       # 변경로그 생성
/daily-digest     # 오늘 회의 요약
```

### 설치 확인

Claude에게 사용 가능한 스킬 목록을 요청하세요:
```
What skills are available?
```

### 사전 준비: Atlassian MCP 플러그인 (change-log 필수)

1. `/plugin` 명령어로 플러그인 브라우저 열기
2. `atlassian@claude-plugins-official` 검색 및 설치
3. Atlassian 계정으로 인증
4. `Status: ✔ connected`, `Auth: ✔ authenticated` 확인

### 🔄 MCP 토큰 만료 시 재인증

1. `/plugin` -> `atlassian@claude-plugins-official` 선택
2. `2. Re-authenticate` 선택
3. 브라우저에서 재로그인 후 권한 승인

## 🗂️ 저장소 구조

```
.
├── plugins/
│   ├── change-log/                      # 변경로그 생성 플러그인
│   │   ├── .claude-plugin/plugin.json
│   │   ├── CLAUDE.md
│   │   ├── README.md
│   │   └── skills/change-log/SKILL.md
│   │
│   └── daily-digest/                    # 일일 회의 다이제스트 플러그인
│       ├── .claude-plugin/plugin.json
│       ├── CLAUDE.md
│       ├── README.md
│       ├── commands/digest-setup.md
│       └── skills/daily-digest/
│           ├── SKILL.md
│           └── references/
│               ├── poke-format.md
│               └── slack-format.md
│
├── .claude-plugin/
│   └── marketplace.json                 # 마켓플레이스 설정
└── README.md                            # 이 파일
```

각 플러그인은 다음을 포함합니다:
- **`.claude-plugin/plugin.json`**: 플러그인 매니페스트 (이름, 버전, 설명, 작성자)
- **`CLAUDE.md`**: 플러그인 레벨 규칙
- **`README.md`**: 플러그인 상세 문서
- **`skills/`**: 스킬 프롬프트 및 참조 자료
- **`commands/`**: 추가 커맨드 (선택)

## 🔮 로드맵

- 🧪 **test-generator**: 기존 코드에서 지능형 테스트 생성
- 📚 **doc-sync**: 코드 변경사항과 문서 동기화 유지
- 🔐 **security-audit**: 자동화된 보안 취약점 스캐닝
- 🎯 **code-reviewer**: AI 기반 코드 리뷰 및 제안
- 🔄 **migration-helper**: 프레임워크/라이브러리 마이그레이션 지원

*새로운 플러그인에 대한 아이디어가 있으신가요?* [이슈 열기](https://github.com/kdanuu/tna-plugin-marketplace/issues) 또는 풀 리퀘스트를 제출해주세요!

## 🤝 기여하기

1. **버그 리포트 또는 기능 요청** - [GitHub Issues](https://github.com/kdanuu/tna-plugin-marketplace/issues)를 통해
2. **개선사항 제출** - 풀 리퀘스트를 통해
3. **자신의 플러그인 공유** - 포함시키고 싶습니다!

### 새 플러그인 추가하기

1. `plugins/` 아래에 새 디렉토리 생성
2. `.claude-plugin/plugin.json` 매니페스트 추가
3. `CLAUDE.md`, `README.md`, `skills/` 구조 구성
4. `.claude-plugin/marketplace.json` 업데이트
5. 플러그인을 철저히 테스트
6. 풀 리퀘스트 제출

기존 플러그인을 참조 구조로 확인하세요.

## 📋 요구사항

- [Claude Code CLI](https://github.com/anthropics/claude-code) (최신 버전 권장)
- Git (버전 관리 기능용)
- 각 플러그인별 필수 MCP 서버는 해당 플러그인 문서 참조

## 📄 라이선스

MIT License - 자세한 내용은 [LICENSE](LICENSE) 파일 참조

## 👤 제작자

**danwoo-kim** ([@kdanuu](https://github.com/kdanuu))이 제작 및 유지 관리합니다.

## 🌟 지원하기

이 플러그인이 도움이 되었다면:
- ⭐ 이 저장소에 스타를 눌러주세요
- 🐛 발견한 이슈를 리포트해주세요
- 💡 새로운 기능이나 플러그인을 제안해주세요
- 📢 팀과 공유해주세요

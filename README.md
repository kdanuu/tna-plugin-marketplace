# Claude Code 스킬 컬렉션

[Claude Code](https://github.com/anthropics/claude-code)를 위한 프로덕션 레벨의 스킬 모음으로, AI 기반 자동화를 통해 개발 워크플로우를 향상시킵니다.

## 📦 제공되는 스킬

이 저장소는 **모노레포**로 구성되어 있으며, 각각의 특정 개발 과제를 해결하도록 설계된 여러 Claude Code 스킬을 포함합니다:

### 🔄 [change-log](skills/change-log/)
**Jira & Confluence 자동 변경로그 생성기**

Git 브랜치에서 자동으로 종합적인 변경 로그를 생성하고 Jira 통합과 함께 Confluence에 게시합니다.

**주요 기능:**
- 🎯 브랜치 이름에서 자동 Jira 티켓 감지
- 📊 AI 기반 git diff 분석
- 📝 원클릭 Confluence 문서화
- 🔄 스마트 페이지 관리 (생성 또는 추가)
- 🤖 지능형 영향 분석 및 기술 요약
- 🔐 Atlassian MCP 플러그인을 통한 안전한 인증
- 🔄 MCP 세션 만료 시 자동 재인증 (v2.1.0+)

**사전 요구사항:**
- ⚠️ [Atlassian MCP 플러그인](https://github.com/modelcontextprotocol/servers/tree/main/src/atlassian) 설치 및 인증 필수

**사용 사례:** 릴리즈 문서화, 팀 협업, 변경사항 추적

[→ 전체 문서 보기](skills/change-log/)

---

### 🛠️ [api-codegen](skills/api-codegen/)
**프로덕션 레벨 API 클라이언트 생성기**

Swagger/OpenAPI 명세서에서 타입 안전한 프로덕션 레벨의 API 클라이언트 코드를 대화형 커스터마이징과 함께 생성합니다.

**주요 기능:**
- 📋 Swagger/OpenAPI 파싱 (URL 또는 로컬 파일)
- 🔍 기존 프로젝트 구조 및 코드 스타일 분석
- 🎨 프로젝트 컨벤션에 맞는 코드 생성
- ✅ 포괄적인 단위 및 통합 테스트 작성
- 🔧 다양한 언어 지원 (Kotlin, Java, TypeScript, Python)
- 🏗️ 프레임워크 인식 (Spring Boot, React, Vue, FastAPI 등)

**사용 사례:** 마이크로서비스 통합, 서드파티 API 연동, 백엔드-프론트엔드 정렬

[→ 전체 문서 보기](skills/api-codegen/)

---

## 🚀 빠른 시작

### 마켓플레이스를 통한 설치

1. **마켓플레이스 추가:**
```bash
/plugin marketplace add kdanuu/tna-plugin-marketplace
```

2. **스킬 설치:**
```bash
# 변경로그 생성기 설치
/plugin install change-log

# 또는 API 코드 생성기 설치
/plugin install api-codegen
```

3. **다음 대화에서 사용:**
```bash
/change-log
# 또는
/api-codegen https://api.example.com/swagger.json
```

### 설치 확인

Claude에게 사용 가능한 스킬 목록을 요청하세요:
```
What skills are available?
```

응답에서 설치된 스킬을 확인할 수 있습니다.

## 📖 사용 방법

각 스킬은 자체 종합 문서를 제공합니다:
- [change-log 사용 가이드](skills/change-log/)
- [api-codegen 사용 가이드](skills/api-codegen/)

기본 사용 패턴:
```bash
# 스킬 명령어 사용
/스킬이름 [옵션]

# 또는 자연어로
"변경로그 생성해줘"
"swagger에서 API 클라이언트 만들어줘"
```

## 🗂️ 저장소 구조

```
.
├── skills/
│   ├── change-log/                    # 변경로그 생성 플러그인
│   │   ├── .claude-plugin/
│   │   │   └── plugin.json            # 플러그인 매니페스트
│   │   └── SKILL.md                   # 스킬 프롬프트 & 문서
│   └── api-codegen/                   # API 코드 생성기 플러그인
│       ├── .claude-plugin/
│       │   └── plugin.json            # 플러그인 매니페스트
│       └── SKILL.md                   # 스킬 프롬프트 & 문서
├── .claude-plugin/
│   └── marketplace.json               # 마켓플레이스 설정
└── README.md                          # 이 파일
```

각 플러그인은 다음을 포함합니다:
- **`.claude-plugin/plugin.json`**: 플러그인 메타데이터 (이름, 버전, 설명, 작성자)
- **`SKILL.md`**: 실제 스킬 프롬프트 및 상세 문서

## 🔮 로드맵 & 향후 스킬

생산성을 높이는 새로운 스킬로 이 컬렉션을 지속적으로 확장하고 있습니다. 계획된 추가 사항:

- 🧪 **test-generator**: 기존 코드에서 지능형 테스트 생성
- 📚 **doc-sync**: 코드 변경사항과 문서 동기화 유지
- 🔐 **security-audit**: 자동화된 보안 취약점 스캐닝
- 🎯 **code-reviewer**: AI 기반 코드 리뷰 및 제안
- 🔄 **migration-helper**: 프레임워크/라이브러리 마이그레이션 지원

*새로운 스킬에 대한 아이디어가 있으신가요?* [이슈 열기](https://github.com/kdanuu/tna-plugin-marketplace/issues) 또는 풀 리퀘스트를 제출해주세요!

## 🤝 기여하기

기여를 환영합니다! 다음과 같이 도울 수 있습니다:

1. **버그 리포트 또는 기능 요청** - [GitHub Issues](https://github.com/kdanuu/tna-plugin-marketplace/issues)를 통해
2. **개선사항 제출** - 풀 리퀘스트를 통해
3. **자신의 스킬 공유** - 포함시키고 싶습니다!

### 새 스킬 추가하기

1. `skills/` 아래에 새 디렉토리 생성
2. 스킬 프롬프트가 포함된 `SKILL.md` 파일 추가
3. `.claude-plugin/marketplace.json` 업데이트
4. 스킬을 철저히 테스트
5. 풀 리퀘스트 제출

기존 스킬을 참조 구조로 확인하세요.

## 📋 요구사항

- [Claude Code CLI](https://github.com/anthropics/claude-code) (최신 버전 권장)
- Git (버전 관리 기능용)
- 스킬별 추가 요구사항:
  - **change-log**: Jira/Confluence 통합을 위한 [Atlassian MCP 플러그인](https://github.com/modelcontextprotocol/servers/tree/main/src/atlassian) 필수
  - 전체 요구사항은 개별 스킬 문서 참조

## 📄 라이선스

MIT License - 자세한 내용은 [LICENSE](LICENSE) 파일 참조

## 👤 제작자

**danwoo-kim** ([@kdanuu](https://github.com/kdanuu))이 제작 및 유지 관리합니다.

## 🌟 지원하기

이 스킬들이 도움이 되었다면:
- ⭐ 이 저장소에 스타를 눌러주세요
- 🐛 발견한 이슈를 리포트해주세요
- 💡 새로운 기능이나 스킬을 제안해주세요
- 📢 팀과 공유해주세요

---

**Claude와 함께 즐거운 코딩하세요!** 🎉

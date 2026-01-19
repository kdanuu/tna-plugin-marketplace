# Change Log Generator for Claude Code

[English](README.md) | 한국어

Jira 기반 피처 브랜치의 종합적인 변경 로그를 자동으로 생성하고 Confluence에 게시하는 Claude Code 스킬입니다.

## 주요 기능

- 🎯 **자동 Jira 티켓 감지** - 브랜치 이름에서 티켓 번호 추출
- 📊 **Git diff 분석** - AI 기반 변경사항 요약
- 📝 **Confluence 통합** - 자동 문서화
- 🔄 **스마트 페이지 관리** - 기존 페이지에 추가하거나 새로 생성
- 🤖 **AI 생성 인사이트** - 개요, 기술 세부사항, 영향 분석

## 설치

### Claude Code 마켓플레이스를 통한 설치 (권장)

1. 마켓플레이스 추가:
```bash
/plugin marketplace add kdanuu/change-log
```

2. 플러그인 설치:
```bash
/plugin install change-log
```

3. 다음 대화에서 스킬이 자동으로 사용 가능해집니다.

### 설치 확인

설치 후 Claude에게 다음과 같이 물어보세요:
```
What skills are available?
```

목록에서 `change-log`를 확인할 수 있습니다.

## 사용 방법

### 최초 설정

처음 사용할 때 스킬이 대화형 설정 과정을 안내합니다:

```bash
/change-log
```

필요한 정보:
1. **Confluence 페이지 URL** - 변경 로그를 생성할 부모 페이지
2. **Atlassian 이메일** - Atlassian 계정 이메일
3. **Atlassian API 토큰** - https://id.atlassian.com/manage-profile/security/api-tokens 에서 생성

### 변경 로그 생성

설정이 완료되면 다음과 같이 실행하세요:

```bash
/change-log
```

또는 다음과 같이 말해도 됩니다:
- "create changelog"
- "update confluence changelog"

스킬은 다음 작업을 수행합니다:
1. 브랜치 이름에서 Jira 티켓 추출 (예: `feature/PROJ-123-description`)
2. 현재 브랜치와 `develop` 브랜치 간의 git 변경사항 분석
3. Jira 티켓 정보 가져오기
4. AI 기반 변경사항 요약 생성
5. 포맷된 페이지로 Confluence에 게시

## 브랜치 이름 형식

브랜치는 다음 패턴을 따라야 합니다:
- `feature/JIRA-123-description`
- `bugfix/JIRA-456-fix-something`

브랜치가 이 패턴과 일치하지 않으면, 스킬이 티켓 번호를 수동으로 입력하도록 요청합니다.

## 생성되는 변경 로그 형식

각 변경 로그에는 다음 내용이 포함됩니다:
- **Jira 티켓 정보** - 요약, 설명, 상태
- **개요** - 변경사항의 개략적 요약
- **기술 세부사항** - 주요 아키텍처 또는 구현 변경사항
- **변경된 파일** - 수정, 추가, 삭제된 파일 목록
- **커밋 히스토리** - 브랜치의 모든 커밋
- **영향 분석** - 영향을 받는 시스템 부분
- **코드 통계** - 변경된 라인 수, 수정된 파일 수

## 설정

설정은 `~/.claude/confluence-changelog.json`에 저장되며 다음을 포함합니다:
- Jira 및 Confluence URL
- API 인증 정보
- Space Key 및 부모 페이지 ID
- 브랜치 패턴 (커스터마이징 가능)
- 변경 로그 페이지 제목 템플릿

## 변경 로그 페이지 제목 예시

```
Change Log - 2026-01 - OAuth2 사용자 인증 구현
```

형식: `Change Log - {YYYY-MM} - {Jira 티켓 요약}`

## 요구사항

- Claude Code CLI
- Git 저장소
- API 액세스 권한이 있는 Jira 계정
- 쓰기 권한이 있는 Confluence 계정

## 라이선스

MIT

## 제작자

danwoo-kim

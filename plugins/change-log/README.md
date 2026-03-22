# Change Log

Jira 기반 feature 브랜치의 변경로그를 자동 생성하여 Confluence에 게시하는 Claude Code 플러그인.

## 지원 기능

| 기능 | 설명 |
|------|------|
| **Git 분석** | diff, 커밋 히스토리, 변경 파일 자동 분석 |
| **Jira 연동** | 브랜치명에서 티켓 번호 추출, 티켓 정보 조회 |
| **Confluence 게시** | 상세 변경로그를 Confluence 페이지로 자동 생성 |
| **로컬 저장** | 마크다운 파일로 로컬에도 보관 |
| **PR 연동** | GitHub PR 정보 자동 추출 및 링크 포함 |

## 필수 MCP 서버

- **Atlassian MCP** (`atlassian@claude-plugins-official`) — Jira/Confluence API 접근

## 설치

```bash
/plugin marketplace add kdanuu/tna-plugin-marketplace
/plugin install change-log
```

## 사용법

```bash
/change-log
```

첫 실행 시 Confluence 페이지 URL을 입력하면 Space Key와 Page ID가 자동 추출됩니다.

## 설정 파일

`~/.claude/confluence-changelog.json`에 저장됩니다.

```json
{
  "confluenceSpaceKey": "DEVX",
  "confluenceParentPageId": "5444796456",
  "branchPattern": "^(feature|bugfix)/([A-Z]+-\\d+)",
  "changelogPageTitle": "Change Log - {YYYY-MM} - {summary}"
}
```

API 토큰은 이 파일에 저장되지 않습니다. Atlassian MCP 플러그인이 인증을 관리합니다.

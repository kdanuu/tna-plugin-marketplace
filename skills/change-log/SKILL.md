---
name: change-log
description: Generates a comprehensive Change Log for Jira-based feature branches and publishes it to Confluence when merging to develop. Use when creating changelog, updating confluence changelog, or documenting feature branch changes.
user-invocable: true
---

# Confluence Change Log Generator

## Description
Generates a comprehensive Change Log for Jira-based feature branches and publishes it to Confluence when merging to develop.

**IMPORTANT**: This skill communicates with users in Korean (한국어). All questions, confirmations, and error messages should be in Korean.

## Initial Setup (First-time Use Only)

**IMPORTANT**: Before processing any changelog request, ALWAYS check if configuration exists first.

### Step 0: Check Configuration
1. Use the Read tool to check if `~/.claude/confluence-changelog.json` exists
2. If the file does NOT exist or Read fails:
   - Inform the user that initial setup is required
   - Explain what information is needed
   - Guide them through the setup process (see "Setup Process" below)
   - DO NOT proceed with changelog generation until config.json is created
3. If the file exists, read it and validate required fields are present

### Setup Process (Interactive)
When config.json doesn't exist, guide the user through this interactive setup:

**IMPORTANT**: Ask questions ONE BY ONE and WAIT for user responses. DO NOT use AskUserQuestion tool with multiple choice options - just ask directly in conversation and wait for their input.

1. **Explain what's needed** (in Korean):
   ```
   이 스킬을 사용하려면 Jira와 Confluence 자격 증명을 설정해야 합니다.
   몇 가지 질문을 드리고 설정 파일을 생성해드리겠습니다.

   필요한 정보:
   - 변경 로그를 생성할 Confluence 페이지 URL (여기서 모든 정보를 추출합니다)
   - Atlassian 계정 이메일
   - Atlassian API 토큰 (없으시면 생성 방법을 안내해드립니다)
   ```

2. **Ask for Confluence Page URL** (simplest approach - get everything from one URL):
   - Ask (in Korean): "변경 로그를 생성할 Confluence 페이지 URL을 알려주세요."
   - Provide example: "페이지 URL을 복사해서 붙여넣어주세요. 예: https://your-company.atlassian.net/wiki/spaces/DEV/pages/123456789/Change+Logs"
   - Wait for user to provide the full Confluence page URL
   - Parse the URL using regex pattern: `https://([^/]+)/wiki/spaces/([^/]+)/pages/(\d+)`
     - Group 1: domain (e.g., myrealtrip.atlassian.net)
     - Group 2: spaceKey (e.g., DEVX)
     - Group 3: pageId (e.g., 5444796456)
   - From the parsed URL, derive ALL required URLs:
     - `jiraBaseUrl`: `https://{domain}` (Jira uses same domain without /wiki)
     - `confluenceBaseUrl`: `https://{domain}/wiki`
     - `confluenceSpaceKey`: Group 2
     - `confluenceParentPageId`: Group 3
   - If parsing fails, fall back to asking individually (in Korean):
     - "URL을 파싱할 수 없습니다. Jira 기본 URL을 알려주세요. (예: https://your-company.atlassian.net)"
     - "Confluence Space Key를 알려주세요."
     - "Parent Page ID를 알려주세요."

3. **Ask for API Tokens** (one question at a time, in Korean):
   - Ask: "Atlassian 계정 이메일을 알려주세요."
   - Wait for user to provide email
   - Ask: "Atlassian API 토큰을 알려주세요."
   - Provide help text: "토큰이 없으시면 여기서 생성하세요: https://id.atlassian.com/manage-profile/security/api-tokens"
   - Wait for user to provide the API token

4. **Create config.json**:
   - Use Write tool to create `~/.claude/confluence-changelog.json` (in global Claude Code config directory, NOT inside skill folder)
   - Include all collected information
   - Use the same API token for both Jira and Confluence (they share the same Atlassian token)
   - Use the same email for both jiraEmail and confluenceEmail

5. **Confirm setup complete** (in Korean):
   ```
   ✅ 설정이 ~/.claude/confluence-changelog.json에 성공적으로 저장되었습니다.
   🔒 API 토큰은 로컬에 저장되며 git에 커밋되지 않습니다.

   이제 /change-log를 사용하여 변경 로그를 생성할 수 있습니다.
   ```

### Configuration Fields
- `jiraBaseUrl`: Jira instance URL (e.g., https://your-company.atlassian.net)
- `confluenceBaseUrl`: Confluence instance URL (usually {jiraBaseUrl}/wiki)
- `confluenceSpaceKey`: Confluence space key (e.g., DEV)
- `confluenceParentPageId`: Parent page ID where changelogs will be created
- `jiraEmail`: Jira account email
- `jiraApiToken`: Jira API token
- `confluenceEmail`: Confluence account email (usually same as jiraEmail)
- `confluenceApiToken`: Confluence API token (usually same as jiraApiToken)
- `branchPattern`: Regex pattern for branch names (default: "^(feature|bugfix)/([A-Z]+-\\d+)")
- `changelogPageTitle`: Template for changelog page titles (default: "Change Log - {YYYY-MM} - {summary}")
  - `{YYYY-MM}` will be replaced with current year-month (e.g., 2026-01)
  - `{summary}` will be replaced with Jira ticket summary (작업 요약)
  - Example result: "Change Log - 2026-01 - OAuth2 사용자 인증 구현"

## Process (After Configuration is Complete)

### 1. Extract Jira Ticket from Branch (Optional)
- Get current git branch name
- Extract Jira ticket number from branch name pattern: `feature/JIRA-123-*`, `bugfix/JIRA-123-*`, or `hotfix/JIRA-123-*`
- If no Jira ticket found in branch name:
  - Ask user in Korean: "이 브랜치와 연관된 Jira 티켓이 있나요?"
  - If user says no or doesn't provide one, continue WITHOUT Jira ticket
  - Changelog will be created without Jira ticket information

### 2. Gather Change Information
- Get git diff between current branch and develop: `git diff develop...HEAD`
- Get commit history: `git log develop..HEAD --pretty=format:"%h - %an, %ar : %s"`
- Get list of changed files: `git diff --name-status develop...HEAD`
- Count lines changed: `git diff --stat develop...HEAD`
- **Extract PR information** (if available):
  - Use `gh pr list --head {current_branch} --json number,title,url` to find associated PR
  - If PR exists, extract: PR number, title, URL
  - If no PR found or gh command fails, continue without PR information

### 3. Fetch Jira Ticket Information (if Jira ticket exists)
If a Jira ticket number was found or provided:
- Using Jira REST API: GET `/rest/api/3/issue/{ticketNumber}`
- Extract: summary, description, issue type, status, assignee, priority
- This information will be used in the changelog title and Jira comment

If no Jira ticket:
- Skip this step and continue with changelog generation

### 4. Generate AI-Powered Change Summary
Analyze the code changes, Jira ticket information, and business context to generate a COMPREHENSIVE changelog:
- **Business Context**: Why this change was needed (from Jira ticket description and analysis)
- **Overview**: Detailed summary of what changed and the business/technical reasoning
- **Technical Details**: In-depth explanation of architectural and implementation changes
- **Impact Analysis**: Comprehensive analysis of affected systems, dependencies, and potential side effects
- **Breaking Changes**: Any breaking changes, migration notes, or compatibility concerns
- **Testing Notes**: How to test or verify the changes
- **Dependencies**: New dependencies added or updated

### 5. Format Change Log for Confluence
Create **COMPREHENSIVE and DETAILED** changelog in Confluence Storage Format that serves as complete documentation:
```
<h2>{JIRA-TICKET or Branch Name}: {Ticket Summary}</h2>
<p><strong>Date:</strong> {current date}</p>
<p><strong>Author:</strong> {git author}</p>
<p><strong>Branch:</strong> {branch name}</p>
{If Jira ticket exists: <p><strong>Jira Ticket:</strong> <a href="{jira link}">{ticket number}</a> - {ticket status}</p>}
{If PR exists: <p><strong>PR:</strong> <a href="{pr url}">#{pr number}</a> - {pr title}</p>}

<h3>Business Context</h3>
<p>{Why this change was needed from business perspective, based on Jira description and analysis}</p>

<h3>Overview</h3>
<p>{Detailed summary explaining what changed, why it changed, and the expected outcome.
Include business reasoning and technical motivation. This should be 3-5 sentences minimum.}</p>

<h3>Technical Changes</h3>
<p>{Comprehensive explanation of implementation details:
- What components were modified
- How the architecture changed
- Design decisions and rationale
- Code structure changes
Use paragraphs and nested lists for clarity}</p>

<h3>Impact Analysis</h3>
<p>{Detailed analysis:
- Which systems/modules are affected
- Downstream dependencies
- Performance implications
- Security considerations
- Compatibility notes}</p>

<h3>Breaking Changes & Migration Notes</h3>
<p>{If applicable: detailed migration guide, API changes, configuration updates needed}</p>

<h3>Testing & Verification</h3>
<p>{How to test these changes, what scenarios to verify}</p>

<h3>Files Changed ({count})</h3>
<ul>
{list of changed files with status and brief description of what changed in each file}
</ul>

<h3>Commit History</h3>
<ul>
{commit messages with explanatory context}
</ul>

<h3>Code Statistics</h3>
<p>Files changed: {count} | Insertions: {+lines} | Deletions: {-lines}</p>

<h3>Related Documentation</h3>
<p>{Links to related documentation, ADRs, or design docs if applicable}</p>
```

### 6. Publish to Confluence
- Determine page title format: `Change Log - {YYYY-MM} - {Jira Ticket Summary or Branch Summary}`
  - Example: "Change Log - 2026-01 - OAuth2 사용자 인증 구현"
  - Use the Jira ticket summary (NOT the ticket number) in the title if available
  - If no Jira ticket, use a brief summary of the branch changes
  - Extract YYYY-MM from current date
- Check if a changelog page with this title already exists under parent page
- If exists: Append to existing page content using Confluence REST API
- If not exists: Create new page under parent page with the generated title
- Use Confluence REST API: PUT `/rest/api/content/{pageId}` to update or POST `/rest/api/content` to create

### 7. Save to Local change-log Directory
- Create `change-log/` directory in the project root if it doesn't exist
- Generate markdown filename: `{YYYY-MM-DD}-{jira-ticket-or-branch-name}.md`
  - Example: `2026-01-20-SIM-71.md` or `2026-01-20-feature-user-auth.md`
- Convert the changelog to **Markdown format** (not Confluence Storage Format):
  ```markdown
  # {JIRA-TICKET or Branch Name}: {Ticket Summary}

  **날짜:** {current date}

  **담당자:** {git author} ({author email})

  **브랜치:** {branch name}

  {If Jira ticket exists: **Jira 티켓:** [{ticket number}]({jira link}) - {ticket status}}

  {If PR exists: **PR:** [#{pr number}]({pr url}) - {pr title}}

  ## 📋 개요
  {Detailed summary explaining what changed, why it changed, and the expected outcome}

  ## 🔧 주요 기술적 변경사항
  {Comprehensive explanation of implementation details with numbered sections}

  ## 📊 영향도 분석
  ### 영향 받는 모듈
  {Which systems/modules are affected}

  ### 신규 의존성
  {New dependencies added}

  ### 호환성
  {Compatibility notes}

  ## 📁 변경된 파일 ({count}개)
  ### 신규 추가 (주요 파일)
  {list of added files with brief description}

  ### 수정
  {list of modified files with brief description}

  ## 💻 주요 커밋 히스토리 ({count}개 커밋)
  {commit messages - show recent 10-15 commits}

  ## 📈 코드 통계
  - 변경된 파일: {count}개
  - 추가: {+lines}줄
  - 삭제: {-lines}줄
  - 순 증가: {net}줄

  ## ✅ 완료 사항
  {List of completed tasks/features}

  ---
  *생성일: {YYYY-MM-DD}*
  ```
- Save the markdown file to `change-log/{filename}.md`
- Inform user that changelog was saved locally

### 8. Update Jira Ticket (if Jira ticket exists)
If a Jira ticket number was found or provided:
- Post a comment to the Jira ticket using Jira REST API: POST `/rest/api/3/issue/{ticketKey}/comment`
- Comment format should be BRIEF and link to Confluence for details:
```
✅ 변경 로그가 Confluence에 게시되었습니다.

📄 [상세 내용 보기|{Confluence 링크}]

*변경 파일: {count}개 | 추가: +{lines}, 삭제: -{lines} | 커밋: {count}개*
```

### 9. Confirm Success
Display the following in Korean:
- ✅ Confluence 페이지 링크: {url}
- 💾 로컬 파일: `change-log/{filename}.md`
- {If Jira updated: 📝 Jira 티켓 업데이트됨: {jira url}}
- 📊 통계: {files} 파일, +{lines}/-{lines} 줄, {commits}개 커밋

## Error Handling
All error messages should be displayed in Korean:

- **Git errors**: "Git 저장소가 아니거나 develop/main 브랜치를 찾을 수 없습니다."
- **API call failures**: "Jira/Confluence API 호출에 실패했습니다. 설정 파일과 네트워크 연결을 확인해주세요."
- **Configuration errors**: "설정 파일을 확인해주세요: ~/.claude/confluence-changelog.json"
- **Branch pattern mismatch**: "Jira 티켓 번호를 추출할 수 없습니다. 수동으로 입력하시겠어요?"
- **No PR found**: Continue without PR information (not an error)
- Always show helpful error messages with suggested fixes in Korean

## Tools Required
- Bash: for git commands
- Read: for reading config file
- Write: for creating config file if needed
- WebFetch or API calls: for Jira and Confluence APIs (may need to use bash curl)

## Example Usage

### First-time Use (Setup Required)
```
User: /change-log
Assistant: 이 스킬을 사용하려면 Jira와 Confluence 자격 증명을 설정해야 합니다.
몇 가지 질문을 드리고 설정 파일을 생성해드리겠습니다.

필요한 정보:
- 변경 로그를 생성할 Confluence 페이지 URL (여기서 모든 정보를 추출합니다)
- Atlassian 계정 이메일
- Atlassian API 토큰 (없으시면 생성 방법을 안내해드립니다)

변경 로그를 생성할 Confluence 페이지 URL을 알려주세요.
페이지 URL을 복사해서 붙여넣어주세요. 예: https://your-company.atlassian.net/wiki/spaces/DEV/pages/123456789/Change+Logs

User: https://mycompany.atlassian.net/wiki/spaces/DEV/pages/123456789/Change+Logs

---
name: change-log
description: Generates a comprehensive Change Log for Jira-based feature branches and publishes it to Confluence when merging to develop. Use when creating changelog, updating confluence changelog, or documenting feature branch changes.
user-invocable: true
---

# Confluence Change Log Generator

## Description
Generates a comprehensive Change Log for Jira-based feature branches and publishes it to Confluence when merging to develop.

**IMPORTANT**: This skill communicates with users in Korean (한국어). All questions, confirmations, and error messages should be in Korean.

## Prerequisites

**⚠️ REQUIRED: Atlassian MCP Plugin**

This skill requires the Atlassian MCP plugin to be installed and authenticated. Before using this skill, ensure:

1. **Install Atlassian MCP plugin**:
   - The user must have `@modelcontextprotocol/server-atlassian` installed in their Claude Code configuration
   - Check if MCP tools are available by looking for tools starting with `mcp__plugin_atlassian_atlassian__`

2. **Verify MCP Authentication**:
   - The user must have authenticated with Atlassian through the MCP plugin
   - If MCP tools are not available, inform the user in Korean:
     ```
     ⚠️ 이 스킬을 사용하려면 Atlassian MCP 플러그인이 설치되고 인증되어야 합니다.

     설치 방법:
     1. /plugin 명령어를 입력하여 플러그인 목록을 여세요
     2. 'atlassian@claude-plugins-official'를 검색하거나 찾아서 선택하세요
     3. Enter를 눌러 설치하세요
     4. 브라우저가 열리면 Atlassian 계정으로 로그인하세요
     5. 권한을 승인하고 인증을 완료하세요
     6. 인증 완료 후 이 스킬을 다시 실행해주세요
     ```
   - Do NOT proceed without MCP plugin access

3. **Get Cloud ID**:
   - Before any Atlassian API calls, retrieve the cloud ID using `mcp__plugin_atlassian_atlassian__getAccessibleAtlassianResources`
   - Store the cloud ID for use in subsequent API calls

4. **MCP Session Validation**:
   - At the start of skill execution, validate MCP session by calling `mcp__plugin_atlassian_atlassian__getAccessibleAtlassianResources`
   - If the call fails with authentication error (401/403) or returns empty resources:
     - Show the re-authentication error message (see Error Handling section)
     - Stop execution (do not retry automatically)
   - If session validation succeeds, proceed with changelog generation

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
   이 스킬을 사용하려면 Confluence 페이지 정보를 설정해야 합니다.
   (Atlassian MCP 플러그인을 통해 인증이 이미 완료되어 있어야 합니다)

   필요한 정보:
   - 변경 로그를 생성할 Confluence 페이지 URL (Space Key와 Parent Page ID를 추출합니다)
   ```

2. **Ask for Confluence Page URL** (simplest approach - get everything from one URL):
   - Ask (in Korean): "변경 로그를 생성할 Confluence 페이지 URL을 알려주세요."
   - Provide example: "페이지 URL을 복사해서 붙여넣어주세요. 예: https://your-company.atlassian.net/wiki/spaces/DEV/pages/123456789/Change+Logs"
   - Wait for user to provide the full Confluence page URL
   - Parse the URL using regex pattern: `https://([^/]+)/wiki/spaces/([^/]+)/pages/(\d+)`
     - Group 2: spaceKey (e.g., DEVX)
     - Group 3: pageId (e.g., 5444796456)
   - If parsing fails, fall back to asking individually (in Korean):
     - "URL을 파싱할 수 없습니다. Confluence Space Key를 알려주세요."
     - "Parent Page ID를 알려주세요."

3. **Create config.json**:
   - Use Write tool to create `~/.claude/confluence-changelog.json` (in global Claude Code config directory, NOT inside skill folder)
   - Include only the necessary Confluence information (authentication is handled by MCP)

4. **Confirm setup complete** (in Korean):
   ```
   ✅ 설정이 ~/.claude/confluence-changelog.json에 성공적으로 저장되었습니다.
   🔒 인증은 Atlassian MCP 플러그인을 통해 안전하게 관리됩니다.

   이제 /change-log를 사용하여 변경 로그를 생성할 수 있습니다.
   ```

### Configuration Fields
The configuration file only needs minimal information since MCP handles authentication:

- `confluenceSpaceKey`: Confluence space key (e.g., DEVX)
- `confluenceParentPageId`: Parent page ID where changelogs will be created (e.g., "5444796456")
- `branchPattern`: Regex pattern for branch names (optional, default: "^(feature|bugfix)/([A-Z]+-\\d+)")
- `changelogPageTitle`: Template for changelog page titles (optional, default: "Change Log - {YYYY-MM} - {summary}")
  - `{YYYY-MM}` will be replaced with current year-month (e.g., 2026-01)
  - `{summary}` will be replaced with Jira ticket summary (작업 요약)
  - Example result: "Change Log - 2026-01 - OAuth2 사용자 인증 구현"

**Note**: No API tokens or credentials needed in the config file - MCP handles all authentication.

## Process (After Configuration is Complete)

### 0. Get Cloud ID (Required for all MCP calls)
- Before any Atlassian API calls, retrieve the cloud ID:
  - Use `mcp__plugin_atlassian_atlassian__getAccessibleAtlassianResources` to get available cloud IDs
  - **Error Handling**:
    - If the call fails with authentication error (401, 403, or "unauthorized" in error message):
      - Show Korean error message (same as in Error Handling section):
        ```
        🔐 Atlassian MCP 세션이 만료되었습니다. 재인증이 필요합니다.

        재인증 방법:
        1. /plugin 명령어로 플러그인 목록을 여세요
        2. 'atlassian@claude-plugins-official' 또는 'Plugin:atlassian:atlassian MCP Server'를 찾아서 Enter를 눌러 선택하세요
        3. 메뉴에서 '2. Re-authenticate'를 선택하세요
        4. 브라우저가 자동으로 열리면 Atlassian 계정으로 다시 로그인하세요
        5. 권한 승인 후 완료하세요
        6. 플러그인 상태가 'Status: ✔ connected', 'Auth: ✔ authenticated'인지 확인하세요
        7. 재인증 완료 후 이 스킬을 다시 실행해주세요
        ```
      - Stop execution (do not retry automatically)
  - Store the cloud ID (it will be used in all subsequent MCP tool calls)
  - If multiple cloud IDs are available, use the first one or ask the user
- **IMPORTANT**: All MCP Atlassian tools require a `cloudId` parameter
- **IMPORTANT**: If any MCP call fails with authentication error during execution, show the same re-authentication message and stop

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
- Use MCP tool: `mcp__plugin_atlassian_atlassian__getJiraIssue`
  - Parameters: `cloudId` (from step 0), `issueIdOrKey` (e.g., "SIM-71")
- Extract from response: `fields.summary`, `fields.description`, `fields.issuetype.name`, `fields.status.name`, `fields.assignee`, `fields.priority`
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
- Determine base page title format: `Change Log - {YYYY-MM} - {Jira Ticket Summary or Branch Summary}`
  - Example: "Change Log - 2026-01 - OAuth2 사용자 인증 구현"
  - Use the Jira ticket summary (NOT the ticket number) in the title if available
  - If no Jira ticket, use a brief summary of the branch changes
  - Extract YYYY-MM from current date

- **Check if page with same base title exists**:
  - Use `mcp__plugin_atlassian_atlassian__getPagesInConfluenceSpace` to list pages in the space
  - Filter by `spaceId` (from config's `confluenceParentPageId`)
  - Search for pages starting with the base title (e.g., "Change Log - 2026-01 - OAuth2 사용자 인증 구현")
  - Look for existing update numbers in titles matching pattern: `{base_title}` or `{base_title} (Update N)`
  - Find the highest update number N (if any exist)

- **Determine final page title**:
  - If NO pages with same base title exist: Use base title as-is
    - Example: "Change Log - 2026-01 - OAuth2 사용자 인증 구현"
  - If pages with same base title exist: Increment the highest update number
    - Example: If "Change Log - 2026-01 - OAuth2 사용자 인증 구현" exists, create "Change Log - 2026-01 - OAuth2 사용자 인증 구현 (Update 1)"
    - Example: If "...(Update 1)" exists, create "...(Update 2)"
  - This ensures each changelog deployment is a separate, independent document

- **Create new page** (always create, never update):
  - Use `mcp__plugin_atlassian_atlassian__createConfluencePage`
  - Parameters: `cloudId`, `spaceId` (get from spaceKey using getConfluenceSpaces), `parentId`, `title` (the final title with update number if needed), `body`, `contentFormat: "markdown"`

**Note**: MCP tools support markdown format, which is easier to work with than Confluence Storage Format

**Rationale**: Creating separate pages for each deployment provides:
- Clear version history tracking
- No risk of overwriting existing changelog content
- Easy comparison between different deployments
- Better audit trail for changes

### 7. Save to Local change-log Directory
- Create `change-log/` directory in the project root if it doesn't exist
- Generate markdown filename: `{YYYY-MM-DD}-{jira-ticket-or-branch-name}[-update-N].md`
  - If this is the first changelog: `2026-01-20-SIM-71.md` or `2026-01-20-feature-user-auth.md`
  - If this is an update: `2026-01-20-SIM-71-update-1.md`, `2026-01-20-SIM-71-update-2.md`, etc.
  - The update number should match the Confluence page update number for consistency
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
- Use MCP tool: `mcp__plugin_atlassian_atlassian__addCommentToJiraIssue`
  - Parameters:
    - `cloudId` (from step 0)
    - `issueIdOrKey` (e.g., "SIM-71")
    - `commentBody` (in Markdown format)
- Comment format should be BRIEF and link to Confluence for details:
```markdown
✅ 변경 로그가 Confluence에 게시되었습니다.

📄 [상세 내용 보기]({Confluence 링크})

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

- **MCP not available**:
  ```
  ⚠️ Atlassian MCP 플러그인이 설치되지 않았거나 인증되지 않았습니다.

  설정 방법:
  1. Claude Code 설정에서 Atlassian MCP 서버를 추가하세요
  2. Atlassian 계정으로 인증을 완료하세요
  3. 이 스킬을 다시 실행해주세요
  ```
- **MCP Session Expired (401/403 errors)**:
  ```
  🔐 Atlassian MCP 세션이 만료되었습니다. 재인증이 필요합니다.

  재인증 방법:
  1. /plugin 명령어로 플러그인 목록을 여세요
  2. 'atlassian@claude-plugins-official' 또는 'Plugin:atlassian:atlassian MCP Server'를 찾아서 Enter를 눌러 선택하세요
  3. 메뉴에서 '2. Re-authenticate'를 선택하세요
  4. 브라우저가 자동으로 열리면 Atlassian 계정으로 다시 로그인하세요
  5. 권한 승인 후 완료하세요
  6. 플러그인 상태가 'Status: ✔ connected', 'Auth: ✔ authenticated'인지 확인하세요
  7. 재인증 완료 후 이 스킬을 다시 실행해주세요
  ```
- **Git errors**: "Git 저장소가 아니거나 develop/main 브랜치를 찾을 수 없습니다."
- **MCP call failures**: "Atlassian API 호출에 실패했습니다. MCP 플러그인 인증을 확인해주세요."
- **Configuration errors**: "설정 파일을 확인해주세요: ~/.claude/confluence-changelog.json"
- **Branch pattern mismatch**: "Jira 티켓 번호를 추출할 수 없습니다. 수동으로 입력하시겠어요?"
- **No PR found**: Continue without PR information (not an error)
- **Cloud ID not found**: "Atlassian Cloud ID를 찾을 수 없습니다. MCP 플러그인 설정을 확인해주세요."
- Always show helpful error messages with suggested fixes in Korean

## Tools Required
- Bash: for git commands
- Read: for reading config file
- Write: for creating config file if needed and saving local markdown files
- MCP Atlassian tools (required):
  - `mcp__plugin_atlassian_atlassian__getAccessibleAtlassianResources`: Get cloud ID
  - `mcp__plugin_atlassian_atlassian__getJiraIssue`: Fetch Jira ticket information
  - `mcp__plugin_atlassian_atlassian__getConfluenceSpaces`: Get space ID from space key
  - `mcp__plugin_atlassian_atlassian__getPagesInConfluenceSpace`: Check if changelog page exists and find update numbers
  - `mcp__plugin_atlassian_atlassian__createConfluencePage`: Create new changelog page (always creates new page, never updates)
  - `mcp__plugin_atlassian_atlassian__addCommentToJiraIssue`: Add comment to Jira ticket

## Example Usage

### First-time Use (Setup Required)
```
User: /change-log
Assistant: 이 스킬을 사용하려면 Confluence 페이지 정보를 설정해야 합니다.
(Atlassian MCP 플러그인을 통해 인증이 이미 완료되어 있어야 합니다)

필요한 정보:
- 변경 로그를 생성할 Confluence 페이지 URL (Space Key와 Parent Page ID를 추출합니다)

변경 로그를 생성할 Confluence 페이지 URL을 알려주세요.
페이지 URL을 복사해서 붙여넣어주세요. 예: https://your-company.atlassian.net/wiki/spaces/DEV/pages/123456789/Change+Logs

User: https://mycompany.atlassian.net/wiki/spaces/DEV/pages/123456789/Change+Logs


Assistant: Space Key "DEV"와 Parent Page ID "123456789"를 확인했습니다.

✅ 설정이 ~/.claude/confluence-changelog.json에 성공적으로 저장되었습니다.
🔒 인증은 Atlassian MCP 플러그인을 통해 안전하게 관리됩니다.

이제 /change-log를 사용하여 변경 로그를 생성할 수 있습니다.
```

### Regular Use (After Setup)

**First deployment:**
```
User: /change-log
Assistant:
[Checking MCP plugin availability...]
[Getting Cloud ID from MCP...]
[Extracting Jira ticket from branch: feature/SIM-71-oauth-implementation]
[Fetching Jira ticket SIM-71...]
[Gathering git changes...]
[Generating comprehensive changelog...]
[Checking for existing changelog pages...]
[No existing page found - creating new page]
[Publishing to Confluence...]
[Saving local markdown file...]
[Adding comment to Jira ticket...]

✅ 변경 로그가 성공적으로 생성되었습니다\!

📄 Confluence: https://mycompany.atlassian.net/wiki/spaces/DEV/pages/987654321/Change+Log+-+2026-01+-+OAuth2+사용자+인증+구현
💾 로컬 파일: change-log/2026-01-20-SIM-71.md
📝 Jira 티켓 업데이트됨: https://mycompany.atlassian.net/browse/SIM-71
📊 통계: 15 파일, +342/-89 줄, 8개 커밋
```

**Subsequent deployment (same feature, same month):**
```
User: /change-log
Assistant:
[Checking MCP plugin availability...]
[Getting Cloud ID from MCP...]
[Extracting Jira ticket from branch: feature/SIM-71-oauth-implementation]
[Fetching Jira ticket SIM-71...]
[Gathering git changes...]
[Generating comprehensive changelog...]
[Checking for existing changelog pages...]
[Found existing page: "Change Log - 2026-01 - OAuth2 사용자 인증 구현"]
[Creating new page with update number: "Change Log - 2026-01 - OAuth2 사용자 인증 구현 (Update 1)"]
[Publishing to Confluence...]
[Saving local markdown file...]
[Adding comment to Jira ticket...]

✅ 변경 로그가 성공적으로 생성되었습니다\!

📄 Confluence: https://mycompany.atlassian.net/wiki/spaces/DEV/pages/987654322/Change+Log+-+2026-01+-+OAuth2+사용자+인증+구현+(Update+1)
💾 로컬 파일: change-log/2026-01-20-SIM-71-update-1.md
📝 Jira 티켓 업데이트됨: https://mycompany.atlassian.net/browse/SIM-71
📊 통계: 8 파일, +123/-45 줄, 3개 커밋
```

## Configuration File Example

`~/.claude/confluence-changelog.json`:
```json
{
  "confluenceSpaceKey": "DEVX",
  "confluenceParentPageId": "5444796456",
  "branchPattern": "^(feature|bugfix)/([A-Z]+-\\d+)",
  "changelogPageTitle": "Change Log - {YYYY-MM} - {summary}"
}
```

**Note**: This is much simpler than before\! No API tokens, URLs, or email addresses needed.
Authentication is handled securely by the Atlassian MCP plugin.

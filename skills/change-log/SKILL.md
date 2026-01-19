---
name: change-log
description: Generates a comprehensive Change Log for Jira-based feature branches and publishes it to Confluence when merging to develop. Use when creating changelog, updating confluence changelog, or documenting feature branch changes.
user-invocable: true
---

# Confluence Change Log Generator

## Description
Generates a comprehensive Change Log for Jira-based feature branches and publishes it to Confluence when merging to develop.

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

1. **Explain what's needed**:
   ```
   To use this skill, I need to set up your Jira and Confluence credentials.
   I'll ask you a few questions and create the configuration file for you.

   You'll need:
   - The Confluence page URL where you want to create changelogs (I'll extract all the details from this)
   - Your Atlassian account email
   - Your Atlassian API token (I'll guide you to create one if needed)
   ```

2. **Ask for Confluence Page URL** (simplest approach - get everything from one URL):
   - Ask: "What is the Confluence page URL where you want to create changelogs?"
   - Provide example: "Just copy-paste the page URL, e.g., https://your-company.atlassian.net/wiki/spaces/DEV/pages/123456789/Change+Logs"
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
   - If parsing fails, fall back to asking individually:
     - "I couldn't parse the URL. What is your Jira base URL? (e.g., https://your-company.atlassian.net)"
     - "What is your Confluence Space Key?"
     - "What is the Parent Page ID?"

3. **Ask for API Tokens** (one question at a time):
   - Ask: "What is your Atlassian account email?"
   - Wait for user to provide email
   - Ask: "What is your Atlassian API token?"
   - Provide help text: "If you don't have one, create it at: https://id.atlassian.com/manage-profile/security/api-tokens"
   - Wait for user to provide the API token

4. **Create config.json**:
   - Use Write tool to create `~/.claude/confluence-changelog.json` (in global Claude Code config directory, NOT inside skill folder)
   - Include all collected information
   - Use the same API token for both Jira and Confluence (they share the same Atlassian token)
   - Use the same email for both jiraEmail and confluenceEmail

5. **Confirm setup complete**:
   ```
   ✅ Configuration saved successfully at ~/.claude/confluence-changelog.json
   🔒 Your API tokens are stored locally and will not be committed to git.

   You can now use /change-log to generate changelogs.
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
  - Ask user if there's an associated Jira ticket
  - If user says no or doesn't provide one, continue WITHOUT Jira ticket
  - Changelog will be created without Jira ticket information

### 2. Gather Change Information
- Get git diff between current branch and develop: `git diff develop...HEAD`
- Get commit history: `git log develop..HEAD --pretty=format:"%h - %an, %ar : %s"`
- Get list of changed files: `git diff --name-status develop...HEAD`
- Count lines changed: `git diff --stat develop...HEAD`

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

### 7. Update Jira Ticket (if Jira ticket exists)
If a Jira ticket number was found or provided:
- Post a comment to the Jira ticket using Jira REST API: POST `/rest/api/3/issue/{ticketKey}/comment`
- Comment format should be BRIEF and link to Confluence for details:
```
✅ 변경 로그가 Confluence에 게시되었습니다.

📄 [상세 내용 보기|{Confluence 링크}]

*변경 파일: {count}개 | 추가: +{lines}, 삭제: -{lines} | 커밋: {count}개*
```

### 8. Confirm Success
- Display link to updated Confluence page
- If Jira ticket was updated, display link to Jira ticket
- Show summary of what was logged (files changed, lines added/deleted, commits)

## Error Handling
- If git operations fail, inform user they need to be in a git repository
- If API calls fail, check configuration and network connectivity
- If branch doesn't match pattern, ask user for manual input
- Always show helpful error messages with suggested fixes

## Tools Required
- Bash: for git commands
- Read: for reading config file
- Write: for creating config file if needed
- WebFetch or API calls: for Jira and Confluence APIs (may need to use bash curl)

## Example Usage

### First-time Use (Setup Required)
```
User: /change-log
Assistant: I need to set up your Jira and Confluence credentials first.

To use this skill, I need to set up your Jira and Confluence credentials.
I'll ask you a few questions and create the configuration file for you.

You'll need:
- Your Atlassian/Jira URL
- Confluence Space Key and Parent Page ID
- API tokens for Jira and Confluence

What is your Jira/Atlassian URL? (e.g., https://your-company.atlassian.net)

User: https://mycompany.atlassian.net
```

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

### 1. Extract Jira Ticket from Branch
- Get current git branch name
- Extract Jira ticket number from branch name pattern: `feature/JIRA-123-*` or `bugfix/JIRA-123-*`
- If no Jira ticket found, ask user to provide ticket number manually

### 2. Gather Change Information
- Get git diff between current branch and develop: `git diff develop...HEAD`
- Get commit history: `git log develop..HEAD --pretty=format:"%h - %an, %ar : %s"`
- Get list of changed files: `git diff --name-status develop...HEAD`
- Count lines changed: `git diff --stat develop...HEAD`

### 3. Fetch Jira Ticket Information
Using Jira REST API:
- GET `/rest/api/3/issue/{ticketNumber}`
- Extract: summary, description, issue type, status, assignee, priority

### 4. Generate AI-Powered Change Summary
Analyze the code changes and generate:
- **Overview**: High-level summary of what changed and why
- **Technical Details**: Key architectural or implementation changes
- **Impact Analysis**: What parts of the system are affected
- **Breaking Changes**: Any breaking changes or migration notes
- **Dependencies**: New dependencies added or updated

### 5. Format Change Log
Create structured changelog in Confluence Storage Format:
```
<h2>{JIRA-TICKET}: {Ticket Summary}</h2>
<p><strong>Date:</strong> {current date}</p>
<p><strong>Author:</strong> {git author}</p>
<p><strong>Branch:</strong> {branch name}</p>
<p><strong>Jira Ticket:</strong> <a href="{jira link}">{ticket number}</a></p>

<h3>Overview</h3>
<p>{AI-generated overview}</p>

<h3>Technical Changes</h3>
<p>{AI-generated technical details}</p>

<h3>Files Changed ({count})</h3>
<ul>
{list of changed files with status: Modified/Added/Deleted}
</ul>

<h3>Commit History</h3>
<ul>
{commit messages}
</ul>

<h3>Impact Analysis</h3>
<p>{AI-generated impact analysis}</p>

<h3>Code Statistics</h3>
<p>Files changed: {count} | Insertions: {+lines} | Deletions: {-lines}</p>
```

### 6. Publish to Confluence
- Determine page title format: `Change Log - {YYYY-MM} - {Jira Ticket Summary}`
  - Example: "Change Log - 2026-01 - OAuth2 사용자 인증 구현"
  - Use the Jira ticket summary (NOT the ticket number) in the title
  - Extract YYYY-MM from current date
- Check if a changelog page with this title already exists under parent page
- If exists: Append to existing page content using Confluence REST API
- If not exists: Create new page under parent page with the generated title
- Use Confluence REST API: PUT `/rest/api/content/{pageId}` to update or POST `/rest/api/content` to create

### 7. Confirm Success
- Display link to updated Confluence page
- Show summary of what was logged

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

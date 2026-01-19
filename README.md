# Change Log Generator for Claude Code

English | [한국어](README_KR.md)

A Claude Code skill that automatically generates comprehensive change logs for Jira-based feature branches and publishes them to Confluence.

## Features

- 🎯 **Automatic Jira ticket detection** from branch names
- 📊 **Git diff analysis** with AI-powered summaries
- 📝 **Confluence integration** for automatic documentation
- 🔄 **Smart page management** - creates or appends to existing pages
- 🤖 **AI-generated insights** - overview, technical details, and impact analysis

## Installation

### Via Claude Code Marketplace (Recommended)

1. Add the marketplace:
```bash
/plugin marketplace add kdanuu/change-log
```

2. Install the plugin:
```bash
/plugin install change-log
```

3. The skill will be automatically available in your next conversation.

### Verify Installation

After installation, verify by asking Claude:
```
What skills are available?
```

You should see `change-log` in the list.

## Usage

### First-time Setup

On first use, the skill will guide you through an interactive setup:

```bash
/change-log
```

You'll need:
1. **Confluence page URL** - The parent page where changelogs will be created
2. **Atlassian email** - Your Atlassian account email
3. **Atlassian API token** - Create one at https://id.atlassian.com/manage-profile/security/api-tokens

### Generating a Changelog

Once configured, simply run:

```bash
/change-log
```

Or say:
- "create changelog"
- "update confluence changelog"

The skill will:
1. Extract the Jira ticket from your branch name (e.g., `feature/PROJ-123-description`)
2. Analyze git changes between your branch and `develop`
3. Fetch Jira ticket information
4. Generate an AI-powered change summary
5. Publish to Confluence with a formatted page

## Branch Name Format

Your branch should follow this pattern:
- `feature/JIRA-123-description`
- `bugfix/JIRA-456-fix-something`

If your branch doesn't match this pattern, the skill will ask you to provide the ticket number manually.

## Generated Changelog Format

Each changelog includes:
- **Jira ticket information** - Summary, description, status
- **Overview** - High-level summary of changes
- **Technical Details** - Key architectural or implementation changes
- **Files Changed** - List of modified, added, or deleted files
- **Commit History** - All commits in the branch
- **Impact Analysis** - Affected parts of the system
- **Code Statistics** - Lines changed, files modified

## Configuration

Configuration is stored at `~/.claude/confluence-changelog.json` and includes:
- Jira and Confluence URLs
- API credentials
- Space key and parent page ID
- Branch pattern (customizable)
- Changelog page title template

## Example Changelog Page Title

```
Change Log - 2026-01 - OAuth2 사용자 인증 구현
```

Format: `Change Log - {YYYY-MM} - {Jira Ticket Summary}`

## Requirements

- Claude Code CLI
- Git repository
- Jira account with API access
- Confluence account with write permissions

## License

MIT

## Author

Created by danwoo-kim

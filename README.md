# Slack PR Notifications Plugin for Claude Code

Automatically posts PR creation notifications and push updates to a Slack channel when Claude Code creates PRs or pushes code on your behalf.

## Features

- Posts a notification when `gh pr create` succeeds
- Posts a threaded reply when `git push` updates an existing PR
- Tracks thread mappings so push updates appear as replies to the original PR message

## Setup

1. **Create a Slack App** at https://api.slack.com/apps
2. **Add bot token scope**: `chat:write` under OAuth & Permissions
3. **Install** the app to your workspace
4. **Copy** the Bot User OAuth Token
5. **Create config file** at `~/.config/claude-slack/config.json`:

```json
{
  "botToken": "xoxb-your-token-here",
  "defaultChannel": "C_YOUR_CHANNEL_ID"
}
```

6. **Invite the bot** to your channel: `/invite @your-bot-name`

## Installing the plugin

Add to your Claude Code settings (`~/.claude/settings.json`):

```json
{
  "extraKnownMarketplaces": {
    "slack-pr-notifications": {
      "source": {
        "source": "directory",
        "path": "/path/to/claude-slack-pr-plugin"
      }
    }
  },
  "enabledPlugins": {
    "slack-pr-notifications@slack-pr-notifications": true
  }
}
```

Or if published to GitHub:

```json
{
  "extraKnownMarketplaces": {
    "slack-pr-notifications": {
      "source": {
        "source": "github",
        "repo": "your-org/claude-slack-pr-plugin"
      }
    }
  },
  "enabledPlugins": {
    "slack-pr-notifications@slack-pr-notifications": true
  }
}
```

# Slack PR Notifications Plugin for Claude Code

Automatically posts PR creation notifications and push updates to a Slack channel when Claude Code creates PRs or pushes code on your behalf.

## Features

- Posts a notification when `gh pr create` succeeds
- Posts a threaded reply when `git push` updates an existing PR
- Tracks thread mappings so push updates appear as replies to the original PR message

## Setup

After installing the plugin, run the interactive setup wizard in Claude Code:

```bash
/slack-pr-setup
```

This will walk you through creating a Slack App, configuring your bot token, selecting a channel, and validating the connection.

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

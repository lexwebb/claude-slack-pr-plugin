---
name: slack-pr-setup
description: Interactive setup wizard for configuring Slack PR notifications — bot token, channel selection, and validation.
user-invocable: true
---

Walk the user through configuring Slack PR notifications. This creates or updates `~/.config/claude-slack/config.json`.

## Steps

### 1. Check existing config

Read `~/.config/claude-slack/config.json`. If it exists and is valid JSON with `botToken` and `defaultChannel`, tell the user:
> "You already have Slack PR notifications configured (channel: `CHANNEL_ID`). Want to reconfigure?"

If they say no, stop.

### 2. Get the bot token

Ask the user for their Slack Bot User OAuth Token. Guide them:

> To get a bot token:
> 1. Go to https://api.slack.com/apps and create a new app (or select an existing one)
> 2. Under **OAuth & Permissions**, add the `chat:write` bot token scope
> 3. Install the app to your workspace
> 4. Copy the **Bot User OAuth Token** (starts with `xoxb-`)

Use AskUserQuestion to prompt: "Paste your Slack Bot User OAuth Token (xoxb-...)"

Validate it starts with `xoxb-`. If not, ask again.

### 3. Validate the token

Test the token by calling the Slack API:

```bash
curl -s https://slack.com/api/auth.test \
  -H "Authorization: Bearer BOT_TOKEN" \
  -H "Content-Type: application/json"
```

If `"ok": false`, tell the user the token is invalid and ask them to try again.
If `"ok": true`, confirm: "Token valid — authenticated as **BOT_NAME** in **TEAM_NAME** workspace."

### 4. Get the channel

Ask the user which channel to post to. Suggest they can provide either:
- A channel name (e.g. `#frontend`)
- A channel ID (e.g. `C059SBTHSNQ`)

If they provide a channel name, look up the ID by posting a test and checking the error, or ask them to find it in Slack (channel details > scroll to bottom > Channel ID).

Use AskUserQuestion to prompt for the channel.

### 5. Validate the channel

Test posting to the channel:

```bash
curl -s -X POST https://slack.com/api/chat.postMessage \
  -H "Authorization: Bearer BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"channel": "CHANNEL_ID", "text": ":white_check_mark: Slack PR notifications configured successfully!"}'
```

If `"ok": false` with `"channel_not_found"` or `"not_in_channel"`, tell the user:
> "The bot isn't in that channel. Invite it with `/invite @BOT_NAME` in the channel, then try again."

If successful, confirm: "Test message sent to the channel."

### 6. Write the config

Create the directory and write the config:

```bash
mkdir -p ~/.config/claude-slack
```

Write `~/.config/claude-slack/config.json`:
```json
{
  "botToken": "xoxb-...",
  "defaultChannel": "C_CHANNEL_ID"
}
```

Also ensure `~/.config/claude-slack/threads.json` exists (create as `{}` if not).

### 7. Confirm

Tell the user:
> "Slack PR notifications are configured. PR creation and push notifications will now post to your channel automatically."

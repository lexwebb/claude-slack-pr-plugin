---
name: slack-pr
description: Posts PR creation notifications and push updates to a configured Slack channel. Claude invokes this automatically after creating PRs or pushing changes — not user-invocable.
user-invocable: false
---

You have just created a PR or pushed changes on the user's behalf. Post a notification to the configured Slack channel.

## When to invoke this skill

- **After `gh pr create` succeeds** — post a new PR notification
- **After `git push` succeeds (when the user asked you to push)** — post a threaded push update

## Configuration

Read the Slack config from `~/.config/claude-slack/config.json`:

```json
{
  "botToken": "xoxb-...",
  "defaultChannel": "C_CHANNEL_ID"
}
```

If the config file does not exist or is invalid, warn the user:
> "Slack notification skipped — config not found at `~/.config/claude-slack/config.json`. Run `/slack-pr-setup` or see the plugin README for setup instructions."

Do NOT block the PR or push workflow.

## Mode 1: PR Created

After `gh pr create` succeeds:

1. Parse the PR URL from the command output.
2. Get repo and PR details: run `gh pr view --json url,title,body,number,headRefName,author` and parse the JSON. The `author.login` field gives the GitHub username of who created the PR.
3. Read `~/.config/claude-slack/config.json` for `botToken` and `defaultChannel`.
4. Post to Slack:

```bash
curl -s -X POST https://slack.com/api/chat.postMessage \
  -H "Authorization: Bearer BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "CHANNEL_ID",
    "text": ":twisted_rightwards_arrows: *PR_TITLE*\n_SHORT_DESCRIPTION_\n\nPR: PR_URL\nBranch: `BRANCH_NAME`\nAuthor: AUTHOR_LOGIN"
  }'
```

Replace the placeholders with actual values:

- `SHORT_DESCRIPTION` is the first line of the PR body, or a 1-sentence summary if the body is long. Wrap it in `_underscores_` for italic rendering in Slack mrkdwn.
- `AUTHOR_LOGIN` is the GitHub username from `author.login`.
- Use only standard Slack emojis (e.g. `:twisted_rightwards_arrows:`, `:arrow_up:`). Do NOT use custom emojis like `:pull-request:`, `:github:`, or `:git-branch:` — they won't render in most workspaces.

5. Parse the response JSON. Extract `ts` from the response (this is the message timestamp needed for threading).
6. Save the thread mapping. Read `~/.config/claude-slack/threads.json` (create it as `{}` if it doesn't exist), add the entry, and write back:

```json
{
  "PR_URL": {
    "threadTs": "MESSAGE_TS",
    "channel": "CHANNEL_ID",
    "createdAt": "ISO_TIMESTAMP"
  }
}
```

7. Confirm to the user: "Posted PR notification to Slack."

## Mode 2: Push Update

After `git push` succeeds at the user's request:

1. Get the current PR URL: run `gh pr view --json url` and parse the JSON. If no PR exists for this branch, skip silently.
2. Read `~/.config/claude-slack/threads.json`. Look up the PR URL.
3. If a thread entry exists, post a threaded reply:

```bash
curl -s -X POST https://slack.com/api/chat.postMessage \
  -H "Authorization: Bearer BOT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "CHANNEL_ID",
    "thread_ts": "THREAD_TS",
    "text": ":arrow_up: *Pushed updates*\n- SUMMARY_OF_CHANGES"
  }'
```

`SUMMARY_OF_CHANGES` should be 1-2 sentences describing what changed in this push, based on your conversation context about what you just did.

4. If no thread entry exists (PR was created before the plugin was installed), fall back to posting a new top-level message using the PR Created format from Mode 1, and save the new thread mapping.

5. Confirm to the user: "Posted push update to Slack thread."

## Error handling

- If `curl` fails or the Slack API returns `"ok": false`, report the error to the user but do NOT retry or block the workflow.
- If `threads.json` is corrupted, recreate it as `{}` and post a new top-level message.

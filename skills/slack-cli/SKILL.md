---
name: slack-cli
description: Interact with Slack from the command line. Send messages, read channel history, list channels and users, upload files, manage reminders, and more. Use when the user wants to communicate with Slack, check messages, or automate Slack workflows.
metadata:
  author: michaelliv
  version: "1.0"
compatibility: Requires slack-cli (brew install rockymadden/rockymadden/slack-cli), curl, and jq. See references/INSTALLATION.md for setup.
---

# Slack CLI

Interact with Slack workspaces from the command line using `slack-cli` and direct Slack API calls.

> **Assume `slack-cli` is already installed and configured.** If the `slack` command is not found or returns `not_inited`, refer to [references/INSTALLATION.md](references/INSTALLATION.md) for setup instructions.

## When to Use

- User wants to send, update, or delete Slack messages
- User wants to read channel history or search messages
- User wants to list channels, users, or conversations
- User wants to upload files to Slack
- User wants to manage reminders, snooze, or presence
- User wants to automate Slack workflows

## Setup

If `slack` is not available or not configured, see [references/INSTALLATION.md](references/INSTALLATION.md) for full setup instructions.

The token is stored at the path configured in the `slack` binary's `etcdir` variable (typically `/opt/homebrew/etc/slack-cli/.slack` on macOS). You can also use the `SLACK_CLI_TOKEN` environment variable.

## Built-in Commands

The `slack` CLI supports these commands natively:

### Chat

```bash
# Send a message
slack chat send 'Hello world!' '#channel'

# Send with rich formatting
slack chat send --text 'text' --channel '#channel' --color good --title 'Title' --pretext 'Pretext'

# Pipe content as a message
echo "some output" | slack chat send --channel '#channel'

# Update a message (requires timestamp and channel)
slack chat update 'Updated text' 1405894322.002768 '#channel'

# Delete a message
slack chat delete 1405894322.002768 '#channel'

# Chain: send, capture ts+channel, then update
slack chat send 'hello' '#general' --filter '.ts + "\n" + .channel' | \
  xargs -n2 slack chat update 'goodbye'
```

### Files

```bash
# Upload a file
slack file upload README.md '#channel'

# Upload with metadata
slack file upload README.md '#channel' --comment 'See attached' --title 'README'

# Create a Slack post from markdown
slack file upload --file post.md --filetype post --title 'Post Title' --channels '#channel'

# List files
slack file list
slack file list --filter '[.files[] | {id, name, size}]'

# File info / delete
slack file info F2147483862
slack file delete F2147483862
```

### Reminders

```bash
# Add a reminder
slack reminder add 'lunch' $(date -v +30M "+%s")

# List / complete / delete
slack reminder list
slack reminder complete Rm7MGABKT6
slack reminder delete Rm7MGABKT6
```

### Snooze (Do Not Disturb)

```bash
slack snooze start 60          # Start snooze for 60 minutes
slack snooze info              # Check your snooze status
slack snooze end               # End snooze
```

### Presence

```bash
slack presence active
slack presence away
```

### Global Options

All commands support:
- `--filter|-f <jq-filter>` — Apply a jq filter to the JSON response
- `--compact|-cp` — Compact JSON output
- `--monochrome|-m` — No color in jq output
- `--trace|-x` — Enable bash trace for debugging

## Direct API Calls

The `slack` CLI doesn't cover all Slack API methods. For anything beyond the built-in commands, call the Slack API directly using `curl`. Read the token from the CLI's config file.

### Reading the Token

```bash
TOKEN=$(cat /opt/homebrew/etc/slack-cli/.slack)
# Or if SLACK_CLI_TOKEN is set:
TOKEN="${SLACK_CLI_TOKEN}"
```

### List Channels

```bash
curl -s -H "Authorization: Bearer ${TOKEN}" \
  "https://slack.com/api/conversations.list?types=public_channel,private_channel&limit=200" | \
  jq '[.channels[] | {name, id, is_private}]'
```

### Read Channel History

```bash
# Get recent messages from a channel (use channel ID)
curl -s -H "Authorization: Bearer ${TOKEN}" \
  "https://slack.com/api/conversations.history?channel=C05SGNZ8TSN&limit=20" | \
  jq '[.messages[] | {user, text, ts}]'
```

### List Users

```bash
curl -s -H "Authorization: Bearer ${TOKEN}" \
  "https://slack.com/api/users.list" | \
  jq '[.members[] | {name, id, real_name}]'
```

### Search Messages

```bash
curl -s -H "Authorization: Bearer ${TOKEN}" \
  "https://slack.com/api/search.messages?query=keyword&count=10" | \
  jq '[.messages.matches[] | {channel: .channel.name, text, ts}]'
```

### Reactions

```bash
# Add a reaction
curl -s -X POST -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C05SGNZ8TSN","timestamp":"1234567890.123456","name":"thumbsup"}' \
  "https://slack.com/api/reactions.add"

# Get reactions on a message
curl -s -H "Authorization: Bearer ${TOKEN}" \
  "https://slack.com/api/reactions.get?channel=C05SGNZ8TSN&timestamp=1234567890.123456"
```

### Thread Replies

```bash
# Get replies in a thread (use the parent message ts)
curl -s -H "Authorization: Bearer ${TOKEN}" \
  "https://slack.com/api/conversations.replies?channel=C05SGNZ8TSN&ts=1234567890.123456" | \
  jq '[.messages[] | {user, text, ts}]'
```

### Post to a Thread

```bash
curl -s -X POST -H "Authorization: Bearer ${TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"channel":"C05SGNZ8TSN","text":"reply text","thread_ts":"1234567890.123456"}' \
  "https://slack.com/api/chat.postMessage"
```

## Tips

- Channel arguments accept `#name` format for built-in commands, but direct API calls require channel IDs (e.g. `C05SGNZ8TSN`)
- Use `--filter` with any built-in command to extract specific fields via jq
- Pipe commands together using `--filter '.ts + "\n" + .channel'` to chain send/update/delete
- For direct API calls, always quote the URL to prevent shell glob expansion
- Pagination: most list endpoints support `cursor` and `limit` parameters — check `response_metadata.next_cursor` in the response

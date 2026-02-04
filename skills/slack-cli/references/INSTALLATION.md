# Slack CLI Installation

## Prerequisites

- `curl` — for HTTP requests
- `jq` — for JSON processing

Both are typically available on macOS. Install jq via `brew install jq` if needed.

## Install slack-cli

### Via Homebrew (recommended)

```bash
brew tap rockymadden/rockymadden
brew install rockymadden/rockymadden/slack-cli
```

### Via curl

```bash
curl -O https://raw.githubusercontent.com/rockymadden/slack-cli/master/src/slack
chmod +x slack
# Move to somewhere on your PATH
mv slack /usr/local/bin/
```

### Via git clone

```bash
git clone git@github.com:rockymadden/slack-cli.git
cd slack-cli
make install bindir=/usr/local/bin etcdir=/usr/local/etc
```

## Create a Slack App and Get a User Token

You need a **User OAuth Token** (`xoxp-...`) to act as yourself.

1. Go to https://api.slack.com/apps and click **Create New App** > **From scratch**
2. Name it anything (e.g. "CLI") and select your workspace
3. Go to **OAuth & Permissions** in the sidebar
4. Under **User Token Scopes**, add the scopes you need:

### Recommended Scopes

| Scope | Purpose |
|---|---|
| `chat:write` | Send, update, delete messages |
| `channels:read` | List public channels |
| `channels:history` | Read public channel messages |
| `groups:read` | List private channels |
| `groups:history` | Read private channel messages |
| `im:read` | List DMs |
| `im:history` | Read DMs |
| `mpim:read` | List group DMs |
| `mpim:history` | Read group DMs |
| `files:read` | List and view files |
| `files:write` | Upload files |
| `dnd:read` | View snooze status |
| `dnd:write` | Start/end snooze |
| `reminders:read` | List reminders |
| `reminders:write` | Add/complete/delete reminders |
| `search:read` | Search messages and files |
| `users:read` | List users |
| `reactions:read` | View reactions |
| `reactions:write` | Add/remove reactions |
| `pins:read` | View pins |
| `pins:write` | Pin/unpin messages |

> **Note:** Add scopes in batches and reinstall if you encounter `invalid_scope` errors during installation. Some newer or granular scopes (e.g. `canvases:*`, `*:write.invites`, `*:write.topic`, `search:read.*`) may cause installation to fail.

5. Click **Install to Workspace** at the top and authorize
6. Copy the **User OAuth Token** (`xoxp-...`)

## Configure slack-cli

### Via init command

```bash
slack init
# Paste your token when prompted
```

### Via environment variable

```bash
export SLACK_CLI_TOKEN='xoxp-your-token'
```

Add to `~/.zshrc` or `~/.bashrc` to persist.

## Verify

```bash
slack chat send "test" "#general"
```

## Token Location

The token is stored at the `etcdir` path configured in the slack binary. On a Homebrew install this is typically:

```
/opt/homebrew/etc/slack-cli/.slack
```

You can reference this in scripts:

```bash
TOKEN=$(cat /opt/homebrew/etc/slack-cli/.slack)
```

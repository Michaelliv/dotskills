# Jira CLI Installation

## Prerequisites

- `curl` — for HTTP requests
- `jq` — for JSON processing
- `psst` — for credential management

## Install jira-cli

```bash
brew install ankitpokhrel/jira-cli/jira-cli
```

## Get an API Token

1. Go to https://id.atlassian.com/manage-profile/security/api-tokens
2. Click **Create API token**
3. Name it (e.g. "CLI") and copy the token

## Store Credentials

Store all Jira credentials in the global psst vault with the `jira` tag:

```bash
psst --global set JIRA_API_TOKEN --tag jira   # Your API token
psst --global set JIRA_USER --tag jira        # Your email (e.g., user@company.com)
psst --global set JIRA_SERVER --tag jira      # Server URL (e.g., https://company.atlassian.net)
```

## Initialize jira-cli

Run `jira init` to configure the CLI:

```bash
psst --global JIRA_API_TOKEN -- jira init \
  --installation cloud \
  --server $(psst --global get JIRA_SERVER) \
  --login $(psst --global get JIRA_USER) \
  --project YOUR-PROJECT
```

Replace `YOUR-PROJECT` with your default Jira project key (e.g. `PROJ`).

## Verify

```bash
psst --global JIRA_API_TOKEN -- jira issue list --plain
```

This should list recent issues in your default project.

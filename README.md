# Bavlio MCP Server

The Bavlio MCP server is a hosted [Model Context Protocol](https://modelcontextprotocol.io) server that gives AI assistants like Claude, ChatGPT, Cursor, and Codex direct access to B2B lead generation and sales outreach. It exposes 22 tools for finding leads, enriching contact data, building and launching multi-step email and LinkedIn campaigns, and reading replies, all against a live [Bavlio](https://bavlio.com) account.

It is a **remote** server. There is nothing to install, clone, or run locally. You point your MCP client at one URL and sign in with OAuth.

```
https://mcp.bavlio.com/mcp
```

This repository is the public documentation and manifest for that server. The server itself is hosted by Bavlio and its source is not distributed.

## Quick start

Add the server to Claude Code with one command:

```bash
claude mcp add --transport http bavlio https://mcp.bavlio.com/mcp
```

The first tool call opens a browser window for OAuth sign-in. After you approve, the connection persists.

For other clients, see [Example configurations](#example-configurations) below.

## What can an AI assistant do with Bavlio?

Once connected, an assistant can carry out a complete outbound workflow in conversation:

- **Find leads by criteria.** Describe an ideal customer profile in plain language. Bavlio parses it into structured filters, quotes the credit cost before spending anything, and returns named people with titles, companies, domains, and LinkedIn URLs.
- **Bring your own lists.** Import a lead list you already own as a dataset, then enrich it with verified work email addresses.
- **Build campaigns.** Create a draft campaign, write or critique the copy for each step, set the sequence and schedule, choose sending accounts, and preview the exact message a specific lead will receive before anything sends.
- **Launch and control.** Launch a campaign, then pause or resume it.
- **Read replies.** List inbox replies across email and LinkedIn, and read a full conversation thread.
- **Check account state.** Read the credit balance and list every connected sending channel.

Lead discovery and enrichment are metered in credits. Every spending tool quotes a maximum charge and requires explicit confirmation before it runs, so an agent cannot silently spend your balance.

## Which tools are available?

All 22 tools, grouped by domain.

### Leads

| Tool | What it does |
| --- | --- |
| `bavlio_plan_lead_search` | Parse a natural language ICP into structured filters and return a no-spend quote |
| `bavlio_run_lead_search` | Execute a quoted lead search after explicit confirmation |

### Datasets

| Tool | What it does |
| --- | --- |
| `bavlio_get_datasets` | List datasets, or inspect one with columns and sample rows |
| `bavlio_import_dataset` | Import a lead list you already own as a dataset |
| `bavlio_enrich_dataset` | Quote or run enrichment, including work email finding and verification |

### Campaigns

| Tool | What it does |
| --- | --- |
| `bavlio_get_campaigns` | List campaigns or inspect one in detail |
| `bavlio_create_campaign` | Create a draft campaign |
| `bavlio_set_campaign_sequence` | Replace the step sequence on a draft campaign |
| `bavlio_write_step_copy` | Compose or critique the copy for a campaign step |
| `bavlio_set_campaign_schedule` | Set sending windows and sender accounts |
| `bavlio_preview_message` | Render the exact message a specific lead will receive |
| `bavlio_launch_campaign` | Launch a campaign |
| `bavlio_set_campaign_state` | Pause or resume a running campaign |
| `bavlio_delete_campaign` | Delete a campaign |
| `bavlio_get_templates` | List or read saved email templates |
| `bavlio_save_template` | Create or update an email template |
| `bavlio_delete_template` | Delete an email template |

### Inbox

| Tool | What it does |
| --- | --- |
| `bavlio_list_inbox` | List replies across email and LinkedIn |
| `bavlio_get_thread` | Read one full email or LinkedIn conversation |

### Account and jobs

| Tool | What it does |
| --- | --- |
| `bavlio_get_balance` | Read the current credit balance and plan tier |
| `bavlio_list_channels` | List connected mailboxes, LinkedIn accounts, and sender profiles |
| `bavlio_get_job_status` | Poll the status of a long running job |

Full argument and response reference: [docs/tools.md](docs/tools.md).

## How does authentication work?

The server implements the MCP authorization spec with OAuth 2.1, including Dynamic Client Registration. You never paste an API key into your MCP client.

1. Your client discovers the authorization server from `https://mcp.bavlio.com/.well-known/oauth-protected-resource/mcp`.
2. It registers itself dynamically and opens a browser consent screen.
3. You approve, and the client receives a scoped token bound to your Bavlio account and organization.

The authorization server supports PKCE with `S256`, Dynamic Client Registration, and only the `authorization_code` and `refresh_token` grants.

### Headless access with an API key

For servers, cron jobs, and other non-interactive clients, you can send a `bav_live_` API key from your Bavlio dashboard as a bearer token instead of completing the OAuth flow:

```json
{
  "mcpServers": {
    "bavlio": {
      "type": "http",
      "url": "https://mcp.bavlio.com/mcp",
      "headers": { "Authorization": "Bearer bav_live_your_key_here" }
    }
  }
}
```

Treat the key like a password. It carries the same organization scope as your account.

Every tool call is scoped to the organization of the signed-in user. Full detail: [docs/authentication.md](docs/authentication.md).

## Example configurations

### Claude Code

```bash
claude mcp add --transport http bavlio https://mcp.bavlio.com/mcp
```

### Claude Desktop

See [examples/claude-desktop.json](examples/claude-desktop.json).

### Cursor

See [examples/cursor.json](examples/cursor.json).

### VS Code

See [examples/vscode.json](examples/vscode.json).

### Any client supporting streamable HTTP

```json
{
  "mcpServers": {
    "bavlio": {
      "type": "http",
      "url": "https://mcp.bavlio.com/mcp"
    }
  }
}
```

## Do I need a Bavlio account?

Yes. The server operates on live account data, so it requires a Bavlio account to sign in to. You can create one at [bavlio.com](https://bavlio.com). Lead discovery and enrichment consume credits from your plan; reading campaigns, templates, inbox replies, and account state does not.

## Frequently asked questions

### Is this an MCP server for lead generation?

Yes. Lead discovery is the primary use case. `bavlio_plan_lead_search` turns a plain language description of a target market into structured filters and a cost quote, and `bavlio_run_lead_search` executes it. Results come back as a dataset you can then enrich and campaign against without leaving the conversation.

### Can it send cold email?

Yes. You can build a multi step email sequence, attach sending accounts, preview the rendered message for a real lead, and launch. Bavlio manages sending infrastructure, deliverability, and reply detection.

### Does it support LinkedIn outreach?

LinkedIn is one of the supported channels alongside email. `bavlio_list_inbox` and `bavlio_get_thread` cover LinkedIn replies, and campaigns can include LinkedIn steps where a LinkedIn account is connected. Connecting an account itself is done in the Bavlio dashboard, because it requires an interactive browser sign-in that an agent cannot complete.

### Can an agent spend my credits without asking?

No. Lead search follows a two step plan-then-confirm protocol. `bavlio_plan_lead_search` returns a maximum charge and a preview id but spends nothing. Credits are only consumed when `bavlio_run_lead_search` is called with that preview id and explicit confirmation. Previews expire after 15 minutes.

### Do I need to install or self host anything?

No. It is a hosted remote server reachable at `https://mcp.bavlio.com/mcp`. There is no package to install and no local process to run.

### Which MCP clients work with it?

Any client that supports remote MCP over streamable HTTP with OAuth. That includes Claude Code, Claude Desktop, Cursor, VS Code, and Codex.

## Troubleshooting

Common connection and authorization problems are covered in [docs/troubleshooting.md](docs/troubleshooting.md).

## Links

- Bavlio: <https://bavlio.com>
- MCP endpoint: <https://mcp.bavlio.com/mcp>
- Official MCP Registry entry: `com.bavlio/bavlio`
- Model Context Protocol: <https://modelcontextprotocol.io>

## License

The documentation in this repository is released under the MIT License. See [LICENSE](LICENSE). The Bavlio MCP server itself is a hosted commercial service and its source is not distributed here.

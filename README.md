# Bavlio MCP server: LinkedIn and email outreach for AI agents

A hosted LinkedIn MCP server for Claude Code, Cursor, Claude Desktop, VS Code and any client that supports remote [Model Context Protocol](https://modelcontextprotocol.io). An agent sends LinkedIn connection requests with a note and follow-up messages as campaign steps, under per-account daily and weekly caps, and reads the LinkedIn inbox. The same campaigns can send email. 22 tools, one URL, OAuth sign-in:

```
https://mcp.bavlio.com/mcp
```

> **Status: early access.** The server runs today for existing [Bavlio](https://bavlio.com) customers. New accounts join the waitlist at [bavlio.com/waitlist](https://bavlio.com/waitlist?utm_source=github&utm_medium=readme&utm_campaign=bavlio-mcp).

It is a **remote** server. There is nothing to install, clone or run locally. This repository is the public documentation and manifest for that server; the server is hosted by Bavlio and its source is not distributed.

## Quick start

These steps work for existing Bavlio customers. We're not opening new accounts right now; new users join the [waitlist](https://bavlio.com/waitlist?utm_source=github&utm_medium=readme&utm_campaign=bavlio-mcp).

Add the server to Claude Code with one command:

```bash
claude mcp add --transport http bavlio https://mcp.bavlio.com/mcp
```

The first tool call opens a browser window for OAuth sign-in. After you approve, the connection persists.

Cursor, in `~/.cursor/mcp.json`:

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

For other clients, see [Example configurations](#example-configurations) below.

## What can an agent do on LinkedIn?

| Task | Tools |
| --- | --- |
| See your connected LinkedIn accounts and mailboxes | `bavlio_list_channels` |
| Find people by role, company and market, with a cost quote first | `bavlio_plan_lead_search`, `bavlio_run_lead_search` |
| Build a campaign: a connection request with a note, then follow-up messages | `bavlio_create_campaign`, `bavlio_set_campaign_sequence`, `bavlio_write_step_copy`, `bavlio_set_campaign_schedule` |
| See the exact message one person will receive | `bavlio_preview_message` |
| Launch, pause and resume | `bavlio_launch_campaign`, `bavlio_set_campaign_state` |
| Read LinkedIn and email replies, and a full thread | `bavlio_list_inbox`, `bavlio_get_thread` |

What it doesn't do:

- Send a single message or connection request outside a campaign. Every send is a campaign step, so the caps and pacing below apply.
- Connect a LinkedIn account. You do that once in the Bavlio dashboard, because it needs an interactive sign-in.
- Reply to a LinkedIn message. Replies go through Bavlio's REST API (`POST /api/v1/linkedin/conversations/{chat_id}/reply`), not this server.

## LinkedIn sending caps

Each connected LinkedIn account runs under these caps by default:

| Limit | Value |
| --- | --- |
| Connection requests a day | 20 by default (an account can be set to 25 at most). The engine sends the lower of that and the weekly cap spread over the days the account sends: 20 a day sending Monday to Friday, 14 a day sending every day |
| Connection requests per rolling 7 days | 100 |
| Messages a day | 100 |
| A newly connected account | Starts lower and ramps up over 14 days |

A reply takes the person out of the sequence, and an opt-out blocks them on every channel. The full numbers and how pacing works: [LinkedIn automation limits](https://bavimail.com/blog/linkedin-automation-limits).

## Does LinkedIn have an official API for this?

Not one every developer can use. LinkedIn's open permissions cover sign-in, the member's email address, and posting as that member. Sending messages and connection requests is not among them; other permissions need LinkedIn's approval through a partner program ([LinkedIn, on learn.microsoft.com](https://learn.microsoft.com/en-us/linkedin/shared/authentication/getting-access)). This server acts as your own connected account, under the caps above. LinkedIn's [User Agreement](https://www.linkedin.com/legal/user-agreement) prohibits unauthorized automated methods, so any tool that sends from a member account carries a risk of restriction, this one included. The caps keep each account's volume low.

More on the REST side: [LinkedIn API for AI agents](https://bavimail.com/feature/linkedin-api).

## What else can an assistant do with Bavlio?

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

Yes. The server operates on live account data, so it needs a Bavlio account to sign in to. We're not opening new accounts right now; join the waitlist at [bavlio.com/waitlist](https://bavlio.com/waitlist?utm_source=github&utm_medium=readme&utm_campaign=bavlio-mcp). Lead discovery and enrichment consume credits from your plan; reading campaigns, templates, inbox replies, and account state does not.

### Only need email?

[Bavimail](https://bavimail.com/?utm_source=github&utm_medium=readme&utm_campaign=bavlio-mcp), from the same team, has its own MCP server for email, [`@bavimail/mcp-server`](https://github.com/Bavlio/bavimail-mcp-server). It is open to anyone, with 12 tools for sending, inbound email, aliases and domains, and runs on a key from the free plan.

## Frequently asked questions

### Is this an MCP server for lead generation?

Yes. Lead discovery is the primary use case. `bavlio_plan_lead_search` turns a plain language description of a target market into structured filters and a cost quote, and `bavlio_run_lead_search` executes it. Results come back as a dataset you can then enrich and campaign against without leaving the conversation.

### Can it send cold email?

Yes. You can build a multi step email sequence, attach sending accounts, preview the rendered message for a real lead, and launch. Bavlio manages sending infrastructure, deliverability, and reply detection.

### Is there a LinkedIn MCP server that can send connection requests and messages?

Yes. This server sends connection requests with a note and follow-up messages as campaign steps, paced under each account's caps, and reads LinkedIn replies with `bavlio_list_inbox` and `bavlio_get_thread`. It is in early access for existing Bavlio customers; new accounts join the waitlist.

### Can I send LinkedIn messages from Claude Code?

Yes, if you're an existing Bavlio customer. Add the server with `claude mcp add --transport http bavlio https://mcp.bavlio.com/mcp`, sign in, and ask Claude Code to build and launch a campaign with a connection request and follow-up messages. Connecting the LinkedIn account itself is done once in the Bavlio dashboard, because it requires an interactive browser sign-in that an agent cannot complete.

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
- Waitlist: <https://bavlio.com/waitlist>
- LinkedIn MCP server page: <https://bavimail.com/feature/linkedin-mcp-server>
- LinkedIn automation limits: <https://bavimail.com/blog/linkedin-automation-limits>
- MCP endpoint: <https://mcp.bavlio.com/mcp>
- Official MCP Registry entry: `com.bavlio/bavlio`
- Model Context Protocol: <https://modelcontextprotocol.io>

## License

The documentation in this repository is released under the MIT License. See [LICENSE](LICENSE). The Bavlio MCP server itself is a hosted commercial service and its source is not distributed here.

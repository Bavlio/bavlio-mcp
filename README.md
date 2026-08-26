# Bavlio MCP Server

Remote [MCP](https://modelcontextprotocol.io) server for [Bavlio](https://bavlio.com), the LinkedIn + cold email outreach platform. It turns any MCP client (Claude, ChatGPT, Codex, Cursor, VS Code, and others) into a complete outbound engine: find leads, build and launch personalized campaigns across LinkedIn and email, and manage replies.

## Endpoint

```
https://mcp.bavlio.com/mcp
```

- Transport: streamable HTTP
- Auth: OAuth 2.0 with dynamic client registration (connect with your Bavlio account), or a `bav_live_` API key from your Bavlio dashboard
- Registry: published as `com.bavlio/bavlio` in the [official MCP Registry](https://registry.modelcontextprotocol.io)

## Quickstart (Claude)

Add a custom connector with the URL above, or install from the Claude connector directory (search "Bavlio"). Sign in with your Bavlio account when prompted. A free account is enough to plan lead searches, import lists, and draft campaigns; sending requires a paid plan with a connected mailbox or LinkedIn account.

## Tools (22)

**Leads and data**
- `bavlio_plan_lead_search` - plan and quote a lead search from plain language, no spend
- `bavlio_run_lead_search` - execute a quoted search after explicit confirmation
- `bavlio_import_dataset` - import a lead list you already own
- `bavlio_enrich_dataset` - quote or run enrichment (verified emails, LinkedIn, domains)
- `bavlio_get_datasets` - list or inspect datasets

**Campaigns**
- `bavlio_create_campaign` - create a draft campaign from a dataset
- `bavlio_set_campaign_sequence` - replace a campaign's step sequence (LinkedIn + email steps)
- `bavlio_set_campaign_schedule` - set timing and sender selection
- `bavlio_write_step_copy` - draft or critique step copy against real lead context
- `bavlio_preview_message` - render a message exactly as a lead will receive it
- `bavlio_launch_campaign` - launch (confirmation required)
- `bavlio_set_campaign_state` - pause or resume
- `bavlio_delete_campaign` - delete (confirmation required)
- `bavlio_get_campaigns` - list or inspect campaigns

**Templates**
- `bavlio_get_templates` / `bavlio_save_template` / `bavlio_delete_template`

**Inbox**
- `bavlio_list_inbox` - email + LinkedIn replies in one view
- `bavlio_get_thread` - read a full conversation

**Account**
- `bavlio_get_balance` - credit balance
- `bavlio_list_channels` - connected sending channels
- `bavlio_get_job_status` - background job status

Every write tool is annotated honestly (`destructiveHint`, `idempotentHint`), spend is always quoted before it happens, and nothing sends without explicit confirmation.

## Links

- Product: https://bavlio.com
- Docs: https://bavlio.com/docs
- Privacy: https://bavlio.com/privacy
- Support: support@bavlio.com

The server itself is closed-source and hosted; this repository documents the public MCP surface.

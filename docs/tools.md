# Tool reference

The Bavlio MCP server exposes 22 tools. Every tool is single purpose. There are no catch-all action dispatchers, so an assistant can select the right tool from its name and description alone.

Full JSON argument schemas are published by the server itself and are visible to any connected client through `tools/list`. This page describes what each tool is for and the behavior that is not obvious from the schema.

## Leads

### `bavlio_plan_lead_search`

Parses a natural language description of a target market into structured filters and returns a quote. Spends nothing.

Returns a `preview_id`, a `thread_id`, the parsed filters, and a `maximum_charge_credits`. Previews expire **15 minutes** after creation.

Filters are matched as single values, not comma separated lists. Search one industry at a time rather than joining several into one string. Country level locations resolve more reliably than city level ones; put the city in the natural language text instead, where it still influences ranking.

### `bavlio_run_lead_search`

Executes a previously quoted search. Requires the `preview_id` and `thread_id` from a plan, plus explicit confirmation. This is the call that consumes credits.

Results are returned inline and also persisted as a dataset you can enrich or campaign against.

## Datasets

### `bavlio_get_datasets`

Lists datasets, or inspects a single dataset with its column names and a bounded sample of rows.

For a sampled row, the `verification` block is the authoritative email verification verdict. A `catch_all` result is a legitimate outcome rather than a failure: the domain accepts mail for any address, so deliverability of that specific mailbox cannot be confirmed.

### `bavlio_import_dataset`

Imports a lead list you already own as a dataset. Use this when you have specific named people rather than criteria. Lead search cannot reliably locate a specific individual by name, because the filter model has no person name or profile URL field.

### `bavlio_enrich_dataset`

Quotes or runs enrichment over a dataset, including work email discovery and verification. Follows the same quote-then-confirm pattern as lead search.

## Campaigns

### `bavlio_get_campaigns`

Lists campaigns, or inspects one including its sequence, schedule, and metrics.

### `bavlio_create_campaign`

Creates a draft campaign. Campaigns start as drafts and are not sending until launched.

### `bavlio_set_campaign_sequence`

Replaces the step sequence on a campaign. Steps can span email and LinkedIn where the corresponding channel is connected.

### `bavlio_write_step_copy`

Composes new copy for a campaign step, or critiques copy you supply.

### `bavlio_set_campaign_schedule`

Sets sending windows, timezone, pacing, and which sender accounts a campaign uses.

### `bavlio_preview_message`

Renders the exact message a specific lead will receive, with personalization resolved. Use this before launching. It is the cheapest way to catch a bad merge field or an empty personalization.

### `bavlio_launch_campaign`

Launches a campaign. This begins real sending to real people.

### `bavlio_set_campaign_state`

Pauses or resumes a running campaign.

### `bavlio_delete_campaign`

Deletes a campaign. Irreversible.

### `bavlio_get_templates`

Lists saved email templates or reads one in full.

### `bavlio_save_template`

Creates a new email template or updates an existing one.

### `bavlio_delete_template`

Deletes an email template. Irreversible.

## Inbox

### `bavlio_list_inbox`

Lists replies across email and LinkedIn in one view.

### `bavlio_get_thread`

Reads one full conversation thread, email or LinkedIn.

## Account

### `bavlio_get_balance`

Returns spendable credits and the account subscription tier.

### `bavlio_list_channels`

Lists connected LinkedIn accounts, mailboxes, BaviMail domains, and sender profiles.

For a BaviMail domain, `status` is the provider's authoritative verification lifecycle value, for example `pending`, `verified`, or `failed`. No separate derived boolean is returned.

Connecting a new channel is not available through MCP. It requires an interactive browser sign in with the provider, which is done in the Bavlio dashboard.

## Jobs

### `bavlio_get_job_status`

Returns normalized status for a long running job such as an enrichment run. Poll this rather than assuming a job finished.

## Credit spending

Only lead discovery and enrichment consume credits. Reading campaigns, templates, inbox replies, datasets, channels, balance, and job status is free.

Every spending tool follows the same contract: a plan call returns a maximum charge and spends nothing, and a separate run call with explicit confirmation performs the spend. An assistant cannot skip the plan step.

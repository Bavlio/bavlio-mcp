# Troubleshooting

## The client cannot connect

Check the endpoint is reachable and returns a 401 rather than a network error:

```bash
curl -sI https://mcp.bavlio.com/mcp
```

A `401` is correct. It means the server is up and is asking for authorization. A connection error or a 5xx is not expected; if you see one, the service may be degraded.

Confirm the protected resource metadata resolves:

```bash
curl -s https://mcp.bavlio.com/.well-known/oauth-protected-resource/mcp
```

## Authorization loops or repeated consent screens

Some MCP clients key their stored OAuth credential to an ephemeral local callback port, which changes between launches. The client then misses its cache and re-registers. This is client behavior, not a server fault, and it is harmless apart from the extra consent screen.

If a client is stuck reporting unauthorized, remove the server and add it again.

## A lead search returned zero results

This is almost always over constrained filters rather than an empty market.

- Pass a **single value** per filter. A comma joined string such as `"Retail, Software, Finance"` is matched as one literal value and will find nothing. Run one search per industry instead.
- Prefer a **country** level location. City level values can resolve ambiguously, for example matching a same named city in another country. Put the city in the natural language text, where it still influences ranking.
- Remove keyword filters first when narrowing down. They compound quickly.

Broad filters with the city in the text almost always outperform a heavily filtered query.

## A tool reports insufficient credits

Call `bavlio_get_balance` to read the current balance and tier. Only lead discovery and enrichment consume credits. If the balance looks wrong for your plan, contact support rather than retrying.

## A lead search preview expired

Previews expire 15 minutes after creation. Call `bavlio_plan_lead_search` again to get a fresh `preview_id` and `thread_id`, then run that one. Expiry protects you from confirming a quote whose underlying cost has moved.

## No sending channel is available

Campaign tools need a connected mailbox or LinkedIn account. Call `bavlio_list_channels` to see what is connected.

Connecting a channel cannot be done through MCP. It requires an interactive browser sign in with the provider. Connect it in the Bavlio dashboard, then retry.

## A campaign will not launch

Confirm in order:

1. The campaign has a sequence set.
2. The campaign has a schedule and at least one sender account attached.
3. The dataset attached to the campaign has leads with the fields the sequence references. Use `bavlio_preview_message` against a real lead to confirm personalization resolves.

## Getting help

Open an issue on this repository for documentation problems. For account, billing, or delivery issues, contact Bavlio support through <https://bavlio.com>.

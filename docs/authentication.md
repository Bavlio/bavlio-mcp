# Authentication

The Bavlio MCP server implements the MCP authorization specification using OAuth 2.1. You do not paste an API key into your MCP client.

## Endpoint

```
https://mcp.bavlio.com/mcp
```

Transport is streamable HTTP.

## Discovery

Clients discover the authorization server from the protected resource metadata document:

```
https://mcp.bavlio.com/.well-known/oauth-protected-resource/mcp
```

The same document is also served at the well known root for clients that look there first:

```
https://mcp.bavlio.com/.well-known/oauth-protected-resource
```

## Flow

1. Your MCP client requests the protected resource metadata and reads the authorization server URL from it.
2. The client registers itself using Dynamic Client Registration. No manual client id or secret is required.
3. The client opens a browser consent screen. You sign in with your Bavlio account and approve the connection.
4. The client receives an access token scoped to your user and organization.

## Scoping

Every tool call resolves the signed in user to an organization and filters all reads and writes to that organization. A token issued to one account can never read another account's campaigns, datasets, or inbox.

## Sign in requirements

You need a Bavlio account. Create one at <https://bavlio.com>.

Connecting a mailbox or a LinkedIn account is done in the Bavlio dashboard rather than through MCP, because those flows require an interactive browser sign in with the provider that an agent cannot complete on your behalf. Once connected, the MCP tools can use them.

## Re-authorization

Some clients key their stored credential to an ephemeral local callback port and will re-register on the next launch. This is expected client behavior and is harmless. If a client reports that it is unauthorized, remove and re-add the server.

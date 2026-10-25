# Installing the Acrium MCP server

Instructions for an AI agent (e.g. Cline) connecting Acrium as a **remote** MCP server. There is nothing to download, build or run locally: the server is hosted at `https://acriumautomated.com/mcp`.

## Requirements

- An Acrium account. If the user doesn't have one, ask them to sign up at https://acriumautomated.com/setup. Setup is free; the tools are switched on by a Connect or Connect + Voice plan (7-day free trial for new customers, card required).
- An MCP client that supports remote servers over Streamable HTTP.

## Connection details

- URL: `https://acriumautomated.com/mcp`
- Transport: Streamable HTTP
- Auth: OAuth 2.1 (PKCE + dynamic client registration). Clients that can't do an OAuth sign-in can use a personal token in an `Authorization: Bearer` header instead.

## Step 1: Add the server to the MCP settings

Add this entry to the client's MCP settings file (for Cline, `cline_mcp_settings.json`), merging it into any existing `mcpServers`:

```json
{
  "mcpServers": {
    "acrium": {
      "type": "streamableHttp",
      "url": "https://acriumautomated.com/mcp",
      "disabled": false
    }
  }
}
```

## Step 2: Sign in (OAuth)

1. When the client first connects, the server answers with a 401 pointing at https://acriumautomated.com/.well-known/oauth-protected-resource. The client registers itself and opens Acrium's sign-in page in a browser.
2. The user signs in to Acrium.
3. The user picks the workspaces the client may use and unticks any permission they don't want it to have.
4. The user clicks **Allow access**. The client is connected and refreshes its own sign-in from then on.

### If the client can't open a sign-in window

Ask the user to make a personal token in the Acrium dashboard under **Settings → API → AI apps (MCP)** (or from https://acriumautomated.com/connect?ai=other). It is shown once. Then add it as a header:

```json
{
  "mcpServers": {
    "acrium": {
      "type": "streamableHttp",
      "url": "https://acriumautomated.com/mcp",
      "headers": {
        "Authorization": "Bearer acrm_mcp_pat_..."
      },
      "disabled": false
    }
  }
}
```

Never write a real token into a file that gets committed or shared.

## Step 3: Verify

1. Call `list_projects`. It should return the workspaces the user granted and what the client may change in each.
2. Call a read-only tool such as `list_notes` or `get_attention_items`. A normal response (even an empty list) means the connection works.

If writes fail with a permission error, the user didn't grant that permission when connecting; they can reconnect and grant it. Anything that can start or raise ad spend waits for the owner's approval in the Acrium dashboard; that is expected, not an error.

## More

- Setup guide for every app: https://acriumautomated.com/connect
- Docs: https://acriumautomated.com/docs
- Every tool and its input schema: https://acriumautomated.com/.well-known/mcp/server-card.json

<p align="center"><img src="logo-400.png" alt="Acrium logo" width="120" height="120"></p>

# acrium-mcp

Acrium MCP server: let your AI run your business.

> **This repo is documentation only.** The Acrium MCP server is hosted at `https://acriumautomated.com/mcp`. There's nothing here to install, build or self-host. This repo exists so MCP directories and clients have a public place to point at.

## What Acrium is

Acrium plugs the AI a business already uses (Claude, ChatGPT, Grok, Meta AI/Muse, Cursor, Claude Code) into real business tools through a remote MCP server. Once connected, that AI can:

- **Run Google Ads**: create campaigns, ad groups, search and display ads and keywords; set geo targeting; read campaign stats; pause or enable campaigns.
- **Post and schedule to social media**: TikTok, YouTube, X, Pinterest, Bluesky, Facebook, Instagram, Threads and LinkedIn.
- **Manage a Shopify store and inventory**: products, discounts, photos, orders and reports; inventory and SKU sync.
- **Make and answer phone calls** (Connect + Voice plan): a real phone number for the workspace. The AI places calls from a brief, and incoming calls are answered with instructions the owner sets. Transcripts go back to the AI.

The customer's own AI does the thinking. **Anything that can start or raise ad spend waits for the owner's approval** in the Acrium dashboard, whichever AI asked; pausing never waits. Ad spend is billed by Google to the business's own account. The Approval Gate can also hold posts, store changes and emails, and TikTok posts always need a person's approval.

A connection never does more than the person who made it: the AI works as you, in the workspaces you pick, with the permissions you leave on.

Acrium is new and in early access. It's built and run by its founder, Ethyn Malin, who runs his own software business on it and is its only user so far. There are no customer reviews, ratings or user counts yet.

## Connection details

| | |
|---|---|
| Server URL | `https://acriumautomated.com/mcp` |
| Transport | Streamable HTTP |
| Auth | OAuth 2.1 with PKCE and dynamic client registration. Header-only clients use a personal token (`Authorization: Bearer ...`) |
| OAuth metadata | An unauthenticated request gets a 401 pointing at [`/.well-known/oauth-protected-resource`](https://acriumautomated.com/.well-known/oauth-protected-resource) |
| Server card (every tool + input schema, no sign-in) | [`/.well-known/mcp/server-card.json`](https://acriumautomated.com/.well-known/mcp/server-card.json) |
| Official MCP Registry name | `com.acriumautomated/acrium` |

You need an Acrium account: sign up at [acriumautomated.com/setup](https://acriumautomated.com/setup). The full setup guide, with step-by-step instructions per app, is at [acriumautomated.com/connect](https://acriumautomated.com/connect).

## Setup by client

In every app that supports sign-in, the flow is the same: add Acrium as a custom connector (remote MCP server), sign in to Acrium, pick the workspaces it may use and the permissions it gets, then approve.

### Claude (claude.ai, desktop app)

1. Open **Customize → Connectors**, click **+ Add**, then **Add custom connector**.
2. Name it **Acrium**, paste `https://acriumautomated.com/mcp`, click **Continue**, keep the sign-in settings it finds, then click **Add**.
3. Sign in to Acrium, pick your workspace and what it may change, and click **Allow access**.
4. In a chat, switch it on under **+ → Connectors**.

Claude free plans allow one custom connector. On Team or Enterprise, an Owner adds it first under Organization settings → Connectors. Guide: [acriumautomated.com/connect?ai=claude](https://acriumautomated.com/connect?ai=claude)

### ChatGPT

1. Turn on **Developer mode** in ChatGPT's settings.
2. In **Apps & Connectors**, create a custom MCP server: name it **Acrium**, paste `https://acriumautomated.com/mcp`, set Authentication to **OAuth**.
3. Sign in to Acrium, pick your workspace and permissions, and click **Allow access**.
4. In a new chat, add Acrium from the **+** menu.

**Plan limits:** OpenAI currently lists full write access through ChatGPT custom connectors for Business, Enterprise and Edu. Guide: [acriumautomated.com/connect?ai=chatgpt](https://acriumautomated.com/connect?ai=chatgpt)

### Cursor

Settings → MCP → Add new MCP server, with the address as its `url`. In `mcp.json`:

```json
{
  "mcpServers": {
    "acrium": {
      "url": "https://acriumautomated.com/mcp"
    }
  }
}
```

### Claude Code

```bash
claude mcp add --transport http acrium https://acriumautomated.com/mcp
```

Or, with a personal token (made in the Acrium dashboard under Settings → API → AI apps (MCP)):

```bash
claude mcp add --transport http acrium https://acriumautomated.com/mcp \
  --header "Authorization: Bearer acrm_mcp_pat_..."
```

### Grok, Meta AI (Muse) and other apps

- **Grok**: go to grok.com/connectors → **New Connector** → **Custom**, paste the address, sign in. Guide: [connect?ai=grok](https://acriumautomated.com/connect?ai=grok)
- **Grok Bot** (header only): make a personal token, then add a custom MCP server under Settings → Plugins with the address and the token header. Guide: [connect?ai=grok_bot](https://acriumautomated.com/connect?ai=grok_bot)
- **Meta AI (Muse)**: Settings → Connectors → Add custom connector, paste the address, sign in. Guide: [connect?ai=muse](https://acriumautomated.com/connect?ai=muse)
- **Anything else**: look for "custom connector", "remote MCP server" or "add MCP server", paste the address, pick Streamable HTTP if asked, and sign in. If the app only takes headers, use a personal token. Guide: [connect?ai=other](https://acriumautomated.com/connect?ai=other)

AI agents (e.g. Cline) can follow [llms-install.md](llms-install.md).

## Tools by area

The full, current list with input schemas is in the public [server card](https://acriumautomated.com/.well-known/mcp/server-card.json). Highlights:

| Area | Examples |
|---|---|
| Workspaces | `list_projects`, `switch_project`, `get_workspace` |
| Google Ads | `google_ads_create_campaign`, `google_ads_create_search_ad`, `google_ads_add_keywords`, `google_ads_set_geo_targeting`, `google_ads_campaign_stats`, `google_ads_pause_campaign` |
| Social posting | `publish_post`, `schedule_post`, `list_scheduled_posts`, `cancel_scheduled_post` |
| Shopify | `shopify_list_products`, `shopify_create_product`, `shopify_create_discount`, `shopify_list_orders`, `shopify_get_report` |
| Inventory | `list_inventory`, `adjust_inventory_stock`, `add_sku_alias` |
| Calling (Connect + Voice) | `provision_phone_number`, `place_call`, `list_calls`, `set_inbound_instructions` |
| Notes & attention queue | `list_notes`, `search_notes`, `add_note`, `get_attention_items` |
| Video & voiceovers | `make_video`, `get_render_status`, `generate_voiceover` |
| Roles | `list_roles`, `send_message`, `read_messages` |
| Approvals & control | `list_pending_actions`, `decide_pending_action`, `set_approval_gate`, `set_test_mode` |

Reads work on any connection. Writes depend on the permissions granted when connecting, checked on every call.

## Plans

- **Connect**: free for 7 days, then $9.99/month: everything above except calling and voiceovers. 30 minutes of video rendering a month, 5 GB of storage. No AI credits needed: the customer's own AI does the thinking.
- **Connect + Voice**: free for 7 days, then $29.99/month: everything in Connect, plus a phone number, calls both ways and AI voiceovers. 12,000 AI credits a month pay for calls and voiceovers (1,000 during the free trial). 60 minutes of video rendering a month, 20 GB of storage.

New customers get a 7-day free trial on either plan, once per customer; a card is required at checkout, and cancelling in Billing before the trial ends means no charge. After that, monthly, cancel any time. Account setup is free.

## Links

- Setup guide: [acriumautomated.com/connect](https://acriumautomated.com/connect)
- Docs: [acriumautomated.com/docs](https://acriumautomated.com/docs)
- Website: [acriumautomated.com](https://acriumautomated.com/)
- Privacy Policy: [acriumautomated.com/privacy](https://acriumautomated.com/privacy)
- Terms of Service: [acriumautomated.com/terms](https://acriumautomated.com/terms)
- Contact: [malin.ethyn@acriumautomated.com](mailto:malin.ethyn@acriumautomated.com)

## License

The documentation in this repo is MIT licensed (see [LICENSE](LICENSE)). The Acrium service itself is governed by its [Terms of Service](https://acriumautomated.com/terms).

# Publishing to the MCP Registry

Acrium is listed in the official MCP Registry as `com.acriumautomated/acrium`. Publishing proves we own `acriumautomated.com` with a DNS TXT record and a matching Ed25519 private key. The old private key is lost, so this is the one-time setup for a new one. After that, publishing is a single button press.

**The private key is only ever created and stored on your own machine, and in the GitHub secret.** Never paste it into chat, an issue, a commit or a doc.

You need OpenSSL 3.0 or later (`openssl version`). On macOS the built-in LibreSSL can't make Ed25519 keys: run `brew install openssl@3` and use `$(brew --prefix openssl@3)/bin/openssl` wherever it says `openssl` below.

## 1. Generate a new key pair (on your machine)

Work in a folder **outside** this repo, e.g. `~/.acrium-registry`:

```bash
mkdir -p ~/.acrium-registry && cd ~/.acrium-registry
umask 077

# Private key (stays on your machine)
openssl genpkey -algorithm Ed25519 -out key.pem

# Print the TXT record value (this is the PUBLIC key; safe to show)
PUBLIC_KEY="$(openssl pkey -in key.pem -pubout -outform DER | tail -c 32 | base64)"
echo "v=MCPv1; k=ed25519; p=${PUBLIC_KEY}"
```

Back up `key.pem` somewhere safe (a password manager). If you lose it again, you just repeat this guide.

## 2. Put the TXT record on acriumautomated.com

In the DNS provider for `acriumautomated.com`:

1. **Delete any existing TXT record on the apex that starts with `v=MCPv1`.** That's the old key. The registry tries a stale record first, and it will make login fail.
2. Add a new TXT record:

   | Field | Value |
   |---|---|
   | Type | `TXT` |
   | Name / Host | `@` (the apex, `acriumautomated.com` itself). **Not** `_mcp-auth` or any other subdomain. |
   | Value | `v=MCPv1; k=ed25519; p=<PUBLIC_KEY from step 1>` |
   | TTL | default (or 300) |

   As a zone-file line it looks like:
   `acriumautomated.com. IN TXT "v=MCPv1; k=ed25519; p=<PUBLIC_KEY>"`

3. Leave other TXT records (SPF, Google verification and so on) alone.
4. Check that it's live (this can take a few minutes):

   ```bash
   dig +short TXT acriumautomated.com | grep MCPv1
   ```

   You should see exactly **one** `v=MCPv1` line, and it should have the new key.

## 3. Save the private key as the repo secret `MCP_DNS_PRIVATE_KEY`

The registry wants the raw private key as hex. With the GitHub CLI, you can pipe it straight into the secret so it never shows up on screen:

```bash
cd ~/.acrium-registry
openssl pkey -in key.pem -noout -text | grep -A3 "priv:" | tail -n +2 | tr -d ' :\n' \
  | gh secret set MCP_DNS_PRIVATE_KEY --repo ehtyn/acrium-mcp
```

Without the CLI: run the same command minus the `| gh secret set ...` line, copy the 64-character hex it prints, then go to **GitHub → ehtyn/acrium-mcp → Settings → Secrets and variables → Actions → New repository secret**, name it `MCP_DNS_PRIVATE_KEY`, paste the value and save. Then clear your terminal.

## 4. Run the workflow

1. Merge the PR that contains `server.json` at the version you want to publish (currently `1.0.2`).
2. Go to **GitHub → ehtyn/acrium-mcp → Actions → "Publish to MCP Registry" → Run workflow**, on `main`.
3. Pick what happens to the old **v1.0.0** listing (it points at a private repo):
   - `deprecated` (default): still visible, with a warning
   - `deleted`: hidden from default listings
   - `leave alone`
4. Click **Run workflow**. It validates `server.json`, logs in with the DNS key, publishes, updates v1.0.0, then prints what the registry now lists.

Check the result:

```bash
curl -s "https://registry.modelcontextprotocol.io/v0.1/servers?search=com.acriumautomated/acrium" | jq '.servers[].server.version'
```

## Next releases

Bump `version` in `server.json` (every publish needs a new version; you can't republish an old one), merge, then run the workflow again with `leave alone`. The key and TXT record stay the same.

## Troubleshooting

| Error | Fix |
|---|---|
| `MCP_DNS_PRIVATE_KEY is not set` | Do step 3. |
| DNS / signature verification failed | The record isn't live yet, an old `v=MCPv1` record is still there, or the secret came from a different `key.pem`. Recheck step 2 with `dig`. |
| Version already exists | Bump `version` in `server.json`. |
| `invalid audience` | `mcp-publisher` is too old. Bump `MCP_PUBLISHER_VERSION` in `.github/workflows/publish-registry.yml`. |

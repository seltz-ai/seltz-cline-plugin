# Seltz for Cline

<img src="logo.png" alt="Seltz" width="96" height="96">

Seltz gives Cline source-backed results from the public web: general search, news, company profiles, LinkedIn and other public people profiles, financial reports, and Wikipedia. When you already have a URL, Seltz fetches the page as readable text.

This repository holds the setup instructions for Seltz's hosted MCP server. There is nothing to clone or run locally.

New accounts receive [$20 in free request credit](https://docs.seltz.ai/pricing?utm_source=cline&utm_medium=integration&utm_campaign=marketplace).

- Product: [seltz.ai](https://seltz.ai/?utm_source=cline&utm_medium=integration&utm_campaign=marketplace)
- Console: [console.seltz.ai](https://console.seltz.ai/?utm_source=cline&utm_medium=integration&utm_campaign=marketplace)
- MCP reference: [docs.seltz.ai/mcp-server](https://docs.seltz.ai/mcp-server?utm_source=cline&utm_medium=integration&utm_campaign=marketplace)

## Example prompts

> Find the latest official guidance on EU AI Act transparency obligations and link the primary sources.

> Look up Stripe and its current CEO, including the CEO's public professional background.

> Fetch the European Commission's page on the AI Act and summarize the transparency obligations for providers.

## Install

| Setting | Value |
| --- | --- |
| URL | `https://mcp.seltz.ai/mcp` |
| Transport | Streamable HTTP |
| Sign in | Seltz Console, through OAuth |
| API key (optional) | Header `Authorization: Bearer sk_...` |

### Sign in (default)

In the Cline extension:

1. Open **MCP Servers** and go to the **Remote Servers** tab.
2. Enter `seltz` as the server name and `https://mcp.seltz.ai/mcp` as the server URL.
3. Choose **Streamable HTTP** and click **Add Server**.
4. On the Seltz server's row, click **Authenticate**. The Seltz Console opens in your browser. Pick an organization and click **Allow**.

Or add the server to Cline's MCP settings file (`cline_mcp_settings.json`) under `mcpServers`, keeping your other servers, then click **Authenticate** as above:

```json
{
  "mcpServers": {
    "seltz": {
      "type": "streamableHttp",
      "url": "https://mcp.seltz.ai/mcp",
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Set `"type": "streamableHttp"`. Without it, Cline falls back to the legacy SSE transport, and Seltz serves only Streamable HTTP.

In the Cline CLI:

```bash
cline mcp add seltz https://mcp.seltz.ai/mcp --transport http --yes
cline mcp
```

In the `cline mcp` menu, choose **Authorize OAuth** for `seltz`.

The authorization scope the server advertises is `seltz:read`. You don't need an API key for this connection, and you should not add an `Authorization` header. Cline sends the token from the sign-in.

### API key (fallback)

Use a key when sign-in isn't possible, such as on a machine without a browser. Create one in the [Seltz Console](https://console.seltz.ai/api-keys?utm_source=cline&utm_medium=integration&utm_campaign=marketplace). Seltz API keys start with `sk_`. Add a `headers` block to the `seltz` entry:

```json
{
  "mcpServers": {
    "seltz": {
      "type": "streamableHttp",
      "url": "https://mcp.seltz.ai/mcp",
      "headers": {
        "Authorization": "Bearer sk_your_key"
      },
      "disabled": false,
      "autoApprove": []
    }
  }
}
```

Keep `cline_mcp_settings.json` out of chat, screenshots, and version control. It holds your key, or after sign-in, Cline's tokens for Seltz under the entry's `oauth` key.

## Tools

The server exposes `search` and `fetch`. Both are read-only and return Markdown. The live tool schema from the server is the authority if it adds a parameter or a tool.

### `search`

Searches Seltz's index. Each result includes its URL, a publication date when the source has one, snippets, and properties. Results do not include the full page, so call `fetch` on a URL when you need the page.

| Argument | Required | Use |
| --- | --- | --- |
| `query` | yes | The search query. |
| `scope` | no | Which dataset to search. Leave it unset for a general query. |
| `max_results` | no | How many results to return. The default is 10. The server lowers a higher value to 50. |
| `include_domains`, `exclude_domains` | no | Lists of up to 100 domains each. |
| `from_date`, `to_date` | no | Publication-date window, in UTC. A date (`2025-10-28`), a datetime (`2025-10-28T23:00:00Z`), or an offset such as `now-7d`. Not accepted for `companies` or `people`. |

| Scope | What it covers |
| --- | --- |
| `web` | General web pages, including Wikipedia. Currently in beta. |
| `news` | News articles. |
| `companies` | Public company profiles: name, industry, location, and company details. |
| `people` | Public professional profiles, including LinkedIn: name, role, company, and career history. |
| `financial-reports` | SEC filings from public companies (10-K, 10-Q, 8-K, and related forms), returned as searchable chunks. |
| `wikipedia` | Wikipedia articles. |

### `fetch`

Returns the full content of 1 to 5 pages as Markdown, with navigation, footers, and ads removed.

| Argument | Required | Use |
| --- | --- | --- |
| `urls` | yes | Absolute URLs. One to five, with no duplicates. |

`fetch` reads HTML and plain text. PDFs, images, and other binaries come back as `unsupported_content_type`. One call returns at most 100,000 characters of page text, split across the pages.

## What connecting Seltz sends

- The queries you ask Seltz to search, and the page URLs you ask it to fetch, go to `https://mcp.seltz.ai/mcp` over HTTPS.
- Sign-in also contacts `https://console.seltz.ai`, the authorization server Seltz publishes for that endpoint. Cline handles the browser flow and stores the tokens in `cline_mcp_settings.json`, under the entry's `oauth` key.
- With an API key, Cline sends `Authorization: Bearer` and the key from its settings file.
- Calls use your Seltz organization's credit.

## Troubleshooting

**Sign-in.** If Seltz shows as not authenticated, click **Authenticate** on its row again, or choose **Authorize OAuth** in `cline mcp`. An organization admin can also block MCP access from the Console. They lift it under **Connected apps → Organization**.

**API key.** If the server rejects the key, check that the header is `Bearer`, a space, then the key, and that the key starts with `sk_`.

**Won't connect.** Check that the entry has `"type": "streamableHttp"` and the URL ends in `/mcp`.

**No credit.** If the server refuses calls because the organization is out of credit, add credit under **Billing** in the [Seltz Console](https://console.seltz.ai/?utm_source=cline&utm_medium=integration&utm_campaign=marketplace). Signing in again does not add credit.

# Installing Seltz in Cline

Seltz is a hosted MCP server. There is nothing to clone, build, or run. Add one remote server entry, then have the user sign in.

## 1. Add the server

If Cline gave you a server name for this install, use it. Otherwise use `seltz`. The steps below call it "this server".

In Cline's MCP settings file (`cline_mcp_settings.json`), under `mcpServers`, an entry keeps its transport settings either at the top level (`type`, `url`) or under a `transport` key (`transport.type`, `transport.url`), which is what `cline mcp add` writes. Check and change them wherever the entry keeps them.

- If Cline already shows this server as connected, with the tools `search` and `fetch`, change nothing and go to step 3.
- If another entry already has the URL `https://mcp.seltz.ai/mcp`, tell the user instead of adding a second one.
- If this server's entry already has type `streamableHttp` and URL `https://mcp.seltz.ai/mcp`, leave it as it is and go to step 2. It may hold the user's sign-in (an `oauth` key) or API key (`headers`), and changing it can sign them out. If it has `"disabled": true`, tell the user it is turned off instead of changing it.
- If the entry exists with other values, set only its type and URL to the values below and keep its other keys.
- Otherwise add the entry below, with this server's name as the key:

```json
"seltz": {
  "type": "streamableHttp",
  "url": "https://mcp.seltz.ai/mcp",
  "disabled": false,
  "autoApprove": []
}
```

Keep every other server in the file as it is.

- `"type": "streamableHttp"` is required. Without it, Cline falls back to the legacy SSE transport, and Seltz serves only Streamable HTTP.
- Do not add a `headers` block and do not ask the user for an API key. The server signs users in through OAuth.
- Leave `autoApprove` empty unless the user asks otherwise.

## 2. Ask the user to sign in

Signing in needs the user's browser, so you can't finish it yourself. Tell the user:

- In the Cline extension: open **MCP Servers**, find this server's row, and click **Authenticate**.
- In the Cline CLI: run `cline mcp`, then choose **Authorize OAuth** for this server.

The Seltz Console opens in the browser, where the user signs in and clicks **Allow**.

Then ask the user to tell you when they have signed in. Until they do, don't call a Seltz tool and don't edit the entry. A tool call before sign-in fails, and that failure doesn't mean the configuration is wrong.

## 3. Check the connection

After the user confirms, the server lists two tools: `search` and `fetch`. Run one small search to confirm it works, for example `search` with `query` set to `Seltz MCP server` and `max_results` set to `1`. Each call uses a little of the user's credit.

## API key instead of sign-in

Only if the user asks for an API key, or can't use a browser on this machine. Don't ask for the key in chat, and don't write a placeholder key into the file: Cline would connect with it and fail. Show the user this block and ask them to add it inside this server's entry themselves, with a key from <https://console.seltz.ai/api-keys> (Seltz keys start with `sk_`):

```json
"headers": {
  "Authorization": "Bearer sk_your_key"
}
```

Wait for the user to confirm they saved it, then go to step 3.

## If it fails

- Not authenticated: the user clicks **Authenticate** again, or chooses **Authorize OAuth** in `cline mcp`.
- Blocked by an organization admin: the user asks the admin to lift the block in the Seltz Console.
- Out of credit: the user adds credit under **Billing** at <https://console.seltz.ai>. Signing in again does not help.
- Won't connect: check `"type": "streamableHttp"` and that the URL is exactly `https://mcp.seltz.ai/mcp`.

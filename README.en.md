# CaoLiao QR Code MCP

[中文](README.md)

CaoLiao QR Code ([cli.im](https://cli.im)) provides official remote MCP servers that let AI assistants work with the codes, forms and records in CaoLiao. The servers are hosted by CaoLiao; there is nothing to install or deploy locally.

| | Admin MCP | Scan-side MCP |
|---|---|---|
| Who it is for | People who create and manage codes, or the AI working for them | Someone who scanned a code, or the AI working for them |
| What it does | Create codes one by one or in batches, build forms and recurring checks, query records, inspection progress and scan metrics | Read the public content, history and available forms behind one CaoLiao code |
| Endpoint | `https://open.cli.im/mcp` | `https://mcp.objqr.com/mcp` |
| Transport | Streamable HTTP | Streamable HTTP |
| Authentication | OAuth authorization for an organization, no secret | None |
| Suggested name | `caoliao-admin` | `caoliao-scan` |

An A2A service is available on the scan side: `https://a2a.objqr.com`.

Full documentation: <https://cli.im/open-api/en/agent/overview.html>

## Fastest way: let your AI install it

Send this to your AI. The full prompt, which also installs the companion Skill, is in the [quick start](https://cli.im/open-api/en/agent/open-platform-mcp/quick-start.html).

```text
Help me connect CaoLiao Admin MCP.
MCP name: caoliao-admin
MCP endpoint: https://open.cli.im/mcp
Transport: Streamable HTTP (some clients write the type as http)
Authentication: OAuth. No key or secret to enter.

If you can edit this client's MCP configuration, add the server with the name and endpoint above and keep all existing entries.
If you cannot, tell me how to add it by hand. Do not claim that you added it.
OAuth authorization must be done by me in a browser: I sign in to CaoLiao and choose an organization. Tell me how to start it.
When done, call CaoLiao MCP to confirm the connected organization. Verify only; do not create or edit any data.
```

## Admin MCP: client configuration

**WorkBuddy, QwenWork**: search for "草料二维码" in the connector market. Guides: [WorkBuddy](https://cli.im/open-api/en/agent/open-platform-mcp/workbuddy.html), [QwenWork](https://cli.im/open-api/en/agent/open-platform-mcp/qwen-work.html), [Doubao Work](https://cli.im/open-api/en/agent/open-platform-mcp/doubao-work.html).

**Claude Code**

```bash
claude mcp add --transport http caoliao-admin https://open.cli.im/mcp
```

Then type `/mcp` in Claude Code and select `caoliao-admin` to authorize.

**Cursor**: add to `~/.cursor/mcp.json`

```json
{
  "mcpServers": {
    "caoliao-admin": {
      "url": "https://open.cli.im/mcp"
    }
  }
}
```

**VS Code**: add to `.vscode/mcp.json`

```json
{
  "servers": {
    "caoliao-admin": {
      "type": "http",
      "url": "https://open.cli.im/mcp"
    }
  }
}
```

**Claude on the web and desktop**: in Settings → Connectors, add a custom connector with URL `https://open.cli.im/mcp`.

**Other clients**

```json
{
  "mcpServers": {
    "caoliao-admin": {
      "type": "streamable-http",
      "url": "https://open.cli.im/mcp"
    }
  }
}
```

Some clients use `http` for `type`.

After adding the server, complete OAuth when the client prompts you: sign in to CaoLiao and choose the organization to authorize. To verify, ask the AI:

```text
Use the CaoLiao MCP to confirm my connected organization and available capabilities. Verify only; do not create or edit data.
```

### Authorization

Admin MCP follows the MCP authorization specification (OAuth 2.1). Clients that support discovery only need the endpoint.

| Item | Value |
|---|---|
| Protected resource metadata | `https://open.cli.im/.well-known/oauth-protected-resource` |
| Authorization server metadata | `https://oauth.cli.im/.well-known/oauth-authorization-server` |
| Client registration | Dynamic client registration (RFC 7591); no application needed |
| Grant types | Authorization code with PKCE (S256 only), refresh token, device code (RFC 8628) |
| Scopes | `mcp:tools`, `records.read`, `templates.read`, `templates.write`, `qrcodes.write`, `assets.write` |

Authorization uses your existing permissions in the selected organization and does not expand them. To revoke, open **Global Settings → Security & Traceability → Authorization Management** in the CaoLiao console.

See [Developer setup](https://cli.im/open-api/en/agent/open-platform-mcp.html) for details and [Tools](https://cli.im/open-api/en/agent/open-platform-mcp/tools.html) for the tool list.

### Companion Skill

Admin MCP has a companion Agent Skill that guides the AI through preparing data, uploading files and creating codes in batches: [caoliao/caoliao-admin-skills](https://github.com/caoliao/caoliao-admin-skills).

```bash
npx skills add caoliao/caoliao-admin-skills --skill caoliao-admin
```

## Scan-side MCP

No login. Add the endpoint and use it; there are 5 read-only tools.

**Claude Code**

```bash
claude mcp add --transport http caoliao-scan https://mcp.objqr.com/mcp
```

**Other clients**

```json
{
  "mcpServers": {
    "caoliao-scan": {
      "type": "http",
      "url": "https://mcp.objqr.com/mcp"
    }
  }
}
```

Verify with curl:

```bash
curl -X POST https://mcp.objqr.com/mcp \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"demo","version":"0"}}}'
```

Docs: [Scan-side quick start](https://cli.im/open-api/en/agent/public-mcp/quick-start.html), [Tools](https://cli.im/open-api/en/agent/public-mcp/tools.html), [Data scope and authorization](https://cli.im/open-api/en/agent/public-mcp/data-scope.html), [A2A](https://cli.im/open-api/en/agent/a2a/quick-start.html).

## What is in this repository

This repository holds the description and connection configuration for the CaoLiao MCP servers. The server code is not here.

```text
servers/
├── caoliao-admin/server.json   # Admin MCP server description
└── caoliao-scan/server.json    # Scan-side MCP server description
```

`server.json` uses the MCP Registry format. The servers are not yet published to the official Registry.

## Links

- [CaoLiao Open Platform docs](https://cli.im/open-api/en/)
- [API reference](https://cli.im/open-api/en/openapi/v2/overview.html) and [openapi.json](https://open.cli.im/openapi.json)
- [caoliao/caoliao-admin-skills](https://github.com/caoliao/caoliao-admin-skills): Admin Skill
- [caoliao/deepscan-skills](https://github.com/caoliao/deepscan-skills): DeepScan skills
- [caoliao/API](https://github.com/caoliao/API): Open Platform overview

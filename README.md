# 草料二维码 MCP

[English](README.en.md)

草料二维码（[cli.im](https://cli.im)）提供官方的远程 MCP 服务，让 AI 助手直接使用草料里的码、表单和记录。服务由草料托管，不需要本地安装或部署。

| | 管理端 MCP | 扫码端 MCP |
|---|---|---|
| 给谁用 | 做码、管码的人，或替他干活的 AI | 扫到一张码的人，或替他干活的 AI |
| 能做什么 | 建码、批量建码、搭表单和巡检计划，查询记录、巡检进度和扫码统计 | 读取一张草料码的公开内容、履历和可填表单 |
| 服务地址 | `https://open.cli.im/mcp` | `https://mcp.objqr.com/mcp` |
| 传输方式 | Streamable HTTP | Streamable HTTP |
| 鉴权 | OAuth 授权组织，不需要密钥 | 不需要登录 |
| 推荐名称 | `caoliao-admin` | `caoliao-scan` |

扫码端同侧还提供 A2A 服务：`https://a2a.objqr.com`。

完整文档在草料开放平台：<https://cli.im/open-api/agent/overview.html>

## 最快的方式：让 AI 自己安装

把下面这段话发给你的 AI。完整版提示词（包含配套 Skill 的安装）见[快速开始](https://cli.im/open-api/agent/open-platform-mcp/quick-start.html)。

```text
请帮我接入草料二维码管理端 MCP。
MCP 名称：caoliao-admin
MCP 地址：https://open.cli.im/mcp
传输方式：Streamable HTTP（部分客户端的 type 写作 http）
鉴权方式：OAuth，不需要填写任何密钥

如果你能修改当前客户端的 MCP 配置，就按上面的名称和地址添加，保留其他已有配置；
如果不能，请告诉我该怎么手动添加，不要假装已添加。
OAuth 授权需要我本人在浏览器里登录草料账号并选择组织，请告诉我如何发起授权。
完成后调用草料 MCP 确认当前连接的组织，只验证，不创建或修改任何数据。
```

## 管理端 MCP：各客户端的配置

**WorkBuddy、千问办公**：在应用市场搜索「草料二维码」安装。图文步骤见 [WorkBuddy](https://cli.im/open-api/agent/open-platform-mcp/workbuddy.html)、[千问办公](https://cli.im/open-api/agent/open-platform-mcp/qwen-work.html)、[豆包工作](https://cli.im/open-api/agent/open-platform-mcp/doubao-work.html)。

**Claude Code**

```bash
claude mcp add --transport http caoliao-admin https://open.cli.im/mcp
```

然后在 Claude Code 里输入 `/mcp`，选择 `caoliao-admin` 完成授权。

**Cursor**：在 `~/.cursor/mcp.json` 里加入

```json
{
  "mcpServers": {
    "caoliao-admin": {
      "url": "https://open.cli.im/mcp"
    }
  }
}
```

**VS Code**：在 `.vscode/mcp.json` 里加入

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

**Claude 网页版和桌面版**：在「设置 → 连接器」里添加自定义连接器，地址填 `https://open.cli.im/mcp`。

**其他客户端**

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

`type` 的取值因客户端而异，部分客户端使用 `http`。

添加后按客户端提示完成 OAuth 授权：登录草料账号，选择要授权的组织。可以对 AI 说下面这句话验证：

```text
请使用草料 MCP，确认我连接的组织和可用能力。只验证，不创建或修改数据。
```

### 授权机制

管理端 MCP 按 MCP 授权规范（OAuth 2.1）实现，支持自动发现的客户端只需要服务地址。

| 项目 | 内容 |
|---|---|
| 资源元数据 | `https://open.cli.im/.well-known/oauth-protected-resource` |
| 授权服务器元数据 | `https://oauth.cli.im/.well-known/oauth-authorization-server` |
| 客户端注册 | 动态注册（RFC 7591），不需要事先申请 |
| 授权方式 | 授权码 + PKCE（只支持 S256）、刷新令牌、设备码（RFC 8628） |
| scope | `mcp:tools`、`records.read`、`templates.read`、`templates.write`、`qrcodes.write`、`assets.write` |

授权使用你在所选组织里的现有权限，不会扩大权限。撤销授权：草料管理中心「全局设置 → 安全与溯源 → 授权管理」。

详细说明见[开发者接入](https://cli.im/open-api/agent/open-platform-mcp.html)，工具清单见[工具列表](https://cli.im/open-api/agent/open-platform-mcp/tools.html)。

### 配套 Skill

管理端有配套的 Agent Skill，指导 AI 整理资料、上传文件和批量建码：[caoliao/caoliao-admin-skills](https://github.com/caoliao/caoliao-admin-skills)。

```bash
npx skills add caoliao/caoliao-admin-skills --skill caoliao-admin
```

## 扫码端 MCP

不需要登录，添加地址即可使用，共 5 个只读工具。

**Claude Code**

```bash
claude mcp add --transport http caoliao-scan https://mcp.objqr.com/mcp
```

**其他客户端**

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

用 curl 验证：

```bash
curl -X POST https://mcp.objqr.com/mcp \
  -H 'content-type: application/json' \
  -H 'accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"demo","version":"0"}}}'
```

文档：[扫码端快速开始](https://cli.im/open-api/agent/public-mcp/quick-start.html)、[工具列表](https://cli.im/open-api/agent/public-mcp/tools.html)、[数据范围与授权](https://cli.im/open-api/agent/public-mcp/data-scope.html)、[A2A 接入](https://cli.im/open-api/agent/a2a/quick-start.html)。

## 本仓库的内容

本仓库是草料 MCP 服务的说明和接入配置，服务端代码不在这里。

```text
servers/
├── caoliao-admin/server.json   # 管理端 MCP 的服务描述
└── caoliao-scan/server.json    # 扫码端 MCP 的服务描述
```

`server.json` 采用 MCP Registry 的格式，尚未发布到官方 Registry。

## 相关链接

- [草料开放平台文档](https://cli.im/open-api/)
- [API 参考](https://cli.im/open-api/openapi/v2/overview.html) 和 [openapi.json](https://open.cli.im/openapi.json)
- [caoliao/caoliao-admin-skills](https://github.com/caoliao/caoliao-admin-skills)：管理端 Skill
- [caoliao/deepscan-skills](https://github.com/caoliao/deepscan-skills)：草料扫码器技能包
- [caoliao/API](https://github.com/caoliao/API)：开放平台说明

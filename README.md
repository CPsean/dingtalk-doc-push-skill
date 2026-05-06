# dingtalk-doc-push

[English](#english) | [中文](#中文)

---

## 中文

**dingtalk-doc-push** 是一个 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 技能（Skill），通过钉钉文档官方 MCP Server 推送、同步和管理钉钉云端文档。

### 功能

- **推送**：将本地 Markdown 文件创建为钉钉文档
- **更新**：覆盖或追加内容到已有文档
- **拉取**：将云端文档内容下载为本地 Markdown 文件
- **列表**：浏览知识库或文件夹下的文档
- **搜索**：按关键词搜索文档
- **管理**：创建文件夹、重命名、移动、复制、删除文档

所有操作通过[钉钉官方 MCP Server](https://aihub.dingtalk.com/#/detail?mcpId=9629&detailType=marketMcpDetail) 完成，无需本地脚本，API Key 不会写入代码。

### 快速开始

**1. 安装 Skill**

将 `.claude/skills/dingtalk-doc-push` 目录复制到你的项目的 `.claude/skills/` 下（或用户级 `~/.claude/skills/`）。

**2. 开通钉钉文档 MCP 服务**

首次使用时，Skill 会引导你完成配置：

1. 打开[钉钉 AI Hub MCP 页面](https://aihub.dingtalk.com/#/detail?mcpId=9629&detailType=marketMcpDetail)，登录钉钉账号，点击【开通服务】
2. 复制页面右侧的 **StreamableHttp URL**（不是 JSON Config）
3. 将 URL 粘贴给 Claude

Skill 会自动注册 MCP 服务。Claude Code 客户端执行：

```bash
claude mcp add --transport http --scope user dingtalk-doc "<你的 URL>"
```

其他客户端（Claude Desktop、Cursor 等）需手动配置 StreamableHttp 类型的 MCP 服务。

**3.（可选）设置默认知识库**

提供你常用的知识库 URL，后续操作无需每次指定：

```
https://alidocs.dingtalk.com/i/spaces/xxxxx/overview
```

### 使用示例

直接用自然语言和 Claude 对话即可：

- *"把 `docs/design.md` 推送到我的钉钉知识库"*
- *"用本地最新版本更新钉钉上的 PRD 文档"*
- *"列出我知识库里的所有文档"*
- *"在钉钉文档里搜索「认证」"*
- *"把会议纪要从钉钉拉取到本地"*
- *"删除钉钉上的测试文档"*

### 工作流程

```
用户请求
  → Skill 激活（通过 description 触发词匹配）
  → MCP 工具映射（create / update / list / search / delete ...）
  → 执行前确认（所有写操作需用户确认）
  → 钉钉文档 MCP Server
  → 返回结果：文档标��� + 链接
```

### 已知限制

| 限制项 | 详情 |
|---|---|
| **Mermaid 流程图** | ` ```mermaid ` 代码块推送后显示为普通代码块，而非钉钉原生「文本绘图」。推送后需在钉钉编辑器中手动转换。 |
| **对话上传文件编码** | 非 ASCII 文件（中文等）通过对话上传时可能出现不可逆乱码。请始终提供**本地文件路径**。 |
| **文档类型** | `create_document` 和 `update_document` 仅支持 **adoc**（文字类型）文档。 |
| **删除** | 移入回收站（30 天内可恢复），非永久删除。 |

### 安全性

- MCP 服务 URL 包含个人 API Key，仅写入本地配置，**不会提交到版本控制**
- 所有写操作执行前都需要用户明确确认

### 文件结构

```
.claude/skills/dingtalk-doc-push/
├── SKILL.md                  # Skill 入口（触发器 + 指令）
├── README.md                 # 本文件
├── evals/
│   ├── evals.json            # 任务执行测评
│   └── trigger-evals.json    # 触发边界测评
└── references/
    └── mcp-tools.md          # MCP 工具速查表
```

---

## English

**dingtalk-doc-push** is a [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill for pushing, syncing, and managing documents on [DingTalk Docs](https://alidocs.dingtalk.com) via the official DingTalk Document MCP Server.

### Features

- **Push**: Create a DingTalk document from local Markdown files
- **Update**: Overwrite or append content to existing documents
- **Pull**: Download cloud document content to a local Markdown file
- **List**: Browse documents in a knowledge base or folder
- **Search**: Find documents by keyword
- **Manage**: Create folders, rename, move, copy, or delete documents

All operations go through [DingTalk's official MCP Server](https://aihub.dingtalk.com/#/detail?mcpId=9629&detailType=marketMcpDetail) — no local scripts or API keys in code.

### Quick Start

**1. Install the Skill**

Copy the `.claude/skills/dingtalk-doc-push` directory into your project's `.claude/skills/` folder (or user-level `~/.claude/skills/`).

**2. Enable the DingTalk Document MCP Server**

On first use, the skill will guide you through setup:

1. Open the [DingTalk AI Hub MCP page](https://aihub.dingtalk.com/#/detail?mcpId=9629&detailType=marketMcpDetail) and activate the service
2. Copy the **StreamableHttp URL** from the page
3. Paste it into the Claude conversation

The skill automatically registers the MCP server. For Claude Code:

```bash
claude mcp add --transport http --scope user dingtalk-doc "<your-url>"
```

For other clients (Claude Desktop, Cursor, etc.), configure a StreamableHttp MCP server manually.

**3. (Optional) Set a Default Knowledge Base**

Provide your knowledge base URL so you don't have to specify it every time:

```
https://alidocs.dingtalk.com/i/spaces/xxxxx/overview
```

### Usage Examples

Just talk to Claude naturally:

- *"Push `docs/design.md` to my DingTalk knowledge base"*
- *"Update the PRD document on DingTalk with the latest local version"*
- *"List all documents in my knowledge base"*
- *"Search DingTalk docs for 'authentication'"*
- *"Pull the meeting notes from DingTalk to local"*
- *"Delete the test document from DingTalk"*

### How It Works

```
User request
  → Skill activation (matched by description trigger)
  → MCP tool mapping (create / update / list / search / delete ...)
  → Pre-execution confirmation (for all write operations)
  → DingTalk Document MCP Server
  → Result: document title + link
```

### Known Limitations

| Limitation | Detail |
|---|---|
| **Mermaid diagrams** | ` ```mermaid ` blocks are pushed as plain code blocks, not DingTalk's native "text drawing" blocks. Manually convert after pushing. |
| **File encoding on chat upload** | Non-ASCII files (Chinese, etc.) uploaded through the chat may suffer irreversible garbled text. Always provide a **local file path**. |
| **Document types** | `create_document` and `update_document` only support **adoc** (text document) type. |
| **Deletion** | Moves to trash (recoverable within 30 days), not permanent deletion. |

### Security

- The MCP server URL contains a personal API key — written to local config only, **never committed to version control**
- All write operations require explicit user confirmation before execution

### File Structure

```
.claude/skills/dingtalk-doc-push/
├── SKILL.md                  # Skill entry point (trigger + instructions)
├── README.md                 # This file
├── evals/
│   ├── evals.json            # Task execution evals
│   └── trigger-evals.json    # Trigger boundary evals
└── references/
    └── mcp-tools.md          # MCP tool reference
```

### License

MIT

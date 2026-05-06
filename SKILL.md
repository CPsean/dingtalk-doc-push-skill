---
name: dingtalk-doc-push
description: 当用户想要将本地文档推送到钉钉文档、更新钉钉云端文档内容、查询或搜索钉钉文档列表时使用本 Skill。适用于：上传 markdown 到钉钉、同步本地文件到钉钉知识库、列出知识库文档、在钉钉中创建文档或文件夹、初始化钉钉文档 MCP 配置。当用户想推送到飞书、语雀、Notion 等其他平台、只操作本地文件、管理钉钉 IM 消息或群组时，不要使用本 Skill。
---

# 钉钉文档推送

通过钉钉文档官方 MCP Server 操作云端文档。支持创建、更新、查询、搜索文档及文件夹管理。

## 这个 Skill 负责什么

- 将本地 markdown 内容推送为钉钉文档（adoc 类型）
- 更新已有钉钉文档的内容
- 列出知识库或文件夹下的文档节点
- 搜索钉钉文档
- 创建文件夹、重命名/移动/删除文档节点

不负责：
- 推送到飞书、语雀、Notion 等其他平台
- 只修改本地文件（无云端操作）
- 钉钉 IM 消息、钉钉企业成员管理

## 前置条件

钉钉文档 MCP Server 必须已配置到 Claude Code 的 MCP 设置中，MCP 名称为 `dingtalk-doc`。

如果 MCP 工具不可用，先走**初始化流程**，再执行任何文档操作。

## 初始化流程（首次使用或 MCP 不可用时）

> **注意：** 以下步骤 4 的 MCP 注册命令仅适用于 Claude Code 客户端。其他客户端（如 Claude Desktop、Cursor、第三方集成等）的 MCP 配置方式不同，请参考对应客户端的文档或询问用户如何在其客户端中添加 StreamableHttp 类型的 MCP 服务。

1. 告知用户访问帮助手册页面并完成开通：
   > 请打开以下链接，使用钉钉账号登录后，点击页面上的【开通服务】按钮：
   > https://aihub.dingtalk.com/#/detail?mcpId=9629&detailType=marketMcpDetail

2. 开通后复制页面右侧 **StreamableHttp URL**（不是 JSON Config）

3. 请用户将 URL 粘贴给 Claude

4. 收到 URL 后，根据当前客户端类型注册 MCP 服务：
   - **Claude Code**：使用 Bash 工具运行以下命令（`dingtalk-doc` 为 ASCII 名称，MCP 名称不支持中文字符）：
     ```
     claude mcp add --transport http --scope user dingtalk-doc "<用户提供的 URL>"
     ```
   - **其他客户端**：询问用户如何在其客户端中添加 MCP 服务，提供以下信息供用户配置：
     - 传输类型：StreamableHttp（或 HTTP）
     - 服务名称：`dingtalk-doc`（或用户自选）
     - URL：用户提供的 StreamableHttp URL
   > URL 中含有个人 API Key，写入配置后不要在回复中重复显示完整 URL。

5. 询问用户的默认知识库地址：
   > 请在浏览器中打开你常用的钉钉知识库，把地址栏的 URL 粘贴给我（格式类似 `https://alidocs.dingtalk.com/i/spaces/xxxxx/overview`）。
   > 如果暂时不设默认知识库，可以跳过，后续每次操作时再指定。

6. 收到知识库 URL 后，调用 `update-config` skill，追加写入 settings.json：
   ```json
   "env": {
     "DINGTALK_DEFAULT_WORKSPACE_URL": "<用户提供的知识库 URL>"
   }
   ```

7. 配置写入后，提示用户重启 Claude Code 以使 MCP 生效：
   > 请完全退出 Claude Code（关闭应用窗口，或在终端中按 Ctrl+C 结束进程），然后重新启动。仅关闭当前对话或刷新页面**不会**重新加载 MCP 配置。

### 使用默认知识库

- 当用户说"列出我的知识库文档"、"推送到知识库"等未指明位置的操作时，优先使用 `$DINGTALK_DEFAULT_WORKSPACE_URL`
- 先调用 `get_document_info` 传入该 URL 解析出 nodeId，再用 nodeId 调用 `list_nodes` 或作为 `parentNodeId`
- 如果环境变量未设置，直接询问用户要操作哪个知识库

### 更改默认知识库

当用户说"更换默认知识库"、"修改知识库地址"等时：

1. 请用户在浏览器打开目标钉钉知识库，复制地址栏 URL
2. 收到新 URL 后，调用 `update-config` skill 更新 settings.json 中的 `env.DINGTALK_DEFAULT_WORKSPACE_URL`
3. 提示用户重启 Claude Code 使新配置生效

### 安全规则

- URL 中含有个人 API Key，**不得出现在任何版本控制文件中**
- 收到 URL 后直接通过 `claude mcp add` 命令注册，不要在回复中重复显示完整 URL
- 如果用户将 URL 粘贴到聊天中，立即写入配置后不再引用

## 默认路径

1. 确认 MCP 工具可用（若不可用，进入初始化流程）
2. 理解用户意图，映射到对应 MCP 工具（见工具映射表）
3. 如需要 dentryUuid/nodeId 但用户未提供：优先使用 `$DINGTALK_DEFAULT_WORKSPACE_URL` 解析出默认知识库 nodeId；否则调用 `search_documents` 或 `list_nodes` 定位目标，让用户确认后再操作
4. **执行前确认**（只读操作除外，见下方说明）：向用户展示操作摘要，收到明确同意后再调用 MCP 工具
5. 执行操作
6. 报告结果：操作类型、文档标题、文档链接（如有）

### 执行前确认规则

**需要确认的操作**（所有会写入或修改云端数据的操作）：
`create_document`、`update_document`、`create_folder`、`create_file`、`delete_document`、`rename_document`、`move_document`、`copy_document`、`insert_document_block`、`update_document_block`、`delete_document_block`、文件上传（`get_file_upload_info` + `commit_uploaded_file`）

**无需确认的操作**（只读）：
`search_documents`、`list_nodes`、`get_document_info`、`get_document_content`、`list_document_blocks`、`list_permission`

**确认摘要格式**（根据操作类型调整）：

```
即将执行以下操作，请确认：

操作：<创建 / 更新（覆盖）/ 更新（追加）/ 删除 / 重命名 / 移动 / 复制 / 创建文件夹>
目标：<文档标题或文件夹名>
位置：<知识库名或文件夹路径>
说明：<一句话描述本次变更，例如"将以本地文件 xxx.md 的内容覆盖云端文档全文">

确认请回复「是」或「确认」，取消请回复「否」或「取消」。
```

收到取消时：停止操作，不重试，告知用户已取消。

## MCP 工具映射

详见 `references/mcp-tools.md`。常用映射如下：

| 用户意图 | MCP 工具 |
|---------|---------|
| 推送/创建文档（含 markdown 内容） | `create_document` |
| 覆盖或追加文档内容 | `update_document` |
| 列出知识库/文件夹下的文档 | `list_nodes` |
| 按关键词搜索文档 | `search_documents` |
| 获取文档元信息（标题、类型等） | `get_document_info` |
| 读取文档正文 | `get_document_content` |
| 创建文件夹 | `create_folder` |
| 删除文档节点 | `delete_document` |
| 重命名节点 | `rename_document` |
| 移动节点到其他文件夹 | `move_document` |
| 复制节点到其他文件夹 | `copy_document` |
| 精确块级编辑（段落/标题/表格等） | `list_document_blocks` → `insert/update/delete_document_block` |

## 失败处理

- **MCP 工具不可用**：停止，进入初始化流程，不要猜测或尝试其他方式调用
- **未提供目标文档**：先用 `search_documents` 或 `list_nodes` 找到目标，让用户确认后再操作
- **update_document 报错**：确认目标是否为 adoc（文字类型）文档，非 adoc 文档不支持此操作
- **delete_document 前**：确认摘要中必须注明"将移入回收站，30 天内可恢复"，收到确认后再执行
- **API 报错**：原文展示错误信息，不猜测原因，不自动重试

## 关键约束

- `update_document` 和 `create_document` 仅支持 **adoc（文字类型）** 文档，不支持表格、演示、脑图等
- `delete_document` 是移入回收站，不是永久删除，30 天内可从回收站恢复
- `list_nodes` 只返回直接子节点，不递归。需要深层列表时需要多次调用
- **Mermaid 文本绘图**：通过 `create_document` 推送的 ` ```mermaid ` 代码块会被解析为普通代码块，而非钉钉原生文本绘图块。钉钉的文本绘图是私有块类型，API 层未暴露，无法通过 MCP 创建或转换。推送前告知用户此限制，建议推送后在钉钉编辑器中手动将代码块转换为文本绘图
- 操作完成后，从返回值中提取文档链接告知用户，方便直接点击查看

## 对话中上传文件的处理

用户可能在对话中直接上传 markdown 文件并要求推送到钉钉。

### 已知限制

- 对话上传的文件内容是**临时的**，不会持久保存到磁盘，上下文压缩或会话结束后内容即丢失
- **非 ASCII 文件（如中文 markdown）通过对话上传时，0x80-0x9F 范围的字节会被文本处理层丢弃，导致不可逆乱码**。这是客户端文本传输的固有限制，编码修复无法还原丢失的字节

### 处理规则

1. **优先使用本地文件路径**：当用户要推送含非 ASCII 内容（中文、日文等）的文件时，请用户提供本地磁盘路径，通过 Read 工具直接读取文件，避免编码问题
2. **对话上传的文件先检查编码**：如果用户直接在对话中上传了文件，先检查内容是否出现乱码。如果存在乱码，立即告知用户并请其提供本地文件路径
3. **纯 ASCII 内容可直接处理**：如果上传的文件内容全是 ASCII（英文、代码等），可以直接处理，在同一轮回复中完成：读取内容 -> 确认摘要 -> （用户确认后）调用 `create_document`
4. **文档命名**：默认使用上传文件名（去掉扩展名）作为钉钉文档标题。如果用户要求修改名字，按用户指定的名字创建
5. **仅支持 markdown / 纯文本**：对话上传的二进制文件（PDF、图片、Word 等）当前不支持直接推送，需要用户提供本地磁盘路径后走文件上传三步流程

## 资源导航

- `references/mcp-tools.md` — 全部 MCP 工具详细说明，按场景分组

# 扩展点（占位）

**状态：整篇待撰写 / TBD。**

这里只保留以后要写的目录，并链到仓库里已经存在的官方说明。不要把本文件当成接入指南。写法、示例和顺序都还没写。

阅读代码结构见 [code-walkthrough.md](code-walkthrough.md)，运行时边界见 [architecture.md](architecture.md)。

下文「现有文档」都是上游英文文档，内容以那些文件为准。

## 1. coding-agent 扩展（Extensions）

**状态：待撰写 / TBD**

以后写：进程内 TypeScript 扩展的加载位置、`ExtensionAPI`、与内置扩展的替换关系。实现落在 `packages/coding-agent/src/core/extensions/`，内置清单在 `packages/coding-agent/src/extensions/index.ts`。

现有文档：[packages/coding-agent/docs/extensions.md](../../packages/coding-agent/docs/extensions.md)

例子目录：[packages/coding-agent/examples/extensions/](../../packages/coding-agent/examples/extensions/)

后文需要覆盖的注册面（仅列名称，语义待写）：

- 事件：`project_trust`、`resources_discover`、`session_start`、`session_info_changed`、`session_before_switch`、`session_before_fork`、`session_before_compact`、`session_compact`、`session_compact_failed`、`session_shutdown`、`mcp_servers_change`、`session_before_tree`、`session_tree`、`context`、`context_with_system`、`cache_warming_decision`、`before_provider_request`、`before_provider_headers`、`after_provider_response`、`provider_stream_event`、`before_agent_start`、`agent_start`、`agent_end`、`agent_before_settle`、`agent_settled`、`ui_prompt_start`、`ui_prompt_end`、`turn_start`、`turn_end`、`message_start`、`message_update`、`message_end`、`tool_execution_start`、`tool_execution_update`、`tool_execution_end`、`model_select`、`thinking_level_select`、`tool_call`、`tool_result`、`user_bash`、`input`
- 注册：`registerTool`、`registerCommand`、`registerShortcut`、`registerFlag`、`registerProvider`、`registerMcpServer`、`registerVirtualModel`、`registerMessageRenderer`、`registerEntryRenderer`、`registerToolRenderer`、`registerMarkdownTransformer`

类型定义：`packages/coding-agent/src/core/extensions/types.ts` 的 `ExtensionAPI`。

## 2. 技能（Skills）

**状态：待撰写 / TBD**

以后写：`SKILL.md` 如何被发现、如何进入系统提示、何时载入全文。发现逻辑在 `packages/coding-agent/src/core/skills.ts` 和 `resource-loader.ts`。

现有文档：[packages/coding-agent/docs/skills.md](../../packages/coding-agent/docs/skills.md)

## 3. 提示模板（Prompt templates）

**状态：待撰写 / TBD**

以后写：Markdown 模板如何变成斜杠命令，以及和扩展命令的先后关系。加载在 `packages/coding-agent/src/core/prompt-templates.ts`，展开在 `AgentSession.prompt`。

现有文档：[packages/coding-agent/docs/prompt-templates.md](../../packages/coding-agent/docs/prompt-templates.md)

## 4. 主题（Themes）

**状态：待撰写 / TBD**

以后写：交互模式与 HTML 导出的调色板、`system` / `dark` / `light`、用户主题目录。代码在 `packages/coding-agent/src/modes/interactive/theme/`。

现有文档：[packages/coding-agent/docs/themes.md](../../packages/coding-agent/docs/themes.md)

## 5. Pi packages

**状态：待撰写 / TBD**

以后写：用 npm、git 或本地目录把扩展、技能、提示模板、主题打成一份可安装资源。安装逻辑在 `packages/coding-agent/src/core/package-manager.ts`。

现有文档：[packages/coding-agent/docs/packages.md](../../packages/coding-agent/docs/packages.md)

## 6. 模型与供应商接入

**状态：待撰写 / TBD**

以后写：`models.json` 兼容端点、扩展里的 `registerProvider`、虚拟模型、认证存储。这是 CLI 的接入面；线协议实现在 `packages/ai`。

现有文档：

- [packages/coding-agent/docs/models.md](../../packages/coding-agent/docs/models.md)
- [packages/coding-agent/docs/custom-provider.md](../../packages/coding-agent/docs/custom-provider.md)
- [packages/coding-agent/docs/providers.md](../../packages/coding-agent/docs/providers.md)
- [packages/coding-agent/docs/virtual-models.md](../../packages/coding-agent/docs/virtual-models.md)
- [packages/ai/README.md](../../packages/ai/README.md)

## 7. MCP

**状态：待撰写 / TBD**

以后写：CLI 的 `pi mcp` 与 `mcp.json`，以及扩展里的 `registerMcpServer`。客户端库是 `@earendil-works/pi-mcp`。内置扩展可被替换：`packages/coding-agent/src/extensions/mcp/`。

现有文档：

- [packages/coding-agent/docs/mcp.md](../../packages/coding-agent/docs/mcp.md)
- [packages/mcp/README.md](../../packages/mcp/README.md)

## 8. Codemode 与工具搜索

**状态：待撰写 / TBD**

以后写：内置且可替换的 `codemode`、`tool-search` 扩展如何把其他工具暴露给模型。沙箱库是 `@earendil-works/pi-codemode`。

现有文档：

- [packages/coding-agent/docs/codemode.md](../../packages/coding-agent/docs/codemode.md)
- [packages/codemode/README.md](../../packages/codemode/README.md)

## 9. 指令文件与键位

**状态：待撰写 / TBD**

以后写：`AGENTS.md`、`SYSTEM.md`、`APPEND_SYSTEM.md`、`keybindings.json`、内置斜杠命令。它们是配置资源，不是 `ExtensionAPI`。

现有文档：

- [packages/coding-agent/docs/configuration.md](../../packages/coding-agent/docs/configuration.md)
- [packages/coding-agent/docs/settings.md](../../packages/coding-agent/docs/settings.md)
- [packages/coding-agent/docs/keybindings.md](../../packages/coding-agent/docs/keybindings.md)
- [packages/coding-agent/docs/slash-commands.md](../../packages/coding-agent/docs/slash-commands.md)

## 10. 嵌入面：SDK、RPC、print / JSON

**状态：待撰写 / TBD**

以后写：在另一个程序里驱动同一个 `AgentSession` 的三种方式（进程内 SDK、JSONL RPC、一次性 print/JSON）。这是宿主集成点，不是扩展模块。

现有文档：

- [packages/coding-agent/docs/sdk.md](../../packages/coding-agent/docs/sdk.md)
- [packages/coding-agent/docs/rpc.md](../../packages/coding-agent/docs/rpc.md)
- [packages/coding-agent/docs/cli.md](../../packages/coding-agent/docs/cli.md)
- [packages/coding-agent/docs/json.md](../../packages/coding-agent/docs/json.md)
- [packages/coding-agent/examples/sdk/](../../packages/coding-agent/examples/sdk/)

## 11. Durable 扩展

**状态：待撰写 / TBD**

以后写：`defineExtension` / `defineTool` / `section` / `hook` / `wrapTool` 与 coding-agent `ExtensionAPI` 的差异。代码在 `packages/durable/src/harness/define.ts`。本段不提前写用法。

现有文档：

- [packages/durable/README.md](../../packages/durable/README.md)（Extensions、Tools、Hooks 等节）
- [packages/durable/docs/spec.md](../../packages/durable/docs/spec.md)

## 12. Chord 插件与 p2 插件

**状态：待撰写 / TBD**

以后写：facet、registry、host、热替换，以及 p2 插件目录如何挂到这些机制上。这是另一套插件模型，和上面的 coding-agent 扩展分开写。

现有文档：

- [packages/chord/README.md](../../packages/chord/README.md)
- [packages/chord/docs/plugin-authoring.md](../../packages/chord/docs/plugin-authoring.md)
- [packages/chord/docs/design.md](../../packages/chord/docs/design.md)
- [packages/chord/docs/guide-plugins-hosts/book.md](../../packages/chord/docs/guide-plugins-hosts/book.md)
- [apps/p2/docs/plugin-practices.md](../../apps/p2/docs/plugin-practices.md)
- [apps/p2/README.md](../../apps/p2/README.md)

p3 仍是设计笔记（`apps/p3/docs/`），没有独立的扩展 API 可写。若以后成文，再单开一节。

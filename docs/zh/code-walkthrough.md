# 代码导读

这份文档是 [suntydic/pi](https://github.com/suntydic/pi) 源码的阅读地图：从哪进、改某类行为时打开哪个目录、怎样在这个 checkout 里构建和跑起来。产品安装和日常用法在根 [README.md](../../README.md) 和 [packages/coding-agent/docs/index.md](../../packages/coding-agent/docs/index.md)。架构边界在 [architecture.md](architecture.md)。

快照与架构文档相同：包版本 `1.5.1`，Node `>=22.19.0`。

## 先读哪几份

按这个顺序，足够在脑子里留下调用链，再按任务下钻：

1. [packages/coding-agent/docs/how-pi-works.md](../../packages/coding-agent/docs/how-pi-works.md)：CLI 的循环、上下文、会话树、四种界面。短，且和 `AgentSession` 的行为一致。
2. `packages/coding-agent/src/cli.ts` 和 `src/main.ts`：进程怎么选模式并创建会话。
3. `packages/coding-agent/src/core/sdk.ts` 的 `createAgentSession`，然后 `src/core/agent-session.ts` 的 `prompt`、`_startRun`、`_prepareRun`。
4. `packages/agent/src/agent-loop.ts` 的 `runLoop`：模型请求和工具执行的那一层循环。
5. `packages/ai/src/models.ts` 和 `packages/ai/src/providers/all.ts`：请求怎么进供应商。
6. 只有在要做持久化和多进程宿主时，再读 `packages/durable/README.md` 和 `packages/chord/README.md`。p2 的组装看 `apps/p2/README.md`。

## 仓库地图

```text
pi/
  package.json              workspace 根，build / check 脚本
  README.md                 上游产品说明
  test.sh                   隔离家目录后跑测试
  pi-test.sh                用源码启动 pi
  packages/
    schema/                 JSON Schema
    telemetry/              span / metric 契约
    tui/                    终端 UI
    codemode/               QuickJS 沙箱
    mcp/                    MCP 客户端
    ai/                     模型 API、供应商、目录生成
    chord/                  对象运行时
    agent/                  agent 循环
    durable/                持久 harness
    env/                    Durable 的远程执行（含 Rust daemon）
    coding-agent/           pi CLI 和 SDK
    evals/                  行为评测（私有包）
  apps/
    p2/                     实验性 Durable UI
    p3/docs/                Pi Roam 设计笔记，不是应用
  docs/zh/                  本 fork 的中文笔记
```

根 `package.json` 的 `build` 顺序就是依赖顺序：telemetry → schema → chord → tui → codemode → mcp → ai → durable → env → agent → coding-agent。

## `packages/coding-agent/src`

这是改 CLI 产品时停留最久的树。

| 路径 | 作用 |
|---|---|
| `cli.ts` | 可执行入口。调用 `setupCli()` 和 `main()` |
| `cli/setup.ts` | 进程标题、`PI_CODING_AGENT=true`、HTTP dispatcher |
| `cli/args.ts` | 参数解析 |
| `main.ts` | 把参数变成 `createAgentSession` 选项，分派 interactive / print / json / rpc，以及 `auth`、`install` 等管理命令 |
| `index.ts` | npm 包的公共导出 |
| `config.ts` | `~/.pi/agent`、会话目录、包路径。读 `package.json` 的 `piConfig` |
| `core/sdk.ts` | `createAgentSession` |
| `core/agent-session.ts` | 会话运行时：prompt、扩展、压缩、事件、对 `SessionManager` 的写入 |
| `core/agent-session-services.ts` | CLI 用来组装 session 的服务层 |
| `core/session-manager.ts` | JSONL 树。`CURRENT_SESSION_VERSION = 3` |
| `core/model-runtime.ts` | 持有 pi-ai `Models`，接 auth 和 `models.json` |
| `core/model-resolver.ts` | CLI 的模型和范围选择 |
| `core/resource-loader.ts` | 发现扩展、技能、提示模板、主题、`AGENTS.md` / `SYSTEM.md` |
| `core/extensions/` | 扩展类型、jiti 加载、runner |
| `core/tools/` | 内置工具和它们的 TUI 渲染 |
| `core/system-prompt.ts` | 系统提示段落 |
| `core/compaction/` | 上下文压缩 |
| `core/settings-manager.ts` | `settings.json` |
| `core/auth-storage.ts` | `auth.json` |
| `core/package-manager.ts` | Pi package（npm / git / 本地目录） |
| `modes/interactive/` | 交互 TUI。`interactive-mode.ts` 是总控 |
| `modes/print-mode.ts` | print 与 JSON 事件输出 |
| `modes/rpc/` | RPC 模式和 `RpcClient` |
| `extensions/` | 随 pi 发布的内置扩展：`llama`、`mcp`、`codemode`、`tool-search` |
| `experimental/source-resolver.ts` | `pi-test.sh` 用来直接跑 TypeScript 源码 |
| `utils/` | 路径、shell、图片、剪贴板等。多数是 CLI 细节 |

从源码跑 CLI 时，`pi-test.sh` 用 `node --import <source-resolver> packages/coding-agent/src/cli.ts`。安装后的 `pi` 命令跑的是 esbuild 打出来的 `dist/bundle/cli.js`（`package.json` 的 `bin` 和 `build` 脚本）。

交互界面很大。先看 `modes/interactive/interactive-mode.ts` 里对 `session.prompt` 的调用，再按组件名打开 `components/`。主题逻辑在 `theme/`。不要从单个消息组件倒着猜会话状态；状态在 `AgentSession`。

## 其他包里值得先打开的文件

| 包 | 先打开 |
|---|---|
| `packages/agent` | `src/agent.ts`（`Agent` 类、队列、钩子字段）、`src/agent-loop.ts`（`runLoop`）、`src/types.ts` |
| `packages/ai` | `src/index.ts`（根导出边界）、`src/models.ts`（`createModels`、流式入口）、`src/types.ts`、`src/providers/all.ts`、`src/api/` 下与目标协议同名的文件、`src/auth/` |
| `packages/ai/catalog` | 模型目录的源。README 说明生成方式。生成结果是 `src/models.generated.ts` |
| `packages/tui` | `README.md` 的组件列表，然后 `src/` 里对应组件。交互模式从 `@earendil-works/pi-tui` 引入 `TUI`、`Editor`、`Markdown` 等 |
| `packages/mcp` | `README.md`，然后 `src/index.ts` |
| `packages/codemode` | `README.md`。CLI 侧的工具包装在 `packages/coding-agent/src/extensions/codemode/` |
| `packages/schema` | `README.md`。Chord 和 pi-ai 的 schema 构建器从这里来 |
| `packages/telemetry` | `README.md`。看契约即可，CLI 主路径目前不创建适配器 |
| `packages/chord` | `README.md` 的入口表，然后 `docs/design.md`。子目录 `src/contracts`、`src/documents`、`src/store`、`src/node/plugins`、`src/node/http`、`src/node/workers` 各有 README |
| `packages/durable` | `README.md`，`src/harness/harness.ts`，`src/harness/generation.ts`，`src/harness/define.ts`，`src/session/session.ts`。规范是 `docs/spec.md` |
| `packages/env` | `README.md`，`docs/protocol.md`，`daemon/`（Rust） |
| `apps/p2` | `README.md`，`src/hosts/server.ts`，`plugins/<name>/`。实践约定是 `docs/plugin-practices.md` |
| `apps/p3/docs` | 设计状态，例如 `status-2026-11-09.md`、`decisions.md`。没有运行入口 |

`packages/agent` 只有六个源文件，适合在改循环语义之前整包读完。`agent-session.ts` 和 `interactive-mode.ts` 都超过数千行；用符号搜索 `prompt`、`_startRun`、`appendMessage`，不要从头线性读。

## 改什么时打开哪里

| 任务 | 从这里开始 |
|---|---|
| 启动参数、默认模式、首次设置 | `src/main.ts`，`src/cli/args.ts` |
| 用户按回车之后、请求模型之前 | `AgentSession.prompt`、`_preparePrompt`、`_prepareRun` |
| 模型已经在回复、工具如何执行 | `packages/agent/src/agent-loop.ts` |
| 增加或修改内置文件/shell 工具 | `src/core/tools/`，以及 `src/core/tools/renderers/` 的 TUI 呈现 |
| 系统提示正文 | `src/core/system-prompt.ts`，加上 `resource-loader.ts` 里上下文文件的发现 |
| 会话文件格式、分支、恢复 | `src/core/session-manager.ts`，文档 `docs/session-format.md` |
| 压缩策略 | `src/core/compaction/`，以及 `AgentSession._checkCompaction` |
| 扩展怎么被加载 | `src/core/extensions/loader.ts`、`runner.ts`、`types.ts` 里的 `ExtensionAPI` |
| 内置 MCP / codemode / 工具搜索 | `src/extensions/<name>/` |
| 新的 OpenAI 兼容端点 | 运行时的 `models.json`。代码在 `src/core/model-runtime.ts` 和 `packages/ai` 的 provider 配置 |
| 新的 HTTP 线协议 | `packages/ai/src/api/`，再在对应 `providers/*.ts` 里挂上 |
| 模型名单 | `packages/ai/catalog/`，然后 `npm run generate:models`。不要手改 `models.generated.ts` |
| 终端外观、编辑器键位 | `packages/tui`，以及 `modes/interactive/theme/`、`src/core/keybindings.ts` |
| 主题文件格式 | `modes/interactive/theme/theme-schema.ts`，用户文档 `docs/themes.md` |
| 技能和 `/模板` | `src/core/skills.ts`，`src/core/prompt-templates.ts` |
| Pi package 安装 | `src/core/package-manager.ts`，`src/package-manager-cli.ts` |
| 进程内嵌入 | `src/core/sdk.ts`，例子在 `examples/sdk/`，文档 `docs/sdk.md` |
| 子进程控制 | `src/modes/rpc/`，文档 `docs/rpc.md` |
| 可崩溃恢复的 agent | `packages/durable`，不要在 `AgentSession` 里模拟一套检查点 |
| 多进程插件宿主 | `packages/chord` 和 `apps/p2` |

官方扩展、技能、主题、包的用法以后写在 [extension-points.md](extension-points.md)。那份文件目前只有提纲。

## 在这个 checkout 里构建和运行

依赖安装不要跑生命周期脚本：

```bash
npm install --ignore-scripts
```

构建全部公开包（会刷新模型数据，需要网络）：

```bash
npm run build
```

模型数据已经在本地时：

```bash
npm run build:offline
```

只刷新模型数据：

```bash
npm run hydrate:model-data
```

用源码启动 CLI（可在任意工作目录通过绝对路径调用；脚本自己定位仓库）：

```bash
./pi-test.sh
./pi-test.sh --no-env          # 去掉脚本里列出的供应商环境变量后再启动
```

类型检查和 lint 是根目录的：

```bash
npm run check
```

它跑 Biome、依赖钉版本检查、TypeScript `tsc --noEmit`、coding-agent 示例的 `tsc`，以及若干仓库脚本。它不跑测试。`apps/p2` 不在这次检查里：

```bash
cd apps/p2 && npm run check
```

测试用仓库自带的隔离脚本（临时 `HOME`，不带供应商 API key）：

```bash
./test.sh
```

只跑一个包里的一个文件时，从该包目录调用 vitest，避免拉起会在有 endpoint 环境变量时激活的 e2e。例如 coding-agent：

```bash
node "$(git rev-parse --show-toplevel)/node_modules/vitest/dist/cli.js" --run test/specific.test.ts
```

`packages/tui` 使用 Node 自带测试：`node --test test/specific.test.ts`。p2 的套件是 `cd apps/p2 && npm test`（TUI 测试需要 Node 26，浏览器测试需要 `CHROMIUM_BIN`）。

coding-agent 的行为套件在 `packages/coding-agent/test/suite/`，用 `test/suite/harness.ts` 和 pi-ai 的 faux provider，不调用真实供应商。

Chord 和 Durable 里会拉起子进程或用 esbuild 打包的测试，需要先有 `dist/`。只做类型检查时用 `npm run check`，不必为了阅读去构建。

p2 的启动方式写在 `apps/p2/README.md`：仓库根安装依赖并 `npm run hydrate:model-data` 之后，`cd apps/p2 && npm start`，默认端口 8788。

## 本地状态落在哪

从源码跑和从 npm 安装跑，默认都写用户目录，而不是仓库目录：

- agent 目录：`~/.pi/agent`，或环境变量 `PI_CODING_AGENT_DIR`
- 会话目录：`~/.pi/agent/sessions/`，或 `PI_CODING_AGENT_SESSION_DIR`
- 项目资源：工作目录下的 `.pi/`，信任通过后加载

`pi-test.sh` 不会自动把这些路径指到仓库里。用 `./test.sh` 时，脚本会换一个临时 `HOME`，避免读到本机凭证。

p2 的数据目录是 `~/.pi/p2`（`P2_DATA_DIR` 可改），和 CLI 的 agent 目录分开。

## 和上游文档的关系

`packages/coding-agent/docs/` 是 CLI 的用户手册（命令、会话、扩展、SDK、RPC、安全）。各包 README 是该库的 API 说明。`packages/chord/docs/design.md` 和 `packages/durable/docs/spec.md` 是那两条实验线的规则正文。本目录只补「代码怎么拼在一起」，不重复那些手册。

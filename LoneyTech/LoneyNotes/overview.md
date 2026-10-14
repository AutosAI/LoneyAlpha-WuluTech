# LoneyNotes：LangAlpha 项目研究总结

## 项目概览
LangAlpha 是一个基于**程序化工具调用 (PTC)** 模式的金融研究平台。该系统通过沙箱化执行，让 AI 代理能够进行复杂的金融数据分析、图表绘制和多步研究。

### 核心架构要点

1. **代理系统**
   - 单一 AI 代理，双角色模式：Analyst (分析员) 和 Chief of Staff (主管)
   - 使用 `create_agent()` 从 `langchain.agents` 创建
   - 自定义中间件堆栈，包含 `BackgroundSubagentOrchestrator`
   - 内置 5 个子代理：research、general-purpose、data-prep、equity-analyst、report-builder

2. **角色系统**
   - **Analyst role**: 在工作区生成交付物，处理新分析
   - **Chief of Staff role**: 在用户的 "Home" 文件夹中运行，协调所有工作区
   - `resolve_turn_route` 决定每个转向使用哪个角色
   - Flash Agent: 当 flag 关闭时的助手路径，跳过子代理和沙箱

3. **数据库与多工作器**
   - 原始异步 `psycopg3` (无 ORM)
   - 两个独立连接池：app data + LangGraph checkpointer
   - Alembic 迁移使用原始 SQL via `op.execute()`
   - **关键**：进程内的模块级状态从不作为 liveness/truth，真相生活在 Postgres 中，Redis 只做协调/传输

4. **层级结构**
   User → Computer → Workspace → Thread → Turns

5. **提示与记忆**
   - **提示**: `src/ptc_agent/agent/prompts/templates/` 的 Jinja2 模板
   - **长期记忆**: LangGraph `BaseStore` 按代理层级
   - **备忘录存储**: 用户上传的文档，存储在 S3 兼容存储
   - **scratchpad**: 每个线程的 `.agents/scratchpad/<thread-id>/` 文件夹
   - **线程transcripts**: JSONL 存储在 `thread_transcripts` 字节列中

6. **文件安装 (FUSE)**
   - 每个计算机的 FUSE daemon 服务器持有的文件
   - 沙箱路径是指向 `/mnt/livefs` 的符号链接
   - Daemon 用每个计算机的 opaque token 进行身份验证

### 桌面版本架构

#### 两个版本，同一源码

| 特性 | `oss` | `saas` |
|------|-------|--------|
| 首次运行入口 | 服务器选择 | hosted sign-in |
| 平台控制台 | 不存在 | 共享应用窗口 |
| 配置 | `config/default.json` (已提交) | `config/build.json` (gitignored) |
| 应用名称 | `LangAlpha OSS` | `LangAlpha` |
| Bundle ID | `ai.langalpha.desktop.oss` | `ai.langalpha.desktop` |
| URL 方案 | `langalpha-oss://` | `langalpha://` |

#### 构建系统
- `electron-builder.yml`：每个版本的身份标识
- `scripts/build.mjs`：根据版本重写 `electron-builder.yml`
- **重要**：不能同时构建两个版本（config/build.json 的竞态条件）
- 输出分离：`dist/${EDITION}` (dist/oss vs dist/saas)

#### OAuth 与深链
- **OAuth**: Google 拒绝在嵌入式 webview 中操作，Shell 通过 loopback 监听处理
- **深链**: `langalpha://` 携带 3 种链接类型：
  - 通用链接 → `/callback`
  - `integrations/login/<name>/callback`
  - `integrations/install/<name>/callback`

### MCP 协议契约

1. **数据客户端抽象**
   - `src/data_client/` - 金融数据协议抽象
   - 两个连接池：app data + LangGraph checkpointer

2. **内置 MCP 服务器**
   - `plugins/`：内置 MCP 服务器和技能
   - 一个 Agent Plugins 1.0.0 包/组
   - `mcp.json`：每一层都有 `additionalProperties: false`
   - 服务器元数据 lives in `plugin.json` under `extensions["ai.langalpha"]`

3. **服务器端 LLM 调用**
- 通过 `LLMService.complete`，永不直接使用 `create_llm()`
- 合同在 `src/server/AGENTS.md`

4. **第三方 MCP 服务器**
- 通过 `uvx`/`npx` 与 pinned versions 隔离
- 永不使用共享 app venv 或 sandbox system Python
- 添加/更新 API 在 shared-env 命令上发出警告

5. **工具 docstring**
- 遵循 `src/tools/AGENTS.md` 表面约定
- 直接工具：一个用于选择信号的行
- MCP 工具：完整的机器可读形状
- Docstrings 是提示表面 - 每一句话都影响决策

### 前端架构

- **技术栈**: React 19 + Vite + TypeScript + Tailwind CSS + shadcn/ui
- **状态管理**: React Query
- **路径别名**: `@` → `web/src/` (双模式 auth 的陷阱: `VITE_HOST_MODE`)
- **15 关键文件**: index.js, main.js, policy.js, origins.js, oauth.js, deeplink.js, preload.js, pdf.js, downloads.js, notify.js, outage.js, updater.js, probe.js, store.js

### 关键检查清单

1. 阅读 `src/server/AGENTS.md` § Multi-worker 并在触发 turn 生命周期前验证
2. 通过 `scripts/multiworker_gate/` 验证
3. 从不将本地注册表作为 liveness/truth
4. 两个版本分开构建（构建过程中的 race condition）
5. 签名和 notarization 在 CI 中为两个版本都已配置

## 已完成的研究文件

1. `research_project_overview.md` - 项目架构概览
2. `research_roles.md` - 角色与转向路由
3. `desktop_editions.md` - OSS vs SaaS 版本详情
4. `mcp_contract.md` - MCP 协议契约
5. `frontend_architecture.md` - 前端架构细节
6. `research_summary.md` - 研究综合摘要

## 关键洞察

1. **PTC 模式**: LLM 不直接调用 MCP 工具 - 编写 Python 导入生成的包装模块和沙箱中的函数
2. **多工作器**: 真相在 Postgres，Redis 只做协调，进程内状态不可信
3. **两个版本**: 同一源码，不同输出，不同身份标识
4. **Electron shell**: 远程 URL 包装，不打包 web bundle
5. **feature-detected**: 从不 version-detected (用于框架/更新)

## 下一步行动建议

1. 审查 `src/server/AGENTS.md` 中的 multiworker gate 检查清单
2. 检查 `mcp_servers/AGENT_CONTRACT.md` 了解代理合约细节
3. 探索 `plugins/` 结构和内置 MCP 服务器
4. 理解 data_client 抽象层
5. 根据需要开始实际开发任务
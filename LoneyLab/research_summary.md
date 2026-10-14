# Research Summary: LangAlpha Project Overview

## Created Research Materials

### 1. `research_project_overview.md`
- **Project Purpose**: LangAlpha financial research platform with PTC pattern
- **Core Architecture**: Agent system, roles, multi-worker server, data/db, prompts/memory, file mount
- **Key Conventions**: Python 3.13+, config split, plugins, LLM service calls
- **Build & Deployment**: Desktop editions, third-party MCP servers, critical warnings

### 2. `research_roles.md`
- **AgentRole Enum**: analyst vs chief_of_staff
- **Role Responsibilities**: analyst produces deliverables, chief_of_staff coordinates across workspaces
- **`resolve_turn_route`**: picks role per turn, chief_of_staff in Home, decides workspace vs Flash
- **Flash Agent**: assistant path when flag off, skips subagents and sandbox

### 3. `desktop_editions.md`
- **Two Editions**: OSS vs SaaS with comparison table
- **`src/config.js`**: Merges editions, refuses saas without platformOrigin
- **`scripts/build.mjs`**: Rewrites electron-builder.yml per edition
- **Build Commands**: OSS default, SaaS requires env vars
- **OAuth & Deep Links**: Generic catcher, scheme links (login/install)

### 3. `mcp_contract.md`
- **Financial Data Protocol**: Raw psycopg3, two connection pools
- **MCP Servers**: Built-in vs third-party, mcp.json closed structure
- **Server-Side LLM**: LLMService.complete, not create_llm()
- **Tool Docstrings**: src/tools/AGENTS.md surface conventions
- **Agent Contract**: mcp_servers/AGENT_CONTRACT.md, codegen versioning

### 4. `frontend_architecture.md`
- **Tech Stack**: React 19 + Vite + TS + Tailwind + React Query
- **Path Alias**: `@` → web/src, dual-mode auth landmine
- **Layout Artifacts**: 15 files with specific purposes
- **Edition System**: OSS vs SaaS with config differences
- **Frame/Titlebar**: Feature-detected, 38px drag region
- **OAuth**: loopback listener, OS port
- **Network Failure**: captive portal detection, net.isOnline limitations

## Key Project Insights

### Architecture Highlights
1. **PTC Pattern**: LLM doesn't call MCP tools directly - writes Python that imports wrappers
2. **Multi-worker**: Truth in Postgres, Redis coordination, never local state
3. **Two editions**: Same source, separate outputs, different identities
4. **Electron shell**: Remote URL wrapper, no web bundle shipped
5. **Feature-detected**: Never version-detected for frames/updates

### Critical Checklists
- Read `src/server/AGENTS.md` § Multi-worker before touching turn lifecycle
- Verify with `scripts/multiworker_gate/`
- Never treat local registries as liveness/status truth
- Signing and notarization wired on CI for both editions
- Build editions separately (race on config/build.json)

## Recommended Next Steps
1. Review `src/server/AGENTS.md` for multiworker gate checklist
2. Examine `mcp_servers/AGENT_CONTRACT.md` for agent contract details
3. Check `src/tools/AGENTS.md` for tool docstring conventions
3. Explore `plugins/` structure and built-in MCP servers
4. Understand the data_client abstraction layer
</parameter
# Research: Financial Data Protocol & MCP Servers

## Data Client Abstraction
- `src/data_client/` - Financial data protocol abstraction
- Raw `psycopg3` async (no ORM)
- Two separate connection pools: app data + LangGraph checkpointer

## MCP Servers (Built-in)
- **`plugins/`** holds built-in MCP servers and skills
- One Agent Plugins 1.0.0 package per group
- Same format as user-uploaded plugins
- Read at config load by `src/ptc_agent/config/plugins.py`

### MCP Server Contract
- `mcp_servers/` keeps runtime they share (`_bootstrap`, envelope, output schemas)
- `mcp.json` closed (`additionalProperties: false` at every level)
- Server's `description`, `instruction`, `tool_exposure_mode`, `vault_blueprints` live in `plugin.json` under `extensions["ai.langalpha"]`

### Server-Side LLM Calls
- Go through `LLMService.complete`, never `create_llm()` directly
- Contract in `src/server/AGENTS.md`

## Third-party MCP Servers
- Isolated via `uvx`/`npx` with pinned versions
- Never from shared app venv or sandbox system Python
- Add/update API warns on shared-env commands
- `classify_startup_failure` in `src/ptc_agent/core/mcp_registry.py`

## Tool Docstrings
- Follow `src/tools/AGENTS.md` surface conventions
- Direct tools: one line for selection signal
- MCP tools: full machine shape for agent code indexing
- Docstrings are prompt surface - every sentence affects decision making

## Agent Contract
- `mcp_servers/AGENT_CONTRACT.md` defines output schemas
- Docstrings can't be reworded as side effects without breaking tests
- `MCP_CLIENT_CODEGEN_VERSION` caches generated wrappers

# Research: Agent Roles and Turn Routing

## Role System

### AgentRole Enum
- `analyst`: Default role for workspace analysts
- `chief_of_staff`: Runs in user's Home directory

### Role Responsibilities

#### Analyst Role
- Produces deliverables in workspace
- Gets `equity-analyst` subagent when new analysis goes to workspace
- Coordinates within workspace boundaries

#### Chief of Staff Role
- Coordinates across all workspaces
- Manages workspaces, delegates to analysts
- Handles threads outside workspace
- Withholds `equity-analyst` subagent (new analysis goes to workspace)
- Manages threads and delegates to analysts

### `resolve_turn_route` (`src/server/services/turn_runtime.py`)
- Picks role per turn
- `chief_of_staff` in user's Home
- Decides whether work outside workspace goes to Home or Flash

### Flash Agent
- Assistant path when `chief_of_staff` flag is off
- Skips subagents and sandbox
- Reaches MCP only through directly bound tools

## Key Mechanics
- Role adds `<role>` prompt section
- Coordination tools: `manage_workspaces`, `delegate_to_analyst`, `agent_output`, `manage_threads`
- `<activity>` baseline block: recent workspaces, threads, holdings
- Coordination tools from `src/tools/secretary/`

## Important Checklist
- Read `src/server/AGENTS.md` § Multi-worker before touching turn lifecycle
- Verify with `scripts/multiworker_gate/`
- Never treat local registries as liveness/status truth
- Truth lives in Postgres with advisory locks
- Separate pools: app data + LangGraph checkpointer
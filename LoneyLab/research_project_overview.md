# Research Project: LangAlpha Codebase Overview

## Project Purpose
LangAlpha is a financial research platform built with a Programmatic Tool Calling (PTC) pattern. The system enables programmatic interaction with financial data through sandboxed execution, allowing AI agents to perform complex analysis, charting, and multi-step financial research.

## Core Architecture

### Agent System
- **Single AI agent** with dual roles: Analyst (default) and Chief of Staff
- Built with `create_agent()` from `langchain.agents`
- Wrapped in custom middleware stack with `BackgroundSubagentOrchestrator`
- **Subagents**: research, general-purpose, data-prep, equity-analyst, report-builder

### Roles System
- **Analyst role**: Produces deliverables in workspace
- **Chief of Staff role**: Runs in "Home" folder, coordinates work across workspaces
- `resolve_turn_route` decides role per turn

### Multi-worker Server
- Backend runs with `--workers N`
- Multiple processes handle: requests, SSE consumer, outbox drainer, recovery scanner
- Truth lives in Postgres; Redis is coordination/transport
- **Critical**: Never introduce module-level state local to a process

### Data & Database
- Raw `psycopg3` async (no ORM)
- Two separate connection pools: app data + LangGraph checkpointer
- Alembic migrations use raw SQL via `op.execute()`

### Hierarchy
User → Computer → Workspace → Thread → Turns

### Prompt & Memory System
- **Prompts**: Jinja2 templates in `src/ptc_agent/agent/prompts/templates/`
- **Long-term memory**: LangGraph `BaseStore` per agent tier
- **Memo store**: User-uploaded docs in S3-compatible storage
- **Scratchpad**: Per-thread folder at `.agents/scratchpad/<thread-id>/`
- **Thread transcripts**: JSONL stored in `thread_transcripts` bytes column

### File Mount (FUSE)
- Per-computer FUSE daemon serves server-held files
- Sandbox paths are symlinks into `/mnt/livefs`
- Daemon authenticates with opaque per-computer token

## Key Conventions
- Python 3.13+, async-first
- Config split: `.env` for credentials, YAML for behavior
- Per-environment overlays via `APP_ENV`
- `plugins/` holds built-in MCP servers and skills
- Server-side LLM calls go through `LLMService.complete`

## Build & Deployment
- **Desktop**: Electron shell around hosted web app
- Two editions: `oss` (self-hosted) and `saas` (hosted)
- `electron-builder.yml` with per-edition identities
- `scripts/build.mjs` handles edition substitutions
- Separate outputs for each edition (`dist/oss` vs `dist/saas`)

## Third-party MCP Servers
- Isolated via `uvx`/`npx` with pinned versions
- Built-in servers import through `_bootstrap` (era-proof)
- Add/update API warns on shared-env commands

## Critical Warning
**Multi-worker server checklist** must be reviewed before touching turn lifecycle, streaming, or subagent ownership.

## Key Documentation Files
- `src/server/AGENTS.md` - Multi-worker checklist
- `mcp_servers/AGENT_CONTRACT.md` - Agent contract
- `src/tools/AGENTS.md` - Tool docstring conventions
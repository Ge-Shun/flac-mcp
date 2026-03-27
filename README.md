# pfc-mcp

[English](https://github.com/yusong652/pfc-mcp/blob/main/README.md) | [简体中文](https://github.com/yusong652/pfc-mcp/blob/main/README.zh-CN.md)

[![PyPI](https://img.shields.io/pypi/v/pfc-mcp)](https://pypi.org/project/pfc-mcp/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue)](https://www.python.org/)

**MCP server that gives AI agents full access to [ITASCA PFC](https://www.itascacg.com/software/pfc) - browse documentation, run simulations, and execute code, all through natural conversation.**

Built on the [Model Context Protocol](https://modelcontextprotocol.io/), pfc-mcp turns any MCP-compatible AI client (Claude Code, Codex CLI, Gemini CLI, OpenCode, toyoura-nagisa, etc.) into a PFC co-pilot that can look up commands, execute code interactively, run and monitor long-running simulations, and create plots.

![pfc-mcp demo](https://raw.githubusercontent.com/yusong652/pfc-mcp/assets/pfc-mcp.gif)

<a href="https://glama.ai/mcp/servers/yusong652/pfc-mcp">
  <img width="380" height="200" src="https://glama.ai/mcp/servers/yusong652/pfc-mcp/badge" alt="pfc-mcp MCP server" />
</a>

## Tools (10)

### Documentation (5) - no bridge required

- Browse PFC command tree, Python SDK reference, and reference docs (contact models, range elements, plot items)
- Command docs support PFC `6.0`, `7.0`, and `9.0` via the `version` parameter on command tools
- Search commands and Python APIs by keyword (BM25 ranked)

### Execution (5) - requires bridge in a running PFC process

- **pfc_execute_code** - synchronous REPL: run Python snippets, query model state, create plots, export data
- **pfc_execute_task** - submit long-running scripts for async execution with full lifecycle management
- **pfc_check_task_status** / **pfc_interrupt_task** / **pfc_list_tasks** - poll output, cancel tasks, browse history

## Quick Start

### Prerequisites

- **ITASCA PFC 6.0, 7.0, or 9.0** installed
- **[uv](https://docs.astral.sh/uv/getting-started/installation/)** installed (for `uvx`)

### Agentic Setup (Recommended)

Copy this to your AI agent and let it self-configure:

```text
Fetch and follow this bootstrap guide end-to-end:
https://raw.githubusercontent.com/yusong652/pfc-mcp/main/docs/agentic/pfc-mcp-bootstrap.md
```

### Manual Setup

**1. Register the MCP server** in your client config:

```json
{
  "mcpServers": {
    "pfc-mcp": {
      "command": "uvx",
      "args": ["pfc-mcp"]
    }
  }
}
```

**2. Start the bridge from inside PFC:**

Get the bootstrap script here:

`https://raw.githubusercontent.com/yusong652/pfc-mcp/main/pfc-mcp-bridge/bootstrap_bridge.py`

Then use either of these two flows inside PFC:

- Copy the file contents into the PFC IPython console and run them
- Or download the file and execute it in PFC GUI

This bootstrap script is the recommended everyday entrypoint:

- If `pfc-mcp-bridge` is not installed yet, it installs the latest version and starts it
- If it is already installed, it shows the current version and lets you choose whether to upgrade before startup
- It then starts the bridge in the current PFC Python environment

### Start Bridge & Verify

![PFC GUI Python console](https://raw.githubusercontent.com/yusong652/pfc-mcp/assets/install.png)

**Verify** - reconnect your MCP client and ask the agent to call `pfc_list_tasks` to verify the full MCP + bridge connection.

## Design Highlights

- **Documentation as a boundary map** - browse and search tools let agents discover what PFC can do, reducing hallucinated commands
- **Task queue with live status** - scripts are queued and executed sequentially; agents can poll output and status in real time
- **Callback-based control** - gracefully interrupt long-running `cycle()` calls; execute code mid-simulation via cycle-gap callbacks

## Runtime Model

| Component | PyPI | Python | Role |
|-----------|------|--------|------|
| **pfc-mcp** | [![PyPI](https://img.shields.io/pypi/v/pfc-mcp)](https://pypi.org/project/pfc-mcp/) | >= 3.10 | MCP server (documentation + execution client) |
| **pfc-mcp-bridge** | [![PyPI](https://img.shields.io/pypi/v/pfc-mcp-bridge)](https://pypi.org/project/pfc-mcp-bridge/) | >= 3.6 | WebSocket bridge inside PFC process (GUI or console); uses Python 3.6 on PFC 6/7 and Python 3.10 on PFC 9 |

Documentation tools work standalone. Execution tools require a running bridge. Command browsing and search support `version=6.0|7.0|9.0`.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `uvx` not found | [Install uv](https://docs.astral.sh/uv/getting-started/installation/) or switch client MCP config to `command: "uv"` with `args: ["tool", "run", "pfc-mcp"]` |
| Bridge won't start | Download the bootstrap script again and rerun it in PFC, either by pasting it into the IPython console or by executing the downloaded file in PFC GUI |
| Tasks not processing / cannot connect | If execution tools return `ok=false`, `error.code=bridge_unavailable`, and `error.details.reason=cannot connect to bridge service`, start bridge in PFC (`pfc_mcp_bridge.start()`) and ensure `PFC_MCP_BRIDGE_URL` matches the active bridge URL |
| Bridge on custom port | Set MCP server env `PFC_MCP_BRIDGE_URL=ws://localhost:<bridge-port>` (for example `ws://localhost:9002`) |
| Connection failed | Check bridge is running, target port is available, see `.pfc-mcp-bridge/bridge.log` |

## Development

```bash
uv sync --group dev    # Install with dev dependencies
uv run pytest          # Run tests
uv run pfc-mcp         # Run server locally
```

### Running from Source

For a complete developer workflow, see [Developer Guide: Install and Run from Source](docs/development/source-install.md).

Quick version:

- Point your MCP client at a local checkout with `uv run --directory`
- Start the bridge from local source with `%run .../pfc-mcp-bridge/start_bridge.py`
- Use the embedded PFC interpreter from a terminal if you need to install `pfc-mcp-bridge` from local source

Example MCP config:

```json
{
  "mcpServers": {
    "pfc": {
      "command": "uv",
      "args": ["run", "--directory", "/path/to/pfc-mcp", "pfc-mcp"]
    }
  }
}
```

## License

MIT - see [LICENSE](LICENSE).

<!-- mcp-name: io.github.yusong652/pfc -->

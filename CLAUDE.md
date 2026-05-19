# CLAUDE.md

Guidance for coding agents working in the `flac-mcp` repository.

## Project Overview

`flac-mcp` provides an MCP server for ITASCA FLAC workflows plus a bridge runtime that runs inside FLAC GUI.

This project spans two runtime contexts:

- `src/flac_mcp/` (Python >= 3.10): MCP server package used by clients/tooling
- `itasca-mcp-bridge/` (FLAC embedded Python, often 3.6): WebSocket bridge running inside FLAC GUI — a **separate repository** (`yusong652/itasca-mcp-bridge`) vendored here as a git submodule

Treat these as separate deployment targets with independent release cycles.

## Core Architecture

### MCP side (`src/flac_mcp`)

- Exposes documentation tools and execution tools through FastMCP
- Communicates with bridge via WebSocket client (`flac_mcp.bridge.client`)
- Returns a unified tool envelope: `ok`, `data`, `error`
- Dual execution model: synchronous REPL (`flac_execute_code`) for quick queries, script-first async (`flac_execute_task` + `flac_check_task_status`) for long-running simulations

### Bridge side (`itasca-mcp-bridge`)

- Runs in FLAC GUI process
- Owns thread-safe interaction with ITASCA SDK
- Handles long-running tasks and diagnostics
- Must be started from FLAC GUI (for example with `%run .../itasca-mcp-bridge/start_bridge.py`)

## Repository Layout

```text
flac-mcp/
├── src/flac_mcp/
│   ├── bridge/          # MCP-side bridge client/task manager
│   ├── knowledge/       # command/API/reference search system
│   ├── tools/           # MCP tool implementations
│   ├── formatting.py    # shared response formatting
│   └── server.py        # MCP server entrypoint
├── itasca-mcp-bridge/      # git submodule → separate repo; runtime executed inside FLAC GUI
└── tests/               # MCP/tool contract tests
```

## Development Commands

Run from repository root.

```bash
uv sync
uv sync --group dev
uv run flac-mcp
uv run pytest tests/test_phase2_tools.py
uv run pytest tests/test_tool_contracts.py
```

## Engineering Rules

1. Keep MCP and bridge concerns separate.
   - Do not couple MCP logic to FLAC GUI internals.
   - Do not introduce application/session policy into bridge runtime.

2. Preserve execution semantics for each model.
   - `flac_execute_code` runs synchronous snippets and returns stdout/result immediately.
   - `flac_execute_task` submits scripts and returns quickly.
   - Progress/result retrieval goes through `flac_check_task_status`.

3. Maintain structured tool contracts.
   - Prefer stable machine-readable keys over ad-hoc text parsing.
   - Use the unified envelope for all tool business payloads:
     - success: `{"ok": true, "data": ...}`
     - error: `{"ok": false, "error": {"code": str, "message": str, "details"?: object}}`
   - Enforce coherence: `ok=true` must not include `error`; `ok=false` must include `error`.
   - Do not require duplicate presentation fields (for example, `display`) when they mirror structured data.
   - Let clients render human-facing formatting from structured fields.
   - Documentation tools must keep `data` consistent as:
     - `source`: `"commands" | "python_api" | "reference"`
     - `action`: `"browse" | "query"`
     - `entries`: `list[object]`
     - `summary`: `object` (counts/hints/context)
   - Keep query/path/input echo minimal; prefer putting necessary context in `summary` or `error.details.input`.

4. Keep compatibility when practical.
   - If moving shared helpers, keep thin compatibility re-exports when tests or downstream code rely on old import paths.

5. Respect runtime constraints.
   - MCP package uses modern deps (`websockets>=15`).
   - Bridge side may require legacy-compatible deps (`websockets==9.1`) in FLAC Python.

## Testing Expectations

- For tool/contract changes, run:
  - `tests/test_phase2_tools.py`
  - `tests/test_tool_contracts.py`
Mock bridge based tests are preferred for deterministic CI.

## Documentation Sources

FLAC searchable docs live under:

- `src/flac_mcp/knowledge/resources/command_docs/`
- `src/flac_mcp/knowledge/resources/python_sdk_docs/`
- `src/flac_mcp/knowledge/resources/references/`

When changing schema/content shape, verify browse/query tool behavior remains consistent.

## Release Process

The two packages live in **separate repositories** and release independently via GitHub Actions on tag push (PyPI Trusted Publishing — OIDC, no tokens).

| Package | Repo | Tag pattern | Workflow | Version file | PyPI environment |
|---------|------|-------------|----------|--------------|------------------|
| `flac-mcp` | `yusong652/flac-mcp` (this repo) | `v*` (e.g. `v0.1.0`) | `.github/workflows/publish.yml` | `src/flac_mcp/__init__.py` | `pypi` |
| `itasca-mcp-bridge` | `yusong652/itasca-mcp-bridge` (submodule) | `v*` (e.g. `v0.1.0`) | `.github/workflows/publish.yml` | `src/itasca_mcp_bridge/__init__.py` | `pypi` |

Steps to release `flac-mcp` (run in this repo):

1. Bump `__version__` in `src/flac_mcp/__init__.py` (hatch dynamic versioning — single source of truth).
2. Add a matching `## [x.y.z]` entry to `CHANGELOG.md` (the publish workflow hard-fails without it).
3. Commit and push to `main`.
4. Tag and push: `git tag vX.Y.Z && git push origin vX.Y.Z`.

Releasing `itasca-mcp-bridge` follows the same steps **inside its own repository** (`yusong652/itasca-mcp-bridge`), not from here.

The `flac-mcp` publish workflow runs tests before publishing; the bridge workflow publishes directly.

**Important**: the tag version and the `__version__` in `__init__.py` must match. PyPI rejects uploads for versions that already exist.

CI also runs on every push/PR to `main` (`.github/workflows/test.yml`): ruff check, ruff format, mypy, and pytest with coverage.

## Commit Style

Use conventional prefixes seen in repository history, for example:

- `feat: ...`
- `fix: ...`
- `refactor: ...`
- `test: ...`
- `docs: ...`

Keep commit messages focused on why the change was needed.

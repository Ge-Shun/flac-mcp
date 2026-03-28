# pfc-mcp

[English](https://github.com/yusong652/pfc-mcp/blob/main/README.md) | [简体中文](https://github.com/yusong652/pfc-mcp/blob/main/README.zh-CN.md)

[![PyPI](https://img.shields.io/pypi/v/pfc-mcp)](https://pypi.org/project/pfc-mcp/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue)](https://www.python.org/)

**一个为 [ITASCA PFC](https://www.itascacg.com/software/pfc) 提供完整能力的 MCP 服务器：可浏览文档、运行仿真、执行代码，并通过自然语言交互完成操作。**

基于 [Model Context Protocol](https://modelcontextprotocol.io/) 构建，`pfc-mcp` 可以把任何兼容 MCP 的 AI 客户端（Claude Code、Codex CLI、Gemini CLI、OpenCode、toyoura-nagisa 等）变成 PFC 协作助手：查询命令、交互式执行代码、运行和监控长时仿真、创建图表。

![pfc-mcp demo](https://raw.githubusercontent.com/yusong652/pfc-mcp/assets/pfc-mcp.gif)

## 工具（10）

### 文档类（5）- 无需 bridge

- 浏览 PFC 命令树、Python SDK 参考、参考文档（接触模型、range 元素、plot 项目）
- 命令文档支持通过 `version` 参数选择 PFC `6.0`、`7.0` 和 `9.0`
- 基于关键词搜索命令和 Python API（BM25 排序）

### 执行类（5）- 需要在运行中的 PFC 进程中启动 bridge

- **pfc_execute_code** - 同步 REPL：运行 Python 代码片段、查询模型状态、创建图表、导出数据
- **pfc_execute_task** - 提交长时脚本进行异步执行，支持完整生命周期管理
- **pfc_check_task_status** / **pfc_interrupt_task** / **pfc_list_tasks** - 轮询输出、取消任务、浏览历史

## 快速开始

### 前置条件

- 已安装 **ITASCA PFC 6.0、7.0 或 9.0**
- 已安装 **[uv](https://docs.astral.sh/uv/getting-started/installation/)**（用于 `uvx`）

### 智能体自动配置（推荐）

将以下文本复制给你的 AI 智能体，让它自动完成配置：

```text
Fetch and follow this bootstrap guide end-to-end:
https://raw.githubusercontent.com/yusong652/pfc-mcp/main/docs/agentic/pfc-mcp-bootstrap.md
```

### 手动配置

**1. 在客户端配置中注册 MCP 服务：**

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

**2. 在 PFC 中启动 bridge：**

下载 [`addon.py`](addon.py)，然后在 PFC 中任选一种方式执行：

- 把这个文件的内容复制到 PFC 的 IPython 控制台里运行
- 或者先把这个文件下载到本地，再在 PFC GUI 里执行它

### 验证

重连 MCP 客户端后，让智能体调用 `pfc_list_tasks`，确认 MCP 与 bridge 连接均正常。

## 设计亮点

- **以文档为边界地图**：浏览与搜索工具帮助智能体明确 PFC 能力边界，减少“幻觉命令”
- **实时状态的任务队列**：脚本按顺序排队执行，智能体可实时轮询输出和状态
- **基于回调的控制**：可优雅中断长时间 `cycle()`；通过 cycle 间隙回调在仿真进行中执行代码

## 运行时模型

| 组件 | PyPI | Python | 角色 |
|------|------|--------|------|
| **pfc-mcp** | [![PyPI](https://img.shields.io/pypi/v/pfc-mcp)](https://pypi.org/project/pfc-mcp/) | >= 3.10 | MCP 服务器（文档工具 + 执行客户端） |
| **pfc-mcp-bridge** | [![PyPI](https://img.shields.io/pypi/v/pfc-mcp-bridge)](https://pypi.org/project/pfc-mcp-bridge/) | >= 3.6 | PFC 进程内 WebSocket bridge（GUI 或控制台）；PFC 6/7 使用 Python 3.6，PFC 9 使用 Python 3.10 |

文档工具可独立使用；执行工具依赖已运行的 bridge。命令浏览与搜索支持 `version=6.0|7.0|9.0`。

## 故障排查

| 现象 | 处理方式 |
|---------|-----|
| 找不到 `uvx` | [安装 uv](https://docs.astral.sh/uv/getting-started/installation/)，或将客户端 MCP 配置改为 `command: "uv"`、`args: ["tool", "run", "pfc-mcp"]` |
| Bridge 启动失败 | 重新获取 bootstrap 脚本后，在 PFC 中再次运行它：可以把内容重新粘贴到 IPython 控制台，或直接在 PFC GUI 中执行下载好的脚本文件 |
| 任务不执行 / 无法连接 | 若执行工具返回 `ok=false`、`error.code=bridge_unavailable`、`error.details.reason=cannot connect to bridge service`，请在 PFC 中启动 bridge（`pfc_mcp_bridge.start()`），并确认 `PFC_MCP_BRIDGE_URL` 与 bridge 实际地址一致 |
| Bridge 使用自定义端口 | 将 MCP 服务端环境变量设为 `PFC_MCP_BRIDGE_URL=ws://localhost:<bridge-port>`（例如 `ws://localhost:9002`） |
| 连接失败 | 检查 bridge 是否运行、目标端口是否可用，查看 `.pfc-mcp-bridge/bridge.log` |

## 开发

详见 [开发者指南：从源码安装与运行](docs/development/source-install.zh-CN.md)。

## 许可证

MIT，详见 [LICENSE](LICENSE)。

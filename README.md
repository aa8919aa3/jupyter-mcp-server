<!--
  ~ Copyright (c) 2023-2024 Datalayer, Inc.
  ~
  ~ BSD 3-Clause License
-->

[![Datalayer](https://assets.datalayer.tech/datalayer-25.svg)](https://datalayer.io)

[![Become a Sponsor](https://img.shields.io/static/v1?label=Become%20a%20Sponsor&message=%E2%9D%A4&logo=GitHub&style=flat&color=1ABC9C)](https://github.com/sponsors/datalayer)

# 🪐 ✨ Jupyter MCP Server

[![Github Actions Status](https://github.com/datalayer/jupyter-mcp-server/workflows/Build/badge.svg)](https://github.com/datalayer/jupyter-mcp-server/actions/workflows/build.yml)
[![PyPI - Version](https://img.shields.io/pypi/v/jupyter-mcp-server)](https://pypi.org/project/jupyter-mcp-server)
[![smithery badge](https://smithery.ai/badge/@datalayer/jupyter-mcp-server)](https://smithery.ai/server/@datalayer/jupyter-mcp-server)

Jupyter MCP Server is a [Model Context Protocol](https://modelcontextprotocol.io) (MCP) server implementation that provides interaction with 📓 Jupyter notebooks running in any JupyterLab (works also with your 💻 local JupyterLab).

![Jupyter MCP Server](https://assets.datalayer.tech/jupyter-mcp/jupyter-mcp-server-claude-demo.gif)

## 使用 uvx 管理虛擬環境

[uvx](https://github.com/astral-sh/uv) 是一個快速的 Python 套件安裝器和虛擬環境管理工具，可以替代 Docker 方案在本地運行 MCP 服務器。以下是使用 uvx 設定的步驟：

### 安裝 uvx

```bash
# 安裝 uvx (macOS/Linux)
curl -LsSf https://astral.sh/uv/install.sh | sh

# 或者使用 pip
pip install uv
```

### 建立並啟用虛擬環境

```bash
# 建立新的虛擬環境
uv venv .venv

# 啟用虛擬環境
# macOS/Linux
source .venv/bin/activate
# Windows
# .venv\Scripts\activate
```

### 安裝相依套件

```bash
# 使用 uvx 安裝相依套件
uv pip install jupyterlab jupyter-collaboration ipykernel 
uv pip install datalayer_pycrdt
uv pip install -e .  # 從本地安裝 jupyter-mcp-server
```

## 在 VSCode 中整合 MCP 服務器

VSCode 透過 Jupyter 擴充功能支援 Jupyter notebook。您可以讓 MCP 服務器與 VSCode 中的 Jupyter 整合，而不需要依賴 Docker。

### 前置需求

1. 安裝 [VSCode Jupyter 擴充功能](https://marketplace.visualstudio.com/items?itemName=ms-toolsai.jupyter)
2. 確保已安裝並正確設定 Python 環境
3. 如果還沒有，也安裝 [Claude Desktop 應用程式](https://claude.ai/download)

### 設定步驟

#### 1. 在 VSCode 中啟動 Jupyter 服務器

1. 在 VSCode 中開啟命令面板 (Cmd/Ctrl+Shift+P)
2. 輸入 `Jupyter: Create New Jupyter Notebook`
3. 選擇您的 Python 環境（應包含您使用 uvx 安裝的相依套件）
4. 查看 Jupyter 服務器狀態列以確認服務器正在運行，並記下 URL 與權杖

或者，您可以從終端直接啟動 Jupyter 服務器：

```bash
jupyter lab --port 8888 --IdentityProvider.token MY_TOKEN --ip 0.0.0.0
```

#### 2. 在本地運行 MCP 服務器

```bash
# 確保已啟用虛擬環境
source .venv/bin/activate

# 設定環境變數
export SERVER_URL="http://localhost:8888"
export TOKEN="MY_TOKEN"  # 使用您的 Jupyter 服務器權杖
export NOTEBOOK_PATH="path/to/your/notebook.ipynb"  # 相對於 Jupyter 服務器的筆記本路徑

# 啟動 MCP 服務器
python -m jupyter_mcp_server.server
```

#### 3. 設定 Claude Desktop 使用本地 MCP 服務器

為 Claude Desktop 建立配置檔案，指向本地 MCP 服務器而非 Docker 容器：

##### macOS/Linux
```bash
CLAUDE_CONFIG=${HOME}/.config/Claude/claude_desktop_config.json
cat <<EOF > $CLAUDE_CONFIG
{
  "mcpServers": {
    "jupyter": {
      "command": "${HOME}/path/to/your/.venv/bin/python",
      "args": [
        "-m",
        "jupyter_mcp_server.server"
      ],
      "env": {
        "SERVER_URL": "http://localhost:8888",
        "TOKEN": "MY_TOKEN",
        "NOTEBOOK_PATH": "notebook.ipynb"
      }
    }
  }
}
EOF
```

##### Windows
```
# 在 %USERPROFILE%\.config\Claude\ 目錄中建立 claude_desktop_config.json 檔案
{
  "mcpServers": {
    "jupyter": {
      "command": "C:\\path\\to\\your\\.venv\\Scripts\\python.exe",
      "args": [
        "-m",
        "jupyter_mcp_server.server"
      ],
      "env": {
        "SERVER_URL": "http://localhost:8888",
        "TOKEN": "MY_TOKEN",
        "NOTEBOOK_PATH": "notebook.ipynb"
      }
    }
  }
}
```

### 在 VSCode 中測試 MCP 整合

1. 在 VSCode 中開啟您的 Jupyter 筆記本
2. 啟動 Claude Desktop 並連接到您的筆記本
3. 使用 Claude 指示添加並執行程式碼單元
4. 觀察 VSCode 中的筆記本是否正確更新

### 故障排除

- **Python 環境問題**：確保 VSCode 使用的 Python 環境包含所有必要的相依套件
- **記憶體不足錯誤**：增加分配給 Python 的記憶體
- **MCP 服務器無法連接**：確認 SERVER_URL 和 TOKEN 環境變數正確設定，且 Jupyter 服務器可從本地訪問
- **VSCode Jupyter 擴充功能未顯示 notebook**：嘗試重新整理 VSCode 或重新啟動 Jupyter 服務器

## Start JupyterLab

Make sure you have the following installed. The collaboration package is needed as the modifications made on the notebook can be seen thanks to [Jupyter Real Time Collaboration](https://jupyterlab.readthedocs.io/en/stable/user/rtc.html).

```bash
pip install jupyterlab jupyter-collaboration ipykernel
pip uninstall -y pycrdt datalayer_pycrdt
pip install datalayer_pycrdt
```

Then, start JupyterLab with the following command.

```bash
jupyter lab --port 8888 --IdentityProvider.token MY_TOKEN --ip 0.0.0.0
```

You can also run `make jupyterlab`.

> [!NOTE]
>
> The `--ip` is set to `0.0.0.0` to allow the MCP server running in a Docker container to access your local JupyterLab.

## Use with Claude Desktop

Claude Desktop can be downloaded [from this page](https://claude.ai/download) for macOS and Windows.

For Linux, we had success using this [UNOFFICIAL build script based on nix](https://github.com/k3d3/claude-desktop-linux-flake)

```bash
# ⚠️ UNOFFICIAL
# You can also run `make claude-linux`
NIXPKGS_ALLOW_UNFREE=1 nix run github:k3d3/claude-desktop-linux-flake \
  --impure \
  --extra-experimental-features flakes \
  --extra-experimental-features nix-command
```

To use this with Claude Desktop, add the following to your `claude_desktop_config.json` (read more on the [MCP documentation website](https://modelcontextprotocol.io/quickstart/user#2-add-the-filesystem-mcp-server)).

> [!IMPORTANT]
>
> Ensure the port of the `SERVER_URL`and `TOKEN` match those used in the `jupyter lab` command.
>
> The `NOTEBOOK_PATH` should be relative to the directory where JupyterLab was started.

### Claude Configuration on macOS and Windows

```json
{
  "mcpServers": {
    "jupyter": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "SERVER_URL",
        "-e",
        "TOKEN",
        "-e",
        "NOTEBOOK_PATH",
        "datalayer/jupyter-mcp-server:latest"
      ],
      "env": {
        "SERVER_URL": "http://host.docker.internal:8888",
        "TOKEN": "MY_TOKEN",
        "NOTEBOOK_PATH": "notebook.ipynb"
      }
    }
  }
}
```

### Claude Configuration on Linux

```bash
CLAUDE_CONFIG=${HOME}/.config/Claude/claude_desktop_config.json
cat <<EOF > $CLAUDE_CONFIG
{
  "mcpServers": {
    "jupyter": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "SERVER_URL",
        "-e",
        "TOKEN",
        "-e",
        "NOTEBOOK_PATH",
        "--network=host",
        "datalayer/jupyter-mcp-server:latest"
      ],
      "env": {
        "SERVER_URL": "http://localhost:8888",
        "TOKEN": "MY_TOKEN",
        "NOTEBOOK_PATH": "notebook.ipynb"
      }
    }
  }
}
EOF
cat $CLAUDE_CONFIG
```

## Components

### Tools

The server currently offers 2 tools:

1. `add_execute_code_cell`

- Add and execute a code cell in a Jupyter notebook.
- Input:
  - `cell_content`(string): Code to be executed.
- Returns: Cell output.

2. `add_markdown_cell`

- Add a markdown cell in a Jupyter notebook.
- Input:
  - `cell_content`(string): Markdown content.
- Returns: Success message.

## Building

You can build the Docker image it from source.

```bash
make build-docker
```

## Installing via Smithery

To install Jupyter MCP Server for Claude Desktop automatically via [Smithery](https://smithery.ai/server/@datalayer/jupyter-mcp-server):

```bash
npx -y @smithery/cli install @datalayer/jupyter-mcp-server --client claude
```

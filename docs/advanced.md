## 進階

## 非互動／CI 模式

在自動化流程中以無頭方式執行 Codex。以下為 GitHub Action 步驟範例：

```yaml
- name: Update changelog via Codex
  run: |
    npm install -g @openai/codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    codex exec --full-auto "update CHANGELOG for next release"
```

## 追蹤／詳細記錄

由於 Codex 以 Rust 撰寫，因此可透過環境變數 `RUST_LOG` 來設定記錄行為。

TUI 預設為 `RUST_LOG=codex_core=info,codex_tui=info`，並將記錄輸出到 `~/.codex/log/codex-tui.log`。您可以在另一個終端機持續執行以下指令，以即時監看寫入的記錄：

```
tail -F ~/.codex/log/codex-tui.log
```

相較之下，非互動模式（`codex exec`）預設為 `RUST_LOG=error`，而訊息會直接列印在輸出中，因此無需另外監看檔案。

更多設定選項請參閱 Rust 文件［<a href="https://docs.rs/env_logger/latest/env_logger/#enabling-logging">`RUST_LOG`</a>］。

## Model Context Protocol（MCP）

您可以在 `~/.codex/config.toml` 中定義 [`mcp_servers`](./config.md#mcp_servers) 區段，讓 Codex CLI 使用 MCP 伺服器。其設計意圖與 Claude 與 Cursor 等工具在 JSON 組態檔中定義 `mcpServers` 的方式相似，但 Codex 採用 TOML 而非 JSON，因此格式略有不同，例如：

```toml
# IMPORTANT: the top-level key is `mcp_servers` rather than `mcpServers`.
[mcp_servers.server-name]
command = "npx"
args = ["-y", "mcp-server"]
env = { "API_KEY" = "value" }
```

> [!TIP]
> 這項功能仍屬實驗性質，不過 Codex CLI 也能透過 `codex mcp` 以 MCP「伺服器」身分執行。若您以 MCP 用戶端（例如 `npx @modelcontextprotocol/inspector codex mcp`）啟動並送出 `tools/list` 請求，您會看到只有一個名為 `codex` 的工具，能接受多樣化輸入，包含用於覆寫各項設定的通用 `config` 對映。歡迎試用並透過 GitHub Issues 回饋。

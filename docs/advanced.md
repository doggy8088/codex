## 進階

## 非互動式 / CI 模式

在管道中無頭執行 Codex。GitHub Action 步驟範例：

```yaml
- name: Update changelog via Codex
  run: |
    npm install -g @openai/codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    codex exec --full-auto "update CHANGELOG for next release"
```

## 追蹤 / 詳細記錄

因為 Codex 是用 Rust 撰寫的，它會遵循 `RUST_LOG` 環境變數來配置其記錄行為。

TUI 預設為 `RUST_LOG=codex_core=info,codex_tui=info`，記錄訊息會寫入 `~/.codex/log/codex-tui.log`，因此您可以在另一個終端中執行以下命令來監控記錄訊息的寫入：

```
tail -F ~/.codex/log/codex-tui.log
```

相比之下，非互動式模式（`codex exec`）預設為 `RUST_LOG=error`，但訊息會內嵌列印，因此無需監控額外的檔案。

有關配置選項的更多資訊，請參閱 Rust 關於 [`RUST_LOG`](https://docs.rs/env_logger/latest/env_logger/#enabling-logging) 的文件。

## Model Context Protocol (MCP)

可以透過在 `~/.codex/config.toml` 中定義 [`mcp_servers`](./config.md#mcp_servers) 區段來配置 Codex CLI 以利用 MCP 伺服器。它旨在模仿 Claude 和 Cursor 等工具在其各自 JSON 配置檔案中定義 `mcpServers` 的方式，儘管 Codex 格式略有不同，因為它使用 TOML 而非 JSON，例如：

```toml
# 重要：頂層鍵是 `mcp_servers` 而非 `mcpServers`。
[mcp_servers.server-name]
command = "npx"
args = ["-y", "mcp-server"]
env = { "API_KEY" = "value" }
```

> [!TIP]
> 雖然有些實驗性，但 Codex CLI 也可以透過 `codex mcp` 作為 MCP _伺服器_ 執行。如果您使用 MCP 客戶端（如 `npx @modelcontextprotocol/inspector codex mcp`）啟動它並發送 `tools/list` 請求，您會看到只有一個工具 `codex`，它接受各種輸入，包括一個用於您可能想要覆蓋的任何內容的通用 `config` 映射。歡迎試用並透過 GitHub issues 提供意見回饋。
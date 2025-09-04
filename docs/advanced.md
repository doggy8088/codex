## 進階

## 非互動式 / CI 模式

在管道中以無介面模式執行 Codex。GitHub Action 步驟範例：

```yaml
- name: Update changelog via Codex
  run: |
    npm install -g @openai/codex
    export OPENAI_API_KEY="${{ secrets.OPENAI_KEY }}"
    codex exec --full-auto "update CHANGELOG for next release"
```

## 追蹤 / 詳細日誌

因為 Codex 是用 Rust 語言撰寫的，它會遵循 `RUST_LOG` 環境變數來設定其日誌行為。

TUI 預設為 `RUST_LOG=codex_core=info,codex_tui=info`，且日誌訊息會寫入 `~/.codex/log/codex-tui.log`，所以您可以在另一個終端機中保持以下指令執行，以監控日誌訊息的寫入：

```
tail -F ~/.codex/log/codex-tui.log
```

相較之下，非互動式模式 (`codex exec`) 預設為 `RUST_LOG=error`，但訊息會直接印出，因此無需監控另一個檔案。

有關設定選項的更多資訊，請參閱 Rust 關於 [`RUST_LOG`](https://docs.rs/env_logger/latest/env_logger/#enabling-logging) 的文件。

## 模型上下文協議 (MCP)

Codex CLI 可透過在 `~/.codex/config.toml` 中定義一個 [`mcp_servers`](./config.md#mcp_servers) 區段來設定以利用 MCP 伺服器。此舉旨在模仿 Claude 和 Cursor 等工具在其各自的 JSON 設定檔中定義 `mcpServers` 的方式，不過 Codex 的格式略有不同，因為它使用 TOML 而非 JSON，例如：

```toml
# IMPORTANT: the top-level key is `mcp_servers` rather than `mcpServers`.
[mcp_servers.server-name]
command = "npx"
args = ["-y", "mcp-server"]
env = { "API_KEY" = "value" }
```

> [!TIP]
> 這項功能尚在實驗階段，但 Codex CLI 也可以透過 `codex mcp` 作為 MCP _伺服器_ 執行。如果您使用像 `npx @modelcontextprotocol/inspector codex mcp` 這樣的 MCP 客戶端啟動它，並向其發送一個 `tools/list` 請求，您將會看到只有一個工具 `codex`，它接受各種輸入，包括一個用於覆寫任何您想修改的設定的 `config` 萬用對應。歡迎隨意試用並透過 GitHub issues 提供回饋。
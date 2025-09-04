# Codex CLI（Rust 實作）

我們以獨立的原生可執行檔提供 Codex CLI，確保安裝零相依。

## 安裝 Codex

目前安裝 Codex 最簡單的方式是透過 `npm`，未來我們也會發佈到更多套件管理器。

```shell
npm i -g @openai/codex@native
codex
```

您也可以直接從［<a href="https://github.com/openai/codex/releases">GitHub Releases</a>］下載對應您平台的版本。

## Rust 版 CLI 的新功能

我們正[致力於縮小 TypeScript 與 Rust 兩個 Codex CLI 實作之間的差距](https://github.com/openai/codex/issues/1262)，同時也請注意 Rust 版 CLI 具備 TypeScript 版所沒有的多項功能！

### 設定

Codex 支援豐富的設定選項。請注意 Rust 版 CLI 使用 `config.toml`（而非 `config.json`）。詳細請見［<a href="../docs/config.md">`docs/config.md`</a>］。

### Model Context Protocol 支援

Codex CLI 可作為 MCP 用戶端，啟動時連線到 MCP 伺服器。詳見組態文件中的［<a href="../docs/config.md#mcp_servers">`mcp_servers`</a>］章節。

此外雖仍屬實驗性質，但您也可以執行 `codex mcp` 將 Codex 以 MCP「伺服器」身分啟動。可搭配［<a href="https://github.com/modelcontextprotocol/inspector">`@modelcontextprotocol/inspector`</a>］試用：

```shell
npx @modelcontextprotocol/inspector codex mcp
```

### 通知

您可以設定在代理完成一回合時執行的腳本，以啟用通知。請見［<a href="../docs/config.md#notify">通知文件</a>］，其中包含如何透過 macOS 上的 [terminal-notifier](https://github.com/julienXX/terminal-notifier) 取得桌面通知的詳細範例。

### 使用 `codex exec` 以程式化／非互動方式執行 Codex

若要以非互動方式執行，請運行 `codex exec PROMPT`（也可透過 `stdin` 傳入提示）。Codex 會持續處理您的任務，直到判定完成並結束。輸出會直接列印到終端機；您可設定 `RUST_LOG` 環境變數以查看更詳細的運作情形。

### 使用 `@` 進行檔案搜尋

輸入 `@` 會在工作區根目錄觸發模糊檔名搜尋。使用上下鍵選擇結果，按 Tab 或 Enter 以所選路徑取代 `@`。按 Esc 可取消搜尋。

### 連按 Esc 編輯上一則訊息

當輸入區為空時，按 Esc 以啟動「回溯」模式。再按一次 Esc 會開啟對話稿預覽並反白最後一則使用者訊息；重複按 Esc 可逐步回到更早的訊息。按 Enter 確認後，Codex 會自該處分叉對話、相應裁剪可見對話稿，並預先填入所選的使用者訊息，方便您編輯後重新送出。

在對話稿預覽中，啟用編輯時頁尾會顯示 `Esc edit prev` 提示。

### `--cd`/`-C` 旗標

有時在啟動 Codex 前先 `cd` 到您要使用的「工作根目錄」並不方便。所幸 `codex` 支援 `--cd` 選項，您可以指定任意資料夾。開始新會話時，請在 TUI 中查看 **workdir** 以確認 Codex 確實採用了 `--cd` 指定的路徑。

### Shell 自動補全

您可以這樣產生 shell 自動補全腳本：

```shell
codex completion bash
codex completion zsh
codex completion fish
```

### 體驗 Codex 的沙盒

若要測試在 Codex 提供的沙盒中執行指令的行為，我們在 Codex CLI 中提供以下子指令：

```
# macOS
codex debug seatbelt [--full-auto] [COMMAND]...

# Linux
codex debug landlock [--full-auto] [COMMAND]...
```

### 透過 `--sandbox` 選擇沙盒策略

Rust 版 CLI 提供專屬的 `--sandbox`（`-s`）旗標，讓您不用動用通用的 `-c/--config` 選項即可選擇沙盒策略：

```shell
# 以預設唯讀沙盒執行 Codex
codex --sandbox read-only

# 允許代理在目前工作區內寫入，同時仍封鎖網路存取
codex --sandbox workspace-write

# 危險！完全停用沙盒（僅在您已於容器或其他隔離環境中執行時使用）
codex --sandbox danger-full-access
```

同樣的設定也可透過 `~/.codex/config.toml` 的頂層鍵 `sandbox_mode = "MODE"` 永久化，例如 `sandbox_mode = "workspace-write"`。

## 程式碼結構

此資料夾是 Cargo 工作區的根目錄。內容包含許多實驗性程式碼，但以下是關鍵 crate：

- [`core/`](./core)：包含 Codex 的商業邏輯。最終希望成為一個通用的函式庫 crate，方便其他使用 Codex 的 Rust/原生應用程式。
- [`exec/`](./exec)：用於自動化的「無頭」CLI。
- [`tui/`](./tui)：啟動以 [Ratatui](https://ratatui.rs/) 打造的全螢幕 TUI 的 CLI。
- [`cli/`](./cli)：CLI 多功能工具，透過子指令提供上述各種 CLI。

## 開始使用

### CLI 使用方式

| 指令               | 目的                       | 範例                            |
| ------------------ | -------------------------- | ------------------------------- |
| `codex`            | 互動式 TUI                 | `codex`                         |
| `codex "..."`      | 互動式 TUI 的初始提示      | `codex "修復 lint 錯誤"`        |
| `codex exec "..."` | 非互動式「自動化模式」     | `codex exec "解釋 utils.ts"`    |

關鍵標誌：`--model/-m`、`--ask-for-approval/-a`。

恢復選項：

- `--resume`：開啟最近對話的互動式選擇器（顯示第一個真實使用者訊息的預覽）。與 `--continue` 衝突。
- `--continue`：恢復最近的對話而不顯示選擇器（如果不存在則回退到重新開始）。與 `--resume` 衝突。

範例：

```shell
codex --resume
codex --continue
```

### 以提示作為輸入運行

您也可以使用提示作為輸入來運行 Codex CLI：

```shell
codex "向我解釋這個程式碼庫"
```

```shell
codex --full-auto "建立最精美的待辦事項應用程式"
```

就是這樣 - Codex 將建構檔案、在沙盒中運行、安裝任何遺漏的相依項目，並顯示實時結果。核准變更後，它們將提交到您的工作目錄。

### 範例提示

以下是一些您可以複製貼上的簡短範例。將引號中的文字替換為您自己的任務。更多技巧和使用模式請參閱 [提示指南](https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md)。

| ✨  | 您輸入的內容                                                                   | 會發生什麼                                                                 |
| --- | ------------------------------------------------------------------------------ | -------------------------------------------------------------------------- |
| 1   | `codex "將 Dashboard 元件重構為 React Hooks"`                                 | Codex 重寫類別元件、執行 `npm test` 並顯示差異。                          |
| 2   | `codex "產生新增使用者表的 SQL 遷移"`                                          | 推斷您的 ORM、建立遷移檔案，並在沙盒資料庫中執行它們。                    |
| 3   | `codex "為 utils/date.ts 撰寫單元測試"`                                        | 產生測試、執行它們，並迭代直到通過。                                      |
| 4   | `codex "使用 git mv 批量重新命名 *.jpeg -> *.jpg"`                            | 安全地重新命名檔案並更新匯入/使用。                                        |
| 5   | `codex "解釋這個正規表達式的作用：^(?=.*[A-Z]).{8,}$"`                        | 輸出逐步的人類解釋。                                                      |
| 6   | `codex "仔細審查這個儲存庫，並提議 3 個高影響力且範圍明確的 PR"`              | 在當前程式碼庫中建議有影響力的 PR。                                       |
| 7   | `codex "尋找漏洞並建立安全審查報告"`                                          | 找出並解釋安全漏洞。                                                      |

### 使用 AGENTS.md 進行記憶

您可以使用 `AGENTS.md` 檔案為 Codex 提供額外的指示和指導。Codex 會在以下位置尋找 `AGENTS.md` 檔案，並由上而下合併它們：

1. `~/.codex/AGENTS.md` - 個人全域指導
2. 儲存庫根目錄的 `AGENTS.md` - 共享專案說明
3. 當前工作目錄中的 `AGENTS.md` - 子資料夾/功能特定內容

有關如何使用 AGENTS.md 的更多資訊，請參閱 [AGENTS.md 官方文件](https://agents.md/)。

### 技巧與快捷鍵

#### Use `@` for file search

Typing `@` triggers a fuzzy-filename search over the workspace root. Use up/down to select among the results and Tab or Enter to replace the `@` with the selected path. You can use Esc to cancel the search.

#### Image input

Paste images directly into the composer (Ctrl+V / Cmd+V) to attach them to your prompt. You can also attach files via the CLI using `-i/--image` (comma‑separated):

```bash
codex -i screenshot.png "Explain this error"
codex --image img1.png,img2.jpg "Summarize these diagrams"
```

#### Esc–Esc to edit a previous message

When the chat composer is empty, press Esc to prime “backtrack” mode. Press Esc again to open a transcript preview highlighting the last user message; press Esc repeatedly to step to older user messages. Press Enter to confirm and Codex will fork the conversation from that point, trim the visible transcript accordingly, and pre‑fill the composer with the selected user message so you can edit and resubmit it.

In the transcript preview, the footer shows an `Esc edit prev` hint while editing is active.

#### Shell completions

Generate shell completion scripts via:

```shell
codex completion bash
codex completion zsh
codex completion fish
```

#### `--cd`/`-C` flag

Sometimes it is not convenient to `cd` to the directory you want Codex to use as the "working root" before running Codex. Fortunately, `codex` supports a `--cd` option so you can specify whatever folder you want. You can confirm that Codex is honoring `--cd` by double-checking the **workdir** it reports in the TUI at the start of a new session.

## 開始使用

### CLI 使用

| 命令               | 目的                                | 範例                               |
| ------------------ | ---------------------------------- | ---------------------------------- |
| `codex`            | 互動式 TUI                         | `codex`                            |
| `codex "..."`      | 互動式 TUI 的初始提示              | `codex "fix lint errors"`          |
| `codex exec "..."` | 非互動式「自動化模式」              | `codex exec "explain utils.ts"`    |

關鍵旗標：`--model/-m`、`--ask-for-approval/-a`。

恢復選項：

- `--resume`：開啟最近會話的互動式選擇器（顯示第一個真實用戶訊息的預覽）。與 `--continue` 衝突。
- `--continue`：恢復最近的會話而不顯示選擇器（如果不存在則會回退到重新開始）。與 `--resume` 衝突。

範例：

```shell
codex --resume
codex --continue
```

### 以提示作為輸入執行

您也可以以提示作為輸入執行 Codex CLI：

```shell
codex "explain this codebase to me"
```

```shell
codex --full-auto "create the fanciest todo-list app"
```

就是這樣 - Codex 會建立檔案、在沙盒中執行、安裝任何缺少的相依套件，並顯示實際結果。核准變更後，它們會被提交到您的工作目錄。

### 範例提示

以下是一些可以複製貼上的小範例。將引號中的文字替換為您自己的任務。請參閱[提示指南](https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md)以獲得更多提示和使用模式。

| ✨  | 您輸入的內容                                                                     | 發生什麼事                                                                      |
| --- | ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| 1   | `codex "Refactor the Dashboard component to React Hooks"`                       | Codex 重寫類別組件，執行 `npm test`，並顯示差異。                              |
| 2   | `codex "Generate SQL migrations for adding a users table"`                      | 推斷您的 ORM，建立遷移檔案，並在沙盒資料庫中執行它們。                         |
| 3   | `codex "Write unit tests for utils/date.ts"`                                    | 產生測試，執行它們，並迭代直到通過。                                           |
| 4   | `codex "Bulk-rename *.jpeg -> *.jpg with git mv"`                               | 安全地重新命名檔案並更新匯入/使用。                                            |
| 5   | `codex "Explain what this regex does: ^(?=.*[A-Z]).{8,}$"`                      | 輸出逐步的人類解釋。                                                          |
| 6   | `codex "Carefully review this repo, and propose 3 high impact well-scoped PRs"` | 建議目前程式碼庫中具影響力的 PR。                                             |
| 7   | `codex "Look for vulnerabilities and create a security review report"`          | 找出並解釋安全漏洞。                                                          |

### AGENTS.md 記憶功能

您可以使用 `AGENTS.md` 檔案為 Codex 提供額外的指示和指導。Codex 會在以下位置尋找 `AGENTS.md` 檔案，並由上而下合併：

1. `~/.codex/AGENTS.md` - 個人全域指導
2. 儲存庫根目錄的 `AGENTS.md` - 共享專案備註
3. 目前工作目錄中的 `AGENTS.md` - 子資料夾/功能特定資訊

有關如何使用 AGENTS.md 的更多資訊，請參閱[官方 AGENTS.md 文件](https://agents.md/)。

### 提示與快捷鍵

#### 使用 `@` 進行檔案搜尋

輸入 `@` 會觸發工作空間根目錄的模糊檔案名搜尋。使用上/下選擇結果，使用 Tab 或 Enter 將 `@` 替換為選定的路徑。您可以使用 Esc 取消搜尋。

#### 圖片輸入

直接將圖片貼到編輯器中（Ctrl+V / Cmd+V）以將其附加到您的提示。您也可以透過 CLI 使用 `-i/--image`（逗號分隔）附加檔案：

```bash
codex -i screenshot.png "Explain this error"
codex --image img1.png,img2.jpg "Summarize these diagrams"
```

#### Esc–Esc 編輯先前的訊息

當對話編輯器為空時，按 Esc 啟動「回溯」模式。再次按 Esc 開啟突出顯示最後一個用戶訊息的對話記錄預覽；重複按 Esc 逐步回到較舊的用戶訊息。按 Enter 確認，Codex 會從該點分岔對話，相應地修剪可見對話記錄，並用選定的用戶訊息預填編輯器，以便您可以編輯並重新提交。

在對話記錄預覽中，頁腳會在編輯作用中時顯示 `Esc edit prev` 提示。

#### Shell 補全

透過以下方式產生 shell 補全腳本：

```shell
codex completion bash
codex completion zsh
codex completion fish
```

#### `--cd`/`-C` 旗標

有時不方便在執行 Codex 之前 `cd` 到您希望 Codex 用作「工作根目錄」的目錄。幸運的是，`codex` 支援 `--cd` 選項，讓您可以指定任何所需的資料夾。您可以在新會話開始時檢查 TUI 中報告的 **workdir** 來確認 Codex 是否遵循 `--cd`。
## 快速開始

### CLI 用法

| 指令                | 用途                                 | 範例                              |
| ------------------ | ---------------------------------- | ------------------------------- |
| `codex`            | 互動式 TUI                          | `codex`                         |
| `codex "..."`      | 互動式 TUI 的初始提示字串             | `codex "fix lint errors"`       |
| `codex exec "..."` | 非互動「自動化模式」                  | `codex exec "explain utils.ts"` |

重要旗標：`--model/-m`、`--ask-for-approval/-a`。

繼續會話選項：

- `--resume`：開啟最近會話的互動式選擇器（會預覽第一則實際的使用者訊息）。與 `--continue` 衝突。
- `--continue`：直接延續最近一次會話，不顯示選擇器（若不存在則回退為全新開始）。與 `--resume` 衝突。

範例：

```shell
codex --resume
codex --continue
```

### 以提示字串作為輸入執行

您也可以以提示字串作為輸入來執行 Codex CLI：

```shell
codex "explain this codebase to me"
```

```shell
codex --full-auto "create the fanciest todo-list app"
```

就這樣——Codex 會產生腳手架檔案、在沙盒中執行、安裝缺少的相依套件，並即時顯示結果。當您核准變更後，這些變更會提交到您的工作目錄。

### 提示字串範例

以下是幾個可直接複製貼上的小範例。請把引號內的文字替換成您自己的工作內容。更多技巧與使用範式請參閱［<a href="https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md">提示字串指南</a>］。

| ✨  | 您輸入的內容                                                                      | 系統會做什麼                                                                |
| --- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 1   | `codex "Refactor the Dashboard component to React Hooks"`                       | Codex 會將類別元件改寫為 Hooks，執行 `npm test`，並顯示差異。            |
| 2   | `codex "Generate SQL migrations for adding a users table"`                      | 推斷您的 ORM、建立遷移檔，並在沙盒化的資料庫中執行。                     |
| 3   | `codex "Write unit tests for utils/date.ts"`                                    | 產生測試、執行測試，並反覆修正直到通過。                                 |
| 4   | `codex "Bulk-rename *.jpeg -> *.jpg with git mv"`                               | 安全地重新命名檔案並更新匯入/使用處。                                     |
| 5   | `codex "Explain what this regex does: ^(?=.*[A-Z]).{8,}$"`                      | 產生逐步的人類可讀說明。                                                   |
| 6   | `codex "Carefully review this repo, and propose 3 high impact well-scoped PRs"` | 在現有程式碼庫提出影響力高且範圍明確的 PR 建議。                         |
| 7   | `codex "Look for vulnerabilities and create a security review report"`          | 尋找並說明安全性問題。                                                     |

### 使用 AGENTS.md 的記憶體

您可以透過 `AGENTS.md` 檔案提供 Codex 額外的指示與引導。Codex 會在下列位置尋找 `AGENTS.md`，並自上而下合併：

1. `~/.codex/AGENTS.md` - 個人全域指引
2. 儲存庫根目錄的 `AGENTS.md` - 專案共用筆記
3. 目前工作目錄中的 `AGENTS.md` - 子資料夾/功能的具體說明

如需更多 AGENTS.md 的使用方式，請參閱［<a href="https://agents.md/">官方 AGENTS.md 文件</a>］。

### 小撇步與快捷方式

#### 使用 `@` 進行檔案搜尋

輸入 `@` 會在工作區根目錄觸發模糊檔名搜尋。使用上下鍵在結果中選擇，按 Tab 或 Enter 以所選路徑取代 `@`。按 Esc 可取消搜尋。

#### 影像輸入

將影像直接貼到輸入區（Ctrl+V / Cmd+V）以附加到提示字串。您也可以透過 CLI 使用 `-i/--image`（以逗號分隔）附加檔案：

```bash
codex -i screenshot.png "Explain this error"
codex --image img1.png,img2.jpg "Summarize these diagrams"
```

#### 連按 Esc 編輯上一則訊息

當輸入區為空時，按 Esc 以啟動「回溯」模式。再按一次 Esc 會開啟對話稿預覽並反白最後一則使用者訊息；重複按 Esc 可逐步回到更早的訊息。按 Enter 確認後，Codex 會自該處分叉對話、相應裁剪可見對話稿，並預先填入所選的使用者訊息，方便您編輯後重新送出。

在對話稿預覽中，啟用編輯時頁尾會顯示 `Esc edit prev` 提示。

#### Shell 自動補全

您可以這樣產生 shell 自動補全腳本：

```shell
codex completion bash
codex completion zsh
codex completion fish
```

#### `--cd`/`-C` 旗標

有時在啟動 Codex 前先 `cd` 到您要使用的「工作根目錄」並不方便。所幸 `codex` 支援 `--cd` 選項，您可以指定任意資料夾。開始新會話時，請在 TUI 中查看 **workdir** 以確認 Codex 確實採用了 `--cd` 指定的路徑。

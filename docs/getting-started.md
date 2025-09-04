## 入門指南

### CLI 用法

| 指令               | 用途                               | 範例                            |
| ------------------ | ---------------------------------- | ------------------------------- |
| `codex`            | 互動式 TUI                         | `codex`                         |
| `codex "..."`      | 用於互動式 TUI 的初始提示          | `codex "fix lint errors"`       |
| `codex exec "..."` | 非互動式「自動化模式」             | `codex exec "explain utils.ts"` |

主要旗標：`--model/-m`、`--ask-for-approval/-a`。

續存選項：

- `--resume`：開啟一個互動式選擇器以選擇最近的對話階段（顯示第一條真實使用者訊息的預覽）。與 `--continue` 衝突。
- `--continue`：續存最近的對話階段，而不顯示選擇器（如果不存在任何對話階段，則退回至重新開始）。與 `--resume` 衝突。

範例：

```shell
codex --resume
codex --continue
```

### 以提示作為輸入執行

您也可以使用提示作為輸入來執行 Codex CLI：

```shell
codex "explain this codebase to me"
```

```shell
codex --full-auto "create the fanciest todo-list app"
```

就這麼簡單 - Codex 將會建構檔案骨架、在沙箱內執行它、安裝任何
缺失的相依套件，並向您顯示即時結果。批准變更後，
它們將會被提交至您的工作目錄。

### 提示範例

以下是一些您可以複製貼上的簡短範例。請將引號中的文字替換為您自己的任務。欲了解更多提示技巧和使用模式，請參閱 [提示指南](https://github.com/openai/codex/blob/main/codex-cli/examples/prompting_guide.md)。

| ✨  | 您輸入的內容                                                                    | 會發生什麼事                                                                |
| --- | ------------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 1   | `codex "將 Dashboard 元件重構成 React Hooks"`                       | Codex 會重寫類別元件，執行 `npm test`，並顯示差異。    |
| 2   | `codex "為新增 users 資料表產生 SQL 遷移"`                      | 推斷您的 ORM，建立遷移檔案，並在沙盒資料庫中執行。 |
| 3   | `codex "為 utils/date.ts 撰寫單元測試"`                                     | 產生測試，執行它們，並迭代直到通過為止。               |
| 4   | `codex "使用 git mv 大量重新命名 *.jpeg -> *.jpg"`                                | 安全地重新命名檔案並更新匯入/使用之處。                            |
| 5   | `codex "解釋這個正規表達式的作用：^(?=.*[A-Z]).{8,}$"`                      | 輸出一步步的人性化解釋。                                   |
| 6   | `codex "仔細審查此 repo，並提出 3 個高影響力且範圍明確的 PR"` | 在目前的程式碼庫中建議有影響力的 PR。                            |
| 7   | `codex "尋找漏洞並建立一份安全審查報告"`          | 找出並解釋安全性錯誤。                                           |

### 使用 AGENTS.md 的記憶體

您可以使用 `AGENTS.md` 檔案為 Codex 提供額外的指令和指引。Codex 會在以下位置尋找 `AGENTS.md` 檔案，並由上而下合併它們：

1. `~/.codex/AGENTS.md` - 個人的全域指引
2. `AGENTS.md` 在 repo 根目錄 - 共用的專案筆記
3. `AGENTS.md` 在目前工作目錄 - 子資料夾/特定功能

有關如何使用 AGENTS.md 的更多資訊，請參閱 [AGENTS.md 官方文件](https://agents.md/)。

### 提示與捷徑

#### 使用 `@` 進行檔案搜尋

輸入 `@` 會觸發工作區根目錄的模糊檔名搜尋。使用上/下鍵在結果中選取，並按 Tab 或 Enter 以所選路徑取代 `@`。您可以按 Esc 取消搜尋。

#### 圖片輸入

將圖片直接貼到編輯器中 (Ctrl+V / Cmd+V) 以附加到您的提示。您也可以透過 CLI 使用 `-i/--image` (以逗號分隔) 附加檔案：

```bash
codex -i screenshot.png "Explain this error"
codex --image img1.png,img2.jpg "Summarize these diagrams"
```

#### 按兩次 Esc 編輯先前的訊息

當聊天撰寫區為空時，按下 Esc 鍵以準備進入「回溯」模式。再按一次 Esc 鍵以開啟對話紀錄預覽，並突顯最後一則使用者訊息；重複按下 Esc 鍵可逐步跳至更早的使用者訊息。按下 Enter 鍵確認後，Codex 將從該點分岔對話、相應地修剪可見的對話紀錄，並將所選的使用者訊息預先填入撰寫區，以便您編輯和重新提交。

在對話紀錄預覽中，當編輯功能啟用時，頁腳會顯示 `Esc edit prev` 的提示。

#### Shell 自動補全

透過以下方式產生 shell 自動補全腳本：

```shell
codex completion bash
codex completion zsh
codex completion fish
```

#### `--cd`/`-C` 旗標

有時在執行 Codex 之前，要先 `cd` 到您希望 Codex 當作「工作根目錄」的資料夾並不方便。幸運的是，`codex` 支援 `--cd` 選項，讓您可以指定任何想要的資料夾。您可以透過再次檢查新會話開始時 TUI 中回報的 **workdir**，來確認 Codex 是否遵循 `--cd` 的設定。

## 沙箱與核准

### 核准模式

我們為 Codex 在您電腦上的運作方式選擇了一個強大的預設值：`Auto`。在此核准模式下，Codex 可以自動讀取檔案、進行編輯，並在工作目錄中執行指令。然而，Codex 若要存取工作目錄以外的內容或網路，則需要您的核准。

當您只想聊天，或想在深入操作前進行規劃時，可以使用 `/approvals` 指令切換到 `Read Only`（唯讀）模式。

如果您需要 Codex 在未經核准的情況下讀取檔案、進行編輯，並執行有網路存取權的指令，您可以使用 `Full Access`（完整存取權）。在使用前請務必謹慎。

#### 預設值與建議

- Codex 預設在沙箱中執行，並有嚴格的保護措施：它會防止編輯工作區外的檔案，並在未啟用時阻擋網路存取。
- 啟動時，Codex 會偵測資料夾是否受版本控制，並建議：
  - 版本控制的資料夾：`Auto`（工作區寫入 + 依請求核准）
  - 非版本控制的資料夾：`Read Only`（唯讀）
- 工作區包含當前目錄和如 `/tmp` 的暫存目錄。您可以使用 `/status` 指令查看哪些目錄在工作區內。
- 您可以明確設定這些項目：
  - `codex --sandbox workspace-write --ask-for-approval on-request`
  - `codex --sandbox read-only --ask-for-approval on-request`

### 我可以在沒有任何核准的情況下執行嗎？

是的，您可以使用 `--ask-for-approval never` 來停用所有核准提示。此選項適用於所有 `--sandbox` 模式，因此您仍然可以完全控制 Codex 的自主程度。它會在您提供的任何限制下盡力而為。

### 常見的沙箱與核准組合

| 意圖                                  | 旗標                                                                                   | 效果                                                                                    |
| --------------------------------------- | ----------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| 安全的唯讀瀏覽                        | `--sandbox read-only --ask-for-approval on-request`                                            | Codex 可以讀取檔案並回答問題。Codex 需要核准才能進行編輯、執行指令或存取網路。 |
| 唯讀非互動式 (CI)                     | `--sandbox read-only --ask-for-approval never`                                                 | 僅讀取；絕不提升權限                                                                    |
| 允許編輯儲存庫，有風險時詢問        | `--sandbox workspace-write --ask-for-approval on-request`                                      | Codex 可以在工作區內讀取檔案、進行編輯及執行指令。對於工作區外的操作或網路存取，Codex 需要核准。 |
| 自動（預設）                          | `--full-auto` (相當於 `--sandbox workspace-write` + `--ask-for-approval on-failure`)          | Codex 可以在工作區內讀取檔案、進行編輯及執行指令。當沙箱內的指令失敗或需要提升權限時，Codex 需要核准。 |
| YOLO（不建議）                        | `--dangerously-bypass-approvals-and-sandbox` (別名: `--yolo`)                                  | 無沙箱；無提示                                                                            |

> 注意：在 `workspace-write` 模式中，網路預設為停用，除非在設定檔中啟用（`[sandbox_workspace_write].network_access = true`）。

#### 在 `config.toml` 中微調

```toml
# approval mode
approval_policy = "untrusted"
sandbox_mode    = "read-only"

# full-auto mode
approval_policy = "on-request"
sandbox_mode    = "workspace-write"

# Optional: allow network in workspace-write mode
[sandbox_workspace_write]
network_access = true
```

您也可以將預設組態儲存為 **設定檔**：

```toml
[profiles.full_auto]
approval_policy = "on-request"
sandbox_mode    = "workspace-write"

[profiles.readonly_quiet]
approval_policy = "never"
sandbox_mode    = "read-only"
```

### 實驗 Codex 沙箱

為了測試指令在 Codex 提供的沙箱下執行時會發生什麼情況，我們在 Codex CLI 中提供了以下子指令：

```
# macOS
codex debug seatbelt [--full-auto] [COMMAND]...

# Linux
codex debug landlock [--full-auto] [COMMAND]...
```

### 平台沙箱詳細資訊

Codex 用於實作沙箱策略的機制取決於您的作業系統：

- **macOS 12+** 使用 **Apple Seatbelt**，並透過 `sandbox-exec` 搭配與指定的 `--sandbox` 相對應的設定檔 (`-p`) 來執行指令。
- **Linux** 結合使用 Landlock/seccomp API 來強制執行 `sandbox` 設定。

請注意，在 Docker 等容器化環境中執行 Linux 時，如果主機/容器的設定不支援必要的 Landlock/seccomp API，沙箱功能可能無法運作。在這種情況下，我們建議您設定 Docker 容器，使其提供您所尋求的沙箱保障，然後在容器內使用 `--sandbox danger-full-access`（或更簡單地說，使用 `--dangerously-bypass-approvals-and-sandbox` 旗標）來執行 `codex`。
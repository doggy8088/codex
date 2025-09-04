## 沙盒與核准

### 核准模式

我們為 Codex 在您電腦上的運作選擇了功能完善的預設模式：`Auto`。在此模式下，Codex 可自動讀取檔案、進行編輯，並在工作目錄中執行指令。然而，若要在工作目錄之外作業或存取網路，Codex 仍需您的核准。

如果您暫時只想聊天，或想先規劃再動手，可使用 `/approvals` 指令切換為 `Read Only` 模式。

若您需要 Codex 無需核准即可讀取檔案、編輯與具備網路存取執行指令，您可以使用 `Full Access`。請審慎使用。

#### 預設與建議

- Codex 預設在沙盒中執行並具備強力防護：未啟用前，會阻止在工作區外編輯檔案並封鎖網路存取。
- 啟動時，Codex 會偵測資料夾是否受版本控制並提出建議：
  - 受版本控制的資料夾：`Auto`（工作區可寫入 + on-request 核准）
  - 非版本控制資料夾：`Read Only`
- 工作區包含目前目錄與 `/tmp` 等暫存目錄。使用 `/status` 可查看工作區涵蓋哪些路徑。
- 您也可明確指定：
  - `codex --sandbox workspace-write --ask-for-approval on-request`
  - `codex --sandbox read-only --ask-for-approval on-request`

### 可以完全不需核准就執行嗎？

可以，您可使用 `--ask-for-approval never` 停用所有核准提示。此選項適用於各種 `--sandbox` 模式，因此您仍可完全掌控 Codex 的自主程度。Codex 會在您提供的限制下盡力完成工作。

### 常見沙盒與核准組合

| 目的                                   | 旗標                                                                                             | 效果                                                                                               |
| -------------------------------------- | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------- |
| 安全的唯讀瀏覽                         | `--sandbox read-only --ask-for-approval on-request`                                             | Codex 可讀取檔案並回答問題；編輯、執行指令或網路存取需核准。                                          |
| 唯讀非互動（CI）                       | `--sandbox read-only --ask-for-approval never`                                                  | 僅唯讀；不會升級權限。                                                                                |
| 允許編輯儲存庫，風險操作再詢問         | `--sandbox workspace-write --ask-for-approval on-request`                                       | Codex 可在工作區讀取、編輯與執行指令；工作區外或網路存取需核准。                                      |
| Auto（預設組合）                       | `--full-auto`（等同 `--sandbox workspace-write` + `--ask-for-approval on-failure`）             | Codex 可在工作區讀取、編輯與執行指令；沙盒指令失敗或需升級時會請求核准。                                |
| YOLO（不建議）                         | `--dangerously-bypass-approvals-and-sandbox`（別名：`--yolo`）                                  | 無沙盒；不提示。                                                                                    |

> 注意：在 `workspace-write` 中，除非組態啟用（`[sandbox_workspace_write].network_access = true`），否則預設停用網路。

#### 在 `config.toml` 進行微調

```toml
# 核准模式
approval_policy = "untrusted"
sandbox_mode    = "read-only"

# full-auto 模式
approval_policy = "on-request"
sandbox_mode    = "workspace-write"

# 可選：在 workspace-write 模式允許網路
[sandbox_workspace_write]
network_access = true
```

您也可以將預設組合儲存為 **profiles**：

```toml
[profiles.full_auto]
approval_policy = "on-request"
sandbox_mode    = "workspace-write"

[profiles.readonly_quiet]
approval_policy = "never"
sandbox_mode    = "read-only"
```

### 體驗 Codex 的沙盒

若要測試在 Codex 提供的沙盒中執行指令的行為，我們在 Codex CLI 中提供以下子指令：

```
# macOS
codex debug seatbelt [--full-auto] [COMMAND]...

# Linux
codex debug landlock [--full-auto] [COMMAND]...
```

### 各平台沙盒機制細節

Codex 採用的沙盒機制取決於您的作業系統：

- **macOS 12+**：使用 **Apple Seatbelt**，並透過 `sandbox-exec` 及對應 `--sandbox` 的設定檔（`-p`）執行指令。
- **Linux**：使用 Landlock/seccomp API 的組合來強制執行 `sandbox` 組態。

請注意：在 Docker 等容器化環境執行 Linux 時，若主機/容器設定不支援必要的 Landlock/seccomp API，沙盒可能無法運作。此時建議您先將 Docker 容器設定為能提供所需的沙盒保障，再於容器內使用 `--sandbox danger-full-access`（或更簡單地使用 `--dangerously-bypass-approvals-and-sandbox` 旗標）執行 `codex`。

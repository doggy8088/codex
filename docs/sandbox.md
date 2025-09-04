## 沙盒與核准

### 核准模式

我們為 Codex 在您電腦上的運作方式選擇了一個強大的預設：`Auto`。在此核准模式下，Codex 可以自動讀取檔案、進行編輯，並在工作目錄中執行命令。然而，Codex 需要您的核准才能在工作目錄外運作或存取網路。

當您只想聊天，或者想在深入之前進行規劃時，您可以使用 `/approvals` 命令切換到 `Read Only` 模式。

如果您需要 Codex 讀取檔案、進行編輯和執行具有網路存取權限的命令，而無需核准，您可以使用 `Full Access`。在這樣做之前請謹慎行事。

#### 預設和建議

- Codex 預設在具有強力防護的沙盒中執行：它阻止編輯工作空間外的檔案，並阻斷網路存取，除非啟用。
- 啟動時，Codex 偵測資料夾是否受版本控制並建議：
  - 受版本控制的資料夾：`Auto`（工作空間寫入 + 按需求核准）
  - 非版本控制的資料夾：`Read Only`
- 工作空間包括目前目錄和暫存目錄（如 `/tmp`）。使用 `/status` 命令查看哪些目錄在工作空間中。
- 您可以明確設定這些：
  - `codex --sandbox workspace-write --ask-for-approval on-request`
  - `codex --sandbox read-only --ask-for-approval on-request`

### 可以在沒有任何核准的情況下執行嗎？

是的，您可以使用 `--ask-for-approval never` 停用所有核准提示。此選項適用於所有 `--sandbox` 模式，因此您仍對 Codex 的自主性等級有完全控制。它會在您提供的任何約束條件下盡力嘗試。

### 常見的沙盒 + 核准組合

| 意圖                                    | 旗標                                                                                            | 效果                                                                                            |
| --------------------------------------- | ----------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| 安全的唯讀瀏覽                          | `--sandbox read-only --ask-for-approval on-request`                                            | Codex 可以讀取檔案並回答問題。Codex 需要核准才能進行編輯、執行命令或存取網路。                 |
| 唯讀非互動式 (CI)                       | `--sandbox read-only --ask-for-approval never`                                                 | 僅讀取；絕不升級                                                                               |
| 讓它編輯儲存庫，如有風險則詢問          | `--sandbox workspace-write --ask-for-approval on-request`                                      | Codex 可以讀取檔案、進行編輯，並在工作空間中執行命令。Codex 需要核准才能在工作空間外執行動作或存取網路。 |
| 自動（預設）                            | `--full-auto`（等同於 `--sandbox workspace-write` + `--ask-for-approval on-failure`）         | Codex 可以讀取檔案、進行編輯，並在工作空間中執行命令。當沙盒命令失敗或需要升級時，Codex 需要核准。 |
| YOLO（不建議）                          | `--dangerously-bypass-approvals-and-sandbox`（別名：`--yolo`）                                 | 無沙盒；無提示                                                                                 |

> 注意：在 `workspace-write` 中，除非在配置中啟用（`[sandbox_workspace_write].network_access = true`），否則預設停用網路。

#### 在 `config.toml` 中微調

```toml
# 核准模式
approval_policy = "untrusted"
sandbox_mode    = "read-only"

# 全自動模式
approval_policy = "on-request"
sandbox_mode    = "workspace-write"

# 選用：在 workspace-write 模式中允許網路
[sandbox_workspace_write]
network_access = true
```

您也可以將預設儲存為 **設定檔**：

```toml
[profiles.full_auto]
approval_policy = "on-request"
sandbox_mode    = "workspace-write"

[profiles.readonly_quiet]
approval_policy = "never"
sandbox_mode    = "read-only"
```

### 實驗 Codex 沙盒

為了測試在 Codex 提供的沙盒下執行命令時會發生什麼，我們在 Codex CLI 中提供以下子命令：

```
# macOS
codex debug seatbelt [--full-auto] [COMMAND]...

# Linux
codex debug landlock [--full-auto] [COMMAND]...
```

### 平台沙盒詳細資訊

Codex 實作沙盒政策所使用的機制取決於您的作業系統：

- **macOS 12+** 使用 **Apple Seatbelt** 並透過 `sandbox-exec` 搭配設定檔 (`-p`) 執行命令，該設定檔對應於指定的 `--sandbox`。
- **Linux** 使用 Landlock/seccomp API 的組合來執行 `sandbox` 設定。

請注意，在容器化環境（如 Docker）中執行 Linux 時，如果主機/容器設定不支援必要的 Landlock/seccomp API，沙盒可能無法運作。在這種情況下，我們建議設定您的 Docker 容器以提供您所需的沙盒保證，然後在容器內使用 `--sandbox danger-full-access`（或更簡單地使用 `--dangerously-bypass-approvals-and-sandbox` 旗標）執行 `codex`。
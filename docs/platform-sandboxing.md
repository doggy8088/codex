### 平台沙盒詳細資訊

codex 用於實作沙盒政策的機制取決於您的作業系統：

- **macOS 12+** 使用 **Apple Seatbelt** 並使用 `sandbox-exec` 與對應於指定的 `--sandbox` 的設定檔（`-p`）來運行指令。
- **Linux** 使用 Landlock/seccomp API 的組合來執行 `sandbox` 配置。

請注意，在如 Docker 等容器化環境中運行 Linux 時，如果主機/容器配置不支援必要的 Landlock/seccomp API，沙盒可能無法工作。在這種情況下，我們建議配置您的 Docker 容器使其提供您所尋找的沙盒保證，然後在容器內使用 `--sandbox danger-full-access`（或更簡單的 `--dangerously-bypass-approvals-and-sandbox` 標誌）運行 `codex`。 
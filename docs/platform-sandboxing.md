### 平台沙盒詳細資訊

Codex 實作沙盒政策所使用的機制取決於您的作業系統：

- **macOS 12+** 使用 **Apple Seatbelt** 並透過 `sandbox-exec` 搭配設定檔 (`-p`) 執行命令，該設定檔對應於指定的 `--sandbox`。
- **Linux** 使用 Landlock/seccomp API 的組合來執行 `sandbox` 設定。

請注意，在容器化環境（如 Docker）中執行 Linux 時，如果主機/容器設定不支援必要的 Landlock/seccomp API，沙盒可能無法運作。在這種情況下，我們建議設定您的 Docker 容器以提供您所需的沙盒保證，然後在容器內使用 `--sandbox danger-full-access`（或更簡單地使用 `--dangerously-bypass-approvals-and-sandbox` 旗標）執行 `codex`。
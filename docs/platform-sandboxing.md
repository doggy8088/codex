### 各平台沙盒機制細節

Codex 採用的沙盒機制取決於您的作業系統：

- **macOS 12+**：使用 **Apple Seatbelt**，並透過 `sandbox-exec` 及對應 `--sandbox` 的設定檔（`-p`）執行指令。
- **Linux**：使用 Landlock/seccomp API 的組合來強制執行 `sandbox` 組態。

請注意：在 Docker 等容器化環境執行 Linux 時，若主機/容器的設定不支援必要的 Landlock/seccomp API，沙盒可能無法運作。此時建議您先將 Docker 容器設定為能提供所需的沙盒保障，再於容器內使用 `--sandbox danger-full-access`（或更簡單地使用 `--dangerously-bypass-approvals-and-sandbox` 旗標）執行 `codex`。

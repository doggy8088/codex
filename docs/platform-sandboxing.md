### 平台沙盒實作細節

Codex 用來實作沙盒策略的機制取決於您的作業系統：

- **macOS 12+** 使用 **Apple Seatbelt** 技術，並透過 `sandbox-exec` 搭配對應至所指定 `--sandbox` 的設定檔 (`-p`) 來執行指令。
- **Linux** 則結合使用 Landlock/seccomp API 來強制執行 `sandbox` 設定。

請注意，在 Docker 等容器化環境中執行 Linux 時，若主機/容器的設定不支援必要的 Landlock/seccomp API，沙盒機制可能無法運作。在這種情況下，我們建議您設定 Docker 容器，使其提供您所期望的沙盒保障，然後在容器內使用 `--sandbox danger-full-access`（或更簡單地使用 `--dangerously-bypass-approvals-and-sandbox` 旗標）來執行 `codex`。
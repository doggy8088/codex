## 安裝與建置

### 系統需求

| 需求                        | 詳細資訊                                                         |
| --------------------------- | --------------------------------------------------------------- |
| 作業系統                    | macOS 12+、Ubuntu 20.04+/Debian 10+，或 Windows 11 **透過 WSL2** |
| Git（選用，建議）           | 2.23+ 以支援內建 PR 協助功能                                   |
| RAM                         | 最少 4-GB（建議 8-GB）                                         |

### DotSlash

GitHub Release 也包含一個名為 `codex` 的 [DotSlash](https://dotslash-cli.com/) 檔案用於 Codex CLI。使用 DotSlash 檔案可以輕量地提交到原始碼控制中，確保所有貢獻者使用相同版本的可執行檔，無論他們使用什麼平台進行開發。

### 從原始碼建置

```bash
# 複製儲存庫並導航到 Cargo 工作空間的根目錄。
git clone https://github.com/openai/codex.git
cd codex/codex-rs

# 如有必要，安裝 Rust 工具鏈。
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
rustup component add rustfmt
rustup component add clippy

# 建置 Codex。
cargo build

# 使用範例提示啟動 TUI。
cargo run --bin codex -- "explain this codebase to me"

# 進行變更後，確保程式碼乾淨。
cargo fmt -- --config imports_granularity=Item
cargo clippy --tests

# 執行測試。
cargo test
```
## 安裝與建構

### 系統需求

| 需求                         | 說明                                                            |
| --------------------------- | --------------------------------------------------------------- |
| 作業系統                    | macOS 12+、Ubuntu 20.04+/Debian 10+，或透過 WSL2 的 Windows 11    |
| Git（選用，建議）           | 2.23+，以使用內建 PR 協助功能                                     |
| 記憶體                      | 至少 4 GB（建議 8 GB）                                           |

### DotSlash

GitHub Release 也包含一個名為 `codex` 的 [DotSlash](https://dotslash-cli.com/) 檔案。使用 DotSlash 可在版本控制中進行輕量提交，確保所有貢獻者在各自平台上都使用相同版本的可執行檔。

### 從原始碼建構

```bash
# 複製儲存庫並切換到 Cargo 工作區根目錄。
git clone https://github.com/openai/codex.git
cd codex/codex-rs

# 如有需要，安裝 Rust 工具鏈。
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
rustup component add rustfmt
rustup component add clippy

# 建構 Codex。
cargo build

# 以示例提示啟動 TUI。
cargo run --bin codex -- "explain this codebase to me"

# 變更後請確保程式碼乾淨。
cargo fmt -- --config imports_granularity=Item
cargo clippy --tests

# 執行測試。
cargo test
``` 

## 安裝與建置

### 系統需求

| 需求                        | 詳細資訊                                                      |
| --------------------------- | --------------------------------------------------------------- |
| 作業系統                    | macOS 12+、Ubuntu 20.04+/Debian 10+，或 Windows 11 **透過 WSL2** |
| Git (選用，建議)          | 2.23+，用於內建的 PR 輔助工具                                   |
| 記憶體                      | 最低 4-GB (建議 8-GB)                                         |

### DotSlash

GitHub Release 也包含一個名為 `codex` 的 Codex CLI [DotSlash](https://dotslash-cli.com/) 檔案。使用 DotSlash 檔案可以輕量地提交到原始碼控制，以確保所有貢獻者無論使用何種開發平台，都能使用相同版本的可執行檔。

### 從原始碼建置

```bash
# Clone the repository and navigate to the root of the Cargo workspace.
git clone https://github.com/openai/codex.git
cd codex/codex-rs

# Install the Rust toolchain, if necessary.
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"
rustup component add rustfmt
rustup component add clippy

# Build Codex.
cargo build

# Launch the TUI with a sample prompt.
cargo run --bin codex -- "explain this codebase to me"

# After making changes, ensure the code is clean.
cargo fmt -- --config imports_granularity=Item
cargo clippy --tests

# Run the tests.
cargo test
``` 
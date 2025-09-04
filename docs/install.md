## 安裝與建構

### 系統需求

| Requirement                 | Details                                                         |
| --------------------------- | --------------------------------------------------------------- |
| Operating systems           | macOS 12+, Ubuntu 20.04+/Debian 10+, or Windows 11 **via WSL2** |
| git (optional, recommended) | 2.23+ 為 built-在 PR helpers                                   |
| RAM                         | 4-GB minimum (8-GB recommended)                                 |

### DotSlash

GitHub 發布版本也包含一個名為 `codex` 的 [DotSlash](https://dotslash-cli.com/) 檔案供 Codex CLI 使用。使用 DotSlash 檔案可以對原始碼控制進行輕量級提交，以確保所有貢獻者使用相同版本的可執行檔，無論他們用於開發的平台為何。

### 從原始碼建構

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
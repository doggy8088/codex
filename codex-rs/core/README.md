# codex-core

此 crate 實作 Codex 的商業邏輯，供各種以 Rust 撰寫的 Codex UI 使用。

## 相依項目

請注意：`codex-core` 假設執行環境中具備某些輔助工具。目前包含：

### macOS

需存在 `/usr/bin/sandbox-exec`。

### Linux

當 `arg0` 為 `codex-linux-sandbox` 時，包含 `codex-core` 的可執行檔需執行等同於 `codex debug landlock` 的動作。詳情請見 `codex-arg0` crate。

### 所有平台

當 `arg1` 為 `--codex-run-as-apply-patch` 時，包含 `codex-core` 的可執行檔需模擬虛擬的 `apply_patch` CLI。詳情請見 `codex-arg0` crate。

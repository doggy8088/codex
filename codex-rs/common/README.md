# codex-common

此 crate 用於放置需在整個工作區多個 crate 間共用、但不適合放在 `core` 的工具程式。

對於較小範圍的工具功能，建議做法是在 `Cargo.toml` 的 `[features]` 新增 feature，並視情況在 `lib.rs` 以 `#[cfg]` 控制其啟用。

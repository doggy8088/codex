# oai-codex-ansi-escape

包裝 <https://crates.io/crates/ansi-to-tui> 功能的幾個小型輔助函式：

```rust
pub fn ansi_escape_line(s: &str) -> Line<'static>
pub fn ansi_escape<'a>(s: &'a str) -> Text<'a>
```

優點：

- `ansi_to_tui::IntoText` 不需在整個 TUI crate 內都引入作用域。
- 若 `IntoText` 回傳 `Err`，我們會 `panic!()` 並記錄，以便呼叫端不必處理。

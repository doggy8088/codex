# Rust/codex-rs

在 codex-rs 資料夾中，Rust 程式碼位於此處：

- Crate 名稱以 `codex-` 為前綴。例如，`core` 資料夾的 crate 名為 `codex-core`
- 當使用 format! 且您可以將變數內聯到 {} 中時，請務必這樣做。
- 永遠不要新增或修改任何與 `CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR` 或 `CODEX_SANDBOX_ENV_VAR` 相關的程式碼。
  - 您在沙盒中操作，每當您使用 `shell` 工具時，將設定 `CODEX_SANDBOX_NETWORK_DISABLED=1`。任何使用 `CODEX_SANDBOX_NETWORK_DISABLED_ENV_VAR` 的現有程式碼都是考慮到這一事實而編寫的。它通常用於提早退出測試，作者知道您由於沙盒限制而無法執行這些測試。
  - 同樣，當您使用 Seatbelt (`/usr/bin/sandbox-exec`) 產生程序時，子程序上將設定 `CODEX_SANDBOX=seatbelt`。想要自己執行 Seatbelt 的整合測試無法在 Seatbelt 下執行，因此對 `CODEX_SANDBOX=seatbelt` 的檢查也常用於適當地提早退出測試。

在 Rust 程式碼變更後自動執行 `just fmt`（在 `codex-rs` 目錄中）；無需請求核准執行它。在完成對 `codex-rs` 的變更之前，執行 `just fix -p <project>`（在 `codex-rs` 目錄中）來修復程式碼中的任何 linter 問題。偏好使用 `-p` 進行範圍限定以避免緩慢的工作空間範圍 Clippy 建構；只有在變更共享 crate 時才執行不帶 `-p` 的 `just fix`。此外，執行測試：

1. 為變更的特定專案執行測試。例如，如果在 `codex-rs/tui` 中進行了變更，執行 `cargo test -p codex-tui`。
2. 一旦通過，如果在 common、core 或 protocol 中進行了任何變更，使用 `cargo test --all-features` 執行完整的測試套件。
   當互動式執行時，在執行 `just fix` 和測試完成之前詢問使用者；`just fmt` 不需要核准。

## TUI 樣式慣例

請參閱 `codex-rs/tui/styles.md`。

## TUI code conventions

- Use concise styling helpers 來自 ratatui’s Stylize trait.
  - Basic spans: use "text".into()
  - Styled spans: use "text".red(), "text".green(), "text".magenta(), "text".dim(), etc.
  - Prefer these over constructing styles with `Span::styled` and `Style` directly.
  - 範例: patch summary file lines
    - Desired: vec!["  └ ".into(), "M".red(), " ".dim(), "tui/src/app.rs".dim()]

### TUI Styling (ratatui)

- Prefer Stylize helpers: use "text".dim(), .bold(), .cyan(), .italic(), .underlined() instead 的 manual Style 哪裡 possible.
- Prefer simple conversions: use "text".into() for spans and vec![…].into() for lines; when inference is ambiguous (e.g., Paragraph::new/Cell::from), use Line::from(spans) or Span::from(text).
- Computed styles: if the Style is computed at runtime, using `Span::styled` is OK (`Span::from(text).set_style(style)` is also acceptable).
- Avoid hardcoded white: do not use `.white()`; prefer the default foreground (no color).
- Chaining: combine helpers 透過 chaining 為 readability (e.g., url.cyan().underlined()).
- Single items: prefer "text".into(); use Line::來自(text) 或 Span::來自(text) only 當 the target type isn’t obvious 來自 context, 或 當 using .into() would require extra type annotations.
- Building lines: use vec![…].into() 到 construct a Line 當 the target type is obvious 和 no extra type annotations are needed; otherwise use Line::來自(vec![…]).
- Avoid churn: don’t refactor between equivalent forms (Span::styled ↔ set_style, Line::來自 ↔ .into()) without a clear readability 或 functional gain; follow file‑local conventions 和 do not introduce type annotations solely 到 satisfy .into().
- Compactness: prefer the form 那個 stays 在 one line after rustfmt; 如果 only one 的 Line::來自(vec![…]) 或 vec![…].into() avoids wrapping, choose 那個. 如果 both wrap, pick the one 使用 fewer wrapped lines.

## Snapshot tests

This repo uses snapshot tests (via `insta`), especially in `codex-rs/tui`, to validate rendered output. When UI or text output changes intentionally, update the snapshots as follows:

- 執行 tests 到 generate any updated snapshots:
  - `cargo test -p codex-tui`
- Check 什麼’s pending:
  - `cargo insta pending-snapshots -p codex-tui`
- Review changes by reading the generated `*.snap.new` files directly in the repo, or preview a specific file:
  - `cargo insta show -p codex-tui path/to/file.snap.new`
- Only 如果 您 intend 到 accept all new snapshots 在 這個 crate, 執行:
  - `cargo insta accept -p codex-tui`

如果 您 don’t have the tool:

- `cargo install cargo-insta`

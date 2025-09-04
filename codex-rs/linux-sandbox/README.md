# codex-linux-sandbox

此 crate 負責產生：

- 一個 Linux 用的獨立執行檔 `codex-linux-sandbox`，與 Node.js 版 Codex CLI 一起打包。
- 一個 lib crate，將可執行檔的商業邏輯以 `run_main()` 暴露，藉此：
  - `codex-exec` CLI 可檢查其 arg0 是否為 `codex-linux-sandbox`，若是則以該模式執行。
  - `codex` 多功能 CLI 亦同。
